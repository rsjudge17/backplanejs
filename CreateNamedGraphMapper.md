## Introduction ##

A named-graph mapper provides information to the library about how to process data from non-SPARQL services. This allows the library to handle all of the steps for retrieving data and storing it into the triple store, rather than the author having to hand-code this. Instead, the author can simply create a SPARQL query that refers to the non-SPARQL service.

## Using a non-SPARQL service in a SPARQL query ##

The mechanism for retrieving the data is the same as for SPARQL end-points -- using named graphs. Rather than making a request to the service, processing the returned data, adding it to the triple store, and then querying for that data, an author can simply write a SPARQL query that uses a named graph.

For example, to obtain Twitter updates for a particular person, an author simply needs to write the following SPARQL query:
```
SELECT ?text ?person ?depiction
FROM NAMED <http://www.twitter.com/manusporny>
WHERE {
  ?s a <http://www.twitter.com/status> .
  ?s <http://www.twitter.com/text> ?text .
  ?s <http://www.twitter.com/saidby> ?person .
  ?person <http://xmlns.com/foaf/0.1/depiction> ?depiction
}
```

## Setting up a mapper ##

Mappers are defined by inserting triples into a named graph, called `about-graphs`. The triples consist of:

### Subject ###

The subject of the triples is an identifier for the mapper. It won't be the same as the URL of the service, and shouldn't conflict with other mappers.

### matches ###

The `matches` predicate provides a regular expression that is used to test named-graph URIs. If the URI matches the expression, the mapper will be used to control the service request, and the processing of any returned data.

For our Twitter example, the regular expression would be:
```
/^http\:\/\/www.twitter.com\/(.+)/
```
This matches any URL that begins with "http://www.twitter.com/", and stores the value after the slash ready for further use. In our example, that is "manusporny".

### uri ###

The `uri` predicate provides information on how to construct a URI.

The [Twitter API](http://apiwiki.twitter.com/) describes how [Tweets for a particular person](http://apiwiki.twitter.com/Twitter-REST-API-Method:-statuses-user_timeline) can be obtained -- using URLs that contain the Twitter name. The URL required by our Twitter example would be:
```
http://www.twitter.com/statuses/user_timeline/manusporny.json
```
In order to get from our named-graph of "http://www.twitter.com/manusporny" to this URL, we set the URI predicate to the following value:
```
http://www.twitter.com/statuses/user_timeline/%s.json
```
This tells the mapper to use the value extracted by the `matches` predicate, in place of the "%s" in the `uri` predicate.

### params ###

The `params` predicate is not yet implemented.

### adddata ###

The `adddata` predicate contains a callback that will be executed once data has been received from the service. The function usually contains a series of calls to the `store.insert()` method, in order to add the data returned, as a set of triples.

A great deal of data is returned by Twitter, but to illustrate, an individual post looks something like this:
```
{
  "in_reply_to_user_id":4099141,
  "in_reply_to_status_id":null,
  "truncated":false,
  "source":"<a href=\"http://www.tweetdeck.com/\" rel=\"nofollow\">TweetDeck</a>",
  "favorited":false,
  "geo":null,
  "user":{
    "profile_sidebar_fill_color":"F3F3F3",
    "followers_count":230,
    "description":"Founder/CEO of Digital Bazaar, Inc. - Legal P2P Content Distribution (music, movies, books)",
    "screen_name":"manusporny",
    "following":null,
    "time_zone":"Central Time (US & Canada)",
    "friends_count":76,
    "profile_sidebar_border_color":"DFDFDF",
    "geo_enabled":false,
    "notifications":null,
    "profile_text_color":"333333",
    "url":"http://blog.digitalbazaar.com/",
    "verified":false,
    "statuses_count":445,
    "profile_background_image_url":"http://s.twimg.com/a/1259091217/images/themes/theme7/bg.gif",
    "profile_link_color":"990000",
    "protected":false,
    "profile_background_tile":false,
    "created_at":"Mon Feb 09 16:33:16 +0000 2009",
    "location":"Blacksburg, VA",
    "name":"Manu Sporny",
    "profile_background_color":"EBEBEB",
    "profile_image_url":"http://a1.twimg.com/profile_images/287153318/professional-iran_normal.png",
    "id":20446311,
    "utc_offset":-21600,
    "favourites_count":0
  },
  "in_reply_to_screen_name":"_masaka",
  "created_at":"Thu Nov 26 04:39:08 +0000 2009",
  "id":6072310071,
  "text":"@_masaka Integrated your tests into RDFa Test Suite (TC172-173), will fix librdfa/Raptor by Nov. 30th: http://bit.ly/5Nc5A5"
}
```

The key pieces of information we need to satisfy the SPARQL query, are the name of the person Twittering, what they said, and their picture. In the JSON object returned from the service, they are `user.screen_name`, `text`, and `user.profile_image_url`.

## A Full Example ##

The following is a Twitter-mapper for obtaining statuses about a user:

```
document.meta.store.insert([{
  name: "about-graphs",
  "$": "http://www.twitter.com/",
  "http://argot-hub.googlecode.com/matches": /^http\:\/\/www.twitter.com\/(.+)/,
  "http://argot-hub.googlecode.com/uri": "http://www.twitter.com/statuses/user_timeline/%s.json",
  "http://argot-hub.googlecode.com/params": {
    "http://argot-hub.googlecode.com/callbackParamName": "callback",
    "http://argot-hub.googlecode.com/count": "2"
  },
  "http://argot-hub.googlecode.com/adddata": function(context) {
    for (var i = 0; i != context.data.length; i++) {
      var item = context.data[i],
          uriStatus = "http://www.twitter.com/" + item.user.screen_name + "/statuses/" + item.id,
          uriPerson = "http://www.twitter.com/" + item.user.screen_name;

      document.meta.store.insert([
        {
          name: uriPerson,
          "$": uriStatus,
            "a": "<http://www.twitter.com/status>",
            "http://www.twitter.com/text": item.text,
            "http://www.twitter.com/saidby": "<" + uriPerson + ">"
        }
      ]);

       document.meta.store.insert([
         {
           name: uriPerson,
           "$": uriPerson,
             "http://xmlns.com/foaf/0.1/depiction": "<" + item.user.profile_image_url + ">",
             "http://xmlns.com/foaf/0.1/name": item.user.name
         }
       ]);
    }
  } //adddata()
}]);
```