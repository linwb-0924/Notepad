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
## 实验代码：
### NoteEditor.java（核心）
        @Override
        protected void onDraw(Canvas canvas) {

            int count = getLineCount();
            Rect r = mRect;
            Paint paint = mPaint;

            for (int i = 0; i < count; i++) {


                int baseline = getLineBounds(i, r);

                canvas.drawLine(r.left, baseline + 1, r.right, baseline + 1, paint);
            }

            super.onDraw(canvas);
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        final Intent intent = getIntent();

        final String action = intent.getAction();

        if (Intent.ACTION_EDIT.equals(action)) {

            mState = STATE_EDIT;
            mUri = intent.getData();

        } else if (Intent.ACTION_INSERT.equals(action)
                || Intent.ACTION_PASTE.equals(action)) {

            mState = STATE_INSERT;
            mUri = getContentResolver().insert(intent.getData(), null);

            if (mUri == null) {
                Log.e(TAG, "Failed to insert new note into " + getIntent().getData());
                finish();
                return;
            }

            setResult(RESULT_OK, (new Intent()).setAction(mUri.toString()));

        } else {
            Log.e(TAG, "Unknown action, exiting");
            finish();
            return;
        }

        mCursor = managedQuery(
            mUri,         // The URI that gets multiple notes from the provider.
            PROJECTION,   // A projection that returns the note ID and note content for each note.
            null,         // No "where" clause selection criteria.
            null,         // No "where" clause selection values.
            null          // Use the default sort order (modification date, descending)
        );

        if (Intent.ACTION_PASTE.equals(action)) {
            // Does the paste
            performPaste();
            // Switches the state to EDIT so the title can be modified.
            mState = STATE_EDIT;
        }

        setContentView(R.layout.note_editor);

        mText = (EditText) findViewById(R.id.note);
        if (savedInstanceState != null) {
            mOriginalContent = savedInstanceState.getString(ORIGINAL_CONTENT);
        }
    }
    @Override
    protected void onResume() {
        super.onResume();
        if (mCursor != null) {
            // Requery in case something changed while paused (such as the title)
            mCursor.requery();
            mCursor.moveToFirst();

            // Modifies the window title for the Activity according to the current Activity state.
            if (mState == STATE_EDIT) {
                // Set the title of the Activity to include the note title
                int colTitleIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE);
                String title = mCursor.getString(colTitleIndex);
                Resources res = getResources();
                String text = String.format(res.getString(R.string.title_edit), title);
                setTitle(text);
            // Sets the title to "create" for inserts
            } else if (mState == STATE_INSERT) {
                setTitle(getText(R.string.title_create));
            }

            int colNoteIndex = mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE);
            String note = mCursor.getString(colNoteIndex);
            mText.setTextKeepState(note);

            // Stores the original note text, to allow the user to revert changes.
            if (mOriginalContent == null) {
                mOriginalContent = note;
            }

        } else {
            setTitle(getText(R.string.error_title));
            mText.setText(getText(R.string.error_message));
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        // Save away the original text, so we still have it if the activity
        // needs to be killed while paused.
        outState.putString(ORIGINAL_CONTENT, mOriginalContent);
    }
### NotePad.java（核心）
    public final class NotePad {
    public static final String AUTHORITY = "com.google.provider.NotePad";
    private NotePad() {
    }
    public static final class Notes implements BaseColumns {

        private Notes() {}

        public static final String TABLE_NAME = "notes";

        private static final String SCHEME = "content://";

        private static final String PATH_NOTES = "/notes";

        private static final String PATH_NOTE_ID = "/notes/";

        public static final int NOTE_ID_PATH_POSITION = 1;

        private static final String PATH_LIVE_FOLDER = "/live_folders/notes";

        public static final Uri CONTENT_URI =  Uri.parse(SCHEME + AUTHORITY + PATH_NOTES);

        public static final Uri CONTENT_ID_URI_BASE
            = Uri.parse(SCHEME + AUTHORITY + PATH_NOTE_ID);

        public static final Uri CONTENT_ID_URI_PATTERN
            = Uri.parse(SCHEME + AUTHORITY + PATH_NOTE_ID + "/#");

        public static final Uri LIVE_FOLDER_URI
            = Uri.parse(SCHEME + AUTHORITY + PATH_LIVE_FOLDER);

        public static final String CONTENT_TYPE = "vnd.android.cursor.dir/vnd.google.note";

        public static final String CONTENT_ITEM_TYPE = "vnd.android.cursor.item/vnd.google.note";

        public static final String DEFAULT_SORT_ORDER = "modified DESC";

        public static final String COLUMN_NAME_TITLE = "title";

        public static final String COLUMN_NAME_NOTE = "note";

        public static final String COLUMN_NAME_CREATE_DATE = "created";

        public static final String COLUMN_NAME_MODIFICATION_DATE = "modified";
    }
  ### NotePadProvider.java（核心）  
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
       @Override
       public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

           // Logs that the database is being upgraded
           Log.w(TAG, "Upgrading database from version " + oldVersion + " to "
                   + newVersion + ", which will destroy all old data");

           // Kills the table and existing data
           db.execSQL("DROP TABLE IF EXISTS notes");

           // Recreates the database with a new version
           onCreate(db);
       }
    }
     @Override
      public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,
           String sortOrder) {

       // Constructs a new query builder and sets its table name
       SQLiteQueryBuilder qb = new SQLiteQueryBuilder();
       qb.setTables(NotePad.Notes.TABLE_NAME);

       switch (sUriMatcher.match(uri)) {
           // If the incoming URI is for notes, chooses the Notes projection
           case NOTES:
               qb.setProjectionMap(sNotesProjectionMap);
               break;

           case NOTE_ID:
               qb.setProjectionMap(sNotesProjectionMap);
               qb.appendWhere(
                   NotePad.Notes._ID +    // the name of the ID column
                   "=" +
                   // the position of the note ID itself in the incoming URI
                   uri.getPathSegments().get(NotePad.Notes.NOTE_ID_PATH_POSITION));
               break;

           case LIVE_FOLDER_NOTES:
               // If the incoming URI is from a live folder, chooses the live folder projection.
               qb.setProjectionMap(sLiveFolderProjectionMap);
               break;

           default:
               // If the URI doesn't match any of the known patterns, throw an exception.
               throw new IllegalArgumentException("Unknown URI " + uri);
       }


       String orderBy;
       // If no sort order is specified, uses the default
       if (TextUtils.isEmpty(sortOrder)) {
           orderBy = NotePad.Notes.DEFAULT_SORT_ORDER;
       } else {
           // otherwise, uses the incoming sort order
           orderBy = sortOrder;
       }

       SQLiteDatabase db = mOpenHelper.getReadableDatabase();

       Cursor c = qb.query(
           db,            // The database to query
           projection,    // The columns to return from the query
           selection,     // The columns for the where clause
           selectionArgs, // The values for the where clause
           null,          // don't group the rows
           null,          // don't filter by row groups
           orderBy        // The sort order
       );

       c.setNotificationUri(getContext().getContentResolver(), uri);
       return c;
   }

    @Override
    public String getType(Uri uri) {

       /**
        * Chooses the MIME type based on the incoming URI pattern
        */
       switch (sUriMatcher.match(uri)) {

           // If the pattern is for notes or live folders, returns the general content type.
           case NOTES:
           case LIVE_FOLDER_NOTES:
               return NotePad.Notes.CONTENT_TYPE;

           // If the pattern is for note IDs, returns the note ID content type.
           case NOTE_ID:
               return NotePad.Notes.CONTENT_ITEM_TYPE;

           // If the URI pattern doesn't match any permitted patterns, throws an exception.
           default:
               throw new IllegalArgumentException("Unknown URI " + uri);
       }
    }
    @Override
    public void writeDataToPipe(ParcelFileDescriptor output, Uri uri, String mimeType,
            Bundle opts, Cursor c) {
        // We currently only support conversion-to-text from a single note entry,
        // so no need for cursor data type checking here.
        FileOutputStream fout = new FileOutputStream(output.getFileDescriptor());
        PrintWriter pw = null;
        try {
            pw = new PrintWriter(new OutputStreamWriter(fout, "UTF-8"));
            pw.println(c.getString(READ_NOTE_TITLE_INDEX));
            pw.println("");
            pw.println(c.getString(READ_NOTE_NOTE_INDEX));
        } catch (UnsupportedEncodingException e) {
            Log.w(TAG, "Ooops", e);
        } finally {
            c.close();
            if (pw != null) {
                pw.flush();
            }
            try {
                fout.close();
            } catch (IOException e) {
            }
        }
     }
 ### NoteList（核心）
        private void SearchView(final SimpleCursorAdapter adapter) {
        SearchView searchView = findViewById(R.id.sv);
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
   ### NoteLiveFolder（核心） 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        final Intent intent = getIntent();
        final String action = intent.getAction();

        if (LiveFolders.ACTION_CREATE_LIVE_FOLDER.equals(action)) {

            // Creates a new Intent.
            final Intent liveFolderIntent = new Intent();

            liveFolderIntent.setData(NotePad.Notes.LIVE_FOLDER_URI);

            String foldername = getString(R.string.live_folder_name);
            liveFolderIntent.putExtra(LiveFolders.EXTRA_LIVE_FOLDER_NAME, foldername);

            ShortcutIconResource foldericon =
                Intent.ShortcutIconResource.fromContext(this, R.drawable.live_folder_notes);
            liveFolderIntent.putExtra(LiveFolders.EXTRA_LIVE_FOLDER_ICON, foldericon);

            liveFolderIntent.putExtra(
                    LiveFolders.EXTRA_LIVE_FOLDER_DISPLAY_MODE,
                    LiveFolders.DISPLAY_MODE_LIST);

            Intent returnIntent
                    = new Intent(Intent.ACTION_EDIT, NotePad.Notes.CONTENT_ID_URI_PATTERN);
            liveFolderIntent.putExtra(LiveFolders.EXTRA_LIVE_FOLDER_BASE_INTENT, returnIntent);
            setResult(RESULT_OK, liveFolderIntent);

        } else {

            setResult(RESULT_CANCELED);
        }

        finish();
    }
 ### TitleEditor（核心） 
    @Override
    protected void onResume() {
        super.onResume();

        // Verifies that the query made in onCreate() actually worked. If it worked, then the
        // Cursor object is not null. If it is *empty*, then mCursor.getCount() == 0.
        if (mCursor != null) {

            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();

            // Displays the current title text in the EditText object.
            mText.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();

        // Verifies that the query made in onCreate() actually worked. If it worked, then the
        // Cursor object is not null. If it is *empty*, then mCursor.getCount() == 0.

        if (mCursor != null) {

            // Creates a values map for updating the provider.
            ContentValues values = new ContentValues();

            // In the values map, sets the title to the current contents of the edit box.
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, mText.getText().toString());

            getContentResolver().update(
                mUri,    // The URI for the note to update.
                values,  // The values map containing the columns to update and the values to use.
                null,    // No selection criteria is used, so no "where" columns are needed.
                null     // No "where" columns are used, so no "where" values are needed.
            );

        }
    }
 ### listview.xml
    <SearchView
        android:id="@+id/sv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索"
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
        android:text="无" />
 ### note_editor.xml
     <view xmlns:android="http://schemas.android.com/apk/res/android"
        class="com.example.android.notepad.NoteEditor$LinedEditText"
        android:id="@+id/note"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/transparent"
        android:padding="5dp"
        android:scrollbars="vertical"
        android:fadingEdge="vertical"
        android:gravity="top"
        android:textSize="22sp"
        android:capitalize="sentences"
    />
 ### noteslist_item.xml
     <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dp"
        android:text="1323"
        android:singleLine="true"
        android:layout_margin="0dp"
    />
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textSize="12dp"
        android:gravity="center_vertical"
        android:paddingLeft="5dp"
        android:singleLine="true"
        android:text="132123"
        android:layout_margin="0dp"
        />
 ### title_editor.xml
    <EditText android:id="@+id/title" 
        android:maxLines="1" 
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
      	android:ems="25"
        android:layout_height="wrap_content" 
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
   		
    <Button android:id="@+id/ok"
        android:layout_width="wrap_content" 
        android:layout_height="wrap_content" 
        android:layout_gravity="right"
        android:text="@string/button_ok"
        android:onClick="onClickOk" />
 
