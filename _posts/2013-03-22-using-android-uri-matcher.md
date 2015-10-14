---
layout: post
categories:
- android
- java
title: Using Android's UriMatcher
---

When setting up a `ContentProvider`, I didn't really see any good explanation of how to use the `UriMatcher` class. Here is a small example of it in a `ContentProvider`.

```java
public class MyProvider extends ContentProvider {
  private static final int ORDERS = 1;
  private static final int ORDERS_ID = 2;

  private static final int ITEMS = 3;
  private static final int ITEMS_ID = 4;

  private static final UriMatcher mUriMatcher = new UriMatcher(
      UriMatcher.NO_MATCH);

  // The # sign will match numbers
  static {
    mUriMatcher.addURI(Constants.AUTHORITY, "orders", ORDERS);
    mUriMatcher.addURI(Constants.AUTHORITY, "orders/#", ORDERS_ID);

    mUriMatcher.addURI(Constants.AUTHORITY, "items", ITEMS);
    mUriMatcher.addURI(Constants.AUTHORITY, "items/#", ITEMS_ID);
  }

  @Override
  public int delete(Uri uri, String where, String[] whereArgs) {
    int count = 0;

    // Switch on the match, it will return the integer you passed in previously
    switch (mUriMatcher.match(uri)) {
    case ORDERS:
      count = database.getWritableDatabase().delete("orders", where, whereArgs);
      getContext().getContentResolver().notifyChange(uri, null);
    case ITEMS:
      count = database.getWritableDatabase().delete("items", where, whereArgs);
      getContext().getContentResolver().notifyChange(uri, null);
    }

    return count;
  }

  @Override
  public Cursor query(Uri uri, String[] projection, String selection,
      String[] selectionArgs, String sortOrder) {

    SQLiteQueryBuilder qb = new SQLiteQueryBuilder();

    String defaultOrder = DatabaseHelper.DEFAULT_SORT_ORDER; // "id"

    switch (uriMatcher.match(uri)) {
    case ORDERS:
    case ORDERS_ID:
      qb.setTables("orders");
      defaultOrder = DatabaseHelper.DEFAULT_SORT_ORDER_ORDERS; // "date DESC"
      break;
    case ITEMS:
    case ITEMS_ID:
      qb.setTables("items");
      break;
    }

    String orderBy;

    if (TextUtils.isEmpty(sortOrder)) {
      orderBy = defaultOrder;
    } else {
      orderBy = sortOrder;
    }

    Cursor cursor = qb.query(database.getReadableDatabase(), projection,
        selection, selectionArgs, null, null, orderBy);

    cursor.setNotificationUri(getContext().getContentResolver(), uri);

    return cursor;
  }
}
```
