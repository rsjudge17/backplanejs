## Introduction ##

The Message module comprises a generic message action that can display messages in a variety of ways, with four additional message types that provide a semantic instantiation of one of these types of message.

The workhorse of the message module is `message`. A message can be displayed in one of three different `modes` (called the `level`), and optionally has a title.

## Detail ##

The first step to creating a message is to create some mark-up that is structured like this (the `label` is optional):
```
<xf:message id="msg" level="modal">
  <xf:label>Title of the message</xf:label>
  Some text here.
</xf:message>
```

This doesn't display a message, but it creates something that is ready to be displayed. To actually display the message, the `DOMActivate` event should be dispatched to the message object:
```
var e = document.createEvent("DOMActivate");

e.init();
$('msg').dispatchEvent(e);
```
This can also be triggered by invoking the object's `activate()` method:
```
$('msg').activate();
```
which will also cause the event to fire.

To remove the message the `DOMActivate` event should be dispatched to the message object:
```
var e = document.createEvent("DOMActivate");

e.init();
$('msg').dispatchEvent(e);
```
This can also be triggered by invoking the object's `deactivate()` method:
```
$('msg').deactivate();
```
which will also cause the event to fire. As we'll see below, some message levels mean that the message will take care of removing itself.

_NOTE: It might be asked why this isn't the other way round, and that the method does the work, and the event handler merely calls the method. The problem is that there may be other parts of the code that have registered to receive notifications about these events, so whether the method is called, or the event is dispatched, we still need the event to be triggered._

### Message modes, or levels ###

The `level` attribute determines the behaviour and appearance of the message. There are four predefined values, and implementations are also free to add further values. The predefined values are the following:

#### inplace ####
This means that the message will appear wherever it is in the mark-up, and remain visible until the `DOMActivate` event is received a second time, with the value of '0'.

#### ephemeral ####
Unlike an inplace message, an ephemeral message will disappear after a certain amount of time. This type of message is most appropriate for information that is not crucial to the use of the application, such as a tooltip on a control.

#### modeless ####
A modeless message is one that requires the user's acknowledgment, but doesn't need to stop the functioning of the form.

#### modal ####
Similar to a modeless message in that the user needs to acknowledge the message, but different in that it is not possible to interact with the application until the message has been removed.


### Additional Semantics ###

`level` and `activate()` give authors full control over both when a message should appear, and its appearance and behaviour. However, there are some common message patterns that recur repeatedly in applications, and so they are given a shorthand in this module.

#### hint ####
The first is `hint` (sometimes called _tooltip_), which is an ephemeral message that is displayed in response to a user 'hovering' in some way over a control in an application. The message would usually be displayed next to the element that the user is hovering over. The hint element can use the `for` attribute to indicate which part of the document it should apply to:
```
<input id="sn" name="surname" />

<xf:hint for="sn">Please enter your name</xf:hint>
```
If there is no `for` attribute, the hint will apply to its parent element:
```
<div>
  <xf:hint>This is a hint</xf:hint>
</div>
```
hint is essentially a shorthand for an ephemeral message with no title, that will display on mouseover, and disappear after a certain amount of time. Such a message might be created in 'longhand' like this:
```
<input id="sn" name="surname" />

<xf:message id="hint" level="ephemeral">
  Please enter your name
</xf:message>
```
```
$("sn").onmouseover(
  function() {
    $("hint").activate();
    time(2, function() { $('hint').deactivate(); }
    return;
  }
);
```

#### alert ####
An alert message is an ephemeral message with no title, that is usually displayed in-place in the document when some kind of error condition becomes true. Unlike a hint, it is not usually removed until the error condition has cleared, although implementations of alert might choose to allow this.
```
<input id="work-phone" name="work-phone" />

<xf:alert id="alert" for="work-phone">
  The number should be #-###-###-####.
</xf:alert>
```
alert is essentially a shorthand for an in-place message with no title, that will display on some condition, and be removed when that condition is no longer applicable. Such a message might be created in 'longhand' like this:
```
<input id="work-phone" name="work-phone" />

<xf:message id="alert" level="inplace">
  The number should be #-###-###-####.
</xf:message>
```
```
$("work-phone").oninvalid(
  function() {
    $("alert").activate();
    return;
  }
);

$("work-phone").onvalid(
  function() {
    $("alert").deactivate();
    return;
  }
);
```

#### help ####
A help message is a modeless message with a title that is triggered by the user requesting help, for example by pressing `[F1]`:
```
<input id="zip" name="zipcode" />

<xf:help for="zip">A zipcode is a ....</xf:help>
```

### Using message ###

_Show an example where a form is validated in script, and an `xforms-invalid` event is dispatched to any control that is invalid. This would have the effect of triggering the alert since that would be listening for the event._