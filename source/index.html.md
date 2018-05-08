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

<!-- TODO
Mention: TWApi.Commands.Image.buildUrl({
            mangaId: mangaId,
            chapterId: chapterId,
            page: page
        })
        
Describe JS client usages
-->

# Introduction

Welcome to the TachiWeb API! You can use the API to perform actions and get information on/from your Tachiyomi library.

<aside class="notice">
Remember to replace all instances of <code>localhost:4567</code> in the code examples with whereever your server is actually running
</aside>

# Authentication `GET /auth`

> To grab an authentication token, use this code:

```shell
curl "http://localhost:4567/api/auth" -G \
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
curl "http://localhost:4567/api/your-operation" \
    -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
Do nothing! After you call:

TWApi.Commands.Auth.execute(null, null, { password: "YOUR_PASSWORD" });

the resulting API key is automatically passed to the server on future queries.
```

> Remember to replace `5ujo4i8dvqj84jgt8picvh0sif` with your auth token!

## Verifying Auth Tokens `GET /test_auth`

> Code examples:

```shell
curl "http://localhost:4567/api/test_auth" \
    -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
// Not currently implemented in the JavaScript client
```

You can test auth tokens using this endpoint.

# Manga

## Library manga `GET /library`

> Sample code to get manga in library:

```shell
curl "http://localhost:4567/api/library" \
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

## Thumbnail images `GET /cover`

Use this endpoint to grab the thumbnail image of a manga

> Code examples:

```shell
curl "http://localhost:4567/api/cover/INTERNAL_ID_OF_MANGA" \
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

## Toggle favorite status `GET /fave`

Use this endpoint to change the favorite status of a manga (whether or not it should be visible in the library).

> Code examples that favorite a manga

```shell
curl "http://localhost:4567/api/fave/INTERNAL_ID_OF_MANGA" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif" \
  -d "fave=true"
```

```javascript
TWApi.Commands.Favorite.execute(function() {
	console.log("Favorite status changed!");
}, function() {
    console.log("Failed to change favorite status");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
    fave: true
});
```

> Remember to replace `INTERNAL_ID_OF_MANGA` with the internal ID of the manga!

### Query parameters

| Parameter | Type    | Default | Optional | Description                                                                    |
|-----------|---------|---------|----------|--------------------------------------------------------------------------------|
| fave      | Boolean | NONE    | no       | If set to true, manga will be favorited, otherwise manga will be un-favorited. |

## Get chapters `GET /chapters`

Gets a list of a manga's chapters, sorted by the chapter number. Note that this response should be sorted according to the `SORT_TYPE` manga flag before being displayed to the user.

> Code examples:

```shell
curl "http://localhost:4567/api/chapters/INTERNAL_ID_OF_MANGA" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.Chapters.execute(function(result) {
	console.log(result.content);
}, function() {
    console.log("Failed to get chapters!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA
});
```

> Example response:

```json
{
	"success": true,
	"content": [ // List of chapters in manga
		{
			"date": 1430575817626,               // Milliseconds since epoch of when chapter added
			"source_order": 301,                 // Index of chapter (sorted by order it is displayed on source page)
			"read": false,                       // Whether or not chapter is completely read
			"name": "Shokugeki no Soma - 0",     // Name of chapter
			"chapter_number": 0,                 // Chapter number (extracted from chapter name)
			"download_status": "NOT_DOWNLOADED", // Whether or not chapter is "DOWNLOADED", "DOWNLOADING" or "NOT_DOWNLOADED"
			"id": 302,                           // Internal ID of chapter
			"last_page_read": 0                  // The last page read in the chapter (0-indexed)
		},
		{
			"date": 1430575817626,
			"source_order": 300,
			"read": false,
			"name": "Shokugeki no Soma - 1",
			"chapter_number": 1,
			"download_status": "NOT_DOWNLOADED",
			"id": 301,
			"last_page_read": 0
		},
		{
			"date": 1430575817626,
			"source_order": 299,
			"read": false,
			"name": "Shokugeki no Soma - 2",
			"chapter_number": 2,
			"download_status": "NOT_DOWNLOADED",
			"id": 300,
			"last_page_read": 0
		}
	]
}
```

## Get manga info `GET /manga_info`

Gets a manga's info.

> Code examples:

```shell
curl "http://localhost:4567/api/manga_info/INTERNAL_ID_OF_MANGA" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.MangaInfo.execute(function(result) {
	console.log(result.content);
}, function() {
    console.log("Failed to get manga info!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA
});
```

> Example response:

```json
{
	"success": true,
	"content": {  // Same format as the manga objects you get from the "Library manga" endpoint
		"chapters": 302,
		"artist": "SAEKI Shun",
		"author": "TSUKUDA Yuuto",
		"flags": {
			"DISPLAY_MODE": "NAME",
			"READ_FILTER": "ALL",
			"SORT_DIRECTION": "DESCENDING",
			"SORT_TYPE": "SOURCE",
			"DOWNLOADED_FILTER": "ALL"
		},
		"description": "Yukihira Souma's dream is to become a full-time chef in his father's restaurant and surpass his father's culinary skill. But just as Yukihira graduates from middle schools his father, Yukihira Jouichirou, closes down the restaurant to cook in Europe. Although downtrodden, Souma's fighting spirit is rekindled by a challenge from Jouichirou which is to survive in an elite culinary school where only 10% of the students graduate. Can Souma survive?",
		"source": "ReadMangaToday",
		"title": "Shokugeki no Soma",
		"thumbnail_url": "https://www.readmng.com/uploads/posters/1460123867.jpg",
		"downloaded": false,
		"url": "https://www.readmng.com/shokugeki-no-soma",
		"genres": "Ecchi, Shounen",
		"id": 11,
		"categories": [],
		"favorite": true,
		"status": "Unknown"
	}
}
```

## Get chapter page count `GET /page_count`

Get the amount of pages in a chapter.

> Code examples:

```shell
curl "http://localhost:4567/api/page_count/INTERNAL_ID_OF_MANGA/INTERNAL_ID_OF_CHAPTER" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.PageCount.execute(function(result) {
	console.log(result.page_count);
}, function() {
    console.log("Failed to get page count!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
	chapterId: INTERNAL_ID_OF_CHAPTER
});
```

> Example response:

```json
{
	"success": true,
	"page_count": 46
}
```

## Set chapter reading status `GET /reading_status`

Set the new reading status of a chapter.

> Code examples:

```shell
curl "http://localhost:4567/api/reading_status/INTERNAL_ID_OF_MANGA/INTERNAL_ID_OF_CHAPTER" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif" \
  -d "lp=8"
```

```shell
curl "http://localhost:4567/api/reading_status/INTERNAL_ID_OF_MANGA/INTERNAL_ID_OF_CHAPTER" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif" \
  -d "lp=20" \
  -d "read=true"
```

```javascript
TWApi.Commands.ReadingStatus.execute(null, function() {
    console.log("Failed to update reading status!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
	chapterId: INTERNAL_ID_OF_CHAPTER,
    read: true,
    lastReadPage: 8
});
```

### Query parameters

| Parameter | Type    | Default | Optional | Description                                                                    |
|-----------|---------|---------|----------|--------------------------------------------------------------------------------|
| read      | Boolean | NONE    | yes       | If true, manga will be set as 'read', if false manga will be set as 'unread', if null no changes will be made. |
| lp (js: lastReadPage)      | Number | NONE    | yes       | If non-null, update manga's last-read page to it, otherwise, no changes will be made. |

## Set manga flag `GET /set_flag`

Set a manga's flag.

### Valid flag types

| Name | Valid states    | Description                                                                    |
|-----------|---------|--------------------------------------------------------------------------------|
| `SORT_DIRECTION` | `DESCENDING`, `ASCENDING` | Sort chapters in descending or ascending order. |
| `DISPLAY_MODE` | `NAME`, `NUMBER` | Display the chapter name or the chapter number? |
| `READ_FILTER` | `READ`, `UNREAD`, `ALL` | Display only read, only unread or all chapters? |
| `DOWNLOADED_FILTER` | `DOWNLOADED`, `NOT_DOWNLOADED`, `ALL` | Display only downloaded, only not downloaded or all chapters? |
| `SORT_TYPE` | `SOURCE`, `NUMBER` | Sort by the order chapters are displayed on the source page or by their detected chapter numbers? |

> Request format:

```
http://localhost:4567/api/set_flag/INTERNAL_ID_OF_MANGA/NAME_OF_FLAG/NEW_VALUE_OF_FLAG
```

> Example:

```shell
curl "http://localhost:4567/api/set_flag/11/SORT_TYPE/NUMBER" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.SetFlag.execute(null, function() {
    console.log("Failed to set manga flag!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
    flag: "SORT_TYPE",
    state: "NUMBER"
});
```

## Update manga `GET /update`

Update either manga info or manga chapters.

Update type must be either: `INFO` or `CHAPTERS`.

> Updating manga info:

```shell
curl "http://localhost:4567/api/update/INTERNAL_ID_OF_MANGA/INFO" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.Update.execute(null, function() {
    console.log("Failed to update manga info!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
    updateType: "INFO"
});
``` 

> Updating chapters:

```shell
curl "http://localhost:4567/api/update/INTERNAL_ID_OF_MANGA/CHAPTERS" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.Update.execute(function(e) {
	console.log("Added chapters: ", e.added);
	console.log("Removed chapters: ", e.removed);
}, function() {
    console.log("Failed to update manga chapters!");
}, {
	mangaId: INTERNAL_ID_OF_MANGA,
    updateType: "CHAPTERS"
});
``` 

> Example response (only applicable when updating chapters):

```json
{
	"removed": [], // Chapters removed since last update
                   // WARNING! This array (and the added array) contain chapters in a different format than the "Get chapters" endpoint. This will probably be changed soon.
	"added": [     // Chapters added since last update
		{
			"bookmark": false,                     // Whether or not chapte bookmarked
			"source_order": 0,                     // Index of chapter (sorted by order it is displayed on source page)
			"read": false,                         // Whether or not chapter is completely read
			"date_upload": 1522765444992,          // Milliseconds since epoch of when chapter uploaded to source
			"recognizedNumber": true,              // Whether or not chapter number was able to be extracted from the chapter name
			"chapter_number": 542,                 // Chapter number (extracted from chapter name)
			"name": "The Ruler Of The Land - 542", // Name of chapter
			"date_fetch": 1525357445660,           // Milliseconds since epoch of when chapter added to local database
			"manga_id": 2,                         // Internal ID of manga
			"last_page_read": 0,                   // The last page read in the chapter (0-indexed)
			"url": "/the-ruler-of-the-land/542"    // The URL of the chapter in the source
		},
		{
			"bookmark": false,
			"source_order": 1,
			"read": false,
			"date_upload": 1522765444992,
			"recognizedNumber": true,
			"chapter_number": 541,
			"name": "The Ruler Of The Land - 541",
			"date_fetch": 1525357445659,
			"manga_id": 2,
			"last_page_read": 0,
			"url": "/the-ruler-of-the-land/541"
		}
	],
	"success": true
}
```

## Chapter page images `GET /img`

Use this endpoint to get a single image of a chapter

> Code examples:

```shell
curl "http://localhost:4567/api/img/INTERNAL_ID_OF_MANGA/INTERNAL_ID_OF_CHAPTER/0_INDEXED_PAGE_NUMBER" \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
// Note that you do not need to use JS to fetch the image
// Instead, just set the src attribute of an img element to:
// http://localhost:4567/api/img/INTERNAL_ID_OF_MANGA/INTERNAL_ID_OF_CHAPTER/0_INDEXED_PAGE_NUMBER

// Code for those who still want to fetch the image manually:
TWApi.Commands.Image.execute(null, function() {
    console.log("Could not fetch image!");
}, {
    mangaId: INTERNAL_ID_OF_MANGA,
    chapterId: INTERNAL_ID_OF_CHAPTER,
    page: 0_INDEXED_PAGE_NUMBER
}, function(xhr) {
    // Process the raw XMLHttpRequest object here
});
```

# Catalogues
## List sources `GET /sources`

An endpoint used to list installed sources.

### Query parameters

| Parameter | Type    | Default | Optional | Description                                                                    |
|-----------|---------|---------|----------|--------------------------------------------------------------------------------|
| enabled   | Boolean | false    | yes       | If true, will only return enabled sources, otherwise will return all installed sources. |

> Code examples:

```shell
curl "http://localhost:4567/api/sources" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif" \
  -d "enabled=true"
```

```javascript
TWApi.Commands.Sources.execute(function(result) {
	console.log(result.content);
}, function() {
    console.log("Failed to get enabled sources!");
}, {
	enabled: true
});
```

> Example response:
```json
{
	"success": true,
	"content": [  // List of sources
		{
			"name": "Local manga",   // Name of source
			"supports_latest": true, // Whether or not the source supports the latest updates screen
			"id": 0,                 // Internal ID of source
			"lang": {                // Language of source
				"name": "",          // ISO 639-1 country code of language
				"display_name": ""   // Name of language (in the language itself)
			}
		},
		{
			"name": "Mangahere",
			"supports_latest": true,
			"id": 2,
			"lang": {
				"name": "en",
				"display_name": "English"
			}
		}
	]
}
```

## Get filters `GET /get_filters`

Call this endpoint to grab a list of available filters for a source.

> Code examples:

```shell
curl "http://localhost:4567/api/get_filters/INTERNAL_ID_OF_SOURCE" -G \
  -H "TW-Session: 5ujo4i8dvqj84jgt8picvh0sif"
```

```javascript
TWApi.Commands.GetFilters.execute(function(result) {
	console.log(result.content);
}, function() {
    console.log("Failed to get filters!");
}, {
	id: INTERNAL_ID_OF_SOURCE
});
```

> Example response:
```json
{
	"success": true,
	"content": [  // List of filters (MUST be rendered in order on the page)
		{
			"_cmaps": {                     // Data on what Java types each JSON field is for internal use. 
                                            // DO NOT USE THIS as it can be removed/changed at any time
                                            // without warning.
                                            // This data MUST NOT be modified and MUST be retained when calling the search API with the filters JSON.
				"name": "java.lang.String"  // Example: The 'name' field is a Java String type
			},
			"name": "Lorem ipsum",          // The name of the header
			"_type": "HEADER"               // The type of the filter
                                            // The HEADER filter is a single line of non-editable text used to provide information to the user
		},
		{
			"_cmaps": {
				"name": "java.lang.String"
			},
			"name": "",                     // Should be ignored in separator filters
			"_type": "SEPARATOR"            // The SEPARATOR filter is a horizontal line used to divide two sections of filters
		},
		{
			"_cmaps": {
				"name": "java.lang.String",
				"state": "java.lang.Boolean"
			},
			"name": "Show R-18",            // The name of the filter
			"state": true,                  // The initial state of the filter
			"_type": "CHECKBOX"             // The CHECKBOX filter is a two-state, toggleable filter
		},
		{
			"_cmaps": {
				"name": "java.lang.String",
				"state": "java.lang.String"
			},
			"name": "Author",               // The name of the filter
			"state": "",                    // The initial state of the filter
			"_type": "TEXT"                 // The TEXT filter is a single-line, editable text filter
		},
		{
			"values": [                     // All possible values of the filter
				"All",
				"Japanese Manga",
				"Korean Manhwa",
				"Chinese Manhua"
			],
			"_cmaps": {
				"name": "java.lang.String",
				"state": "java.lang.Integer"
			},
			"name": "Type",                 // The name of the filter
			"state": 0,                     // The initial state of the filter
			"_type": "SELECT"               // The SELECT filter is a drop-down, single-selection filter
                                            // One (no more, no less) item can be selected at a time
                                            // It's state is the index of the selected item (0-indexed)
		},
		{
			"_cmaps": {
				"name": "java.lang.String",
				"state": "java.lang.Integer"
			},
			"name": "Completed",            // The name of the filter
			"state": 0,                     // The initial state of the filter
			"_type": "TRISTATE"             // The TRISTATE filter is a triple-state filter
                                            // Users can cycle through the three states:
                                            // +-------+---------+
                                            // | Index | State   |
                                            // +-------+---------+
                                            // | 0     | IGNORE  |
                                            // | 1     | INCLUDE |
                                            // | 2     | EXCLUDE |
                                            // +-------+---------+
		},
		{
			"state": [
				{
					"_cmaps": {
						"name": "java.lang.String",
						"state": "java.lang.Integer"
					},
					"name": "Action",
					"state": 0,
					"_type": "TRISTATE"
				}
			],
			"_cmaps": {
				"name": "java.lang.String"
			},
			"name": "Genres",               // The name of the group
			"_type": "GROUP"                // The GROUP filter is used to group multiple filters together
                                            // Although nested groups are currently not used, they may be supported in the future
		},
		{
			"values": [                     // All the different attributes we can sort by
				"Series name",
				"Rating",
				"Views",
				"Total chapters",
				"Last chapter"
			],
			"state": {                      // The initial state of the filter
				"index": 2,                 // The current attribute we are sorting by
				"ascending": false          // Are we sorting the attribute ascending or descending?
			},
			"_cmaps": {
				"name": "java.lang.String"
			},
			"name": "Order by",             // The name of the filters
			"_type": "SORT"                 // The SORT filters allows the user to sort by one (no more, no less) specific attribute
                                            // The attribute can be sorted ascending or descending
		}
	]
}
```

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
