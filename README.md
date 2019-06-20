# Cypher Cheatsheet

## Reading Data (Basic Queries)

### Examining the Data Model

```
CALL db.schema
```

### Specifying a Node

```
()
(variable)
(:Label)
(variable:Label)
(:Label1:Label2)
(variable:Label1:Label2)
```

### Retrieve a Node

```
MATCH (variable)
RETURN variable
```
```
MATCH (variable:Label)
RETURN variable
```

### Retrieving Nodes Filtered by a Property Value

```
MATCH (variable {propertyKey: propertyValue})
RETURN variable
```
```
MATCH (variable:Label {propertyKey: propertyValue})
RETURN variable
```
```
MATCH (variable {propertyKey1: propertyValue1, propertyKey2: propertyValue2})
RETURN variable
```
```
MATCH (variable:Label {propertyKey: propertyValue, propertyKey2: propertyValue2})
RETURN variable
```

### Returning Property Values

```
MATCH (variable {prop1: value})
RETURN variable.prop2
```
```
MATCH (variable:Label {prop1: value})
RETURN variable.prop2
```
```
MATCH (variable:Label {prop1: value, prop2: value})
RETURN variable.prop3
```
```
MATCH (variable {prop1:value})
RETURN variable.prop2, variable.prop3
```

### Specifying Aliases

To customize the header titles of the data table.

```
MATCH (variable:Label {propertyKey1: propertyValue1})
RETURN variable.propertyKey2 AS alias2
```

### Querying using relationships

```
MATCH (node1)-[:REL_TYPE]->(node2)
RETURN node1, node2
```
```
MATCH (node1)-[:REL_TYPEA \| :REL_TYPEB]->(node2)
RETURN node1, node2
```
## Reading Data (Advanced Queries)

### Filtering queries using WHERE

Specify an or condition
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released = 2008 OR m.released = 2009
RETURN p, m
```
Specify a range
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released >= 2003 AND m.released <= 2004
RETURN p.name, m.title, m.released
```
Filter by specific property value
```
MATCH (p)-[:ACTED_IN]->(m)
WHERE p:Person AND m:Movie AND m.title='The Matrix'
RETURN p.name
```
Test existence of a property
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name='Jack Nicholson' AND exists(m.tagline)
RETURN m.title, m.tagline
```
Test strings' properties
```
MATCH (p:Person)-[:ACTED_IN]->()
WHERE p.name STARTS WITH 'Michael'
RETURN p.name
```
Use RegEx
```
MATCH (p:Person)
WHERE p.name =~'Tom.*'
RETURN p.name
```
Test list values
```
MATCH (p:Person)
WHERE p.born IN [1965, 1970]
RETURN p.name as name, p.born as yearBorn
```

### Setting path variables
```
MATCH megPath = (meg:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(d:Person),(other:Person)-[:ACTED_IN]->(m)
WHERE meg.name = 'Meg Ryan'
RETURN megPath
```

### Specifying varying length paths
```
MATCH (follower:Person)-[:FOLLOWS*2]->(p:Person)
WHERE follower.name = 'Paul Blythe'
RETURN p
```
Retrieve all paths of any length with the relationship, :RELTYPE from nodeA to nodeB and beyond:
```
(nodeA)-[:RELTYPE*]->(nodeB)
```

### Finding the Shortest Path

```
MATCH p = shortestPath((m1:Movie)-[*]-(m2:Movie))
WHERE m1.title = 'A Few Good Men' AND
      m2.title = 'The Matrix'
RETURN  p
```

### Optional Pattern Matching
```
MATCH (p:Person)
WHERE p.name STARTS WITH 'James'
OPTIONAL MATCH (p)-[r:REVIEWED]->(m:Movie)
RETURN p.name, type(r), m.title
```

### Aggregation
```
MATCH (a)-[:ACTED_IN]->(m)<-[:DIRECTED]-(d)
RETURN a.name, d.name, count(*)
```

### Aggregate results into a list
```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name ='Tom Cruise'
RETURN collect(m.title) AS `movies for Tom Cruise`
```

### Count data items
```
MATCH (actor:Person)-[:ACTED_IN]->(m:Movie)<-[:DIRECTED]-(director:Person)
RETURN actor.name, director.name, count(m) AS collaborations, collect(m.title) AS movies
```

### Additional processing using `WITH`
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH  a, count(a) AS numMovies, collect(m.title) as movies
WHERE numMovies > 1 AND numMovies < 4
RETURN a.name, numMovies, movies
```
```
Another examples:
MATCH (p:Person)
WITH p, size((p)-[:ACTED_IN]->(:Movie)) AS movies
WHERE movies >= 5
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
```

### Eliminate Duplication in Data
Using `DISTINCT`
```
MATCH (p:Person)-[:DIRECTED | :ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN m.released, collect(DISTINCT m.title) AS movies
```
Using `WITH` and `DISTINCT`
```
MATCH (p:Person)-[:DIRECTED | :ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
WITH DISTINCT m
RETURN m.released, m.title
```

### Ordering results
```
MATCH (p:Person)-[:DIRECTED | :ACTED_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN m.released, collect(DISTINCT m.title) AS movies ORDER BY m.released DESC
```

### Limiting the number of results
```
MATCH (m:Movie)
RETURN m.title as title, m.released as year ORDER BY m.released DESC LIMIT 10
```
Using `WITH`
```
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(*) AS numMovies, collect(m.title) as movies
WHERE numMovies = 5
RETURN a.name, numMovies, movies
```

### Unwinding Lists
Creating a list with three elements, unwinding it and then returning the values
```
WITH [1, 2, 3] AS list
UNWIND list AS row
RETURN list, row
```

### Dates
```
MATCH (actor:Person)-[:ACTED_IN]->(:Movie)
WHERE exists(actor.born)
// calculate the age
with DISTINCT actor, date().year  - actor.born as age
RETURN actor.name, age as `age today` ORDER BY actor.born DESC
```
