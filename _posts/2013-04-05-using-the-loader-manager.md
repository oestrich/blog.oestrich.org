---
layout: post
categories:
- android
- java
title: Using the LoaderManager
---

The LoaderManager is a nice way of handling Cursors in an Android activity or fragment. It was a little tricky to get started with so I wanted to have an example I could come back to as a reference of how to use it.

```java
public class FirmInfoActivity extends Activity implements
    LoaderManager.LoaderCallbacks<Cursor> {

    private static final int LOADER_ONE = 0;
    private static final int LOADER_TWO = 1;

    private AdapterOne mAdapterOne;
    private AdapterTwo mAdapterTwo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // This is incredibly important to do, I forgot it a few times
        // and had no idea why it wasn't working.
        getLoaderManager().initLoader(LOADER_ONE, null, this);
        getLoaderManager().initLoader(LOADER_TWO, null, this);
    }

    @Override
    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        switch(id) {
        case LOADER_ONE:
            return new CursorLoader(this, Uri.parse(""),
                    null, null, null, null);
        case LOADER_TWO:
            return new CursorLoader(this, Uri.parse(""),
                    null, null, null, null);
        }
    }

    @Override
    public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
        switch(loader.getId()) {
        case LOADER_ONE:
            // Update based on the loaded cursor;
            break;
        case LOADER_TWO:
            // Update based on the loaded cursor;
            break;
        }
    }

    @Override
    public void onLoaderReset(Loader<Cursor> loader) {
        switch(loader.getId()) {
        case LOADER_ONE:
            mAdapterOne.swapCursor(null);
            break;
        case LOADER_TWO:
            mAdapterTwo.swapCursor(null);
            break:
        }
    }
}
```
