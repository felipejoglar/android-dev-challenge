# Lesson 8 - Storing data in SQLite

![Lesson 8 Banner](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/lesson-8-banner.png)


## Index

- [SQLite Database](#sqlite-database)
- [Creating the Database contract](#creating-the-database-contract)
- [Creating the Database](#creating-the-database)
- [Inserting data into the Database](#inserting-data-into-the-database)
- [Reading data from the Database](#reading-data-from-the-database)
- [Deleting data from the Database](#deleting-data-from-the-database)
- [Updating data in the Database and introducing transactions](#updating-data-in-the-database-and-introducing-transactions)
- [Introducing Unit tests](#introducing-unit-tests)


## SQLite Database

Saving data to a database is ideal for repeating or structured data, such as contact information. *SQLite* is an Open Source database. SQLite supports standard relational database features like SQL syntax, transactions and prepared statements. 

> SQLite is an embedded SQL database engine. Unlike most other SQL databases, SQLite does not have a separate server process. SQLite reads and writes directly to ordinary disk files.<br><br>&ndash; [SQLite webpage](http://sqlite.com/about.html)

The acronym "SQL" in SQLite means *Structured Query Language* which means that SQLite is a SQL compliant database engine which means that SQLite is used to store structured data, unlike `SharedPreference` which is used to store key-value pairs of data. 

We can find an useful SQL Cheatsheet [here](https://d17h27t6h515a5.cloudfront.net/topher/2016/September/57ed880e_sql-sqlite-commands-cheat-sheet/sql-sqlite-commands-cheat-sheet.pdf) and a deeper tutorial [here](https://www.w3schools.com/sql/).

In Android when we use an SQLite database, represented as an `SQLiteDatabase` object, all interactions with the database are through an instance of the `SQLiteOpenHelper` class which executes our requests and manages our database for us. Our app should only interact with the `SQLiteOpenHelper`. There are two data types associated with using SQLite databases in particular, `Cursor` and `ContentValues`.


## Creating the Database contract

One of the main principles of SQL databases is the schema, that is a formal declaration of how the database is organized. The schema is reflected in the SQL statements that we use to create our database. We may find it helpful to create a companion class, known as a *contract class*, which explicitly specifies the layout of our schema in a systematic and self-documenting way.

A contract class is a container for constants that define names for URIs, tables, and columns. The contract class allows us to use the same constants across all the other classes in the same package. This lets us change a column name in one place and have it propagate throughout our code.

A good way to organize a contract class is to put definitions that are global to our whole database in the root level of the class. Then create an inner class for each table that enumerates its columns.

``` java
public class WaitlistContract {

    // Create an inner class named WaitlistEntry class that implements the BaseColumns interface
    // By implementing the BaseColumns interface, our inner class can inherit a primary key 
    // field called _ID that some Android classes such as cursor adaptors will expect it to 
    // have. It's not required, but this can help our Database work harmoniously with the 
    // Android framework.
    public static final class WaitlistEntry implements BaseColumns {
        // Inside create a static final members for the table name and each of the db columns
        public static final String TABLE_NAME = "waitlist";
        public static final String COLUMN_GUEST_NAME = "guestName";
        public static final String COLUMN_PARTY_SIZE = "partySize";
        public static final String COLUMN_TIMESTAMP = "timestamp";
    }

}
```


## Creating the Database

Once we have defined how our database looks, we should implement methods that create and maintain the database and tables.

Just like files that we save on the device's internal storage, Android stores our database in private disk space that's associated application. Our data is secure, because by default this area is not accessible to other applications.

We can use a set of APIs available in `SQLiteOpenHelper` class to obtain references to our database, the system performs the potentially long-running operations of creating and updating the database only when needed and not during app startup. All we need to do is call `getWritableDatabase()` or `getReadableDatabase()`. Because they can be long-running operations, be sure that we call this methods in a background thread.

To use `SQLiteOpenHelper`, create a subclass that overrides the `onCreate()` and `onUpgrade()`. This open helper class also provides additional methods that we can override as needed.
- `onDowngrade()`, the default implementation rejects downgrades.
- `onConfigure()`, called before onCreate. Use this only to call methods that configure the parameters of the database connection.
- `onOpen()`, any work other than configuration that needs to be done before the database is opened.

``` java
// Extend the SQLiteOpenHelper class
public class WaitlistDbHelper extends SQLiteOpenHelper {

    // The Database name
    private static final String DATABASE_NAME = "waitlist.db";

    // If we change the Database schema, we must increment the Database version
    private static final int DATABASE_VERSION = 1;

    // Create a Constructor that takes a context and calls the parent constructor
    // Constructor
    public WaitlistDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    // Override the onCreate method
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        // Inside, create an String query called SQL_CREATE_WAITLIST_TABLE that will 
        // create the table
        final String SQL_CREATE_WAITLIST_TABLE = "CREATE TABLE " + WaitlistEntry.TABLE_NAME + " (" +
                WaitlistEntry._ID + " INTEGER PRIMARY KEY AUTOINCREMENT," +
                WaitlistEntry.COLUMN_GUEST_NAME + " TEXT NOT NULL, " +
                WaitlistEntry.COLUMN_PARTY_SIZE + " INTEGER NOT NULL, " +
                WaitlistEntry.COLUMN_TIMESTAMP + " TIMESTAMP DEFAULT CURRENT_TIMESTAMP" +
                ");";

        // Execute the query by calling execSQL on sqLiteDatabase and pass the string 
        // query SQL_CREATE_WAITLIST_TABLE
        sqLiteDatabase.execSQL(SQL_CREATE_WAITLIST_TABLE);
    }

    // Override the onUpgrade method
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        // For now simply drop the table and create a new one. This means if we change the
        // DATABASE_VERSION the table will be dropped.
        // In a production app, this method might be modified to ALTER the table
        // instead of dropping it, so that existing data is not deleted.
        sqLiteDatabase.execSQL("DROP TABLE IF EXISTS " + WaitlistEntry.TABLE_NAME);
        onCreate(sqLiteDatabase);
    }
}
```


## Inserting data into the Database

Insert data into the database by passing a `ContentValues` object to the `insert()` method. The open helper's `insert()` method calls `SQLiteDatabase.insert()`, which is a `SQLiteDatabase` convenience method to insert a row into the database. (It's a convenience method, because we do not have to write the SQL query.)

Similar to how extras stores data, an instance of `ContentValues` stores data as key-value pairs, where the key is the name of the column and the value is the value for the cell. One instance of `ContentValues` represents one row of a table.

The `insert()` method for the database requires that the values to fill a row are passed as an instance of ContentValues.

``` java
// Gets the data repository in write mode
SQLiteDatabase db = mDbHelper.getWritableDatabase();

// Create a new map of values, where column names are the keys
ContentValues values = new ContentValues();
values.put(WaitlistEntry.COLUMN_GUEST_NAME, name);
values.put(WaitlistEntry.COLUMN_PARTY_SIZE, partySize);

// Insert the new row, returning the primary key value of the new row
long newRowId = db.insert(WaitlistEntry.TABLE_NAME, null, values);
```

The first argument for `insert()` is simply the table name.

The second argument tells the framework what to do in the event that the `ContentValues` is empty. If we specify the name of a column, the framework inserts a row and sets the value of that column to null. If we specify null, like in the code sample, the framework does not insert a row when there are no values.


## Reading data from the Database

To read from a database, use the `query()` method, passing it our selection criteria and desired columns. The method combines elements of `insert()` and `update()`, except the column list defines the data we want to fetch, rather than the data to insert. The results of the query are returned to us in a `Cursor` object.

The `SQLiteDatabase` always presents the results as a `Cursor` in a table format that resembles that of a SQL database.

We can think of the data as an array of rows. A cursor is a pointer into one row of that structured data. The `Cursor` class provides methods for moving the cursor through the data structure, and methods to get the data from the fields in each row.

The `Cursor` class has a number of subclasses that implement cursors for specific types of data.

- `SQLiteCursor` exposes results from a query on a `SQLiteDatabase`. `SQLiteCursor` is not internally synchronized, so code using a `SQLiteCursor` from multiple threads should perform its own synchronization when using the `SQLiteCursor`.
- `MatrixCursor` is an all-rounder, a mutable cursor implementation backed by an array of objects that automatically expands internal capacity as needed.

Some common operations on cursor are:

- `getCount()` returns the number of rows in the cursor.
- `getColumnNames()` returns a string array holding the names of all of the columns in the result set in the order in which they were listed in the result.
- `getPosition()` returns the current position of the cursor in the row set.
- Getters are available for specific data types, such as `getString(int column)` and `getInt(int column)`.
- Operations such as `moveToFirst()` and `moveToNext()` move the cursor.
- `close()` releases all resources and makes the cursor completely invalid. Remember to call close to free resources.

``` java 
SQLiteDatabase db = mDbHelper.getReadableDatabase();

// Define a projection that specifies which columns from the database
// we will actually use after this query.
String[] projection = {
    WaitlistEntry._ID,
    WaitlistEntry.COLUMN_GUEST_NAME,
    WaitlistEntry.COLUMN_PARTY_SIZE
    };

// Filter results WHERE "name" = 'My Guest Name'
String selection = WaitlistEntry.COLUMN_GUEST_NAME + " = ?";
String[] selectionArgs = { "My Guest Name" };

// How we want the results sorted in the resulting Cursor
String sortOrder = WaitlistEntry.COLUMN_TIMESTAMP + " DESC";

Cursor cursor = db.query(
    WaitlistEntry.TABLE_NAME,                 // The table to query
    projection,                               // The columns to return
    selection,                                // The columns for the WHERE clause
    selectionArgs,                            // The values for the WHERE clause
    null,                                     // don't group the rows
    null,                                     // don't filter by row groups
    sortOrder                                 // The sort order
    );
```

When a method call returns a cursor, we iterate over the result, extract the data, do something with the data, and finally, we must close the cursor to release the memory. Failing to do so can crash our app when it runs out of memory.
The cursor starts before the first result row, so on the first iteration we move the cursor to the first result if it exists. If the cursor is empty, or the last row has already been processed, then the loop exits. Don't forget to close the cursor once we're done with it.

``` java
// Perform a query and store the result in a Cursor
Cursor cursor = db.query(...);
try {
    while (cursor.moveToNext()) {
        // Do something with the data
     }
} finally {
    cursor.close();
}
```

## Deleting data from the Database

To delete rows from a table, we need to provide selection criteria that identify the rows. We can delete using any criteria, and the method returns the number of items that were actually deleted.

The database API provides a mechanism for creating selection criteria that protects against [SQL injection](https://en.wikipedia.org/wiki/SQL_injection). The mechanism divides the selection specification into a selection clause and selection arguments. The clause defines the columns to look at. The arguments are values to test against that are bound into the clause. Because the result isn't handled the same as a regular SQL statement, it is immune to SQL injection.

``` java
// Define 'where' part of query.
String selection = WaitlistEntry.COLUMN_GUEST_NAME + " LIKE ?";
// Specify arguments in placeholder order.
String[] selectionArgs = { "My Guest Name" };
// Issue SQL statement.
db.delete(WaitlistEntry.TABLE_NAME, selection, selectionArgs);
```


## Updating data in the Database and introducing transactions

When we need to modify a subset of our database values, use the `update()` method.

Updating the table combines the content values syntax of `insert()` with the where syntax of `delete()`.

``` java
SQLiteDatabase db = mDbHelper.getWritableDatabase();

// New value for one column
ContentValues values = new ContentValues();
values.put(WaitlistEntry.COLUMN_GUEST_NAME, newName);

// Which row to update, based on the id
String selection = WaitlistEntry._ID + " = ";
String[] selectionArgs = { myId };

int count = db.update(
    WaitlistEntry.TABLE_NAME,
    values,
    selection,
    selectionArgs);
```

A *transaction* symbolizes a unit of work performed within a database management system against a database, and treated in a coherent and reliable way independent of other transactions. A transaction generally represents any change in a database.

We can use transactions:
- When performing multiple operations that all need to complete to keep database consistent, for example, updating pricing of related items for a sale event.
- To batch multiple independent operations to improve performance, such as mass inserts.

Transactions can be nested, and the `SQLiteDatabase` class provides additional methods to manage nested transactions.

``` java
db.beginTransaction();
try {
    // Do all the Database work for transaction, like insert, update, delete...
    // If all the operations ended successfully the transaction finish
    db.setTransactionSuccessful();
} finally {
    // If ANY of the operations did not end successfully, ALL the operations are 
    // canceled
    db.endTransaction();
}
```

A database transaction, by definition, must be atomic, consistent, isolated and durable. Database practitioners often refer to these properties of database transactions using the acronym *ACID*.


## Introducing Unit tests

Writing and running tests is a critical part of the software development process. Testing our code can help us catch issues early in development and improve the robustness of our code as our app gets larger and more complex. With tests in our code, we can exercise small portions of our app in isolation, and in an automatable and repeatable manner. Because the code we write to test our app doesn't end up in the production version of our app; it lives only on our development machine, alongside our app's code in Android Studio.

*Local unit tests* are tests that are compiled and run entirely on our local machine with the Java Virtual Machine (JVM). Use local unit tests to test the parts of our app (such as the internal logic) that do not need access to the Android framework or an Android device or emulator, or those for which we can create fake ("mock" or stub) objects that pretend to behave like the framework equivalents.

Unit tests should be the fundamental tests in our app testing strategy. By creating and running unit tests against our code, we can verify that the logic of individual functional code areas or units is correct. Running unit tests after every build helps us catch and fix problems introduced by code changes to our app.

This is a complex and deep theme, in this lesson we are only shown how to run the unit tests that are provided to us with the project. But this is a very interesting topic that we as software developers must learn, there are some resources about testing at the end of the [references](#references) section.

To run our local unit tests, use these steps:

- To run a single test, right-click that test method and select **Run**.
- To test all the methods in a test class, right-click the test file in the project view and select **Run**.
- To run all tests in a directory, right-click on the directory and select **Run tests**.

The project builds, if necessary, and the testing view appears at the bottom of the screen. If all the tests we ran are successful, the progress bar at the top of the view turns green. A status message in the footer also reports "Tests Passed."

![Run test view](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/run-test-ok.png)


### References
[SQL SQLite commands cheatsheet](https://d17h27t6h515a5.cloudfront.net/topher/2016/September/57ed880e_sql-sqlite-commands-cheat-sheet/sql-sqlite-commands-cheat-sheet.pdf)<br>
[SQL tutorial](https://www.w3schools.com/sql/)<br>
[Saving Data Using SQLite](https://developer.android.com/training/data-storage/sqlite.html)<br>
[`android.database.sqlite` reference](https://developer.android.com/reference/android/database/sqlite/package-summary.html)<br>
[`Cursor` reference](https://developer.android.com/reference/android/database/Cursor.html)<br>
[`ContentValues` reference](https://developer.android.com/reference/android/content/ContentValues.html)<br>
[`SQLiteDatabase` reference](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html)<br>
[Android SQLite database unit testing is easy](https://medium.com/@elye.project/android-sqlite-database-unit-testing-is-easy-a09994701162) by Elye<br>
[Fundamentals of Testing](https://developer.android.com/training/testing/fundamentals.html)<br>
[Android Testing Guide](https://ravidsrk.github.io/android-testing-guide/)<br>
[Android Developer Fundamentals Course - 3.2: Testing your App](https://google-developer-training.gitbooks.io/android-developer-fundamentals-course-concepts/content/en/Unit%201/32_c_testing_your_app.html)<br>
[Android Testing Codelab - Google Developers](http://bit.ly/23IfqMx)<br>

###### Note: the images of the headers used in this serie of articles are from Udacity's [Developing Android Apps Course](https://www.udacity.com/course/new-android-fundamentals--ud851)