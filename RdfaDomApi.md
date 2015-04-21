**NOTE: This is work in progress...a scratchpad for keeping track of ideas relating to the W3C DOM API spec.**

# Introduction #

RDFa provides a means to attach properties to elements in XML and HTML documents. Since the purpose of these additional properties is to provide information about real-world items, such as people, films, companies, events, and so on, properties are grouped via a key or identifier into _property groups_.

The RDFa API provides a set of interfaces that make it easy to manipulate DOM objects that contain information that is also part of a property group. This specification looks at these interfaces.

A document that contains RDFa effectively provides two data layers. The first layer is the information about the document itself, such as the relationship between the elements, the value of its attributes, the origin of the document, and so on, and this information is usually provided by the Document Object Model, or DOM `[DOM]`.

The second data layer comprises information provided by embedded metadata, such as company names, film titles, ratings, and so on, and this is usually provided by RDFa `[RDFACORE]` or Microformats `[MICROFORMATS]`.

Whilst this embedded information _could_ be accessed via the usual DOM interfaces -- for example, by iterating through child elements and checking attribute values -- the potentially complex interrelationships between the data mean that it is more efficient for developers if they have access to the data _after_ it has been interpreted.

For example, a document may contain the name of a person in one section and the phone number of the same person in another; whilst the basic DOM interfaces provide access to these two pieces of information through normal navigation, it is more convenient for authors to have these two pieces of information available in one property collection, reflecting the final property group.

**NOTE: Flesh out this example so it's absolutely clear what a property group is.**

All of this is achieved through the RDFa DOM extensions API.

# Use cases #

**NOTE: Needs a lot of work! These examples are consciously geared towards high-profile uses of RDFa, such as Rich Snippets, SearchMonkey, Facebook, IMDB. However, there also needs to be some non-browser use-cases.**

## Event publishing ##

Amy has enriched her band's web-site to include Google Rich Snippets event information, which surfaces information for the search engine to use to enhance search results. She also adds a reference to some JavaScript that will automatically add a link to Google Calendar to her page.

Brian finds Amy's web-site through Google and opens the band's page. He decides that he wants to go to the next gig, and is easily able to add the details to his calendar by clicking on the link that has been automatically added.

## License information ##

Dale has a site that contains a number of images, showcasing his photography. He has already used RDFa to add licensing information about the images to his pages, following the instructions provided by Creative Commons. Now he would like the correct CC icon to appear on top of each image so that users of his site can quickly see which licenses apply to which items.

## Display of chemical compounds ##

Marie is a chemist, researching new kinds of blah de blah. She writes about her research on her blog, and often makes reference to chemical compounds. She would like any reference to these compounds to automatically have a picture of the compound's structure shown as a tooltip, and a link to the compound's entry on the National Center for Biotechnology Information `[NCBI]` web-site.

## Showing maps on a site ##

Richard has created a site that lists his favourite restaurants. Rather than generating additional data and code to show each location with Yahoo! Maps, he instead adds RDFa to each restaurant, and then uses the RDFa API to generate a map for all addresses on the page.

**NOTE: Weak...keep the principle but come up with a better scenario.**

# An overview of the API #

This API provides a number of interfaces to enable:
**parsing of DOM objects that contain embedded metadata;** extraction of the embedded metadata into a store;
**querying the store in order to act on the DOM objects referred to.**

## Using the API in a browser context ##

A common use of this API will be in a browser context, either via native support or through JavaScript libraries. This means that most authors will not need to construct stores, parsers and query objects -- at least for default RDFa processing -- since this can be done automatically.

### hasFeature ###

Developers can check for the presence of the RDFa API on a DOM object using the `hasFeature()` method on the `DOMImplementation` interface. (See `[DOM3CORE]`.)

### getElementsByType ###

Implementations that support this specification in the context of a DOM object must support the `getElementsByType()` method on the `document` object. This method allows authors to select all elements that have a particular `@typeof` value. For example, the following markup expresses a property group of type _Person_ in the Friend-of-a-Friend vocabulary `[FOAF]`:
```
<div typeof="foaf:Person">
  <span property="foaf:name">Albert Einstein</span>
</div>
```
To locate all elements that are associated with property groups that have a type of _Person_ we would do the following:
```
var ar = document.getElementsByType( "http://xmlns.com/foaf/0.1/Person" );
```
Note that all CURIEs and prefix-mappings used by the author in the initial source document are available for use in queries:
```
var ar = document.getElementsByType( "foaf:Person" );
```

### The meta object ###

The primary interface that authors will have access to is the Meta interface. This should be exposed via the `meta` property on the `document` object:
```
var store = document.meta.createStore();
```

## Using the API in other contexts ##

The standard sequence for using this API is to create a store, attach it to a parser and a query interface, and then initiate parsing to place the results into the store:
```
var meta = new Meta();
var store = meta.createStore();
var parser = meta.createParser("rdfa", store);
var query = meta.createQuery("rdfa", store);

parser.parse();
```
Note that the API supports different types of parser being created, as well as different types of query engine.

With reference to the previous section, a browser that supports RDFa parsing and RDFa querying by default must carry out all of these steps on document loading, and then expose the `getElementsByType` object via `document`. In other words, on loading a new document, the browser acts as if it has done the following:
```
var meta = new Meta();
var store = meta.createStore();
var query = meta.createQuery("rdfa", store);
var parser = meta.createParser("rdfa", store);

document.data = { };
document.data.meta = meta;
document.data.store = store;
document.data.query = query;
document.getElementsByType = function( t ) {
  return this.data.query.getElementsByType( t );
}

parser.parse();
```
An implementation is free to add additional steps to this sequence. For example, if a browser supports hCard parsing  then this could also be automatically be carried out in the initialisation phase:
```
.
.
.
parser = meta.createParser("rdfa", store);
parser.parse();
parser = meta.createParser("hcard", store);
parser.parse();
.
.
.
```

## Processing sequence ##

The basic operation of this API is to:
  1. create a Store object, which will hold information obtained from parsing;
  1. create a Parser object, passing it a pointer to a store;
  1. initiate parsing, to extract information from some object -- usually a DOM object -- and place it into the store;
  1. create a Query object which can be used to interrogate the information placed in the store;
  1. run queries against the store and act on the DOM objects returned.

Some platforms may merge one or more of these steps as a convenience to developers. For example, a browser that supports this API may carry out the first four steps when a document loads, and then expose a Query interface to allow developers to access the property groups. Some approaches to this will be discussed in the next section, but before we look at those, we'll give a brief overview of how each of these phases would normally be accomplished.

### Creating a Store object ###

To create a store the `createStore` method is called:
```
var meta = new Meta();
var store = meta.createStore();
```
The store object created supports the Store interfaces providing methods to add metadata to the store. These methods are used during parsing to populate the store but they can also be used directly to add additional information. Examples of this are shown later.

### Creating a Parser object ###

Once a store has been created, it can be linked to one or more parsers:
```
var parser = meta.createParser("rdfa", store);
```
Note that an implementation may support many types of parser, so the specific parser required needs to be specified. For example, an implementation may also support a Microformats hCard `[HCARD]` parser:
```
var parser = meta.createParser("hCard", store);
```
Implementations may also support different versions of a parser, for example:
```
var parser = meta.createParser("rdfa 1.0", store);
var parser = meta.createParser("rdfa 1.1", store);
```

**ISSUE** Probably should have a URI to identify parsers rather than a string, since not only are there many different Microformats, but also, people may end up wanting to add parsers for RDF/XML, different varieties of JSON, and so on. However, if we treat the parameter here as a CURIE, then we can avoid having long strings. If we do that, then the version number would need to be elided with the language type: "rdfa1.0", "rdfa1.1", and so on.

### Parsing a DOM object ###

Once we have a parser, we can use it to extract information from sources that contain embedded data. In the following example we extract data from the `Document` object:
```
parser.parse( document );
```
Since the parser is connected to a store, the property groups obtained from processing the document are now available in the variable `store`.

A store can be used more than once for parsing. For example, if we wanted to apply an hCard Microformat parser to the same document, and put the extracted data into the same store, we could do this:
```
var meta = document.meta;
var store = meta.createStore();

meta.createParser("rdfa", store).parse();
meta.createParser("hCard", store).parse();
```
The store will now contain property groups from the RDFa parsing, as well as property groups from the hCard parsing.

(If the developer wishes to reuse the store but clear it first, then the `Store.clear()` method can be used.)

**Diagram: Show the connection between a property group and the DOM.**

### Creating a Query object ###

Query objects are used to interrogate stores and obtain a list of DOM objects that are linked to property groups. Since there are a number of languages and techniques that can be used to express queries, we need to specify the type of query object that we'd like:
```
var query = meta.createQuery("rdfa", store);
```

### Selecting DOM objects ###

The features available via a Query object will depend on the implementation. However, all conforming processors will provide the basic element selection mechanisms described here.

**NOTE: This is intended to allow other specs to implement SPARQL queries, and such like.**

#### Selecting by type ####

Perhaps the most basic task is to select property groups of a particular type. The type of a property group is set in RDFa via the special attribute `@typeof`. For example, the following markup expresses a property group of type _Person_ in the Friend-of-a-Friend vocabulary `[FOAF]`:
```
<div typeof="foaf:Person">
  <span property="foaf:name">Albert Einstein</span>
</div>
```
To locate all property groups that are people we would do the following:
```
var query = meta.createQuery("rdfa", store);
var ar = query.select( { a: "foaf:Person" } );
```
Note that the Query object has access to the CURIEs and prefix-mappings that were used by the author in the initial source document, so they can also be used in queries. But it's also possible to write the same query in a way that is independent of any prefix-mappings:
```
var ar = query.select( { a: "http://xmlns.com/foaf/0.1/Person" } );
```

#### Selecting by property values ####

The previous query selected all property groups of a certain type, but it did so by indicating that the property `a` should have a specific value. Queries can also specify other properties. For example, given the following mark-up:
```
<div typeof="foaf:Person">
  <span property="foaf:name">Albert Einstein</span>
</div>
<div typeof="foaf:Person">
  <span property="foaf:name">Marie Curie</span>
</div>
```
the following query would select all property groups of type _Person_ that also have the the name property set to "Marie Curie":
```
var ar = query.select( {
  a: "http://xmlns.com/foaf/0.1/Person",
  "http://xmlns.com/foaf/0.1/name": "Marie Curie"
} );
```
As before, prefix-mappings can also be used:
```
var ar = query.select( {
  a: "foaf:Person",
  "foaf:name": "Marie Curie"
} );
```

### Using property groups ###

#### The Origin pointer ####

`Query.select()` returns a list of property groups that match the query. By default there is always a property called `origin` which refers back to an element in the source document -- usually the element that contains the `@typeof` declaration.

The `origin` property allows programmers to manipulate DOM objects based on the embedded metadata that they refer to. For example, to set a thin blue border on each _Person_, we could do this:
```
var ar = query.select( { a: "http://xmlns.com/foaf/0.1/Person" } );

for (var i = 0; i < ar.length; i++) {
  ar[ i ].origin.style.border = "1px solid blue";
}
```
Given the earlier markup, this code would add a border around each `div` that contains `@typeof="foaf:Person"`. However, we could also manipulate the `span` element that contains each person's name by passing a second parameter to the query, to indicate which 'origin' element we want:
```
var ar = query.select( { a: "foaf:Person" }, "foaf:name" );

for (var i = 0; i < ar.length; i++) {
  ar[ i ].origin.style.border = "1px solid green";
}
```

#### Obtaining extra properties ####

A query is not restricted to returning the source element for the RDFa; it can also return the embedded information that was represented in the source document.

For example, assume our source document contains the following event, marked up using the Google Rich Snippet Event format (example taken from the Rich Snippet tutorial, and slightly modified):
```
<div vocab="http://rdf.data-vocabulary.org/#" typeof="Event">
  <a href="http://www.example.com/events/spinaltap" rel="v:url" 
     property="summary">Spinal Tap</a>
  <img src="spinal_tap.jpg" rel="v:photo" />
  <span property="description">After their highly-publicized search for a new drummer, 
  Spinal Tap kicks off their latest comeback tour with a San Francisco show. </span>

  When:
  <span property="startDate" content="20091015T1900Z">Oct 15, 7:00PM</span>—
  <span property="endDate" content="20091015T2100Z">9:00PM</span>
</div>
```
To query for all _Event_ property groups we know that we can do this:
```
var ar = query.select( { a: "http://rdf.data-vocabulary.org/#Event" } );
```
However, to obtain the summary, start date and end date, we need only do this:
```
var ar = query.select( {
  a: "http://rdf.data-vocabulary.org/#Event",
  "http://rdf.data-vocabulary.org/#summary": "?summary",
  "http://rdf.data-vocabulary.org/#startDate": "?start",
  "http://rdf.data-vocabulary.org/#endDate": "?end"
} );
```
Any string that begins with "?" indicates that we'll accept any value in that position, and further, that the value should be exposed as a property in the property group. The name of the property is given by the string following the "?".

Exposing the embedded data in each property group makes it easy to create an HTML anchor that will allow users to add the event to their Google Calendar, as follows:
```
var anchor, button, i, pg;

for (i = 0; i < ar.length; i++) {
  // Get the property group:
  pg = ar[ i ];

  // Create the anchor
  anchor = document.createElement( "a" );

  // Point to Google Calendar
  anchor.href = "http://www.google.com/calendar/event?action=TEMPLATE"
    + "&text=" + pg.summary + "&dates=" + pg.start + "/" + pg.end;

  // Add the button
  button = document.createElement( "img" );
  button.src = "http://www.google.com/calendar/images/ext/gc_button6.gif";
  anchor.appendChild( button );

  // Add the link and button to the DOM object
  pg.origin.appendChild( anchor );
}
```
The result will be that the event has an HTML `a` element at the end (and any _Event_ on the page will follow this pattern):
```
<div vocab="http://rdf.data-vocabulary.org/#" typeof="Event">
  .
  .
  .
  <a href="http://www.google.com/calendar/event?action=TEMPLATE&text=Spinal+Tap&dates=20091015T1900Z/20091015T2100Z">
    <img src="http://www.google.com/calendar/images/ext/gc_button6.gif" />
  </a>
</div>
```
For more detailed information about queries see the Query interface.

# The API in detail #

## The Meta interface ##

The `meta` interface supports the following methods and properties:

### createStore ###

### createParser ###

### createQuery ###

## Stores ##

Data stores are created using the createStore() method. Stores are provided to parsers, which in turn use them to store the data collected during parsing:
```
var meta = new Meta();
var store = meta.createStore();
var parser = meta.createParser("rdfa", store);

parser.parse();
```

The `Store` interface supports the following methods and properties:

### add ###

The `Store.add` method adds a triple to a named graph in a store.

The following code snippet uses the Google geocoding API to add the latitude and longitude of an address to the default graph:
```
// Create a Google Maps geocoder object
var geocoder = new google.maps.Geocoder();

// Set up a function to geocode an address and add the lat/long to a store
function geocode( store, subj, address ) {
  geocoder.geocode( { 'address': address }, function(results, status) {
    if (status == google.maps.GeocoderStatus.OK) {
      store.add("default", subj, "http://www.w3.org/2003/01/geo/wgs84_pos#lat",
        results[0].geometry.location.lat());
      store.add("default", subj, "http://www.w3.org/2003/01/geo/wgs84_pos#long",
        results[0].geometry.location.long());
    }
  });
  return;
}
```
For a complete working through of this example, see Appendix A.

### clear ###

The `Store.clear` method clears all named graphs in a store.

## Parsers ##

A parser acts on a DOM object and extracts information which is placed into a store.

# Glossary #

## Source document ##

The source document is the document that the DOM object is based on, which is then parsed by the Parser.

## Document layer ##

## Data layer ##

## Parser ##

A parser is a processor that runs on a DOM object, interprets various parts of the markup in the DOM, and then places the interpretation in one or more graphs, in one or more data stores.

(The RDFa specification does not limit itself to operating over a DOM object, so in fact a parser could be created that runs over other input types. However, since we're dealing here with a DOM API, then it seems reasonable to limit ourselves to RDFa parsers that operate over DOM objects.)

## Graph ##

A graph is a collection of information that has been parsed from a DOM object. A Parser will always create at least one graph, called the _default graph_. However, it's possible to create others, and these will be identified by a URI.

## Data store ##

Once the parser has completed its processing of the DOM object, all of the data retrieved can be found in a graph within a data store. If numerous parsers act on a document then there may be numerous stores available to query.

## Property group ##

The result of querying against a store is a collection of one or more property objects. These objects contain at least one property, which is a pointer to the DOM element that contains the properties that matched the query. (How the properties match is explained in more detail below.)

Property groups will often contain more than one property, and these additional properties will have been indicated in the query.

# Appendix A: Full examples #

All of the following examples assume that the document has been parsed, and that all the information has been placed in a store, as follows:
```
document.data = { };
document.data.meta = new Meta();
document.data.store = meta.createStore();
document.data.query = document.data.meta.createQuery("rdfa", document.data.store);
document.data.parser = document.data.meta.createParser("rdfa", document.data.store);

document.getElementsByType = function( t ) {
  return this.data.query.getElementsByType( t );
}

document.data.parse();
```

## Geocode Google Rich Snippets Address ##

```
var store = document.data.store;
var query = document.data.query;

var anchor, ar, button, geocoder, i, pg;

// Create a Google Maps geocoder object
geocoder = new google.maps.Geocoder();

// Set up a function to geocode an address and add the lat/long to a store
function geocode( store, subj, address ) {
  geocoder.geocode( { 'address': address }, function(results, status) {
    if (status == google.maps.GeocoderStatus.OK) {
      store.add("default", subj, "a", "http://rdf.data-vocabulary.org/#Geo");
      store.add("default", subj, "http://rdf.data-vocabulary.org/#latitude",
        results[0].geometry.location.lat());
      store.add("default", subj, "http://rdf.data-vocabulary.org/#longitude",
        results[0].geometry.location.long());
    }
  });
  return;
}

// Select all addresses
ar = query.select( {
  a: "http://rdf.data-vocabulary.org/#Address",
  "http://rdf.data-vocabulary.org/#street-address": "?street",
  "http://rdf.data-vocabulary.org/#locality": "?locality",
  "http://rdf.data-vocabulary.org/#region": "?region",
  "http://rdf.data-vocabulary.org/#postal-code": "?zip",
  "http://rdf.data-vocabulary.org/#country-name": "?country"
} );

for (i = 0; i < ar.length; i++) {
  // Get the property group:
  pg = ar[ i ];

  // Geocode the address and add result to store
  geocode( store, pg.subject, pg.street,
    + ", " + pg.locality
    + ", " + pg.region
    + ", " + pg.zip
    + ", " + pg.country;
}
```

# IDEAS THAT STILL NEED INTEGRATING #

## Events ##

RDFa parsing can begin either after the DOM document is fully loaded, or whilst the document is still loading. Since there are a number of factors that might influence when the data is actually available to authors, the RDFa API provides notifications about the different stages of the lifecylce. Authors can register for these notifications, and then act accordingly.

### meta-data-ready ###

NOTE: The name is intended to mimic the usual 'data ready' events.

The `meta-data-ready` event is dispatched to all listeners when parsing has been completed for each store. This means that if a DOM supports multiple parsers, there will be more than one occurrence of the `meta-data-ready` event.

The `Event` object contains a pointer to the relevant store:
```
MetaEvent : Event {
  MetaStore store;
};
```

NOTE: Again, this is all standard layout stuff for specs with events in, so we can copy this when we're ready.

### Datatypes ###

The data returned uses language-specific data types.

### Index references ###

If an item in the array refers to another item in the array, this will be indicated with an index.
```
var ar = query.select(
  {
    "http://xmlns.com/foaf/0.1/name": "?name",
    "http://xmlns.com/foaf/0.1/knows": "?knows"
  }
);
```
The result would be
```
ar = [
  {
    "name": "Mark Birbeck",
    "knows": 1
  },
  {
    "name": "Benjamin",
    "knows: 0
  }
];
```

## Formatters ##

The DOM API supports formatters which can be applied to particular data. For example, to apply a CSS class to all organisations on a page, first select the organisations:
```
var ar = query.select({ a: "vcard-org" });
```
then apply the chosen CSS class to each:
```
document.meta.format(
  ar,
  {
    "resourceStyle": "org"
  }
);
```

## Tokens ##

The following default tokens are defined:

| **Token** | **Full URI** |
|:----------|:-------------|
| resourceStyle | http://www.w3.org/2004/09/fresnel#resourceStyle |

They can be modified using `document.meta.token[...]`.

<a href='Hidden comment: 
== Bootsrapping/Performance/Not sure ==

Since RDFa parsing over large documents can consume resources, it is not always necessary to carry out parsing on every document.

Instead, programmers should register for one of the document loaded or document ready events, and then initiate parsing. For example, if using the YUI library, an author might do this:
```
var store = document.meta.createStore();

YAHOO.util.Event.onDOMReady(
  function () {
    var parser = document.meta.createParser("rdfa", store);

    parser.parse();
  }
);
```

The data obtained from processing the document is now available in the variable store.
'></a>

<a href='Hidden comment: 
There is nothing to prevent an implementation from parsing the entire document automatically, in which case an author need not call the parse() method explicitly. Since all stores that are connected to a DOM object are referenced in the store array, the store would be obtained as follows:
```
YAHOO.util.Event.onDOMReady(
  function () {
    var store = document.meta.store[ 0 ];
```
Check the documentation of your chosen platform or library for more details on this.
'></a>