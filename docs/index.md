# Tank Top TV API

Welcome to the Tank Top TV APIs.  These are free to use for non-commercial use, but please get in touch with us at <hello@tanktop.tv> if you'd like to use them for commercial purposes.

You will need an API key.  Contact <hello@tanktop.tv> for details.

1. [Movie and TV matching API](#Matching)
2. [TV and Movie Source Ingestion](#Ingestion)
3. [User Profiles](#UserProfiles)
4. [User Actions and Events](#UserActions)
5. [Personalised Movie & TV lists](#PersonalizedLists)
6. [Recommendations](#Recommendations)
7. [Search & typeahead](#Search)

We're still in the process of documenting our APIs but please get in touch if you're interested in using an endpoint that is not fully written-up yet.

## API usage

You will need an API key.  Contact <hello@tanktop.tv> for details.

Data is returned in JSON format.

## <a name="Matching"></a> Movie and TV matching API
Use this API to match items in your catalog against the TankTop entertainment graph.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<apikey>/match/<itemtype>/`| Get matches for a TV programme or movie|

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


### Matching and ingestion service
Provide us with a catalog feed or API and we will match and ingest it for you.

## <a name="Ingestion"></a> TV and Movie Source Ingestion
Tell TankTop about your Movie and TV catalog so that we can list it in our public services and/or in your white-label UI.  Please contact us to be allocated a source ID.

|Endpoint | Function|
|:--|:--|
|`POST /api/1/ingest/<sourceid>/`| Set the current contents for a TV and Movie source |

Post a JSON body to set the current contents of the source

```json
{
    timestamp: <Format?>
    tv: []
    movies: []
}
```

### TV format

Documentation coming soon.

### Movie format

Documentation coming soon.

## TV Programme and Movie data
Get metadata for the movies & TV shows.

**Note**: This does not include synopses or images.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/<itemtype>/<itemid>/`| Get metadata for a single item |
|`GET /api/1/<itemtype>/?id=X&id=Y`| Get metadata for a number of items |

Metadata format documentation coming soon.

## <a name="UserProfiles"></a> User Profiles

User profiles are only visible to the service that creates them.  The service may provide their own user IDs.

|Endpoint | Function|
|:--|:--|
|`PUT /api/1/user/<userid>/` | create a user using your user id and set its initial profile|
|`POST /api/1/user/` | create a user, we'll choose the user id|
|`PATCH /api/1/user/<userid>/` | update an existing user profile|
|`GET /api/1/user/<userid>/` | retrieve user profile|
|`DELETE /api/1/user/<userid>/` | deactivate a user|


## <a name="UserActions"></a> User Actions and Events
This is for tracking events that take place on your system so that user actions can feed into personalized listings and recommendations, as well as into analytics tracking.


|Endpoint | Function|
|:--|:--|
|`POST /api/1/event/<userid>/<eventname>/<itemtype>/<itemid>/[<value>/]` | Create an event or add to a list |
|`DELETE /api/1/event/<userid>/<eventname>/<itemtype>/<itemid>/`| Delete an event or remove from a list|
|`GET /api/1/event/<userid>/<eventname>/<itemtype>/`| Get the items in a list, or for which an event has happened|


POST to add to a list, DELETE to remove, GET returns the items currently in a list

### Event descriptions

Documentation coming soon.

## <a name="PersonalizedLists"></a> Personalized Movie and TV lists

Get personalized listings for movie & TV shows.  This is the list of available shows on your services, not including seen or hidden items, taking into account your category preferences, and ordered according to goodness, recency, category balance - essentially this is the 'magic' personalized lists of things available to you, ordered according to what you might be most interested to watch.

|Endpoint | Function|
|:--|:--|
|`GET /api/1/list/<userid>/<itemtype>/`| Get the personalized item list for the user |
|`GET /api/1/list/_/<itemtype>/`| Get the best item list for anonymous users |

Return value is a list of ids.  Can also expand that to include metadata associated with the ID, and list inclusion data for specified events.  E.g. you could ask for the first 12 movie IDs, get the catalog metadata for them, and whether the user has rated the items or added them to their watchlist.


## <a name="Recommendations"></a> Recommendations

- Item-item similarity
- Recommended items for a user

Documentation coming soon.

See also [Personalized Lists](#PersonalizedLists).

## <a name="Search"></a> Search and Typeahead

Documentation coming soon.

