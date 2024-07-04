# Databases

Types:
1. Relational
2. Document
3. Graph

## 1. Relational

Commonly known as SQL, the relational database model organuizes data into relations (called tables) where each relatiuon is an unordered collection of tuples (rows).

Often requires ORM (object-relational mapping) frameworks to be used as a translation layer between the SQL data model and the objects in the application code.

Advantages:
- supports many-to-one relationships since it is normal to reference rows in other tables by ID -> joins are easy!

### When to use
- when the application uses many-to-many relationships


### Databases

MySQL, PostgreSQL, SQLite

### 

## 2. Document

### When to use
- applciation uses one-to-many relations
- Use when the application has a document-like structure (i.e. a tree of one-to-many relationships where the tree is typically all loaded in at once)
    - can be done with relational 
- the entire document needs to be accessed all at once (storage locality)
    - like for rendering on a web page
- no relationships between records

### Databases

MongoDB, RethiunkDB, CouchDB, Espresso

Advantages:
- offer schema flexibility
- for some applications, the serialized data can be closer to the data structures used by the application

Disadvantages:
- joins are not easily supported, therefore many-to-one relationships are
    - can be emulated through application code



## 3. Graph

Contains two types of entities: verteces (nodes) and edges (relationships or arcs)
- Examples:
    - Social graphs: verteces are people, and edges indicate which people know each other
    - The web graph: Vertices are web pages and edges represent HTML links to other pages (wikipedia is a graph, the internet is a graph, WWW duh!)
    - Road or rail networks: Verteces are junctions and edges are roads or railway lines between them

Provides a consistent way of storing completely different types of objects in a single datastore
- example: Facebook maintains a single grapth with many diofferent types of objects in a single datastore. Verteces reprecent people, locations, events, checkins, comments. Edges represent which people are friends with each other, which checkin happened at which locoation.

### When to use
- highly interconnected data (many-to-many relations)

## Other concepts

1. Normalization:
- idea that duplication is removed from a database
if you're duplicating values that could be stored in one place, the schema is not normalized
- Example:
    - storing city name as a str in a database row, as opposed to using an ID that maps to the city name in a separate table. 
    - If the city name ever changes, th hardcoded str would need to change everywhere it is used if the str was used as opposed to just an ID. 
        - If an ID was used then the table containing that ID/ str representation (normalized) is the only place it needs to change.

- Normalization requires many-to-one relationships (many people live in a city or many people work in a particular industry) which does not fit nicely into the document model.

2. Declarative vs imperative query languages
- SQL typically uses imperative query lenaguages that allow the db engine to optimize quieries