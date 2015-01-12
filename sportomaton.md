---
title: Personalizing sports content
layout: default
---

Personalization of sports contents
- surfaces live sports content that is relevant for each user
- associates live and non-live content that might be of interest to a user based on their sports preferences

*This API is not only available through the free, non-commercial API agreement. Please contact us at <hello@tanktop.tv> if you would like to personalize sports content.*

## Sports definitions

| Endpoint | Function |
| :-- | :-- |
| `GET /api/1/<apikey>/sport`| Get a list of sports and their IDs |
| `GET /api/1/<apikey>/sport/<sportid>/team` | Get a list of the teams and their IDs for this sport |

## Creating sports preferences

The system can infer sports preferences from viewing content, but you may wish to bootstrap the process by allowing users to specify the sports and teams they are interested in.

| Endpoint | Function |
| :-- | :-- |
| `POST /api/1/<apikey>/user/<userid>/sport/<sportid>`| This user enjoys this sport |
| `POST /api/1/<apikey>/user/<userid>/sport/<sportid>/<teamid>` | This user follows this team |
| `GET /api/1/<apikey>/user/<userid>/sport`| Get a list of the sports and teams this user follows |
| `DELETE /api/1/<apikey>/user/<userid>/sport/<sportid>` | This user is not interested in this sport |
| `DELETE /api/1/<apikey>/user/<userid>/sport/<sportid>/<teamid>` | This user is not following this team |

## Identifying sports content

If the metadata contains sufficent information, content can be automatically associated with the sports and teams that it covers. Alternatively you may wish to manually associate particular pieces of content with a sport and/or a team.

For example, the coverage of a live football match will be associated with both the home and away teams.

| Endpoint | Function |
| :- | :- |
| `POST /api/1/<apikey>/sport/<sportid>/movie/<movieid>` | Associate a movie with a sport |
| `POST /api/1/<apikey>/sport/<sportid>/programme/<progid>` | Associate an entire TV show (all series & episodes) with a sport |
| `POST /api/1/<apikey>/sport/<sportid>/episode/<episodeid>` | Associate an episode with a sport |
| `POST /api/1/<apikey>/sport/<sportid>/team/<teamid>/movie/<movieid>` | Associate a movie with a sports team |
| `POST /api/1/<apikey>/sport/<sportid>/team/<teamid>/programme/<progid>` | Associate a programme with a sports team |
| `POST /api/1/<apikey>/sport/<sportid>/team/<teamid>/episode/<episodeid>` | Associate an episode with a sports team |
| `GET /api/1/<apikey>/sport/<sportid>` | Get a list of the current content related to this sport |
| `GET /api/1/<apikey>/sport/<sportid>/team/<teamid>` | Get a list of the current content related to this team |

## Personalized sports content listings

There is an endpoint for retrieving all the current sports content relevant to a given user.

`GET /api/1/<apikey>/user/<userid>/sport`

[Notifications](../UserEvents.html#Notifications) can be automatically generated for a user whenever there is content available relevant to their sports preferences.

Relevant sports content will score highly in the overall list for the user.

