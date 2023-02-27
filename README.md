# Quick start guide

Install the [Fictioneers JavaScript SDK](https://github.com/fictioneers/node-sdk) using the package manager of your choice:

```
npm install fictioneers
```

or

```
yarn add fictioneers
```

## Create an API client

Next initialize an API client.

If your code is running in a trusted environment (e.g. a backend service) you are safe to pass a secret key for the `apiKey` parameter - otherwise you should reference a visible key.

You can read more about the [different types of API keys in the API documentation](https://docs.fictioneers.co.uk/#authentication).

```js
import { Fictioneers } from "fictioneers";

const fictioneersApiClient = new Fictioneers({
    apiKey: "s_xxxxx"
})
```

## Create a User

When using the [Audience API](https://docs.fictioneers.co.uk/#fictioneers-apis-audience-api) all requests are scoped to a specific audience user on a particular published timeline.

```js
// get the published timeline ID from the Fictioneers UI
const publishedTimelineID = 'bNN98PD6nUfa18z9LO6Z';

// set the user ID on the API client
fictioneersApiClient.setUser('123456789');

// create the user over API
const createUserResponse = await fictioneersApiClient.createUser({
    publishedTimelineID,
});
```

## Get User Timeline Events

`UserTimelineEvents` are the core resources on a user timeline.

```js
const userTimelineEventsResponse = await fictioneersApiClient.getUserTimelineEvents();
```

They are stateful objects, which can hold references to content, and have multiple links to other `UserTimelineEvents`.

## Follow a Link

These links represent an implicit or explicit choice the user must take on their journey along the timeline, and afford the end user choice and agency in their experience.

```js
const userTimelineEventsResponse = await fictioneersApiClient.followLinkUserTimelineEvent(
  fromEventID,
  linkID
 );
```

Following a link will change the state of the related `UserTimelineEvents`:

* The `from` event will move from `ACTIVE` to `VISITED` state.
* The `to` event will move from `UNVISITED` to `ACTIVE` state.

These changes are available in the `userTimelineEventsResponse.meta.changed_timeline_events` array.

## Best Practices

Recommended best practices to implement when building an integration:

### Re-use API Client

There is a small performance win if you re-use this object instead of re-initializing for an existing user, because the API client object handles the lifecycle and expiry of access tokens for you.

### Response Meta

To reduce the amount of API calls an integration requires, the Audience API returns a JSON response with three keys:

```json
{
    "data": {}, // the main resource
    "error": {}, // details if something goes wrong
    "meta": {}, // all updated object references to keep local state in sync 
}
```

This removes the need to chain API calls. For example if you were to call `followLinkUserTimelineEvent`, you might be tempted to chain calls to `getUserTimelineEvents` and `getUserStoryState`.

Instead you can avoid these subsequent API calls and instead update your local state with references from `meta.changed_timeline_events` and `meta.user_story_state`.
