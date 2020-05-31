# 期中作业Notepad
## 姓名：林伟彬   学号： 116052017126 

## 界面效果：

### 点击编写按钮添加进行添加笔记
<img src="https://github.com/linwb-0924/Notepad/blob/master/image/1.jpg" width="375" />

### 在第一行输入标题，第二行开始输入正文，点击保存按钮保存
<img src="https://github.com/linwb-0924/Notepad/blob/master/image/2.jpg" width="375" />

### 添加后界面效果
<img src="https://github.com/linwb-0924/Notepad/blob/master/image/3.jpg" width="375" />

### 进行搜索，当有搜索结果
<img src="https://github.com/linwb-0924/Notepad/blob/master/image/4.jpg" width="375" />

### 进行搜索，当无搜索结果
<img src="https://github.com/linwb-0924/Notepad/blob/master/image/5.jpg" width="375" />
## 实验核心代码：

## 时间戳显示

### 首先修改NoteList文件中PROJECTION的内容，添加modif字段，这样在后面的搜索中才能从SQLite中读取修改时间的字段。
        private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        };
### 修改适配器内容，增加dataColumns中装配到ListView的内容，所以要同时增加一个文本框来存放时间。
           final String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
                int[] viewIDs = { android.R.id.text1, R.id.text2};
              
              
### 修改layout文件夹中notelist_item的内容，增加一个textview组件，因为有两个组件，所以我们也要相应的为他们添加一个布局。
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            android:paddingLeft="6dip"
            android:paddingRight="6dip"
            android:paddingBottom="3dip">

            <TextView
                android:id="@android:id/text1"
                android:layout_width="match_parent"
                android:layout_height="?android:attr/listPreferredItemHeight"
                android:textAppearance="?android:attr/textAppearanceLarge"
                android:gravity="center_vertical"
                android:paddingLeft="5dip"
                android:singleLine="true"
            />
            <TextView
                android:id="@+id/text2"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:textAppearance="?android:attr/textAppearanceLarge"
                android:gravity="center_vertical"
                android:singleLine="true"
                />
        </LinearLayout>


### 修改NoteEditor中updateNote()方法中的时间类型.首先获得long类型的now时间，在运用SimpleDateFormat类设置时间类型，这里我用的是“年/月/天 时/分”，可以根据不同的显示精度进行调整。将now转化成Date类型时间，其实做到这一步，将d放进数据库即可，不过显示得太精密了，太长了，所以我将他修改成我想要得sf类型。String format = sf.format(d)
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

## 搜索框的添加

### 新增布局文件searchview.xml，设置一个搜索的布局文件，由searchview，textview，listview组件组成，为搜索后所表现的样子
                <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical">
                    <SearchView
                        android:id="@+id/search2"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:queryHint="请输入搜索内容"
                        >
                    </SearchView>

                    <ListView
                        android:paddingLeft="10dp"
                        android:paddingRight="10dp"
                        android:id="@android:id/list"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_weight="1"
                        android:drawSelectorOnTop="false" />
                    <TextView
                        android:paddingLeft="10dp"
                        android:paddingRight="10dp"
                        android:id="@android:id/empty"
                        android:gravity="center"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_weight="1"
                        android:text="无搜索结果" />
                </LinearLayout>

        ### 修改NotesList一开始加载的布局文件
                setContentView(R.layout.searchview);
        ### 首先找到search组件，创造searchView得响应时间，根据传入得s进行搜索，然后写下selection语句NotePad.Notes.COLUMN_NAME_TITLE + " GLOB ‘" + s + "’"，运用getContentResolver().query对SQLite里面得数据进行搜索，PROJECTION则是在设置时间戳得时候已经书写好，如果s为空，则是把所有信息显示出来。最后更新adapter
         private void SearchView(final SimpleCursorAdapter adapter) {
                SearchView searchView = findViewById(R.id.search2);
                searchView.setSubmitButtonEnabled(true);
                searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
                    @Override
                    public boolean onQueryTextSubmit(String query) {
                        return false;
                    }

                    @Override
                    public boolean onQueryTextChange(String s) {
                        Cursor newCursor;

                        if (!s.equals("")) {
                            String selection = NotePad.Notes.COLUMN_NAME_TITLE + " GLOB '*" + s + "*'";
                            newCursor = getContentResolver().query(
                                    getIntent().getData(),
                                    PROJECTION,
                                    selection,
                                    null,
                                    NotePad.Notes.DEFAULT_SORT_ORDER
                            );
                        } else {
                            newCursor = getContentResolver().query(
                                    getIntent().getData(),
                                    PROJECTION,
                                    null,
                                    null,
                                    NotePad.Notes.DEFAULT_SORT_ORDER
                            );
                        }
                        adapter.swapCursor(newCursor); // 视图将同步更新！
                        return true;
                    }
                });
            }

