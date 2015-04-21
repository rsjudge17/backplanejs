## Deploying backplanejs with your application ##

To deploy backplanejs with your application, you'll need to unpack a copy of the distribution archive into a suitable location in your project. You can get a distribution from the [download page](http://code.google.com/p/backplanejs/downloads/list); place it into the root directory of your project and unpack it.

When you unpack the archive a directory is automatically created for backplanejs which has the version number of the library as part of its name. It's important to retain this version number in the URLs you use if you want to take advantage of efficient browser caching. Common practice is to set a very long expiry time on scripts like this, so that users of your application don't need to keep requesting the library.

With the library in the root directory it can be referred to from anywhere in your project, as follows:

```
<script src="/backplanejs-0-6-5-yui-2.8.0/backplane-min.js" type="text/javascript">/**/</script>
<link rel="stylesheet" href="/backplanejs-0-6-5-yui-2.8.0/assets/backplane-min.css" />
```

## Deploying backplanejs with Drupal ##

To use backplanejs with Drupal you will need to install the Drupal backplanejs module as well as a copy of the particular version of the library you want to use.

(The Drupal backplanejs module was originally created by [Alex Sansom](http://webBackplane.com/alex-sansom).)

### Installing the Drupal backplanejs module ###

The Drupal module is available on the [download page](http://code.google.com/p/backplanejs/downloads/list). Once the module has been downloaded, copy it to the modules directory and unpack it. The resulting directory should then be renamed to `backplanejs`. The result should be a directory as follows:
```
<drupal_home>/sites/all/modules/backplanejs
```
If you go to the module administration page:
```
[Administer >> Site building >> Modules]
```
the module should now show up. Check the check-box next to the name in order to enable it:

### Installing a backplanejs library ###

With the module installed the next step is to install a version of the backplanejs library. The steps to do this are exactly as for deploying backplanejs with your application, except the archive is placed into the module directory that you created in the previous step, before being unpacked.

Once unpacked you will have a copy of the library in a location something like this:
```
<drupal_home>/sites/all/modules/backplanejs/backplanejs-0-6-5-yui-2.8.0
```

### Enabling a particular version of backplanejs ###

The Drupal module allows more than one version of backplanejs to be installed -- just place as many versions as you like into the module directory.

To enable a particular version of the library for use in Drupal, go to:
```
[Administer >> Content management >> backplanejs library settings]
```
and choose 'Add version'. Once a version number has been added it will then be available for use in nodes.

This step merely makes a version of backplanejs available for use, but it doesn't actually add the library to any pages. You can choose to add the library to all nodes of a particular type, or to specific nodes.

### Adding backplanejs support to node types ###

The most common scenario will probably be to add the library to all pages of a particular type. You might create a new content type for XForms samples, for example.

New content types are created by going to:
```
[Administer >> Content types]
```
and then selecting 'Add content type'. Whilst adding the type, select the option 'backplanejs libraries', and choose the version number of the library that you'd like to use. (You can also add a version number to a content type that already exists.)

### Adding backplanejs support to specific nodes ###

The same options that are available when editing node types are also available when editing individual nodes.

### Ensuring Drupal doesn't break your forms ###

Drupal will almost certainly mess up your forms in two ways. The first is that it will add `<br>` tags to your content when it sees blank lines, and the second is that it will strip out XForms tags.

It's easy to fix this by adding a new input format. Go to:
```
[Administer >> Site configuration >> Input formats]
```
and choose 'Add input format'. Name it something like 'XForms' and choose which users will have the rights to use this format (you will almost certainly want to limit this). Then disable the 'HTML filter', 'Line break converter' and 'URL filter' options. You will then have this input format available when you edit nodes.

### Checking your Drupal configuration ###

To check that everything is set up correctly, create a new node and in the backplanejs options choose the library version you have installed. Then paste the following markup into the body:
```
<xf:trigger>
  <xf:label>Drink me!</xf:label>
  <xf:hint>Click here to drink.</xf:hint>
</xf:trigger>
```
Save the node and you should see a simple button on the page, with the label 'Drink me!', and there should be a tooltip with the text 'Click here to drink.'.

If you view the source you should see:
  * that the `html` element has namespace entries for `xf` and `ev`;
  * that a script called `backplanejs-min.js` has been included;
  * that a stylesheet called `backplanejs-min.css` has also been included;
  * that your XForms markup appears exactly as above, with no tags missing, and no extra `br` tags.