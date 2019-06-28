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
## Creating Nodes and Relationships

### Creating nodes

```
CREATE (optionalVariable :optionalLabels {optionalProperties})
```
Example:
```
CREATE (m:Movie:Action {title: 'Batman Begins'})
RETURN m.title //optionally you can return
```

### Creating multiple nodes

```
CREATE
(:Person {name: 'Michael Caine', born: 1933}),
(:Person {name: 'Liam Neeson', born: 1952}),
(:Person {name: 'Katie Holmes', born: 1978}),
(:Person {name: 'Benjamin Melniker', born: 1913})
```

### Adding labels to a node

```
SET nodeVariable:Label         // adding one label to node referenced by the variable x
```
```
SET nodeVariable:Label1:Label2 // adding two labels to node referenced by the variable x
```
Example:
```
MATCH (m:Movie)
WHERE m.title = 'Batman Begins'
SET m:Action
RETURN labels(m)
```

#### Removing labels from a node

```
REMOVE nodeVariable:Label    // remove the label from the node referenced by the variable x
```
Example:
```
MATCH (m:Movie:Action)
WHERE m.title = 'Batman Begins'
REMOVE m:Action
RETURN labels(m)
```

### Adding properties to a node

```
SET nodeVariable.propertyName = value
```
```
SET nodeVariable.propertyName1 = value1    , nodeVariable.propertyName2 = value2
```
When you assign the new property values to the Node using a `=`, you must make sure that the new properties object includes all of the properties and their values for the node as the existing properties for the node get overwritten. Example:
```
SET x = {propertyName1: value1, propertyName2: value2}
```
However, if you specify `+=` when assigning to a property, the value at valueX is updated if the propertyNnameX exists for the node. If the propertyNameX does not exist for the node, then the property is added to the node. Example:
```
SET x += {propertyName1: value1, propertyName2: value2}
```

### Removing properties from a node

```
REMOVE x.propertyName
```
or
```
SET x.propertyName = null
```

### Creating relationships

```
CREATE (x)-[:REL_TYPE]->(y)
```
```
CREATE (x)<-[:REL_TYPE]-(y)
```

Example 1:
```
MATCH (a:Person), (m:Movie)
WHERE a.name = 'Michael Caine' AND m.title = 'Batman Begins'
CREATE (a)-[:ACTED_IN]->(m)
RETURN a, m
```
Example 2:
```
MATCH (a:Person), (m:Movie), (p:Person)
WHERE a.name = 'Liam Neeson' AND
      m.title = 'Batman Begins' AND
      p.name = 'Benjamin Melniker'
CREATE (a)-[:ACTED_IN]->(m)<-[:PRODUCED]-(p)
RETURN a, m, p
```

### Adding properties to relationships

```
SET r.propertyName = value
```
```
SET r.propertyName1 = value1    , r.propertyName2 = value2
```
```
SET r = {propertyName1: value1, propertyName2: value2}
```
```
SET r += {propertyName1: value1, propertyName2: value2}
```

Example:
```
MATCH (a:Person), (m:Movie)
WHERE a.name = 'Christian Bale' AND m.title = 'Batman Begins'
CREATE (a)-[rel:ACTED_IN]->(m)
SET rel.roles = ['Bruce Wayne','Batman']
RETURN a, m
```
You can even add properties to a relationship when the relationship is created:
```
MATCH (a:Person), (m:Movie)
WHERE a.name = 'Christian Bale' AND m.title = 'Batman Begins'
CREATE (a)-[:ACTED_IN {roles: ['Bruce Wayne', 'Batman']}]->(m)
RETURN a, m
```
You can test to see if the relationship exists before you create it as follows:

```
MATCH (a:Person),(m:Movie)
WHERE a.name = 'Christian Bale' AND
      m.title = 'Batman Begins' AND
      NOT exists((a)-[:ACTED_IN]->(m))
CREATE (a)-[rel:ACTED_IN]->(m)
SET rel.roles = ['Bruce Wayne','Batman']
RETURN a, rel, m
```

### Removing properties from a relationship

```
MATCH (a:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE a.name = 'Christian Bale' AND m.title = 'Batman Begins'
REMOVE rel.roles
RETURN a, rel, m
```

### Deleting relationships

```
MATCH (a:Person)-[rel:ACTED_IN]->(m:Movie)
WHERE a.name = 'Christian Bale' AND m.title = 'Batman Begins'
DELETE rel
RETURN a, m
```

### Deleting nodes

If you want to delete a node, delete its relationships first using `DETACH`.

Node without existing relationships:
```
MATCH (p:Person)
WHERE p.name = 'Liam Neeson'
DELETE p
```
Node with existing relationships:
```
MATCH (p:Person)
WHERE p.name = 'Liam Neeson'
DETACH DELETE  p
```

### Using `MERGE` to create nodes

```
MERGE (variable:Label{nodeProperties})
RETURN variable
```
Example:
```
MERGE (a:Actor {name: 'Michael Caine'})
SET a.born = 1933
RETURN a
```

### Using `MERGE` to create relationships

```
MERGE (variable:Label {nodeProperties})-[:REL_TYPE]->(otherNode)
RETURN variable
```
Example:
```
MATCH (p:Person), (m:Movie)
WHERE m.title = 'Batman Begins' AND p.name ENDS WITH 'Caine'
MERGE (p)-[:ACTED_IN]->(m)
RETURN p, m
```

### Specifying creation behavior when merging

`ON CREATE` will specify instructions to execute if no existing node is found during the `MERGE` call and needs to be created. Example:
```
MERGE (a:Person {name: 'Sir Michael Caine'})
ON CREATE SET a.birthPlace = 'London',
              a.born = 1934
RETURN a
```

You can also specify an `ON MATCH` clause during merge processing. If the exact node is found, you can update its properties, change labels, or do some other logic. Example:
```
MERGE (a:Person {name: 'Sir Michael Caine'})
ON CREATE SET a.born = 1934,
              a.birthPlace = 'UK'
ON MATCH SET a.birthPlace = 'UK'
RETURN a
```

## More about Cypher

### Cypher parameters

Parameters begin with the `$` symbol

```
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = $actorName
RETURN m.released, m.title ORDER BY m.released DESC
```
To set them before calling the query:
```
:param actorName => 'Tom Hanks'
```

### Analyzing Cypher execution

There are two Cypher keywords you can use to prefix a Cypher statement with to analyze a query:

- `EXPLAIN` gives an estimate of the graph engine processing that will occur if that statement is executed. Does NOT execute the Cypher statement.
- `PROFILE` gives also data about the performance of the query but the Cypher statement gets executed.

```
EXPLAIN MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name = $actorName AND
      m.released <  $year
RETURN p.name, m.title, m.released
```

### Managing constraints and node keys

Cypher can also be used to:
- **uniqueness constraint**. Makes sure a certain property is unique.
- **existence constraint**. Makes sure a certain property exists (is not null).
- **node key**. Ensures that a group of values of a node's properties is unique.

#### Ensuring that a property value for a node is unique

Node of type *Movie* must have each a unique *title*:
```
CREATE CONSTRAINT ON (m:Movie) ASSERT m.title IS UNIQUE
```

#### Ensuring that properties exist

All nodes of type *Movie* must have a *tagline* property:
```
CREATE CONSTRAINT ON (m:Movie) ASSERT exists(m.tagline)
```
All relationships of type *REVIEWED* must have a *rating* property:
```
CREATE CONSTRAINT ON ()-[rel:REVIEWED]-() ASSERT exists(rel.rating)

```

#### Retrieving constraints defined for the graph

Get back a list of all the constraints you defined:
```
CALL db.constraints()
```

#### Dropping constraints

Delete a constraint you defined:
```
DROP CONSTRAINT ON ()-[rel:REVIEWED]-() ASSERT exists(rel.rating)
```

#### Creating Node Keys

In every *Person* node the properties *name* and *born* must be different:
```
CREATE CONSTRAINT ON (p:Person) ASSERT (p.name, p.born) IS NODE KEY
```

### Managing Indexes

#### Indexes for range searches

```
MATCH (m:Movie)
WHERE 1990 < m.released < 2000
SET m.videoFormat = 'DVD'
```

#### Creating indexes

```
CREATE INDEX ON :Movie(released)
```

#### Retrieving indexes

```
CALL db.indexes()
```

#### Dropping indexes

```
DROP INDEX ON :Movie(released, videoFormat)
```

## Importing Data

In cypher you can:

- Load data from a URL (http(s) or file).
- Process data as a stream of records.
- Create or update the graph with the data being loaded.
- Use transactions during the load.
- Transform and convert values from the load stream.
- Load up to 10M nodes and relationships.

Example datasets:

**movies_to_load.csv**
```
id,title,country,year,summary
1,Wall Street,USA,1987, Every dream has a price.
2,The American President,USA,1995, Why can't the most powerful man in the world have the one thing he wants most?
3,The Shawshank Redemption,USA,1994, Fear can hold you prisoner. Hope can set you free.
```
**persons_to_load.csv**
```
Id,name,birthyear
1,Charlie Sheen, 1965
2,Oliver Stone, 1946
3,Michael Douglas, 1944
4,Martin Sheen, 1940
5,Morgan Freeman, 1937
```
**roles_to_load.csv**
```
personId,movieId,role
1,1,Bud Fox
4,1,Carl Fox
3,1,Gordon Gekko
4,2,A.J. MacInerney
3,2,President Andrew Shepherd
5,3,Ellis Boyd 'Red' Redding
```

### Importing normalized data

```
LOAD CSV WITH HEADERS FROM url-value //url-value can be a resource or a file on your system
AS row        // row is a variable that is used to extract data
```



Counting data from the dataset, to get an idea of how much data will get loaded.
```
LOAD CSV WITH HEADERS
FROM 'http://data.neo4j.com/intro-neo4j/movies_to_load.csv'
AS line
RETURN count(*)
```

Visually inspect a little piece of data to see if it is exactly as you expected it.
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/intro-neo4j/movies_to_load.csv'
AS line
RETURN * LIMIT 1
```

Formatting the data before it's loaded (notice the use of *trim()* to clean up strings from accidental whitespaces).
```
LOAD CSV WITH HEADERS
FROM 'http://data.neo4j.com/intro-neo4j/movies_to_load.csv'
AS line
RETURN line.id, line.title, toInteger(line.year), trim(line.summary)
```

This query specifically creates *Movie* nodes from the CSV data.
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/intro-neo4j/movies_to_load.csv'
AS line
CREATE (movie:Movie { movieId: line.id, title: line.title, released: toInteger(line.year) , tagline: trim(line.summary)})
```

This query will create the *Person* nodes. Notice we are using `MERGE` to avoid duplicate data.
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/intro-neo4j/persons_to_load.csv'
AS line
MERGE (actor:Person { personId: line.Id })
ON CREATE SET actor.name = line.name,
              actor.born = toInteger(trim(line.birthyear))
```

This query will create the *ACTED_IN* relationship.
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/intro-neo4j/roles_to_load.csv'
AS line
MATCH (movie:Movie { movieId: line.movieId })
MATCH (person:Person { personId: line.personId })
CREATE (person)-[:ACTED_IN { roles: [line.role]}]->(movie)
```

### Importing denormalized data

Dataset:

**movie_actor_roles_to_load.csv**
```
title;released;summary;actor;birthyear;characters
Back to the Future;1985;17 year old Marty McFly got home early last night. 30 years early.;Michael J. Fox;1961;Marty McFly
Back to the Future;1985;17 year old Marty McFly got home early last night. 30 years early.;Christopher Lloyd;1938;Dr. Emmet Brown
```

To load this data you use these Cypher statements:
```
LOAD CSV WITH HEADERS
FROM 'https://data.neo4j.com/intro-neo4j/movie_actor_roles_to_load.csv'
AS line FIELDTERMINATOR ';'
MERGE (movie:Movie { title: line.title })
ON CREATE SET movie.released = toInteger(line.released),
              movie.tagline = line.summary
MERGE (actor:Person { name: line.actor })
ON CREATE SET actor.born = toInteger(line.birthyear)
MERGE (actor)-[r:ACTED_IN]->(movie)
ON CREATE SET r.roles = split(line.characters,',')
```

### Importing a large dataset

⚠️ Uploading more than 100,000 rows of data? It is recommended to prefix your `LOAD CSV` clause with a `PERIODIC COMMIT` hint. This allows the database to regularly commit the import transactions to avoid memory churn for large transaction-states. ⚠️
