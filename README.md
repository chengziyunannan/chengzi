# 期中NotePad
##1.时间戳
首先先在notelist——item中添加一个Textview来显示时间戳，在NotePadProvider与NoteEditor中修改时间戳的显示格式，dataColumns数组和viewIDs数组中添加对应的标记名和item中对应显示时间戳的组件ID，这样即可看到以毫秒显示出来的时间
如下图
<img src="https://user-images.githubusercontent.com/113671496/202910799-50010ce9-baf0-4a68-84bd-20d4565c97a6.png" width="250"/>

代码如下

notelist-item.xml
 ```java
 <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="27dp"
        android:layout_margin="0dp"
        android:gravity="center_vertical"
        android:paddingLeft="240dip"
        android:singleLine="true"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textSize="18dp" />
```
        
notepadprovider.java
```java
Long now = Long.valueOf(System.currentTimeMillis());
//修改 需要将毫秒数转换为时间的形式yy.MM.dd HH:mm:ss

Date date = new Date(now);

SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");

String dateFormat = simpleDateFormat.format(date);
//转换为yy.MM.dd HH:mm:ss形式的时间

if(values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
}

if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
} 
```

NoteEditor.java
  ```java
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
NoteList.java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
    };


String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE }//加入修改时间;
int[] viewIDs = { android.R.id.text1, R.id.text2 }//加入修改时间;
  ```

##2.搜索
先在list_options_menu.xml中添加一个item，然后再新建一个搜索界面的布局文件note_search.xml，在NoteList中添加搜索功能的跳转事件，新建一个NoteSearch来实现跳转的功能，最后在清单文件AndroidManifest.xml中注册notesearch
如图
<img src="https://user-images.githubusercontent.com/113671496/202910805-cf6d8e24-ea0a-4088-989d-378b566c2a9e.png" width="250"/>

代码如下

list_options_menu.xml
  ```java
<item
      android:id="@+id/menu_search"
      android:icon="@android:drawable/ic_menu_search"
      android:title="@string/menu_search"
      android:showAsAction="always" />
 ```
note_search.xml
  ```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <SearchView
        android:id="@+id/search"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false" />

    <ListView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```
NoteList.java
```java

        case R.id.menu_search:  
        //查找功能  
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));  
          Intent intent = new Intent(this, NoteSearch.class);  
          this.startActivity(intent);  
          return true;  
```

NoteSearch.java
```java
package com.example.android.notepad;
import android.app.Activity;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;
import android.widget.Toast;

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
{
    ListView listView;
    SQLiteDatabase sqLiteDatabase;
    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };

    public boolean onQueryTextSubmit(String query) {
        Toast.makeText(this, "您选择的是："+query, Toast.LENGTH_SHORT).show();
        return false;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = (SearchView) findViewById(R.id.search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = (ListView) findViewById(R.id.list);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //设置该SearchView显示搜索按钮
        searchView.setSubmitButtonEnabled(true);

        //设置该SearchView内默认显示的提示文本
        searchView.setQueryHint("查找");
        searchView.setOnQueryTextListener(this);

    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
}
```

AndroidManifest.xml
```java
<activity android:name=".NoteSearch" android:label="@string/menu_search" />
```


3.改变主页的背景颜色
在数据库中增加一个字段“color”用于存放每条笔记的背景颜色，在NotePadProvider中修改创建数据库的语句并实例化和设置静态对象的块，增加创建新笔记时需要执行的语句，给每条笔记设置默认背景颜色。NoteList和NoteSearch中添加color属性使得笔记列表显示时从数据库中读取color的代码使用bindView将颜色填充到ListView。新建一个MyCursorAdapter继承SimpleCursorAdapter
如图
<img src="https://user-images.githubusercontent.com/113671496/202911182-e2604fe7-21aa-4a86-940c-1d85e106231a.jpg" width="250"/>


代码如下



NotePad.java
```java
public static final String COLUMN_NAME_BACK_COLOR = "color";
public static final int DEFAULT_COLOR = 0;
public static final int YELLOW_COLOR = 1;
public static final int BLUE_COLOR = 2;
public static final int GREEN_COLOR = 3;
public static final int RED_COLOR = 4; 
```
NotePadProvider.java
```java
public void onCreate(SQLiteDatabase db) {
    db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //数据库增加color属性
            + ");");
}



static{
    ·····
            // add Maps "color" to "color"
    sNotesProjectionMap.put(
        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        NotePad.Notes.COLUMN_NAME_BACK_COLOR);
    ·····
}



// 笔记背景默认为白色
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
}
```

NoteList.java
NoteSearch.java
```java
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        //扩展 显示时间戳
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
        //扩展 显示笔记背景颜色
        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
};
```

MyCursorAdapter.java
```java
public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                        String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的先前存入的笔记背景颜色的编码，再设置笔记的背景颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
NoteList.java
NoteSearch.java
```java
MyCursorAdapter adapter
    = new MyCursorAdapter(
              this,
              R.layout.noteslist_item,
              cursor, 
              dataColumns,
              viewIDs
      );
```

color_select.xml
```java
<selector xmlns:android="http://schemas.android.com/apk/res/android">
<item android:drawable="@color/color1"
    android:state_pressed="true"/>
</selector>
```

notelist_item.xml
```java
android:background="@drawable/color_select"
```

4.改变编写界面的背景颜色

在NoteEditor中增加color属性使显示时能够查询到color属性，在editor_options_menu.xml中添加一个更改背景的按钮往NoteEditor.java中添加增加上一步新增的选项的执行语句，新建一个布局来选择背景颜色，新建一个类来实现颜色的选择，
如图
<img src="https://user-images.githubusercontent.com/113671496/202910844-41f0b23f-2817-49f1-aa41-6b603ff111cd.jpg" width="250"/>

代码如下

NoteEditor.java
```java
private static final String[] PROJECTION =
    new String[] {
        NotePad.Notes._ID,
        NotePad.Notes.COLUMN_NAME_TITLE,
        NotePad.Notes.COLUMN_NAME_NOTE,
            //增加 背景颜色
        NotePad.Notes.COLUMN_NAME_BACK_COLOR
};



//读取颜色数据
if(mCursor!=null){
    mCursor.moveToFirst();
    int x = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR);
    int y = mCursor.getInt(x);
    Log.i("NoteEditor", "color"+y);
    switch (y){
        case NotePad.Notes.DEFAULT_COLOR:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
        case NotePad.Notes.YELLOW_COLOR:
            mText.setBackgroundColor(Color.rgb(247, 216, 133));
            break;
        case NotePad.Notes.BLUE_COLOR:
            mText.setBackgroundColor(Color.rgb(165, 202, 237));
            break;
        case NotePad.Notes.GREEN_COLOR:
            mText.setBackgroundColor(Color.rgb(161, 214, 174));
            break;
        case NotePad.Notes.RED_COLOR:
            mText.setBackgroundColor(Color.rgb(244, 149, 133));
            break;
        default:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
    }
}

```

editor_options_menu.xml
```java
<item android:id="@+id/menu_color"
android:title="color"
android:icon="@drawable/ic_menu_edit"
android:showAsAction="always"/>
```


NoteEditor.java
```java
case R.id.menu_color://跳转改变颜色的activity
    Intent intent = new Intent(null,mUri);
    intent.setClass(NoteEditor.this,NoteColor.class);
    NoteEditor.this.startActivity(intent);
    break;
```

note_color.xml
```java
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#ffffff"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#fdef90"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#6dc1ed"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#94e49f"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="#f19696"
        android:onClick="red"/>
```

NoteColor.java
```java
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, 
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,       
                PROJECTION,  
                null,      
                null,    
                null        
        );
    }
    @Override
    protected void onResume(){
        if(mCursor.moveToFirst()){
            color = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
            Log.i("NoteColor", "before"+color);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //将修改的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        Log.i("NoteColor", "cun"+color);
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
        int x = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR);
        int y = mCursor.getInt(x);
        Log.i("NoteColor", "du"+y);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }
}
```
AndroidManifest.xml
```java
<activity android:name="NoteColor"
    android:theme="@android:style/Theme.Holo.Light.Dialog"
    android:label="Select Color"/>
    ```
