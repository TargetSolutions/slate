---
title: CrewSense API Reference

language_tabs:
  - shell: cURL


toc_footers:
  - <a href='https://crewsense.com/Application/ControlPanel/Options'>Sign Up for a Developer Key</a>

search: true
---

# Introduction

Welcome to the CrewSense API!
Find out how to send requests to our API, and what you can do with it.


```php
> Initiating a client
$client = new \GuzzleHttp\Client([
    'base_uri' => 'https://api.crewsense.com'
]);
```

At this time, the API is does not expose every part of the application, but you can already 
do lots of very useful things with it! Query the department schedule, manage labels, create reports, add / edit work types, add / edit users, view system log,
time off types, list shift trades and much more.

Our examples are in CURL.


# Authentication

Our API uses OAuth 2.0 for authentication, specifically the _client credentials_ grant type.  
The authentication process consists of requesting an access token with your organization's 
_API key_ and _API secret_, and using this access token in the `Authentication` HTTP header 
of any subsequent requests.

## Getting your API credentials

To generate API key/secret pairs, go to the [System Settings](https://crewsense.com/Application/ControlPanel/Options/) page, click "Integrations", and click "Generate new API credentials". The credentials will be listed in the table on that page.

If you no longer use the API credentials or you suspect they have been compromised, please delete them, and generate new ones instead, if needed.

## Receiving an access token

You use access tokens to authorize any requests made towards our API. To request an access token, issue a POST request to https://api.crewsense.com/oauth/access_token

> Receiving an access token:

```shell
curl -v https://api.crewsense.com/oauth/access_token \
     -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_SECRET_KEY" \
     -d "grant_type=client_credentials"
```

> If the request is successful and you credentials are correct, you should receive a JSON string like this:

```json
{
    "access_token": "DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN",
    "token_type": "bearer",
    "expires": 1426274440,
    "expires_in": 86400
}
```


<aside class="notice">
Substitute your credentials for `YOUR_CLIENT_ID` and `YOUR_SECRET_KEY` 
</aside>

The <code>token_type</code> signifies that you have to use HTTP headers to authorize requests (see the next section). expires is a UNIX timestamp of the expiration date of the token (after which you have to request a new one). <code>expires_in</code> shows the expiration length in seconds.

> Client access tokens currently expire in one week. If you try to use an expired access token, you might receive a response like:

```json
{
    "status": 401,
    "error": "unauthorized",
    "error_message": "Access token is not valid"
}
```
> In this case, you simply have to request a new access token using the method described above.

## Authorizing requests

To access protected resources in the API, you have to sign the HTTP requests with an <code>Authorization header</code>, using the <code>access_token</code> acquired in the previous step.

<code>Authorization: Bearer DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN</code>

> curl example

```shell
curl -v https://api.crewsense.com/v1/schedule \
     -H "Authorization: Bearer DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN"
```

# Conventions

We use a simplified version of the ISO 8601 standard. Dates are represented in the <code>YYYY-MM-DD</code> format. Most dates with timestamps follow the <code>YYYY-MM-DD hh:mm:ss</code> format (e.g. “2015-03-15 19:33:59”), where the timestamp is “timezoneless”, it is implied to be in your organization’s timezone (or the timezone is irrelevant).

For a few timestamp type data fields, we use the (still ISO 8601 standard) <codE>YYYY-MM-DDThh:mm:ss+00:00</code> format (example: “2015-03-21T19:45:33-06:00”). This is used for fields like contact time, response time, creation date etc., where the timezone may be important (due to daylight savings time for example).

# Schedule
## GET /schedule
### <span class="get">GET</span> /schedule

> An example of returned schedule data looks like this:

```json
{
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-22 08:00:00",

   "days": [
      {
         "date": "2015-03-15",
         "assignments": [
            {
               "shifts": [
                  ... shift data ...
               ],
            }
         ],
         "time_off": [
            {
               ... time off data ...
            }
         ],
         "callbacks": [
            {
               ... callback data ...
            }
         ],
         "trades": [
            {
               ... trade data ...
            }
         ],
         "misc": [
            {
               ... misc. hours data ...
            }
         ]
      }
   ],

   "prev": {
      "href": "https://api.crewsense.com/v1/schedule?start=2015-03-08%2008:00:00&end=2015-03-15%2008:00:00"
   },
   "next": {
      "href": "https://api.crewsense.com/v1/schedule?start=2015-03-22%2008:00:00&end=2015-03-22%2008:00:00"
   }
}
```

The schedule consists of shifts, time offs, callbacks, trades, notes, activities and misc. hours. They are wrapped by a top-level object containing metadata about the requested schedule (start date, end date, links for the next and previous period).

This endpoint has two required parameters:

### Query Parameters

Field | Description | Format
--------- | ------- | -----------
start | The date you need the data from | datetime
end | 	The date you need the data to | datetime

<aside class='warning'>While we are trying to make the API RESTful, some resources, including this one, are more of a convenient packaging of multiple resources for querying. You cannot issue a <code>POST</code> or <code>DELETE</code> request on this endpoint.</aside>

In the following sections, we try to introduce all the important data in the schedule resource.

## days

<code>days</code>

This array contains all days occurring between the start and end date requested. Each object in the array contains the <code>date</code> key, and arrays of objects occurring that day. For the following sections, we refer to one object in this array as a <code>day</code>.


## day.assignments

<code>day.assignments</code>

> day.assignments

This array contains the assignments of the day.

```json
{
   "id": 1234,
   "href": "https://api.crewsense.com/v1/assignments/1234",
   "date": "2015-03-15",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "name": "Station 1",
   "minimum_staffing": 3,
   "shifts": [
      ... see next section ...
   ]
}
```

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the assignment|integer
href |Link to full object	| string (URL)
date | The day the assignment starts on | date
start | Start date of assignment |datetime
end | End date of assignment | datetime
name | Title of assignment | string
minimum_staffing | Employees needed |	integer
shifts | Employees working the assignment	| array

## day.assignment.shifts

> day.assignment.shifts

This array holds data about the employees scheduled for the assignment on the given day.

```json
{
   "id": 456789,
   "href": "https://api.crewsense.com/v1/shifts/456789",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "hold_over": 0,
   "recurring": true,
   "user": {
      "id": 848,
      "href": "https://api.crewsense.com/v1/users/848",
      "name": "John Doe"
   },
   "scheduled_by": {
      "id": 138,
      "href": "https://api.crewsense.com/v1/users/138",
      "name": "Joe Boss"
   },
   "work_type": {
      "id": 33,
      "href": "https://api.crewsense.com/v1/work_types/33",
      "name": "Regular Time",
      "work_code": "REG001"
   },
   "labels": [
      {
         "id": 12,
         "href": "https://api.crewsense.com/v1/labels/12",
         "label": "ENG"
      }
   ]
}
```
### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the work shift|integer
href |Link to full object	| string (URL)
start | Start date of shift |datetime
end | End date of shift | datetime
recurring	| Is it a regularly occurring shift? | boolean
user|Employee working the shift|	See Users
scheduled_by|	Admin who assigned the shift|	See Users
work_type|	Type of work shift|	See section-wt
labels|	Applied Crew Scheduler labels|	array; see Labels

You will notice that some of the included objects have <code>href</code> properties. This is because we are only returning a sensible subset of the available data about these objects. If you make a <span class="get">GET</span> request to the provided URL, you can retrieve all of the available information about them.

## day.time_off

All approved time off for the day is in this array, including long term and recurring leave that has an occurrence fall on this day. 

> day.time_off

```json
{
   "id": 623492,
   "href": "https://api.crewsense.com/v1/time_off/623492",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "user": {
      "id": 848,
      "href": "https://api.crewsense.com/v1/users/848",
      "name": "John Doe"
   },
   "admin": {
      "id": 138,
      "href": "https://api.crewsense.com/v1/users/138",
      "name": "Joe Boss"
   },
   "time_off_type": {
      "id": 45,
      "href": "https://api.crewsense.com/v1/time_off_types/45",
      "name": "Sick Leave [SL]"
   }
}
```

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the time off|integer
href |Link to full object	| string (URL)
start | Start date of time off entry|datetime
end | End date of time off entry | datetime
user|Employee on time off| See Users
admin|	Admin who approved time off|	See Users
time_off_type|	Type of time off |	See Time off Types

## day.callbacks

> day.callbacks

In this array you will find all finalized callbacks for the day. Callback shifts that were drag & dropped to a work assignment will not be included, they are under <code>day.assignment.shifts</code>

```json
{
   "id": 64012,
   "href": "https://api.crewsense.com/v1/callbacks/64012",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "minimum_staffing": 1,
   "records": [
      {
         "id": 2165743,
         "user": {
            "id": 848,
            "href": "https://api.crewsense.com/v1/users/848",
            "name": "John Doe"
         },
         "start": "2015-03-15 08:00:00",
         "end": "2015-03-16 08:00:00",
         "work_site": null
      }
   ]
   "title": {
      "id": 112,
      "href": "https://api.crewsense.com/v1/titles/112",
      "name": "Firefighter"
   }
}
```

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the callback|integer
href |Link to full object	| string (URL)
start | Start date of the callback shift|datetime
end | End date of the callback shift| datetime
minimum_staffing| Number of employees needed in this callback| integer
records|	Accepting employees	array| ; see section-cbr
title|Employee type needed time off|	See section-title

<aside class="notice"><code>records</code> gives you all accepting employees of the callback. You can request more data about certain pieces of the callback using the <code>href</code> links provided.</aside>

## day.trades
> day.trades

<code>trades</code> contains all accepted and finalized shift trades for the day.

```json
{
   "id": 4355,
   "href": "https://api.crewsense.com/v1/trades/4355",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "requesting_user": {
      "id": 848,
      "href": "https://api.crewsense.com/v1/users/848",
      "name": "John Doe"
   },
   "accepting_user": {
      "id": 138,
      "href": "https://api.crewsense.com/v1/users/138",
      "name": "Jack Smith"
   },
   "admin": {
      "id": 98,
      "href": "https://api.crewsense.com/v1/users/98",
      "name": "Steve Boss"
   }
}
```
> Follow the top-level href link to receive all information about the trade.

## day.misc

> day.misc

```json
{
   "id": 47711,
   "href": "https://api.crewsense.com/v1/misc/47711",
   "date": "2015-03-16",
   "length": 4.5,
   "user": {
      "id": 848,
      "href": "https://api.crewsense.com/v1/users/848",
      "name": "John Doe"
   },
   "work_type": "Training"
}
```
This array provides data about any miscellaneous hours added for the day.

## day.notes, day.activities
This contains the Crew Scheduler notes for the day formatted in <code>HTML</code> format


# Time Off

## GET /time_off_types

<span class="get">GET</span> /time_off_types

Get all non-deleted time off types in the system. 

> /time_off_types

```json
[
   {
      "id": "5",
      "label": "Sick",
      "work_code": "SL",
      "required_buffer": "0.00",
      "instance_limit": "1",
      "primary_color": "#2474a9",
      "secondary_color": "#FFFFFF",
      "force_include": true,
      "forward": false,
      "href": "https://api.crewsense.com/v1/time_off_types/5"
   }
   {
      "id": "6",
      "label": "Vacation",
      "work_code": "VAC",
      "required_buffer": "0.00",
      "instance_limit": "0",
      "primary_color": "#3f5647",
      "secondary_color": "#FFFFFF",
      "force_include": false,
      "forward": true,
      "href": "https://api.crewsense.com/v1/time_off_types/6"
   }
]
```

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id|	Unique identifier of the time off type|	integer
href|	Link to full object|	string (URL)
label|	Name of the time off type|	string
work_code	|Shortcode of the time off type|	string
required_buffer|	Hours needed between request and start of the time off entry|	decimal
instance_limit|	Max. allowed number of this type in a yea|	integer
primary_color|	Main color of the type (background color)|	RGB hex
secondary_color|	Text color of the type|	RGB hex
force_include|	Ignore time off of this type in callbacks|	boolean
forward|	Forward time off of this type to other admins if not handled|	boolean

## POST /time_offs/:id/deny

Deny time off request

## POST /time_offs/:id/approve

Approve time off request

# Labels
## GET /labels

> GET /labels

<span class="get">GET</span> /labels

Receive a list of all crew scheduler labels available for the company. 

```json
[
   {
      "id": "1773",
      "label": "CPT",
      "color": "#CCCCCC",
      "text_color": "#333333",
      "position": "1"
   },
   {
      "id": "1774",
      "label": "ENG",
      "color": "#ff0000",
      "text_color": "#ffffff",
      "position": "2"
   }
]
```
### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id|	Unique identifier of the label|	integer
label|	The text appearing on the label|	string
color|	The background color of the label|	RGB hex
text_color|	The text color of the label|	RGB hex
position|	Relative position of shifts with this label|	integer


## GET /labels/{id}

> example: GET /labels/1773

<span class="get">GET</span> /labels

Receive the details of one particular label.


```json
{
   "id": "1773",
   "label": "CPT",
   "color": "#CCCCCC",
   "text_color": "#333333",
   "position": "1"
}
```

## POST /labels

<span class="post">POST</span> /labels

Create a new crew scheduler label in the system. Required fields:

<code>label</code> - the text on the label

<code>color</code> - the background color of the label, in HEX format (#RRGGBB)

<code>text_color</code> - the text color of the label, in HEX format

Optional fields:

<code>position</code> - The relative position of shifts with this label inside an assignment

## POST /labels/{id}

<span class="post">POST</span> /labels/{id}

Change an existing crew scheduler label in the system. Required fields:

<code>label</code> - the text on the label

<code>color</code> - the background color of the label, in HEX format (#RRGGBB)

<code>text_color</code> - the text color of the label, in HEX format

Optional fields:

<code>position</code> - The relative position of shifts with this label inside an assignment

## DELETE /labels/{id}

<span class="delete">DELETE</span> /labels/{id}

Remove an existing crew scheduler label from the system.

# Filters

Manage specialty classification filters

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id|	Unique identifier of the filter|	integer
label|	The name of the filter|	string
created_on|	Timestamp of the creation of this filter|	timestamp
user|	The user who created this resource|	User

## GET /filters

<span class="get">GET</span> /filters

> GET /filters example response:

Receive a list of all active specialty classification filters 

```json
[
   {
      "id": "7",
      "label": "Rescue Certified",
      "created_on": "2014-10-29T02:17:51-0700",
      "user": {
         id: "848",
         name: "John Doe"
      }
   },
   {
      "id": "8",
      "label": "Dive Team",
      "created_on": "2014-10-30T12:04:01-0700",
      "user": {
         id: "848",
         name: "John Doe"
      }
   }
]
```

## GET /filters/{id}

<span class="get">GET</span> /filters/{id}

> GET /filters/7 example response:

```json
{
   "id": "7",
      "label": "Rescue Certified",
      "created_on": "2014-10-29T02:17:51-0700",
      "deleted": "0",
      "user": {
         id: "848",
         name: "John Doe"
      }
}
```

The <code>deleted</code> key indicates if the filter has been deleted, 0 - active, 1 - deleted.

## POST /filters

<span class="post">POST</span> /filters

Create a new specialty classification filter in the system. Required fields:

<code>label</code> - the name of the new specialty classification filter

## POST /filters/{id}

<span class="post">POST</span> /filters/{id}

Change an existing specialty classification filter in the system. Required fields:

<code>label</code> - the new name of the classification filter

## DELETE /filters/{id}

<span class="delete">DELETE</span> /filters/{id}

Remove an existing specialty classification filter from the system.

# Users

## GET /users

> GET /users

```json
[
   {
      "user_id": 1234,
      "employee_id": "DEV123",
      "username": "olinagy",
      "first_name": "Oliver",
      "last_name": "Nagy",
      "full_name": "Oliver Nagy",
      "role": "Deputy",
      "emails": [
         "oli.nagy@example.com",
         "oli.nagy@otherexample.net"
      ],
      "phone_numbers": [
         "5555555555",
         "1231231232"
      ]
   },
   ...
]
```
<span class="get"> GET</span> /users

List all non-deleted, active users of the company

## GET /users/:id

> GET /users/:id

```json
{
   "user_id": 1234,
   "employee_id": "DEV123",
   "username": "olinagy",
   "first_name": "Oliver",
   "last_name": "Nagy",
   "full_name": "Oliver Nagy",
   "role": "Deputy",
   "emails": [
      "oli.nagy@example.com",
      "oli.nagy@otherexample.net"
   ],
   "phone_numbers": [
      "5555555555",
      "1231231232"
   ]
}
```
## GET /users/[user id]/titles

Returns all CallBack lists user is associated with

## GET /users/[user id]/filters

Returns all Speciality Filers user is associated with

## GET /users/[user id]/groups

Returns all Groups user is associated with

<span class="get"> GET</span> /users/:id

## PUT /users

<span class="put">PUT</span> /users

> PUT /users

```json
{
   "employee_id": "DEV123",
   "username": "olinagy",
   "first_name": "Oliver",
   "last_name": "Nagy",
   "full_name": "Oliver Nagy",
   "role": "Deputy",
   "phone": "5555555555",
   "mail": "info@example.com"
}
```
Create a new user. Send <code>JSON</code> data in the body

## PATCH /users/:id

> PATCH /users/:id

```json
{
   "first_name": "Oliver",
   "last_name": "Nagy"
}
```

<span class="patch"> PATCH</span> /users/:id

Update a user with the <code>user_id</code>. Send <code>JSON</code> data in the request body. You only need to send data that you want to update.

##  GET /users/:id/time_offs

Retrieve time off entries for user(s)

## POST /time_offs

Request time off

## PATCH /time_offs/:id

Edit time off Data

## GET /users/:user_id/timeoff/accrual/bank

> example: GET /users/1234/timeoff/accrual/bank

```json
[
   {
      "hours": 1001,
      "time_off_type": {
         "id": 5,
         "name": "Sick"
      }
   },
   {
      "hours": -125.024,
      "time_off_type": {
         "id": 6,
         "name": "Vacation"
      }
   },
   {
      "hours": 29.2125,
      "time_off_type": {
         "id": 7,
         "name": "Earned Time"
      }
   },

   ...
]
```

<span class="get">GET</span> /users/:user_id/timeoff/accrual/bank

Return the current time off bank of the user with the <code>user_id</code>. 


## GET /users/:user_id/timeoff/accrual/profile

```json
[
   {
      "time_off_type": {
         "id": 5,
         "name": "Sick"
      },
      "accrual_type": "Accrues 10 hours every 28 days"
   },
   {
      "time_off_type": {
         "id": 6,
         "name": "Vacation"
      },
      "accrual_type": "Accrues one hour every 10 hours worked (max. 12 hours)"
   },
   {
      "time_off_type": {
         "id": 7,
         "name": "Earned Time"
      },
      "accrual_type": "No automatic accrual"
   },

   ...

]
```
<span class="get">GET</span> /users/:user_id/timeoff/accrual/profile

Return the accrual type for each time off type based on the employee’s accrual profile.

# Logs

Query the system logs

## GET /logs(/:after)

> GET /logs(/:after)

```json
{
   "data": [
      {
         "log_id": 329473,
         "created": "2016-10-13T16:35:52+0200",
         "event_description": "CallBack Staffing logged in.",
         "user": {
             "id": 1,
             "name": "CallBack Staffing",
             "href": "https://api.crewsense.com/v1/users/1"
         }
      },
      ...
   ],
   "pagination": {
      "prev": "https://api.crewsense.com/v1/logs/150",
      "next": "https://api.crewsense.com/v1/logs/250"
   }
}
```
<span class="get"> GET</span> /logs(/:after)

Whenever any change is made in the system, we add a system log entry. The endpoints below allow access to these system logs.

We return 50 log entries per page. The <code>prev</code> and <code>next</code> links provide pagination through all of the system logs.

# Announcements

Manage system announcements of your company.

## GET /announcements

<span class="get">GET</span> /announcements

> GET /announcements

```json
{
   "data": [
        {
            "id": 238,
            "company_id": 8,
            "title": "Test",
            "body": "<p>This is a great HTML <strong>message!</strong></p>",
            "first_name": "Boss",
            "last_name": "Doe",
            "user": {
                "id": 848,
                "name": "Boss Doe",
                "href": "https://api.crewsense.com/v1/users/848"
            }
        },
        ...
    ],
    "pagination": {
        "prev": null,
        "next": null
    }
}
```

Retrieve the latest, non-deleted announcements.

## POST /announcements

<span class="post">POST</span> /announcements

Create a new company announcement.

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
body|	The text of the announcement, in HTML. | Required
title|	Announcement title

## PUT /announcements/:id

<span class="put"> PUT</span> /announcements/:id

Update a company announcement identified by <code>:id</code>.

Field | Description | Required?
--------- | ------- | -----------
body|	The text of the announcement, in HTML. | Required
title|	Announcement title


## DELETE /announcements/:id

<span class="delete">DELETE</span> /announcements/:id

Delete the announcement by the id <code>id</code>.

# Qualifiers

## GET /qualifiers

<span class="get">GET</span> /qualifiers

> GET /qualifiers

```json
[
   {
      "id": 7,
      "name": "Captain",
      "shortcode": "CPT",
      "modified_by": null,
      "modified_on": null
   },
   {
      "id": 8,
      "name": "Firefighter",
      "shortcode": "FF",
      "modified_by": null,
      "modified_on": null
   }
]
```

Retrieve all active qualifiers in your system. 

## GET /qualifiers/:id

> GET /qualifiers/:id

```json
{
   "id": 8,
   "name": "Firefighter",
   "shortcode": "FF-01",
   "title_id": 23,
   "created_by": 848,
   "created_on": "2015-11-18 05:19:27",
   "modified_by": null,
   "modified_on": null,
   "deleted_at": null,
   "deleted_by": null
}
```

Retreive all information about a specific qualifer by <code>id</code>.

## POST /qualifers

<span class="post">POST</span> /qualifiers

Creates a new qualifier.

> POST /qualifiers example response:

```json
{
   "inserted": 1
}
```
### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
name |	Descriptive name of the qualifier| Y
shortcode	| Shortened name of the qualifier, to be displayed on the Crew Scheduler

## DELETE /qualifiers/:id

<span class="delete">DELETE</span> /qualifiers/:id

Delete a qualifier. The qualifier will be soft-deleted, which means we can manually restore it if you think you’ve made a mistake deleting it.

## GET /qualifiers/:id/users
<span class="get">GET</span> /qualifiers/:id/users

> GET /qualifiers/:id/users

```json
[
   {
      "id": 98,
      "name": "Hass Brycen",
      "ranking": 0
   },
   {
      "id": 138,
      "name": "Oliver Nagy",
      "ranking": 0
   },
   ...
]
```

Retrieve all associated users of a qualifier. 

# Payroll

## GET /payroll

<span class="get">GET</span> /payroll

> GET /payroll

```json
[
  {
    "type": "work shift",
    "user_id": "19218",
    "employee_id": "417",
    "name": "DAVID SMITH",
    "work_type": "Regular Time",
    "work_code": "REG",
    "start": "2016-08-13 08:00:00",
    "end": "2016-08-14 08:00:00",
    "assignment_name": "Engine 1",
    "labels": "CPT",
    "notes": null,
    "total_hours": 24
  },
  ...
]
```

Returns all payroll data for date range / time. Can optionally return single employee payroll data by passing <code>user_id</code>

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
start |	start date / time of payroll range| Y
end	| end date / time of payroll range | Y
user_id | user id of employee |

# Forms

## GET /forms/:id/submissions

Returns submitted form data