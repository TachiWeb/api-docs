---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the TachiWeb API! You can use the API to perform actions and get information on/from your Tachiyomi library.

<aside class="notice">
Remember to replace all instances of <code>localhost:4567</code> in the code examples with whereever your server is actually running
</aside>

# Authentication `/auth`

> To grab an authentication token, use this code:

```shell
curl "/api/auth" -G
  -d "password=YOUR_PASSWORD"
```

```javascript
TWApi.Commands.Auth.execute(function(result) {
    // You can actually just ignore the result, more on this later
    var apiToken = result.token; 

    console.log("My API token is: " + apiToken);
}, function() {
    console.log("Authentication failed!");
}, {
    password: "YOUR_PASSWORD"
});
```
> Make sure to replace `YOUR_PASSWORD` with the user's password.

Almost every endpoint in the TachiWeb API requires authentication (if the user set a password). To authenticate, you must first obtain an authentication token. Then you must provide the authentication token when performing your API requests. **You do not need a session token if the user set no password.**

<aside class="notice">
Authentication tokens <b>regularly</b> expire (without notice)! Always ensure that your code is able to handle errors thrown when authentication fails! Alternatively, you can just verify your authentication token before every request to ensure you never call the API with an invalid auth token.
</aside>

TachiWeb expects for the token to be included in all API requests to the server if a password is set. You can do this in two ways:

1. Pass it in the `TW-Session` header, example: `TW-Session: 5ujo4i8dvqj84jgt8picvh0sif`
2. Provide a `session` cookie. Example header with session cookie: `Cookie: session=5ujo4i8dvqj84jgt8picvh0sif;`. **This should be the default choice when using the JavaScript client as calling `TWApi.Commands.Auth.execute` will automatically set the session cookie if authentication was successful!**

> Example code that passes the authentication token:

```shell
curl "http://localhost:4567/api/your-operation"
    -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
Do nothing! After you call:

TWApi.Commands.Auth.execute(null, null, { password: "YOUR_PASSWORD" });

the resulting API key is automatically passed to the server on future queries.
```

> Remember to replace `5ujo4i8dvqj84jgt8picvh0sif` with your auth token!

## Verifying Auth Tokens `/test_auth`

> Code examples:

```shell
curl "http://localhost:4567/api/test_auth"
    -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
// Not currently implemented in the JavaScript client
```

You can test auth tokens using this endpoint.

# Manga

## Library manga `/library`

> Sample code to get manga in library:

```shell
curl "http://localhost:4567/api/library"
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.Library.execute(function(result) {
    console.log("Result: ", result);
}, function() {
    console.log("Failed to get library!");
});
```

> The above command returns JSON structured like this:

```json
{
	"success": true,
	"content": [ // List of manga in library
		{
			"chapters": 522,                          // Total number of chapters in manga
			"unread": 522,                            // Number of unread chapters
			"artist": "Lee Gwang Su",
			"author": "Son Jae ho",
			"flags": {
				"DISPLAY_MODE": "NAME",               // Display chapter 'NAME's or 'NUMBER's for this manga?
				"READ_FILTER": "ALL",                 // Show only 'READ', 'UNREAD' or 'ALL' chapters in this manga?
				"SORT_DIRECTION": "DESCENDING",       // Sort chapters in this manga 'ASCENDING' or 'DESCENDING'?
				"SORT_TYPE": "SOURCE",                // Sort by 'SOURCE' order or by chapter 'NUMBER's?
				"DOWNLOADED_FILTER": "ALL"            // Show only 'DOWNLOADED', 'NOT_DOWNLOADED' or 'ALL' chapters in this manga?
			},
			"description": "Lorem ipsum stuff...",
			"source": "ReadMangaToday",               // The name of the source this manga belongs to
			"title": "Noblesse",
			"thumbnail_url": "https://www.readmng.com/uploads/posters/0001_7.jpg",
			"downloaded": false,                      // Whether or not some of the chapters in this manga are downloaded?
			"url": "https://www.readmng.com/noblesse",
			"genres": "Action, Adventure, Comedy, School Life, Shounen, Supernatural",
			"id": 7,                                  // Internal ID of this manga
			"categories": [                           // List of categories this manga is in
                {
                    "id": 1,                          // Internal ID of category
                    "name": "Cool stuff!"             // Name of category
                }
            ],
			"favorite": true,                         // Whether or not this manga is favorited
			"status": "Ongoing"                       // Either 'Ongoing', 'Completed', 'Licensed' or 'Unknown'
		}
	]
}
```

This endpoint fetches all manga in the user's library

## Thumbnail images `/cover`

Use this endpoint to grab the thumbnail image of a manga

> Code examples:

```shell
curl "http://localhost:4567/api/cover/INTERNAL_ID_OF_MANGA"
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
// It is not recommended you use JS to fetch the image
// Instead, just set the src attribute of an img element to:
// http://localhost:4567/api/cover/INTERNAL_ID_OF_MANGA

// Code for those who still want to fetch the image manually:
TWApi.Commands.Cover.execute(null, function() {
    console.log("Could not fetch thumbnail!");
}, {
    mangaId: INTERNAL_ID_OF_MANGA
}, function(xhr) {
    // Process the raw XMLHttpRequest object here
});
```
> Remember to replace `INTERNAL_ID_OF_MANGA` with the internal ID of the manga you wish to fetch the thumbnail of

<!--
### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete
-->
