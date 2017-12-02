# Lesson 5 - Intents

![Lesson 5 Banner](https://github.com/fjoglar/android-dev-challenge/blob/master/assets/lesson-5-banner.png)


## Index

- [Intents](#intents)
  - [Intent types](#intent-types)
  - [Building an intent](#building-an-intent)
  - [Explicit intent](#explicit-intent)
  - [Passing data through intents](#passing-data-through-intents)
  - [Implicit intent](#implicit-intent)
  - [Understanding URIs](#understanding-uris)
  - [Media Types](#media-types)


## Intents

*Intents* are asynchronous messages which allow application components to request functionality from other Android components. Intents allow us to interact with components from the same applications as well as with components contributed by other applications. For example, an activity can start an external activity for taking a picture.

Intents are objects of the `android.content.Intent` type. Our code can send them to the Android system defining the components we are targeting. For example, via the `startActivity()` method we can define that the intent should be used to start an activity.

An intent can contain data via a `Bundle`. This data can be used by the receiving component.

Although intents facilitate communication between components in several ways, there are three fundamental use cases:

+ **Starting an activity**. We can start a new instance of an `Activity` by passing an Intent to `startActivity()`. The `Intent` describes the activity to start and carries any necessary data.<br>
If we want to receive a result from the activity when it finishes, call `startActivityForResult()`. Our activity receives the result as a separate `Intent` object in our activity's `onActivityResult()` callback. 
+ **Starting a service**. A `Service` is a component that performs operations in the background without a user interface.<br>
We can start a service to perform a one-time operation by passing an `Intent` to `startService()`. The `Intent` describes the service to start and carries any necessary data.
+ **Delivering a broadcast**. A broadcast is a message that any app can receive. The system delivers various broadcasts for system events, such as when the system boots up or the device starts charging. We can deliver a broadcast to other apps by passing an `Intent` to `sendBroadcast()` or `sendOrderedBroadcast()`.


### Intent types

There are two types of intents:

+ **Explicit intents** specify the component to start by name. We'll typically use an explicit intent to start a component in our own app, because we know the class name of the activity or service we want to start. For example, we can start a new activity in response to a user action or start a service to download a file in the background.
+ **Implicit intents** do not name a specific component, but instead declare a general action to perform, which allows a component from another app to handle it. For example, if we want to show the user a location on a map, we can use an implicit intent to request that another capable app show a specified location on a map.


### Building an intent

An `Intent` object carries information that the Android system uses to determine which component to start, plus information that the recipient component uses in order to properly perform the action (such as the action to take and the data to act upon).

The primary information contained in an `Intent` is the following:

+ **Component name**<br>
The name of the component to start. This is optional, but it's the critical piece of information that makes an intent *explicit*, meaning that the intent should be delivered only to the app component defined by the component name. Without a component name, the intent is implicit and the system decides which component should receive the intent based on the other intent information. If we need to start a specific component in our app, we should specify the component name.

+ **Action**<br>
A string that specifies the generic action to perform (such as view or pick).
In the case of a *broadcast intent*, this is the action that took place and is being reported. The action largely determines how the rest of the intent is structured, particularly the information that is contained in the data and extras.

+ **Data**<br>
The URI (a `Uri` object) that references the data to be acted on and/or the MIME type of that data. The type of data supplied is generally dictated by the intent's action.
When creating an intent, it's often important to specify the type of data (its MIME type) in addition to its URI. Specifying the MIME type of our data helps the Android system find the best component to receive our intent.

+ **Category**<br>
A string containing additional information about the kind of component that should handle the intent. Any number of category descriptions can be placed in an intent, but most intents do not require a category.

These properties listed above represent the defining characteristics of an intent. By reading these properties, the Android system is able to resolve which app component it should start. 

However, an intent can carry additional information that does not affect how it is resolved to an app component. An intent can also supply the following information:

+ **Extras**<br>
Key-value pairs that carry additional information required to accomplish the requested action. Just as some actions use particular kinds of data URIs, some actions also use particular extras.

+ **Flags**<br>
Flags are defined in the `Intent` class as metadata for the intent. The flags may instruct the Android system how to launch an activity (for example, which task the activity should belong to) and how to treat it after it's launched (for example, whether it belongs in the list of recent activities).


### Explicit intent

Explicit intents explicitly define the component which should be called by the Android system, by using the Java class as identifier, they are typically used within an application as the classes in an application are controlled by the application developer. 

``` java
// Executed in an Activity, so 'this' is the Context
Intent startActivityTwoIntent = new Intent(this, ActivityTwo.class);
startActivity(startActivityTwoIntent);
```

The `Intent(Context, Class)` constructor supplies the app `Context` and the component a `Class` object. As such, this intent explicitly starts the `ActivityTwo` class in the app.


### Passing data through intents

Optionally an intent can also contain additional data based on an instance of the `Bundle` class which can be retrieved from the intent via the `getExtras()` method.

We can also add data directly to the `Bundle` via the `putExtra()` methods of the `Intent` object. Extras are key/value pairs. The key is always of type `String` and as value we can use the primitive data types (`int`, `float`, …​) plus objects of type `String`, `Bundle`, `Parcelable` and `Serializable`.

The receiving component can access this information via the `getAction()` and `getData()` methods on the `Intent` object. This `Intent` object can be retrieved via the `getIntent()` method.

This code shows an example to open and show a String passed from one `Activity` to other via `Intent`:

``` java
public class MainActivity extends AppCompatActivity {

    /* Fields that will store our EditText and Button */
    private EditText mNameEntry;
    private Button mDoSomethingCoolButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Set widgets instances
        mDoSomethingCoolButton = (Button) findViewById(R.id.b_do_something_cool);
        mNameEntry = (EditText) findViewById(R.id.et_text_entry);

        /* Setting an OnClickListener allows us to do something when this button is clicked. */
        mDoSomethingCoolButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                // Retrieve the text from the EditText and store it in a variable
                String enteredText = mNameEntry.getText().toString();

                // Create the parameters for the intent
                Context context = MainActivity.this;
                Class destinationActivity = ChildActivity.class;

                Intent startChildActivityIntent = new Intent(context, destinationActivity);

                // Use the putExtra method to put the String from the EditText in the Intent
                startChildActivityIntent.putExtra(Intent.EXTRA_TEXT, enteredText);
 
                // Then we start the child activity
                startActivity(startChildActivityIntent);
            }
        });
    }
}
```

``` java
public class ChildActivity extends AppCompatActivity {

    /* Field to store our TextView */
    private TextView mDisplayText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_child);

        mDisplayText = (TextView) findViewById(R.id.tv_display);

        // Use the getIntent method to store the Intent that started this Activity in a variable
        Intent intent = getIntent();
        
        // Create an if statement to check if this Intent has the extra we passed from MainActivity
        if (intent.hasExtra(Intent.EXTRA_TEXT)) {
            // If the Intent contains the correct extra, retrieve the text
            String passedText = intent.getStringExtra(Intent.EXTRA_TEXT);
            // If the Intent contains the correct extra, use it to set the TextView text
            mDisplayText.setText(passedText);
        }
    }
```


### Implicit intent

<img src="https://github.com/fjoglar/android-dev-challenge/blob/master/assets/images/intent.png" width="400" align="right" hspace="10">

Implicit intents specify the action which should be performed and optionally data which provides content for the action. When we create an implicit intent, the Android system finds the appropriate component to start by comparing the contents of the intent to the *intent filters* declared in the manifest file of other apps on the device. If the intent matches an intent filter, the system starts that component and delivers it the Intent object. If multiple intent filters are compatible, the system displays a dialog so the user can pick which app to use.

An intent filter is an expression in an app's manifest file that specifies the type of intents that the component would like to receive. For instance, by declaring an intent filter for an activity, we make it possible for other apps to directly start our activity with a certain kind of intent. Likewise, if we do *not* declare any intent filters for an activity, then it can be started only with an explicit intent.

For example, if we have content that we want the user to share with other people, create an intent with the `ACTION_SEND` action and add extras that specify the content to share. When we call `startActivity()` with that intent, the user can pick an app through which to share the content.

``` java
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```

If multiple apps can respond to the intent and the user might want to use a different app each time, we should explicitly show a chooser dialog. The **chooser dialog asks the user to select which app to use for the action** (the user cannot select a default app for the action). To show the chooser, create an `Intent` using `createChooser()` and pass it to `startActivity()`.

``` java
Intent sendIntent = new Intent(Intent.ACTION_SEND);
// ...

// Always use string resources for UI text.
// This says something like "Share this photo with"
String title = getResources().getString(R.string.chooser_title);
// Create intent to show the chooser dialog
Intent chooser = Intent.createChooser(sendIntent, title);

// Verify the original intent will resolve to at least one activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

One of the most used implicit intents is for **sharing content** with other users and friends. So in order to provide this functionality to our app users we can make use of the provided `ShareCompat` class.

```java
private void shareText (String text) {
    // Create a String variable called mimeType and set it to "text/plain"
    String mimeType = "text/plain";
    // Create a title for the chooser window that will pop up
    String title = "Share with:";
    // Use ShareCompat.IntentBuilder to build the Intent and start the chooser
    ShareCompat.IntentBuilder
            .from(this)
            .setType(mimeType)
            .setChooserTitle(title)
            .setText(text)
            .startChooser();
}
 ```


### Understanding URIs

A Uniform Resource Identifier (URI) is a string of characters used to identify a resource.

Such identification enables interaction with representations of the resource over a network, typically the World Wide Web, using specific protocols.

A generic URI is of the form:

```
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
```

It comprises:

+ The **scheme**, consisting of a sequence of characters beginning with a letter and followed by any combination of letters, digits, plus (`+`), period (`.`), or hyphen (`-`). Although schemes are case-insensitive, the canonical form is lowercase and documents that specify schemes must do so with lowercase letters. It is followed by a colon (`:`).<br>
Examples of popular schemes include `http(s)`, `ftp`, `mailto`, `file`, `data`, and `irc`.
+ Two slashes (`//`): This is required by some schemes and not required by some others. When the authority component is absent, the path component cannot begin with two slashes.
+ An **authority** part, comprising:
  + An optional authentication section of a user name and password, separated by a colon, followed by an at symbol (`@`)
  + A "**host**", consisting of either a registered name (including but not limited to a hostname), or an IP address. IPv4 addresses must be in dot-decimal notation, and IPv6 addresses must be enclosed in brackets (`[ ]`).
  + An optional port number, separated from the hostname by a colon
+ A **path**, which contains data, usually organized in hierarchical form, that appears as a sequence of segments separated by slashes. Such a sequence may resemble or map exactly to a file system path, but does not always imply a relation to one. The path must begin with a single slash (`/`) if an authority part was present, and may also if one was not, but must not begin with a double slash. The path is always defined, though the defined path may be empty (zero length), therefore no trailing slash.
+ An optional **query**, separated from the preceding part by a question mark (`?`), containing a query string of non-hierarchical data. Its syntax is not well defined, but by convention is most often a sequence of attribute–value pairs separated by a delimiter.
+ An optional **fragment**, separated from the preceding part by a hash (`#`). The fragment contains a fragment identifier providing direction to a secondary resource, such as a section heading in an article identified by the remainder of the URI. When the primary resource is an HTML document, the fragment is often an id attribute of a specific element, and web browsers will scroll this element into view.

An example:

```
                    hierarchical part
        ┌───────────────────┴─────────────────────┐
                    authority               path
        ┌───────────────┴───────────────┐┌───┴────┐
  abc://username:password@example.com:123/path/data?key=value#fragid1
  └┬┘   └───────┬───────┘ └────┬────┘ └┬┘           └───┬───┘ └──┬──┘
scheme  user information     host     port            query   fragment

  urn:example:mammal:monotreme:echidna
  └┬┘ └──────────────┬───────────────┘
scheme              path
```


### Media Types

A media type (also **MIME** -Multipurpose Internet Mail Extensions- type and content type) is a two-part identifier for file formats and format contents transmitted on the Internet.

A media type is composed of a *type*, a *subtype*, and *optional parameters*. Media type consists of top-level type name and sub-type name, which is further structured into so-called "trees". Media types can optionally define companion data, known as parameters.

```
top-level type name / subtype name [ ; parameters ]
```

The currently registered top-level type names are: **application**, **audio**, **example**, **font**, **image**, **message**, **model**, **multipart**, **text**, **video**.

Common examples:
+ `application/xml`
+ `application/zip`
+ `application/pdf`
+ `application/msword (.doc)`
+ `audio/mpeg`
+ `text/html`
+ `text/plain`
+ `image/png`
+ `image/jpeg`
+ `image/gif`


### References
[Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html)<br>
[Common Intents](https://developer.android.com/guide/components/intents-common.html)<br>
[`Intent` reference](https://developer.android.com/reference/android/content/Intent.html)<br>
[`ShareCompat` reference](https://developer.android.com/reference/android/support/v4/app/ShareCompat.html)<br>
[Android Intents - Tutorial](http://www.vogella.com/tutorials/AndroidIntent/article.html) by Vogella



###### Note: the images of the headers used in this serie of articles are from Udacity's [Developing Android Apps Course](https://www.udacity.com/course/new-android-fundamentals--ud851)
