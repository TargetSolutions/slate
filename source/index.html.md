---
title: CrewSense API Reference

language_tabs:
  - shell: cURL
  - php: PHP

toc_footers:
  - <a href='https://crewsense.com/Application/ControlPanel/Options'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the CrewSense API documentation!
Find out how to send requests to our API, and what you can do with it.


```php
> Initiating a client
$client = new \GuzzleHttp\Client([
    'base_uri' => 'https://api.crewsense.com'
]);
```

The API is in a beta state and does not expose every part of the application, but you can already 
do lots of very useful things with it! Query the department schedule, manage labels, work types, 
time off types, list shift trades and much more.

Our examples are in PHP, using [Guzzle 6](http://docs.guzzlephp.org/en/latest/), and in shell script using curl.


# Authentication

Our API uses OAuth 2.0 for authentication, specifically the _client credentials_ grant type.  
The authentication process consists of requesting an access token with your organization's 
_API key_ and _API secret_, and using this access token in the `Authentication` HTTP header 
of any subsequent requests.

## Getting your API credentials

To generate API key/secret pairs, go to the [System Settings](https://crewsense.com/Application/ControlPanel/Options/) page, click "Integrations", and click "Generate new API credentials". The credentials will be listed in the table on that page.

If you no longer use the API credentials or you suspect they have been compromised, please delete them, and generate new ones instead, if needed.

> Retrieve an access token:

```shell
curl -v https://api.crewsense.com/oauth/access_token \
        -d "client_id=YOUR_CLIENT_ID" \
        -d "client_secret=YOUR_SECRET_KEY" \
        -d "grant_type=client_credentials"
```

```php
curl -v https://api.crewsense.com/oauth/access_token \
        -d "client_id=YOUR_CLIENT_ID" \
        -d "client_secret=YOUR_SECRET_KEY" \
        -d "grant_type=client_credentials"
```

> Substitute your credentials for `YOUR_CLIENT_ID` and `YOUR_SECRET_KEY`

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

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

