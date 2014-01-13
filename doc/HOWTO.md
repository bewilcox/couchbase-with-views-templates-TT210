Tutorial for the Telosys Couchbase bundle
===========================================

Note
-----
For this tutorial, we start from a derby database. This database manage a bookstore (Book, Author, Reviews etc ..). 
You can follow this tutorial with your own database.  
If you want to start with the same db : [https://dl.dropboxusercontent.com/u/52079610/BookstoreDB_v2_Derby10.9.zip](https://dl.dropboxusercontent.com/u/52079610/BookstoreDB_v2_Derby10.9.zip)  
The database name is bookstore and user/pwd is root/admin.

1. Project Creation
----------------

* You can start by create a new __maven__ project or start on an existing one.

2. Initialize Telosys Tools
----------------------------------
- Open the properties window of your project
- In the left entries, select __"Telosys Tools"__ and fill all project informations in "General", "Packages" and "Folders" tab (right side)
- Click on the __"Apply"__ button
- Click on __"Init Telosys Tools"__ in the "General" tab => __The "Telosys Tools" directory is now added in the project__
- In the "download" tab, download from this Github repository the __"couchbase-with-views-bundle-TT210"__
- Click on __"Download selected bundle(s)"__ and verify the checkbox __"Install downloaded bundle(s)"__ is checked.
- After this, close the project properties window.
- In the project, you should now have the __"couchbase-with-views-bundle-TT210"__ directory under __"TelosysTools/templates"__

3. Update maven dependencies
----------------------------------
- Under the __"couchbase-with-views-bundle-TT210"__ directory, open the __/misc/maven-pom.txt__
- Copy and paste the dependencies to your project pom (if they are not already exist)
- For this example, we use a dependency for the Derby driver. Adapt it to your database if needed.

4. Database repository generation
-------------------------------
- Open the database configuration file : "TelosysTools / databases.dbcfg"
- The database editor is displayed
- Add a new connection to your database
- Test the connection to get "Connection is OK"
- Click on "Meta-data" tab, fill the schema name and click on "Get tables" to see the tables of your database
- If all is OK, click on "Generate repository" button to generate a new ".dbrep" file in the "TelosysTools/" directory 

5. Manage entities links
-----------------------
- __All the links are generated__ automatically and they are visible under the the ".dbrep" file in the "TelosysTools/repos" directory, under the  __"Links between entities"__ tab. 

- As we say on the beginning, switch between a relational word to a NoSQL world is not trivial. To simplify this exploration, we manage the  
classic link (Lazy and Eager) as follows :  
__Lazy__ : Document contain the id of the link (ex : Book contain the author property)  
__Eager__ :Document contain the all object of the link (embedded doc) (ex: Author is part of the Book document)  
__We recommend to use Lazy at first__

6. Generate the Couchbase API
----------------------------
- Click on the __"Bulk generation"__ tab
- In the entities list, __select all entities__ you want to manage (be careful for links, __all linked entities must be selected__)
- Select the __"couchbase-with-views-bundle-TT210"__ templates bundle in the dropdown list => all templates are now visible in right list
- Select all templates and select __"copy static resources"__
- Click on the __"Generate"__ button
- You should have a successfull message
- The project source code __should compile successfully__

7. The persistence service
---------------------------
- For example, if in our database , we have a Book table, the generator create BookPersistence class.
- This class allows you to manage CRUD Operations like `findById()`, `removeAll()`, etc.
- It __generate for you, methods to retrieve all Many-To-One links__. For example, a Book contain the Author Id. So, in  
the BookPersistence class you will find the method `getBooksByAuthor(String id)`


8. The views
----------
- All needed views are generate in the __resources\couchbase\views__
- They are one js file per entity
- The file is a succession of blocks structured as follows :  
__Each block represent a view :__

```javascript
	//startview [view_name]
    function (doc, meta) {  
      if(meta.id.match("Author:")) (  
       emit(meta.id, null)  
      )  
    } 
    //endview 
```
- If you want to create a new view, you have to respect this syntax.
- You can add view on existing js files or create new ones.
- The `DatabaseConnectionProvider.initDatabase()` will create or update your modifications __directly on the Couchbase server__.


Configure the connection
-------------------------
- Open the __couchbase.properties__ file under the resources directory.
- You will find the following parameters :
- `couchbase.uris` : URL to the Couchbase server
- `couchbase.bucket` : Bucket name
- `couchbase.password` : Bucket password (empty if none)
- `couchbase.waitPersistOnDisk` : Use during document creation. It allows you to ensure that the document is persisted on disk on at least one server
- `couchbase.waitForIndexation` : Use during query. It allows to force update index before the view execution. You'll be ensure that all documents are indexed and processed by the view.
- `couchbase.database.version` : Version of your database. This version is checked by the `DatabaseConnectionProvider.initDatabase()` method.  
If versions are equal : no update view is performed  
If version on the file is newer than the document version created by this API : All views are created or updated.

Getting Started 
----------------
Here is an example for use this API.  

__Assert that :__ 
- Your Couchbase server is running.
- The bucket declared on the property file exist.
  
__Example with Book and Author entities :__ 
```java
public class Main {
	// Instanciate the connection provider
	// You can implement injection if you want
	DatabaseConnectionProvider databaseConnectionProvider = new CouchbaseConnectionProvider();
	
	// Initialize database
	// It will create or update all views declared on js files (only if the version on the property file is newer)
	databaseConnectionProvider.initDatabase();
	
	//Author creation
	Author author = new Author();
	author.setFirstName("John");
	author.setLastName("Doe");
	author.setId(1);
		
	// Book creation
	Book myBook = new Book();
	myBook.setId(1);
	myBook.setAuthor("1");
	myBook.setTitle("titre de mon livre");
	
	// Persist the documents
	BookPersistence bookPersistence = new BookPersistence(databaseConnectionProvider);
	AuthorPersistence authorPersistence = new AuthorPersistence(databaseConnectionProvider);
		
	authorPersistence.saveOrUpdate(author);
	bookPersistence.saveOrUpdate(myBook);
	
	// Get all books for a given author
	Collection<Book> books = bookPersistence.getBooksByAuthor(String.valueOf(author.getId()));
	
	// Close the connection
	databaseConnectionProvider.closeConnection();
}
```
Go to [http://localhost:8091/index.html](http://localhost:8091/index.html) to see what you have recorded.
Enjoy!