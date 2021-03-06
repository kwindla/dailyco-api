Using the Daily.co API, you can manage domains, teams, and video call rooms. And you can embed video calls into any web app or web view.

There are currently six API methods:
  - [Get invite link, rooms list, and user list for a team](#api-team-info)
  - [Create a room](#api-room-create)
  - [Change privacy setting for a room](#api-room-update)
  - [Delete a room](#api-room-delete)
  - [Remove a user from a team](#api-user-remove)
  - [Get information about meetings and participants](#api-meetings-info)

Stay tuned, because we're adding more features to the API! Right now we're working on meeting recording and calls/usage data API methods.

# Embedding video calls

To embed a live video call into any web page, just create an iframe that looks like this:

```
  <iframe src="[your-meeting-link]" allow="microphone; camera; autoplay">
```

Here's a [code sample](https://github.com/kwindla/dailyco-api/blob/master/embedding-sample.html). And that sample page [is here live for testing](https://kwindla.github.io/dailyco-api/embedding-sample.html).

## URL parameters

Query string parameters appended to the meeting link can be used to modify the in-call UX:

  - `name=<url-encoded human readable name string>` -- set the visible user name. This user name will also be part of the meeting attendance/analytics data.

# API basics

All API requests are served by: `https://prod-ks.pluot.blue/`

Methods that take argument bodies all expect JSON-formatted data.

All data is returned as JSON. All methods return JSON objects that include the fields documented below, but the objects may have *additional* fields, as well, as we add to the API. Please be lenient in the data you accept, and also don't count on receiving any data other than what is documented here.

We strive to return an HTTP 200 response for every API request. Errors are reported as an `error` field in the JSON-formatted response object. We only return non-200 HTTP responses if there was a network, gateway, or unexpected and unanticipated server error.

To use the API you need a Daily.co developer token. Right now, we're in beta. If you'd like to try out the API, please contact us at help@daily.co. We'll create a token for you.

Each API call will need to include your access token, in an `Authorization: Bearer` HTTP header. For example:

```
curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XGET https://prod-ks.pluot.blue/domains/by-name/my-awesome-team
```

# Specification

[Here is a specification file](dailyco-api-openapi3.yaml) in Swagger/OpenAPI-3.0 format.

# Methods

## <a name="api-team-info"></a>Get invite link, rooms list, and user list for a team

**GET `/domains/by-name/<team-name>`**

```
> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XGET https://prod-ks.pluot.blue/domains/by-name/my-awesome-team
```

### Success

Returns a team invite link, a list of rooms, and a list of users.

```
  { "name": <team-name>,
    "teamInviteLink": <invite-link>, 
    "rooms": [ { "name": <room-name>, "dialInPIN": <string-PIN>, ... }, ... ]
    "users": { <user-email>: { "email": <user-email>,
                               "name": <user-name>,
                               "avatarSrc": <avatar-image-url>,
                               "role": <"owner"|"member">, ... },
               ... }
  }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```

A team invite link looks like `https://d.daily.co/?tk=sCSla44kOd2d`. To be added to a team, a user must click on that link and either create a Daily.co account or log into an existing account. (If a user has already logged in, then the link will just add the user to the team and then redirect straight to the user's Daily dashboard.)

Note that users are guaranteed to have an email address and role, but they may not have provided a name and an avatarSrc.

## <a name="api-room-create"></a>Create a room

Creates a new room. The new room needs a name. You can also optionally supply a privacy setting for the room. Privacy setting values can be: `public`, `org`, or `private`. Privacy defaults to `org`, which is what we call "team" in the Daily.co application UI (our apologies for this divergence in nomenclature). **The combined team name and room name must be 35 characters or less.**

The URL for of the new room (for joining meetings) will be: `https://<team-name>.daily.co/<room-name>`

**POST to `/domains/by-name/<team-name>/room`**

*Request body: `{ "name": <room-name>, "privacy": <optional privacy setting> }`*

```
> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XPOST -d '{"name":"room-a"}' http://prod-ks.pluot.blue/domains/by-name/my-awesome-team/room
```

### Success

Returns information about the room.

```
  { "name": <room-name>,
    "dialInPIN": <string-PIN>
  }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```

## <a name="api-room-update"></a>Change the privacy setting of a room

Changes the privacy setting of a room. Privacy setting values can be: `public`, `org`, or `private`. The `org` setting is what we refer to as "team" in the Daily.co application UI (our apologies for this divergence in nomenclature).

**POST to `/domains/by-name/<team-name>/room/<room-name>`**

*Request body: `{ "privacy": <privacy setting> }`*

```
> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XPOST -d '{"name":"room-a"}' http://prod-ks.pluot.blue/domains/by-name/my-awesome-team/room/my-awesome-room
```

### Success

Returns an object that includes the field `updated: true`

```
  { "updated": true }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```

### <a name="api-room-delete"></a>Delete a room

**DELETE `/domains/by-name/<team-name>/room/<room-name>`**

```
curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XDELETE http://prod-ks.pluot.blue/domains/by-name/my-awesome-team/room/an-old-room
```

### Success

Returns an object that includes the field `deleted: true`.

```
{ "deleted": "true" }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```

## <a name="api-user-remove"></a>Remove a user from a team

Removes a user from the team. (Remember that there is no way to add a user to a team. You can send a team invite link, but the user must click through the link and sign up or log in. You can, however, remove a user from a team at any time, using this API method.)

**DELETE `/domains/by-name/<team-name>/user/<user-email>`**

```
> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XDELETE https://prod-ks.pluot.blue/domains/by-name/my-awesome-team/user/kwindla@gmail.com
```

### Success

Returns an object that includes the field `deleted: true`.

```
{ "deleted": "true" }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```


## <a name="api-meetings-info"></a>Get information about meetings and participants

Retrieves a list of meetings that have happened within a specific timeframe. Optionally, you can also limit the results to meetings in a particular room. Start and end time arguments can either be UNIX timestamps or ISO Date Strings.

**POST `/domains/by-name/<team-name>/meetings`**

**Request body: `{ "start": <time range start time>, "end": <time range end time>, room: <optional room name> }`**

```
> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XPOST -d '{"start":"2018-07-10T20:00:00.000Z", "end":"2018-07-25T00:00:00.000Z"}' https://prod-ks.pluot.blue/domains/by-name/my-awesome-team/meetings
```

### Success

Returns a list of meetings and a boolean `moreResults` flag. Entries in the meetings list include the domain name, room name, meeting start time, meeting end time, and list of participants. Entries in the participants lists include the user's email if the user is logged in (or 'guest', or 'Daily.co TV', if not). They also include meeting join and meeting leave times. All times are accurate to within one minute, but definitely not accurate to the nearest second!

This call returns information about a maximum of 100 meetings. The `moreResults` flag is set to true if more meetings were available for this time range.

```
  { meetings: [ { domain: "my-awesome-team", room: "hello", mtgStart: "2018-07-21T22:07:11.000Z", mtgEnd: "2018-07-21T22:07:29.000Z", participants: [ {email: "guest", mtgJoin: "2018-07-22T20:35:57.000Z", mtgLeave: "2018-07-22T20:38:07.000Z"} ] } ], moreResults: false }
```

### Failure

Returns an error object, with a message field. 

```
  { error: { message: "message" } }
```
