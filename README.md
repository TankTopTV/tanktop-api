# Tank Top TV API

Welcome to the Tank Top TV APIs.  These are free to use for non-commercial use within reasonable usage limits, but please get in touch with us at <hello@tanktop.tv> if you'd like to use them for commercial purposes.

1. [Movie and TV matching API](#Matching)
2. [Movie and TV sources](#Sources)
3. [TV and Movie Source Ingestion](#Ingestion)
4. [EPG Ingestion](#EPG)
5. [Listing Movies and TV](#ItemLists)
6. [TV and Movie data](#ItemData)
7. [User Profiles](#UserProfiles)
8. [User Actions and Events](#UserActions)
9. [Personalised Movie & TV lists](#PersonalizedLists)
10. [Notifications](#Notifications)
11. [Recommendations](#Recommendations)
12. [Url lookup](#URLLookup)
13. [Popular](#Popular)
14. [Editorial Collections](#Collections)


We're still in the process of documenting our APIs but please get in touch if you're interested in using an endpoint that is not fully written-up yet.

## API usage

You will need an API key to use the API.  To get one sign in to your Tank Top account at http://tanktop.tv then go to http://tanktop.tv/account/settings#apps and create an app.  You will need to agree to our [terms of service](http://tanktop.tv/apiterms) and provide us with a little detail about what you are building.

The API is hosted at http://api.tanktop.tv.  Data is returned in JSON format.

If you have any problems please contact us at <hello@tanktop.tv>.

## <a name="Matching"></a> Movie and TV matching API
Use this API to match items in your catalog against the TankTop entertainment graph.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/match/<itemtype>/`| Get matches for a TV programme or movie|

Valid itemtypes are `movie` and `programme`.

#### Parameters
| Name     | Value  | Required |
|:--       |:--     | :--:     |
| title    | Programme Title | y |
| year     | Year of first release | n |
| director | Director (Movies only)  | n |
| cast     | Cast members.  Any cast members you include help narrow down the match. Repeat the parameter to specify multiple cast members | n |

#### Return values

| Name | Description |
|:-----|:------------|
| match_type | M, N or C.  M means there is a good match, N means there's no good match.  C means there are multiple good looking matches, but its too close to say which is correct |

If match_type is M the response contains the following additional fields

| Name | Description |
|:-----|:------------|
| title | title |
| director | Director - movies only |
| year | Year of first release |
| tt_id | TankTop ID for the item |
| rt_id | Rotten Tomatoes ID for the item (if available) |
| imdb_id | IMDB ID for the item (if available) |

If match_type is N or C then the response contains the following additional fields

| Name | Description |
|:-----|:------------|
| candidates| An array of candidate matches, best first |

The array contains elements with the following fields

| Name | Description |
|:-----|:------------|
| score | match score.  Higher is better.  Anything above 3 is considered a reasonable match |
| debug | An array of strings detailing the progress of this match |
| title | title |
| director | Director - movies only |
| year | Year of first release |
| tt_id | Tanktop ID for the item |
| rt_id | Rotten Tomatoes ID for the item (if available) |
| imdb_id | IMDB ID for the item (if available) |

#### Examples

##### Request
`GET http://api.tanktop.tv/api/1/<apikey>/match/movie/?title=The%20Hunger%20Games&director=Gary%20Ross&year=2012`

##### Response
```json
{
    "director": "Gary Ross",
    "imdb_id": "1392170",
    "match_type": "M",
    "rt_id": 771190618,
    "title": "The Hunger Games",
    "tt_id": 2108,
    "year": 2012
}
```

##### Request
`GET http://api.tanktop.tv/api/1/<apikey>/match/programme/?title=Vikings&cast=Travis+Fimmel&cast=Clive+Standen`
##### Response
```json
{
    "imdb_id": "tt2306299",
    "match_type": "M",
    "title": "Vikings",
    "tt_id": 268,
    "tvdb_id": 260449,
}
```

### Matching and ingestion service
Provide us with a catalog feed or API and we will match and ingest it for you.  Please contact us at <hello@tanktop.tv> if you are interested.

## <a name="Sources"></a> Movie and TV Sources
Sources are providers of movies and TV, e.g. BBC iPlayer, iTunes.  This endpoint lists the sources that are currently
available.  You will need source IDs from this endpoint to query what TV shows and Movies are available across a basket
of sources.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/source/`| List the sources currently available |

| key | value |
|:--|:--|
|id | Identifier for the source |
|payment_model| Either F for free or Ad supported, P for pay as you go, S for subscription or U for unknown.  New sources may be briefly listed as unknown |
|country| Country for the source.  For example iTunes has different catalogs for the UK and US. These are listed as two separate sources with different countries.|
|display_name| A user-appropriate name for the source |
|name| TankTop's internal name for the source |


#### Response
```json
{
    "sources": [ {
            "id": 1,
            "payment_model": "F",
            "country": "GB",
            "display_name": "BBC",
            "name": "BBC"
        }, {
            "id": 2,
            "payment_model": "P",
            "country": "GB",
            "display_name": "Virgin Online Movies",
            "name": "VirginTV"
        }
    ]
}
```

## <a name="Ingestion"></a> TV and Movie Source Ingestion
Tell TankTop about your Movie and TV catalog so that we can list it in our public services and/or in your white-label UI.  Please contact us to be allocated a source ID.  Note only the owner of a source may ingest it.

|Endpoint | Function|
|:--|:--|
|`POST /api/1/<apikey>/ingest/<sourceid>/`| Set the current contents for a TV and Movie source |

Post a JSON body to set the current contents of the source.

Further Documentation coming soon.

## <a name="ItemLists"></a>TV and Movie lists
Get lists of TV & Movies.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/list/_/<itemtype>/`| List TV and movies |

Valid values for itemtype are 'movie' and 'programme'.

### Parameters

| Name      | Value  |
|:--        |:--     |
| source_id | source IDs to include. Items are only returned if they are currently available on one of the given services. If none are specified then all items are returned whether the items are current or not |
| director_id| Movies only.  Return movies directed by this director|
| actor_id  | Return items staring the actor|
|category_id| Return items in the category|
|max_price  | Return items that are available on the included sources below the supplied price.  Price should be in pence or cents.  Note we do not yet support currency conversions |
|min_year | Filter by release year|
|max_year | Filter by release year|
|rent     | Set to true to return only items that are available to rent|
|buy      | Set to true to return only items that are available to buy|
|start    | Where to start in the list of results. default 0|
|count    | How many ids to return. default 12|
|query    | A query string that is matched against Title, actor name and director name|

### Returned values

| Name     | Value  |
|:--       |:--     |
| ids      | A list of movie or TV ids |


### Examples
##### Request
`GET http://api.tanktop.tv/api/1/<api_key>/list/_/movie/?start=0&count=5&max_year=1960`

##### Response
```json
{"ids": [691, 29106, 13785, 10104, 9318]}
```

##### Request
`GET http://api.tanktop.tv/api/1/<api_key>/list/_/programme/?start=0&count=5`

##### Response
```json
{"ids": [379, 668, 6416, 546, 269]}
```

## <a name="EPG"></a>EPG Ingestion
Feed live TV data into the system.

Live TV content it is matched to existing catalog data so that
- users can be offered recommendations from live TV as well as for on-demand content
- users can be notified about content they are interested in when it appears on live TV
- data about live viewing can more effectively influence overall content recommendations

Please contact us at <hello@tanktop.tv> if you are interested in feeding EPG data into the Tank Top TV system. This is only available for commercial users.

## <a name="ItemData"></a>TV and Movie data
Get metadata for movies & TV shows.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/<itemtype>/<itemid>/`| Get metadata for a single item |
|`GET /api/1/<apikey>/<itemtype>/?id=X&id=Y`| Get metadata for a number of items |

### Returned values

| Name       | Value  |
|:--         |:--     |
| programmes | A list of TV programme objects |
| movies     | A list of movie objects |

#### Programme objects

| Name       | Value  |
|:--         |:--     |
| id         | The Tank Top id for the programme |
| imdb_id    | IMDB ID for the programme, if available |
| tvdb_id    | TVDB ID for the programme, if available |
| actors     | A list of actor objects |
| categories | A list of category objects |
| image_url  | TVDB image for the programme.|
| url        | TVDB page for the programme.|
| our_url    | Tank Top TV page for the programme |
| rating     | TVDB rating for the programme |
| rating_count | Number of TVDB ratings contributing to rating |
| title      | Programme title |
| instances  | A list of programme instance objects |


#### Movie objects

| Name       | Value  |
|:--         |:--     |
| id         | The Tank Top id for the movie |
| rt_id      | Rotten Tomatoes ID for the movie, if available |
| imdb_id    | IMDB ID for the movie, if available |
| freebase_id| Freebase ID for the movie (/film/film object) |
| actors     | A list of actor objects |
| categories | A list of category objects |
| url        | Rotten Tomatoes page for the movie.|
| our_url    | Tank Top TV page for the movie |
| wikipedia_url | Wikipedia page for the movie (en version)|
| title      | Movie title |
| director_id   | Tank Top ID for the movie director |
| director_name | Name of the director |
| instances  | A list of movie instance objects |
| runtime    | movie runtime in minutes |
| year       | movie release year |


#### Actor objects

| Name       | Value  |
|:--         |:--     |
| id   | Tank Top actor ID |
| name | The actor's name  |


#### Category Objects
Note that TV categories are distinct from Movie categories but may have overlapping names
or ids

| Name       | Value  |
|:--         |:--     |
| id   | Tank Top category ID |
| name | Category name |

#### Programme Instance Objects
| Name        | Value  |
|:--          |:--     |
| id          | Tank Top programme instance ID |
| source_id   | Source ID |
| source_name | Source display name |
| country     | Source country |
| url         | URL for the source's page for the programme |
| series      | list of series numbers available on the source |

#### Movie Instance objects

| Name        | Value  |
|:--          |:--     |
| id          | Tank Top movie instance ID |
| source_id   | Source ID |
| source_name | Source display name |
| country     | Source country |
| url         | URL for the source's page for the movie |
| expires     | if present, when the movie will no longer be available.  Time is in milliseconds since 1 Jan 1970 UTC |
| price_rent | SD rental price.  Price is in pence or cents depending on the country. 0 means free, -1 indicates as part of a subscription plan.  null or not present indicates unavailable. |
| price_rent_hd | HD rental price |
| price_buy | SD purchase price |
| price_buy_hd | HD purchase price |

### Examples
##### Request
`GET http://api.tanktop.tv/api/1/<api_key>/movie/691/`

##### Response
```json
{
    "films": [
        {
            "director_id": 586,
            "our_url": "http%3A%2F%2Fmovies.tanktop.tv%2Fmovie%2F691%2FNorth+by+Northwest",
            "director_name": "Alfred Hitchcock",
            "instances": [
                {"price_rent": 249, "source_name": "Film4oD", "url": "/serviceclick/691/movie/1069/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 3},
                {"price_rent": 249, "price_buy": 699, "source_name": "BlinkBox", "url": "/serviceclick/691/movie/6299/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 6},
                {"price_rent": 249, "source_name": "Virgin Online Movies", "url": "/serviceclick/691/movie/12916/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 2},
                {"price_buy_hd": 999, "price_buy": 799, "source_name": "Google Play", "price_rent": 249, "url": "/serviceclick/691/movie/30621/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 17, "price_rent_hd": 349},
                {"price_buy_hd": 1299, "price_buy": 899, "source_name": "Sony", "price_rent": 249, "url": "/serviceclick/691/movie/34075/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 18, "price_rent_hd": 349},
                {"price_rent": 249, "price_buy": 699, "source_name": "iTunes", "url": "/serviceclick/691/movie/39821/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 24, "price_rent_hd": 349},
                {"price_rent": 249, "source_name": "EE Film Store", "url": "/serviceclick/691/movie/54301/movieinstance/", "country": "GB", "expires": 1401577200000.0, "visible": true, "source_id": 26},
                {"price_rent": 249, "price_buy": 699, "source_name": "Wuaki", "url": "/serviceclick/691/movie/64087/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 32},
                {"source_name": "Amazon Instant", "url": "/serviceclick/691/movie/84603/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 35, "price_rent_hd": 349},
                {"price_buy_hd": 999, "source_name": "Amazon Instant", "url": "/serviceclick/691/movie/87077/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 35},
                {"price_rent": 299, "price_buy": 999, "source_name": "iTunes", "url": "/serviceclick/691/movie/105798/movieinstance/", "country": "US", "expires": null, "visible": false, "source_id": 37, "price_rent_hd": 399},
                {"price_rent": 249, "source_name": "Amazon Instant", "url": "/serviceclick/691/movie/138938/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 35},
                {"price_buy": 699, "source_name": "Amazon Instant", "url": "/serviceclick/691/movie/142458/movieinstance/", "country": "GB", "expires": null, "visible": true, "source_id": 35},
                {"price_buy_hd": 1299, "source_name": "Amazon Instant Video", "price_rent": 199, "url": "/serviceclick/691/movie/148656/movieinstance/", "country": "US", "expires": null, "visible": false, "source_id": 39},
                {"price_buy_hd": 1299, "price_buy": 999, "source_name": "Google Play", "price_rent": 199, "url": "/serviceclick/691/movie/163010/movieinstance/", "country": "US", "expires": null, "visible": false, "source_id": 41, "price_rent_hd": 299},
                {"price_buy_hd": 1299, "price_buy": 899, "source_name": "Redbox Instant", "price_rent": 299, "url": "/serviceclick/691/movie/176971/movieinstance/", "country": "US", "expires": null, "visible": false, "source_id": 42, "price_rent_hd": 399}
            ],
            "year": 1959,
            "imdb_id": "0053125",
            "id": 691,
            "categories": [
                {"name": "Mystery & Suspense", "id": 6},
                {"name": "Classics", "id": 14}
            ],
            "rt_id": 18639,
            "url": "http://www.rottentomatoes.com/m/north-by-northwest/",
            "title": "North by Northwest",
            "actors": [
                {"name": "James Mason", "id": 1967},
                {"name": "Cary Grant", "id": 2125},
                {"name": "Eva Marie Saint", "id": 4604},
                {"name": "Jesse Royce Landis", "id": 4605},
                {"name": "Leo G Carroll", "id": 4606}
            ],
            "runtime": 136
        }
    ]
}
```

##### Request
`GET http://api.tanktop.tv/api/1/<api_key>/programme/379/`

##### Response
```json
{
    "programmes": [
        {
            "rating": 9.3000000000000007,
            "tvdb_id": 81189,
            "title": "Breaking Bad",
            "url": "http://thetvdb.com/?tab=series&id=81189",
            "series": {},
            "our_url": "http%3A%2F%2Fmovies.tanktop.tv%2Fprogramme%2F379%2F%2FBreaking+Bad",
            "imdb_id": "tt0903747",
            "instances": [
                {"price_buy_hd": null, "source_name": "BlinkBox", "price_rent": null, "url": "/serviceclick/379/programme/626/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 6, "id": 626, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Sony", "price_rent": null, "url": "/serviceclick/379/programme/2398/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 18, "id": 2398, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Google Play", "price_rent": null, "url": "/serviceclick/379/programme/4894/programmeinstance/", "series": [null, 1, 2, 3, 4, 5], "price_buy": null, "visible": 1, "country": "GB", "source_id": 17, "id": 4894, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Netflix", "price_rent": -1, "url": "/serviceclick/379/programme/9954/programmeinstance/", "series": [1, 2, 3, 4, 5], "price_buy": null, "visible": 1, "country": "GB", "source_id": 8, "id": 9954, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Wuaki", "price_rent": null, "url": "/serviceclick/379/programme/10727/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 32, "id": 10727, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Xbox", "price_rent": null, "url": "/serviceclick/379/programme/13403/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 12, "id": 13403, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Netflix US", "price_rent": -1, "url": "/serviceclick/379/programme/15852/programmeinstance/", "series": [1, 2, 3, 4, 5], "price_buy": null, "visible": 0, "country": "US", "source_id": 38, "id": 15852, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "iTunes", "price_rent": null, "url": "/serviceclick/379/programme/19500/programmeinstance/", "series": [1, 3, 4, 5], "price_buy": null, "visible": 0, "country": "US", "source_id": 37, "id": 19500, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Amazon Instant", "price_rent": null, "url": "/serviceclick/379/programme/34747/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 35, "id": 34747, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Google Play", "price_rent": null, "url": "/serviceclick/379/programme/37393/programmeinstance/", "series": [null, 1, 2, 3, 4, 5], "price_buy": null, "visible": 0, "country": "US", "source_id": 41, "id": 37393, "price_rent_hd": null}
            ],
            "synopsis": "Walter White, a struggling high school chemistry teacher, is diagnosed with advanced lung cancer. He turns to a life of crime, producing and selling methamphetamine accompanied by a former student, Jesse Pinkman, with the aim of securing his family's financial future before he dies.",
            "rating_count": 614,
            "image_url": "http://thetvdb.com/banners/posters/81189-22.jpg",
            "actors": [
                {"name": "Bob Odenkirk", "id": 1074},
                {"name": "Bryan Cranston", "id": 3293},
                {"name": "Jonathan Banks", "id": 5692},
                {"name": "Giancarlo Esposito", "id": 5788},
                {"name": "Aaron Paul", "id": 10136}
            ],
            "id": 379,
            "categories": [
                {"name": "Comedy", "id": 1},
                {"name": "Drama", "id": 4},
                {"name": "Crime", "id": 8},
                {"name": "Thriller", "id": 9},
                {"name": "Suspense", "id": 11}
            ]
        }
    ]
}
```


## <a name="UserProfiles"></a> User Profiles

We support independent sets of user profiles for different services, one of which is the Tank Top TV service. Access to
retrieving information from or Tank Top TV user profiles is via OAuth.  Please let us know at <hello@tanktop.tv> if this is something you are interested in.

You can use our User profiles API to store user settings and details as JSON data.  User profiles are only visible to the service that creates them.  The service may provide their own user IDs (up to 32 characters).

|Endpoint | Function|
|:--|:--|
|`PUT /api/1/<apikey>/user/<userid>/` | create a user using your user id and set its initial profile|
|`POST /api/1/<apikey>/user/` | create a user, we'll choose the user id|
|`PATCH /api/1/<apikey>/user/<userid>/` | update an existing user profile|
|`POST /api/1/<apikey>/user/<userid>/` | update an existing user profile|
|`GET /api/1/<apikey>/user/<userid>/` | retrieve user profile|
|`DELETE /api/1/<apikey>/user/<userid>/` | deactivate a user|

Profiles are JSON and you can store pretty much anything you want (but please, no important personal information just yet).  Currently, the only restrictions are that the user ID is stored under the key 'id', and that top-level keys starting _ are reserved for our future use.

## <a name="UserActions"></a> User Actions and Events
This is for tracking events that take place on your system so that user actions can feed into personalized listings and recommendations, as well as into analytics tracking.

|Endpoint | Function|
|:--|:--|
|`POST /api/1/<apikey>/event/<userid>/<eventname>/<itemtype>/<itemid>/[<value>/]` | Create an event or add to a list |
|`DELETE /api/1/<apikey>/event/<userid>/<eventname>/<itemtype>/<itemid>/`| Delete an event or remove from a list|
|`GET /api/1/<apikey>/event/<userid>/<eventname>/<itemtype>/`| Get the items in a list, or for which an event has happened|


POST to add to a list, DELETE to remove, GET returns the items currently in a list.

Note that the userid does not have to be registered with Tank Top: you don't need to use the user profile API to make use of personalised lists or events.

### Parameters

The GET version accepts a single parameter **id** which can be repeated. This specifies the item ids you want to query to see if there are any events associated with those items.  Our expectation is
that you use events in the following manner.

1. You retrieve a list of movie or programme ids.  See [below](#PersonalizedLists) for how you can filter these by event so that you can, say, list only items on a watchlist, or exclude hidden or seen items.
2. You retrieve the meta data for these ids.
3. You get events for these ids, so that you can, for example, show the user's ratings for a show, or whether it is included in a watchlist.

### Event descriptions

The Tank Top TV platform supports event reporting of event types with any arbitrary name, although the pre-defined events described here have specific meanings. If you want to report additional event types we recommend you discuss this with us so that we can agree a reserved event name that

Posting these pre-defined events influences the Tank Top TV recommendations and personalization calculations. We recognize that not all UIs support all event types, so we work with commercial users of our platform to tune the parameters associated with the different event types.

#### Content events

These events are associated with content items (movies, TV programmes / series / episodes, whether on-demand or live.)

| Event name | Description | Value |
|:--         |:--          |:--    |
| watchlist  | User wants to watch this item (later) - this adds the item to their watchlist | Not used |
| seen       | User marks this item as seen | Not used |
| hide       | User wants to strike this item out of lists (because they are not interested in it) | Not used |
| rated      | User's rating for this item | A numeric rating.  Use positive numbers for positive sentiments, negative numbers for negative sentiments. |
| progress   | User is partway through watching this item (applies to movies & TV episodes, but not to TV programmes or series) | Number of minutes watched so far |

#### Section events

Events supported for item type 'section' are described in [Section Lists](listOfLists.html#Events)

#### <a name="ChannelEvents"><a/> Channel events

These events are associated with live TV channels

| Event name | Description | Value |
|:--         |:--          |:--    |
| hide       | User wants to strike this channel out of lists (because they are not interested in it) | Not used |
| start      | User starts watching this channel | Timestamp |
| end        | User stops watching this channel (e.g. channel change, TV off, DVR or on-demand content) | Timestamp |

### Examples
#### Request
Here we add movie 1 to user 7's watchlist.  There is no response.

``` POST http://api.tanktop.tv/api/1/<apikey>/event/7/watchlist/movie/1/ ```

#### Request
Here we ask if movies 1, 24 and 72 are in the user's watchlist.  1 and 24 are, 72 is not.

``` GET http://api.tanktop.tv/api/1/<apikey>/event/7/watchlist/movie/?id=1&id=24&id=72 ```

#### Response
```json
{
    "watchlist": {1:null, 24:null}
}
```

## <a name="PersonalizedLists"></a> Personalized Movie and TV lists

Get personalized listings for movie & TV shows.  This is the list of available shows on your services, not including seen or hidden items, taking into account your category preferences, and ordered according to goodness, recency, category balance - essentially this is the 'magic' personalized lists of things available to you, ordered according to what you might be most interested to watch.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/list/<userid>/<itemtype>/`| Get the personalized item list for the user |

This is essentially an extension of [Listing Movies and TV](#ItemLists), adding a personalised dimension.  The URL is the same, with the _ replaced by a user ID.  We also have additional parameters to control how the list is filtered by events.

Note that the userid does not have to be registered with Tank Top: you don't need to use the user profile API to make use of personalised lists or events.

### Parameters

| Name       | Value  |
|:--         |:--     |
| filter     | An event name.  Include only items for which the user has registered an event of this name.  E.g. specify `filter=watchlist` to show the user's watchlist |
| exclude    | An event name.  Exclude items for which the user has registered an event of this name.  E.g. specify `exclude=seen&exclude=hide` to exclude items the user has seen or hidden |

## <a name="Notifications"></a> Notifications

We'll tell you when a movie or TV show becomes available.  You can filter by price, source, and you can insist on HD.  We'll notify you by a GET to a webhook URL you specify. You will only be notified once per request.

|Endpoint | Function|
|:--|:--|
|`POST /api/1/<apikey>/notification/<userid>/<itemtype>/<itemid>/`| Request notification for a movie or TV programme |
|`GET /api/1/<apikey>/notification/<userid>/<itemtype>/<itemid>/` | Get a list of current notifications |
|`GET /api/1/<apikey>/notification/<userid>/<itemtype>/` | Get a list of current notifications |
|`GET /api/1/<apikey>/notification/<userid>/` | Get a list of current notifications |
|`DELETE /api/1/<apikey>/notification/<userid>/<itemtype>/<itemid>/`| Cancel the notification request |
|`DELETE /api/1/<apikey>/notification/<userid>/<itemtype>/`| Cancel the notification requests for this item type |
|`DELETE /api/1/<apikey>/notification/<userid>/`| Cancel the notification requests for this user |


### Parameters

Parameters apply to POST only, and should be sent either as a urlformencoded or JSON body.

| Name      | Value  | Required |
|:--        |:--     |:--       |
| source_id | source IDs to filter on. You will only be notified when the movie or programme is available on a specified source.  If you do not specify any source_ids then all sources are considered | N |
| url       | Webhook URL.  We will issue a POST to this URL when the notification request is satisfied.  We will add urlformencoded parameters user_id, item_type and item_id | Y |
| price     | Price in pence or cents.  We will notify only if the item is available for this value or less.  If not specified any match at any price will generate a notification | N |
| hd        | Set to true if you only want a match when the content is available in HD | N |

### Returned values
GET requests return a list of JSON objects containing the following fields.

| Name       | Value  |
|:--         |:--     |
| item_type | Either "movie" or "programme" |
| item_id   | The movie or programme ID |
| source_id | A list of source IDs that could satisfy this request |
| price     | Requested maximum price.  null if no maximum requested |
| hd        | true if only HD instances should satisfy this request |
| url       | the webhook url |
| notified  | True if a POST has been sent to the webhook. |


### Examples
#### Request
Here we ask, on behalf of user 7, to be notified when movie 1 becomes available on sources 7, 37 or 28.  There is no response.

``` POST http://api.tanktop.tv/api/1/<apikey>/notification/7/movie/1/ ```

Post body (content-type: application/json)
```json
{
    "source_id" : [7, 37, 28]
}
```

#### Request
Here we ask what notifications are outstanding for user 7.

``` GET http://api.tanktop.tv/api/1/<apikey>/notification/7/ ```

#### Response
```json
{
    "notifications": [{
        "url": "http://example.com/notification",
        "notified": false,
        "price": null,
        "item_type": "movie",
        "source_id": [7, 37, 28],
        "item_id": 1,
        "hd": false
    }]
}
```

## <a name="Recommendations"></a> Recommendations

- Item-item similarity
- Recommended items for a user

Documentation coming soon.

See also [Personalized Lists](#PersonalizedLists).

## Full lists in one query

This endpoint combines the results from id lists, movie and programme data and events all in one result.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/fulllist/<userid>/<itemtype>/`| Get the personalized item list for the user, rendered in JSON, with event filtering and annotation |

### Parameters
Parameters for lists, movie and programme data and events all mixed together.

### Examples

```GET http://api.tanktop.tv/api/1/<apikey>/fulllist/<userid>/movie/?start=0&count=1&exclude=seen&exclude=hide&event=watchlist&event=rated```

```json
{
    "programmes": [{
            "rating": 9.3000000000000007,
            "tvdb_id": 81189,
            "title": "Breaking Bad",
            "url": "http://thetvdb.com/?tab=series&id=81189",
            "series": {},
            "our_url": "http%3A%2F%2Fmovies.tanktop.tv%2Fprogramme%2F379%2F%2FBreaking+Bad",
            "imdb_id": "tt0903747",
            "instances": [
                {"price_buy_hd": null, "source_name": "BlinkBox", "price_rent": null, "url": "/serviceclick/379/programme/626/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 6, "id": 626, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Amazon Instant", "price_rent": null, "url": "/serviceclick/379/programme/34747/programmeinstance/", "series": [1, 2, 3, 4, 5, 6], "price_buy": null, "visible": 1, "country": "GB", "source_id": 35, "id": 34747, "price_rent_hd": null},
                {"price_buy_hd": null, "source_name": "Google Play", "price_rent": null, "url": "/serviceclick/379/programme/37393/programmeinstance/", "series": [null, 1, 2, 3, 4, 5], "price_buy": null, "visible": 0, "country": "US", "source_id": 41, "id": 37393, "price_rent_hd": null}
            ],
            "synopsis": "Walter White, a struggling high school chemistry teacher, is diagnosed with advanced lung cancer. He turns to a life of crime, producing and selling methamphetamine accompanied by a former student, Jesse Pinkman, with the aim of securing his family's financial future before he dies.",
            "rating_count": 614,
            "image_url": "http://thetvdb.com/banners/posters/81189-22.jpg",
            "actors": [
                {"name": "Bob Odenkirk", "id": 1074},
                {"name": "Bryan Cranston", "id": 3293},
                {"name": "Jonathan Banks", "id": 5692},
                {"name": "Giancarlo Esposito", "id": 5788},
                {"name": "Aaron Paul", "id": 10136}
            ],
            "id": 379,
            "categories": [
                {"name": "Comedy", "id": 1},
                {"name": "Drama", "id": 4},
                {"name": "Crime", "id": 8},
                {"name": "Thriller", "id": 9},
                {"name": "Suspense", "id": 11}
            ]
        }]
    "watchlist": {379: null},
    "rated": {379: 1},
}
```

## <a name="URLLookup"></a> Url lookup
Look up movies and TV episodes by the URL from a VOD provider website.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/lookup/byurl/`| Look up an episode or movie by its url |

### Parameters

| Name      | Value  | Required |
|:--        |:--     |:--       |
|url        | The canonical URL for the episode or film | Yes |

So far we have tested this with BBC IPlayer, ITVPlayer, 4oD & Demand 5. Please read the notes on these below.

#### BBC IPlayer
Urls look like this http://www.bbc.co.uk/iplayer/episode/b048zhsh/hardtalk-patrick-honohan-governor-central-bank-of-ireland.  The correct link is shown in the canonical `<link>` tag in the header of the IPlayer page for the episode (shown below). It does _not_ match the URL provided in the BBC RSS feeds or from the programme website (which is missing the slug).

```
<link rel="canonical" href="http://www.bbc.co.uk/iplayer/episode/b048zhsh/hardtalk-patrick-honohan-governor-central-bank-of-ireland">
```

You can use the following javascript within an episode page (i.e. as part of a browser extension or bookmarklet) to get the canonical URL.

```javascript
var url = document.evaluate("//link[@rel='canonical']/@href", document, null, XPathResult.STRING_TYPE, null).stringValue;
```

#### ITVPlayer
Please provide the link for an episode, not for a programme. For example https://www.itv.com/itvplayer/all-star-mr-and-mrs/series-6/episode-2, not https://www.itv.com/itvplayer/all-star-mr-and-mrs.

Note if you land on a programme page, the latest episode is highlighted.  To get the episode URL of the highlighted episode you can use javascript like the following.

```javascript
var episode = document.getElementsByClassName("node-episode episode-highlight")[0];
var url = document.location.protocol + '//' + document.location.host + episode.getAttribute('about');
```

#### 4oD
Episode URLs include a fragment identifier (a string following # at the end of the URL).  For example http://www.channel4.com/programmes/this-old-thing-the-vintage-clothes-show/4od#3720634. You need to include the fragment ID in requests to the Tank Top API.

On the 4oD site, links without fragments will refer to the latest available episode of the programme.E.g http://www.channel4.com/programmes/this-old-thing-the-vintage-clothes-show/4od currently shows series 1 episode 2 in a "hero" section at the top of the page, as well as highlighting it below in a list of available episodes.  In the list, the episode is represented by an li tag with class="episode-item item-data cf has-audio-describe selected".  The key classes are 'episode-item' and 'selected'.  We can use these classes to find the correct URL for the episode with the following javascript.

```javascript
var episode_li = document.getElementsByClassName("episode-item selected")[0];
var url = document.location.href + '#' + episode_li.dataset.assetid;
```

#### Demand 5
Please provide the link for an episode, not for a programme. For example: http://www.channel5.com/shows/thomas-and-friends/episodes/the-lion-of-sodor, not http://www.channel5.com/shows/thomas-and-friends.

Note if you land on the programme page, the latest episode is displayed in a box to the right of the screen.  You can find the url for this episode with the following javascript.

```javascript
var elt = document.getElementById('demand_episode_thumbnail');
var url = document.location.protocol + '//' + document.location.host + elt.children[0].getAttribute('href');
```

### Returned values
Tank Top represents movies by movie objects, and TV content by a hierachy of programme, series and episode objects.  To represent references to movies and TV on websites it has 'movie instance' objects, 'programme instance', 'series instance' and 'episode instance' objects. Instance objects link to the main object once their identity has been resolved, but can exist without links to the main objects if their identity has not been determined.

This API returns the instance objects the url refers to.  At present it looks for movie and episode instances, but in future it could be extended to include series and programme instances.  It allows for returning lists of instances in case a VOD site does not provide unique stable urls for individual items.

The JSON objects returned are as follows.

| Name       | Value  |
|:--         |:--     |
| episode_instances | A list of matching episode instances |
| movie_instances   | A list of matching movie instances |

The following fields are returned for episode instances.

| Name       | Value  |
|:--         |:--     |
| id | Tank Top ID for the instance |
| source_id | Tank Top ID for the source (i.e. for the VoD website) |
| series_instance_id | Tank Top ID for the series instance |
| programme_instance_id | Tank Top ID for the programme instance |
| programme_id | Tank Top ID for the programme. May be null|
| beamly_eid | Beamly episode ID if available |
| beamly_sid | Beamly series ID if available |
| beamly_bce | Beamly Broadcast Event ID if available |

The following fields are returned for movie instances

| Name       | Value  |
|:--         |:--     |
| id | Tank Top ID for the instance |
| source_id | Tank Top ID for the source (i.e. for the VoD website) |
| movie_id | Tank Top ID for the movie. May be null|
| beamly_eid | Beamly episode ID if available |
| beamly_sid | Beamly series ID if available |
| beamly_bce | Beamly Broadcast Event ID if available |

See https://develop.beamly.com/apis/guide/epg for details about Beamly identifiers.

## <a name="Popular"></a> Popular content

List of currently popular content on your service.  (Please contact us at <hello@tanktop.tv> if you want this endpoint enabled for your service.)

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/popular/<itemtype>/[days/]`| Most popular items on your service over the last specified number of days (defaults to 7).  Itemtype may be 'movie' or 'programme' |
|`GET /api/1/<apikey>/popular/live`| Most popular live channels on your service at this time (based on [Channel Events](#ChannelEvents)). Response includes data about the currently-playing item (as specified by the EPG) |

## <a name="Collections"></a> Curated collections

Manage editorial collections of content on your service.  (Please contact us at <hello@tanktop.tv> if you want this endpoint enabled for your service.)

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/collection/`| Retrieve all the curated collection |
|`GET /api/1/<apikey>/collection/<id>` | Retrieve the specified curated collection |
|`POST /api/1/<apikey>/collection/`| Create a new curated collection |
|`PATCH /api/1/<apikey>/collection/<id>`| Add a new content item to a curated collection |
|`DELETE /api/1/<apikey>/collection/<id>`| Remove curated collection |

## Credits
Some TV information is provided by http://www.thetvdb.com/ under a [Creative Commons 3.0 licence](http://creativecommons.org/licenses/by/3.0/us/)
