---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://sponsus.org/developers/oauth'>Create an OAuth app</a>

includes:
  - errors

search: true
---

# Introduction

![](https://docs-dev.sponsus.org/images/logo.png)

Sponsus is a new Social Sponsorship platform being developed from the ground up with the needs of the creator and supporter in mind! As a creator, you can share your content, and receive sponsorship from people interested in supporting your work, and provide rewards in a tier system based on how much a sponsor wishes to to give! As a user, you can sponsor your favorite creators and help them achieve their dreams through an easy to use interface that charges automatically without needing micromanagement. Manage your sponsorships through an very easy to use panel, and receive recognition for your awesomeness from the creators through a robust and customizable plethora of tools we provide to the creators! 

<aside class="notice">
    The Sponsus API is still being built. The API is in a stable state but may return 500 errors for newer routes. This is so that we can work out all of the bugs and make sure it returns the right value. If you get some of these errors, please contact support and we can help fix it for you!
</aside>

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: api_key"
```

> Make sure to replace `api_key` with your API key.

Sponsus uses API keys to allow access to the API. You can create a new API key [here](https://sponsus.org/developers/keys)

For all requests that feature `@me` or are personal to the account holder, Sponsus expects the Authorization header to be set like this:

`Authorization: api_key`

<aside class="notice">
  You must replace <code>api_key</code> with your personal API key.
</aside>

# OAuth

## 1) Create an application

Before you can connect your website to Sponsus, you need to create an application. [Click here to go there now](https://sponsus.org/developers/oauth).
Keep hold of the client ID and client secret as these will be needed to verify things later


## 2) Redirect to our OAuth url

> To authorize, use this url:

```http
GET https://sponsus.org/oauth/authorize
    ?response_type=code
    &client_id=<clientID>
    &redirect_uri=<one of your redirect URIs listed in your app>
    &scope=identify,sponsoring
```

> Make sure to fill in your details before redirecting the user to this url!

Once the application has been made, we can start to get users connected to your website in no time! 
To activate the OAuth flow, go to this route:

### HTTP Request

`GET https://sponsus.org/oauth/authorize`

### Query Parameters

Parameter | Description
--------- | -----------
***Required*** <br /> response_type | OAuth grant type. Set this to code.
***Required*** <br /> client_id | Your client ID
***Required*** <br /> redirect_uri | One of your redirect_uris that you provided in step 1
***Required (at least one)*** <br /> scope | This parameter will allow your app to read/write to the given scope. For a list of valid scopes, [click here!](#scopes). It will be displayed to the user in human-friendly terms when signing in with Patreon
state | This will be added to the redirect URI when the user has authorized or denied the OAuth flow.

## 3) Handle the OAuth callback

> An example redirect URL:

```http
GET https://example.com/callback
    ?code=<code>
    &state=<state>
```

Once the user has authorized (or denied) the OAuth request, they are redirected back to the site defined under `redirect_uri`.
This redirect will have some parameters that you need to complete the OAuth flow such as the `code` and your `state` if you set that during step 2

### Query Parameters

Parameter | Description
--------- | -----------
code | The temporary authorization code that is given back to us in step 4<br>Be aware that this has a expire time of 1 minute
state | If set during step 2, this can be a way of identifying a request or user


## 4) Validate the OAuth code

> OAuth access token retrevial

```http
POST https://api.sponsus.org/v1/oauth2/token

code=<code param from step 3>
&grant_type=authorization_code
&client_id=<client ID>
&client_secret=<client secret>
&redirect_uri=<redirect_uri that was used in step 2>
```

> The API will respond with th  is if the request was successful

```json
{
    "access_token": "3AQQoI56Z1CEDC7F3SVjfkQ892AVvvOj",
    "refresh_token": "Vm8hrWGtkgNEVqKDRVt9bv1Ff6jvfLXp",
    "scope": [
        "identify",
        "sponsoring"
    ],
    "expires_in": 2592000,
    "success": true
}
```

Make sure to also include this header: `Content-Type: application/x-www-form-urlencoded`

<aside class="warning">
  In accordance with RFC 6749, the token URL only accepts a content type of x-www-form-urlencoded. JSON content is not permitted and will return an error.
</aside>

Your server should handle GET requests in Step 3 by performing the following request on the server (not as a redirect):

### HTTP Request

`POST https://api.sponsus.org/v1/oauth/token`

**Please note that anything past this point is part of the API and must use the API url**

### Query Parameters

Parameter | Description
--------- | -----------
***Required*** <br /> code | The code you got from the redirect in step 3
***Required*** <br /> grant_type | Set this to `authorization_code`
***Required*** <br /> client_id | Your client ID
***Required*** <br /> client_secret | Your client secret
***Required*** <br /> redirect_uri | The redirect URI you used for step 2

# Profiles
