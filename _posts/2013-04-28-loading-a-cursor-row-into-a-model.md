---
layout: post
categories:
- android
title: Loading a Cursor Row into a Model
---

When using a Cursor in Android I prefer to pull out a row into a Java object. The Java objects will mostly be getters and setters, but will also include convenience methods as well. In order to make going from a cursor row into a model instance I create a static method on each model class called `fromCursor`.

This method will pull out columns into the appropriate member variable. It will only pull out a column if it is in the selected columns.

    public class MyModel {
        public static MyModel fromCursor(Cursor cursor) {
            MyModel myModel = new MyModel();

            if (cursor.getColumnIndex("_id") != -1) {
                myModel.setId(cursor.getInt(cursor.getColumnIndex("_id")));
            }

            if (cursor.getColumnIndex("description") != -1) {
                myModel.setDescription(cursor.getString(cursor.getColumnIndex("description")));
            }

            /* ... */

            return myModel;
        }
    }
