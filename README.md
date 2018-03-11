Using the Daily.co API, you can manage domains, teams, and video call rooms.

There are currently five API methods:
  - [Get invite link, rooms list, and user list for a team](api-team-info)
  - Create a room
  - Delete a room
  - Remove a user from a team
  - Create a new team (a new Daily.co domain)

Stay tuned, because we're adding more features to the API! Right now we're working on meeting recording and calls/usage data API methods. 

# API basics

All API requests are served by: `https://prod-ks.pluot.blue/`

Methods that take argument bodies all expect JSON-formatted data. All data is returned as JSON. All methods return JSON objects that include the fields documented below, but the objects may have **additional** fields, as well, as we add to the API. Please be lenient in the data you accept, and also don't count on receiving any data other than what is documented here.

To use the API you need a Daily.co developer token. Right now, we're in beta. If you'd like to try out the API, please contact us at help@daily.co. We'll create a token for you.

Each API call will need to include your access token, in an `Authorization: Bearer` HTTP header. For example:

```
curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XGET https://prod-ks.pluot.blue/domains/by-name/my-awesome-team
```

# Methods

## <a name="api-team-info"></a>Get invite link, rooms list, and user list for a team

*GET `/domains/by-name/<team-name>`*

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

A team invite link looks like "https://d.daily.co/?tk=sCSla44kOd2d". To be added to a team, a user must click on that link and either create a Daily account or log into an existing account. (If a user has already logged in, then the link will just add the user to the team and then redirect straight to the user's Daily dashboard.)

Note that users are guaranteed to have an email address and role, but they may not have provided a name and an avatarSrc.

## Create a room

*POST to `/domains/by-name/<team-name>/room`*

*Request body: `{ "name": <room-name> }`*

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





1. Create a new team (a new Daily domain). The new team will be owned by your user.

POST /domains/claim
json request body: { "domain": <team-name> }


> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XPOST -d '{"domain":"team-1"}' https://prod-ks.pluot.blue/domains/claim

success: returns a bunch of opaque information about the domain
  { name: "team-1", ... }
failure: returns { error: { message: } }


2. get team invite link and list all rooms and members

GET /domains/by-name/<team-name>


> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XGET https://prod-ks.pluot.blue/domains/by-name/team-1

success: returns a bunch of useful information about the domain
  { "name": <team-name>,
    "teamInviteLink": <invite-link>, 
    "rooms": [ { "name": <room-name>, "dialInPIN": <string-PIN>, ... }, ... ]
    "users": { <user-email>: { "email": <user-email>,
                               "name": <user-name>,
                               "avatarSrc": <avatar-image-url>,
                               "role": <"owner"|"member">, ... },
               ... }
  }
failure: returns { error: { message: } }

Note that users are guaranteed to have an email address and role, but will only have a name and an avatarSrc if they signed into Daily using Google oauth. (For now. We will build an account settings panel that lets users specify name and avatar inside Daily, at some point.)

A team invite link looks like "https://d.daily.co/?tk=sCSla44kOd2d". To be added to a team, a user must click on that link and either create a Daily account or log into an existing account. (If a user has already logged in, then the link will just add the user to the team and then redirect straight to the user's Daily dashboard.)

3. create a room
POST to /domains/by-name/<team-name>/room
json request body: { "name": <room-name> }

Curl command (assuming that "team-1" is the name of an existing team and $TOKEN is your API token).


> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XPOST -d '{"name":"room-a"}' http://prod-ks.pluot.blue/domains/by-name/team-1/room

success: returns information about the room
  { "name": <room-name>, "dialInPIN": <string-PIN>, ... }
failure: returns { error: { message: } }

4. remove member from domain

DELETE /domains/by-name/<team-name>/user/<user-email>


> curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XDELETE https://prod-ks.pluot.blue/domains/by-name/team-1/user/kwindla@gmail.com

success: returns { deleted: true }
failure: returns { deleted: false } or { error: { message: } }

5. delete a room

DELETE /domains/by-name/<team-name>/room/<room-name>

curl -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" -XDELETE http://prod-ks.pluot.blue/domains/by-name/team-1/room/room-a

success: returns { deleted: true }
failure: returns { deleted: false } or { error: { message: } }
