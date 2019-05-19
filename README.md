# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
NoteList中显示条目增加时间显示
在NotePad原应用中，笔记列表只显示了笔记的标题。对它做时间扩展，把时间放在标题下方。 
先找到列表中item的布局：noteslist_item.xml。 
在这个布局文件中有一个TextView是笔记列表的标题（item）了。
```
<TextView xmlns:android="http://schemas.android.com/apk/res/android".
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
/>
```
在标题下方加时间显示，就要在标题的TextView加一个时间的TextView。但是由于原应用列表item只需要一个标题，所以不需要用上别的布局；要多加一个时间TextView，就要把标题TextView和时间TextView放入垂直的线性布局。 
由于要美化UI，所以将TextView原字体颜色改为黑色。新加的时间TextView字体大小小于标题TextView。
```
<?xml version="1.0" encoding="utf-8"?>
<!--添加一个垂直的线性布局-->
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--原标题TextView-->
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"
        android:singleLine="true"
        />
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
</LinearLayout>
```
先查看程序如何定义数据库结构的，NotePadProvider.java中:
```
@Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
            + ");");
    }
```
可以看出，NotePad数据库已经存在时间信息。 
再到NoteList.java文件中查看，是如何将数据装填到列表中。 
可以发现，当前Activity所用到的数据被定义在PROJECTION中：
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
    };
```
通过Cursor从数据库中读取出：
```
Cursor cursor = managedQuery(
            getIntent().getData(),            // Use the default content URI for the provider.
            PROJECTION,                       // Return the note ID and title for each note.
            null,                             // No where clause, return all records.
            null,                             // No where clause, therefore no where column values.
            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
```
通过SimpleCursorAdapter装填：
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
SimpleCursorAdapter adapter
    = new SimpleCursorAdapter(
            this, // The Context for the ListView
            R.layout.noteslist_item, // Points to the XML for a list item
            cursor,   // The cursor to get items from
            dataColumns,
            viewIDs
    );
// Sets the ListView's adapter to be the cursor adapter that was just created.
setListAdapter(adapter);
```
要将时间显示，首先要在PROJECTION中定义显示的时间，原应用有两种时间，我选择修改时间作为显示的时间。
```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, 
    };
```
Cursor不变，在dataColumns，viewIDs中补充时间部分：
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
```

SimpleCursorAdapter不变（在改笔记列表颜色时需要做出改变，下文会涉及）。做完这些，标题下确实会多显示一行数据，但是，这并不是我们平常所见到的时间格式，而是时间戳，需要对这些数据进行转化，使人能看的懂。 
我选则的方法时把时间戳改为以时间格式存入，改动地方分别为NotePadProvider中的insert方法和NoteEditor中的updateNote方法。前者为创建笔记时产生的时间，后者为修改笔记时产生的时间。下面代码中的dateTime即为转化后的时间格式，将其用ContentValues的put方法存入数据库。
```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);
```
运行结果：


































