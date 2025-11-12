#books #book_sumary #reference #spring_boot 

## Chapter 3 Working with Data

- When it comes to working with relational data, Java developers have several options. The two most common choices are JDBC and the JPA. Spring supports both of these with abstractions.
- Spring JDBC support is rooted in the `JdbcTemplate` class. `JdbcTemplate` provides a means by which developers can perform SQL operations against a relational database.
- `JdbcTemplate` is a helper class provided by the Spring Framework to make JDBC easier. Normally, raw JDBC requires you to manually handle database connections, statements, and result sets, which can be tedious and error-prone. `JdbcTemplate` abstracts that away.

Querying a database with `JDBCTemplate`.
```java
private JdbcTemplate jdbc; 

@Override 
public Ingredient findOne(String id) { 
	return jdbc.queryForObject( "select id, name, type from Ingredient where id=?", this::mapRowToIngredient, id); 
	} 

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException { 
	return new Ingredient( 
		rs.getString("id"), 
		rs.getString("name"), 
		Ingredient.Type.valueOf(rs.getString("type"))
	); 
}
```

- `jdbc.query()` - returns all rows which match the query. Returns `Iterable<DataType>`.

```java
@Override 
public Iterable findAll() { 
	return jdbc.query("select id, name, type from Ingredient",
	this::mapRowToIngredient); 
}
```

- Inserting data with `JDBCTemplate`.
```java
@Override 
public Ingredient save(Ingredient ingredient) { 
	jdbc.update( "insert into Ingredient (id, name, type) values (?, ?, ?)", 
		ingredient.getId(), 
		ingredient.getName(),
		ingredient.getType().toString()
	); 
	return ingredient; 
}

// The update() method, used when saving ingredient data, doesn’t help you get at the generated ID
```
- The update() method also can accept a `PreparedStatementCreator` and a `KeyHolder`. It’s the `KeyHolder` that will provide the generated taco ID. But in order to use it, you must also create a `PreparedStatementCreator`.
- Creating a `PreparedStatementCreator` is nontrivial. Start by creating a `PreparedStatementCreatorFactory`, giving it the SQL you want to execute, as well as the types of each query parameter. Then call `new PreparedStatementCreator`() on that factory, passing in the values needed in the query parameters to produce the `PreparedStatementCreator`.
```java
private long saveTacoInfo(Taco taco) { 
	taco.setCreatedAt(new Date()); 	
	PreparedStatementCreator psc = new PreparedStatementCreatorFactory( "insert into Taco (name, createdAt) values (?, ?)", Types.VARCHAR, Types.TIMESTAMP )
	
	.newPreparedStatementCreator( Arrays.asList( 
		taco.getName(), new Timestamp(taco.getCreatedAt().getTime()))); 
	KeyHolder keyHolder = new GeneratedKeyHolder(); jdbc.update(psc, keyHolder); 
	
	return keyHolder.getKey().longValue(); 
}
```
- If there’s a file named `schema.sql` in the root of the application’s classpath, then the SQL in that file will be executed against the database when the application starts. Spring Boot will also execute a file named `data.sql` from the root of the classpath when the application starts.
- `JDBCSimpleInsert`: `SimpleJdbcInsert` has a couple of useful methods for executing the insert: `execute()` and `executeAndReturnKey()`. Both accept a Map, where the map keys correspond to the column names in the table the data is inserted into. The map values are inserted into those columns.
```java
private long saveOrderDetails(Order order) {
	@SuppressWarnings("unchecked") 
	Map values = objectMapper.convertValue(order, Map.class); 
	values.put("placedAt", order.getPlacedAt()); 
	long orderId = orderInserter
		.executeAndReturnKey(values) 
		.longValue(); 
    return orderId; 
}

```
```java
private void saveTacoToOrder(Taco taco, long orderId) { 
	Map values = new HashMap<>(); 
	values.put("tacoOrder", orderId); 
	values.put("taco", taco.getId());
	orderTacoInserter.execute(values); 
}
```
