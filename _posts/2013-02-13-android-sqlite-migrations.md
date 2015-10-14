---
layout: post
categories:
- android
- sqlite
title: Android SQLite Migrations
---

<img class="float-img" alt="Migrating birds" src="/images/bird_migrations.jpg" />

I started up on android again and started using the SQLite database for the first time. Coming from rails I really wanted to have some kind of migrations for the database. I googled around and didn't really see anything available so I made up a simple class that solves my problem.

Note that this hasn't been used outside of my test app. I also have only created one migration with it so far, but I like how it went. An abstract class might also not be the best way to go, but I haven't used it since college and wanted to try it out again.

##### src/org/oestrich/myapp/migrations/Migration.java
```java
package org.oestrich.myapp.migrations;

import android.database.sqlite.SQLiteDatabase;

public abstract class Migration {
        private SQLiteDatabase mDatabase;

        public Migration(SQLiteDatabase database) {
                mDatabase = database;
        }

        public abstract void up();

        public abstract void down();

        public SQLiteDatabase getDatabase() {
                return mDatabase;
        }
}
```

##### src/org/oestrich/myapp/migrations/InitialDatabase.java
```java
package org.oestrich.myapp.migrations;

import android.database.sqlite.SQLiteDatabase;

public class InitialDatabase extends Migration {

        public InitialDatabase(SQLiteDatabase database) {
                super(database);
        }

        @Override
        public void up() {
                getDatabase().execSQL(
                        "CREATE TABLE my_model (_id INTEGER PRIMARY KEY AUTOINCREMENT);");
        }

        @Override
        public void down() {
                getDatabase().execSQL("DROP TABLE my_model;");
        }

}
```

##### src/org/oestrich/myapp/database_helper.rb
```java
package org.oestrich.myapp;

public class DatabaseHelper extends SQLiteOpenHelper {
        // ...

        @Override
        public void onCreate(SQLiteDatabase db) {
                try {
                        db.beginTransaction();

                        new InitialDatabase(db).up();

                        // Add more migrations here

                        db.setTransactionSuccessful();
                } finally {
                        db.endTransaction();
                }
        }

        // ...
}
```

[Image Source](http://www.flickr.com/photos/rwjensen/3482243965/)
