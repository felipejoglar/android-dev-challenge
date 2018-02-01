---
layout: page
title: Lesson 3 - Connect to the Internet
cover: lesson-3-banner.png
---

# Lesson 3 - Connect to the Internet

![Lesson 3 Banner](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/lesson-3-banner.png)


## Index

- [Logging](#logging)
- [Toasts](#toasts)
- [Resources](#resources)
- [Menus](#menus)
- [Fetching HTTP request](#fetching-http-request)
- [Permissions](#permissions)
- [Threading basics](#threading-basics)
- [AsyncTask](#asynctask)


## Logging

Logging is one of the most useful techniques when we are debugging. The Android system uses a centralized system for all logs, the Logcat. The Logcat window in Android Studio displays system messages, such as when a garbage collection occurs, and messages that you added to your app with the Log class. It displays messages in real time and keeps a history so you can view older messages.<br>
To display just the information of interest, we can create filters, modify how much information is displayed in messages, set priority levels, display messages produced by app code only, and search the log. By default, logcat shows the log output related to the most recently run app only.<br>
When an app throws an exception, logcat shows a message followed by the associated stack trace containing links to the line of code.

The Log class allows us to create log messages that appear in logcat. Generally, we should use the following log methods, listed in order from the highest to lowest priority:

- `Log.e(String, String)` (error)
- `Log.w(String, String)` (warning)
- `Log.i(String, String)` (information)
- `Log.d(String, String)` (debug)
- `Log.v(String, String)` (verbose)

![Android Studio Logcat](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/logcat.png)

- ### References
[Write and View Logs with Logcat](https://developer.android.com/studio/debug/am-logcat.html)<br>
[Log reference](https://developer.android.com/reference/android/util/Log.html)


## Toasts

<img src="https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/toast.png" width="300" align="right" hspace="10">

A toast provides simple feedback about an operation in a small popup. It only fills the amount of space required for the message and the current activity remains visible and interactive. Toasts automatically disappear after a timeout.

They are very useful to show to the user simple feedback, as a task that has finished or small messages. We also use them during development to provide information to us visually within the app and not having to put a log message and look for it in the Logcat.

The syntax is very easy, the method takes three parameters: the application `Context`, the text message, and the duration for the toast, don't forget to call `show()` to display the `Toast`:

``` java
Context context = getApplicationContext();
CharSequence text = "Hello toast!";
int duration = Toast.LENGTH_SHORT;

Toast toast = Toast.makeText(context, text, duration);
toast.show();
```

- ### References
[Toast API guide](https://developer.android.com/guide/topics/ui/notifiers/toasts.html)<br>
[Toast reference](https://developer.android.com/reference/android/widget/Toast.html)

## Resources

Resources are the additional files and static content that our code uses, such as bitmaps, layout definitions, user interface strings, animation instructions, and more.<br>
We should always externalize resources such as images and strings from our application code, so that we can maintain them independently. Externalizing our resources also allows us to provide alternative resources that support specific device configurations such as different languages or screen sizes.

The res directory is where we should put our resource files. It's included in every Android project. Inside of the res directory, are sub folder for the following types of resources. We may have a subset of these directories, depending on the types of resources we're using in our app. 

**How to use Resources in XML and Java?** We've already seen resources in action. For example, in the `MainActivity`, you have already seen usage of resources. When we say `setContentView(R.layout.activity_main)`, we are referencing a resource (the `activity_main.xml`) file to use as the layout of `MainActivity`.

Let's see som examples of accessing resources:

In XML:
``` xml
<Button
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="@string/submit" />
```

In Java:
``` java
ImageView imageView = (ImageView) findViewById(R.id.myimageview);
imageView.setImageResource(R.drawable.myimage);
```

In Java, we can get a String saved in res->values->strings.xml by calling the `getString` method. If we're in an `Activity`, you can just call `getString`, and pass in the String resource ID. For example, let's look at Sunshine's strings.xml file:

``` xml
<string name="today">Today</string>
```

So to access that String from code:

``` java
String myString = getString(R.string.today);
```

In XML, you can access a String by using the @string accessor method. For the same String defined above, you could access it like this:

``` xml
<TextView text=”@string/today />
```

- ### References
[App Resources](https://developer.android.com/guide/topics/resources/index.html)<br>
[Providing Resources](https://developer.android.com/guide/topics/resources/providing-resources.html)<br>
[Accessing Resources
](https://developer.android.com/guide/topics/resources/accessing-resources.html)

## Menus

Menus are a common user interface component in many types of applications. To provide a familiar and consistent user experience, we should use the Menu APIs to present user actions and other options in your activities.

Android provides a standard XML format to define menu items. Instead of building a menu in your activity's code, you should define a menu and all its items in an XML menu resource. Like this:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/search"
          android:icon="@drawable/ic_search"
          android:title="@string/search"
          android:showAsAction="ifRoom"/>
</menu>
```

In code, to specify the options menu for an activity, we need to override `onCreateOptionsMenu()`. In this method, we can inflate our menu resource. 

``` java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.menu_name, menu);
    return true;
}
```

When the user selects an item from the options menu the system calls our activity's `onOptionsItemSelected()` method. This method passes the MenuItem selected. You can identify the item by calling getItemId(), which returns the unique ID for the menu item. We need to match this ID against known menu items to perform the appropriate action. For example:

``` java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle item selection
    switch (item.getItemId()) {
        case R.id.search:
            newSearch();
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

When we successfully handle a menu item, return `true`. If you don't handle the menu item, you should call the superclass implementation of `onOptionsItemSelected()` which returns `false`.

- ### References
[Menus API guide](https://developer.android.com/guide/topics/ui/menus.html)<br>
[Menu Resource API guide](https://developer.android.com/guide/topics/resources/menu-resource.html)


## Fetching HTTP request

Now we are going to connect our app with Internet to get the data needed to show in our app. So the first part would be get the permission so we can access the Internet. We will see [permission](#permissions) in detail bellow, for now we have to add an `<uses-permission` to our app manifest:

``` xml
<uses-permission android:name="android.permission.INTERNET" />
```

Most network-connected Android apps use [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) to send and receive data. The Android platform includes the `HttpsURLConnection` client, which supports TLS, streaming uploads and downloads, configurable timeouts, IPv6, and connection pooling.

To avoid creating an unresponsive UI, we must not perform network operations on the UI thread. By default, Android 3.0 (API level 11) and higher requires you to perform network operations on a thread other than the main UI thread; if we don't, a `NetworkOnMainThreadException` is thrown. We will see how to solve it [later](#threading-basics).

Uses of this class follow a pattern:

1. Obtain a new `HttpURLConnection` by calling `URL.openConnection()` and casting the result to `HttpURLConnection`.
2. Prepare the request. The primary property of a request is its `URI`. Request headers may also include metadata such as credentials, preferred content types, and session cookies.
3. Optionally upload a request body. Instances must be configured with `setDoOutput(true)` if they include a request body. Transmit data by writing to the stream returned by `getOutputStream()`.
4. Read the response. Response headers typically include metadata such as the response body's content type and length, modified dates and session cookies. The response body may be read from the stream returned by `getInputStream()`. If the response has no body, that method returns an empty stream.
5. Disconnect. Once the response body has been read, the `HttpURLConnection` should be closed by calling `disconnect()`. Disconnecting releases the resources held by a connection so they may be closed or reused.

``` java
URL url = new URL("http://www.android.com/");
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
try {
    InputStream in = new BufferedInputStream(urlConnection.getInputStream());
    readStream(in);
} finally {
    urlConnection.disconnect();
}
```

  - ### References
[Connecting to the Network](https://developer.android.com/training/basics/network-ops/connecting.html)<br>
[HttpURLConnection](https://developer.android.com/reference/java/net/HttpURLConnection.html)<br>
[Networking Security Tips](https://developer.android.com/training/articles/security-tips.html#Networking)<br>
[`Uri.Builder`](https://developer.android.com/reference/android/net/Uri.Builder.html)


## Permissions

Permissions are the way to maintain security for the system and users, Android requires our apps to request permission before the apps can use certain system data and features. Depending on how sensitive the area is, the system may grant the permission automatically, or it may ask the user to approve the request.

Because each Android app operates in a process sandbox, apps must explicitly request access to resources and data outside their sandbox. They request this access by declaring the *permissions* they need for additional capabilities not provided by the basic sandbox.

A basic Android app has no permissions associated with it by default, meaning it cannot do anything that would adversely impact the user experience or any data on the device. To make use of protected features of the device, you must include one or more `<uses-permission>` tags in your app manifest. For example:

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.app.myapp" >
    <!-- Permission for accessing the Internet 
    Allows applications to open network sockets. -->
    <uses-permission android:name="android.permission.INTERNET" />
    ...
</manifest>
```

Beginning in Android 6.0 (API level 23), users grant permissions to apps while the app is running, not when they install the app.

- ### References
[System Permissions](https://developer.android.com/guide/topics/permissions/index.html)<br>
[Requesting Permissions](https://developer.android.com/guide/topics/permissions/requesting.html)<br>
[Requesting Permissions at Run Time](https://developer.android.com/training/permissions/requesting.html)<br>
[<uses-permission> API guide](https://developer.android.com/guide/topics/manifest/uses-permission-element.html)<br>
[App Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro.html)<br>
[Manifest permission list](https://developer.android.com/reference/android/Manifest.permission.html)


## Threading basics

Every Android developer, at one point or another, needs to deal with threads in their application.

When an application is launched in Android, it creates the first thread of execution, known as the *main thread* or *UI thread*. The main thread is responsible for dispatching events to the appropriate user interface widgets as well as communicating with components from the Android UI toolkit.

To keep your application responsive, it *is essential to avoid using the main thread to perform any operation that may end up keeping it blocked*.

Network operations and database calls, as well as loading of certain components, are common examples of operations that one should avoid in the main thread. When they are called in the main thread, they are called synchronously, which means that the UI will remain completely unresponsive until the operation completes. For this reason, they are usually performed in separate threads, which thereby avoids blocking the UI while they are being performed.

Android provides many ways of creating and managing threads, and many third-party libraries exist that make thread management a lot more pleasant.

- ### References
[Processes and Threads](https://developer.android.com/guide/components/processes-and-threads.html)<br>
[Android Threading: All You Need to Know](https://www.toptal.com/android/android-threading-all-you-need-to-know)


## AsyncTask

So in order to avoid that our application block the UI thread and do this long connecting to the Internet process, we need to call the method that fetchs HTTP request from a background thread, and this can be done with a class Android provides called `AsyncTask`.

`AsyncTask` enables proper and easy use of the UI thread. This class allows us to perform background operations and publish results on the UI thread without having to manipulate threads and/or handlers.

An asynchronous task is defined by a computation that runs on a background thread and whose result is published on the UI thread. An asynchronous task is defined by 3 generic types, called `Params`, `Progress` and `Result`, and 4 steps, called `onPreExecute`, `doInBackground`, `onProgressUpdate` and `onPostExecute`.

When an asynchronous task is executed, the task goes through 4 steps:

1. `onPreExecute()`, invoked on the UI thread before the task is executed. This step is normally used to setup the task, for instance by showing a progress bar in the user interface.
2. `doInBackground(Params...)`, invoked on the background thread immediately after `onPreExecute()` finishes executing. This step is used to perform *background computation that can take a long time*. The parameters of the asynchronous task are passed to this step. The result of the computation must be returned by this step and will be passed back to the last step. This step can also use `publishProgress(Progress...)` to publish one or more units of progress. These values are published on the UI thread, in the `onProgressUpdate(Progress...)` step.
3. `onProgressUpdate(Progress...)`, invoked on the UI thread after a call to `publishProgress(Progress...)`. The timing of the execution is undefined. This method is used to display any form of progress in the user interface while the background computation is still executing. For instance, it can be used to animate a progress bar or show logs in a text field.
4. `onPostExecute(Result)`, invoked on the UI thread after the background computation finishes. The result of the background computation is passed to this step as a parameter.

`AsyncTask` must be subclassed to be used. The subclass will override at least one method (`doInBackground(Params...)`), and most often will override a second one (`onPostExecute(Result)`)`.

``` java
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
    
    @Override
    protected Long doInBackground(URL... urls) {
        int count = urls.length;
        long totalSize = 0;
        
        for (int i = 0; i < count; i++) {
            totalSize += Downloader.downloadFile(urls[i]);
            publishProgress((int) ((i / (float) count) * 100));
            
            // Escape early if cancel() is called
            if (isCancelled()) break;
        }
        return totalSize;
    }

    @Override
    protected void onProgressUpdate(Integer... progress) {
        setProgressPercent(progress[0]);
    }

    @Override
    protected void onPostExecute(Long result) {
        showDialog("Downloaded " + result + " bytes");
    }
}
```
 
Once created, a task is executed very simply:

```  java
new DownloadFilesTask().execute(url1, url2, url3);
```

`AsyncTask`, however, falls short if you need your deferred task to run beyond the lifetime of the activity/fragment. It is worth noting that even something as simple as screen rotation can cause the activity to be destroyed.

- ### References
[Threading Performance](https://developer.android.com/topic/performance/threads.html)<br>
[`AsyncTask`](https://developer.android.com/reference/android/os/AsyncTask.html)


###### Note: the images of the headers used in this serie of articles are from Udacity's [Developing Android Apps Course](https://www.udacity.com/course/new-android-fundamentals--ud851)
