---
layout: post
title: Lesson 9 - Content Provider
cover: lesson-9-banner.png
author: Felipe Joglar
permalink: /lessons/09
summary: "This lesson build off the last by introducing Content Providers. Content Providers are a core Android Framework component that help us provide and manage access to our app's data. In this lesson, we'll learn how to leverage a content provider to get data from other apps on the phone."
---

<img src="{{site.baseurl}}/assets/banner/{{page.cover}}" alt="{{pagle.title}}"/>

{{page.summary}}

## Index

- [Content Provider](#content-provider)
- [Content Provider Permissions](#content-provider-permissions)
- [Access a Content Provider](#access-a-content-provider)
- [Content URIs](#content-uris)


## Content Provider

A `ContentProvider` is a component that interacts with a repository. The app doesn't need to know where or how the data is stored, formatted, or accessed.

Content providers can help an application manage access to data stored by itself, stored by other apps, and provide a way to share data with other apps, they encapsulate the data. Content providers are the standard interface that connects data in one process with code running in another process. Implementing a content provider has many advantages, most importantly we can configure a content provider to allow other applications to securely access and modify our app data.

<img src="{{site.baseurl}}/assets/images/content-provider-overview.png" alt="COntent Provider overview" width="450" align="right" hspace="10">

We can use content providers if we plan to share data. If we don’t plan to share data, we may still use them because they provide a nice abstraction. This abstraction allows us to make modifications to our application data storage implementation without affecting other existing applications that rely on access to our data. In this scenario only our content provider is affected and not the applications that access it.

The Android framework includes content providers that manage data such as audio, video, images, and personal contact information. We can see some of them listed in the reference documentation for the [`android.provider`](https://developer.android.com/reference/android/provider/package-summary.html) package. With some restrictions, these providers are accessible from any Android application.

Content providers come with some implicit *advantages*. They offer granular control over the permissions for accessing data. We can choose to restrict access to a content provider from solely within our application, grant blanket permission to access data from other applications, or configure different permissions for *reading and writing* data. Also we can use a content provider to abstract away the details for accessing different data sources in our application.


## Content Provider Permissions

A provider's application can specify *permissions* that other applications must have in order to access the provider's data. These permissions ensure that the user knows what data an application will try to access. Based on the provider's requirements, other applications request the permissions they need in order to access the provider.

Android system comes with a set of content providers that allow the developers use the phones user data, like the calendar, dictionary and contacts.

To get the permissions needed to access a provider, an application requests them with a `<uses-permission>` element in its manifest file.

``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    ...>

    <!-- Add the correct permission to access the content provider -->
    <uses-permission android:name="android.permission.READ_USER_DICTIONARY" />
    <uses-permission android:name="android.permission.WRITE_USER_DICTIONARY" />
    
    <application>
        ...
    </application>

</manifest>
```

As we can note, the User Dictionary Provider requires the `android.permission.READ_USER_DICTIONARY` permission to retrieve data from it. The provider has the separate `android.permission.WRITE_USER_DICTIONARY` permission for inserting, updating, or deleting data. The for the contacts and other content providers.


## Access a Content Provider

When Content Providers are used to expose data, Content Resolvers are the corresponding class used to query and perform transactions on those Content Providers. Content Providers provide an abstraction from the underlying data while Content Resolvers provide an abstraction from the Content Provider being queried or transacted.

The Content Resolver includes query and transaction methods corresponding to those defined within our Content Providers. The `Content Resolver` does not need to know the implementation of the Content Providers it is interacting with — each query and transaction method simply accepts a URI that specifies the Content Provider to interact with.

To get a list of the words from the User Dictionary Provider (one of the system providers), we call `ContentResolver.query()`. The `query()` method calls the `ContentProvider.query()` method defined by the User Dictionary Provider. The following lines of code show a `ContentResolver.query()` call:

``` java
// A "projection" defines the columns that will be returned for each row
String[] mProjection =
{
    UserDictionary.Words._ID,    // Contract class constant for the _ID column name
    UserDictionary.Words.WORD,   // Contract class constant for the word column name
    UserDictionary.Words.LOCALE  // Contract class constant for the locale column name
};

// Defines a string to contain the selection clause
String mSelectionClause = null;

// Initializes an array to contain selection arguments
String[] mSelectionArgs = {""};

// Queries the user words list and returns results
Cursor cursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause                    // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows
```

Where the `Uri` parameter maps to the table in the provider. The `projection` is an array of columns that should be included for each row retrieved. The `selection` specifies the criteria for selecting rows and the `sortOrder` specifies the order in which rows appear in the returned `Cursor`.

The `ContentResolver.query()` client method always returns a `Cursor` containing the columns specified by the query. A `Cursor` object provides random read access to the rows and columns it contains. Using `Cursor` methods, we can iterate over the rows in the results, determine the data type of each column, get the data out of a column, and examine other properties of the results, as we did in the [Storing data in SQLite]({{site.baseurl}}/lessons/08) lesson.


## Content URIs

A **content URI** is a URI that identifies data in a provider. Content URIs include the symbolic name of the entire provider (its **authority**) and a name that points to a table (a **path**). When we call a client method to access a table in a provider, the content URI for the table is one of the arguments.

A content URI for content providers has this general form:
```
scheme://authority/path/ID
```

- **scheme** is always `content://` for content URIs.
- **authority** represents the domain, and for content providers customarily ends in `.provider`.
- **path** is the path to the data.
- **ID** uniquely identifies the data set to search.

In the preceding lines of code, the constant `CONTENT_URI` contains the content URI of the user dictionary's "words" table. The `ContentResolver` object parses out the URI's authority, and uses it to "resolve" the provider by comparing the authority to a system table of known providers. The `ContentResolver` can then dispatch the query arguments to the correct provider.

The ContentProvider uses the path part of the content URI to choose the table to access. A provider usually has a path for each table it exposes.

The full URI for the "words" table is: 

```
content://user_dictionary/words
```

Where the `user_dictionary` string is the provider's *authority*, and the `words` string is the table's *path*. The string `content://` (the scheme) is always present, and identifies this as a content URI.

Many providers allow us to access a single row in a table by appending an ID value to the end of the URI. For example, to retrieve a row whose `_ID` is `4` from user dictionary, we can use this content URI:

``` java
Uri singleUri = ContentUris.withAppendedId(UserDictionary.Words.CONTENT_URI,4);
```

We often use id values when we've retrieved a set of rows and then want to update or delete one of them.


### References
[Content Providers API Guide](https://developer.android.com/guide/topics/providers/content-providers.html)<br>
[Content Provider Basics API Guide](https://developer.android.com/guide/topics/providers/content-provider-basics.html)<br>
[`Manifest.permission` reference](https://developer.android.com/reference/android/Manifest.permission.html)<br>
[`ContentResolver` reference](https://developer.android.com/reference/android/content/ContentResolver.html)<br>