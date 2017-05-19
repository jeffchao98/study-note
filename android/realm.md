# Realm Study Note

## Outline
This study note is base on the experience when conduct a pre-research on Realm by implement the related sample code in Android(Java).

In this article, we will study the characteristic of realm by reference the common usage of SQLite.

## Summary
In SQLite, the developer MUST understand the SQL grammar, in order to compose the correct SQL statements, in addition to make the flow can execute SQL features correctly such as create table, query, upgrade table,...

In Realm, the developer only need to understand the keyword of SQL statement, because the library already prepare the APIs for frequently used SQL command, however, which also means if the api did not exist yet, the developer can not use the SQL feature in Realm.

## Basic Knowhow
The amount of the query per second: Realm( 31 )>SQLite( 14 )

The amount of the record insert per second: Realm( 94k )<SQLite( 178k )

## Create/Initialize
In SQLite, the developer need to put the proper statement which should base on the SQL grammar into the method ***SQLiteDatabase.execSQL()*** so the table can be created.

In Realm, the developer describe the structure of the table by define a new class, which should extends the class ***RealmObject***

## Write/Queries
In SQLite
```
(1) Use SQLiteOpenHelper extended custom-defined table class, and use the method getWritableDatabase()
then generate SQLiteDatabase item
(2) Use query method of the generated SQLiteDatabase item in order to create Cursor item, so the
flow can locate the records match the given conditions.
(3) Depends on the flow, execute the related api (update/insert)
(4) Close the generated Cursor item
(5) Close the generated SQLiteDatabase item
```

In Realm
```
(1) Use getInstance() method to get the Realm item
(2) Use Realm.beginTransaction() for start operate
(3) Use the related APIs to execute needed SQL command
(4) Use Realm.commitTransaction() for commit the change, or use Realm.cancelTransaction() to discard
the change
```

## Result
In SQLite, the flow should use the ***Cursor*** item to locate and get the result

In Realm, the flow use the method ***Realm.where([class name].class)*** in order to locate the specific class ( as know as the table class ), and store the content of the table inside the list item ***RealmQuery<[class name]>***, which the flow can search in detail from this item

## Restriction of the table class in Realm

### 1. Primary key and writing APIs
When write the data in the table, if your table class ***include @PrimaryKey statement***, the api recommended use is copyToRealmOrUpdate, as the example below:
For example, we have a table class named TestData, so the full procedure of the data writing should be:
```Java
TestData testData = new TestData();
realm.beginTransaction();
testData.setItemA(a);
testData.setItemB(b);
realm.copyToRealmOrUpdate(testData);
realm.commitTransaction();
```

If your table class ***has no @PrimaryKey statement involved***, the key api recommended use is createObject,
like the example below, if the table class still is TestData:
```Java
realm.beginTransaction();
TestData testData = realm.createObject(TestData.class);
testData.setItemA(a);
testData.setItemB(b);
realm.commitTransaction();
```

In some cases, you still can use createObject api instead of copyToRealmOrUpdate when @PrimaryKey statement involved,
as the example below shows:
```Java
realm.beginTransaction();
TestData testData = realm.createObject(TestData.class , [PrimaryKey Value]);
testData.setItemA(a);
testData.setItemB(b);
realm.commitTransaction();
```
However, this approach is only available for the action when add a new data in the table, the exception error will occurred if data updated which means the Primary value already exist in the table.

### 2. Table class change and database configuration
In Realm, If we can not always sure that the table structure always keep the same in every future build, we can setup ***RealmConfiguration*** item when every time we create Realm data in the flow.
As the initial step in SQLite, the developer can assign the ***database name*** and the ***database version number*** for distinguish the change, in order to keep the flow from occur the exception crash( io.realm.exceptions.RealmMigrationNeededException ) when the table structure changed in the new build.


However, in Realm, unless you want to discard all of the previous record when you made the change on the table class, which you can just change the database name for adapt the modified table class, otherwise, you have to define the migration flow in detail by create new ***RealmMigration*** item, and add this item into RealmConfiguration item when create Realm data.
As the example shows below:
```Java
Realm.init(context);
RealmConfiguration myConfig = new RealmConfiguration.Builder()
.name("[database name in String]")
.schemaVersion([database version number in int])
.migration(new MigrationItem())
.build();
realm = Realm.getInstance(myConfig);
```



## Under studying topics

A. The way to execute the writing in thread:
In Realm, we can use executeTransactionAsync:
```Java
realm.executeTransactionAsync(new Realm.Transaction() {
  @Override
  public void execute(Realm bgRealm) {
      //DB actions
    }
    }, [Optional]new Realm.Transaction.OnSuccess() {
    @Override
    public void onSuccess() {
      // Transaction was a success.
    }
    }, [Optional]new Realm.Transaction.OnError() {
      @Override
      public void onError(Throwable error) {
        // Transaction failed and was automatically canceled.
      }
  });
```
In SQLite, trying to find if any specific way to do, however in this case we just need to combine with regular solution in Android like thread or AsyncTask

B. For Realm, the restriction when access the DB from both background and foreground
