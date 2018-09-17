# Sqlite应用 {#sqlite应用}

## 1. 设计表结构： {#1-设计表结构：}

| Fields | Type | Key |
| :--- | :--- | :--- |
| id | INT | PRI |
| name | TEXT |  |
|  | phone number | TEXT |

## 2. 新建实体类 {#2-新建实体类}

```
public class Contact {

    //private variables
    int _id;
    String _name;
    String _phone_number;

    // Empty constructor
    public Contact(){

    }
    // constructor
    public Contact(int id, String name, String _phone_number){
        this._id = id;
        this._name = name;
        this._phone_number = _phone_number;
    }

    // constructor
    public Contact(String name, String _phone_number){
        this._name = name;
        this._phone_number = _phone_number;
    }
    // getting ID
    public int getID(){
        return this._id;
    }

    // setting id
    public void setID(int id){
        this._id = id;
    }

    // getting name
    public String getName(){
        return this._name;
    }

    // setting name
    public void setName(String name){
        this._name = name;
    }

    // getting phone number
    public String getPhoneNumber(){
        return this._phone_number;
    }

    // setting phone number
    public void setPhoneNumber(String phone_number){
        this._phone_number = phone_number;
    }
}

```

## 3. 数据库操作类（DataBase Handler） {#3-数据库操作类（database-handler）}

负责“增删改查”等操作，继承自 SQLiteOpenHelper。 重写 onCreate\(\) 和 onUpgrage\(\)，其中onCreate\(\)方法在数据库创建时调用，在此处写建表的代码。onUpgrage\(\)在数据库升级时调用，比如修改了表结构、增加了约束条件等。

```
public class DatabaseHandler extends SQLiteOpenHelper {

    // All Static variables
    // Database Version
    private static final int DATABASE_VERSION = 1;

    // Database Name
    private static final String DATABASE_NAME = "contactsManager";

    // Contacts table name
    private static final String TABLE_CONTACTS = "contacts";

    // Contacts Table Columns names
    private static final String KEY_ID = "id";
    private static final String KEY_NAME = "name";
    private static final String KEY_PH_NO = "phone_number";

    public DatabaseHandler(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    // Creating Tables
    @Override
    public void onCreate(SQLiteDatabase db) {
        String CREATE_CONTACTS_TABLE = "CREATE TABLE " + TABLE_CONTACTS + "("
                + KEY_ID + " INTEGER PRIMARY KEY," + KEY_NAME + " TEXT,"
                + KEY_PH_NO + " TEXT" + ")";
        db.execSQL(CREATE_CONTACTS_TABLE);
    }

    // Upgrading database
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // Drop older table if existed
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_CONTACTS);

        // Create tables again
        onCreate(db);
    }

```

### 3.1 增删改查的操作 {#31-增删改查的操作}

在 DatabaseHandler 类中增加以下方法

#### 3.1.1 插入新数据 {#311-插入新数据}

```
public void addContact(Contact contact) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_NAME, contact.getName()); // Contact Name
    values.put(KEY_PH_NO, contact.getPhoneNumber()); // Contact Phone Number

    // Inserting Row
    db.insert(TABLE_CONTACTS, null, values);
    db.close(); // Closing database connection
}

```

#### 3.1.2 查询数据 {#312-查询数据}

查询一条数据：

```
public Contact getContact(int id) {
    SQLiteDatabase db = this.getReadableDatabase();

    Cursor cursor = db.query(TABLE_CONTACTS, new String[] { KEY_ID,
            KEY_NAME, KEY_PH_NO }, KEY_ID + "=?",
            new String[] { String.valueOf(id) }, null, null, null, null);
    if (cursor != null)
        cursor.moveToFirst();

    Contact contact = new Contact(Integer.parseInt(cursor.getString(0)),
            cursor.getString(1), cursor.getString(2));

    return contact;
}

```

查询所有数据：

```
public List
<
Contact
>
 getAllContacts() {
    List
<
Contact
>
 contactList = new ArrayList
<
Contact
>
();
    // Select All Query
    String selectQuery = "SELECT  * FROM " + TABLE_CONTACTS;

    SQLiteDatabase db = this.getWritableDatabase();
    Cursor cursor = db.rawQuery(selectQuery, null);

    // looping through all rows and adding to list
    if (cursor.moveToFirst()) {
        do {
            Contact contact = new Contact();
            contact.setID(Integer.parseInt(cursor.getString(0)));
            contact.setName(cursor.getString(1));
            contact.setPhoneNumber(cursor.getString(2));
            // Adding contact to list
            contactList.add(contact);
        } while (cursor.moveToNext());
    }

    // return contact list
    return contactList;
}

```

获取数据条数：

```
 public int getContactsCount() {
        String countQuery = "SELECT  * FROM " + TABLE_CONTACTS;
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(countQuery, null);
        cursor.close();

        // return count
        return cursor.getCount();
    }

```

#### 3.1.3 更新一条数据 {#313-更新一条数据}

```
public int updateContact(Contact contact) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_NAME, contact.getName());
    values.put(KEY_PH_NO, contact.getPhoneNumber());

    // updating row
    return db.update(TABLE_CONTACTS, values, KEY_ID + " = ?",
            new String[] { String.valueOf(contact.getID()) });
}

```

#### 3.1.4 删除一条数据 {#314-删除一条数据}

```
public void deleteContact(Contact contact) {
    SQLiteDatabase db = this.getWritableDatabase();
    db.delete(TABLE_CONTACTS, KEY_ID + " = ?",
            new String[] { String.valueOf(contact.getID()) });
    db.close();
}

```

## 4. 在 activity 中调用 {#4-在-activity-中调用}

```
package com.androidhive.androidsqlite;

import java.util.List;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.widget.TextView;

public class AndroidSQLiteTutorialActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        DatabaseHandler db = new DatabaseHandler(this);

        /**
         * CRUD Operations
         * */
        // Inserting Contacts
        Log.d("Insert: ", "Inserting .."); 
        db.addContact(new Contact("Ravi", "9100000000"));        
        db.addContact(new Contact("Srinivas", "9199999999"));
        db.addContact(new Contact("Tommy", "9522222222"));
        db.addContact(new Contact("Karthik", "9533333333"));

        // Reading all contacts
        Log.d("Reading: ", "Reading all contacts.."); 
        List
<
Contact
>
 contacts = db.getAllContacts();       

        for (Contact cn : contacts) {
            String log = "Id: "+cn.getID()+" ,Name: " + cn.getName() + " ,Phone: " + cn.getPhoneNumber();
                // Writing Contacts to log
        Log.d("Name: ", log);
    }
    }
}

```

# 一个数据库中有多张表的情况 {#一个数据库中有多张表的情况}

以应用Todo为例,该应用有两个功能：创建待办事项，给待办事项添加标签。

## 1. 设计表结构 {#1-设计表结构}

实现上述两个功能需要3张表： 表1 todos，存储待办事项； 表2 tags，存储标签列表； 表3 todo\_tags，存储关联到待办事项的标签。

| Fields | Type | Key |
| :--- | :--- | :--- |
| id | INT | PK |
| note | TEXT |  |
|  | created\_at | DATETIME |

| Fields | Type | Key |
| :--- | :--- | :--- |
| id | INT | PK |
| tag\_name | TEXT |  |
|  | created\_at | DATETIME |

| Fields | Type | Key |
| :--- | :--- | :--- |
| id | INT | PK |
| todo\_id | TEXT | FK |
| tag\_id | DATETIME | FK |

* PK: Primary Key
* FK: Foreign Key

## 2. 创建实体类 {#2-创建实体类}

```
public class Todo {

    int id;
    String note;
    int status;
    String created_at;

    // constructors
    public Todo() {
    }

    public Todo(String note, int status) {
        this.note = note;
        this.status = status;
    }

    public Todo(int id, String note, int status) {
        this.id = id;
        this.note = note;
        this.status = status;
    }

    // setters
    public void setId(int id) {
        this.id = id;
    }

    public void setNote(String note) {
        this.note = note;
    }

    public void setStatus(int status) {
        this.status = status;
    }

    public void setCreatedAt(String created_at){
        this.created_at = created_at;
    }

    // getters
    public long getId() {
        return this.id;
    }

    public String getNote() {
        return this.note;
    }

    public int getStatus() {
        return this.status;
    }
}

```

```
public class Tag {

    int id;
    String tag_name;

    // constructors
    public Tag() {

    }

    public Tag(String tag_name) {
        this.tag_name = tag_name;
    }

    public Tag(int id, String tag_name) {
        this.id = id;
        this.tag_name = tag_name;
    }

    // setter
    public void setId(int id) {
        this.id = id;
    }

    public void setTagName(String tag_name) {
        this.tag_name = tag_name;
    }

    // getter
    public int getId() {
        return this.id;
    }

    public String getTagName() {
        return this.tag_name;
    }
}

```

## 3. 创建数据库操作类 {#3-创建数据库操作类}

```
public class DatabaseHelper extends SQLiteOpenHelper {

    // Logcat tag
    private static final String LOG = "DatabaseHelper";

    // Database Version
    private static final int DATABASE_VERSION = 1;

    // Database Name
    private static final String DATABASE_NAME = "contactsManager";

    // Table Names
    private static final String TABLE_TODO = "todos";
    private static final String TABLE_TAG = "tags";
    private static final String TABLE_TODO_TAG = "todo_tags";

    // Common column names
    private static final String KEY_ID = "id";
    private static final String KEY_CREATED_AT = "created_at";

    // NOTES Table - column nmaes
    private static final String KEY_TODO = "todo";
    private static final String KEY_STATUS = "status";

    // TAGS Table - column names
    private static final String KEY_TAG_NAME = "tag_name";

    // NOTE_TAGS Table - column names
    private static final String KEY_TODO_ID = "todo_id";
    private static final String KEY_TAG_ID = "tag_id";

    // Table Create Statements
    // Todo table create statement
    private static final String CREATE_TABLE_TODO = "CREATE TABLE "
            + TABLE_TODO + "(" + KEY_ID + " INTEGER PRIMARY KEY," + KEY_TODO
            + " TEXT," + KEY_STATUS + " INTEGER," + KEY_CREATED_AT
            + " DATETIME" + ")";

    // Tag table create statement
    private static final String CREATE_TABLE_TAG = "CREATE TABLE " + TABLE_TAG
            + "(" + KEY_ID + " INTEGER PRIMARY KEY," + KEY_TAG_NAME + " TEXT,"
            + KEY_CREATED_AT + " DATETIME" + ")";

    // todo_tag table create statement
    private static final String CREATE_TABLE_TODO_TAG = "CREATE TABLE "
            + TABLE_TODO_TAG + "(" + KEY_ID + " INTEGER PRIMARY KEY,"
            + KEY_TODO_ID + " INTEGER," + KEY_TAG_ID + " INTEGER,"
            + KEY_CREATED_AT + " DATETIME" + ")";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        // creating required tables
        db.execSQL(CREATE_TABLE_TODO);
        db.execSQL(CREATE_TABLE_TAG);
        db.execSQL(CREATE_TABLE_TODO_TAG);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // on upgrade drop older tables
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_TODO);
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_TAG);
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_TODO_TAG);

        // create new tables
        onCreate(db);
    }

```

## 4.1 todo表的增删改查的操作 {#41-todo表的增删改查的操作}

### 4.1.1 在todo表中增加一条数据 {#411-在todo表中增加一条数据}

```
public long createToDo(Todo todo, long[] tag_ids) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TODO, todo.getNote());
    values.put(KEY_STATUS, todo.getStatus());
    values.put(KEY_CREATED_AT, getDateTime());

    // insert row
    long todo_id = db.insert(TABLE_TODO, null, values);

    // 给新建的待办事项添加标签，在todo_tags表中增加相应数据
    for (long tag_id : tag_ids) {
        createTodoTag(todo_id, tag_id);
    }

    return todo_id;
}

```

### 4.1.2 从todo表中查询一条数据 {#412-从todo表中查询一条数据}

SELECT \* FROM todos WHERE id = 1;

```
public Todo getTodo(long todo_id) {
    SQLiteDatabase db = this.getReadableDatabase();

    String selectQuery = "SELECT  * FROM " + TABLE_TODO + " WHERE "
            + KEY_ID + " = " + todo_id;

    Log.e(LOG, selectQuery);

    Cursor c = db.rawQuery(selectQuery, null);

    if (c != null)
        c.moveToFirst();

    Todo td = new Todo();
    td.setId(c.getInt(c.getColumnIndex(KEY_ID)));
    td.setNote((c.getString(c.getColumnIndex(KEY_TODO))));
    td.setCreatedAt(c.getString(c.getColumnIndex(KEY_CREATED_AT)));

    return td;
}

```

### 4.1.3 查询todo表中的所有数据 {#413-查询todo表中的所有数据}

SELECT \* FROM todos;

```
public List
<
Todo
>
 getAllToDos() {
    List
<
Todo
>
 todos = new ArrayList
<
Todo
>
();
    String selectQuery = "SELECT  * FROM " + TABLE_TODO;

    Log.e(LOG, selectQuery);

    SQLiteDatabase db = this.getReadableDatabase();
    Cursor c = db.rawQuery(selectQuery, null);

    // looping through all rows and adding to list
    if (c.moveToFirst()) {
        do {
            Todo td = new Todo();
            td.setId(c.getInt((c.getColumnIndex(KEY_ID))));
            td.setNote((c.getString(c.getColumnIndex(KEY_TODO))));
            td.setCreatedAt(c.getString(c.getColumnIndex(KEY_CREATED_AT)));

            // adding to todo list
            todos.add(td);
        } while (c.moveToNext());
    }

    return todos;
}

```

### 4.1.4 查询有相同标签的todo数据 {#414-查询有相同标签的todo数据}

SELECT \* FROM todos td, tags tg, todo\_tags tt WHERE tg.tag\_name = ‘Watchlist’ AND tg.id = tt.tag\_id AND td.id = tt.todo\_id;

```
/*
 * getting all todos under single tag
 * */
public List
<
Todo
>
 getAllToDosByTag(String tag_name) {
    List
<
Todo
>
 todos = new ArrayList
<
Todo
>
();

    String selectQuery = "SELECT  * FROM " + TABLE_TODO + " td, "
            + TABLE_TAG + " tg, " + TABLE_TODO_TAG + " tt WHERE tg."
            + KEY_TAG_NAME + " = '" + tag_name + "'" + " AND tg." + KEY_ID
            + " = " + "tt." + KEY_TAG_ID + " AND td." + KEY_ID + " = "
            + "tt." + KEY_TODO_ID;

    Log.e(LOG, selectQuery);

    SQLiteDatabase db = this.getReadableDatabase();
    Cursor c = db.rawQuery(selectQuery, null);

    // looping through all rows and adding to list
    if (c.moveToFirst()) {
        do {
            Todo td = new Todo();
            td.setId(c.getInt((c.getColumnIndex(KEY_ID))));
            td.setNote((c.getString(c.getColumnIndex(KEY_TODO))));
            td.setCreatedAt(c.getString(c.getColumnIndex(KEY_CREATED_AT)));

            // adding to todo list
            todos.add(td);
        } while (c.moveToNext());
    }

    return todos;
}

```

### 4.1.5 更新一条todo数据，只更新内容,不更新标签 {#415-更新一条todo数据，只更新内容不更新标签}

```
public int updateToDo(Todo todo) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TODO, todo.getNote());
    values.put(KEY_STATUS, todo.getStatus());

    // updating row
    return db.update(TABLE_TODO, values, KEY_ID + " = ?",
            new String[] { String.valueOf(todo.getId()) });
}

```

### 4.1.6 删除一条todo数据 {#416-删除一条todo数据}

```
public void deleteToDo(long tado_id) {
    SQLiteDatabase db = this.getWritableDatabase();
    db.delete(TABLE_TODO, KEY_ID + " = ?",
            new String[] { String.valueOf(tado_id) });
}

```

## 4.2 tag表的增删改查操作 {#42-tag表的增删改查操作}

### 4.2.1 插入一条数据 {#421-插入一条数据}

```
public long createTag(Tag tag) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TAG_NAME, tag.getTagName());
    values.put(KEY_CREATED_AT, getDateTime());

    // insert row
    long tag_id = db.insert(TABLE_TAG, null, values);

    return tag_id;
}

```

### 4.2.2 查询所有tag {#422-查询所有tag}

```
public List
<
Tag
>
 getAllTags() {
    List
<
Tag
>
 tags = new ArrayList
<
Tag
>
();
    String selectQuery = "SELECT  * FROM " + TABLE_TAG;

    Log.e(LOG, selectQuery);

    SQLiteDatabase db = this.getReadableDatabase();
    Cursor c = db.rawQuery(selectQuery, null);

    // looping through all rows and adding to list
    if (c.moveToFirst()) {
        do {
            Tag t = new Tag();
            t.setId(c.getInt((c.getColumnIndex(KEY_ID))));
            t.setTagName(c.getString(c.getColumnIndex(KEY_TAG_NAME)));

            // adding to tags list
            tags.add(t);
        } while (c.moveToNext());
    }
    return tags;
}

```

### 4.2.3 更新一个tag {#423-更新一个tag}

```
public int updateTag(Tag tag) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TAG_NAME, tag.getTagName());

    // updating row
    return db.update(TABLE_TAG, values, KEY_ID + " = ?",
            new String[] { String.valueOf(tag.getId()) });
}

```

### 4.2.4 删除1个标签和它标记的所有todo数据 {#424-删除1个标签和它标记的所有todo数据}

```
public void deleteTag(Tag tag, boolean should_delete_all_tag_todos) {
    SQLiteDatabase db = this.getWritableDatabase();

    // before deleting tag
    // check if todos under this tag should also be deleted
    if (should_delete_all_tag_todos) {
        // get all todos under this tag
        List
<
Todo
>
 allTagToDos = getAllToDosByTag(tag.getTagName());

        // delete all todos
        for (Todo todo : allTagToDos) {
            // delete todo
            deleteToDo(todo.getId());
        }
    }

    // now delete the tag
    db.delete(TABLE_TAG, KEY_ID + " = ?",
            new String[] { String.valueOf(tag.getId()) });
}

```

## 4.3 todo\_tags 的相关操作 {#43-todotags-的相关操作}

### 4.3.1 给一条todo数据增加tag {#431-给一条todo数据增加tag}

```
public long createTodoTag(long todo_id, long tag_id) {
        SQLiteDatabase db = this.getWritableDatabase();

        ContentValues values = new ContentValues();
        values.put(KEY_TODO_ID, todo_id);
        values.put(KEY_TAG_ID, tag_id);
        values.put(KEY_CREATED_AT, getDateTime());

        long id = db.insert(TABLE_TODO_TAG, null, values);

        return id;
    }

```

### 4.3.2 去掉todo数据的一个tag {#432-去掉todo数据的一个tag}

```
public int updateNoteTag(long id, long tag_id) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TAG_ID, tag_id);

    // updating row
    return db.update(TABLE_TODO, values, KEY_ID + " = ?",
            new String[] { String.valueOf(id) });
}

```

### 4.3.3 修改todo数据的tag {#433-修改todo数据的tag}

```
public int updateNoteTag(long id, long tag_id) {
    SQLiteDatabase db = this.getWritableDatabase();

    ContentValues values = new ContentValues();
    values.put(KEY_TAG_ID, tag_id);

    // updating row
    return db.update(TABLE_TODO, values, KEY_ID + " = ?",
            new String[] { String.valueOf(id) });
}

```

### 4.3.4 关闭数据库 {#434-关闭数据库}

* 操作完数据库一定要记得关闭

```
 public void closeDB() {
        SQLiteDatabase db = this.getReadableDatabase();
        if (db != null 
&
&
 db.isOpen())
            db.close();
    }
```



