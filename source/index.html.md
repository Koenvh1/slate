---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python

toc_footers:
  - <a href='https://play.asti.ga/privacy.html'>Privacy policy</a>
  - <a href='https://play.asti.ga/terms.html'>Terms of service</a>
  - <a href='https://github.com/lord/slate' target='_blank'>Documentation Powered by Slate</a>

includes:

search: true
---

# Introduction

Astiga implements the [Subsonic](https://subsonic.org) API for connection to third-party apps, and is largely Subsonic compatible.
Some slight alterations have been made to the Subsonic API, so not all functions may work the same way. 

Every request to the Astiga API has to be either a HTTP POST request or HTTP GET request.  
All endpoints below assume `https://play.asti.ga/rest/` as base URL, and `.view` as extension (though this may be omitted).
So an example URL would be `https://play.asti.ga/rest/ping.view`.

# Required parameters
Every request to an API endpoint must contain the following parameters, either as query parameters or as POST body encoded with `x-www-form-urlencoded`:

Parameter | Required | Default | Comment  
--------- | -------- | ------- | -------  
u | Yes | | The email address used to sign up for Astiga.  
p | Yes* | | The password, either in clear text or hex-encoded with a "enc:" prefix. 
t | Yes* | | The authentication token.
s | Yes* | | A random salt used for the password hash.
v | Yes | | The protocol version implemented by the client, currently anything larger than 1.0.0 is accepted, though 1.16.0 is preferred.
c | Yes | | A unique string identifying the client application.
f | No | xml | Data type to be returned from the API, either `xml`, `json` or `jsonp`.
callback | No | | In case `f` is `jsonp`, provide the name of the callback function here.

# Authentication

> To use the token authentication, use this code:

```python
import hashlib

s = "42"
t = hashlib.md5(password + s).hexdigest()  # Make sure that t is lowercase
```

> To use hex authentication, use this code:

```python
p = "enc:" + password.encode('utf-8').hex()  # Make sure that p is lowercase
```

> To use plain authentication, use this code:

```python
p = password 
```
<aside class="notice">
Even though Subsonic recommends using token authentication, hex or plain authentication are preferred for Astiga.
</aside>

Astiga supports three different types of authentication: Token, hex-encoded and plain.
In the case of plain or hex-encoded authentication, provide the `p` parameter with either the password or `enc:` + the hex encoding of the password.
In the case of token encoding, generate a (random) value `t`, and generate the md5 hash of the password + `t`, and send that as the `t` parameter.

Using either hex-encoded or plain authentication is preferred. Because Astiga forces HTTPS, these details are already transmitted encrypted.
The `password` can in the case of hex-encoded or plain authentication be either the user's password or the token as listed on the apps page.
In the case of token authentication, the `password` has to be the token listed on the apps page.


# Formats

## General structure

> The XML structure looks like this:

```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <subsonic-response status="failed" version="1.16.0" xmlns="http://subsonic.org/restapi">
    <error code="40" message="Wrong username or password. Make sure to use the Subsonic token."/>
  </subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "failed",
    "version": "1.16.0",
    "error": {
      "code": 40,
      "message": "Wrong username or password. Make sure to use the Subsonic token."
    }
  }
}
```
<aside class="notice">
XML is the default format used by the Astiga API. It follows the <a href="http://www.subsonic.org/pages/inc/api/schema/subsonic-rest-api-1.16.0.xsd">Subsonic XSD</a> schema for the most part, though there may be some alterations.
</aside> 

Every response is wrapped in a subsonic-response, which contains a status and version as children. In case status is not `ok`, but `failed`, it will contain an `error` with code and message. 

## Errors

Code | Description
---- | -----------
10 | Required parameter is missing.
20 | Incompatible Subsonic REST protocol version. Client must upgrade.
40 | Wrong username or password.
50 | User is not authorized for the given operation.
70 | The requested data was not found.

# System

## ping

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi"/>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Used to test connectivity with the server. Takes no extra parameters.

### Query Parameters

None

## getLicense

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <license valid="true" email="dummy@asti.ga" licenseExpires="2020-12-22T21:18:20"/>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "license": {
      "valid": true,
      "email": "dummy@asti.ga",
      "licenseExpires": "2020-12-22T21:18:39"
    }
  }
}
```

Dummy endpoint for compatibility with Subsonic clients. Will always return valid as true, dummy@asti.ga as email (it is not a functional email), and the license expiration date is always one year ahead of the current time.

### Query Parameters

None

# Browsing

## getMusicFolders

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <musicFolders>
  <musicFolder id="709000" name="Backblaze Demo"/>
  <musicFolder id="107345" name="MyDrive demo"/>
 </musicFolders>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "musicFolders": {
      "musicFolder": [
        {
          "id": "709000",
          "name": "Backblaze Demo"
        },
        {
          "id": "107345",
          "name": "MyDrive demo"
        }
      ]
    }
  }
}
```

Returns all configured top-level music folders (storages). 

### Query Parameters

None

## getIndexes

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <indexes lastModified="1550452788000" ignoredArticles="The">
  <index name="L">
   <artist id="470418" name="LAPFOX TRAX" coverArt="406191"/>
  </index>
  <index name="O">
   <artist id="173160" name="October" coverArt="406191"/>
  </index>
  <index name="P">
   <artist id="470419" name="Portal Stories Mel" coverArt="406191"/>
  </index>
  <child id="3506290" parent="173205" title="Rick Astley - Never Gonna Give You Up (8 bit Remix)" artist="" artistId="2610" album="" albumId="173205" genre="" coverArt="30919" size="9686274" contentType="audio/mpeg" suffix="mp3" duration="242" path="Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3" type="music" isDir="false" bitRate="320" created="2019-12-22T21:26:12" year="2016"/>
 </indexes>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "indexes": {
      "lastModified": 1550452788000,
      "ignoredArticles": "The",
      "index": [
        {
          "name": "L",
          "artist": [
            {
              "id": "470418",
              "name": "LAPFOX TRAX",
              "coverArt": "406191"
            }
          ]
        },
        {
          "name": "O",
          "artist": [
            {
              "id": "173160",
              "name": "October",
              "coverArt": "406191"
            }
          ]
        },
        {
          "name": "P",
          "artist": [
            {
              "id": "470419",
              "name": "Portal Stories Mel",
              "coverArt": "406191"
            }
          ]
        }
      ],
      "child": [
        {
          "id": "3506290",
          "parent": "173205",
          "title": "Rick Astley - Never Gonna Give You Up (8 bit Remix)",
          "artist": "",
          "artistId": "2610",
          "album": "",
          "albumId": "173205",
          "genre": "",
          "coverArt": "30919",
          "size": "9686274",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 242,
          "path": "Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-22T21:26:09",
          "year": 2016
        }
      ]
    }
  }
}
```

Returns an indexed structure of all top-level content.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
musicFolderId | No | | If specified, only return artists in the music folder with the given ID. See getMusicFolders.
ifModifiedSince | No | | If specified, only return a result if the artist collection has changed since the given time (in milliseconds since 1 Jan 1970).

## getMusicDirectory

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <directory id="173160" name="October">
  <child id="3506291" parent="173160" title="Art" isDir="true"/>
  <child id="3506292" parent="173160" title="The creation of all things" artist="Soft and Furious" artistId="173127" album="October" albumId="176263" genre="" coverArt="173129" size="4010903" contentType="audio/mpeg" suffix="mp3" duration="250" path="October/Soft and Furious - October - 01 - The creation of all things.mp3" type="music" isDir="false" bitRate="128" created="2019-12-22T21:30:53" year="2018" track="1"/>
  ...
 </directory>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "directory": {
      "id": "173160",
      "name": "October",
      "child": [
        {
          "id": "3506291",
          "parent": "173160",
          "title": "Art",
          "isDir": true
        },
        {
          "id": "3506292",
          "parent": "173160",
          "title": "The creation of all things",
          "artist": "Soft and Furious",
          "artistId": "173127",
          "album": "October",
          "albumId": "176263",
          "genre": "",
          "coverArt": "173129",
          "size": "4010903",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 250,
          "path": "October/Soft and Furious - October - 01 - The creation of all things.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 128,
          "created": "2019-12-22T21:31:17",
          "year": 2018,
          "track": 1
        },
        ...
      ]
    }
  }
}
```

Returns an indexed structure of all top-level content.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
musicFolderId | No | | If specified, only return artists in the music folder with the given ID. See getMusicFolders.
ifModifiedSince | No | | If specified, only return a result if the artist collection has changed since the given time (in milliseconds since 1 Jan 1970).

## getGenres

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <genres>
  <genre songCount="40" albumCount="1"><![CDATA[Electronic]]></genre>
  <genre songCount="1" albumCount="1"><![CDATA[Jazz]]></genre>
 </genres>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "genres": {
      "genre": [
        {
          "songCount": "40",
          "albumCount": "1",
          "value": "Electronic"
        },
        {
          "songCount": "1",
          "albumCount": "1",
          "value": "Jazz"
        }
      ]
    }
  }
}
```

Returns all genres.

## getArtists

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <artists ignoredArticles="">
  <index name="A">
   <artist id="158" name="Aurastys" coverArt="32951"/>
  </index>
  <index name="D">
   <artist id="443" name="Darius" coverArt="33236"/>
  </index>
  ...
 </artists>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "artists": {
      "ignoredArticles": "",
      "index": [
        {
          "name": "A",
          "artist": [
            {
              "id": "158",
              "name": "Aurastys",
              "coverArt": "32951"
            }
          ]
        },
        {
          "name": "D",
          "artist": [
            {
              "id": "443",
              "name": "Darius",
              "coverArt": "33236"
            }
          ]
        },
        ...
      ]
    }
  }
}
```

Similar to getIndexes, but organizes music according to ID3 tags.

### Query Parameters

None

## getArtist

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <artist id="158" name="Aurastys" coverArt="32951" albumCount="1">
  <album id="175695" parent="158" title="ON Trax Vol. 7" album="ON Trax Vol. 7" name="ON Trax Vol. 7" artist="Aurastys" artistId="158" isDir="true" duration="412" songCount="0" coverArt="31855" created="2019-12-22T22:28:54" year="2014"/>
 </artist>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "artist": {
      "id": "158",
      "name": "Aurastys",
      "coverArt": "32951",
      "albumCount": 1,
      "album": [
        {
          "id": "175695",
          "parent": "158",
          "title": "ON Trax Vol. 7",
          "album": "ON Trax Vol. 7",
          "name": "ON Trax Vol. 7",
          "artist": "Aurastys",
          "artistId": "158",
          "isDir": true,
          "duration": 412,
          "songCount": 0,
          "coverArt": "31855",
          "created": "2019-12-22T22:28:39",
          "year": 2014
        }
      ]
    }
  }
}
```

Returns details for an artist, including a list of albums. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The artist ID.

## getAlbum

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <album id="199910" parent="443" title="ON Trax Vol. 7" album="ON Trax Vol. 7" name="ON Trax Vol. 7" artist="Darius" artistId="443" isDir="true" duration="139" songCount="0" coverArt="31855" created="2019-12-22T22:31:20" year="2014">
  <song id="3506323" parent="199910" title="Adam" artist="Darius" artistId="443" album="ON Trax Vol. 7" albumId="199910" genre="" coverArt="31855" size="7921578" contentType="audio/x-flac" suffix="flac" duration="139" path="LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac" type="music" isDir="false" bitRate="455" created="2019-12-22T22:31:20" year="2014" track="1"/>
  ...
 </album>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "album": {
      "id": "199910",
      "parent": "443",
      "title": "ON Trax Vol. 7",
      "album": "ON Trax Vol. 7",
      "name": "ON Trax Vol. 7",
      "artist": "Darius",
      "artistId": "443",
      "isDir": true,
      "duration": 139,
      "songCount": 0,
      "coverArt": "31855",
      "created": "2019-12-22T22:31:44",
      "year": 2014,
      "song": [
        {
          "id": "3506323",
          "parent": "199910",
          "title": "Adam",
          "artist": "Darius",
          "artistId": "443",
          "album": "ON Trax Vol. 7",
          "albumId": "199910",
          "genre": "",
          "coverArt": "31855",
          "size": "7921578",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 139,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 455,
          "created": "2019-12-22T22:31:44",
          "year": 2014,
          "track": 1
        },
        ...
      ]
    }
  }
}
```

Returns details for an album, including a list of songs. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The album ID.

## getSong

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <song id="3506323" parent="199910" title="Adam" artist="Darius" artistId="443" album="ON Trax Vol. 7" albumId="199910" genre="" coverArt="31855" size="7921578" contentType="audio/x-flac" suffix="flac" duration="139" path="LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac" type="music" isDir="false" bitRate="455" created="2019-12-22T22:36:07" year="2014" track="1"/>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "song": {
      "id": "3506323",
      "parent": "199910",
      "title": "Adam",
      "artist": "Darius",
      "artistId": "443",
      "album": "ON Trax Vol. 7",
      "albumId": "199910",
      "genre": "",
      "coverArt": "31855",
      "size": "7921578",
      "contentType": "audio/x-flac",
      "suffix": "flac",
      "duration": 139,
      "path": "LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac",
      "type": "music",
      "isDir": false,
      "bitRate": 455,
      "created": "2019-12-22T22:35:48",
      "year": 2014,
      "track": 1
    }
  }
}
```

Returns details for a song. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The song ID.

## getVideos

Not supported

## getVideoInfo

Not supported

## getArtistInfo

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <artistInfo>
  <similarArtist id="-1" name="Daren"></similarArtist>
  <similarArtist id="2623" name="E.V."></similarArtist>
  <biography>Aurastys is an alias used by Renard under the LapFox Trax (www.lapfoxtrax.com) label who is a female chozo that produces ambient and drone, as well as noise. Aurastys' works generally explore dark, dynamic, and isolated sci-fi themes.

Aurastys was the first of Renard's aliases with a visual identity based on a pre-existing fabricated species, the other being Truxton. &lt;a href=&quot;https://www.last.fm/music/Aurastys&quot;&gt;Read more on Last.fm&lt;/a&gt;. User-contributed text is available under the Creative Commons By-SA License; additional terms may apply.</biography>
  <musicBrainzId>b9686e5a-2abc-48a0-ac07-bb3d04e6b1d0</musicBrainzId>
  <lastFmUrl>https://www.last.fm/music/Aurastys</lastFmUrl>
  <smallImageUrl>https://lastfm.freetls.fastly.net/i/u/34s/2a96cbd8b46e442fc41c2b86b821562f.png</smallImageUrl>
  <mediumImageUrl>https://lastfm.freetls.fastly.net/i/u/64s/2a96cbd8b46e442fc41c2b86b821562f.png</mediumImageUrl>
  <largeImageUrl>https://lastfm.freetls.fastly.net/i/u/174s/2a96cbd8b46e442fc41c2b86b821562f.png</largeImageUrl>
 </artistInfo>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "artistInfo": {
      "similarArtist": [
        {
          "id": -1,
          "name": "Daren"
        },
        {
          "id": 2623,
          "name": "E.V."
        }
      ],
      "biography": "Aurastys is an alias used by Renard under the LapFox Trax (www.lapfoxtrax.com) label who is a female chozo that produces ambient and drone, as well as noise. Aurastys' works generally explore dark, dynamic, and isolated sci-fi themes.\n\nAurastys was the first of Renard's aliases with a visual identity based on a pre-existing fabricated species, the other being Truxton. <a href=\"https://www.last.fm/music/Aurastys\">Read more on Last.fm</a>. User-contributed text is available under the Creative Commons By-SA License; additional terms may apply.",
      "musicBrainzId": "b9686e5a-2abc-48a0-ac07-bb3d04e6b1d0",
      "lastFmUrl": "https://www.last.fm/music/Aurastys",
      "smallImageUrl": "https://lastfm.freetls.fastly.net/i/u/34s/2a96cbd8b46e442fc41c2b86b821562f.png",
      "mediumImageUrl": "https://lastfm.freetls.fastly.net/i/u/64s/2a96cbd8b46e442fc41c2b86b821562f.png",
      "largeImageUrl": "https://lastfm.freetls.fastly.net/i/u/174s/2a96cbd8b46e442fc41c2b86b821562f.png"
    }
  }
}
```

Returns artist info with biography, image URLs and similar artists, using data from last.fm. A similar artist's id is -1 if the artist is not present in your library.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The artist, album or song ID.
count | No | 20 | Max number of similar artists to return.
includeNotPresent | No | false | Whether to return artists that are not present in the media library.

## getArtistInfo2

Identical to getArtistInfo

## getAlbumInfo

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <albumInfo notes="" musicBrainzId="" lastFmUrl="https://www.last.fm/music/Darius/ON+Trax+Vol.+7" smallImageUrl="https://lastfm.freetls.fastly.net/i/u/34s/0e379b0966f8456dc51f0b2a0306024e.png" mediumImageUrl="https://lastfm.freetls.fastly.net/i/u/64s/0e379b0966f8456dc51f0b2a0306024e.png" largeImageUrl="https://lastfm.freetls.fastly.net/i/u/174s/0e379b0966f8456dc51f0b2a0306024e.png"/>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "albumInfo": {
      "notes": null,
      "musicBrainzId": null,
      "lastFmUrl": "https://www.last.fm/music/Darius/ON+Trax+Vol.+7",
      "smallImageUrl": "https://lastfm.freetls.fastly.net/i/u/34s/0e379b0966f8456dc51f0b2a0306024e.png",
      "mediumImageUrl": "https://lastfm.freetls.fastly.net/i/u/64s/0e379b0966f8456dc51f0b2a0306024e.png",
      "largeImageUrl": "https://lastfm.freetls.fastly.net/i/u/174s/0e379b0966f8456dc51f0b2a0306024e.png"
    }
  }
}
```

Returns album notes, image URLs etc, using data from last.fm.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The album ID.

## getAlbumInfo2

Identical to getAlbumInfo

## getSimilarSongs

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <similarSongs>
   <song id="3506323" parent="199910" title="Adam" artist="Darius" artistId="443" album="ON Trax Vol. 7" albumId="199910" genre="" coverArt="31855" size="7921578" contentType="audio/x-flac" suffix="flac" duration="139" path="LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac" type="music" isDir="false" bitRate="455" created="2019-12-22T22:31:20" year="2014" track="1"/>
 </similarSongs>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "similarSongs": {
      "song": [
        {
          "id": "3506323",
          "parent": "199910",
          "title": "Adam",
          "artist": "Darius",
          "artistId": "443",
          "album": "ON Trax Vol. 7",
          "albumId": "199910",
          "genre": "",
          "coverArt": "31855",
          "size": "7921578",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 139,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 455,
          "created": "2019-12-22T22:35:48",
          "year": 2014,
          "track": 1
        }
      ]
    }
  }
}
```

Returns a random collection of songs from the given artist and similar artists, using data from last.fm. Typically used for artist radio features.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The artist, album or song ID.
count | No | 50 | Max number of songs to return.

## getSimilarSongs2

Identical to getSimilarSongs

## getTopSongs

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <topSongs>
  <song id="3506346" parent="178179" title="Time And Time Again" artist="Ian Wiese" artistId="946" album="Portal Stories: Mel Soundtrack" albumId="178179" genre="Jazz" coverArt="32077" size="10566459" contentType="audio/mpeg" suffix="mp3" duration="261" path="Portal Stories Mel/31 - Time And Time Again.mp3" type="music" isDir="false" bitRate="320" created="2019-12-22T22:57:58" year="2015" track="31"/>
 </topSongs>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "topSongs": {
      "song": [
        {
          "id": "3506346",
          "parent": "178179",
          "title": "Time And Time Again",
          "artist": "Ian Wiese",
          "artistId": "946",
          "album": "Portal Stories: Mel Soundtrack",
          "albumId": "178179",
          "genre": "Jazz",
          "coverArt": "32077",
          "size": "10566459",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 261,
          "path": "Portal Stories Mel/31 - Time And Time Again.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-22T22:58:37",
          "year": 2015,
          "track": 31
        }
      ]
    }
  }
}
```

Returns top songs for the given artist, using data from last.fm.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
artist | Yes | | The artist name.
count | No | 50 | Max number of songs to return.

# Album/song lists

## getAlbumList

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <albumList>
  <album id="178178" parent="885" title="Portal Stories: Mel Soundtrack" album="Portal Stories: Mel Soundtrack" name="Portal Stories: Mel Soundtrack" artist="Harry Callaghan" artistId="885" isDir="true" duration="110" songCount="0" coverArt="32077" created="2019-12-22T23:06:14" year="2015"/>
  ...
 </albumList>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "albumList": {
      "album": [
        {
          "id": "178178",
          "parent": "885",
          "title": "Portal Stories: Mel Soundtrack",
          "album": "Portal Stories: Mel Soundtrack",
          "name": "Portal Stories: Mel Soundtrack",
          "artist": "Harry Callaghan",
          "artistId": "885",
          "isDir": true,
          "duration": 110,
          "songCount": 0,
          "coverArt": "32077",
          "created": "2019-12-22T23:04:54",
          "year": 2015
        },
        ...
      ]
    }
  }
}
```

Returns a list of random, newest, highest rated etc. albums.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The list type. Must be one of the following: `random`, `newest`, `highest`, `frequent`, `recent`, `alphabeticalByName`, `alphabeticalByArtist`, `starred`, `byYear`, `byGenre`.
size | No | 10 | The number of albums to return. Max 500.
offset | No | 0 | The list offset. Useful if you for example want to page through the list of newest albums.
fromYear | Yes (if `type` is `byYear`) | | The first year in the range. If `fromYear > toYear` a reverse chronological list is returned.
toYear | Yes (if `type` is `byYear`) | | The last year in the range. 
genre | Yes (if `type` is `genre`) | | The name of the genre. 
musicFolderId | No | | Only return albums in the music folder with the given ID. See `getMusicFolders`.

## getAlbumList2

Identical to getAlbumList

## getRandomSongs

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <randomSongs>
  <song id="3506318" parent="176263" title="Your worst fear" artist="Soft and Furious" artistId="173127" album="October" albumId="176263" genre="" coverArt="173129" size="833329" contentType="audio/mpeg" suffix="mp3" duration="51" path="October/Soft and Furious - October - 27 - Your worst fear.mp3" type="music" isDir="false" bitRate="128" created="2019-12-22T23:16:10" year="2018" track="27"/>
  ...
 </randomSongs>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "randomSongs": {
      "song": [
        {
          "id": "3506314",
          "parent": "176263",
          "title": "A very curious blimp over venice",
          "artist": "Soft and Furious",
          "artistId": "173127",
          "album": "October",
          "albumId": "176263",
          "genre": "",
          "coverArt": "173129",
          "size": "2324425",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 145,
          "path": "October/Soft and Furious - October - 23 - A very curious blimp over venice.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 128,
          "created": "2019-12-22T23:16:59",
          "year": 2018,
          "track": 23
        },
        ...
      ]
    }
  }
}
```

Returns random songs matching the given criteria.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
size | No | 10 | The maximum number of songs to return. Max 500.
genre | No | | Only returns songs belonging to this genre.
fromYear | No | | Only return songs published after or in this year.
toYear | No | | Only return songs published before or in this year.
musicFolderId | No | | Only return songs in the music folder with the given ID. See `getMusicFolders`.

## getSongsByGenre

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <songsByGenre>
  <song id="3506346" parent="178179" title="Time And Time Again" artist="Ian Wiese" artistId="946" album="Portal Stories: Mel Soundtrack" albumId="178179" genre="Jazz" coverArt="32077" size="10566459" contentType="audio/mpeg" suffix="mp3" duration="261" path="Portal Stories Mel/31 - Time And Time Again.mp3" type="music" isDir="false" bitRate="320" created="2019-12-22T23:20:50" year="2015" track="31"/>
 </songsByGenre>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "songsByGenre": {
      "song": [
        {
          "id": "3506346",
          "parent": "178179",
          "title": "Time And Time Again",
          "artist": "Ian Wiese",
          "artistId": "946",
          "album": "Portal Stories: Mel Soundtrack",
          "albumId": "178179",
          "genre": "Jazz",
          "coverArt": "32077",
          "size": "10566459",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 261,
          "path": "Portal Stories Mel/31 - Time And Time Again.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-22T23:20:23",
          "year": 2015,
          "track": 31
        }
      ]
    }
  }
}
```

Returns songs in a given genre.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
genre | Yes | | The genre name.
count | No | 10 | The maximum number of songs to return. Max 500.
offset | No | 0 | The offset. Useful if you want to page through the songs in a genre.
musicFolderId | No | | Only return albums in the music folder with the given ID. See `getMusicFolders`.

## getNowPlaying

Not supported

## getStarred

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <starred>
  <album id="178178" parent="885" title="Portal Stories: Mel Soundtrack" album="Portal Stories: Mel Soundtrack" name="Portal Stories: Mel Soundtrack" artist="Harry Callaghan" artistId="885" isDir="true" duration="155" songCount="0" coverArt="32077" created="2019-12-22T23:29:22" starred="2019-12-22T23:29:22" year="2015"/>
 </starred>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "starred": {
      "song": [],
      "album": [
        {
          "id": "178178",
          "parent": "885",
          "title": "Portal Stories: Mel Soundtrack",
          "album": "Portal Stories: Mel Soundtrack",
          "name": "Portal Stories: Mel Soundtrack",
          "artist": "Harry Callaghan",
          "artistId": "885",
          "isDir": true,
          "duration": 155,
          "songCount": 0,
          "coverArt": "32077",
          "created": "2019-12-22T23:29:35",
          "starred": "2019-12-22T23:29:35",
          "year": 2015
        }
      ],
      "artist": []
    }
  }
}
```

Returns details for an album, including a list of songs. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
musicFolderId | No | | Only return results from the music folder with the given ID. See `getMusicFolders`.

## getStarred2

Identical to getStarred

# Artist lists

## getArtistList

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <artistList>
  <artist id="1861" name="RQ" coverArt="34654" songCount="7"/>
  ...
 </artistList>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "artistList": {
      "artist": [
        {
          "id": "1862",
          "name": "RQ vs DARIUS vs EUGENE",
          "coverArt": "34655",
          "songCount": 1
        },
        ...
      ]
    }
  }
}
```

Returns a list of random, newest, highest rated etc. albums.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
type | Yes | | The list type. Must be one of the following: `random`, `newest`, `highest`, `frequent`, `recent`, `alphabeticalByName`, `alphabeticalByArtist`, `starred`, `byYear`, `byGenre`.
size | No | 10 | The number of albums to return. Max 500.
offset | No | 0 | The list offset. Useful if you for example want to page through the list of newest albums.
fromYear | Yes (if `type` is `byYear`) | | The first year in the range. If `fromYear > toYear` a reverse chronological list is returned.
toYear | Yes (if `type` is `byYear`) | | The last year in the range. 
genre | Yes (if `type` is `genre`) | | The name of the genre. 
musicFolderId | No | | Only return albums in the music folder with the given ID. See `getMusicFolders`.

## getArtistList2

Identical to getArtistList

# Song lists

## getNewaddedSongs

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <newaddedSongs>
  <song id="3506333" parent="199918" title="Billy Joe's Dubstep Adventure" artist="Truxton" artistId="2237" album="ON Trax Vol. 7" albumId="199918" genre="" coverArt="31855" size="26394168" contentType="audio/x-flac" suffix="flac" duration="224" path="LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac" type="music" isDir="false" bitRate="939" created="2019-12-24T00:46:16" year="2014" track="11"/>
 </newaddedSongs>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "newaddedSongs": {
      "song": [
        {
          "id": "3506333",
          "parent": "199918",
          "title": "Billy Joe's Dubstep Adventure",
          "artist": "Truxton",
          "artistId": "2237",
          "album": "ON Trax Vol. 7",
          "albumId": "199918",
          "genre": "",
          "coverArt": "31855",
          "size": "26394168",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 224,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 939,
          "created": "2019-12-24T00:45:46",
          "year": 2014,
          "track": 11
        },
        ...
      ]
    }
  }
}
```
Returns new added songs. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
count | No | 20 | The maximum number of songs to return. Max 500.
offset | No | 0 | The offset. Useful if you want to page through the songs.

## getLastplayedSongs

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <lastplayedSongs>
  <song id="3506333" parent="199918" title="Billy Joe's Dubstep Adventure" artist="Truxton" artistId="2237" album="ON Trax Vol. 7" albumId="199918" genre="" coverArt="31855" size="26394168" contentType="audio/x-flac" suffix="flac" duration="224" path="LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac" type="music" isDir="false" bitRate="939" created="2019-12-24T00:46:16" year="2014" track="11"/>
 </lastplayedSongs>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "lastplayedSongs": {
      "song": [
        {
          "id": "3506333",
          "parent": "199918",
          "title": "Billy Joe's Dubstep Adventure",
          "artist": "Truxton",
          "artistId": "2237",
          "album": "ON Trax Vol. 7",
          "albumId": "199918",
          "genre": "",
          "coverArt": "31855",
          "size": "26394168",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 224,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 939,
          "created": "2019-12-24T00:45:46",
          "year": 2014,
          "track": 11
        },
        ...
      ]
    }
  }
}
```
Returns new added songs. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
count | No | 20 | The maximum number of songs to return. Max 500.
offset | No | 0 | The offset. Useful if you want to page through the songs.

## getMostplayedSongs

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <mostplayedSongs>
  <song id="3506333" parent="199918" title="Billy Joe's Dubstep Adventure" artist="Truxton" artistId="2237" album="ON Trax Vol. 7" albumId="199918" genre="" coverArt="31855" size="26394168" contentType="audio/x-flac" suffix="flac" duration="224" path="LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac" type="music" isDir="false" bitRate="939" created="2019-12-24T00:46:16" year="2014" track="11"/>
 </mostplayedSongs>
</subsonic-response>

```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "mostplayedSongs": {
      "song": [
        {
          "id": "3506333",
          "parent": "199918",
          "title": "Billy Joe's Dubstep Adventure",
          "artist": "Truxton",
          "artistId": "2237",
          "album": "ON Trax Vol. 7",
          "albumId": "199918",
          "genre": "",
          "coverArt": "31855",
          "size": "26394168",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 224,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Truxton - ON Trax Vol. 7 - 11 Billy Joe's Dubstep Adventure.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 939,
          "created": "2019-12-24T00:45:46",
          "year": 2014,
          "track": 11
        },
        ...
      ]
    }
  }
}
```
Returns most played songs. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
count | No | 20 | The maximum number of songs to return. Max 500.
offset | No | 0 | The offset. Useful if you want to page through the songs.

# Searching

## search

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <searchResult>
  <match id="3506346" parent="178179" title="Time And Time Again" artist="Ian Wiese" artistId="946" album="Portal Stories: Mel Soundtrack" albumId="178179" genre="Jazz" coverArt="32077" size="10566459" contentType="audio/mpeg" suffix="mp3" duration="261" path="Portal Stories Mel/31 - Time And Time Again.mp3" type="music" isDir="false" bitRate="320" created="2019-12-23T16:42:23" year="2015" track="31"/>
 </searchResult>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "searchResult": {
      "match": [
        {
          "id": "3506346",
          "parent": "178179",
          "title": "Time And Time Again",
          "artist": "Ian Wiese",
          "artistId": "946",
          "album": "Portal Stories: Mel Soundtrack",
          "albumId": "178179",
          "genre": "Jazz",
          "coverArt": "32077",
          "size": "10566459",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 261,
          "path": "Portal Stories Mel/31 - Time And Time Again.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-23T16:41:54",
          "year": 2015,
          "track": 31
        }
      ]
    }
  }
}
```

Returns a listing of files matching the given search criteria. Supports paging through the result. Please use search2 instead.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
artist | No | | Artist to search for.
album | No | | Album to search for.
title | No | | Any field to search for, not just titles.
any | No | | Any field to search for.
count | No | 20 | Maximum number of results to return.
offset | No | 0 | Search result offset, used for paging.
newerThan | No | | Only return matches that are newer than this. Given as milliseconds since 1970.

## search2

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <searchResult2>
  <song id="3506346" parent="178179" title="Time And Time Again" artist="Ian Wiese" artistId="946" album="Portal Stories: Mel Soundtrack" albumId="178179" genre="Jazz" coverArt="32077" size="10566459" contentType="audio/mpeg" suffix="mp3" duration="261" path="Portal Stories Mel/31 - Time And Time Again.mp3" type="music" isDir="false" bitRate="320" created="2019-12-23T16:44:56" year="2015" track="31"/>
  <artist id="946" name="Ian Wiese" coverArt="33739"/>
 </searchResult2>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "searchResult2": {
      "song": [
        {
          "id": "3506346",
          "parent": "178179",
          "title": "Time And Time Again",
          "artist": "Ian Wiese",
          "artistId": "946",
          "album": "Portal Stories: Mel Soundtrack",
          "albumId": "178179",
          "genre": "Jazz",
          "coverArt": "32077",
          "size": "10566459",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 261,
          "path": "Portal Stories Mel/31 - Time And Time Again.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-23T16:45:37",
          "year": 2015,
          "track": 31
        }
      ],
      "album": [],
      "artist": [
        {
          "id": "946",
          "name": "Ian Wiese",
          "coverArt": "33739"
        }
      ]
    }
  }
}
```

Returns albums, artists and songs matching the given search criteria. Supports paging through the result.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
query | Yes | | Search query.
artistCount | No | 20 | Maximum number of artists to return.
artistOffset | No | 0 | Search result offset for artists. Used for paging.
albumCount | No | 20 | Maximum number of albums to return.
albumOffset | No | 0 | Search result offset for albums. Used for paging.
songCount | No | 20 | Maximum number of songs to return.
songOffset | No | 0 | Search result offset for songs. Used for paging.
musicFolderId | No | | Only return results from the music folder with the given ID. See `getMusicFolders`.

## search3

Identical to search2

# Playlists

## getPlaylists

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <playlists>
  <playlist id="3512518" owner="demo@asti.ga" name="Portal Stories" public="false" songCount="41" coverArt="406191" created="2019-12-23T16:50:42"/>
 </playlists>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "playlists": {
      "playlist": [
        {
          "id": "3512518",
          "owner": "demo@asti.ga",
          "name": "Portal Stories",
          "public": false,
          "songCount": 41,
          "coverArt": "406191",
          "created": "2019-12-23T16:50:07"
        }
      ]
    }
  }
}
```

Returns all playlists.

### Query Parameters

None

## getPlaylist

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <playlist id="3512518" name="Backblaze Demo" owner="demo@asti.ga" songCount="41" created="2019-12-23T16:52:57">
  <entry id="3512519" parent="173205" title="01 - Interfacing.mp3" artist="" artistId="2610" album="" albumId="173205" genre="" coverArt="30919" size="1" contentType="audio/mpeg" suffix="mp3" duration="0" path="Portal Stories Mel/01 - Interfacing.mp3" type="music" isDir="false" bitRate="320" created="2019-12-23T16:52:56"/>
  ...
 </playlist>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "playlist": {
      "id": "3512518",
      "name": "Backblaze Demo",
      "owner": "demo@asti.ga",
      "songCount": 41,
      "created": "2019-12-23T16:53:23",
      "entry": [
        {
          "id": "3512519",
          "parent": "173205",
          "title": "01 - Interfacing.mp3",
          "artist": "",
          "artistId": "2610",
          "album": "",
          "albumId": "173205",
          "genre": "",
          "coverArt": "30919",
          "size": 1,
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 0,
          "path": "Portal Stories Mel/01 - Interfacing.mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-23T16:53:23"
        },
        ...
      ]
    }
  }
}
```

<aside class="danger">
Songs listed in a playlist are not guaranteed to exist, as a playlist may contain references to files that were later removed.
</aside>
Returns a listing of files in a saved playlist.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | ID of the playlist to return, as obtained by `getPlaylists`.

## createPlaylist

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <playlist id="3512520" name="2019-12-23" owner="demo@asti.ga" songCount="22" created="2019-12-23T17:51:26">
  <entry id="3506323" parent="199910" title="Adam" artist="Darius" artistId="443" album="ON Trax Vol. 7" albumId="199910" genre="" coverArt="31855" size="7921578" contentType="audio/x-flac" suffix="flac" duration="139" path="LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac" type="music" isDir="false" bitRate="455" created="2019-12-23T17:51:26" year="2014" track="1"/>
  ...
 </playlist>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "playlist": {
      "id": "3512520",
      "name": "2019-12-23",
      "owner": "demo@asti.ga",
      "songCount": 22,
      "created": "2019-12-23T17:48:52",
      "entry": [
        {
          "id": "3506323",
          "parent": "199910",
          "title": "Adam",
          "artist": "Darius",
          "artistId": "443",
          "album": "ON Trax Vol. 7",
          "albumId": "199910",
          "genre": "",
          "coverArt": "31855",
          "size": "7921578",
          "contentType": "audio/x-flac",
          "suffix": "flac",
          "duration": 139,
          "path": "LAPFOX TRAX/ON Trax Vol. 7/Darius - ON Trax Vol. 7 - 01 Adam.flac",
          "type": "music",
          "isDir": false,
          "bitRate": 455,
          "created": "2019-12-23T17:48:52",
          "year": 2014,
          "track": 1
        },
        ...
      ]
    }
  }
}
```

Creates (or updates) a playlist.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
playlistId | Yes (if updating) | | The playlist ID.
name | Yes (if creating) | | The human-readable name of the playlist.
songId | No | | ID of a song in the playlist. Use one `songId` parameter for each song in the playlist.

## updatePlaylist

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Updates a playlist. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
playlistId | Yes | | The playlist ID.
name | No | | The human-readable name of the playlist.
comment | No | | The playlist comment.
songIdToAdd | No | | Add this song with this ID to the playlist. Multiple parameters allowed.
songIndexToRemove | No | | Remove the song at this position in the playlist. Multiple parameters allowed.

## deletePlaylist

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Deletes a saved playlist.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The playlist ID.

## reorderPlaylist

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Reorder a playlist. Move a song from one index to another index.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
fromIndex | Yes | | The index to move.
toIndex | Yes | | Where to move the song from `fromIndex` to.

# Media retrieval

## stream

Streams a given media file.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The id of the file to stream. 

## download

Identical to stream

## hls 

Not supported

## getCaptions

Not supported

## getCoverArt

Returns a cover art image.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The id of the cover art. 
size | No | | The size of the image to be returned, e.g. 200x200

## getLyrics

Not supported

## getAvatar

Returns the avatar for this user as a square image.

### Query Parameters

None

# Media annotation

## star

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Attaches a star to a song, album or artist.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | No | | The ID of the file (song) or folder (album/artist) to star. Multiple parameters allowed.
albumId | No | | The ID of an album to star. Multiple parameters allowed.
artistId | No | | The ID of an artist to star. Multiple parameters allowed.

## unstar

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Removes the star from a song, album or artist.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | No | | The ID of the file (song) or folder (album/artist) to unstar. Multiple parameters allowed.
albumId | No | | The ID of an album to unstar. Multiple parameters allowed.
artistId | No | | The ID of an artist to unstar. Multiple parameters allowed.

## setRating

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Sets the rating for a music file.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The ID of the file (song) or folder (album/artist) to rate. 
rating | Yes | | The rating between 1 and 5 (inclusive), or 0 to remove the rating. 

## scrobble

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Scrobble a song to last.fm (if linked to the Astiga account).

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | The ID of the file to scrobble. 
time | No | | The time (in milliseconds since 1 Jan 1970) at which the song was listened to.
submission | No | True | Whether this is a "submission" or a "now playing" notification.

# Sharing

## getShares

Not supported

## createShare

Not supported

## updateShare

Not supported

## deleteShare

Not supported

# Podcast

## getPodcasts

> The XML structure looks like this:

```xml

<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <podcasts>
  <channel id="1188f7c4-25b1-11ea-a12f-1a74fb57fd49" url="https://podcasts.files.bbci.co.uk/b006qykl.rss" title="In Our Time" description="Melvyn Bragg and guests discuss the history of ideas" coverArt="3512897" originalImageUrl="http://ichef.bbci.co.uk/images/ic/3000x3000/p06p0j3s.jpg" status="completed">
   <episode id="3512898" title="Auden" path="podcasts/p07yc2wz" artist="BBC Radio 4" album="In Our Time" art="3028083" genre="Podcast" duration="3205" contentType="audio/mpeg" bitRate="256" isDir="false" parent="1188f7c4-25b1-11ea-a12f-1a74fb57fd49" year="2019" size="51280000" suffix="mp3" isVideo="false" created="2019-12-23T18:22:10" status="completed" streamId="3512898" channelId="1188f7c4-25b1-11ea-a12f-1a74fb57fd49" description="Melvyn Bragg and guests discuss the life and poetry of WH Auden (1907-1973) up to his departure from Europe for the USA in 1939. As well as his personal life, he addressed suffering and confusion, and the moral issues that affected the wider public in the 1930s and tried to unpick what was going wrong in society and to understand those times. He witnessed the rise of totalitarianism in the austerity of that decade, travelling through Germany to Berlin, seeing Spain in the Civil War and China during its wars with Japan, often collaborating with Christopher Isherwood. In his lifetime his work attracted high praise and intense criticism, and has found new audiences in the fifty years since his death, sometimes taking literally what he meant ironically. With Mark Ford Poet and Professor of English at University College London Janet Montefiore Professor Emerita of 20th Century English Literature at the University of Kent And Jeremy Noel-Tod Senior Lecturer in Literature and Creative Writing at the University of East Anglia Producer: Simon Tillotson" publishDate="2019-12-19T10:15:00"/>
   ...
  </channel>
 </podcasts>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "podcasts": {
      "channel": [
        {
          "id": "1188f7c4-25b1-11ea-a12f-1a74fb57fd49",
          "url": "https://podcasts.files.bbci.co.uk/b006qykl.rss",
          "title": "In Our Time",
          "description": "Melvyn Bragg and guests discuss the history of ideas",
          "coverArt": "3512897",
          "originalImageUrl": "http://ichef.bbci.co.uk/images/ic/3000x3000/p06p0j3s.jpg",
          "status": "completed",
          "episode": [
            {
              "id": "3512898",
              "title": "Auden",
              "path": "podcasts/p07yc2wz",
              "artist": "BBC Radio 4",
              "album": "In Our Time",
              "art": "3028083",
              "genre": "Podcast",
              "duration": "3205",
              "contentType": "audio/mpeg",
              "bitRate": 256,
              "isDir": false,
              "parent": "1188f7c4-25b1-11ea-a12f-1a74fb57fd49",
              "year": 2019,
              "size": "51280000",
              "suffix": "mp3",
              "isVideo": false,
              "created": "2019-12-23T18:27:11",
              "status": "completed",
              "streamId": "3512898",
              "channelId": "1188f7c4-25b1-11ea-a12f-1a74fb57fd49",
              "description": "Melvyn Bragg and guests discuss the life and poetry of WH Auden (1907-1973) up to his departure from Europe for the USA in 1939. As well as his personal life, he addressed suffering and confusion, and the moral issues that affected the wider public in the 1930s and tried to unpick what was going wrong in society and to understand those times. He witnessed the rise of totalitarianism in the austerity of that decade, travelling through Germany to Berlin, seeing Spain in the Civil War and China during its wars with Japan, often collaborating with Christopher Isherwood. In his lifetime his work attracted high praise and intense criticism, and has found new audiences in the fifty years since his death, sometimes taking literally what he meant ironically. With Mark Ford Poet and Professor of English at University College London Janet Montefiore Professor Emerita of 20th Century English Literature at the University of Kent And Jeremy Noel-Tod Senior Lecturer in Literature and Creative Writing at the University of East Anglia Producer: Simon Tillotson",
              "publishDate": "2019-12-19T10:15:00"
            },
            ...
          ]
        }
      ]
    }
  }
}
```

Returns all podcast channels and (optionally) their episodes.
Requires Astiga Premium to function.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
includeEpisodes | No | true | Whether to include Podcast episodes in the returned result.
id | No | | If specified, only return the Podcast channel with this ID.

## getNewestPodcasts

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <newestPodcasts>
  <episode id="3512898" title="Auden" path="podcasts/p07yc2wz" artist="BBC Radio 4" album="In Our Time" art="3028083" genre="Podcast" duration="3205" contentType="audio/mpeg" bitRate="256" isDir="false" parent="1188f7c4-25b1-11ea-a12f-1a74fb57fd49" year="2019" size="51280000" suffix="mp3" isVideo="false" created="2019-12-23T22:53:33" status="completed" streamId="3512898" channelId="1188f7c4-25b1-11ea-a12f-1a74fb57fd49" description="Melvyn Bragg and guests discuss the life and poetry of WH Auden (1907-1973) up to his departure from Europe for the USA in 1939. As well as his personal life, he addressed suffering and confusion, and the moral issues that affected the wider public in the 1930s and tried to unpick what was going wrong in society and to understand those times. He witnessed the rise of totalitarianism in the austerity of that decade, travelling through Germany to Berlin, seeing Spain in the Civil War and China during its wars with Japan, often collaborating with Christopher Isherwood. In his lifetime his work attracted high praise and intense criticism, and has found new audiences in the fifty years since his death, sometimes taking literally what he meant ironically. With Mark Ford Poet and Professor of English at University College London Janet Montefiore Professor Emerita of 20th Century English Literature at the University of Kent And Jeremy Noel-Tod Senior Lecturer in Literature and Creative Writing at the University of East Anglia Producer: Simon Tillotson" publishDate="2019-12-19T10:15:00"/>
  ...
 </newestPodcasts>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "newestPodcasts": {
      "episode": [
        {
          "id": "3512898",
          "title": "Auden",
          "path": "podcasts/p07yc2wz",
          "artist": "BBC Radio 4",
          "album": "In Our Time",
          "art": "3028083",
          "genre": "Podcast",
          "duration": "3205",
          "contentType": "audio/mpeg",
          "bitRate": 256,
          "isDir": false,
          "parent": "1188f7c4-25b1-11ea-a12f-1a74fb57fd49",
          "year": 2019,
          "size": "51280000",
          "suffix": "mp3",
          "isVideo": false,
          "created": "2019-12-23T22:52:32",
          "status": "completed",
          "streamId": "3512898",
          "channelId": "1188f7c4-25b1-11ea-a12f-1a74fb57fd49",
          "description": "Melvyn Bragg and guests discuss the life and poetry of WH Auden (1907-1973) up to his departure from Europe for the USA in 1939. As well as his personal life, he addressed suffering and confusion, and the moral issues that affected the wider public in the 1930s and tried to unpick what was going wrong in society and to understand those times. He witnessed the rise of totalitarianism in the austerity of that decade, travelling through Germany to Berlin, seeing Spain in the Civil War and China during its wars with Japan, often collaborating with Christopher Isherwood. In his lifetime his work attracted high praise and intense criticism, and has found new audiences in the fifty years since his death, sometimes taking literally what he meant ironically. With Mark Ford Poet and Professor of English at University College London Janet Montefiore Professor Emerita of 20th Century English Literature at the University of Kent And Jeremy Noel-Tod Senior Lecturer in Literature and Creative Writing at the University of East Anglia Producer: Simon Tillotson",
          "publishDate": "2019-12-19T10:15:00"
        },
        ...
      ]
    }
  }
}
```

Returns the most recently published podcast episodes.

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
count | No | 20 | The maximum number of episodes to return.

## refreshPodcasts

Not supported, automatically done by Astiga

## createPodcastChannel

Not supported

## deletePodcastChannel

Not supported

## deletePodcastEpisode

Not supported

## downloadPodcastEpisode

Not supported

# Jukebox

## jukeboxControl

Not supported

# Internet radio

## getInternetRadioStations

Not supported

## createInternetRadioStation

Not supported

## updateInternetRadioStation

Not supported

## deleteInternetRadioStation

Not supported

# Chat

## getChatMessages

Not supported

## addChatMessage

Not supported

# User management

## getUser

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <user username="demo@asti.ga" email="demo@asti.ga" scrobblingEnabled="false" adminRole="false" settingsRole="false" downloadRole="true" uploadRole="false" playlistRole="true" coverArtRole="true" commentRole="false" podcastRole="false" streamRole="true" jukeboxRole="false" shareRole="false"/>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "user": {
      "username": "demo@asti.ga",
      "email": "demo@asti.ga",
      "scrobblingEnabled": false,
      "adminRole": false,
      "settingsRole": false,
      "downloadRole": true,
      "uploadRole": false,
      "playlistRole": true,
      "coverArtRole": true,
      "commentRole": false,
      "podcastRole": false,
      "streamRole": true,
      "jukeboxRole": false,
      "shareRole": false
    }
  }
}
```

Get details about a given user. All values are static, except for `username`, `email`, and `scrobblingEnabled`.

### Query Parameters

None

## getUsers

Not supported

## createUser

Not supported

## updateUser

Not supported

## deleteUser

Not supported

## changePassword

Not supported

# Bookmarks

## getBookmarks

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <bookmarks>
  <bookmark position="92456" comment="" username="demo@asti.ga" changed="2019-12-23T23:02:34">
   <entry id="3506290" parent="173205" title="Rick Astley - Never Gonna Give You Up (8 bit Remix)" artist="" artistId="2610" album="" albumId="173205" genre="" coverArt="30919" size="9686274" contentType="audio/mpeg" suffix="mp3" duration="242" path="Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3" type="music" isDir="false" bitRate="320" created="2019-12-23T23:02:34" year="2016"/>
  </bookmark>
 </bookmarks>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "bookmarks": {
      "bookmark": [
        {
          "position": "92456",
          "comment": "",
          "username": "demo@asti.ga",
          "changed": "2019-12-23T23:01:22",
          "entry": {
            "id": "3506290",
            "parent": "173205",
            "title": "Rick Astley - Never Gonna Give You Up (8 bit Remix)",
            "artist": "",
            "artistId": "2610",
            "album": "",
            "albumId": "173205",
            "genre": "",
            "coverArt": "30919",
            "size": "9686274",
            "contentType": "audio/mpeg",
            "suffix": "mp3",
            "duration": 242,
            "path": "Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3",
            "type": "music",
            "isDir": false,
            "bitRate": 320,
            "created": "2019-12-23T23:01:22",
            "year": 2016
          }
        }
      ]
    }
  }
}
```

Returns all bookmarks for this user. A bookmark is a position within a certain media file.

## createBookmark

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Creates or updates a bookmark (a position within a media file).

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | ID of the media file to bookmark. If a bookmark already exists for this file it will be overwritten.
position | Yes | | The position (in milliseconds) within the media file.
comment | No | | A user-defined comment.

## deleteBookmark

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Deletes the bookmark for a given file. 

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | ID of the media file for which to delete the bookmark.

## getPlayQueue

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
 <playQueue current="3506290" position="93460" username="demo@asti.ga" changedBy="DSub" changed="2019-12-23T23:01:19">
  <entry id="3506290" parent="173205" title="Rick Astley - Never Gonna Give You Up (8 bit Remix)" artist="" artistId="2610" album="" albumId="173205" genre="" coverArt="30919" size="9686274" contentType="audio/mpeg" suffix="mp3" duration="242" path="Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3" type="music" isDir="false" bitRate="320" created="2019-12-23T23:05:13" year="2016"/>
 </playQueue>
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0",
    "playQueue": {
      "current": "3506290",
      "position": "93460",
      "username": "demo@asti.ga",
      "changedBy": "DSub",
      "changed": "2019-12-23T23:01:19",
      "entry": [
        {
          "id": "3506290",
          "parent": "173205",
          "title": "Rick Astley - Never Gonna Give You Up (8 bit Remix)",
          "artist": "",
          "artistId": "2610",
          "album": "",
          "albumId": "173205",
          "genre": "",
          "coverArt": "30919",
          "size": "9686274",
          "contentType": "audio/mpeg",
          "suffix": "mp3",
          "duration": 242,
          "path": "Rick Astley - Never Gonna Give You Up (8 bit Remix).mp3",
          "type": "music",
          "isDir": false,
          "bitRate": 320,
          "created": "2019-12-23T23:06:21",
          "year": 2016
        }
      ]
    }
  }
}
```

Returns the state of the play queue for this user (as set by savePlayQueue). This includes the tracks in the play queue, the currently playing track, and the position within this track. Typically used to allow a user to move between different clients/apps while retaining the same play queue (for instance when listening to an audio book).

### Query Parameters

None

## savePlayQueue

> The XML structure looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<subsonic-response status="ok" version="1.16.0" xmlns="http://subsonic.org/restapi">
</subsonic-response>
```

> The JSON structure looks like this:

```json
{
  "subsonic-response": {
    "status": "ok",
    "version": "1.16.0"
  }
}
```

Saves the state of the play queue for this user. This includes the tracks in the play queue, the currently playing track, and the position within this track. Typically used to allow a user to move between different clients/apps while retaining the same play queue (for instance when listening to an audio book).

### Query Parameters

Parameter | Required | Default | Comment
--------- | -------- | ------- | -------
id | Yes | | ID of a song in the play queue. Use one id parameter for each song in the play queue.
current | No | | The ID of the current playing song.
position | No | | The position in milliseconds within the currently playing song.

# Media library scanning

## getScanStatus

Not supported

## startScan

Not supported