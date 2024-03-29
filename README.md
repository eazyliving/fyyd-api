# fyyd api

Documentations version: 0.15 (2020-09-22)

![fyyd logo](https://fyyd.de/images/fyyd2.jpg)

---

This is the documentation for the API to fyyd, the podcast search engine, directory and slow cooker.

Take my apologies for this simple markdown, I tried to use something more fancy, but I took an arrow to my knee.

**~~Please be aware: This version is most likely to change in the very near future. It's my first kind of that API and I'm still learning how to do things the more or less correct way. All that works and all that is fine, but it may not be optimal. I want it to be at least near to optimal.~~**

**Version 0.2 is now fixed and nailed to the wall. What does that mean? It means, that requests, parameters and responses won't change for that version and applications and scripts will not face substantial modifications, rendering them inoperabel. What will happen though, is that requests and parameters will be added and additional data inside responses will occur, as long as these changes don't touch functionality.**

**Bugs or inconsistencies, flaws and so on will stay forever with v0.2, but may be addressed by additional requests or responses. Whatever I learn with this version will find it's way into 0.3 or whatever I'll number the next substantial release.**



## Endpoint for version 0.2

https://api.fyyd.de/0.2/

always prefix this for every request or else you'll end up in version 0.1 which is mostly different from 0.2.

---

## Table of contents

1. [Preface](#preface)
2. [Authorization](#authorization)
3. [Error Handling](#error-handling)
4. [Account](#account)
5. [User](#user)
6. [Action](#action)
7. [Podcast](#podcast)
8. [Episode](#episode)
9. [Curations](#curations)
10. [Collections](#collections)
11. [Alerts](#alerts)
12. [Search](#search)
13. [Feature](#feature)
14. [3rd party interfaces](#3rd-party-interfaces)

---

## Preface

Podcasts and their metadata always has been a fascinating subject for me. So much data, so few approaches to collect and bring it to good use.

Now it's 2017 and there are a few (I know of ~~three~~ two, maybe ~~four~~ three) more or less experimental projects to work with this gigantic loads of data. One of that projects is fyyd, my attempt to create a 

* podcast search engine
* podcast directory
* podcast recommendation system

fyyd is a freetime project, that get's some love now and then. The progress is as fast as I can afford some free time beside family, job and whatever one has to do. 

As I started fyyd, I knew that an API would be crucial for this project to escape the scope of just a webpage and to find more stunning ways to do whatever one can do with that data. As always: The crowd will find a way to use the data you would never have dreamed of.

Version 0.1 of that API found it's way to a small group of developers, leading into the development of [Skoon](https://skoon.stefantrauth.de), an iOS-App to curate podcast's episodes directly from some podcatchers and the integration of fyyd into [podcat](http://www.podc.at/), a lovely designed podcatcher that pleasantly differs from most machine like user interfaces. 

Version 0.1 had its problems and was a rush job. Thanks to Jeanette and Stefan here and now you see the documentation for version 0.2.

Thanks to all who helped, will help and contribute to this small project.

---



# Authorization

Authorization for 3rd party apps and services is provided with an oauth2 interface. At the moment this is a little bit rudimentary, but working.



## App registration

First step is to register your app at fyyd.de. To do that, head to  [https://fyyd.de/dev/app/](https://fyyd.de/dev/app/).

You then create the credentials and maybe an accesstoken (which only works on app context, which is non existent at this time).



## Auth workflow

### user context

Head your client to **https://fyyd.de/oauth/authorize?client_id=XYZ** where client_id is your client's id (wow… ;)

The user logs in (if neccessary) and authorizes the app.

fyyd now redirects to your registered app's callback-URL, appending the token via #token=ACCESSTOKEN.

In all future requests, add an authorization header:

`Authorization: Bearer ACCESSTOKEN`

### app context

The authorization for application only level is working, but there is no use-case for that at the moment.

The most simple way would to create the accesstoken directly at the developers page at fyyd. You may also send a POST request to https://fyyd.de/oauth/token, containing 

`grant_type=client_credentials`

inside the body.

Additionally you need to either set 

an Authorization Header like

`Authorization: Basic base64encode($client_id:$client_secret)`

(please: base64encode has to be calculated and not included in the headers…)

or added as 

`&client_id=$client_id&client_secret=$client_secret`

in the requests body.



---



# Error Handling

In case an error occurs, the API repsonses with an appropriate HTTP status code and, in case it's not a 404, a more or less helpful msg along inside a JSON object.

What could possibly go wrong? Here are some examples:

## Examples

### Missing authorization

```
{
    "errors": {
        "code": 0,
        "message": "not authorized!",
        "http_code": 401,
        "http_message": "HTTP 401 (GET \/0.2\/account\/info)"
    },
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    }
}
```



### Doing smth really forbidden

```
{
  "errors": {
    "code": 0,
    "message": "this is a personal feed and cannot be deleted",
    "http_code": 403,
    "http_message": "HTTP 403 (POST /0.2/curation/delete)"
  },
  "meta": {
    "API_INFO": {
      "API_VERSION": 0.2
    }
  }
}
```



### Missing parameter

```
{
    "errors": {
        "code": 0,
        "message": "no nick or user id provided",
        "http_code": 400,
        "http_message": "HTTP 400 (GET \/0.2\/user\/curations)"
    },
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    }
}
```


### Unknown resource

```
{
    "errors": {
        "code": 0,
        "message": "user not found",
        "http_code": 400,
        "http_message": "HTTP 400 (GET \/0.2\/user\/curations)"
    },
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    }
}
```

---



# Account

Requests for /account always refer to the user identified by the accesstoken. Thus, you of course need to authenticate to use this requests. All other requests are free until now.



### [GET /account/info]

#### Description 

Returns general information about the account

#### Parameters

none

#### Authorization

required

#### Response

	{
		"status": 1,
		"msg": "ok",
		"meta": {
			"API_INFO": {
				"API_VERSION": 0.2
			}
		},
		"data": {
			"nick": "your-nick",
			"id": 42,
			"fullname": "Jon Doe",
			"bio": "This is me!",
			"url": "https:\/\/fyyd.de\/",
			"layoutImageURL": "https:\/\/img.fyyd.de\/user\/layout\/42.jpg",
			"thumbImageURL": "https:\/\/img.fyyd.de\/user\/thumbs\/42.png",
			"microImageURL": "https:\/\/img.fyyd.de\/user\/micro\/42.png"
		}
	}



### [GET /account/curations]

#### Description

Returns this account's curations. Includes non-public curations as well.

#### Parameters

none

#### Authorization

required

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Retro Computing",
            "id": 1099,
            "description": "Alles \u00fcber Rechner, f\u00fcr die es garantiert keine Garantie mehr gibt.",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/1099.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/1099.png",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/1099.png",
            "public": 0,
            "type": 1,
            "slug": "retro-computing",
            "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/retro-computing",
            "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/retro-computing"
        },
        {
            "title": "Netzpolitik",
            "id": 623,
            "description": "Vorsicht, k\u00f6nnte Spuren von Netzpolitik enthalten!",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/623.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/623.png",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/623.png",
            "public": 1,
            "type": 1,
            "slug": "netzpolitik",
            "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/netzpolitik",
            "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/netzpolitik"
        },
        {
            "title": "Podcasting",
            "id": 586,
            "description": "Alles \u00fcber das Podcasting",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/586.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/586.png",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/586.png",
            "public": 1,
            "type": 1,
            "slug": "podcasting",
            "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/podcasting",
            "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/podcasting"
        }
     
    ]
}
```



### [GET /account/collections]

#### Description

Returns this account's collections.

#### Parameters

none

#### Authorization

required

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Netzpolitik",
            "id": 1,
            "description": "(Netz)politische Podcasts",
            "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/1.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/1.png",
            "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/1.png",
            "slug": "netzpolitik",
            "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/netzpolitik"
        },
        {
            "title": "Gamechanger",
            "id": 8,
            "description": "",
            "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/8.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/8.png",
            "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/8.png",
            "slug": "gamechanger",
            "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/gamechanger"
        },
        {
            "title": "Lieblingspodcasts",
            "id": 11,
            "description": "Was ich so mit auf die Insel nehmen w\u00fcrde..",
            "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/11.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/11.png",
            "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/11.png",
            "slug": "lieblingspodcasts",
            "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/lieblingspodcasts"
        }
    ]
}
```

------



# User

Gets public available information about a registered account at fyyd



### [GET /user]

#### Description

Retrieves the accounts profile information

#### Parameters

* **user_id (required, int)** the users account id
* **or nick (optional, string)** identifies the user by nick

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": {
          "nick": "eazy",
          "id": 42,
          "fullname": "John Doe",
          "bio": "This is me!",
          "url": "https:\/\/fyyd.de\/",
          "layoutImageURL": "https:\/\/img.fyyd.de\/user\/layout\/42.jpg",
          "thumbImageURL": "https:\/\/img.fyyd.de\/user\/thumbs\/42.png",
          "microImageURL": "https:\/\/img.fyyd.de\/user\/micro\/42.png"
      }
    }


### [GET /user/curations[/episodes]]

#### Description

Retrieves the curations maintained by the given user. You may append /episodes to also get the content of this curations. If you're authenticated and you request your own curations, non-public curations will be included also.

#### Parameters

- **user_id (required, int)** the users account id 
- **or nick (optional, string)** identifies the user by nick

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "title": "Netzpolitik",
              "id": 623,
              "description": "Vorsicht, k\u00f6nnte Spuren von Netzpolitik enthalten!",
              "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/623.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/623.png",
              "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/623.png",
              "public": 1,
              "type": 1,
              "slug": "netzpolitik",
              "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/netzpolitik",
              "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/netzpolitik"
          },
          {
              "title": "Podcasting",
              "id": 586,
              "description": "Alles \u00fcber das Podcasting",
              "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/586.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/586.png",
              "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/586.png",
              "public": 1,
              "type": 1,
              "slug": "podcasting",
              "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/podcasting",
              "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/podcasting"
          }
      ]
    }


### [GET /user/collections[/podcasts]]

#### Description

Retrieves the collections maintained by the given user. You may append /podcasts to also get the content of this collections.

#### Parameters

- **user_id (required, int)** the users account id
- **or nick (optional, string)** identifies the user by nick

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "title": "Netzpolitik",
              "id": 1,
              "description": "(Netz)politische Podcasts",
              "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/1.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/1.png",
              "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/1.png",
              "slug": "netzpolitik",
              "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/netzpolitik"
          },
          {
              "title": "Gamechanger",
              "id": 8,
              "description": "",
              "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/8.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/8.png",
              "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/8.png",
              "slug": "gamechanger",
              "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/gamechanger"
          }
      ]
    }
---



# Action

This is the actions API to fyyd. It's a more or less formfree key-value-store.

Requests for /action always refer to the user identified by the accesstoken and the used app.

### [POST /action]

#### Description

Saves an action for the authorized user.

The action always referes to an object inside fyyd's database, which may be:

* an episode
* a podcast
* a curation
* a collection
* a user

The action itself is represented by a string you may choose freely. Some of these actions will be used in future features of fyyd. This is not implemented at the time this documentation is written. 

#### Authorization

required

#### Parameters

+ **object_id (required, int)** the id of the object
+ **object_type (required, string)** the type of the object. episode, podcast, curation, collection, user
+ **action (required, string)** a free to choose action
+ **metadata (optional, string)** further data you'd like to associate to this action

#### Response

Returns 204 - No Content



### [POST /action/delete]

#### Description

Deletes an action for the authorized user.

#### Authorization

required

#### Parameters

- **action_id (required, int)** the id of the action

#### Response

Returns 204 - No Content



### [GET /action]

#### Description

Gets all actions for the authorized user, triggered by the authorized app and filtered by the given criteria.

#### Authorization

required

#### Parameters

- **object_id (optional, int)** the id of the object
- **object_type (optional, string)** the type of the object. episode, podcast, curation, collection, user
- **action (optional, string)** a free to choose action
- **metadata (optional, string)** further data you'd like to associate to this action
- **date_start (optional, date string)** optional start date for this query
- **date_end (optional, date string)** optional end date for this query

Choose one or more of these parameters. Providing none of them returns all actions issued by the logged in user with the authorized app.

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "id": 1,
              "object_id": 85,
              "object_type": "podcast",
              "action": "test",
              "metadata": "More data",
              "date": "2017-04-17 20:04:22",
              "user_id": 23,
              "app_id": 42
          },
          {
              "id": 2,
              "object_id": 85,
              "object_type": "podcast",
              "action": "test",
              "metadata": "More data, really!",
              "date": "2017-04-17 20:09:01",
              "user_id": 23,
              "app_id": 42
          }
      ]
    }
---



# Podcast

Retrieves information about a podcast

### [GET /podcast/[/episodes]]

#### Description

Gets information about the podcast with id {id}. You may retrieve the episodes by appending /episodes.

Additionally you can address the resultset with {page} and {count}.

#### Parameters

- **podcast_id (required, int)** the podcast's id 
- *or* **podcast_slug (alternative, string)** the podcast's slug 
- **page (optional, int)** the page you want to address, default: 0
- **count (optional,int)** the page's size, default: 50

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
      	"paging": {
                "count": 1,
                "page": 0,
                "first_page": 0,
                "last_page": 22,
                "next_page": 1,
                "prev_page": null
          },
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": {
          "title": "Schall und Rauch",
          "id": 64,
          "xmlURL": "http:\/\/schallrauch.hoersuppe.de\/sur\/m4a",
          "htmlURL": "http:\/\/schallrauch.hoersuppe.de\/sur\/show",
          "imgURL": "http:\/\/schallrauch.hoersuppe.de\/feedimages\/itunes1400.png",
          "status": 399,
          "slug": "schall-und-rauch",
          "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/64.jpg",
          "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/64.png",
          "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/64.png",
          "language": "de",
          "lastpoll": "2017-04-17 14:42:41",
          "generator": "firtz podcast publisher v2.0",
          "categories": [
                52
            ],
          "lastpub": "2016-01-08 21:15:22",
          "rank": 22,
          "url_fyyd": "https:\/\/fyyd.de\/podcast\/64",
          "description": "Alexander und Christian, zwei mittelreife Fr\u00fcchte der 70er, sozialisiert in den 80ern ver\u00f6ffentlichen ihre monatlichen Schw\u00e4tzchen auf der Parkbank.",
          "subtitle": "Zwei Opas erz\u00e4hlen vom Krieg",
          "episodes": [
              {
                  "title": "Stasi, Sherlock und die ARD",
                  "id": 1742,
                  "guid": "sur-sur022",
                  "url": "http:\/\/schallrauch.hoersuppe.de\/sur\/show\/sur022",
                  "enclosure": "http:\/\/cdn.podseed.org\/schallundrauch\/sur022.m4a",
                  "podcast_id": 64,
                  "imgURL": "http:\/\/schallrauch.hoersuppe.de\/feedimages\/itunes1400.png",
                  "pubdate": "2016-01-08 21:15:22",
                  "duration": 5579,
                  "url_fyyd": "https:\/\/fyyd.de\/episode\/1742",
                  "description": "<p>Alexander erz\u00e4hlt von seinen Eindr\u00fccken beim Besuch der BStU, war dar\u00fcber hinaus in London und macht mir die Nase lang, weil er dort die aktuelle Folge Sherlock schon geschaut hat. Im Hotel. So ein Hund!<\/p>\n\n<p>Daf\u00fcr aber hat Deutschland durchaus etwas zu bieten im Bereich der Serien, man sollte nur nicht unbedingt beim Tatort suchen. Alexander rezitiert Schweiger, was dann schnell das Ende der Folge einl\u00e4utet&hellip;<\/p>\n"
              }
          ]
       }
    }



### [GET /podcast/season]

#### Description

Gets episodes from a season of a podcast with id {podcast_id}. 

Additionally you can address the resultset with {page} and {count}.

#### Parameters

- **podcast_id (required, int)** the podcast's id 
- *or* **podcast_slug (alternative, string)** the podcast's slug 
- **season_number (required, int)** the seasons number
- **episode_number (optional, int)** an episodes number to retrieve
- **page (optional, int)** the page you want to address, default: 0
- **count (optional,int)** the page's size, default: 50

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "paging": {
            "count": 50,
            "page": 0,
            "first_page": 0,
            "last_page": 0,
            "next_page": null,
            "prev_page": null
        },
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": {
        "title": "Raumzeit",
        "id": 703,
        "xmlURL": "https:\/\/feeds.metaebene.me\/raumzeit\/m4a",
        "htmlURL": "http:\/\/raumzeit-podcast.de",
        "imgURL": "https:\/\/meta.metaebene.me\/media\/raumzeit\/raumzeit-icon-1400x1400.jpg",
        "status": 200,
        "slug": "raumzeit",
        "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/703.jpg",
        "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/703.png",
        "smallImageURL": "https:\/\/img.fyyd.de\/pd\/small\/703.jpg",
        "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/703.png",
        "language": "de",
        "lastpoll": "2018-01-21T11:04:01+01:00",
        "generator": "Podlove Podcast Publisher v2.7.0.build467",
        "categories": [
            48,
            62
        ],
        "lastpub": "2018-01-20T11:03:23+01:00",
        "rank": 46,
        "url_fyyd": "https:\/\/fyyd.de\/podcast\/raumzeit\/0",
        "description": "Raumzeit ist eine Serie von Gespr\u00e4chen mit Wissenschaftlern, Ingenieuren, Astronauten und Projektleitern \u00fcber Raumfahrt. Jede Episode r\u00fcckt einen Themenbereich in den Fokus und diskutiert ausf\u00fchrlich alle Aspekte und Details.  ",
        "subtitle": "Raumfahrt und kosmische Angelegenheiten",
        "episode_count": 68,
        "episodes": [
            {
                "title": "Philae",
                "id": 24580,
                "guid": "podlove-2015-04-28t23:48:53+00:00-4a76622f3e60548",
                "url": "http:\/\/raumzeit-podcast.de\/2015\/06\/16\/rz058-philae\/",
                "enclosure": "https:\/\/tracking.feedpress.it\/link\/13437\/1885139\/rz058-philae.m4a",
                "podcast_id": 703,
                "imgURL": "https:\/\/meta.metaebene.me\/media\/raumzeit\/rz058-philae.jpg",
                "pubdate": "2015-06-16T19:51:06+02:00",
                "duration": 5587,
                "status": 200,
                "num_season": 2,
                "num_episode": 58,
                "url_fyyd": "https:\/\/fyyd.de\/episode\/24580",
                "description": "...",
                "chapters": [
                    {
                        "start": "00:00:00.000",
                        "start_ms": 0,
                        "title": "Intro"
                    }
                  
                ],
                "content_type": "audio\/mp4"
            }
      ]
   }
}
```

### [POST /podcast/action]

#### Description

Performes an action to a podcast owned by you (see claim-button on every podcast page).

At the moment, 'check' is the only action to perform. This triggers an update within the next poll cycle.

#### Authorization

required

#### Parameters

- **podcast_id (required, int)** the podcast's id
- *or* **podcast_slug (alternative, string)** the podcast's slug
- **action (required, string)** the action to perform:
  * **check** 

#### Response

returns a 204: No Content



### [GET /podcasts]

#### Description

Gets information about all podcasts within fyyd.

Additionally you can address the resultset with {page} and {count}. 

Please note that this request is the plural form of podcast, podcasts  ;-)

#### Parameters

- **page (optional, int)** the page you want to address, default: 0
- **count (optional,int)** the page's size, default: 50

 #### Response

The response includes data about the pagination.

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "paging": {
              "count": 2,
              "page": 0,
              "first_page": 0,
              "last_page": 7094,
              "next_page": 1,
              "prev_page": null
          },
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "title": "Freakonomics Radio",
              "id": 1,
              "xmlURL": "http:\/\/feeds.feedburner.com\/freakonomicsradio",
              "htmlURL": "http:\/\/www.wnyc.org\/articles\/freakonomics-podcast",
              "imgURL": "https:\/\/media2.wnyc.org\/i\/1400\/1400\/l\/80\/1\/wn16_wnycstudios_freakonomics-rev3.png",
              "status": 390,
              "slug": "freakonomics-radio",
              "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/1.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/1.png",
              "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/1.png",
              "language": "en",
              "lastpoll": "2017-04-16 00:34:49",
              "generator": null,
              "categories": [
                52
           	  ],
              "lastpub": "2017-03-16 04:00:00",
              "rank": 1,
              "url_fyyd": "https:\/\/fyyd.de\/podcast\/1",
              "description": "Have fun discovering the hidden side of everything with host Stephen J. Dubner, co-author of the best-selling \"Freakonomics\u201d books. Each week, hear surprising conversations that explore the riddles of everyday life and the weird wrinkles of human nature\u2014from cheating and crime to parenting and sports. Dubner talks with Nobel laureates and provocateurs, social scientists and entrepreneurs \u2014 and his \u201cFreakonomics\u201d co-author Steve Levitt. After just a few episodes, this podcast will have you too thinking like a Freak. Produced by WNYC Studios, home of other great podcasts such as \u201cRadiolab,\" \"Death, Sex & Money,\" and \"On the Media.\" ",
              "subtitle": "Have fun discovering the hidden side of everything with host Stephen J. Dubner, co-author of the best-selling \"Freakonomics\u201d books. Each week, hear surprising conversations that explore the riddles of everyday life and the weird wrinkles of human nature\u2014f"
          },
          {
              "title": "Skeptoid",
              "id": 3,
              "xmlURL": "http:\/\/skeptoid.com\/podcast.xml",
              "htmlURL": "https:\/\/skeptoid.com",
              "imgURL": "http:\/\/skeptoid.com\/images\/1500b.jpg",
              "status": 304,
              "slug": "skeptoid",
              "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/3.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/3.png",
              "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/3.png",
              "language": "en",
              "lastpoll": "2017-04-17 02:39:44",
              "generator": null,
              "categories": [
                      52
               ],
              "lastpub": "2017-04-11 02:00:00",
              "rank": 1,
              "url_fyyd": "https:\/\/fyyd.de\/podcast\/3",
              "description": "Since 2006, the weekly Skeptoid podcast has been taking on all the most popular myths and revealing the true science, true history, and true lessons we can learn from each. Free subscribers get the most recent 50 episodes, premium subscribers (skeptoid.com) can access the full archive, all ad-free.",
              "subtitle": "A weekly look at the true facts behind urban legends and stories you've probably heard."
          }
      ]
    }


Additionally links inside the header are provided:

`Link: <https://api.fyyd.de/0.2/podcasts/1/2>; rel="next", <https://api.fyyd.de/0.2/podcasts/0/2>; rel="first", <https://api.fyyd.de/0.2/podcasts/7094/2>; rel="last"`



### [GET /categories]

#### Description

Gets the complete categories tree

#### Parameters

none

#### Response	

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "id": 1,
              "slug": "arts",
              "name": "Arts",
              "name_de": "Kunst",
              "subcategories": [
                  {
                      "id": 2,
                      "slug": "design",
                      "name": "Design",
                      "name_de": "Design"
                  },
                  {
                      "id": 3,
                      "slug": "fashion-beauty",
                      "name": "Fashion & Beauty",
                      "name_de": "Mode & Sch\u00f6nheit"
                  },
                  {
                      "id": 4,
                      "slug": "food",
                      "name": "Food",
                      "name_de": "Essen"
                  },
                  {
                      "id": 5,
                      "slug": "literature",
                      "name": "Literature",
                      "name_de": "Literatur"
                  },
                  {
                      "id": 6,
                      "slug": "performing-arts",
                      "name": "Performing Arts",
                      "name_de": "Darstellende K\u00fcnste"
                  },
                  {
                      "id": 7,
                      "slug": "visual-arts",
                      "name": "Visual Arts",
                      "name_de": "Bildende K\u00fcnste"
                  }
              ]
          },
          {
              "id": 8,
              "slug": "business",
              "name": "Business",
              "name_de": "Wirtschaft",
              "subcategories": [
                  {
                      "id": 9,
                      "slug": "business-news",
                      "name": "Business News",
                      "name_de": "Wirtschaftsnachrichten"
                  },
                  {
                      "id": 10,
                      "slug": "careers",
                      "name": "Careers",
                      "name_de": "Karriere"
                  },
                  {
                      "id": 11,
                      "slug": "investing",
                      "name": "Investing",
                      "name_de": "Geldanlage"
                  },
                  {
                      "id": 12,
                      "slug": "management-marketing",
                      "name": "Management & Marketing",
                      "name_de": "Management & Markteting"
                  },
                  {
                      "id": 13,
                      "slug": "shopping",
                      "name": "Shopping",
                      "name_de": "Shopping"
                  }
              ]
          },
          {
              "id": 14,
              "slug": "comedy",
              "name": "Comedy",
              "name_de": "Kom\u00f6dien",
              "subcategories": []
          }
         ]
       ]
     }



### [GET /category]

#### Description

Retrieves the podcasts inside the specified category. The categories system referres to Apple's iTunes Categories. Categories not in Apple's specification are ignored on import.

#### Parameters

* **category_id (required, int)** the category's id (see /categories)
* **page (optional, int)** the page you want to address, default: 0
* **count (optional,int)** the page's size, default: 50

#### Response


    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "paging": {
              "count": 2,
              "page": 0,
              "first_page": 0,
              "last_page": 117,
              "next_page": 1,
              "prev_page": null
          },
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": {
          "category": {
              "id": 61,
              "slug": "professional",
              "title": "Professional",
              "url": "https:\/\/fyyd.de\/discover\/cat\/professional\/0"
          },
          "parent": {
              "id": 57,
              "slug": "sports-recreation",
              "title": "Sports & Recreation",
              "url": "https:\/\/fyyd.de\/discover\/cat\/sports-recreation\/0"
          },
          "podcasts": [
              {
                  "title": "vollraute - der gladbach podcast mit der raute im herzen -",
                  "id": 170,
                  "xmlURL": "http:\/\/feedpress.me\/vollrautepodcast",
                  "htmlURL": "http:\/\/vollraute.de",
                  "imgURL": "http:\/\/vollraute.de\/wp-content\/cache\/podlove\/5a\/cc055669c606ec882f928be09be530\/vollraute-der-gladbach-podcast-mit-der-raute-im-herzen_original.png",
                  "status": 390,
                  "slug": "vollraute-der-gladbach-podcast-mit-der-raute-im-herzen",
                  "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/170.jpg",
                  "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/170.png",
                  "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/170.png",
                  "language": "de",
                  "lastpoll": "2017-04-16 00:36:11",
                  "generator": "Podlove Podcast Publisher v2.4.1",
                  "categories": [
                      52
                  ],
                  "lastpub": "2017-04-11 21:14:53",
                  "rank": 1,
                  "url_fyyd": "https:\/\/fyyd.de\/podcast\/170",
                  "description": "Ein Podcast \u00fcber die Borussia aus M\u00f6nchengladbach.\nNie neutral, sondern immer durch die Vereinsbrille.",
                  "subtitle": "Ein Podcast \u00fcber die Borussia aus M\u00f6nchengladbach"
              },
              {
                  "title": "Collinas Erben",
                  "id": 296,
                  "xmlURL": "http:\/\/fokus-fussball.de\/feed\/mp3\/",
                  "htmlURL": "http:\/\/fokus-fussball.de",
                  "imgURL": "http:\/\/fokus-fussball.de\/wp-content\/cache\/podlove\/5f\/716d5e4a7909b9b87b75a456524d89\/collinas-erben_original.jpg",
                  "status": 304,
                  "slug": "collinas-erben",
                  "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/296.jpg",
                  "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/296.png",
                  "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/296.png",
                  "language": "de",
                  "lastpoll": "2017-04-17 16:12:25",
                  "generator": "Podlove Podcast Publisher v2.3.8",
                  "categories": [
                      52
                  ],
                  "lastpub": "2016-11-15 08:53:35",
                  "rank": 10,
                  "url_fyyd": "https:\/\/fyyd.de\/podcast\/296",
                  "description": "Fu\u00dfball ist ein einfaches Spiel \u2013 deshalb ist es so erfolgreich: Der Ball ist rund. Das Spiel dauert um die 90 Minuten.\n\nProblematisch wird es nur, wenn Sonderf\u00e4lle auftreten. Wenn ein Spieler pl\u00f6tzlich mit nacktem Oberk\u00f6rper auf den Zaun springt, wenn ein Engl\u00e4nder doch mal mit einem Wembley-Tor trifft oder wenn auch nach der zigsten Wiederholung nicht klar ist, ob es ein Foul in der 88. Minute war und ob es im oder vor dem Strafraum passierte.\n\nDer Schiedsrichter-Podcast Collinas Erben m\u00f6chte f\u00fcr mehr Fachwissen auf den Trib\u00fcnen und an den Empfangsger\u00e4ten sorgen. Alex Feuerherdt und Klaas Reese sprechen \u00fcber das Schiedsrichterwesen im Allgemeinen und die Spielregeln im Speziellen.",
                  "subtitle": "Der Schiedsrichter-Podcast"
              }
          ]
      }
    }



### [GET /podcast/recommend]

#### Description

Gets information about what fyyd thinks, might be a good idea to listen to also. At the moment, this is based on collections information.

#### Parameters

- **podcast_id (required, int)** the Podcast's id
- *or* **podcast_slug (alternative, string)** the podcast's slug
- **count (optional,int)** the number of recommended podcasts, default: 10

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Explikator",
            "id": 158,
            "xmlURL": "http:\/\/explikator.de\/feed\/podcast\/",
            "htmlURL": "http:\/\/explikator.de\/podcast",
            "imgURL": "http:\/\/explikator.de\/wp-content\/cache\/podlove\/ed\/92555a31e2f10f4922dd5b5b8b2b77\/explikator_original.png",
            "status": 304,
            "slug": "explikator",
            "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/158.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/158.png",
            "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/158.png",
            "language": "de",
            "lastpoll": "2017-04-23 10:13:56",
            "generator": "Podlove Podcast Publisher v2.3.18",
            "user_id": null,
            "categories": [
                  52
             ],
            "lastpub": "2017-04-20 00:01:21",
            "rank": 29,
            "url_fyyd": "https:\/\/fyyd.de\/podcast\/158",
            "description": "Jeden Werktag 10 Minuten Podcast und ein neuer Musiktitel. Themen sind Wissenschaft, Bewusstsein, Geschichte, Film & TV und Entertainment. Unbekanntes & Unbeachtetes, Hintergrund & Oberfl\u00e4che. Mal ernster, mal satirischer aber immer Morgen-Kaffee-tauglich!",
            "subtitle": "Morgenradio 2.0: Jeden Werktag 10 Minuten Unbeachtetes & Unbekanntes aus Gesellschaft und Kultur."
        } 
    ]
}
```



### [GET /podcast/latest]

#### Description

Returns information about the last added podcasts. Provide a count and/or an ID to be the starting point.

#### Parameters

- **since_id (optional, int)** the last podcast's id you know of. Get the podcasts added after that ID. Maximum: 1000 episodes.
- **count (optional, int, default: 20)** get the latest count nums of podcasts. 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Polytox Podcast (Polytox-Podcast)",
            "id": 49782,
            "xmlURL": "http:\/\/polytox.org\/feed\/mp3\/",
            "htmlURL": "http:\/\/polytox.org\/polytox-podcast",
            "imgURL": "http:\/\/polytox.org\/wp-content\/cache\/podlove\/a3\/c6f4f8b44edc93e3b7b3ef2205f69c\/polytox-podcast_original.jpg",
            "status": 200,
            "slug": "polytox-podcast-polytox-podcast",
            "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/49782.jpg",
            "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/49782.png",
            "smallImageURL": "https:\/\/img.fyyd.de\/pd\/small\/49782.jpg",
            "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/49782.png",
            "language": "de",
            "lastpoll": "2017-11-06T20:10:05+01:00",
            "generator": "Podlove Podcast Publisher v2.6.2",
            "categories": [],
            "lastpub": "2017-11-05T17:04:18+01:00",
            "rank": null,
            "url_fyyd": "https:\/\/fyyd.de\/podcast\/polytox-podcast-polytox-podcast\/0",
            "description": "Der Podcast des Polytox Zines mit Falk Fatal. Jede Folge gibt es Punk, Pop & Papperlapapp rund um die Subkultur.",
            "subtitle": "Der Subkultur-Podcast",
            "episode_count": 15
        }
    ]
}
```

### [GET /podcast/collections]

#### Description

Returns information about the collections this podcast is collected in.

#### Parameters

- **podcast_id (required, int)** the Podcast's id
- *or* **podcast_slug (alternative, string)** the podcast's slug
- **count (optional,int)** the number of recommended podcasts, default: 10
- **page (optional,int)** the page of the result, default: 0

#### Response

```

    "status": 1,
    "msg": "ok",
    "meta": {
        "paging": {
            "count": 3,
            "page": 0,
            "first_page": 0,
            "last_page": 19,
            "next_page": 1,
            "prev_page": null
        },
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "H\u00f6rt das!",
            "id": 48,
            "description": "",
            "layoutImageURL": "https:\/\/fyyd.de\/images\/collection300.png?et=",
            "smallImageURL": "https:\/\/fyyd.de\/images\/collection300.png?et=",
            "thumbImageURL": "https:\/\/fyyd.de\/images\/collection80.png?et=",
            "microImageURL": "https:\/\/fyyd.de\/images\/collection20.png?et=",
            "slug": "88610109b7c51842990bcfdbc1938696",
            "user_id": 1111,
            "url": "https:\/\/fyyd.de\/user\/Lesefreude\/collection\/88610109b7c51842990bcfdbc1938696"
        },
        {
            "title": "Bildung\/Schule\/Sozio",
            "id": 64,
            "description": "",
            "layoutImageURL": "https:\/\/fyyd.de\/images\/collection300.png?et=",
            "smallImageURL": "https:\/\/fyyd.de\/images\/collection300.png?et=",
            "thumbImageURL": "https:\/\/fyyd.de\/images\/collection80.png?et=",
            "microImageURL": "https:\/\/fyyd.de\/images\/collection20.png?et=",
            "slug": "bildung-schule-sozio",
            "user_id": 1135,
            "url": "https:\/\/fyyd.de\/user\/Anke\/collection\/bildung-schule-sozio"
        }
    ]
}
```



---



# Episode

Request to retrieve an episode's information

### [GET /episode]

#### Description

Returns information about a single episode

#### Parameters

- **episode_id (required, int)** the episode's id

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
       "data": {
            "title": "Space Hulk (Stay Forever, Folge 33)",
            "id": 42,
            "guid": "796a53669cb29b0504fdd4f0a3f9a41d",
            "url": "http:\/\/kaliban.podspot.de\/files\/Stay_Forever_Ep33_Space_Hulk.mp3",
            "enclosure": "http:\/\/kaliban.podspot.de\/files\/Stay_Forever_Ep33_Space_Hulk.mp3",
            "podcast_id": 5,
            "imgURL": "",
            "pubdate": "2014-03-06 18:10:30",
            "duration": null,
            "url_fyyd": "https:\/\/fyyd.de\/episode\/42",
            "description": "Christian hasst Space Hulk, Gunnar liebt es. Fight!"
        }
    }


### [GET /episode/latest]

#### Description

Returns information about the last added episodes. Provide a count and/or an ID to be the starting point.

#### Parameters

- **since_id (optional, int)** the last episode's id you know of. Get the episodes added after that ID. Maximum: 1000 episodes.
- **count (optional, int, default: 20)** get the latest count nums of episodes. 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "CR123 Biometrische Vollerfassung",
            "id": 3334,
            "guid": "http:\/\/chaosradio.ccc.de\/cr123.html",
            "url": "http:\/\/chaosradio.ccc.de\/cr123.html",
            "enclosure": "http:\/\/chaosradio.ccc.de\/archive\/chaosradio_123.mp3",
            "podcast_id": 119,
            "imgURL": "https:\/\/img.fyyd.de\/pd\/layout\/119.jpg",
            "pubdate": "2007-05-06T03:20:00+02:00",
            "duration": 10777,
            "url_fyyd": "https:\/\/fyyd.de\/episode\/3334",
            "description": "Fotofahndung, zentrale Fingerabdruckdatei und die Personenkennziffer"
        },
        {
            "title": "CR122 Der Bundestrojaner",
            "id": 3335,
            "guid": "http:\/\/chaosradio.ccc.de\/cr122.html",
            "url": "http:\/\/chaosradio.ccc.de\/cr122.html",
            "enclosure": "http:\/\/chaosradio.ccc.de\/archive\/chaosradio_122.mp3",
            "podcast_id": 119,
            "imgURL": "https:\/\/img.fyyd.de\/pd\/layout\/119.jpg",
            "pubdate": "2007-03-29T13:00:00+02:00",
            "duration": 10603,
            "url_fyyd": "https:\/\/fyyd.de\/episode\/3335",
            "description": "Nie wurden Ihre Grundrechte so verletzt..."
        }
    ]
}
```



### [GET /episode/curations]

#### Description

Returns information about the curations this episode is in.

#### Parameters

- **episode_id (required, int)** the episode's id
- **count (optional,int)** the number of recommended podcasts, default: 10
- **page (optional,int)** the page of the result, default: 0

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "paging": {
            "count": 2,
            "page": 0,
            "first_page": 0,
            "last_page": 1,
            "next_page": 1,
            "prev_page": null
        },
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Museumsfeed",
            "id": 593,
            "description": "Eine Zusammenstellung von Podcast-Episoden mit Museums- & Ausstellungsfokus: Besuche, Menschen, Projekte...",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/593.jpg?et=",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/593.png?et=",
            "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/593.jpg?et=",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/593.png?et=",
            "public": 1,
            "slug": "museumsfeed",
            "user_id": 1097,
            "type": 1,
            "categories": [
                52,
                1,
                15
            ],
            "url": "https:\/\/fyyd.de\/user\/tinowa\/curation\/museumsfeed",
            "xmlURL": "https:\/\/feeds.fyyd.de\/tinowa\/museumsfeed"
        },
        {
            "title": "Netzpolitik",
            "id": 623,
            "description": "Vorsicht, k\u00f6nnte Spuren von Netzpolitik enthalten!",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/623.jpg?et=",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/623.png?et=",
            "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/623.jpg?et=",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/623.png?et=",
            "public": 1,
            "slug": "netzpolitik",
            "user_id": 1000,
            "type": 1,
            "categories": [
                52,
                39,
                27
            ],
            "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/netzpolitik",
            "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/netzpolitik"
        }
    ]
}
```

---



# Curations

All about Curations inside fyyd. Retrieve and set.

### [GET /curation[/episodes]]

#### Description

Returns information about a single curation, given the curation's id. May append episodes, not paged at the moment. If you're authenticated and you request your own curation, non-public curations will be accessable too.

#### Authorization

required for getting your own curation, otherwise you only get public curations.

#### Parameters

- **curation_id (required, int)** the curation's id

#### Response


    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": {
          "title": "Testfeed",
          "id": 601,
          "description": "nur ein Testfeed",
          "layoutImageURL": "https:\/\/fyyd.de\/images\/curation300.png",
          "thumbImageURL": "https:\/\/fyyd.de\/images\/curation80.png",
          "microImageURL": "https:\/\/fyyd.de\/images\/curation20.png",
          "public": 1,
          "type": 1,
          "slug": "testfeed",
          "user_id": 1000,
          "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/testfeed",
          "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/testfeed",
          "episodes": [
              {
                  "title": "A!195 \u2013 Raussismus",
                  "id": 1662345,
                  "guid": "podlove-2017-04-14t12:27:42+00:00-56a19c3a4209a8d",
                  "url": "https:\/\/aufwachen-podcast.de\/2017\/04\/14\/a195-raussismus\/",
                  "enclosure": "https:\/\/aufwachen-podcast.de\/podlove\/file\/3244\/s\/feed\/c\/itunes\/195raussismus.m4a",
                  "podcast_id": 95,
                  "imgURL": "",
                  "pubdate": "2017-04-14 15:05:11",
                  "duration": 16018,
                  "favedDate": "2017-04-17 16:46:39",
                  "url_fyyd": "https:\/\/fyyd.de\/episode\/1662345",
                  "description": "Freitag, 14. April 2017, 15:05 Uhr<p>Frankreich steht seit Monaten Kopf."
              },
              {
                  "title": "LdN051 BVB-Bus, Steuern, Syrien, US-Au\u00dfen-\u201cPolitik\u201d, Altmaier",
                  "id": 1661432,
                  "guid": "podlove-2017-04-14t07:57:05+00:00-5ee52bc0e03190f",
                  "url": "https:\/\/www.kuechenstud.io\/lagedernation\/2017\/04\/14\/ldn051-bvb-bus-steuern-syrien-us-aussen-politik-altmaier\/",
                  "enclosure": "https:\/\/www.kuechenstud.io\/lagedernation\/podlove\/file\/1795\/s\/feed\/c\/mp3\/LdN051.mp3",
                  "podcast_id": 44980,
                  "imgURL": "",
                  "pubdate": "2017-04-14 10:07:59",
                  "duration": 3945,
                  "favedDate": "2017-04-17 16:45:58",
                  "url_fyyd": "https:\/\/fyyd.de\/episode\/1661432",
                  "description": "<p>Liebe Lage-Gemeinde!<\/p>\n<p>P\u00fcnktlich zu Ostertagen die neue Lage."
              }
          ]
      }
    }



### [POST /curation]

#### Description

Creates or modifies a curation. 

#### Authorization

required

#### Parameters

- **curation_id (opt, int)** when set, an existing curation is modified
- **title (req when created, string)** the title to set for this curation
- **description (opt, string)** the description for this curation
- **slug (opt, string)** the slug to set for this curation. If not set while creation, the title will be slugified. *When set, but empty while modifying a curation, also a slugified title will overwrite the existing slug*.
- **public (opt, int, def: 0)** defines, if this curation will be set to public and thus accessable for all.
- **image (opt, binary)** upload an optional image for the curation. Use multipart/form-data ([RFC 2388](http://www.ietf.org/rfc/rfc2388.txt)) and provide a content-type.
- **categories (opt, array)** A json encoded array of category IDs. These are the same you get by requesting /categories

#### Response

Creating and mofiying a curation returns the complete object of the curation as in GET /curation.



### [POST /curation/delete]

#### Description

Deletes a curation. Please note: there's no coming back, no questions, no backup. It's deleted!

Please note also: You cannot delete your very own personal curation, it's tied to your account.

#### Authorization

required

#### Parameters

- **curation_id (req, int)** the id of the curation you want to delete

#### Response

returns 204: No content



### [POST /curate]

#### Description

Adds OR removes the episode identified by episode_id into the curation identified by curation_id. The curation must be owned by the user identified by the accesstoken. The state of the episode inside this curation toggles with each call, the state is returned inside response's data section.

If you want to enforce a state without knowing what state the episode had before, use **force_state**.

#### Authorization

required

#### Parameters

- **curation_id (required, int)** the curation's id
- **episode_id (required, int)** the episode's id
- **why (optional, string)** a text to indicate, why you curate this episode
- **force_state (optional, bool)** enforce adding or removing the episode from the curation

#### Response

    {
        "status": 1,
        "msg": "ok",
        "meta": {
            "API_INFO": {
                "API_VERSION": 0.2
            }
        },
        "data": {
            "state": 1,
            "text": "added AdH039: Singende Nonne to TestKuration"
        }
    }


### [GET /curate]

#### Description

Retuns the state of the curation of an episode in a curation.

#### Authorization

required

#### Parameters

- **curation_id (required, int)** the curation's id
- **episode_id (required, int)** the episode's id

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": {
        "state": 1
    }
}
```



### [GET /category/curation]

#### Description

Retrieves the curations inside the specified category. The categories system referres to Apple's iTunes Categories.

Please note: Curations' categories calculate automatically by the podcasts' categories of the curated episodes. In future, those categories also could get set manually by the user if desired. At the moment the categories are calculated and thus could change with every curated episode. 

Additionally you can only find public curations with that function.

#### Parameters

- **category_id (required, int)** the category's id (see /categories)
- *or* **category_slug (required, string)** the category's slug  
- **page (optional, int)** the page you want to address, default: 0
- **count (optional,int)** the page's size, default: 50

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "paging": {
            "count": 50,
            "page": 0,
            "first_page": 0,
            "last_page": 0,
            "next_page": null,
            "prev_page": null
        },
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": {
        "category": {
            "id": 5,
            "slug": "literature",
            "title": "Literatur",
            "url": "https:\/\/fyyd.de\/discover\/cat\/literature\/0"
        },
        "parent": {
            "id": 1,
            "slug": "arts",
            "title": "Kunst",
            "url": "https:\/\/fyyd.de\/discover\/cat\/arts\/0"
        },
        "curations": [
            {
                "title": "Krieg der Poesie",
                "id": 569,
                "description": "Alle Episoden aus dem Krieg der Poesie der Geschichtenkapsel",
                "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/569.jpg?et=",
                "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/569.png?et=",
                "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/569.jpg?et=",
                "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/569.png?et=",
                "public": 1,
                "slug": "kriegderpoesie",
                "user_id": 1016,
                "type": 1,
                "categories": [
                    1,
                    5,
                    52
                ],
                "url": "https:\/\/fyyd.de\/user\/Herrvonspeck\/curation\/kriegderpoesie",
                "xmlURL": "https:\/\/feeds.fyyd.de\/Herrvonspeck\/kriegderpoesie"
            },
            {
                "title": "Jahresendfestkalender 2016",
                "id": 1008,
                "description": "Alle Geschichten, Gedichte, H\u00f6rspiele und Wortklaubereien, die im Rahmen des Jahresendfestkalenders 2016 erschienen.",
                "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/1008.jpg?et=",
                "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/1008.png?et=",
                "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/1008.jpg?et=",
                "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/1008.png?et=",
                "public": 1,
                "slug": "jefk2016",
                "user_id": 1016,
                "type": 1,
                "categories": [
                    1,
                    5,
                    52
                ],
                "url": "https:\/\/fyyd.de\/user\/Herrvonspeck\/curation\/jefk2016",
                "xmlURL": "https:\/\/feeds.fyyd.de\/Herrvonspeck\/jefk2016"
            },
            {
                "title": "alboh - h\u00f6rspiele",
                "id": 1072,
                "description": "",
                "layoutImageURL": "https:\/\/fyyd.de\/images\/curation300.png?et=",
                "thumbImageURL": "https:\/\/fyyd.de\/images\/curation80.png?et=",
                "smallImageURL": "https:\/\/fyyd.de\/images\/curation300.png?et=",
                "microImageURL": "https:\/\/fyyd.de\/images\/curation20.png?et=",
                "public": 1,
                "slug": "073e20f0dd79291717527c2405bad9ba",
                "user_id": 1409,
                "type": 1,
                "categories": [
                    1,
                    5,
                    39
                ],
                "url": "https:\/\/fyyd.de\/user\/alboh\/curation\/073e20f0dd79291717527c2405bad9ba",
                "xmlURL": "https:\/\/feeds.fyyd.de\/alboh\/073e20f0dd79291717527c2405bad9ba"
            }
        ]
    }
}
```



---



# Collections

This part is all about collections at fyyd. The content part of collections are podcasts.



### [GET /collection[/podcasts]]

#### Description

Returns information about a single collection, given the collection's id. May append podcasts, not paged at the moment.

#### Parameters

- **collection_id (required, int)** the collection's id

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": {
          "title": "Netzpolitik",
          "id": 1,
          "description": "(Netz)politische Podcasts",
          "layoutImageURL": "https:\/\/img.fyyd.de\/collection\/layout\/1.jpg",
          "thumbImageURL": "https:\/\/img.fyyd.de\/collection\/thumbs\/1.png",
          "microImageURL": "https:\/\/img.fyyd.de\/collection\/micro\/1.png",
          "slug": "netzpolitik",
          "user_id": 1000,
          "url": "https:\/\/fyyd.de\/user\/eazy\/collection\/netzpolitik",
          "podcasts": [
              {
                  "title": "#heiseshow (Audio)",
                  "id": 5037,
                  "xmlURL": "https:\/\/www.heise.de\/heiseshow.rss",
                  "htmlURL": "http:\/\/www.heise.de\/heiseshow.rss",
                  "imgURL": "https:\/\/www.heise.de\/imgs\/02\/5\/1\/1\/7\/THUMB_heiseshow_quadrat-b3e99ad376d2732e.jpg",
                  "status": 304,
                  "slug": "heiseshow-audio",
                  "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/5037.jpg",
                  "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/5037.png",
                  "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/5037.png",
                  "language": "de",
                  "lastpoll": "2017-04-17 05:20:40",
                  "generator": null,
                  "categories": [
                      52
                  ],
                  "lastpub": "2017-04-13 02:00:00",
                  "rank": 1,
                  "url_fyyd": "https:\/\/fyyd.de\/podcast\/5037",
                  "description": "Immer donnerstags um 16 Uhr diskutieren wir in der #heiseshow mit G\u00e4sten \u00fcber aktuelle Ereignisse aus der Hightech-Welt und der Netzpolitik",
                  "subtitle": "Jede Woche Donnerstag live (Audio)"
              },
              {
                  "title": "Breitband - Medien und digitale Kultur (ganze Sendung) - Deutschlandradio Kultur",
                  "id": 503,
                  "xmlURL": "http:\/\/www.deutschlandradiokultur.de\/podcast-breitband-medien-und-digitale-kultur-ganze-sendung.1552.de.podcast.xml",
                  "htmlURL": "http:\/\/www.deutschlandradio.de\/weiterleitung-breitband-de.233.de.html",
                  "imgURL": "http:\/\/www.deutschlandradiokultur.de\/media\/files\/2\/258cfe6db750912b0bb36410d2fdf775v2.jpg",
                  "status": 390,
                  "slug": "breitband-medien-und-digitale-kultur-ganze-sendung-deutschlandradio-kultur",
                  "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/503.jpg",
                  "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/503.png",
                  "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/503.png",
                  "language": "de",
                  "lastpoll": "2017-04-17 17:12:06",
                  "generator": null,
                  "categories": [
                      52
                  ],
                  "lastpub": "2017-04-15 13:05:01",
                  "rank": 14,
                  "url_fyyd": "https:\/\/fyyd.de\/podcast\/503",
                  "description": "Feed provided by Deutschlandradio. Click to visit.",
                  "subtitle": "Die Beitr\u00e4ge zur Sendung"
              }
           ]
    	}
    }



### [POST /collection]

#### Description

Creates or modifies a collection. 

#### Authorization

required

#### Parameters

- **collection_id (opt, int)** when set, an existing collection is modified
- **title (req when created, string)** the title to set for this collection
- **description (opt, string)** the description for this collection
- **slug (opt, string)** the slug to set for this collection. If not set while creation, the title will be slugified. *When set, but empty while modifying a collection, also a slugified title will overwrite the existing slug*.
- **public (opt, int, def: 0)** defines, if this collection will be set to public and thus accessable for all.
- **image (opt, binary)** upload an optional image for the collection.  Use multipart/form-data ([RFC 2388](http://www.ietf.org/rfc/rfc2388.txt)) and provide a content-type.

#### Response

Creating and mofiying a collection returns the complete object of the collection as in GET /collection.



### [POST /collection/delete]

#### Description

Deletes a collection. Please note: there's no coming back, no questions, no backup. It's deleted!

#### Authorization

required

#### Parameters

- **collection_id (req, int)** the ID of the collection you want to delete

#### Response

returns 204: No content



### [POST /collect]

#### Description

Adds OR removes the podcast identified by podcast_id into or from the collection identified by collection_id. The collection must be owned by the user identified by the accesstoken. The status of the podcast inside this collection toggles. 

If you want to enforce a state without knowing what state the podcast had before, use **force_state**.

#### Authorization

required

#### Parameters

- **collection_id (required, int)** the collection's id
- **podcast_id (required, int)** the podcasts's id 
- **force_state (optional, bool)** enforce adding or removing the podcast from the collection.


#### Response

    {
        "status": 1,
        "msg": "ok",
        "meta": {
            "API_INFO": {
                "API_VERSION": 0.2
            }
        },
        "data": {
            "state": 1,
            "text": "added Auf dem Holzweg to Gamechanger"
        }
    }


### [GET /collect]

#### Description

Retuns the state of the collection of a podcast in a collection.

#### Authorization

required

#### Parameters

- **collection_id (required, int)** the collection's id
- **podcast_id (required, int)** the podcast's id

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": {
        "state": 1
    }
}
```



------





# Alerts

Here you can request, modify and create alerts. Alerts are automatically fired searches that aggregate found episodes. 

### [GET /alert/[episodes]]

#### Description

This request outputs all data (optionally including episodes) for an alert 

#### Parameters

- **alert_id (required, int)** the alert's id.

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": "0.2"
        },
        "SERVER": "195.201.115.6",
        "duration": 0
    },
    "data": {
        "id": 4711,
        "term": "brexit",
        "slug": "brexit",
        "lastcheck": "2018-10-03 17:27:00",
        "dirty": 0,
        "dirty_feed": 0,
        "active": 1,
        "schedule": 3,
        "episodes": [
            {
                "title": "Nordirland: Warum eine Wiedervereinigung mit Irland kein Tabu ist",
                "id": 3046029,
                "guid": "f720fd08-8a37-49ef-8b7b-e9670db98bea",
                "url": "",
                "enclosure": "http:\/\/podcasts.srf.ch\/world\/audio\/Zwischen-den-Schlagzeilen_03-10-2018-1017.1538559667163.mp3?assetId=f720fd08-8a37-49ef-8b7b-e9670db98bea",
                "podcast_id": 48792,
                "imgURL": "https:\/\/img-1.fyyd.de\/pd\/layout\/48792effbaec22b537b263c09a3984805a407.jpg",
                "pubdate": "2018-10-03T10:17:00+02:00",
                "duration": 806,
                "status": 200,
                "num_season": 0,
                "num_episode": 0,
                "inserted": "2018-10-03T12:00:02+02:00",
                "url_fyyd": "https:\/\/api.fyyd.de\/episode\/3046029",
                "description": "Harter oder weicher Brexit: noch ist das Scheidungsprozedere von der EU offen. In Nordirland geschieht zur Zeit \u00dcberraschendes. Aktuelle Umfragen zeigen, dass im Fall eines harten Brexit eine Mehrheit f\u00fcr eine Wiedervereinigung mit Irland ist. Nicht nur Katholiken, sondern auch Protestanten.\r\n\r\nWas steht f\u00fcr Nordirland beim Brexit auf dem Spiel? Und warum muss man bef\u00fcrchten, dass alte innerirische Konflikte wieder aufbrechen k\u00f6nnten? SRF-Korrespondent Martin Alioth erkl\u00e4rt.",
                "content_type": "audio\/mpeg",
                "why": ""
            }
        ]
    }
}
```



### [POST /alert]

#### Description

Creates or modifies an alert. 

#### Authorization

required

#### Parameters

- **alert_id (opt, int)** when set, an existing alert is modified
- **term (req when created, string)** the term to search for 
- **slug (opt, string)** the slug to set for this alert. If not set while creation, the term will be slugified. *When set, but empty while modifying an alert, a slugified title will overwrite the existing slug*.

#### Response

Creating and mofiying an alert returns the complete object of the alert as in GET /curation.



### [POST /alert/delete]

#### Description

Deletes an alert. Please note: there's no coming back, no questions, no backup. It's deleted!

#### Authorization

required

#### Parameters

- **alert_id (req, int)** the id of the alert you want to delete

#### Response

returns 204: No content



---





# Search

This request is the only one in this section for now. The search API will grow in the near future, drafts are already written on how to manage this important part of fyyd's API.

### [GET /search/episode]

#### Description

This request tries to find an episode inside fyyd's database, matching any or some of a set of given criteria.

This reflects Skoon's need to find an episode's id to add to the user's curation. 

#### Parameters

* **title (optional, string)** the episode's title. Search might use parts of the string to find the episode.
* **guid (optional, string)** the episode's GUID as stated inside the podcasts feed.
* **podcast_id (optional, int)** the podcast's id in fyyd's database.
* **podcast_title (optional, string)** the podcast's title. Search might use parts of the string to find the podcast.
* **pubdate (optional, string)** the pubDate as stated inside the podcasts feed.
* **duration (optional, int)**  the duration of the episode in seconds.
* **url (optional, string)** the episode's url as stated inside the podcast's feed.
* **term (optional,string)** a search term to find inside the episodes.

Please note: title, guid, pubdate, duration, url and term add episodes together. Think of a logical OR.

In contrast to that, podcast_id and podcast_title restrict all episodes to podcasts matching to one of podcast_id or podcast_title.

* **count (optional, int, default: 10)** 

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "title": "FS195 Dein Butler ist ein Zombie",
              "id": 1597636,
              "guid": "podlove-2017-03-22t23:23:57+00:00-f1bfca935c876bc",
              "url": "http:\/\/freakshow.fm\/fs195-dein-butler-ist-ein-zombie",
              "enclosure": "https:\/\/tracking.feedpress.it\/link\/13453\/5554460\/fs195-dein-butler-ist-ein-zombie.m4a",
              "podcast_id": 85,
              "imgURL": "",
              "pubdate": "2017-03-23 02:16:34",
              "duration": 14632,
              "url_fyyd": "https:\/\/fyyd.de\/episode\/1597636",
              "description": "\n<p>Open Source Development \u2014 Ultraschall \u2014 Podlove Publisher \u2014 Podcast Empanzipation \u2014 Thunderbolt..."
          }
      ]
    }



### [GET /search/podcast]

#### Description

This request tries to find a podcast inside fyyd's database, matching any or some of a set of given criteria.

#### Parameters

- **title (optional, string)** the podcasts's title. Search might use parts of the string to find the podcast.
- **url (optional, string)** the podcast's url as stated inside the podcast's feed.
- **term (optional,string)** a search term to find inside the podcast.
- **langauge (optional,string)** filters results by language (de, en...)
- **generator (optional,string)** filters results by generator (podlove, castopod etc...)

Please note: title, url and term add episodes together. Think of a logical OR.

- **count (optional, int, default: 10)** 

#### Response

    {
      "status": 1,
      "msg": "ok",
      "meta": {
          "API_INFO": {
              "API_VERSION": 0.2
          }
      },
      "data": [
          {
              "title": "Freak Show",
              "id": 85,
              "xmlURL": "https:\/\/feeds.metaebene.me\/freakshow\/m4a",
              "htmlURL": "http:\/\/freakshow.fm",
              "imgURL": "http:\/\/freakshow.fm\/wp-content\/cache\/podlove\/04\/662a9d4edcf77ea2abe3c74681f509\/freak-show_original.jpg",
              "status": 390,
              "slug": "freak-show",
              "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/85.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/85.png",
              "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/85.png",
              "language": "de",
              "lastpoll": "2017-04-16 00:36:29",
              "generator": "Podlove Podcast Publisher v2.5.0.build315",
              "categories": [
                      52,
                      61
                  ],
              "lastpub": "2017-04-13 02:03:27",
              "rank": 32,
              "url_fyyd": "https:\/\/fyyd.de\/podcast\/85",
              "description": "Die muntere Talk Show um Leben mit Technik, das Netz und Technikkultur. Bisweilen Apple-lastig aber selten einseitig. Wir leben und lieben Technologie und reden dar\u00fcber. Mit Tim, hukl, roddi, Clemens und Denis. Freak Show hie\u00df irgendwann mal mobileMacs.",
              "subtitle": "Menschen! Technik! Sensationen!"
          },
          {
              "title": "Timmy Trumpet \u2013 Freak Show",
              "id": 36645,
              "xmlURL": "http:\/\/timmytrumpet.podtree.com\/feed\/podcast",
              "htmlURL": "http:\/\/timmytrumpet.podtree.com",
              "imgURL": "http:\/\/media2-timmytrumpet.podtree.com\/media\/itunes_image\/FreakShow_Radio_1400x1400.jpg",
              "status": 304,
              "slug": "timmy-trumpet-freak-show",
              "layoutImageURL": "https:\/\/img.fyyd.de\/pd\/layout\/36645.jpg",
              "thumbImageURL": "https:\/\/img.fyyd.de\/pd\/thumbs\/36645.png",
              "microImageURL": "https:\/\/img.fyyd.de\/pd\/micro\/36645.png",
              "language": "en",
              "lastpoll": "2017-04-17 03:58:39",
              "generator": "https:\/\/wordpress.org\/?v=4.5.2",
              "categories": 38,
              "lastpub": "2016-06-27 00:30:17",
              "rank": 0,
              "url_fyyd": "https:\/\/fyyd.de\/podcast\/36645",
              "description": "Australia\u2019s #1 DJ \/ Multi-Platinum selling artist Timmy Trumpet presents a weekly mix of the hottest tracks, bootlegs, mashups and more! Put your headphones on and turn the world off. Subscribe now.\ntimmytrumpet.com\nfb.com\/timmytrumpet\nTwitter.com\/timmytrumpet",
              "subtitle": "Timmy Trumpet - Freak Show\u2028"
          }
      ]
    }


### [GET /search/curation]

#### Description

This request tries to find a curation inside fyyd's database, matching any or some of a set of given criteria.

#### Parameters

- **category_id (optional, int)** the id of a podcast category that this curation belongs to
- **term (optional,string)** a search term to find inside the curation.


- **count (optional, int, default: 10)** 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
        {
            "title": "Netzpolitik",
            "id": 623,
            "description": "Vorsicht, k\u00f6nnte Spuren von Netzpolitik enthalten!",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/623.jpg?et=",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/623.png?et=",
            "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/623.jpg?et=",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/623.png?et=",
            "public": 1,
            "slug": "netzpolitik",
            "user_id": 1000,
            "type": 1,
            "categories": [
                52,
                39,
                27
            ],
            "url": "https:\/\/fyyd.de\/user\/eazy\/curation\/netzpolitik",
            "xmlURL": "https:\/\/feeds.fyyd.de\/eazy\/netzpolitik"
        },
        {
            "title": "Empfehlungen von chkpnt",
            "id": 649,
            "description": "Von mir f\u00fcr gut befundene Podcastfolgen.",
            "layoutImageURL": "https:\/\/img.fyyd.de\/curation\/layout\/649.jpg?et=",
            "thumbImageURL": "https:\/\/img.fyyd.de\/curation\/thumbs\/649.png?et=",
            "smallImageURL": "https:\/\/img.fyyd.de\/curation\/small\/649.jpg?et=",
            "microImageURL": "https:\/\/img.fyyd.de\/curation\/micro\/649.png?et=",
            "public": 1,
            "slug": "public",
            "user_id": 1122,
            "type": 1,
            "categories": [
                52,
                55,
                39
            ],
            "url": "https:\/\/fyyd.de\/user\/chkpnt\/curation\/public",
            "xmlURL": "https:\/\/feeds.fyyd.de\/chkpnt\/public"
        }
    ]
}
```
### [GET /search/user]

#### Description

This request tries to find users based on nick and/or fullname.

#### Parameters

- **nick (optional, string)** a nick or part of it
- **fullname (optional,string)** a full name or part of it


- **count (optional, int, default: 10)** 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        }
    },
    "data": [
       {
            "nick": "funkenstrahlen",
            "id": 1084,
            "fullname": "Stefan Trauth",
            "bio": "Podcasting, Netzpolitik, Development. Alles was mir so begegnet eben.",
            "url": null,
            "layoutImageURL": "https:\/\/img.fyyd.de\/user\/layout\/1084.jpg?et=877005a911b519ff36265925146c12d7",
            "thumbImageURL": "https:\/\/img.fyyd.de\/user\/thumbs\/1084.png?et=877005a911b519ff36265925146c12d7",
            "smallImageURL": "https:\/\/img.fyyd.de\/user\/small\/1084.jpg?et=877005a911b519ff36265925146c12d7",
            "microImageURL": "https:\/\/img.fyyd.de\/user\/micro\/1084.png?et=877005a911b519ff36265925146c12d7"
        },
       {
            "nick": "eazy",
            "id": 1000,
            "fullname": "Christian Bednarek",
            "bio": "Ich bin's nur, der Christian!",
            "url": "https:\/\/fyyd.de\/",
            "layoutImageURL": "https:\/\/img.fyyd.de\/user\/layout\/1000.jpg?et=88e5b04ddcc4c2c059aabefd84d15039",
            "thumbImageURL": "https:\/\/img.fyyd.de\/user\/thumbs\/1000.png?et=88e5b04ddcc4c2c059aabefd84d15039",
            "smallImageURL": "https:\/\/img.fyyd.de\/user\/small\/1000.jpg?et=88e5b04ddcc4c2c059aabefd84d15039",
            "microImageURL": "https:\/\/img.fyyd.de\/user\/micro\/1000.png?et=88e5b04ddcc4c2c059aabefd84d15039"
        }
    ]
}
```
### [GET /search/color]

#### Description

This request compares the most significant color found in the podcasts logo to a given RGB value. 

#### Parameters

- **rgb** (**required**, **string**) hexadecimal RGB values as RRGGBB


- **count (optional, int, default: 10)** 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "count": 1,
        "API_INFO": {
            "API_VERSION": "0.2"
        },
        "SERVER": "195.201.115.6",
        "duration": 1
    },
    "data": [
        {
            "title": "Stormpeonz",
            "id": 54528,
            "xmlURL": "http:\/\/www.stormpeonz.com\/feed\/peonzradio-feed\/",
            "htmlURL": "http:\/\/www.stormpeonz.com",
            "imgURL": "http:\/\/www.stormpeonz.com\/wp-content\/uploads\/2019\/02\/Podcast.png",
            "status": 200,
            "slug": "stormpeonz",
            "layoutImageURL": "https:\/\/img-1.fyyd.de\/pd\/layout\/5452850f1f8d7dc1331b7bc9d71e01225b87b.jpg",
            "thumbImageURL": "https:\/\/img-1.fyyd.de\/pd\/thumbs\/5452850f1f8d7dc1331b7bc9d71e01225b87b.png",
            "smallImageURL": "https:\/\/img-1.fyyd.de\/pd\/small\/5452850f1f8d7dc1331b7bc9d71e01225b87b.jpg",
            "microImageURL": "https:\/\/img-1.fyyd.de\/pd\/micro\/5452850f1f8d7dc1331b7bc9d71e01225b87b.png",
            "language": "de",
            "lastpoll": "2018-12-12T09:08:41+01:00",
            "generator": "Podlove Podcast Publisher v3.0.4",
            "categories": [
                142,
                150
            ],
            "lastpub": "2020-07-27T13:19:15+02:00",
            "rank": 2,
            "url_fyyd": "https:\/\/fyyd.de\/podcast\/stormpeonz\/0",
            "description": "Einmal die Woche besprechen wir das aktuelle Weltgeschehen aus dem Bereich Gaming, oder aber auch nicht so aktuelle Sachen.",
            "subtitle": "Der w\u00f6chentliche Gaming Podcast mit Sven und Thomas.",
            "tcolor": "#fff",
            "color": "#103424",
            "episode_count": "33",
            "iflags": null,
            "paymentURL": null,
            "author": "Thomas, Sven",
            "stats": {
                "medianduration": 0,
                "medianduration_string": "0m",
                "episodecount": 33,
                "pubinterval": 7,
                "pubinterval_string": "w\u00f6chentlich",
                "pubinterval_value": 7,
                "pubinterval_type": 2
            }
        }
    ]
}
```

----



# Feature

This will be an additional, a higher level interface to discover podcasts. You could take all the data the podcasts API deliveres, ranks including, to calculate your own hot and wanted stuff. But why not take my data? Here we are with the first request: Hot podcasts. 

### [GET /feature/podcast/hot]

#### Description

This request delivers the hottest podcasts in fyyd's directory. This are NOT charts. This one is about the most active (listener and producer) podcasts of the last eight days. Hotness values are created once a day for all languages with 50 or more podcasts listed. 

#### Parameters

- **count (optional, int, default: 10)** 
- **language (optional, var, default: none)** 

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        },
        "SERVER": "195.201.115.6",
        "duration": 1
    },
    "data": [
        {
            "title": "\u00d61 Betrifft Geschichte",
            "id": 41507,
            "xmlURL": "http:\/\/static.orf.at\/podcast\/oe1\/oe1_geschichte.xml",
            "htmlURL": "http:\/\/oe1.orf.at\/podcast\/",
            "imgURL": "http:\/\/files.orf.at\/podcast\/oe1\/img\/oe1_geschichte.png",
            "status": 200,
            "slug": "oe1-betrifft-geschichte",
            "language": "de",
            "lastpoll": "2018-05-20T18:37:18+02:00",
            "generator": "ORF.at Publisher for Podcast Services v1.0",
            "categories": [
                48,
                51
            ],
            "lastpub": "2018-05-18T00:00:00+02:00",
            "rank": 138,
            "url_fyyd": "https:\/\/fyyd.de\/podcast\/oe1-betrifft-geschichte\/0",
            "description": "Ob zu Rittern oder Terroristen - Expertinnen und Experten werden so geschickt gefragt, dass sich gut und gerne f\u00fcnf Minuten zuh\u00f6ren l\u00e4sst, wie sich eine Antwort entfaltet. Von Montag bis Freitag.",
            "subtitle": "Fachleute erz\u00e4hlen",
            "episode_count": 508
        },
        {
            "title": "Geschichtenkapsel",
            "id": 5095,
            "xmlURL": "http:\/\/geschichtenkapsel.de\/feed\/mp3\/",
            "htmlURL": "http:\/\/geschichtenkapsel.de",
            "imgURL": "http:\/\/geschichtenkapsel.de\/wordpress\/wp-content\/uploads\/2018\/04\/KapsleLogo_Geschichtenkapsel-1024x1024.png",
            "status": 200,
            "slug": "geschichtenkapsel",
            "language": "de",
            "lastpoll": "2018-05-20T18:38:08+02:00",
            "generator": "Podlove Podcast Publisher v2.7.6",
            "categories": [
                1,
                5,
                52,
                21,
                24
            ],
            "lastpub": "2018-05-20T13:37:39+02:00",
            "rank": 94,
            "url_fyyd": "https:\/\/fyyd.de\/podcast\/geschichtenkapsel\/0",
            "description": "L\u00e4sst du dir gerne Erz\u00e4hlungen, Kurzgeschichten, M\u00e4rchen oder Fabeln ins Ohr s\u00e4useln? Dann bist du hier richtig! In diesem Podcast gibt es selbstgeschriebene Geschichten, die du sonst nirgendwo findest! \u00d6ffne die Geschichtenkapsel und lasse dir fabelhafte Fantastereien, grausigen Grusel und wilde Worte um die Ohren fliegen!",
            "subtitle": "Lass dir was erz\u00e4hlen",
            "episode_count": 156
        }
  ]
}
```



### [GET /feature/podcast/hot/languages]

#### Description

As described above, only for all languages with 50 or more podcasts listed, hotness values are calculated. To get these languages, take this request.

#### Parameters

none

#### Response

```
{
    "status": 1,
    "msg": "ok",
    "meta": {
        "API_INFO": {
            "API_VERSION": 0.2
        },
        "SERVER": "195.201.115.6",
        "duration": 0
    },
    "data": [
        "en",
        "fr",
        "ru",
        "it",
        "de",
        "es",
        "nl"
    ]
}
```

------



# 3rd party interfaces

As this API grows over time, others have made great effords to create implementations for several languages and systems. 



## FyydKit 

[FyydKit](https://github.com/funkenstrahlen/FyydKit) is an implementation for Apple's swift. Stefan Trauth created it while developing [Skoon](https://skoon.stefantrauth.de/), an iOS app for browsing and administrating fyyd's curations. FyydKit is up to date with the API's v0.2.



## fyydAPI

[fyydAPI](https://github.com/JeanetteMueller/fyydAPI) is another switft implemenation but for v0.1. Jeanette Müller created it for her great podcatcher [Podcat](http://www.podc.at/). You can manage and browse your curations and collections inside podcat. 



## fyyd_ex

[fyyd_ex](https://github.com/optikfluffel/fyyd_ex) is an implementation for Elixir. It's WIP at the moment, but looks great to me. Well... I don't have the slightes idea of Elixir, but it really looks great, what [optikfluffel](https://twitter.com/optikfluffel) (remember superfav.de?) did there :)

