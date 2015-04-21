Version 0.9.2

## Introduction ##

RDFj is a set of conventions for:
  * constructing JSON objects in such a way that they can easily be interpreted as RDF;
  * taking RDF and arriving at canonical JSON objects.

The name derives from the fact that it is a very close relative of RDFa. It came about as a result of needing a way to import external RDFa documents from a different domain to the host document, something that is impossible in a browser-based parser. It then evolved into something that could be used more broadly in JavaScript programming.

## Approach ##

The main RDF-related JSON format is [RDF/JSON](http://n2.talis.com/wiki/RDF_JSON_Specification) at Talis. This format provides a useful JSON serialisation of RDF for use in RDF applications that want to export RDF to applications that understand JSON.

However, for applications that primarily use JSON, but also want to use the power of RDF, RDF/JSON is not ideal. For example, RDFj allows JSON objects to be part of the graph, which is particularly useful when functions are part of the JSON object.

## Details ##

### JSON objects and RDF blank nodes ###

To illustrate this approach, let's take a simple JSON object containing properties with values:

```
{
  "name": "Anna Wilder",
  "homepage": "http://example.org/about"
}
```

In RDF parlance this information could be represented as two triples, both associated with the same subject. Of course we don't know what the subject is, but RDF has a built-in mechanism to cope with that; unknown subjects can be represented with a 'blank node', or _bnode_.

If we were to write the two triples down, they might look like this:

```
_:abc <name> "Anna Wilder" .
_:abc <homepage> "http://example.org/about" .
```

or more compactly:

```
_:abc
  <name> "Anna Wilder" ;
  <homepage> "http://example.org/about"
  .
```

Although this mapping is very simple, it illustrates the core of the RDFj approach, which is that the JSON remains natural-looking, even though it is representing RDF.

### URIs versus literals ###

Of course, if we look closely at the RDF we've generated, we'll see that Anna's home-page is represented by a string of text, rather than a URI. What we actually want is this:

```
_:abc
  <name> "Anna Wilder" ;
  <homepage> "<http://example.org/about>"
  .
```

RDFj achieves this by regarding any string of text that begins with '<' and ends with '>' as a URI:

```
{
  "name": "Anna Wilder",
  "homepage": "<http://example.org/about>"
}
```

This makes the JSON more compact than if we had used an additional property to indicate the type of the information.

### Literals ###

#### Language ####

Strings of text sometimes need a language specifier to indicate which language they use. This is particularly important if different translations of the same string are provided.

To indicate the language of a string, simply append '@' followed by the language code. For example, we might have two names for a person, in different languages (we'll meet arrays in a moment):

```
{
   "name": [
     "Ivan Herman",
     "Herman Iván@hu"
   ]
}
```

This means that there are two `name` predicates, one with a value of "Ivan Herman" and the other with a value of "Herman Iván", the latter being in Hungarian.

#### Datatype ####

A string of text can also have a datatype, which will be indicated by a URI. For example, to indicate that a string is of type date, we could use the XSD data type `xsd:date`, as follows:

```
{
  "name": "Anna Wilder",
  "homepage": "<http://example.org/about>"
  "birthday": "1970-04-01^^http://www.w3.org/2001/XMLSchema-datatypes#date"
}
```

The URI for the datatype (in this case `http://www.w3.org/2001/XMLSchema-datatypes#date`) can be abbreviated using any of the techniques for shortening URIs, that are discussed below.

### Predicates as URIs ###

In most RDF usage the predicates will be from identifiable vocabularies, which means that their values will need to be identified with full URIs. In our example we're actually using FOAF properties, so the properties we are using are `http://xmlns.com/foaf/0.1/name` and `http://xmlns.com/foaf/0.1/homepage`.

JSON allows the names of properties to be anything we like, so we can actually use these URIs directly in our objects. However, naming a property with anything more complicated than a standard variable name will require the name to be quoted:

```
{
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

**Note:** Starting and ending the predicate URI with '<' and '>' is optional, since predicates are always URIs.

This maps to:

```
_:abc
  <http://xmlns.com/foaf/0.1/name> "Anna Wilder" ;
  <http://xmlns.com/foaf/0.1/homepage> <http://example.org/about>
  .
```

### Special properties ###

In addition to the properties set using the URI technique just described, there are a few special properties that RDFj reserves for itself. These are '$' to set the subject, 'a' to indicate a type, 'context' to indicate context information, and 'graph' to indicate a graph.

#### Using '$' to set the subject ####

So far our JSON objects have been 'anonymous' in the RDF sense, but much of the time we'll need to give our object an identifier. This can be done with the special property '$'. For example, to indicate the title of Anna's home-page, we might use the Dublin Core `title` property, like this:

```
{
  "$": "<http://example.org/about>",
    "http://purl.org/dc/elements/1.1/title": "Anna's Homepage"
}
```

The triples represented by this JSON object would be:

```
<http://example.org/about>
  <http://purl.org/dc/elements/1.1/title> "Anna's Homepage"
  .
```

**Note:** This might change, since '$' is not only used in SPARQL queries, but has a special status in some Ajax libraries.

#### Using 'context' to set the context ####

The context object provides a further set of properties, which are: 'token' to set a list of token mappings and 'base' to indicate a base URI:

```
{
  "context": {
    "base": ...,
    "token": ...
  }
  "name": "Anna Wilder",
  "homepage": "<http://example.org/about>"
}
```

Setting a context will apply to any nested content, too. For example, when using the 'graph' property (described below), a context can be added to the embedded graph, or to the object containing the graph. Any values set in an embedded graph will override values from higher up.

##### Using 'token' to set a list of token mappings #####

Our initial JSON object looked like this:

```
{
  "name": "Anna Wilder",
  "homepage": "<http://example.org/about>"
}
```

However, to clarify exactly which vocabulary the predicates came from, we used the following as the RDFj representation:

```
{
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

With large amounts of data, these predicates can become cumbersome, and in many situations it is easier to manipulate the values using the shorter strings. To make this possible, the `context` object includes the 'token' property, which contains a list of token mappings:

```
{
  "context": {
    "token": {
      "name": "http://xmlns.com/foaf/0.1/name",
      "homepage": "http://xmlns.com/foaf/0.1/homepage"
    }
  }
  "name": "Anna Wilder",
  "homepage": "<http://example.org/about>"
}
```

(For more on the idea of tokens, see [Tokenising the semantic web](http://webbackplane.com/mark-birbeck/blog/2009/04/30/tokenising-the-semantic-web).)

#### Using 'base' to set a base URL ####

In situations where the context of an RDFj document is not known or has changed from its original location, it is not possible to use relative paths for URIs. However, using relative paths can make the RDFj more compact, and for this reason the 'base' property can be used to set a URI against which relative paths are resolved:

```
{
  "context": {
    "base": "<http://example.org/about>",
    "token": {
      "title": "http://xmlns.com/foaf/0.1/title",
      "maker": "http://xmlns.com/foaf/0.1/maker",
      "name": "http://xmlns.com/foaf/0.1/name",
      "homepage": "http://xmlns.com/foaf/0.1/homepage"
    }
  },
  "$": "<>",
    "title": "Anna's Homepage",
    "maker": {
      "name": "Anna Wilder",
      "homepage": "<>"
    }
}
```

**Note:** This example uses the 'nested object' technique described below.

#### Using 'a' to set the type ####

A common property to set is the RDF 'type', and a typical use would be to indicate that Anna is a 'person'. It's possible to set this using the following syntax:

```
{
  "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", "<http://xmlns.com/foaf/0.1/Person>",
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

However, RDFj provides a more abbreviated form:

```
{
  "a", "<http://xmlns.com/foaf/0.1/Person>",
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

Both approaches would generate the following triples:

```
_:
  a <http://xmlns.com/foaf/0.1/Person> ;
  <http://xmlns.com/foaf/0.1/name> "Anna Wilder" ;
  <http://xmlns.com/foaf/0.1/homepage> <http://example.org/about>
  .
```

Note that 'a' is simply a default token, essentially providing a built-in shorthand for:

```
{
  "context": {
    "token": {
      "a": "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
    }
  },
  "a", "<http://xmlns.com/foaf/0.1/Person>",
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

#### Using 'graph' to embed RDFj in other JSON objects ####

So far all of our examples have shown how every part of a JSON object maps to RDF. However, there will be situations where a JSON object contains other properties that are not to be interpreted as RDF. The `graph` property can be used in a JSON object to indicate another object that can be treated as RDFj:

```
{
  "ref": 73,
  "order": false,
  "graph": {
    "a", "<http://xmlns.com/foaf/0.1/Person>",
    "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
    "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
  }
}
```

When used in this way, all other properties of the JSON object are ignored except for '$' and 'context'. When '$' is present it provides an identifier for the RDFj, effectively creating a _named graph_:

```
{
  "$": "graph1",
  "graph": {
    "a", "<http://xmlns.com/foaf/0.1/Person>",
    "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
    "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
  }
}
```

### Referencing anonymous JSON objects ###

There are often situations where we want to reference bnodes. For example, to say that Anna's site was made by Anna, we can use the FOAF `maker` property. A typical set of triples would be these:

```
<http://example.org/about>
  <http://purl.org/dc/elements/1.1/title> "Anna's Homepage" ;
  <http://xmlns.com/foaf/0.1/maker> _:abc
  .

_:abc
  <http://xmlns.com/foaf/0.1/name> "Anna Wilder" ;
  <http://xmlns.com/foaf/0.1/homepage> <http://example.org/about>
  .
```

To achieve this set of triples in JSON we have two choices. The first is to simply create a name for the anonymous JSON object, using the special '$' property:

```
{
  "$": "<bnode:abc>",
    "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
    "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

We can now refer to this JSON object in another JSON object that represents the home-page:

```
{
  "$": "<http://example.org/about>",
    "http://purl.org/dc/elements/1.1/title": "Anna's Homepage",
    "http://xmlns.com/foaf/0.1/maker>": "<bnode:abc>"
}
```

The second approach is to keep the home-page object anonymous, but to set this JSON object as the actual value of the predicate:

```
{
  "$": "<http://example.org/about>",
    "http://purl.org/dc/elements/1.1/title": "Anna's Homepage",
    "http://xmlns.com/foaf/0.1/maker>": {
      "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
      "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
    }
}
```

Whilst the first approach requires two separate JSON objects, this second only requires one. When converted to RDF there will still be a bnode that connects the two objects, but this will be automatically set to be the same for both objects. (This is much the same as the idea of [chaining](http://www.w3.org/TR/rdfa-syntax/#sec_5.3.) in RDFa.)

This technique shows once again how we're always trying to make the JSON look as 'natural' as possible -- in this case using JSON objects directly.

### Arrays ###

Any item can be represented either with a direct value or an array of values. For example, to represent the two nicknames that Anna uses, we might use the following JSON:

```
{
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/nick": [ "wildling", "wilda" ],
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
}
```

This would give us these triples:

```
_:abc
  <http://xmlns.com/foaf/0.1/name> "Anna Wilder" ;
  <http://xmlns.com/foaf/0.1/nick> "wilding" ;
  <http://xmlns.com/foaf/0.1/nick> "wilda" ;
  <http://xmlns.com/foaf/0.1/homepage> <http://example.org/about>
  .
```

An array can also be the top-level object; a collection of RDFj objects can be represented with an array:

```
[
  {
    "$": "<http://example.org/about>",
      "http://purl.org/dc/elements/1.1/title": "Anna's Homepage",
      "http://xmlns.com/foaf/0.1/maker>": "<bnode:anna+wilder>"
  },
  {
    "$": "<bnode:anna+wilder>",
      "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
      "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
  }
]
```

**Note:** Substituting an array of graphs for a graph can also be done when using the 'graph' special property:

```
{
  "graph": [
    {
      "$": "<http://example.org/about>",
        "http://purl.org/dc/elements/1.1/title": "Anna's Homepage",
        "http://xmlns.com/foaf/0.1/maker>": "<bnode:anna+wilder>"
    },
    {
      "$": "<bnode:anna+wilder>",
        "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
        "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
    }
  ]
}
```

### JSON objects as first-class citizens ###

Many JSON objects can be used directly in RDFj, without requiring the use of explicit datatypes. For example, to set a literal that represents an integer, we simply use the integer directly:

```
{
  "http://xmlns.com/foaf/0.1/name": "Anna Wilder",
  "http://xmlns.com/foaf/0.1/nick": [ "wildling", "wilda" ],
  "http://xmlns.com/foaf/0.1/homepage": "<http://example.org/about>"
  "http://xmlns.com/foaf/0.1/age": 39
}
```

Booleans can also be used in this way. Note that arrays and objects are dealt with specially, as described above. In addition, although functions and regular expressions are processed correctly within the library, they are not valid JSON, and so should be used in RDFj with caution.