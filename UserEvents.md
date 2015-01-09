---
layout: default
title: User, event and personalization APIs
---

1. [User Profiles](#UserProfiles)
2. [User Actions and Events](#Events)
3. [Notifications](#Notifications)
4. [Recommendations](#Recommendations)
5. [Bookmarks](#Bookmarks)

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

## <a name="Events"></a> User Actions and Events
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

#### <a name="ChannelEvents"></a> Channel events

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

See also [Personalized Lists](../EntertainmentData.html#PersonalizedLists).

## <a name="Bookmarks"></a> Bookmarks

We use 'progress' and 'seen' [events](#Events) to track whether a user is part-way through a TV show or film.  In the case of a TV show, this keeps track of which episode they are currently watching or, if they have completed an episode, which episode is up next.  This can be used to implement a "Currently watching" feature in the UI.

In the case of live TV, the channel 'start' and 'stop' events can implicitly generate the content 'progress' and 'seen' events.

| Endpoint | Function |
| :--- | :--- |
| `GET /api/1/<apikey>/bookmark/<userid>` | List of bookmarks for this user |
| `GET /api/1/<apikey>/bookmark/<userid>/<itemtype>/<itemid>` | Bookmark position within a given movie or TV programme |

Bookmark parameter documentation coming soon.


