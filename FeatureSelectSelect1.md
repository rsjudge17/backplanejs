## Introduction ##

In many Ajax libraries, selection lists are often regarded as menus, with the corresponding formatting and actions. In XForms, a `select1` with hierarchical entries could also be regarded as a menu.

### Basic menus ###

The YUI site has examples that show menus that are [expanded](http://developer.yahoo.com/yui/examples/menu/leftnavfrommarkup_source.html), [contracted](http://developer.yahoo.com/yui/examples/menu/topnavfrommarkup_source.html) and [contain images](http://developer.yahoo.com/yui/examples/menu/programsmenu_source.html).

### Button menus ###

In some situations the label on a `select1` doesn't add a great deal of value. In this case authors might want to use a 'button menu', which simply shows the selected value as the button caption. YUI has an example [here](http://developer.yahoo.com/yui/examples/button/btn_example12.html).

One way the YUI example might be marked up is:
```
<xf:select1 ref="x" appearance="xf:trigger">
  <xf:label>Select a category</xf:label>
  <xf:item>
    <xf:label>Category 1</xf:label>
    <xf:value>1</xf:value>
  </xf:item>
  <xf:item>
    <xf:label>Category 2</xf:label>
    <xf:value>2</xf:value>
  </xf:item>
  <xf:item>
    <xf:label>Category 3</xf:label>
    <xf:value>3</xf:value>
  </xf:item>
  <xf:item>
    <xf:label>A Really, Really Long Category 4</xf:label>
    <xf:value>4</xf:value>
  </xf:item>
</xf:select1>
```
The initial value of `xf:label` will be overwritten once the user makes a change, or if the value of node `x` becomes empty.

### Split buttons ###

With a _button menu_, clicking on the button part simply invokes the 'value changer' part. With a _split button_ clicking on the button part can invoke some action, and the 'value changer' is marked out more distinctly. YUI has a number of examples [here](http://developer.yahoo.com/yui/examples/button/btn_example08.html).

There are a number of ways that the first example on that page might be marked up, but one possibility is this:
```
<xf:select1 ref="x" appearance="ux:split-button">
  <xf:label>Split button 1</xf:label>
  <xf:action ev:event="DOMActivate">
    ...
  </xf:action>
  <xf:item>
    <xf:label>One</xf:label>
    <xf:value>1</xf:value>
  </xf:item>
  <xf:item>
    <xf:label>Two</xf:label>
    <xf:value>2</xf:value>
  </xf:item>
  <xf:item>
    <xf:label>Three</xf:label>
    <xf:value>3</xf:value>
  </xf:item>
</xf:select1>
```
This would require that the `DOMActivate` handler is observing the `xf:label`.

### Radial menus ###

Radial menus are a useful UI feature for a number of reasons:

  * They adhere to Fitt's Law, minimising the distance to each menu item from the context point;
  * Used consistently, they become an intuitive gesture system that can improve UX;
  * They look cool. :)

We added radial menus to an XForms project called [Hubbub](http://code.google.com/p/hubbub/) and they're actually pretty simple to get right. An image of a coloured circle on a transparent background is used for the menu itself, then menu items can be displayed positioned equidistant according to their number.

They could be abstracted as `select1[@appearance = "radial"]`.