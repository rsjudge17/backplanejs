## Introduction ##

This tutorial doesn't require a great deal of RDFa knowledge, since much of it is simple cut-and-paste.

We begin with a simple web page into which we'll put some RDFa to indicate someone's Twitter account. Then we'll show how to load the RDFa parser so that it can process this information, and use a lens to act on the data -- initially setting a simple CSS class.

## Preparation ##

### Creating some test mark-up ###

The first thing you'll need is a simple HTML document that contains some RDFa, so create an empty document and paste in the following code (the `DOCTYPE` entry is for CSS purposes):

```
<!DOCTYPE XHTML>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>RDFa Twitter sample</title>
  </head>
  <body>
    <p xmlns:foaf="http://xmlns.com/foaf/0.1/">
      If you're interested in RDFa you might consider following
      Mark Birbeck (@<span
        about="#mark"
        rel="foaf:accountServiceHomepage" resource="http://www.twitter.com/"
        property="foaf:accountName"
        >markbirbeck</span>).
    </p>
  </body>
</html>
```

(I won't be offended if you insert your own Twitter account details into the mark-up.)

The mark-up we're using takes advantage of the Friend-of-a-Friend vocabulary, and if you'd like to read more about how to create a personal profile with FOAF, see [Getting started with RDFa: Creating a basic FOAF profile](http://webbackplane.com/mark-birbeck/blog/2009/04/getting-started-with-rdfa).

### Loading the RDFa Parser ###

The next step is to add the `script` tag that will load the RDFa parser. If you are deploying your own application then you'll want to add the backplanejs library as per the instructions in DeployingBackplanejs.

However, for the purposes of this tutorial you can reference a Google App Engine copy of the library, with the following code added to the `head` element:
```
<script src="http://backplanejs.appspot.com/backplanejs-0.6.2/backplane.js" type="text/javascript">/**/</script>
<link rel="stylesheet" href="http://backplanejs.appspot.com/backplanejs-0.6.2/assets/backplane.css" />
```

The RDFa parser loads automatically after the library has loaded, and then immediately searches the containing HTML document for any RDFa. Once the document has been processed and placed into a 'store', the RDFa parser then examines the store to see if there is any information about how to present that RDFa.

These presentation instructions are called 'lenses', so let's look at how to define one.

## Adding a lens ##

There are many ways to add a lens for the RDFa parser to process, depending on whether the lens is to be shared amongst many pages, and whether the pages and the lens will be served from servers in different domains.

We're going to start with a simple inline lens that will go in the page itself, so paste the following mark-up at the end of your document, just before the closing `body` tag:

```
<div
  xmlns:fresnel="http://www.w3.org/2004/09/fresnel#"
  typeof="fresnel:Group"
  style="display: none;"
>
  <div rev="fresnel:group">
    <div typeof="fresnel:Format">
      <div property="fresnel:instanceFormatDomain">
        select: [ "twittername" ],
        where:
        [
          { pattern: [ "?account", "http://xmlns.com/foaf/0.1/accountServiceHomepage", "http://www.twitter.com/" ] },
          { pattern: [ "?account", "http://xmlns.com/foaf/0.1/accountName", "?twittername" ], setUserData: true }
        ]
      </div>

      <span property="fresnel:resourceStyle" datatype="fresnel:styleClass">twitter-person</span>
    </div>
  </div>
</div>
```

A lens comprises a query to select some data to work with, and a set of actions to perform on that data. In this case the query locates all `foaf:accountName` properties that are associated with a Twitter account, and then sets the CSS class of the element that contains the name, to `twitter-person`.

### A quick introduction to jSPARQL ###

The query structure probably looks a bit obtuse, and it's explained in more detail in UsingJsparql; for now the key points to note are:
  * each `pattern` in the query comprises three parts -- an identifier, a property name and a value;
  * if the value begins with a question mark, then it acts as a variable that will take any value;
  * if one or more patterns contain the same variable then it's much like a SQL 'join' in that a match must now fulfil the criteria set by each pattern;
  * the `setUserData` attribute indicates that we want the query results to contain a pointer to the HTML element on which the particular pattern appeared -- in this case the account name.

Recall that our sample had the following mark-up:
```
<span
  about="#mark"
  rel="foaf:accountServiceHomepage" resource="http://www.twitter.com/"
  property="foaf:accountName"
  >markbirbeck</span>
```
This means we have an item that has two properties; the first is `foaf:accountServiceHomepage` and the second is `foaf:accountName`. You can think of this as being much like a JSON object with the following structure:
```
{
  accountServiceHomepage: "http://www.twitter.com/",
  accountName: "markbirbeck"
}
```
The query in the lens is looking for objects that match this pattern, except that the query has a variable in the position of `accountName`. This is because we want to find as many Twitter accounts as there are in the page.

### Acting on the query results ###

The second part of the lens describes what to do with the query results. There are many things that the library allows us to do, including retrieving more data, but in this example, all we want to do is to change the way that a Twitter account is displayed in the page.

To achieve this the lens uses the `resourceStyle` property, and gives it a value of `twitter-person`. Now any elements in the page that conform to the specified pattern will have their CSS class modified to include the `twitter-person` class.

The final step therefore is to add a value for this class to the `head` element:
```
<style type="text/css">
  .twitter-person {
    background-color: yellow;
  }
</style>
```

## Testing ##

If you've managed to follow along correctly then when you load your work into a browser you should see a simple page, with your Twitter name highlighted in yellow. To see how the process works when there are multiple entries, try cutting-and-pasting the entry, and changing the account name and `@about` to other values:
```
      Mark Birbeck (@<span
        about="#mark"
        rel="foaf:accountServiceHomepage" resource="http://www.twitter.com/"
        property="foaf:accountName"
        >markbirbeck</span>),
      Ben Adida (@<span
        about="#ben"
        rel="foaf:accountServiceHomepage" resource="http://www.twitter.com/"
        property="foaf:accountName"
        >benadida</span>) and
      Manu Sporny (@<span
        about="#manu"
        rel="foaf:accountServiceHomepage" resource="http://www.twitter.com/"
        property="foaf:accountName"
        >manusporny</span>).
```
and so on.

Finally, just to be certain you fully understand the mechanics, try making slight changes to the mark-up, such as changing property names and the value of `@resource`; any entries you edit in this way should no longer be yellow.

<a href='Hidden comment: 

<script src="formats/twitter.js" type="text/javascript">/**/

Unknown end tag for &lt;/script&gt;


<link rel="stylesheet" href="formats/twitter.css" />

'></a>