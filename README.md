couchbase-with-views-bundle (Only for Telosys Tools 2.1.0)
============================================================

Telosys tools 2.1.0 bundle for Couchbase. Include views generation and version management.

Important
----------
The transition from a relational database to a NoSQL document database is not a trivial thing and require to know   
the business context to achieve an appropriate document design (embedded document, duplication ...).   
This bundle can help you familiarize yourself with the NoSQL world by generating from your relational model a simple API for Couchbase.

This bundle allows you to generate for each of your database entities :
- Associated beans
- Persistence services for classic actions (get, findById, findAll, remove, removeAll, etc.)
- Couchbase views (to retrieve all documents by entity, Many to one relationships.
- Couchbase connection class (connection / disconnection, recording and updating views.)

Installation
-------------
If you have not yet installed the Telosys plugin, I invite you to follow this link : [https://sites.google.com/site/telosystools/](https://sites.google.com/site/telosystools/)  
and choose the bundle "couchbase-with-views-bundle-TT210"


Generated files
------------------

* __[table].java__ : POJOs from your database table
* __[table]Persistence.java__ : Persistence Service for CRUD operations.
* __CouchbaseConnectionProvider.java__ : Simple API for CRUD document operations, create or update views, manage connections.
* __couchbase.properties__ : Properties for using Couchbase (connection parameters, query parameters, version)
* __[table]_views.js__ : Couchbase views for an entity (map/reduce scripts to find all document by the type or by all Many-To-One relations) 

Tutorial
-------------
[How to use it !](doc/HOWTO.md)
