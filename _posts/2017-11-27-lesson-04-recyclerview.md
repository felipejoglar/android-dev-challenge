---
layout: post
title: Lesson 4 - RecyclerView
cover: lesson-4-banner.png
author: Felipe Joglar
permalink: /lessons/04
summary: "Many apps need the ability to display large lists of data. We'll learn how to do this efficiently with Adapters and the RecyclerView class. By the end of this lesson, we'll be efficiently showing off many rows data."
---

<img src="{{site.baseurl}}/assets/banner/{{page.cover}}" alt="{{pagle.title}}"/>

{{page.summary}}

## Index

- [RecyclerView](#recyclerview)
- [ViewHolder](#viewholder)
- [Adapter](#adapter)
- [LayoutManager](#layoutmanager)
- [Handling click events](#handling-click-events)
- [Hooking everything up in the Activity](#hooking-everything-up-in-the-activity)


## RecyclerView

The `RecyclerView` widget is a *more advanced and flexible* version of `ListView` with improved performance and customizability. It was included in API level 22 (Lollipop) in the support-v7 library. This widget is a container for displaying large data sets that can be scrolled very efficiently by maintaining a limited number of views. 

<p align="center">
    <img src="{{site.baseurl}}/assets/images/recyclerview-widget.png" alt="The RecyclerView widget" width="700"/>
</p>

The `RecyclerView` class simplifies the display and handling of large data sets by providing:
- Layout managers for positioning items.
- Default animations for common item operations, such as removal or addition of items.

<img src="{{site.baseurl}}/assets/images/recyclerview-vs-listview.png" alt="RecyclerView vs ListView" width="400" align="right" hspace="10">

Under the `RecyclerView` model, several different components work together to display our data. The overall container for our dynamic user interface is a `RecyclerView` object. We add this object to our activity's or fragment's layout; the `RecyclerView`, fills itself with smaller views representing the individual items. The `RecyclerView` uses the *layout manager* we provide to arrange the items. We can use one of the standard layout managers (such as `LinearLayoutManager` or `GridLayoutManager`), or implement our own. 

The individual items are represented by view holder objects. These objects are instances of the class we define by extending `RecyclerView.ViewHolder`. Each view holder is in charge of displaying a single item, and has its own view. The `RecyclerView` creates only as many view holders as are needed to display the on-screen portion of the dynamic content, plus a few extra. As the user scrolls through the list, the `RecyclerView` takes the off-screen views and rebinds them to the data which is scrolling onto the screen.

The view holder objects are managed by an *adapter*, which we create by extending the `RecyclerView.Adapter` abstract class. The adapter creates view holders as needed and binds the view holders to their data. It does this by assigning the view holder to a position, and calling the adapter's `onBindViewHolder()` method. This method uses the view holder's position to determine what the contents should be.


## ViewHolder

A ViewHolder describes an item view and metadata about its place within the RecyclerView.

`RecyclerView.Adapter` implementations should subclass ViewHolder and add fields for caching potentially expensive `findViewById(int)` results. ViewHolders belong to the adapter. Adapters should feel free to use their own custom ViewHolder implementations to store data that makes binding view contents easier. Implementations should assume that individual item views will hold strong references to ViewHolder objects and that RecyclerView instances may hold strong references to extra off-screen item views for caching purposes.

When the view is first populated, it *creates and binds* some view holders on either side of the list. That way, if the user scrolls the list, the next element is ready to display. As the user scrolls the list, the `RecyclerView` creates new view holders as necessary. It also saves the view holders which have scrolled off-screen, so they can be reused. If the user switches the direction they were scrolling, the view holders which were scrolled off the screen can be brought right back. On the other hand, if the user keeps scrolling in the same direction, the view holders which have been off-screen the longest can be rebound to new data. The view holder does not need to be created or have its view inflated; instead, the app just updates the view's contents to match the new item it was bound to.

An example ViewHolder within an Adapter could look like this:

``` java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
    
    // Provide a reference to the views for each data item
    // Complex data items may need more than one view per item, and
    // we provide access to all the views for a data item in a view holder
    public static class ViewHolder extends RecyclerView.ViewHolder {
        
        // each data item is just a string in this case
        TextView mTextView;

        public ViewHolder(View itemView) {
            super(itemView);
            mTextView = (TextView) itemView.findViewById(R.id.tv_item_id);
        }
    }

    // Rest of the Adapter
    // ...
}
```


## Adapter

The adapter is the piece that will *connect* our data to our `RecyclerView` and determine the ViewHolder which will need to be used to display that data. It is a good practice to make the adapter as "dumb" as possible. No work performed on the data should live in the adapter. Instead, we must handle all data manipulation outside of our adapter, for example in our data model.

Our Adapter must override 3 methods for it to work:
- `onCreateViewHolder(ViewGroup parent, int viewType)`, called when `RecyclerView` needs a new `RecyclerView.ViewHolder` of the given type to represent an item.

   This new ViewHolder should be constructed with a new View that can represent the items of the given type. We can either create a new View manually or inflate it from an XML layout file.

   The new ViewHolder will be used to display items of the adapter using `onBindViewHolder(ViewHolder, int, List)`. Since it will be re-used to display different items in the data set, it is a good idea to cache references to sub views of the View to avoid unnecessary `findViewById(int)` calls.

   ``` java
   // Create new views (invoked by the layout manager)
   @Override
   public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
       // create a new view
       View view = inflater.LayoutInflater.from(parent.getContext())
               .inflate(R.layout.my_item_view, parent, false);
       ViewHolder viewHolder = new ViewHolder(view);

       return viewHolder;
   }
   ```

- `onBindViewHolder(VH holder, int position)`, called by `RecyclerView` to display the data at the specified position.
   
   This method should update the contents of the itemView to reflect the item at the given position.

   Note that `RecyclerView` will not call this method again if the position of the item changes in the data set unless the item itself is invalidated or the new position cannot be determined. For this reason, we should only use the position parameter while acquiring the related data item inside this method and should not keep a copy of it. If we need the position of an item later on (e.g. in a click listener), use `getAdapterPosition()` which will have the updated adapter position.

   ``` java
   // Replace the contents of a view (invoked by the layout manager)
   @Override
   public void onBindViewHolder(ViewHolder holder, int position) {
       // - get element from your dataset at this position
       // - replace the contents of the view with that element
       holder.mTextView.setText(mDataset[position]);
   }
   ```

- `getItemCount()`, that returns the total number of items in the data set held by the adapter.

   ``` java
   // Return the size of your dataset (invoked by the layout manager)
   @Override
   public int getItemCount() {
       if (null == mDataset) return 0;
       return mDataset.length;
   }
   ```

We now have a fully functioning RecyclerView Adapter ready to do its thing.


## LayoutManager

A `LayoutManager` is responsible for measuring and positioning item views within a `RecyclerView` as well as determining the policy for when to recycle item views that are no longer visible to the user. By changing the `LayoutManager` a `RecyclerView` can be used to implement a standard vertically scrolling list, a uniform grid, staggered grids, horizontally scrolling collections and more. Several stock layout managers are provided for general use.

  - [`LinearLayoutManager`](https://developer.android.com/reference/android/support/v7/widget/LinearLayoutManager.html) arranges the items in a one-dimensional list. Using a `RecyclerView` with `LinearLayoutManager` provides functionality like the older `ListView` layout.
  - [`GridLayoutManager`](https://developer.android.com/reference/android/support/v7/widget/GridLayoutManager.html) arranges the items in a two-dimensional grid. Using a `RecyclerView` with `GridLayoutManager` provides functionality like the older `GridView` layout.
  - [`StaggeredGridLayoutManager`](https://developer.android.com/reference/android/support/v7/widget/StaggeredGridLayoutManager.html) arranges the items in a two-dimensional grid, with each column slightly offset from the one before.

If none of these layout managers suits our needs, we can create our own by extending the `RecyclerView.LayoutManager` abstract class.

## Handling click events

Handling click events on the items of a `RecyclerView` is something we have to do by our own. But it is not difficult and by following a set of steps it can be done easily.

In our Adapter:

``` java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {

    // Create a final private AdapterOnClickHandler called mClickHandler
    private final AdapterOnClickHandler mClickHandler;

    // Add an interface called AdapterOnClickHandler
    // Within that interface, define a void method that handles the onClick event
    public interface AdapterOnClickHandler {
        void onClick(String data); 
    }

    // Add a AdapterOnClickHandler as a parameter to the constructor and store it in mClickHandler
    public MyAdapter(AdapterOnClickHandler clickHandler) {
        mClickHandler = clickHandler;
    }

    // Implement View.OnClickListener in the ViewHolder class
    public static class ViewHolder extends RecyclerView.ViewHolder 
          implements View.OnClickListener {
        
        // each data item is just a string in this case
        TextView mTextView;

        public ViewHolder(View itemView) {
            super(itemView);
            mTextView = (TextView) itemView.findViewById(R.id.tv_item_id);

            // Call setOnClickListener on the view passed into the constructor
            view.setOnClickListener(this);
        }

        // Override onClick, passing the clicked item's data to mClickHandler via its onClick method
        @Override
        public void onClick(View view) {
            mClickHandler.onClick(mDataset[getAdapterPosition()]);
        }
    }

    // Rest of the Adapter
    // ...    
        
}
```

Then in our `Activity` we have to implement the `AdapterOnClickHandler.onClick` callback:

``` java
// Implement AdapterOnClickHandler from the MainActivity
public class MainActivity extends AppCompatActivity 
        implements MyAdapter.AdapterOnClickHandler{

    private MyAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Create the adapter passing the Context of the Activity
        mAdapter = new MyAdapter(this);

        // Rest of onCreate()
        // ...
    }

    // Override AdapterOnClickHandler's onClick method
    // Show a Toast when an item is clicked, displaying that item's data
    @Override
    public void onClick(String data) {
        Toast.makeText(getApplicationContext(), data, Toast.LENGTH_LONG).show();
    }

    // Rest of Activity
    // ...
}
```

## Hooking everything up in the Activity

The `Activity` will be the screen that will display our `RecyclerView` and all of its containing data to our users. We need to add one method override for all of this to work, the override `onCreate(Bundle savedInstanceState)`. In the `onCreate` method, we need to add a call to the super method and also add the `setContentView(int layoutResID)` method passing in our Activity’s layout resource id.

Then we will initialize the `MyAdapter` and `RecyclerView` in our `onCreate()` method. After that, we will need to instantiate our `RecyclerView` using the id resource that we created in our Activity’s XML layout file.

Now that we have a `RecyclerView`, there are a few more things we will need to do to make it work. One of the most important being the `LayoutManager`. The next step, setting whether or not the `RecyclerView` has a fixed size, isn’t required but it helps the Android framework optimize the `RecyclerView` by letting it know in advance the the `RecyclerView` size will not be affected by the Adapter contents.

Finally, we will need to attach our `MyAdapter` to the `RecyclerView`.

But first of all we have to add the dependencies for `RecyclerView` to our app's module Gradle file, where `$support_lib_version` is the appropiate version of the [Support Library](https://developer.android.com/topic/libraries/support-library/revisions.html):

``` groovy
dependencies {
    ...
    compile 'com.android.support:recyclerview-v7:$support_lib_version'
}
```

Now our complete `Activity` class should look something like this:

``` java
public class MainActivity extends AppCompatActivity 
        implements MyAdapter.AdapterOnClickHandler{

    private MyAdapter mAdapter;
    private RecyclerView mRecyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Create the adapter passing the Context of the Activity
        mAdapter = new MyAdapter(this);

        // Instantiate our RecyclerView
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview_id);

        mRecyclerView.setHasFixedSize(true);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mRecyclerView.setAdapter(mAdapter);
    }

    // Override AdapterOnClickHandler's onClick method
    // Show a Toast when an item is clicked, displaying that item's data
    @Override
    public void onClick(String data) {
        Toast.makeText(getApplicationContext(), data, Toast.LENGTH_LONG).show();
    }

    // Rest of Activity
    // ...
}
```

And our XML files should be like these:

``` xml
<!--
    activity_main.xml
-->
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview_id"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>

<!--
    my_item_view.xml
-->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:orientation="vertical">

    <TextView
        android:id="@+id/tv_item_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="16dp"/>
</LinearLayout>
```


### References
[Recycler View API Guide](https://developer.android.com/guide/topics/ui/layout/recyclerview.html)<br>
[`RecyclerView` reference](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)<br>
[`RecyclerView.ViewHolder` reference](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ViewHolder.html)<br>
[`RecyclerView.Adapter` reference](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.Adapter.html)<br>
[`RecyclerView.LayoutManager` reference](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.LayoutManager.html)<br>
[Creating Lists and Cards](https://developer.android.com/training/material/lists-cards.html#RecyclerView)<br>
[Android Fundamentals: Working with the RecyclerView, Adapter, and ViewHolder Pattern](https://willowtreeapps.com/ideas/android-fundamentals-working-with-the-recyclerview-adapter-and-viewholder-pattern/) by WillowTreeApps