---
tags: [growth_tracking, work/groww]
---
# SDE-1, January 2023 to June 2023 Achievements

## Major Achievements

### Major Consumer-Payments Achievements

1. **Did not miss a single deadline set by the Product team, and helped them run experiments:** Every single product task given to me has been delivered timely, with complete ownership, and without issues. Other than this basic requirement, I have constantly aided the product team with running data-driven experiments and the implementing the resulting solutions. Eg. Phone number mapper VPA Validation static suffix retry. 

    **Impact:** This timely delivery and focus on quality control have helped meet product and business deadlines, while also providing the rest of the team plenty of time to build on top of the backend, and have extra time left over for their own end-to-end development cycles. The focus on aiding experimentation with the product team has helped improved the SR of various flows.

2. **Solution Building and Implementation of a thread-safe, extensible GRPC Stub Wrapper:** Ended up designing and implementing the final GRPC Wrapper solution after the team had been struggling with it. Later on, improved it further by defining stricter types of actions possible with it, and simplifying the process of layering behaviors onto the GRPC client.  

    **Impact:** The wrapper is used for every single GRPC call across consumer payments and it is the only reason solutions like our client-sided error handling, GRPC layer validation, deadline and retry injection, etc became possible to be implemented.

3. **Consumer Payments Error Management system:** Seeing the rate of error codes showing up in our system, I strongly fought for the implementation of a centralized error handling system, came to a consensus with the team on how it should be implemented, and finished implementing it end-to-end in a few days. It is capable of caching code mappings on the client side and pushing mapping updates to all clients that listen to it.  

    **Impact:** The system now manages and pushes updates for 1500+ error codes, enabling the product and dev team to make system-wide error message changes in seconds. It also enables dynamic changing of which error codes to trace in dashboards and alerts, which error codes to trigger circuit breakers on, etc.

4. **Extensive work on improving Observability:** Extensively learned Grafana UI, PromQL, LogQL, Golang Templating, etc. to figure out how to create dashboards and alerts in Grafana through both logs and Prometheus captured metrics. Using this knowledge, created multiple helpful alert channels and a deep-dive dashboard that gives a single-page view of every aspect of our Consumer Payment system and whatever could be going wrong in it. Moreover, existing alert thresholds were also extensively tuned by me to reduce the number of unnecessary alerts.  

    **Impact:** This has greatly reduced the load on service owners and on-call engineers as they don't have to navigate through multiple pages of Grafana, New Relic, and Percona to get visibility on things. The dashboard is a highly customizable, easy-to-view tool, that can aid in quick resolution. The alert channels themselves alert external party downtime and sudden unexpected error codes so we have timely readiness. The tuning of alert thresholds also helped make the lives of future on-calls easier

5. **Setting standards for On-call execution and Tracking:** I have set the standard for the tracking of On-call issues and tech-debt through Google sheet, Jira tickets, etc., and the process of weekly tech-debt pick up, which improves the overall health of the backend systems.  

    **Impact:** This has created a culture of efficient weekly on-call handoffs where the entire backend team gets visibility on weekly issues. The number of on-call tech debts picked and implemented by me, and the number of issues reported and resolved so far, is also huge.

6. **Ownership of UPI Sub-systems:** Payments Service, VPA Validation Service, Juspay YesBank Adapter Service, Webhook Manager Service, UPI TSP Client SDK, Error Management Service, etc.  

    **Impact:** While I have helped in creating and maintaining every single service of the UPI sub-pod and core common pieces, specifically for the above-mentioned services I have made major pieces and ensured long-term stability, and easy integration with external parties, as well as provided knowledge sharing across the team on their working, whenever needed.

7. **Implementations of multiple POCs for Backend:** Multi-Data Source Repository generator, Circuit Breaker, Custom Prometheus Events, Observability Dashboard, aiding in the testing of SSE POC by writing a custom Python load testing script, etc.  

    **Impact:** Helped the team quickly build up and run experiments to see what ideas are working well, what could be improved and how, and what ideas to discard.
