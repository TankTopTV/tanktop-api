---
title: Section lists
---

# Section Lists

Section lists support the type of UI where content is arranged as a series of (typically) horizontal lists.  The lists are things like "popular movies", "because you watched" (i.e. items similar to something you've just watched), categories you might be interested in, etc.  We call these horizontal lists "sections".

The API allows you to define which sections are available to appear on the screen, and give them weights to control how they are ordered. Sections are then ordered dynamically for each user based on which sections they actually use.

*Section Lists APIs are not included in our free, non-commercial API usage.  Please contact us at <hello@tanktop.tv> if you would like to use Section Lists.*

- Section definitions.  These define on a global basis which sections the admin wants to be included in the UI, and the relative initial priority of each section.

- Section instances.  These track the relative performance of a section for each user, and store information about the contents of each dynamic section.

## Section Definition

Section definitions define which section types are used, and their relative priorities.

| Endpoint | Function |
| :--- | :--- |
| `POST /api/1/<apikey>/sectiondef/` | Create a new section definition |
| `GET /api/1/<apikey>/sectiondef/` | Get a list of section definitions |
| `DELETE /api/1/<apikey>/sectiondef/` | Remove a section definition |

A section definition contains the following fields.

| Field name | Description |
| :--- | :--- |
| type | The type of the section. |
| weight | An initial score to order the sections.  A higher number will tend to keep the section higher in the list. Negative weights pin a section to a position in a list - -1 pins to the first slot, -2 to the second, etc |
| count | Count determines how many times this section can appear in the list.  This is used only for dynamically created sections that have a basis item, e.g. "because you watched" which has a basis item of a movie or programme. Defaults to 1. |
| url | Sections of type "URL" only. The URL used to obtain item information for the section.  This may be templated e.g. "/url/{{0}}/info" and the current user ID will be inserted for {{0}} when the URL is returned as part of a section list instance.  |
| name | User-facing name to use for this section.  For dynamic sections with a basis item this may be a template string (e.g. "Because you watched {{0}}") and the basis item name will be inserted for {{0}} when the name is returned as part of a section list instance. |

### Section types

The section type determines what is contained in each section.  The different types fall into three categories:

- Section type "URL", where the content of the section is retrieved from a specified URL
- Dynamic sections, where the content depends on the user's behaviour e.g. items they have added to a watchlist
- Dynamic sections with a basis item, where the content depends on the user's behaviour and relates to a particular item (the "basis item") e.g. "BecauseYouWatched" where the basis item is a movie or programme that this user watched recently

#### URL sections

In a URL section, the results might be filtered to remove items that a user has already seen or dismissed, but in other respects the content  is the same for all users. These sections are defined by a (templated) URL endpoint where the list of items is obtained from, with space for the the user id to be inserted, so that we can retrieve, for example, popular items excluding ones the user has already consumed or explicitly rejected.

Example endpoints for URL sections types that are provided by Tank Top TV include

- curated collections
- popular items
- new items

URL sections also allow for the flexibiliity to retrieve section data from entirely independent services.

#### Dynamic sections with a basis item

All other section types are "dynamic": their contents vary based on user activity (e.g. "Because you watched" sections). Some of these are built on the basis of a specific item (for example a recently watched programme, or a category that the user enjoys). This basis item is included in the section instance response.

| Type | Basis item | Description |
| :--- | :--- | :--- |
| BecauseYouWatched | Content item (movie or programme) | List of items similar to what the user has recently watched. |
| Category | Category | Items from the user's highest scoring category. |
| BecauseYouActor | Actor | When the system detects that a user is interested in a particular actor, this adds a list of items the actor starred in |

#### Dynamic section without a basis item

Not all dynamic sections have a basis item.

| Type | Description |
| :--- | :--- |
| Watchlist | The user's watchlist.  |
| CurrentlyWatching | Content that the user is part-way through watching |

(Additional dynamic section types are to be defined.)

## Section Instance

The section instance list tells a UI component which sections to display for a user.

If a section is not relevant for a particular user (e.g. they have not added any items to their watchlist) then the section instance is empty and will not be included in the section instance list.

| Endpoint | Function |
| :--- | :--- |
| GET `/api/1/<apikey>/sectionlist/user/<userid>/` | Returns a list of section instances to show this user, in decreasing order of score |

Each section instance contains the following fields.

| Field name | Description |
| :--- | :--- |
| id  | an ID for the section instance, used for reporting events |
| type | type of the section |
| score | Score indicates the performance of this section.  It is a combination of the weight from the section definition and the success of this section with the user |
| basis | If there is a basis item for this section, this identifies its item type and item ID (e.g. the movie which is referred to in a "BecauseYouWatched" section.) |
| name | The name for the section, as defined by the section definition. For dynamic sections the name of the basis item is inserted in the template  |
| url | The URL to use to retrieve items for this section |

## Example

For example, supposed there are two sections defined - one for a curated list and another for a BecauseYouWatched dynamic list. The list of section definitions is as follows.

```
[
    {
        type: "URL",
        weight: 50,
        url: "http://tanktop.tv/api/1/12345/collection/14",
        name: "Editor's picks"
    },
    {
        type: "BecauseYouWatched",
        weight: 40,
        name: "Because you watched {{0}}"
    }
]
```

Let's suppose that our user Bob has clicked on lots of items from the BecauseYouWatched section, and its score has been boosted to 48, but hasn't been interested in items in Editor's Picks, and its score has decayed to 45.  When the UI component retrieves the section instance list to display for Bob, the response is as follows:

```
[
    {
        id: 1001,
        type: "BecauseYouWatched",
        name: "Because You Watched Star Trek"
        url: "/api/1/12345/section/1001"
        weight: 48,
        basis: {
            item_type: programme,
            item_id: 315
        }
    },
    {
        id: 1024,
        type: "URL",
        name: "Editor's Picks",
        url: "http://tanktop.tv/api/1/12345/collection/14",
        weight: 45
    }
]
```

## <a name="Events"></a> Events and Scoring
Scores for section instances decay over time, but are increased if the user selects or consumes content from the section.  The UI needs to report these events so that the section instance is updated appropriately. For example, if the user opens a detail page for an item in a section the UI should track that the page was opened from the section, and report if the content is consumed.

Report user events using the [event API](index.html#UserActions), an item type of "section" and the id from the section instance.  The events to report are as follows

| Event | Description |
| :--- | :--- |
| watchlist | An item has been added to a watchlist for later consumption |
| detail | the user has opened a detail page for an item in the section |
| hide | the user has hidden an item in the section |
| rated | the user has rated an item in the section |
| seen | the user has marked an item in the section as seen |
| consumed | the user has bought, watched or otherwised consumed an item in the section |
| displayed | the section has been displayed to the user.  This is used to reduce the section scores to implement score decay over time. |

 We will work with you to determine appropriate scores for each event that your UI supports.

 These section events should be reported in addition to the per-item event.  For example, when the user adds a movie to their watchlist from within a section there will be a watchlist event related to the movie itself, and another related to its section.
