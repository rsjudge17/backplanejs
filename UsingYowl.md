## Introduction ##

This section outlines how to reference the backplanejs scripts as well as what functions to call to make use of the Notify library. A [full example](http://backplanejs.googlecode.com/hg/_samples/notify/test-yowl.html) is available in the repository which shows a number of the features. It has been tested in IE, Firefox, Chrome and Safari.

## Loading the backplanejs library ##

To load the backplanejs library, see the instructions in DeployingBackplanejs.

## Registering notifications with Yowl ##

The next step is to tell Yowl what notifications your application will be sending to it. This is done by calling the `register()` function with a list of notification 'names', and then using the same name when making the actual call later on. Since the idea is that user should be able to enable and disable the notifications as they choose, the call to `register()` will also indicate which of the notifications should be enabled from the start.

As well as listing notifications, we also need to tell Notify the name of our application, and optionally provide a graphic that will be used in any notification that doesn't provide its own.

A sample initialisation call is:

```
document.Yowl.register(
  "facebook",
  [ "Friend logged on", "Friend logged off", "Friend twittered", "Important announcement" ],
  [ 0, 1, 3, "Friend twittered" ],
  "facebook.gif"
);
```

Note that the list of notifications to be enabled (the third parameter), can refer to items in the full list (the second parameter) either using the item's name, or its index.

## Displaying a notification ##

Once a list of notifications has been registered then any number of calls can be made to display one of the registered notifications. Whether backplanejs will _actually_ display the notification or not will depend on whether the user has chosen to override the default settings provided in the call to `register()`. Similarly, _how_ the notification is displayed will depend on both the parameters set by the programmer in the notification call, and the values configured by the user.

A typical request would look like this:

```
document.Yowl.notify(
  "Friend logged on",
  friend.fullName + " has logged on",
  friend.foreName + " has just logged on, so why not talk to them.",
  "facebook",
  friend.image,
  false,
  0
);
```

This call asks Notify to display a notification called _Friend logged on_, which is the first in the list of notifications registered with the call we saw above. Provided that the user hasn't disabled the notification, then the second parameter will be used as the title of the message, and the third will be the actual message itself. The fourth parameter is the name of the application, and this is used to pick up the default style for the message and a default icon if needed. In this case an image is provided--in the fifth parameter--but it could have been set to `null`, which would have caused the default to be used.

The sixth parameter indicates whether the notification should be 'sticky' or not, which simply means that the user must click on the message before it will disappear. (Normal behaviour for a notification is to fade away after a set amount of time--itself configurable by the user.) Note that setting the 'sticky' value to `true` in a call to `notify()` still might not make the notification sticky if the user has overridden the value in their settings. (And of course the converse is true; a user can choose to make some notifications sticky if they want to be sure not to miss them, even if the programmer has set them to non-sticky.)

The final parameter indicates a priority level, ranging from "very low" through to "emergency". This is usually left at zero to indicate 'use the default', since the user is also able to override this value.

## See Also ##

  * RequirementsYowl
  * YowlDisplayStyleMsAgent
  * YowlDisplayStyleSystray
  * YowlPublicDisplayThemes