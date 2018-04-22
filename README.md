# talk-plugin-gravatar

[Demo](https://snorre.io/setting-up-coral-talk/?commentId=a0db735b-4822-44a2-a1d1-bfba4839e344)

A simple Talk plugin to display Talk users' Gravatar avatars with as much
privacy as possible. The plugin acts as a proxy to gravatar and never directly
reveal the md5sum of a talk user's email. This avoids two key privacy concerns
normally faced when using gravatar:

1. Leaking emails via md5sums of said emails

By adding Gravatar image URLs to your site you are effectively leaking a lot of
information. The md5sum itself will be searchable and might reveal which other
sites the user registered for. md5sums are also trivially bruteforceable, and
any reverse lookup databases of emails that have been part of databreaches will
be even easier to look up.

2. Referer header passed to Gravatar

When a user's browser loads a Gravatar URL it will also send a referer header.
This header will reveal the exact page from which the avatar was loaded. This
info can be used by Gravatar to see which sites a user is posting on.

This plugin was inspired by the blog post
[How Gravatar hurts your visitors](https://fly.io/articles/how-gravatar-hurts-your-visitors/).

## Installation

Add the plugin to the `plugins.default.json` file or create a `plugins.json`
file and add it there. See the [Coral Talk plugin docs](https://docs.coralproject.net/talk/plugins/)
for details about adding plugins.


Example `plugins.json` file:

```json
{
  "server": [
    {"talk-plugin-gravatar": "^0.1.0"}
  ],
  "client": [
    {"talk-plugin-gravatar": "^0.1.0"}
  ]
}
```

Then run `./bin/cli plugins reconcile` for source installs or an appropriate
command for your talk installation to make the plugin available.

## How the plugin works

The plugin works by adding an avatar property to the `User` type in the graphql
schema. When the avatar property resolver is called it will check if the user
model in the database contains an avatar value, if not an uuid is created and
stored. Finally the avatar resolver will return the uuid.

Next the client Avatar component will add an `img` element to a comment
referencing the plugin's avatar endpoint: `<your-talk-domain>/avatar/:avatar`.
The request handler in the plugin will first check if the avatar with the given
id is cached in Talk's Redis cache. Cached avatars are immediately returned. If
an avatar is not cached the request handler will fetch it from Gravatar and
then cache it for 24 hours. The request handler also returns an ETag header
based on an md5sum of the avatar contents to save bandwith.

Caching is set to 24 hours expiry time to avoid hammering Gravatar with
requests from the API. 

## Development

Coming soon
