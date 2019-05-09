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

![](/images/logo.png)

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

# Using the API

```http
POST "https://api.sponsus.org/v1/profiles/@me"
```
> Will return an error if you did not set your Authorization header correctly

```json
{
  "success": false,
  "error": "Authorization header not found in request"
}
```

When the API responds to your command, it will always return a field called `success`. This field is used to know if a request was successful or not.
The API base URL is `https://api.sponsus.org`. It features a versioning system that allows you to upgrade and downgrade as needed. When we are forcing an upgrade, we will give plenty of warning. 

The full URL is:

`https://api.sponsus.org/v1/<route here>`

For example:

`https://api.sponsus.org/v1/profiles/<userID>/avatar`

# Changelog

### 09/05/19 - Updated the docs
Added more information about the profile objects as well as some routes that might be useful with this object. Added documentation about the Profile getter as well as a private endpoint for finding the Sponsorship status of a user. Requires your API token.

### 09/05/19 - Change to Profile responses
Profile responses have been fixed to only return the fields as noted in the docs. There was an issue where old Profile objects still had old fields that were no longer being used.

# OAuth

>  The implicit grant (response type "token") and other response types
>  causing the authorization server to issue access tokens in the
>  authorization response are vulnerable to access token leakage and
>  access token replay as described in Section 3.1, Section 3.2,
>  Section 3.3, and Section 3.6.
>
>  In order to avoid these issues, Clients SHOULD NOT use the implicit
>  grant and any other response type causing the authorization server to
>  issue an access token in the authorization response.
> <br />- *OAuth 2.0 Security Best Current Practice, Nov 2018*

OAuth is a simple way to publish and interact with protected data. It's also a safer and more secure way for people to give you access. We've kept it simple to save you time. 

We do not support the implict flow as it is insecure. As stated in "OAuth 2.0 Security Best Current Practice". Our priority is user security and using an insecure method of logging is something we dont want to allow

Read more about it [here](https://tools.ietf.org/html/draft-ietf-oauth-security-topics-09#section-2.1.2)

### OAuth URLs
URL | Description
--- | -----------
/oauth/authorize | 	Base authorization URL
/oauth/token | Token URL
/oauth/token/revoke | Revoke an access_token

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
If you need help generating this url, there is a tool that generates this for you in the OAuth [application edit page](https://sponsus.org/developers/oauth). 

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

### An example of the page you should get:
![](/images/oauth_2_example.png)

If you see this page, that means that all checks have passed and the user is allowed to authorize this app.

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
POST https://api.sponsus.org/v1/oauth/token

code=<code param from step 3>
&grant_type=authorization_code
&client_id=<client ID>
&client_secret=<client secret>
&redirect_uri=<redirect_uri that was used in step 2>
```

> The API will respond with this if the request was successful

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

## 5) Using the access_token

> Example usage

```shell
curl "https://api.sponsus.org/v1/oauth/@me/profile"
  -H "Authorization: Bearer <access_token>"
```

> Read the documentation for the OAuth API [here](#oauth-endpoints)

Now that you have an access token, you can use any of the OAuth routes while using that as the `Authorization` token.

<aside class="notice">
    Any endpoints with `@me` in the route mean that you can only edit the owner of the access token. This is a security measure done across the entire Sponsus API
</aside>

## Keeping in touch

> OAuth access token refresh<br/>Note the change in grant_type to "refresh_token"

```http
POST https://api.sponsus.org/v1/oauth/token

refresh_token=<refresh_token from step 3>
&grant_type=refresh_token
&client_id=<client ID>
&client_secret=<client secret>
&redirect_uri=<redirect_uri that was used in step 2>
```

> The API will respond with this if the request was successful

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

> The token the refresh is for will be deleted from the API and forgotten


Access tokens have a lifetime of 1 month and once that time has ran out it is no longer useable. If you wish to keep providing services for the access token you can request a new one using the `refresh_token` you got given during step 4.

## Logging out

> OAuth access token revoke

```http
POST https://api.sponsus.org/v1/oauth/token/revoke

access_token=<the users access token>
&client_id=<client ID>
&client_secret=<client secret>
```

> The API will respond with this if the request was successful

```json
{
    "success": true
}
```

> The token will be deleted from our servers


# OAuth Endpoints

## Authentication

> Example of an OAuth request

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: access_token"
```

This collection of routes is for endpoints related to the OAuth login system. You must use a VALID OAuth access token from Sponsus that is also not expired.
To get an access_token for use with Sponsus, please use the authorization code flow shown [here](#oauth). 

## Get Profile

```shell
curl "https://api.sponsus.org/v1/oauth/@me/profile"
  -H "Authorization: Bearer <access_token>"
```

> If the request was successful, you should see something similar to this:

```json
{
    "success": true,
    "profile": {
        "description": ":ghost: I need to set a description!",
        "about": "I have no description! I should go to my dashboard...",
        "status": "public",
        "username": "OAuthTest",
        "_id": "1737920327297667072"
    }
}
```

This endpoint gets the current logged in users [Profile](#profile) object. 

As with all endpoints in this collection, this request will return a JSON formatted response. If the field `success` is `false`, then there is a field called `error` which denotes what happened and what went wrong with your request

### Scopes
This endpoint requires the following scopes: `identify`

# API: Profiles

## Get Profile

```http
GET "https://api.sponsus.org/v1/profiles/<userID OR unique_url>"

e.g. GET "https://api.sponsus.org/v1/profiles/cerulean"
```

> If the request was successful, you should see something similar to this:

```json
{
    "success": true,
    "profile": {
        "description": ":ghost: I need to set a description!",
        "about": "I have no description! I should go to my dashboard...",
        "status": "public",
        "username": "Cerulean",
        "_id": "1737920327297667072"
    }
}
```

Get a users profile information. This is public information and does not require authentication.


# API: Payments

This area is all about the payment system which includes donations, tiers, sponsorships and more.

## Get User's sponsorship status

```http
GET "https://api.sponsus.org/v1/payments/<userID>/sponsoring"
    Headers:
        Authorization: <API Token>
```

> If the request was successful, you should see something similar to this:

```json
{
    "success": true,
    "sponsorship": {
        "userID": 1737920327297667072,
        "created_at": 1557425238.9788720608,
        "targetID": 1729788214794915840,
        "is_active": true,
        "tier": {
            "_id": 1736611689748631552,
            "title": "Tier title!",
            "price": 1,
            "description": "Sponsor me and get really cool reward...",
            "userID": 1729788214794915840,
            "advanced": {},
            "created_at": 1557014705.5216090679,
            "is_active": true
        },
        "has_paid": false
    }
}
```

> Active donations can be grabbed by adding ?include_donations=true to the end of the URL

```json
...
"has_paid": true,
"donations": {
    "total": 50,
    "active_amount": 5
}
```

<aside class="notice">
    You are going to need to set the Authentication header for this to work!
</aside>

This will get the users sponsorship status with you. Requires your API token in order to get the sponsorship status.

### Query params

Name | Description 
---- | -----------
include_donations | Includes an `active_donations` field which denotes how much this user has donated to you as well as any donations that are still vaid as set by your dashboard.

### Examples

Replace \<userID> with the ID of the user you wish to grab the information for.

`GET https://api.sponsus.org/v1/payments/<userID>/sponsoring?include_donations=true`<br/>
`GET https://api.sponsus.org/v1/payments/<userID>/sponsoring`<br/>

# Objects

These are the various objects used in Sponsus's API. This contains both the main API and the objected used in the OAuth API.

## Profile

> Example object

```json
{
    "success": true,
    "profile": {
        "description": ":ghost: I need to set a description!",
        "about": "I have no description! I should go to my dashboard...",
        "status": "public",
        "username": "OAuthTest",
        "_id": "1737920327297667072"
    }
}
```

Name | Type | Description
---- | ---- | -----------
_id | int (Snowflake) | The ID of the user. Unique
username | string | The username of the user.
status | string | A placeholder flag used to denote if the profile is to be hidden from search and viewing.
about | string | Markdown text about the user. Used to let people know what this user is about.
description | string | Description of the user, used for the short description on the side bar. Used to quickly showcase who this user is.

## Sponsorship

> Example Sponsorship object

```json
"sponsorship": {
    "userID": 1737920327297667072,
    "created_at": 1557425238.9788720608,
    "targetID": 1729788214794915840,
    "is_active": true,
    "tier": {
        "_id": 1736611689748631552,
        "title": "Tier title!",
        "price": 1,
        "description": "Sponsor me and get really cool reward...",
        "userID": 1729788214794915840,
        "advanced": {},
        "created_at": 1557014705.5216090679,
        "is_active": true
    },
    "has_paid": false
}
```

Name | Type | Description
---- | ---- | -----------
userID | int (Snowflake) | The ID of the user.
targetID | Snowflake | The ID of the creator this is going towards
created_at | UTC timestamp | The UTC time when the user started this sponsorship
is_active | bool | False if the user has canceled the sponsorship BUT has paid for the remaining of the month
tier | Tier Object | The tier that this sponsorship is for
has_paid | Bool | True if the user has paid for this month. You can grab the amount sponsored via the tier
**Optional** | | 
donations | Donations Object | The total and active donations for this user towards you 