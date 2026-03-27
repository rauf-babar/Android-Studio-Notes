# SQLiteDatabase

**SQLite** is a relational database management system (RDBMS) contained in a C library. In Android, **`SQLiteDatabase`** is the class that manages the actual database file. It provides methods to execute SQL commands (Create, Read, Update, Delete).

To use it properly, we don't just create an `SQLiteDatabase` object directly. Instead, we create a helper class that extends **`SQLiteOpenHelper`**. This helper manages:

1. **Creating** the database (if it doesn't exist).
2. **Opening** connections.
3. **Upgrading** the structure (if you change the version number).

---

### `MyDatabaseHelper.java`


```java
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import androidx.annotation.Nullable;

import java.util.ArrayList;
import java.util.List;

public class MyDatabaseHelper extends SQLiteOpenHelper {

    // Database Info
    private static final String DB_NAME = "MyCollege.db";
    private static final int DB_VERSION = 1;

    // Table Info
    private static final String TABLE_NAME = "students";
    private static final String COL_ID = "id";
    private static final String COL_NAME = "name";
    private static final String COL_GPA = "gpa";

    public MyDatabaseHelper(@Nullable Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }

    // 1. CREATE TABLE (Runs once when app is installed/first run)
    @Override
    public void onCreate(SQLiteDatabase db) {

        // create all required tables 
        String createTableQuery = "CREATE TABLE " + TABLE_NAME + " (" +
                COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                COL_NAME + " TEXT, " +
                COL_GPA + " REAL)";
        db.execSQL(createTableQuery);
    }

    // 2. UPGRADE TABLE (Runs if you change DB_VERSION)
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // mention drop for all tables existing
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
        onCreate(db);
    }

    // =================================================================
    //  C R U D   O P E R A T I O N S
    // =================================================================

    // --- INSERT ---
    public boolean addStudent(String name, double gpa) {
        SQLiteDatabase db = this.getWritableDatabase();

        // METHOD A: Android Helper Way (Safe & Easy)
        ContentValues values = new ContentValues();
        values.put(COL_NAME, name);
        values.put(COL_GPA, gpa);

        // params: Table, NullColHack, Values
        // returns -1 if failed, or the new Row ID if successful
        long result = db.insert(TABLE_NAME, null, values);
        return result != -1;

        /* // METHOD B: Raw SQL Way
        // Note: Harder to handle standard quotes ' inside names (SQL Injection risk)
        String query = "INSERT INTO " + TABLE_NAME + " (" + COL_NAME + ", " + COL_GPA + ") VALUES ('" + name + "', " + gpa + ")";
        db.execSQL(query);
        return true; 
        */
    }

    // --- UPDATE ---
    public boolean updateStudentGPA(int id, double newGpa) {
        SQLiteDatabase db = this.getWritableDatabase();

        // METHOD A: Android Helper Way
        ContentValues values = new ContentValues();
        values.put(COL_GPA, newGpa);

        // params: Table, ContentValues, WhereClause, WhereArgs array
        int rowsAffected = db.update(TABLE_NAME, values, COL_ID + " = ?", new String[]{String.valueOf(id)});
        return rowsAffected > 0;

        /* // METHOD B: Raw SQL Way
        String query = "UPDATE " + TABLE_NAME + " SET " + COL_GPA + " = " + newGpa + " WHERE " + COL_ID + " = " + id;
        db.execSQL(query);
        return true;
        */
    }

    // --- DELETE ---
    public boolean deleteStudent(int id) {
        SQLiteDatabase db = this.getWritableDatabase();

        // METHOD A: Android Helper Way
        // params: Table, WhereClause, WhereArgs array
        int rowsDeleted = db.delete(TABLE_NAME, COL_ID + " = ?", new String[]{String.valueOf(id)});
        return rowsDeleted > 0;

        /* // METHOD B: Raw SQL Way
        String query = "DELETE FROM " + TABLE_NAME + " WHERE " + COL_ID + " = " + id;
        db.execSQL(query);
        return true;
        */
    }

    // --- GET ALL (READ) ---
    public Cursor getAllStudents() {
        SQLiteDatabase db = this.getReadableDatabase();

        // METHOD A: Android Helper Way (query)
        // Table, Columns (null=all), Selection, SelectionArgs, GroupBy, Having, OrderBy
        return db.query(TABLE_NAME, null, null, null, null, null, COL_ID + " DESC");

        /*
        // METHOD B: Raw SQL Way (rawQuery)
        return db.rawQuery("SELECT * FROM " + TABLE_NAME + " ORDER BY " + COL_ID + " DESC", null);
        */
    }

    public ArrayList<Student> getStudentList() {
         ArrayList<Student> studentList = new ArrayList<>();

         // 1. Get the Cursor
         Cursor cursor = getAllStudents();

        // Safety Check
        if (cursor == null) return studentList;

        // 2. Get Column Indices ONCE (Clean & Efficient)
        int idIdx = cursor.getColumnIndex(COL_ID);
        int nameIdx = cursor.getColumnIndex(COL_NAME);
        int gpaIdx = cursor.getColumnIndex(COL_GPA);
    
        // 3. FOR LOOP
        // Start: moveToFirst()
        // Stop:  isAfterLast() (Returns true when we go past the last row)
        // Step:  moveToNext()
        for (cursor.moveToFirst(); !cursor.isAfterLast(); cursor.moveToNext()) {
            
            int id = cursor.getInt(idIdx);
            String name = cursor.getString(nameIdx);
            double gpa = cursor.getDouble(gpaIdx);
    
            studentList.add(new Student(id, name, gpa));
        }

        // 4. Always close!
        cursor.close();
  
        return studentList;
    }

    // --- GET SPECIFIC (READ with Condition) ---
    public Cursor getStudentByName(String searchName) {
        SQLiteDatabase db = this.getReadableDatabase();

        // METHOD A: Android Helper Way
        // Condition: name = ?
        return db.query(TABLE_NAME, null, COL_NAME + " = ?", new String[]{searchName}, null, null, null);

        /*
        // METHOD B: Raw SQL Way
        return db.rawQuery("SELECT * FROM " + TABLE_NAME + " WHERE " + COL_NAME + " = ?", new String[]{searchName});
        */
    }
    
    // --- GET SPECIFIC (Multiple Conditions) ---
    public Cursor getSmartStudents(String name, double minGpa) {
        SQLiteDatabase db = this.getReadableDatabase();

        // Condition: name = ? AND gpa > ?
        String selection = COL_NAME + " = ? AND " + COL_GPA + " > ?";
        String[] selectionArgs = { name, String.valueOf(minGpa) };

        return db.query(TABLE_NAME, null, selection, selectionArgs, null, null, null);
    }
}

```

---
### `MainActivity.java`

```java
import android.database.Cursor;
import android.os.Bundle;
import android.util.Log;
import androidx.appcompat.app.AppCompatActivity;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    MyDatabaseHelper dbHelper;
    private static final String TAG = "DB_TEST";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new MyDatabaseHelper(this);

        // 1. INSERT DATA
        Log.d(TAG, "--- Inserting Data ---");
        dbHelper.addStudent("Ali", 3.2);
        dbHelper.addStudent("Sara", 3.9);
        dbHelper.addStudent("John", 2.5);

        // 2. UPDATE DATA (Update Ali's GPA)
        Log.d(TAG, "--- Updating Ali's GPA to 4.0 ---");
        // Assuming Ali has ID 1 (AutoIncrement starts at 1)
        dbHelper.updateStudentGPA(1, 4.0);

        // 3. DELETE DATA (Delete John)
        Log.d(TAG, "--- Deleting John (ID 3) ---");
        dbHelper.deleteStudent(3);

        // 4. GET ALL STUDENTS (Using your ArrayList function)
        Log.d(TAG, "--- Reading All Students ---");
        ArrayList<Student> list = dbHelper.getStudentList();

        for (Student s : list) {
            Log.d(TAG, "ID: " + s.id + ", Name: " + s.name + ", GPA: " + s.gpa);
        }

        // 5. GET SPECIFIC STUDENT (Using Cursor function)
        Log.d(TAG, "--- Searching for 'Sara' ---");
        Cursor cursor = dbHelper.getStudentByName("Sara");
        
        if (cursor != null && cursor.moveToFirst()) {
            String name = cursor.getString(cursor.getColumnIndex("name"));
            double gpa = cursor.getDouble(cursor.getColumnIndex("gpa"));
            Log.d(TAG, "Found: " + name + " with GPA: " + gpa);
            cursor.close();
        }
    }
}

```
-----
### Helper Methods vs. Raw SQL


| Operation | Method Name | Return Type | Syntax / Parameters | Example (Single & Multi Condition) |
| --- | --- | --- | --- | --- |
| **INSERT** | `db.insert()` | `long`<br><br>(New Row ID or -1) | **Table**: String<br><br>**NullColHack**: String (pass null)<br><br>**Values**: ContentValues | `insert("users", null, contentValues)` |
| **UPDATE** | `db.update()` | `int`<br><br>(Count of rows updated) | **Table**: String<br><br>**Values**: ContentValues<br><br>**WhereClause**: String ("id=?")<br><br>**WhereArgs**: String[] | **Single:** `"id=?"`, `new String[]{"1"}`<br><br>**Multi:** `"age > ? AND city = ?"`, `new String[]{"18", "Lahore"}` |
| **DELETE** | `db.delete()` | `int`<br><br>(Count of rows deleted) | **Table**: String<br><br>**WhereClause**: String<br><br>**WhereArgs**: String[] | **Single:** `"id=?"`, `new String[]{"5"}`<br><br>**Multi:** `"status=? OR score<?"`, `new String[]{"inactive", "50"}` |
| **READ** | `db.query()` | `Cursor`<br><br>(Result set pointer) | **Table**: String<br><br>**Columns**: String[] (null for *)<br><br>**Selection**: String (Where)<br><br>**SelectArgs**: String[]<br><br>**GroupBy/Having/Order**: Strings | **Single:** `"name=?"`, `new String[]{"Ali"}`<br><br>**Multi:** `"gpa > ? AND year = ?"`, `new String[]{"3.5", "2024"}` |
| **RAW SQL** | `db.execSQL()` | `void` | **SQL String**: Full Query<br><br>*(Only for INSERT, UPDATE, DELETE, CREATE)* | `execSQL("DELETE FROM users WHERE id=1")` |
| **RAW QUERY** | `db.rawQuery()` | `Cursor` | **SQL String**: Full Query<br><br>**SelectArgs**: String[] (For ? placeholders) | `rawQuery("SELECT * FROM users WHERE id=?", new String[]{"1"})` |

### Why use `?` (Placeholders)?

You notice I use `COL_ID + " = ?"` instead of `COL_ID + " = " + id`.
This is called **Parameter Binding**.

---

### Storage Classes

| SQLite Type | Description | Java Equivalent | Example Data |
| --- | --- | --- | --- |
| **`INTEGER`** | Whole numbers (positive or negative). | `int`, `long`, `boolean`* | `1`, `404`, `100293` |
| **`REAL`** | Numbers with decimals. | `double`, `float` | `3.14`, `500.25`, `0.05` |
| **`TEXT`** | Strings of text. | `String` | `"Ali"`, `"Lahore"`, `"0300-123"` |
| **`BLOB`** | **B**inary **L**arge **O**bject. Raw data bytes. | `byte[]` | Images, Audio files, PDF bytes |
| **`NULL`** | No value. | `null` | *(Empty cell)* |

---

### Two Important "Missing" Types

SQLite does **not** have dedicated types for Booleans or Dates. You must handle them using the 5 types above.

**1. Boolean (True/False)**

* **How to store:** Use `INTEGER`.
* `0` = `false`
* `1` = `true`


* **Java Logic:**
```java
// Reading from DB
boolean isPass = cursor.getInt(index) == 1; 

```
**2. Date & Time**

* **How to store:** Use `TEXT` or `INTEGER`.
* `TEXT` (Recommended): `"2023-12-25 14:30:00"` (ISO format).
* `INTEGER`: `1672531200` (Unix Timestamp - seconds since 1970).


* **Java Logic:** You save it as a String, and when you read it back, you use a `SimpleDateFormat` to convert it to a Date object.

-----
## **Inner Helper Class pattern** (DB Adapter pattern)

### Key Changes

1. **Outer Class (`MyDatabaseHelper`)**: No longer extends `SQLiteOpenHelper`. It is now just a manager class.
2. **Inner Class (`DbOpenHelper`)**: A `private static` class inside that extends `SQLiteOpenHelper`.
3. **Separate Pointers**: We create `writeDb` and `readDb` variables to hold the specific database instances.
4. **Open/Close**: We added `open()` and `close()` methods to manually manage the connection.

### 1. The Database Class (`MyDatabaseHelper.java`)

```java
import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import androidx.annotation.Nullable;
import java.util.ArrayList;

// Outer Class: Wraps the logic, does NOT extend SQLiteOpenHelper directly
public class MyDatabaseHelper {

    // Database Info
    private static final String DB_NAME = "MyCollege.db";
    private static final int DB_VERSION = 1;

    // Table Info
    private static final String TABLE_NAME = "students";
    private static final String COL_ID = "id";
    private static final String COL_NAME = "name";
    private static final String COL_GPA = "gpa";

    // --- SEPARATE POINTERS ---
    private final Context context;
    private DbOpenHelper dbOpenHelper; // The Inner Helper
    private SQLiteDatabase writeDb;    // Pointer for writing (Insert/Update/Delete)
    private SQLiteDatabase readDb;     // Pointer for reading (Select)

    // Constructor
    public MyDatabaseHelper(Context context) {
        this.context = context;
    }

    // MUST call this in Activity's onCreate()
    public MyDatabaseHelper open() {
        dbOpenHelper = new DbOpenHelper(context);
        writeDb = dbOpenHelper.getWritableDatabase();
        readDb = dbOpenHelper.getReadableDatabase();
        return this;
    }

    // MUST call this in Activity's onDestroy()
    public void close() {
        if (dbOpenHelper != null) {
            dbOpenHelper.close();
        }
    }

    private static class DbOpenHelper extends SQLiteOpenHelper {

        public DbOpenHelper(@Nullable Context context) {
            super(context, DB_NAME, null, DB_VERSION);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            String createTableQuery = "CREATE TABLE " + TABLE_NAME + " (" +
                    COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    COL_NAME + " TEXT, " +
                    COL_GPA + " REAL)";
            db.execSQL(createTableQuery);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
            onCreate(db);
        }
    }

    // --- INSERT (Uses writeDb) ---
    public boolean addStudent(String name, double gpa) {
        ContentValues values = new ContentValues();
        values.put(COL_NAME, name);
        values.put(COL_GPA, gpa);

        long result = writeDb.insert(TABLE_NAME, null, values);
        return result != -1;
    }

    // --- UPDATE (Uses writeDb) ---
    public boolean updateStudentGPA(int id, double newGpa) {
        ContentValues values = new ContentValues();
        values.put(COL_GPA, newGpa);

        int rowsAffected = writeDb.update(TABLE_NAME, values, COL_ID + " = ?", new String[]{String.valueOf(id)});
        return rowsAffected > 0;
    }

    // --- DELETE (Uses writeDb) ---
    public boolean deleteStudent(int id) {
        int rowsDeleted = writeDb.delete(TABLE_NAME, COL_ID + " = ?", new String[]{String.valueOf(id)});
        return rowsDeleted > 0;
    }

    // --- GET ALL (Uses readDb) ---
    public Cursor getAllStudents() {
        return readDb.query(TABLE_NAME, null, null, null, null, null, COL_ID + " DESC");
    }

    // --- GET LIST (Uses readDb) ---
    public ArrayList<Student> getStudentList() {
        ArrayList<Student> studentList = new ArrayList<>();
        Cursor cursor = getAllStudents();

        if (cursor == null) return studentList;

        int idIdx = cursor.getColumnIndex(COL_ID);
        int nameIdx = cursor.getColumnIndex(COL_NAME);
        int gpaIdx = cursor.getColumnIndex(COL_GPA);

        for (cursor.moveToFirst(); !cursor.isAfterLast(); cursor.moveToNext()) {
            int id = cursor.getInt(idIdx);
            String name = cursor.getString(nameIdx);
            double gpa = cursor.getDouble(gpaIdx);
            studentList.add(new Student(id, name, gpa));
        }
        cursor.close();
        return studentList;
    }

    // --- SEARCH (Uses readDb) ---
    public Cursor getStudentByName(String searchName) {
        return readDb.query(TABLE_NAME, null, COL_NAME + " = ?", new String[]{searchName}, null, null, null);
    }
}

```

### 2. The Main Activity (`MainActivity.java`)

**Key Change:** You must call `dbHelper.open()` in `onCreate` and `dbHelper.close()` in `onDestroy`.

```java
import android.database.Cursor;
import android.os.Bundle;
import android.util.Log;
import androidx.appcompat.app.AppCompatActivity;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    MyDatabaseHelper dbHelper;
    private static final String TAG = "DB_TEST";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 1. Initialize Wrapper
        dbHelper = new MyDatabaseHelper(this);

        // 2. OPEN CONNECTION (MANDATORY NOW)
        dbHelper.open();

        Log.d(TAG, "--- Inserting Data ---");
        dbHelper.addStudent("Ali", 3.2);
        dbHelper.addStudent("Sara", 3.9);
        dbHelper.addStudent("John", 2.5);

        Log.d(TAG, "--- Updating Ali's GPA to 4.0 ---");
        dbHelper.updateStudentGPA(1, 4.0);

        Log.d(TAG, "--- Deleting John (ID 3) ---");
        dbHelper.deleteStudent(3);

        Log.d(TAG, "--- Reading All Students ---");
        ArrayList<Student> list = dbHelper.getStudentList();
        for (Student s : list) {
            Log.d(TAG, "ID: " + s.id + ", Name: " + s.name + ", GPA: " + s.gpa);
        }

        Log.d(TAG, "--- Searching for 'Sara' ---");
        Cursor cursor = dbHelper.getStudentByName("Sara");
        if (cursor != null && cursor.moveToFirst()) {
            // Using simple hardcoded column names here for testing
            String name = cursor.getString(cursor.getColumnIndex("name")); 
            double gpa = cursor.getDouble(cursor.getColumnIndex("gpa"));
            Log.d(TAG, "Found: " + name + " with GPA: " + gpa);
            cursor.close();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 3. CLOSE CONNECTION (Prevents memory leaks)
        dbHelper.close();
    }
}

```

