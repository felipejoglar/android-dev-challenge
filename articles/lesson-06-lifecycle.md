# Lesson 6 - Lifecycle

![Lesson 6 Banner](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/lesson-6-banner.png)


## Index

- [Activity Lifecycle](#activity-lifecycle)
- [Loader](#loader)
- [AsynctaskLoader](#asynctaskloader)
- [Implementing a `Loader`](#implementing-a-loader)


## Activity Lifecycle

As a user navigates through, out of, and back to our app, the `Activity` instances in our app transition through different states in their lifecycle. The `Activity` class provides a number of callbacks that allow the activity to know that a state has changed: that the system is creating, stopping, or resuming an activity, or destroying the process in which the activity resides.

Within the lifecycle callback methods, we can declare how our activity behaves when the user leaves and re-enters the activity. A good implementation of the lifecycle callbacks can help ensure that our app avoids:

- Crashing if the user receives a phone call or switches to another app while using our app.
- Consuming valuable system resources when the user is not actively using it.
- Losing the user's progress if they leave our app and return to it at a later time.
- Crashing or losing the user's progress when the screen rotates between landscape and portrait orientation.

Activities in the system are managed as an *activity stack*. When a new activity is started, it is placed on the top of the stack and becomes the running activity -- the previous activity always remains below it in the stack, and will not come to the foreground again until the new activity exits.

<p align="center">
<img src="https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/activity-basic-lifecycle.png" alt="Activity Basic Lifecycle" style="width: 10px;"/>
</p>

An activity has essentially four states:

- If an activity is in the foreground of the screen (at the top of the stack), it is **active** or **running**.
- If an activity has lost focus but is still visible (that is, a new non-full-sized or transparent activity has focus on top of our activity), it is **paused**. A paused activity is completely alive (it maintains all state and member information and remains attached to the window manager), but can be killed by the system in extreme low memory situations.
- If an activity is completely obscured by another activity, it is **stopped**. It still retains all state and member information, however, it is no longer visible to the user so its window is hidden and it will often be killed by the system when memory is needed elsewhere.
- If an activity is paused or stopped, the system can drop the activity from memory by either asking it to finish, or simply killing its process. When it is displayed again to the user, it must be completely restarted and restored to its previous state.

The following diagram shows the important state paths of an Activity. The square rectangles represent callback methods you can implement to perform operations when the Activity moves between states. The colored ovals are major states the Activity can be in.

There are three key loops we may be interested in monitoring within our activity:

- The **entire lifetime** of an activity happens between the first call to `onCreate(Bundle)` through to a single final call to `onDestroy()`. An activity will do all setup of "global" state in `onCreate()`, and release all remaining resources in `onDestroy()`.
- The **visible lifetime** of an activity happens between a call to `onStart()` until a corresponding call to `onStop()`. During this time the user can see the activity on-screen, though it may not be in the foreground and interacting with the user. Between these two methods we can maintain resources that are needed to show the activity to the user. The `onStart()` and `onStop()` methods can be called multiple times, as the activity becomes visible and hidden to the user.
- The **foreground lifetime** of an activity happens between a call to `onResume()` until a corresponding call to `onPause()`. During this time the activity is in front of all other activities and interacting with the user. An activity can frequently go between the resumed and paused states -- for example when the device goes to sleep, when an activity result is delivered, when a new intent is delivered -- so the code in these methods should be fairly lightweight.

<p align="center">
<img src="https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/activity-lifecycle.png" alt="Activity Lifecycle" style="width: 10px;"/>
</p>

In general the movement through an activity's lifecycle looks like this:

| Method | Description |
| --- | --- |
| `onCreate()` | Called when the activity is first created. This is where we should do all of our normal static set up: create views, bind data to lists, etc. This method also provides us with a `Bundle` containing the activity's previously frozen state, if there was one. Always followed by `onStart()`. |
| `onRestart()` | Called after our activity has been stopped, prior to it being started again. Always followed by `onStart()`. |
| `onStart()` | Called when the activity is becoming visible to the user. Followed by `onResume()` if the activity comes to the foreground, or `onStop()` if it becomes hidden. |
| `onResume()` | Called when the activity will start interacting with the user. At this point our activity is at the top of the activity stack, with user input going to it. Always followed by `onPause()`. |
| `onPause()` | Called when the system is about to start resuming a previous activity. This is typically used to commit unsaved changes to persistent data, stop animations and other things that may be consuming CPU, etc. Implementations of this method must be very quick because the next activity will not be resumed until this method returns. Followed by either `onResume()` if the activity returns back to the front, or `onStop()` if it becomes invisible to the user. |
| `onStop()` | Called when the activity is no longer visible to the user, because another activity has been resumed and is covering this one. This may happen either because a new activity is being started, an existing one is being brought in front of this one, or this one is being destroyed. Followed by either `onRestart()` if this activity is coming back to interact with the user, or `onDestroy()` if this activity is going away. |
| `onDestroy()` | The final call we receive before our activity is destroyed. This can happen either because the activity is finishing (someone called `finish()` on it, or because the system is temporarily destroying this instance of the activity to save space. We can distinguish between these two scenarios with the `isFinishing()` method. |

In addition, the method `onSaveInstanceState(Bundle)` is called before placing the activity in such a background state, allowing us to save away any dynamic instance state in our activity into the given `Bundle`, to be later received in `onCreate(Bundle)` or `onRestoreInstanceState(Bundle)` if the activity needs to be re-created.

This is an example `Activity` with the lifecycle methods overriden and use `onSaveInstanceState(Bundle outState)` to maintain the state across configuration changes:

``` java
public class MainActivity extends AppCompatActivity {

    /*
     * This constant String will be used to store the content of the TextView used.
     */
    private static final String LIFECYCLE_CALLBACKS_TEXT_KEY = "callbacks";
    
    private TextView mLifecycleDisplay;

    // ...

    /**
     * Called when the activity is first created. This is where we should do all of our normal
     * static set up: create views, bind data to lists, etc.
     * <p>
     * Always followed by onStart().
     *
     * @param savedInstanceState The Activity's previously frozen state, if there was one.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mLifecycleDisplay = (TextView) findViewById(R.id.tv_lifecycle_events_display);

        // If savedInstanceState is not null and contains LIFECYCLE_CALLBACKS_TEXT_KEY, 
        // set that text on our TextView.
        if (savedInstanceState != null) {
            if (savedInstanceState.containsKey(LIFECYCLE_CALLBACKS_TEXT_KEY)) {
                String allSavedText = savedInstanceState.getString(LIFECYCLE_CALLBACKS_TEXT_KEY);
                mLifecycleDisplay.setText(allSavedText);
            }
        }
    }

    /**
     * Called when the activity is becoming visible to the user.
     * <p>
     * Followed by onResume() if the activity comes to the foreground, or onStop() if it becomes
     * hidden.
     */
    @Override
    protected void onStart() {
        super.onStart();
    }

    /**
     * Called when the activity will start interacting with the user. At this point our activity
     * is at the top of the activity stack, with user input going to it.
     * <p>
     * Always followed by onPause().
     */
    @Override
    protected void onResume() {
        super.onResume();
    }

    /**
     * Called when the system is about to start resuming a previous activity. This is typically
     * used to commit unsaved changes to persistent data, stop animations and other things that may
     * be consuming CPU, etc. Implementations of this method must be very quick because the next
     * activity will not be resumed until this method returns.
     * <p>
     * Followed by either onResume() if the activity returns back to the front, or onStop() if it
     * becomes invisible to the user.
     */
    @Override
    protected void onPause() {
        super.onPause();
    }

    /**
     * Called when the activity is no longer visible to the user, because another activity has been
     * resumed and is covering this one. This may happen either because a new activity is being
     * started, an existing one is being brought in front of this one, or this one is being
     * destroyed.
     * <p>
     * Followed by either onRestart() if this activity is coming back to interact with the user, or
     * onDestroy() if this activity is going away.
     */
    @Override
    protected void onStop() {
        super.onStop();
    }

    /**
     * Called after our activity has been stopped, prior to it being started again.
     * <p>
     * Always followed by onStart()
     */
    @Override
    protected void onRestart() {
        super.onRestart();
    }

    /**
     * The final call we receive before our activity is destroyed. This can happen either because
     * the activity is finishing (someone called finish() on it, or because the system is
     * temporarily destroying this instance of the activity to save space. You can distinguish
     * between these two scenarios with the isFinishing() method.
     */
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }


    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);

        // Put the text from the TextView in the outState bundle
        String textToSave = mLifecycleDisplay.getText().toString();
        outState.putString(LIFECYCLE_CALLBACKS_TEXT_KEY, textToSave);
    }

    // Rest of the activity...
}
```

## Loader

`Loaders` are responsible for performing queries on a separate thread, monitoring the data source for changes, and delivering new results to a registered listener (usually the `LoaderManager`) when changes are detected. These characteristics make `Loaders` a powerful tool for several reasons:

1. *They encapsulate the actual loading of data*. The `Activity` no longer needs to know how to load data. Instead, the `Activity` delegates the task to the `Loader`, which carries out the request behind the scenes and has its results delivered back to the `Activity`.

2. *They abstract out the idea of threads from the client*. The `Activity` does not need to worry about offloading queries to a separate thread, as the `Loader` will do this automatically. This reduces code complexity and eliminates potential thread-related bugs.

3. *They are entirely event-driven*. Loaders monitor the underlying data source and automatically perform new loads for up-to-date results when changes are detected. This makes working with Loaders easy, as the client can simply trust that the `Loader` will auto-update its data on its own. All the `Activity` has to do is initialize the `Loader` and respond to any results that might be delivered. Everything in between is done by the Loader.

`LoaderManager` is responsible for managing one or more Loaders associated with an `Activity`. Each `Activity` has exactly one `LoaderManager` instance that is in charge of starting, stopping, retaining, restarting, and destroying its Loaders. These events are sometimes initiated directly by the client, by calling `initLoader()`, `restartLoader()`, or `destroyLoader()`. Just as often, however, these events are triggered by major Activity lifecycle events. For example, when an `Activity` is destroyed, the `Activity` instructs its `LoaderManager` to destroy and close its Loaders (as well as any resources associated with them).

The `LoaderManager` does not know how data is loaded, nor does it need to. Rather, the `LoaderManager` instructs its Loaders when to start/stop/reset their load, *retaining their state across configuration changes* and providing a simple interface for delivering results back to the client.


## AsynctaskLoader

`AsynctaskLoader` is an abstract `Loader` that provides an `AsyncTask` to do the work.

`AsyncTaskLoader` is a better choice for Activity-bound thread management, because it handles lifecycle changes correctly, delivering the result to the current active activity, preventing duplication of background threads, and helping to eliminate duplication of zombie activities.


## Implementing a `Loader`. 

There is a lot thatweyou must keep in mind when implementing our own custom Loaders. Subclasses must implement `loadInBackground()` and should override `onStartLoading()`, `onStopLoading()`, `onReset()`, `onCanceled()`, and `deliverResult(D results)` to achieve a fully functioning Loader. Overriding these methods is very important as the `LoaderManager` will call them regularly depending on the state of the `Activity` lifecycle. For example, when an `Activity` is first started, the `Activity` instructs the `LoaderManager` to start each of its Loaders in `Activity#onStart()`. If a `Loader` is not already started, the `LoaderManager` calls `startLoading()`, which puts the `Loader` in a started state and immediately calls the Loader’s `onStartLoading()` method. In other words, a lot of work that the `LoaderManager` does behind the scenes relies on the Loader being correctly implemented.

``` java
// Implement the proper LoaderCallbacks interface and the methods of that interface
public class MainActivity extends AppCompatActivity implements
        LoaderManager.LoaderCallbacks<String[]> {

    /*
     * This ID will uniquely identify the Loader. We can use it, for example, 
     * to get a handle on our Loader at a later point in time through the 
     * support LoaderManager.
     */
    private static final int LOADER_ID = 22;

    private ProgressBar mLoadingIndicator;

    // ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_forecast);

        // Implementation of onCreate() ...

        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getSupportLoaderManager().initLoader(SUNSHINE_LOADER_ID, null, this);
    }

    /**
     * Instantiate and return a new Loader for the given ID.
     */
    @Override
    public Loader<String[]> onCreateLoader(int id, Bundle args) {
        return new AsyncTaskLoader<String[]>(this) {
            // When we attempt to access a loader (for example, through initLoader()), 
            // it checks to see whether the loader specified by the ID exists. 
            // If it doesn't, it triggers the LoaderManager.LoaderCallbacks method 
            // onCreateLoader(). This is where we create a new loader.

            /* This String array will hold and help cache our data */
            String[] mCachedData = null;

            /**
             * Subclasses of AsyncTaskLoader must implement this to take care of 
             * loading their data.
             */
            @Override
            protected void onStartLoading() {
                if (mCachedData != null) {
                    deliverResult(mCachedData);
                } else {
                    mLoadingIndicator.setVisibility(View.VISIBLE);
                    forceLoad();
                    // Force an asynchronous load. Unlike startLoading() this will 
                    // ignore a previously loaded data set and load a new one. 
                    // This simply calls through to the implementation's onForceLoad(). 
                    // We generally should only call this when the loader is 
                    // started -- that is, isStarted() returns true.
                    // Must be called from the process's main thread.
                }
            }

            /**
             * This is the method of the AsyncTaskLoader that will load the data
             *  in the background.
             *
             * @return The data from the source as an array of Strings.
             *         null if an error occurs
             */
            @Override
            public String[] loadInBackground() {

                URL dataRequestURL = new URL("https://get.data.example.json");

                try {
                    // This is a ficticious method that gets the data from a given
                    // URL and parses it to a String array.
                    String[] simpleJsonData = 
                            getSimpleWeatherStringsFromJson(dataRequestURL);

                    return simpleJsonData;
                } catch (Exception e) {
                    e.printStackTrace();
                    return null;
                }
            }

            /**
             * Sends the result of the load to the registered listener.
             *
             * @param data The result of the load
             */
            @Override
            public void deliverResult(String[] data) {
                mCachedData = data;
                super.deliverResult(data);
            }
        };
    }

    /**
     * Called when a previously created loader has finished its load.
     *
     * @param loader The Loader that has finished.
     * @param data The data generated by the Loader.
     */
    @Override
    public void onLoadFinished(Loader<String[]> loader, String[] data) {
        mLoadingIndicator.setVisibility(View.INVISIBLE);
        if (data == null) {
            showErrorMessage();
        } else {
            // This is a ficticious method that shows the data to the user
            // by updating the app's UI.
            showDataView(data);
        }
    }

    /**
     * Called when a previously created loader is being reset, and thus
     * making its data unavailable. The application should at this point
     * remove any references it has to the Loader's data.
     *
     * @param loader The Loader that is being reset.
      */
    @Override
    public void onLoaderReset(Loader<String[]> loader) {
        /*
         * We aren't using this method in our example, but we are required to Override
         * it to implement the LoaderCallbacks<String> interface
         */
    }

    // Rest of the Activity implementation

}
```


### References
[`Activity` reference](https://developer.android.com/reference/android/app/Activity.html)<br>
[The Activity Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle.html)<br>
[The Android Lifecycle cheat sheet — part I: Single Activities](https://medium.com/@JoseAlcerreca/the-android-lifecycle-cheat-sheet-part-i-single-activities-e49fd3d202ab) by José Alcérreca<br>
[The Android Lifecycle cheat sheet — part II: Multiple activities](https://medium.com/@JoseAlcerreca/the-android-lifecycle-cheat-sheet-part-ii-multiple-activities-a411fd139f24) by José Alcérreca<br>
[Activity Revival and the case of the Rotating Device](https://medium.com/google-developers/activity-revival-and-the-case-of-the-rotating-device-167e34f9a30d) by Joanna Smith<br>
[Complete Android Fragment & Activity Lifecycle](https://github.com/xxv/android-lifecycle) by Steve Pomeroy<br>
[Loaders API guide](https://developer.android.com/guide/components/loaders.html#summary)<br>
[`Loader` reference](https://developer.android.com/reference/android/content/Loader.html)<br>
[`AsyncTaskLoader` reference](https://developer.android.com/reference/android/content/AsyncTaskLoader.html)<br>
[Making loading data lifecycle aware](https://medium.com/google-developers/making-loading-data-on-android-lifecycle-aware-897e12760832) by Ian Lake<br>
[Life Before Loaders](https://www.androiddesignpatterns.com/2012/07/loaders-and-loadermanager-background.html) by Android Design Patterns blog<br>
[Understanding the LoaderManager](https://www.androiddesignpatterns.com/2012/07/understanding-loadermanager.html) by Android Design Patterns blog<br>
[Implementing Loaders](https://www.androiddesignpatterns.com/2012/08/implementing-loaders.html) by Android Design Patterns blog


###### Note: the images of the headers used in this serie of articles are from Udacity's [Developing Android Apps Course](https://www.udacity.com/course/new-android-fundamentals--ud851)
