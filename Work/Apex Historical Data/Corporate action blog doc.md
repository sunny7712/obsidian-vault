## Beyond the Read Path: Handling the Complexities of Corporate Actions and Data Integrity  
  
The Historical Data API was a success. Algo traders could finally backtest their strategies against 5 years of historical data with sub-second query performance. But serving data fast is only half the battle. The other half? Ensuring the data is *correct*.  
  
For financial data, "correct" means more than just accurate numbers. It means data that reflects the reality of the market over time. And that reality is constantly being altered by corporate actions.  
  
This isn't about a new feature. This is about the fundamental integrity of the data we serve. Without properly handling corporate actions, our 5-year historical data would be a minefield of misleading price and volume information, rendering any backtesting results dangerously inaccurate.  
  
---  
  
### The Corporate Action Challenge: A Wrinkle in Time  
  
Imagine you're backtesting a strategy on a stock. You see a sudden 50% price drop on a specific day. Your strategy might interpret this as a massive sell-off and a signal to sell. But what if it wasn't a sell-off? What if it was a 2-for-1 stock split?  
  
This is the challenge of corporate actions. Events like stock splits, dividends, rights issues, and mergers fundamentally change a stock's price and/or the number of shares outstanding. If you don't account for them, your historical data will be riddled with artificial jumps and drops that never actually occurred in the market.  
  
Our raw candle data, stored in Parquet files, was a perfect snapshot of the market *on a given day*. But it didn't account for the context that corporate actions provide. A simple example:  
  
*   **A 2-for-1 stock split:** The price is halved, and the volume is doubled for all historical data *before* the split.  
*   **A cash dividend:** The price is reduced by the dividend amount on the ex-dividend date.  
  
Simply storing and serving raw candle data was not enough. We needed a robust system to apply these adjustments, ensuring that a 5-year chart of a stock tells a consistent and accurate story.  
  
---  
  
### Defining "Correctness": What We Needed to Achieve  
  
Before we could build a solution, we had to define what a "correct" historical dataset looked like.  
  
*   **Accuracy:** The historical data must be adjusted for all relevant corporate actions. A stock's price history should be a smooth, continuous line, free of artificial gaps caused by splits or dividends.  
*   **Traceability:** We needed to know exactly which corporate actions were applied to the data and when. This is crucial for debugging and for providing transparency to our users.  
*   **Performance:** The adjustment process couldn't significantly slow down our data ingestion or query pipelines.  
*   **Flexibility:** The system had to be able to handle a variety of corporate actions, each with its own unique adjustment logic. It also needed to be extensible to support new types of corporate actions in the future.  
*   **Immutability of Raw Data:** We wanted to preserve the original, unadjusted candle data as the source of truth. This is a critical principle for data integrity and auditing.  
  
---  
  
### Architectural Approaches to Corporate Actions  
  
When deciding how to handle corporate actions, we considered two main architectural approaches:  
  
1.  **Pre-computation at Ingestion Time:** Applying adjustments *before* writing the data to storage.  
2.  **On-the-Fly Adjustment at Read Time:** Applying adjustments *after* reading the raw data from storage.  
  
Let's compare these two approaches.  
  
---   
### An Alternative Considered: Pre-computation at Ingestion Time  
  
In this model, the ingestion pipeline is responsible for fetching the raw candle data, applying any necessary adjustments, and then writing the final, *adjusted* data to the Parquet files in the persistent volume.  
  
This approach has one primary advantage: a simpler and faster read path. The API layer would not need to perform any on-the-fly calculations. It could simply query the Parquet files and return the data as-is.  
  
However, we ultimately decided against this approach for several critical reasons:  
  
*   **Loss of the Original Data:** This is the most significant drawback. If we only store the adjusted data, the original, raw candle data is lost forever. This is a major violation of data integrity principles and would make it impossible to debug issues with the adjustment logic or to regenerate the data with a different adjustment methodology.  
*   **Difficulty with Restatements:** Corporate action data is not always perfect. Sometimes, actions are announced late, or the details are corrected after the fact. In a pre-computation model, handling these restatements would require us to go back and re-process and re-write potentially years of historical Parquet files—a complex, expensive, and error-prone operation.   
*   **Lack of Flexibility:** What if a user wants the unadjusted data for their analysis? With the pre-computation approach, this would be impossible unless we were to store two separate datasets (adjusted and unadjusted), which would double our storage costs and add complexity to the ingestion pipeline.  
*   **Increased Ingestion Complexity:** The ingestion pipeline would become more complex and stateful. It would need to be aware of all corporate action history, which would slow it down and make it more difficult to maintain.  
  
  
---  
  
### Detailed Procedure: On-the-Fly Corporate Action Application  
  
We opted for an architecture that applies adjustments at read time. This approach, while more complex at the read layer, provides far greater flexibility and data integrity. Here is a detailed breakdown of the process.  
  
#### Step 1: Corporate Action Ingestion and Storage  
  
The foundation of our approach is a centralized service dedicated to managing corporate action data.  
  
1.  **Ingestion**: A periodic job fetches new corporate action events from an upstream source.  
2.  **Storage**: Each action is persisted in a relational database. A record contains the instrument's trading symbol, the execution date of the action, the type of action (like 'SPLIT' or 'BONUS'), and a flexible JSON field to store the specific parameters of that action (e.g., `{ "ratioNumerator": 2.0, "ratioDenominator": 1.0 }` for a 2-for-1 split). This JSON context is automatically serialized and deserialized into structured data transfer objects (DTOs) at runtime. An `UPSERT` operation is used to handle new or updated events.  
  
#### Step 2: In-Memory Caching for Performance  
  
To avoid hitting the database for every read request, we aggressively cache corporate action data in memory.  
  
1.  **Scheduled Loading**: The system pre-loads all active corporate actions into an in-memory cache upon application startup and refreshes this cache daily via a scheduled job.  
2.  **Optimized Data Structures**: Two crucial in-memory data structures are created:  
    *   **Symbol Change Graph**: A graph data structure is built in memory to handle symbol changes over time. This graph connects all historical trading symbols for a given instrument. For example, if a stock's symbol changed from 'ABC' to 'XYZ', the graph would link these two symbols. When a query for 'XYZ' comes in, this graph allows the system to know it also needs to look for data under the symbol 'ABC'.  
    *   **Date-Indexed Action Map**: A second, more critical data structure for performance is a time-series map, specifically a `TreeMap` where the keys are dates. Each date maps to another map, which in turn maps trading symbols to a list of corporate actions that occurred on that date. This structure (`TreeMap<LocalDate, Map<String, List<CorporateActionDto>>>`) allows the system to perform highly efficient range queries, such as 'get all corporate actions for symbols X, Y, and Z that occurred after January 1st, 2022'.  
  
#### Step 3: The Read and Transformation Process  
  
When a user requests historical data, the following sequence is executed:  
  
1.  **Symbol Expansion**: The read service first consults the in-memory symbol change graph to get a complete set of all historical symbols for the instrument in the user's request.  
2.  **Raw Data Fetch**: With this complete set of symbols, the service queries the underlying data store (the Parquet files) to fetch all raw, unadjusted candle data across all historical symbols for the requested time range.  
3.  **Sorting**: The collected raw candles are then sorted chronologically by their timestamp.  
4.  **Applying Transformations**: The sorted list of candles is then passed to a transformation engine. This engine first queries the in-memory, date-indexed action map to retrieve a small, relevant, and chronologically sorted list of corporate actions that apply to the symbols and time range of the candle data. The engine then iterates through each corporate action. For each action, it uses a factory pattern to select the appropriate transformation strategy based on the action's type (e.g., a 'Split' strategy, a 'Bonus' strategy). The selected strategy's `apply` method is then executed. This method iterates through the list of candles. For any candle whose timestamp is *before* the corporate action's execution date, it applies the corresponding mathematical adjustment. For example, a 'split' strategy might multiply the OHLC prices and divide the volume by a normalization factor. Because both the candles and the actions are processed in chronological order, this process naturally handles the cumulative effect of multiple corporate actions over many years.  
5.  **Final Response**: The result is a fully adjusted list of candles, which is then formatted and returned to the user.  
  
This architecture provides a powerful combination of data integrity, by never altering the raw source data, and performance, by leveraging in-memory caching and efficient, targeted transformations at read time.  
  
---  
  
### Key Takeaways  
  
1.  **Raw Data is a Liability Without Context:** For financial data, serving raw numbers is not enough. The context provided by corporate actions is essential for accuracy.  
2.  **On-the-Fly Adjustment is a Powerful Pattern:** Applying adjustments at read time, rather than creating new, adjusted datasets, is a flexible and efficient way to handle the dynamic nature of corporate actions.  
3.  **Isolate Complexity:** Corporate actions are a complex domain. Isolating this complexity into a dedicated service is a smart architectural choice that pays dividends in the long run.  
  
---  
  
### What's Next  
  
This system is already in production and has significantly improved the accuracy of our historical data. But we're not done yet. Here's what we're looking at next:  
  
*   **Handling More Complex Corporate Actions:** We're working on adding support for more complex events like mergers and acquisitions, which require more sophisticated adjustment logic.  
*   **User-Facing Options:** We're exploring the idea of giving users the option to request either adjusted or unadjusted data via the API.  
*   **Performance Optimizations:** We're continuously monitoring the performance of the on-the-fly adjustment process and looking for ways to make it even faster.  
  
---  
  
### Conclusion  
  
Building a historical data API that is both fast and accurate required us to think beyond the read path. By building a dedicated service to handle the complexities of corporate actions and applying adjustments on the fly, we were able to deliver a solution that is robust, flexible, and, most importantly, provides our users with data they can trust.