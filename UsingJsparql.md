## Introduction ##

_This section will be changing soon, but is still up-to-date for version 0.6.2._

jSPARQL is an object-based serialisation of SPARQL queries. It is not yet clear whether this syntax is necessary in the long term since ordinary SPARQL queries can be parsed into this object form. But since we haven't yet created this parser, we'll use it for now.

The `meta` object supports a number of ways to query:
  * `query2()`
  * `ask()`

## `query2()` ##

The main way to conduct a query is to use the `query2` method on `document.meta`:
```
var r = document.meta.query2({...});
```
The parameter to the method is an object which represents a jSPARQL query, as described below. `query2` returns a list of objects which have been dynamically constructed to fit the selection criteria.

## `ask()` ##

A commonly used form of query is to ask simply whether some particular triples exist, rather than to obtain those triples. This can be done with the `ask` method:
```
var b = document.meta.ask({...})["boolean"];
```
The parameter to the method is an object which represents a jSPARQL query, as described below.

## Selection criteria ##

The criteria comprises two parts; the first specifies the properties that the returned object should have (the `select` value), whilst the second part indicates the criteria for creating these objects (the `where` value):
```
{
  select: ...,
  where: ...
}
```

To indicate which items should be used to create the dynamic objects, use the `where` property. The `where` property takes an array of patterns to match. For example, to find all subjects that have an `rdf:type` of `foaf:Person`, we could use the following `where` pattern:
```
{
  select: ...,
  where: [
    { pattern: [ "?s", "http://www.w3.org/1999/02/22-rdf-syntax-ns#type", "http://xmlns.com/foaf/0.1/Person" ] }
  ]
}
```
Since collecting items that conform to a certain type is a common action, it's possible to abbreviate a predicate of `rdf:type` to `a`:
```
{
  select: ...,
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] }
  ]
}
```
Further items in the `where` array build up the query. For example, to obtain the names of all people in the triple store, we would do this:
```
{
  select: ...,
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
The query now looks for any triple that is of type `foaf:Person`, and then uses the subject of that triple to find the corresponding `foaf:name` property.

## Using variables ##

Variables can be indicated in patterns by using either a question mark or dollar sign, followed by the name of the variable. For example, the variable `s` can be referred to using either "?s" or "$s".

Variables play two roles. The first is to indicate the values to be returned from the query, and the second is to clarify how 'joins' should be constructed. In the previous example the "?s" variable appears in both of the patterns that are to be matched:
```
{
  select: ...,
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
This indicates that we only want values for "?s" (i.e., subjects) that have _both_ a type of `foaf:Person`, _and_ a property of `foaf:name`.

If we wanted a _union_ of the two lists (i.e., all people, as well as _anything_ that has a `foaf:name` property), then we could write this:
```
{
  select: ...,
  where: [
    { pattern: [ "?s1", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s2", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
The first pattern would give us all people, whilst the second would give us all names -- regardless of whether they came from people, companies, boats or cats.

## Specifying the returned properties ##

Returning to our previous query:
```
{
  select: ...,
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
we have two variables which represent two properties that are available to be placed on each JSON object that will be returned, `s` and `name`. If we want both properties to appear on the JSON objects, we would write:
```
{
  select: [ "s", "name" ],
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
However, it's possible that the `s` value is not going to be used for anything, and was added merely to get the 'joins' to work correctly. In that case there is no particular reason we need to include it in the output list, and we could just obtain a list of names:
```
{
  select: [ "name" ],
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
}
```
The complete example looks like this:
```
var r = document.meta.query2({
  select: [ "name" ],
  where: [
    { pattern: [ "?s", "a", "http://xmlns.com/foaf/0.1/Person" ] },
    { pattern: [ "?s", "http://xmlns.com/foaf/0.1/name", "?name" ] }
  ]
});
```

## Using the returned values ##

The `query2` method returns an object constructed as follows:
```
{
  head: {
    vars: [ ... ]
  },
  results: {
    ordered: false,
    distinct: false,
    bindings: [ ... ]
  }
}
```