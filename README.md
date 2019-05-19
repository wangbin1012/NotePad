

**基本功能核心代码:**
1.增加时间戳显示
(1)notelist_item.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<!-- Copyright (C) 2010 The Android Open Source Project

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<!--添加一个垂直的线性布局-->
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--原标题TextView-->
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/title"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"/>
    
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/timeStamp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"/>
</LinearLayout>
```
NotePadProvider.java
```java
// Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);

// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
}
```

**截图：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519141442811.png)


**根据标题查找便签
(1)NoteSearch.java**
```java
package com.example.notepad;

import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

/**
 * Created by c7 on 2019/5/15.
 */
public class NoteSearch  extends ListActivity  implements SearchView.OnQueryTextListener{
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
    };
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent();
        if (intent.getData() == null) {            intent.setData(NotePad.Notes.CONTENT_URI);
        }        getListView().setOnCreateContextMenuListener(this);
        //为查询文本框注册监听器
        SearchView mSearchView = (SearchView)findViewById(R.id.search_view);//注册监听器
        mSearchView.setIconifiedByDefault(false); //显示搜索的天幕，默认只有一个放大镜图标
        mSearchView.setSubmitButtonEnabled(true); //显示搜索按钮        mSearchView.setBackgroundColor(getResources().getColor(R.color.colorPrimary)); //设置背景颜色
        mSearchView.setOnQueryTextListener(this);
    }
    
    @Override
    public boolean onQueryTextSubmit(String s) {
        return false;
    }
    
    @Override
    public boolean onQueryTextChange(String s) { //Test改变的时候执行的内容
        //Text发生改变时执行的内容
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+s+"%" };//查询条件参数，配合selection参数使用,%通配多个字符
        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。

        //managedQuery(Uri, String[], String, String[], String)等同于Context.getContentResolver().query()
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.用于ContentProvider查询的URI，从这个URI获取数据
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date.用于标识uri中有哪些columns需要包含在返回的Cursor对象中

                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择

                selectionArgs,                    // 查询条件参数，配合selection参数使用

                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.查询结果的排序方式，按照某个columns来排序，例：String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE

        );
        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,                   //context:上下文
                R.layout.noteslist_item,         //layout:布局文件，至少有int[]的所有视图
                cursor,                          //cursor：游标
                dataColumns,                     //from：绑定到视图的数据
                viewIDs                          //to:用来展示from数组中数据的视图
                //flags：用来确定适配器行为的标志，Android3.0之后淘汰
        );
        setListAdapter(adapter);
        return true;
    }
}
```

**note_search.xml**
```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:iconifiedByDefault="false"
        android:queryHint="请输入搜索标题"></SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"></ListView>
</LinearLayout>
```

**搜索功能截图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519142520503.png)
**附加功能：背景更改
更改默认背景为白色
AndroidManifest.xml**
```java
<activity
    android:name=".NotesList"
    android:label="@string/title_notes_list"
    android:theme="@android:style/Theme.Holo.Light">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <action android:name="android.intent.action.EDIT" />
        <action android:name="android.intent.action.PICK" />

        <category android:name="android.intent.category.DEFAULT" />

        <data android:mimeType="vnd.android.cursor.dir/vnd.google.note" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.GET_CONTENT" />

        <category android:name="android.intent.category.DEFAULT" />

        <data android:mimeType="vnd.android.cursor.item/vnd.google.note" />
    </intent-filter>
</activity>


//蓝色背景
case R.id.menu_theme_daytime:
    linearLayout = (LinearLayout) findViewById(R.id.layout);
    linearLayout.setBackgroundColor(Color.Blue);
    title = (TextView) findViewById(R.id.title);
    title.setTextColor(Color.BLACK);
    timeStamp = (TextView) findViewById(R.id.timeStamp);
    timeStamp.setTextColor(Color.BLACK);
    return true;

```
**修改背景截图：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190519142943364.png)
