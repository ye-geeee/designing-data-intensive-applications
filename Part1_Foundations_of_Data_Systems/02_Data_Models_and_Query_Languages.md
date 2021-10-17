# Chapter 2. Data Models and Query Languages

1. [Relational Model Versus Document Model](#Relational-Model-Versus-Document-Model)
2. [The Birth of NoSQL](#The-Birth-of-NoSQL) 


3. [The Object-Relational Mismatch](#The-Object-Relational-Mismatch)
4. [Many-to-One and Many-to-Many Relationships](#Many-to-One-and-Many-to-Many-Relationships)

   
5. [Are Document Databases Repeating History?](#Are-Document-Databases-Repeating-History?)

In a complex application, there are many layers and each layers hides the complexity of the layers below it by providing a clean data model.  
These abstractions allow difference groups of people to work together effectively.  

There are many difference kinds of data models, and every data model embodies assumptions about how it is going to be used.  
Since the data model has a profound effect on what the software above it can and can't do, it's important to choose one that is appropriate to the application.  

<br/>

## Relational Model Versus Document Model

- 1970s ~ early 1980s : _network model_, _hierarchical model_
- late 1980s and early 1990s : Object databases
- early 2000s : XML databases

The best-known data model today is probably that of _SQL_, based on the relation model: data is organized into _relations_.  
The root of relational databases lie in _business data processing_. (today, _transaction processing, batch processing_).  
Now, relational databases turned out to generalize very well, beyond their original scope of business data processing.  

<br/>

## The Birth of NoSQL

In the 2010s, _NoSQL_ is the latest attempt to overthrow the relation model's dominance.  
Followings a re driving forces behind the adoptation of NoSQL.  

- A need fot greater scalability
- A widespread preference for free and open source software over commercial database products
- Specialized query operations
- Frustration with the restrictiveness of relation schemas
- Desire for a more dynamic and expressive data model

<br/>

### Resume example for further explanation

![02_linkedin_profile](../resources/part1/02_linkedin_profile.png)

## The Object Relational Mismatch

**_impedance mismatch_**: disconnect between the models  
If data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.  
Object-relation mapping(ORM) frameworks like `ActiveRecord` and `Hibernate` can be used, but they can not completely hide the differences between the two models.  

Therefore, in resume application  
The JSON representation has better _locality_ thant the multi-table schema.  
In the JSON representation, all the relevant information is in one place, and one query is sufficient.  

## Many to One and Many to Many Relationships

There are some advantages to having standardized lists:  
consistent style and spelling, avoiding ambiguity, ease of updating, localization support, better search

Advantage of using an ID
- it has no meaning to humans, it never needs to change
- no duplication 
- no write overhead when data changes
- no risk consistencies when data changes

Removing such duplication is the key idea behind _normalization_ in databases.  
And normalizing this data requires _many-to-one_ relationships.  

Even if resume application fits well in a join-free document model(JSON), 
data has a tendency of becoming more interconnected as features are added to applications.  
Therefore, it leads to _many-to-many_ model.  

<br/>

## Are Document Databases Repeating History? 

1970s, the most popular database for business data processing was IBM's _Information Management System (IMS)_.  
It used _hierarchical model_, which is similar to the JSON model.  
However, it worked well with one-to-many relationships, but not with many-to-many relationships.  
So, various solutions proposed and most prominent were _relational model_ and _network model_.  

### The network model

The network model was standardized by the Conference on Data Systems Languages(CODASYL), and it is also known as  _CODASYL_ model.  

A record could have multiple parents.  
The link between records was called _access path_ which works like pointers in a programming language.  
A query in CODASYL was performed by moving a cursor through the databases by iterating over lists of records and following access paths.  

However, it was like navigating around an n-dimensional data space and querying and updating the databases were complicated and inflexible.  
It was difficult to make changes to an application's data model.  

### The relational model

In contrast, the relational model lay out all the data in the open: relation(table) is simply a collection of tuples(rows).  
There are no labyrinthine nested structures, no complicated access paths to follow.  

In a relational database, the query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use.  
Thus relational model make it much easier to add new features to applications.  

