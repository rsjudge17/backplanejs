# Basic layer #

The Ajax layer of Notify simply uses the Yahoo User Interface library (YUI) to render notifications. Since this layer requires no components to be installed then user control is limited to the 'current' application.

# User control across applications #

If the user has Google Gears installed their settings can be saved across web applications.

# User control across computers #

One goal of the project is to allow users to save their settings 'in the cloud', so that they can be used whenever they run the same web application from other computers. It's also a goal that various themes for messages should be available from a central location.

## Advanced messaging ##

If Mac OS users have Growl installed then notifications from Yowl will automatically be sent to it. On Windows it is similarly possible to display messages outside of the browser using the [backplanejs System-tray Component](YowlDisplayStyleSystray.md). Also on Windows, if Microsoft Agent is detected, [speech capabilities can be used](YowlDisplayStyleMsAgent.md).

Since there is only one interface to Notify, the programmer does not need to know what capabilities a user's system has--they simply make calls against the Notify API.

## Read more ##

  * UsingYowl
  * [Try it now](http://backplanejs.googlecode.com/hg/_samples/notify/test-yowl.html)