---
title: CrewSense API Reference

language_tabs:
  - shell: cURL


toc_footers:
  - <a href='https://crewsense.com/Application/ControlPanel/Options'>Sign Up for a Developer Key</a>

search: true
---

# Introduction

Welcome to the CrewSense API

Find out how to send requests to our API, and what you can do with it using this developer resource.

The CrewSense API exposes very useful endpoints from within the CrewSense platform. You can do things like query the Schedule, manage Roster Data, Add Time Off's, Manage Trades, View System Log entries, CallBack History, and much more.

All of our examples are in cURL, however any scripting language that supports RESTful calls can be utilized.


# Authentication

Our API uses OAuth 2.0 for authentication, specifically the _client credentials_ grant type.  
The authentication process consists of requesting an access token with your organization's 
_API key_ and _API secret_, and using this access token in the `Authentication` HTTP header 
of any subsequent requests.

## Getting your API credentials

To generate API key/secret pairs, go to the [System Settings](https://crewsense.com/Application/ControlPanel/Options/) page, click "Integrations", and click "Generate new API credentials". The credentials will be listed in the table on that page. 

<aside class='warning'>note: Only users with System Settings permission will be able to access tokens / keys interface within the CrewSense platform</aside>

If you no longer use the API credentials or you suspect they have been compromised, please delete them, and generate new ones instead, if needed.

## Receiving an access token

> Receiving an access token:

```shell
curl -v https://api.crewsense.com/oauth/access_token \
     -d "client_id=YOUR_CLIENT_ID" \
     -d "client_secret=YOUR_SECRET_KEY" \
     -d "grant_type=client_credentials"
```

> If the request is successful and you credentials are correct, you should receive a JSON response like this:

```json
{
    "access_token": "DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN",
    "token_type": "bearer",
    "expires": 1426274440,
    "expires_in": 86400
}
```
You use access tokens to authorize any requests made towards our API. To request an access token, issue a <span class="post">POST</span> request to <code>https://api.crewsense.com/oauth/access_token</code>

<aside class="notice">
Substitute your credentials for `YOUR_CLIENT_ID` and `YOUR_SECRET_KEY` 
</aside>

The <code>token_type</code> signifies that you have to use HTTP headers to authorize requests (see the next section). 

<code>expires</code> is a UNIX timestamp of the expiration date of the token (after which you have to request a new one). 

<code>expires_in</code> shows the expiration length in seconds.

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

> curl example

```shell
curl -v https://api.crewsense.com/v1/schedule \
     -H "Authorization: Bearer DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN"
```

To access protected resources in the API, you have to sign the HTTP requests with an <code>Authorization header</code>, using the <code>access_token</code> acquired in the previous step.

<code>Authorization: Bearer DZs3IeaMP5uEAc2I19kJYl8Tbvsmgq9GaPQPaMjN</code>

# Conventions

We use a simplified version of the ISO 8601 standard. Dates are represented in the <code>YYYY-MM-DD</code> format. Most dates with timestamps follow the <code>YYYY-MM-DD hh:mm:ss</code> format (e.g. “2015-03-15 19:33:59”), where the timestamp is “timezoneless”, it is implied to be in your organization’s timezone (or the timezone is irrelevant).

For a few timestamp type data fields, we use the (still ISO 8601 standard) <codE>YYYY-MM-DDThh:mm:ss+00:00</code> format (example: “2015-03-21T19:45:33-06:00”). This is used for fields like contact time, response time, creation date etc., where the timezone may be important (due to daylight savings time for example).

# Changelog

## 12/04/2018

Batch processing finaliziations now available!  
See [`POST /finaliziation`](#post-finalization) and [`DELETE /finaliziation`](#delete-finalization) for details.

## 11/10/2018

* Implemented [`DELETE /time_offs/{id}`](#delete-time_offs-id) endpoint
* Added indicated groups to [shifts in `GET /schedule`](#day-assignment-shifts)
* Added color / text color to [Qualifiers](#qualifiers)

## 11/02/2018

[Signup Board](#signup-board) now available via API! Create new signup events, list signed up users, add users to the event and more.

## 10/31/2018

```json
{
    "length": 24,
    "break_start": "2018-10-31 12:00:00",
    "break_end": "2018-10-31 12:45:00",
    "break_length": 0.75,
}
```

### New fields in /schedule#day.assignment.shifts

The following fields have been added to the `GET /schedule` endpoint, under shifts:

* `length` - The full length of the shift in hours, including break periods.
* `break_start` - Start of the break period, if any. `null` otherwise.
* `break_end` - End of the break period, if any. `null` otherwise.
* `break_length` - Length of the break period, if any, in hours.

# Schedule

The schedule end point consists of shifts, time offs, callbacks, trades, notes, activities and misc. hours. They are wrapped by a top-level object containing metadata about the requested schedule (start date, end date, links for the next and previous period).

In the following sections, we try to introduce all the important data in the schedule resource.

## GET /schedule

> Example request

```shell
curl -v https://api.crewsense.com/v1/schedule -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "start=2015-03-15 08:00:00" \
     -d "end=2015-03-22 08:00:00"
```

> An example of returned /schedule data GET request looks like this:

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
                  ...shift data displayed here...
               ],
            }
         ],
         "time_off": [
            {
               ...time off data displayed here...
            }
         ],
         "callbacks": [
            {
               ...callback data displayed here...
            }
         ],
         "trades": [
            {
               ...trade data displayed here...
            }
         ],
         "misc": [
            {
               ... misc hours data displayed here...
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
<span class="get">GET</span> /schedule

This endpoint has two required parameters:

### Query Parameters

Field | Description | Format
--------- | ------- | -----------
start | The date you need the data from | datetime
end | 	The date you need the data to | datetime

<aside class='warning'>While we are trying to make the API RESTful, some resources, including this one, are more of a convenient packaging of multiple resources for querying. You cannot issue a <span class="post">POST</span> or <span class="delete">DELETE</span> request on this endpoint.</aside>

## days

This array contains all days occurring between the start and end date requested. Each object in the array contains the <code>date</code> key, and arrays of objects occurring that day. For the following sections, we refer to one object in this array as a <code>day</code>.

## day.assignments

```json
{
    "id": 1234,
    "href": "https://api.crewsense.com/v1/assignments/1234",
    "date": "2015-03-15",
    "start": "2015-03-15 08:00:00",
    "end": "2015-03-16 08:00:00",
    "name": "Station 1",
    "qualifiers_needed": [
        {
            "id": 7,
            "name": "Captains",
            "shortcode": "CPT",
            "color": "#d81b60",
            "text_color": "#FFFFFF",
            "minimum_staffing": 1
        },
        {
            "id": 8,
            "name": "Firefighters",
            "shortcode": "FF-01",
            "color": "#e53935",
            "text_color": "#000000",
            "minimum_staffing": 2
        }
    ],
    "minimum_staffing": 3,
    "is_finalized": true,
    "notes": "Assignment notes on date",
    "shifts": [
        ... see next section ...
    ]
}
```

This array contains the Crew Scheduler assignments of the day.

### Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the assignment|integer
href |Link to full object	| string (URL)
date | The day the assignment starts on | date
start | Start date of assignment |datetime
end | End date of assignment | datetime
name | Title of assignment | string
qualifiers_needed | Qualifier requirements for the assignment, if any. The `minimum_staffing` fields hold the number of people needed for the position. | array
minimum_staffing | Employees needed |	integer
is_finalized | Is the assignment finalized on the date | boolean
notes | Assignment notes on the date | string
shifts | Employees working the assignment	| array

## day.assignment.shifts

```json
{
   "id": 456789,
   "href": "https://api.crewsense.com/v1/shifts/456789",
   "start": "2015-03-15 08:00:00",
   "end": "2015-03-16 08:00:00",
   "length": 24,
   "break_start": "2015-03-15 12:00:00",
   "break_end": "2015-03-15 12:30:00",
   "break_length": 0.5,
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
      "work_code": "REG001",
      "color": "#3B4650",
      "text_color": "#FFFFFF",
      "subtypes": [
        {
          "id": 2,
          "label": "Mandatory"
        }
      ]
   },
   "labels": [
      {
         "id": 12,
         "href": "https://api.crewsense.com/v1/labels/12",
         "label": "ENG",
         "color": "#CCCCCC",
         "text_color": "#333333"
      }
   ],
   "qualifiers": [
      {
         "id": 7,
         "href": "https://api.crewsense.com/v1/qualifiers/7",
         "shortcode": "CPT",
         "name": "Captains",
         "color": "#d81b60",
         "text_color": "#FFFFFF"
      }
   ],
   "groups": [
      {
         "id": 2108,
         "label": "A Shift"
      }
   ]
}
```

This array holds data about the employees scheduled for the assignment on the given day.

### Fields

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the work shift|integer
href |Link to full object	| string (URL)
start | Start date of shift |datetime
end | End date of shift | datetime
length | The full length of the shift (including breaks) in hours | float
break_start | Start of the break period, if any | datetime
break_end | End of the break period, if any | datetime
break_length | Length of the break period in hours | float
recurring	| Is it a regularly occurring shift? | boolean
user|Employee working the shift|	See [Users](#get-users-user_id)
scheduled_by|	Admin who assigned the shift|	See [Users](#get-users-user_id)
work_type|	Type of work shift|	See [Work Types](#get-work_types)
labels|	Applied Crew Scheduler labels| See [Labels](#labels)
qualifiers | Applied CrewSense Qualifiers on the shift | See [Qualifiers](#qualifiers)
groups | User groups set to be indicated in the schedule | See [Groups](#groups)

<aside class="notice">
You will notice that some of the included objects have <code>href</code> properties. This is because we are only returning a sensible subset of the available data about these objects. If you make a <span class="get">GET</span> request to the provided URL, you can retrieve all of the available information about them.
</aside>

## day.time_off

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
      "name": "Vacation",
      "work_code": "VAC",
      "color": "#3f5647",
      "text_color": "#FFFFFF",
      "subtypes": [
        {
          "id": 1,
          "label": "Mandatory"
        },
        {
          "id": 3,
          "label": "Comp"
        }
      ]
   }
}
```

All approved time off for the day is in this array, including long term and recurring leave that has an occurrence fall on this day. 

### Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the time off|integer
href |Link to full object	| string (URL)
start | Start date of time off entry|datetime
end | End date of time off entry | datetime
user|Employee on time off| See [Users](#get-users-user_id)
admin|	Admin who approved time off|	See [Users](#get-users-user_id)
time_off_type|	Type of time off |	See [Time off Types](#get-time_off_types)

## day.callbacks

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
   ],
   "title": {
      "id": 112,
      "href": "https://api.crewsense.com/v1/titles/112",
      "name": "Firefighter"
   }
}
```

In this array you will find all finalized CallBacks for the day. CallBack shifts that were drag & dropped to a work assignment will not be included, as they are found under <code>day.assignment.shifts</code>

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id | Unique identifier of the callback|integer
href |Link to full object	| string (URL)
start | Start date of the callback shift|datetime
end | End date of the callback shift| datetime
minimum_staffing| Number of employees needed in this callback| integer
records|	Accepting employees | array; see [Callback Users](#get-callbacks-id-users)
title|Employee type needed time off|	See section-title

<aside class="notice"><code>records</code> returns all accepting employees pertinent to the CallBack. You can request more data about certain pieces of the CallBack data using the <code>href</code> links provided.</aside>

## day.trades

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

Contains all accepted and finalized shift trades for the day.

## day.misc

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

# Assignments

Allows you to retrieve, modify and create new CrewScheduler Assignments

## <span class="get">GET</span> /assignments

> Example response 

```json
[
  {
    "id": 123,
    "position": 3,
    "name": "Station 1",
    "date": "2014-01-22",
    "start": "2014-01-22 08:00:00",
    "end": "2014-01-23 08:00:00",
    "positions_to_fill": 4,
    "is_work_shift": true,
    "self_scheduling": true,
    "self_scheduling_requires_approval": true,
    "crewsense_eligible": true,
    "hide_open_slots": false,
    "color": "#CCCCCC",
    "location": {
        "address": "199 Northwest Hillcrest Drive",
        "city": "Grants Pass",
        "state": "OR",
        "country": "United States",
        "zip": "97526",
        "latitude": "42.45771410",
        "longitude": "-123.32473050"
    }
  },
  {
    "id": 124,
    "position": 4,
    "name": "Station 2",
    "date": "2014-01-22",
    "start": "2014-01-22 08:00:00",
    "end": "2014-01-23 08:00:00",
    "positions_to_fill": 3,
    "is_work_shift": true,
    "self_scheduling": true,
    "self_scheduling_requires_approval": true,
    "crewsense_eligible": true,
    "hide_open_slots": false,
    "color": "#CCCCCC",
    "location": null
  },
  ...
]
```

Returns all *non-archived* assignments of your organization. An assignment is archive
if it has an Until date in the past, or is non-recurring and has already ended.

The return format is an array of JSON objects.

### Properties

Name | Description | Format | Default
-----|-------------|--------|--------
id | The unique identifier of the assignment | integer | 
position | The order in which assignments should be displayed | integer | 
name | Assignment name | string | 
date | The date this assignment started on | date | 
start | The full date/time of the start of the first occurrence of the assignment | datetime | 
end | The full date/time of the end of the first occurrence of the assignment | datetime | 
positions_to_fill | Number of required personnel on the assignment | integer | 1
is_work_shift | If `false`, shifts in this assignment are ignored by the rest of the system | boolean | `true`
self_scheduling | Are employees allowed to schedule themselves | boolean | `false`
self_scheduling_requires_approval | Only if the above is true: if this is true, self scheduling requires the approval of an administrator | boolean | `false`
crewsense_eligible | Should this assignment be analyzed with CrewSense Intelligence | boolean | `false`
hide_open_slots | Condense the assignment in display, do not show vacant positions | boolean | `false`
color | Background color of the assignment for display purposes, in HEX format | string `#RRGGBB` | #F5F4F2
location | Detailed location information on the assignment. `null` if no location is set. | location | `null`

## <span class="get">GET</span> /assignments/{id}

```json
{
    "id": 670,
    "position": 1,
    "name": "Station 1",
    "date": "2012-12-25 00:00:00",
    "start": "2012-12-25 08:00:00",
    "end": "2012-12-26 08:00:00",
    "positions_to_fill": 2,
    "is_work_shift": true,
    "self_scheduling": true,
    "self_scheduling_requires_approval": true,
    "crewsense_eligible": true,
    "hide_open_slots": false,
    "color": "#F5F4F2",
    "location": {
        "address": "199 Northwest Hillcrest Drive",
        "city": "Grants Pass",
        "state": "OR",
        "country": "United States",
        "zip": "97526",
        "latitude": "42.45771410",
        "longitude": "-123.32473050"
    }
}
```

Retrieve a specific assignment.

For the meaning and format of properties, see [GET /assignments](#get-assignments).

## <span class="get">GET</span> /assignments/{id}/groups

Returns assignment group label data

```json
[
    {
        "id": "943",
        "name": "7367",
        "color": "#1700e6"
    }
]
```

## GET /assignments/{id}/admins

```json
[
    {
        "id": "953",
        "name": "John Smith",
        "href": "https://api.crewsense.com/v1/users/953"
    }
]
```

Returns data on who created the assignment

## PUT /assignments/{id}/admins

<span class="put">PUT</span> /assignments/{id}/admins

Updates admin data for who created the assignment

## <span class="post">POST</span> /assignments

Create a new Crew Scheduler Assignment.

### Parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
name | Assignment name | string | yes
start | The full date/time of the start of the first occurrence of the assignment | `2016-11-28 07:00:00` | yes
end | The full date/time of the end of the first occurrence of the assignment | `2016-11-28 19:00:00` | yes
positions_to_fill | Number of required personnel on the assignment | integer | yes
position | The order in which assignments should be displayed | integer | 
is_work_shift | If `false`, shifts in this assignment are ignored by the rest of the system | boolean | 
self_scheduling | Are employees allowed to schedule themselves | boolean | 
self_scheduling_requires_approval | Only if the above is true: if this is true, self scheduling requires the approval of an administrator | boolean | 
crewsense_eligible | Should this assignment be analyzed with CrewSense Intelligence | boolean | 
hide_open_slots | Condense the assignment in display, do not show vacant positions | boolean | 
color | Background color of the assignment for display purposes, in HEX format | string `#RRGGBB` | 

## PATCH /assignments/{id}

<span class="patch">PATCH</span> /assignments/{id}

Updates assignment info

## DELETE /assignments/{id}

<span class="delete">delete</span> /assignments/{id}

Deletes assignment

## PUT /assignments/{id}/groups

<span class="put">PUT</span> /assignments/{id}/groups

Updates assignment group label

## DELETE /assignments/{id}/groups

<span class="delete">DELETE</span> /assignments/{id}/groups

Deletes ALL assignment group label(s)

## DELETE /assignments/{id}/groups/{group_id}

<span class="delete">DELETE</span> /assignments/{id}/groups/{group_id}

Deletes specific assignment group label

## POST /assignments/{id}/finalization

> Example request

```shell
curl -v https://api.crewsense.com/v1/assignments/123/finalization \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "date=2015-03-15" \
     -d "admin_id=111"
```

> Example response (success)

```json
{
    "success": true
}
```

> Example response (already finalized on selected date)

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "Assignment (#123) has already been finalized on 2015-03-15."
    ]
}
```

> Example response (invalid admin ID)

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "The selected admin id is invalid."
    ]
}
```

Finalize an assignment on a given date. <span class="post">POST</span>ing to this endpoint will set the finalized flag &mdash; indicated by a green check mark in the Crew Scheduler &mdash; on the assignment on the date sent in the parameters.

### Request parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
date | The date to finalize the assignment on. | Date (YYYY-MM-DD) | **Y**
admin_id | ID of the admin user that finalizes the assignment. | Integer | **Y**


## POST /finalization

> Example request

```shell
curl -v https://api.crewsense.com/v1/finalization \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "{ \"assignments\": [ { \"id\": 882, \"dates\": [\"2018-12-01\", \"2018-12-05\"] } ], \"admin_id\": 848 }"
```

> Example response (success)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "All finaliziations processed successfully."
}
```

> Example response (already finalized on certain dates)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Finalizations processed with errors. See the error_messages field.",
    "success_messages": [
        "Assignment (#882) finalized on 2018-12-01."
    ],
    "error_messages": [
        "Assignment (#882) is already finalized on 2018-12-05."
    ]
}
```

> Example response (none succeeded, future date selected)

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "Assignment (#882) Finalization is only allowed on past and current dates."
    ]
}
```

> Example response (invalid admin ID)

```json
{
    "error": "not found",
    "status": 404,
    "error_message": "No query results for model [User]."
}
```

Batch apply finalization on selected assignments on selected dates and/or date ranges. <span class="post">POST</span>ing to this endpoint will set the finalized flag &mdash; indicated by a green check mark in the Crew Scheduler &mdash; on the assignments on the dates sent in the parameters.

### Request parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
assignments | An array of objects containing assignment IDs and dates to finalize. See below | array | **Y**
admin_id | ID of the admin user that finalizes the assignment. | Integer | **Y**

### Assignment object format

You can select individual dates or a range of dates, or both, for each assignment.
Dates in the `dates` array and between (inclusively) the `start` and `end` parameters will be merged and all processed.  
Either `dates` or `start` **and** `end` are required.

Name | Description | Format | Required?
-----|-------------|--------|----------
id | The ID of the assignment being finalized. | integer | **Y**
dates | Array of dates to finalize in YYYY-MM-DD format. | array | N
start | Start of the date range to finalize in YYYY-MM-DD format. | date | N
end | End of the date range to finalize in YYYY-MM-DD format. The end date will be included in the dates finalized. | date | N

## DELETE /assignments/{id}/finalization

> Example request

```shell
curl -v https://api.crewsense.com/v1/assignments/123/finalization \
     --request DELETE \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "{ \"date\": \"2015-03-15\" }"
```

> Example response (success)

```json
{
    "success": true
}
```

> Example response (not finalized on selected date)

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "Assignment (#123) is not finalized on 2015-03-15."
    ]
}
```

Remove finalization of an assignment on a given date. 

### Request parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
date | The date to remove finalization from the assignment. | Date (YYYY-MM-DD) | **Y**


## DELETE /finalization

> Example request

```shell
curl -v https://api.crewsense.com/v1/finalization \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     --request DELETE \
     -d "{ \"assignments\": [ { \"id\": 882, \"dates\": [\"2018-12-01\", \"2018-12-05\"] } ], \"admin_id\": 848 }"
```

> Example response (success)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "All finaliziations processed successfully."
}
```

> Example response (not finalized on certain dates)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Finalizations processed with errors. See the error_messages field.",
    "success_messages": [
        "Assignment (#882) finalized on 2018-12-01."
    ],
    "error_messages": [
        "Assignment (#882) is not finalized on 2018-12-05."
    ]
}
```

> Example response (none succeeded, future date selected)

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "Assignment (#882) Finalization is only allowed on past and current dates."
    ]
}
```

> Example response (invalid admin ID)

```json
{
    "error": "not found",
    "status": 404,
    "error_message": "No query results for model [User]."
}
```

Batch remove finalization on selected assignments on selected dates and/or date ranges. <span class="delete">DELETE</span>ing on this endpoint will remove the finalized flag &mdash; indicated by a green check mark in the Crew Scheduler &mdash; on the assignments on the dates sent in the parameters.

### Request parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
assignments | An array of objects containing assignment IDs and dates to unfinalize. See below | array | **Y**
admin_id | ID of the admin user that unfinalizes the assignment. | Integer | **Y**

### Assignment object format

You can select individual dates or a range of dates, or both, for each assignment.
Dates in the `dates` array and between (inclusively) the `start` and `end` parameters will be merged and all processed.  
Either `dates` or `start` **and** `end` are required.

Name | Description | Format | Required?
-----|-------------|--------|----------
id | The ID of the assignment being unfinalized. | integer | **Y**
dates | Array of dates to unfinalize in YYYY-MM-DD format. | array | N
start | Start of the date range to unfinalize in YYYY-MM-DD format. | date | N
end | End of the date range to unfinalize in YYYY-MM-DD format. The end date will be included in the dates finalized. | date | N

# Shifts

Allows you to retrieve, modify and create new CrewScheduler Shifts.
Recurrence rules are not supported at this time, only one-off shifts can be processed with these endpoints.

## <span class="get">GET</span> /shifts

```json
[
    {
        "id": 28871,
        "assignment_id": 670,
        "date": "2013-08-21",
        "start": "2013-08-21 07:00:00",
        "end": "2013-08-22 07:00:00",
        "user": {
            "id": 848,
            "name": "Boss Doe",
            "href": "https://api.crewsense.com/v1/users/848"
        },
        "scheduled_by": {
            "id": 98,
            "name": "Hass Brycen",
            "href": "https://api.crewsense.com/v1/users/98"
        },
        "work_type": {
            "id": 1,
            "href": "https://api.crewsense.com/v1/work_types/1",
            "name": "Regular time",
            "work_code": "REG001",
            "subtype_id": null
        },
        "labels":  [
            {
                "id": 123,
                "href": "https://api.crewsense.com/v1/labels/123",
                "label": "A Shift"
            }
        ],
        "qualifiers":  [
            {
                "id": 246027,
                "href": "https://api.crewsense.com/v1/qualifiers/246027",
                "shortcode": "FF-01",
                "name": "Firefighters"
            },
            {
                "id": 246027,
                "href": "https://api.crewsense.com/v1/qualifiers/246027",
                "shortcode": "TST",
                "name": "Test"
            }
        ],
        "traded_with": null,
        "from_callback": null,
        "notes": null,
        "created_at": "2018-03-06T08:45:00-08:00",
        "updated_at": "2018-03-06T08:45:00-08:00"
    },
    {
        "id": 35444,
        "assignment_id": 671,
        "date": "2013-08-18",
        "start": "2013-09-30 01:00:00",
        "end": "2013-09-30 08:00:00",
        "user": {
            "id": 848,
            "name": "Boss Doe",
            "href": "https://api.crewsense.com/v1/users/848"
        },
        "scheduled_by": {
            "id": 848,
            "name": "Boss Doe",
            "href": "https://api.crewsense.com/v1/users/848"
        },
        "work_type": {
            "id": 1,
            "href": "https://api.crewsense.com/v1/work_types/1",
            "name": "Regular time",
            "work_code": "REG001",
            "subtype_id": null
        },
        "labels": null,
        "qualifiers": null,
        "traded_with": null,
        "from_callback": null,
        "notes": null,
        "created_at": "2018-03-06T08:45:00-08:00",
        "updated_at": "2018-03-06T08:45:00-08:00"
    },
]
```

Retrieve all one-off non-recurring shifts of a certain user. This endpoint will be extended with more filtering options in the near future.

### Parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
user_id | ID of the user to query shifts for | integer | **yes**

## <span class="post">POST</span> /shifts

Create a new single-day shift for a user.

### Parameters

Name | Description | Format | Required?
-----|-------------|--------|----------
assignment_id | Assignment id | integer | **yes**
user_id | The ID of the user being assigned to a shift | integer | **yes**
date | The full date of the calendar day the shift should appear on | `2016-11-28` | **yes**
start | The full date/time of the start of the shift | `2016-11-28 07:00:00` | **yes**
end | The full date/time of the end of the shift | `2016-11-28 19:00:00` | **yes**
work_type_id | The ID of the work type to be used. See <span class="get">GET</span> /work_types | integer | **yes**
work_subtype_id | The ID of the work subtype to be used. See <span class="get">GET</span> /work_types | integer | no
notes | Any admin notes that should appear on the shift | string | no
from_callback | The ID of the Callback the shift is a result of | integer | no
traded_with | The ID of the user the shift is traded with | integer | no
labels | An array of Label IDs to apply to the shift. See <span class="get">GET</span> /labels | array | no
qualifiers | An array of Qualifier IDs to apply to the shift. See <span class="get">GET</span> /qualifiers | array | no

## <span class="get">GET</span> /shifts/{id}

See above for the output format

## <span class="delete">DELETE</span> /shifts/{id}

Permanently remove all instances of a shift from the schedule.

<aside class="warning">
    This action is not reversible! The shift data will be permanently lost. 
    Make sure not to delete recurring shifts from the system by accident.
</aside>

## <span class="patch">PATCH</span> /shifts/{id}

> Example PATCH payload

```json
{
    "date": "2018-03-01",
    "start": "2018-03-01 08:00:00",
    "end": "2018-03-01 20:00:00",
    "notes": "Changed to AM shift"
}
```

Update an existing shift. Provide any subset of the parameters described in the <span class="post">POST</span> /shifts section.
Currently, the `date`, `start` and `end` parameters are always required for a PATCH operation.

## <span class="get">GET</span> /shifts/{id}/qualifiers

See <span class="get">GET</span> /shifts `qualifiers` key for the output format

## <span class="post">POST</span> /shifts/{id}/qualifiers

Replace the qualifiers on the shift. Provide an array of qualifier ID's as the request body.

## <span class="delete">DELETE</span> /shifts/{id}/qualifiers

Remove all qualifiers from the shift.

## <span class="delete">DELETE</span> /shifts/{id}/qualifiers/{qualifierId}

Remove the qualifier in the second parameter from the shift.

## <span class="get">GET</span> /shifts/{id}/labels

See <span class="get">GET</span> /shifts `labels` key for the output format

## <span class="post">POST</span> /shifts/{id}/labels

Replace the labels on the shift. Provide an array of qualifier ID's as the request body.

## <span class="delete">DELETE</span> /shifts/{id}/labels

Remove all labels from the shift.

## <span class="delete">DELETE</span> /shifts/{id}/labels/{labelId}

Remove the label in the second parameter from the shift.

## GET /work_types

> Example request

```shell
curl -v https://api.crewsense.com/v1/work_types -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": 5,
        "label": "Earned time off",
        "work_code": "ETO",
        "primary_color": "#3B4650",
        "secondary_color": "#FFFFFF",
        "excludes_from_callback": true,
        "position": 0,
        "href": "https://api.crewsense.com/v1/work_types/5"
    },
    {
        "id": 7,
        "label": "Comp time",
        "work_code": null,
        "primary_color": "#3B4650",
        "secondary_color": "#FFFFFF",
        "excludes_from_callback": true,
        "position": 1,
        "href": "https://api.crewsense.com/v1/work_types/7"
    }
]
```

Get all *non-deleted* work types in the system. 

## GET /work_types/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/work_types/5 -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
{
    "id": 5,
    "label": "Earned time off",
    "work_code": "ETO",
    "primary_color": "#3B4650",
    "secondary_color": "#FFFFFF",
    "excludes_from_callback": true,
    "position": 0,
    "href": "https://api.crewsense.com/v1/work_types/5"
}
```

Get a specific work type in the system. 

## GET /work_types/{id}/subtypes

> Example request

```shell
curl -v https://api.crewsense.com/v1/work_types/2/subtypes -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": 4,
        "work_type_id": 2,
        "label": "Holdover",
        "work_code": "HO"
    },
    {
        "id": 8,
        "work_type_id": 2,
        "label": "Mandatory",
        "work_code": "M-OT"
    }
]
```

Get all subtypes of a work type.

## GET /work_subtypes

> Example request

```shell
curl -v https://api.crewsense.com/v1/work_subtypes -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": 4,
        "work_type_id": 2,
        "label": "Holdover",
        "work_code": "HO"
    },
    {
        "id": 8,
        "work_type_id": 2,
        "label": "Mandatory",
        "work_code": "M-OT"
    }
]
```

Get all *non-deleted* work subtypes in the system. 

# Visual Cycles

This endpoint will list all visual cycles an Organization is using in their CrewScheduler. Visual Cycles are used for displaying things like 'FLSA cycles graphically in their calendars.

In the following sections, we try to introduce all the important data in the schedule resource.

## GET /visual_cycles

> An example of returned /visual_cycles data GET request looks like this:

```json
[
   {
   "id": 2,
   "name": "FLSA Cycle #1",
   "start": "2017-01-01",
   "end": "2017-01-28",
   "color": "#f00",
   "text_color": "#fff",
   "created_by": 14690,
   "created_at": "2017-02-06 04:34:04",
   "deleted_by": null,
   "deleted_at": null
   }
,...
]
```
<span class="get">GET</span> /visual_cycles

### Optional Parameters

Passing the following parameters will return only visual cycles that fall within the start and end date

Name | Description | Format | Required?
-----|-------------|--------|----------
start | Start date | 2017-11-28 | no
end | End date | 2017-11-29 | no


## GET /visual_cycles/{id}

<span class="get">GET</span> /visual_cycles/{id}

## POST /visual_cycles

<span class="post">POST</span> /visual_cycles

### Required Parameters

When creating a new visual cycle, the following are required

Name | Description | Format | Required?
-----|-------------|--------|----------
name | name of visual cycle | string | y
start | start date of cycle | datetime | y
end | end date of cycle | datetime | y
color | rgb value of background color | rgbhex | y
text_color | rgb value of background color | rgbhex | y

## PUT /visual_cycles/{id}

<span class="put">PUT</span> /visual_cycles/{id}

## DELETE /visual_cycles/{id}

<span class="delete">DELETE</span> /visual_cycles/{id}


# Time Off

View, Edit and Create new Time Off entries in the system

## GET /time_offs

> Example request

```shell
curl -v https://api.crewsense.com/v1/time_offs \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": "744837",
        "time_off_type": {
            "id": "6",
            "name": "Vacation",
            "work_code": "VAC001"
        },
        "user": {
            "id": "98",
            "name": "Brycen Doe",
            "href": "/users/98"
        },
        "admin": {
            "id": "34942",
            "name": "Oliver CrewSense",
            "href": "/users/34942"
        },
        "start": "2017-07-10 07:00:00",
        "end": "2017-07-11 07:00:00",
        "rrule": null,
        "status": "1",
        "request_date": "2017-07-10T08:19:57-07:00",
        "approval_date": "1969-12-31T16:00:00-08:00",
        "acknowledgement_date": "1969-12-31T16:00:00-08:00",
        "user_note": null,
        "traded_with": null
    }
]
```

Get all time off entries for a given date range.

### Required Parameters

Field | Description | Type
--------- | ------- | -----------
start |	Start date of query timeframe|	datetime
end |	End date of query timeframe |	datetime

## POST /time_offs

<span class="post">POST</span> /time_offs

> POST /time_offs example JSON body:

```json
{
	"user_id": 138,
	"admin_ids": [848, 6461],
	"start_date": "2016-11-02 08:00:00",
	"end_date": "2016-11-03 08:00:00",
	"time_off_type_id": 456,
	"user_note": "I would like to go to Hawaii",
	"status": 1
}
```
Create a new Time Off entry in system

<aside class="notice">note: you can optionally approve the time off by calling /time_offs/{id}/approve or sending a status parameter of "1"</aside>

### Query Parameters

Field  | Description |Required?
--------- | --------- | -----------
user_id|	user id of employee | **Y**
admin_ids | user id of approving admin | **Y**
start_date | start date / time of time off entry| **Y**
end_date | end date / time of time off entry| **Y**
time_off_type_id | id of time off type to use| **Y**
status | pending or approved? *boolean (0 = pending, 1 = approved)* | N
user_note | brief note; Max limit 1000 characters | N

## DELETE /time_offs/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/time_offs/1234 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -X DELETE
```

> Example response (success)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Time Off removed successfully."
}
```

> Example response (not found)

```json
{
    "error": "not found",
    "status": 404,
    "error_message": "No query results for model [TimeOff]."
}
```

Delete a time off entry from the system.

<aside class="warning">
    This is an irreversible action! The time off entry (if recurring, all occurrences) will be permanently removed from the database.
</aside>


## POST /time_offs/{id}/deny
<span class="post">POST</span> /time_offs/{id}/deny

Deny time off request

## POST /time_offs/{id}/approve
<span class="post">POST</span> /time_offs/{id}/deny

Approve time off request

## GET /time_off_types

Get all *non-deleted* time off types in the system. 

<span class="get">GET</span> /time_off_types

> GET /time_off_types example response

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

## POST /time_off_types

Create new time off type in the system.

<span class="post">POST</span> /time_off_types

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

# Accruals

## GET /accruals
<span class="get">GET</span> /accruals

> GET /accruals example response

```json
[
    {
        "time_off_type": {
            "id": "37",
            "name": "Sick"
        },
        "accrual_type": "Accrues 6 hours every pay period (max. 150 hours)"
    },
    {
        "time_off_type": {
            "id": "38",
            "name": "Vacation"
        },
        "accrual_type": "Accrues 144 hours every year (max. 300 hours)"
    },
]
```
View all accrual profiles for company

## GET /users/{user_id}/timeoff/accrual/profile

> GET /users/1234/timeoff/accrual/profile example:

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
Return the accrual profiles for each user

<span class="get">GET</span> /users/{user_id}/timeoff/accrual/profile

## GET /users/{user_id}/timeoff/accrual/bank
<span class="get">GET</span> /users/{user_id}/timeoff/accrual/bank

> GET /users/{user_id}/timeoff/accrual/bank example response

```json
[
    {
        "hours": 592.5,
        "time_off_type": {
            "id": "5",
            "name": "Sick"
        }
    },
    {
        "hours": 746.7104,
        "time_off_type": {
            "id": "6",
            "name": "Vacation"
        }
    }
]
```
Returns all accrual balances a user

## POST /users/{user_id}/timeoff/accrual/bank/{id}
<span class="post">POST</span> /users/{user_id}/timeoff/accrual/bank

Updates users accrual bank balance. Reference time_off_type_id

### Required Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
time_off_type_id | You can issue a GET request to /time_off_types to get a list of your types in the system, and reference the id attribute in that list to find this ID | integer | **y**
hours | The number of hours to add to their bank. If you want to deduct hours, provide a negative number. | integer | **y**
admin_id | The User ID of the user issuing the manual adjustment. | integer | **y**
notes | Any notes you might want to save for the manual entry. | string | n

# Trades

## GET /trades
<span class="get">GET</span> /trades

> GET /trades example response

```json
   [
        {
            "id": "44799",
            "requesting_user": {
                "id": "956",
                "name": "Vincent Ownbey",
                "href": "https://api.crewsense.com/v1/users/956"
            },
            "request_date": "2016-05-10T17:16:15-07:00",
            "start": "2016-06-17T07:00:00-07:00",
            "end": "2016-06-18T07:00:00-07:00",
            "swap_start": "2017-07-14T13:21:18-07:00",
            "swap_end": "2017-07-14T13:21:18-07:00",
            "status": "filled"
        },
```

##GET /trades/{id}/users
<span class="get">GET</span> /trades/{id}/users

List all users who were apart of a specific trade and their responses / status

## POST /trades
<span class="post">POST</span> /trades

Initiate a new trade in the system

> GET /trades example response

```json
   [
        {
            "requesting_user": "956",
            "start": "2016-06-17 07:00:00",
            "end": "2016-06-18 07:00:00",
            "accepting_user_id": "145",
            "approving_user_id": "123"
        },
```

### Required Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
requesting_user | the person who requested the trade | integer | y
start | start date / time of trade | 2016-06-17 07:00:00 | y
end | end date | 2016-06-18 07:00:00 | y
approving_user_id | who approved the trade | integer | y


## PATCH /trades/{id}
<span class="patch">PATCH</span> /trades/{id}

## DELETE /trades/{id}
<span class="delete">DELETE</span> /trades/{id}

Deletes a trade from the system

## PUT /trades/{id}
<span class="put">PUT</span> /trades/{id}

# Labels
## GET /labels

> GET /labels

Receive a list of all crew scheduler labels available for the company. 

<span class="get">GET</span> /labels


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
position|	Relative CrewScheduler position order of this label|	integer


## GET /labels/{id}

> example: GET /labels/1773

Receive the details of one particular label.

<span class="get">GET</span> /labels


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

Create a new crew scheduler label in the system.

<span class="post">POST</span> /labels


### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
label | the text on the label | string | y
color | the background color of the label, in HEX format (#RRGGBB) | rgbhex | y
text_color | the text color of the label, in HEX format (#RRGGBB) | rgbhex | y
position | crewscheduler ordering position | integer | n

## POST /labels/{id}

Change an existing crew scheduler label in the system.

<span class="post">POST</span> /labels/{id}

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
label | the text on the label | string | y
color | the background color of the label, in HEX format (#RRGGBB) | rgbhex | y
text_color | the text color of the label, in HEX format (#RRGGBB) | rgbhex | y
position | crewscheduler ordering position | integer | n

## DELETE /labels/{id}

Remove an existing crew scheduler label from the system.

<span class="delete">DELETE</span> /labels/{id}


# Filters

## GET /filters

> GET /filters example response:

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

Receive a list of all active specialty classification filters 

<span class="get">GET</span> /filters

### Query Parameters

Field | Description | Type
--------- | ------- | -----------
id|	Unique identifier of the filter|	integer
label|	The name of the filter|	string
created_on|	Timestamp of the creation of this filter|	timestamp
user|	The user who created this resource|	User

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

The <code>deleted</code> key indicates if the filter has been deleted, <code>0</code> = active, <code>1</code>= deleted.

## POST /filters

Create a new specialty classification filter in the system. Required fields:

<span class="post">POST</span> /filters

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
label | the name of the new specialty classification filt | string | y

## POST /filters/{id}

Change an existing specialty classification filter in the system. Required fields:

<span class="post">POST</span> /filters/{id}

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
label | the new name of the new specialty classification filt | string | y

## DELETE /filters/{id}

Remove an existing specialty classification filter from the system.

<span class="delete">DELETE</span> /filters/{id}

# Groups

## GET /groups

> GET /groups example response:

```json
[
   {
      "id": "1",
      "label": "A Shift"
   },
   {
      "id": "2",
      "label": "B Shift"
   }
]
```

Receive a list of all groups

<span class="get">GET</span> /groups

## POST /groups

Create a new group in the system. Required fields:

<span class="post">POST</span> /groups

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
name | the name of the new group | string | y

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
      "avatar_url": "https://s3-us-west-2.amazonaws.com/cbs-file-storage/company-8/avatars/586cf2dd6db20-boss_doe-thumb.jpg",
      "date_of_hire": "2011-03-04",
      "date_of_birth": "1987-07-04",
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

List all *non-deleted, active* users of the company.

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
employee_id | The employee ID of the employee being searched for | string | n

<aside class="info">If an employee ID is provided in the parameters, instead of an array, we return just the User object found (if any).</aside>

## GET /users/{user_id}

<span class="get"> GET</span> /users/{user_id}

Returns specific user info. This call will retrieve any extra fields that you might have set up for the user in System Settings.

> GET /users/{user_id}

```json
{
   "user_id": 1234,
   "employee_id": "DEV123",
   "username": "olinagy",
   "first_name": "Oliver",
   "last_name": "Nagy",
   "full_name": "Oliver Nagy",
   "role": "Deputy",
   "avatar_url": "https://s3-us-west-2.amazonaws.com/cbs-file-storage/company-8/avatars/586cf2dd6db20-boss_doe-thumb.jpg",
   "date_of_hire": "2011-03-04",
   "date_of_birth": "1987-07-04",
   "emails": [
      "oli.nagy@example.com",
      "oli.nagy@otherexample.net"
   ],
   "phone_numbers": [
      "5555555555",
      "1231231232"
   ],
    "fields": [
        {
            "id": 1,
            "label": "Retired?",
            "value": "1"
        },
        {
            "id": 2,
            "label": "Gender",
            "value": "Male"
        },
        {
            "id": 3,
            "label": "Eligible for leave",
            "value": "0"
        }
    ]
}
```
## GET /users/{user_id}/titles

<span class="get"> GET</span> /users/{user_id}/titles

Returns all CallBack lists user is associated with

## PUT /users/{user_id}/titles

> PUT /users/{user_id}/titles example request

```json
{
    "ids": [101, 104]
}
```

> PUT /users/{user_id}/titles example response

```json
{
    "success": true
}
```

<span class="put"> PUT</span> /users/{user_id}/titles

Update the list of Callback Lists the user is associated with.
Pass the IDs in the `ids` parameter in the request body. The user's current Callback Lists will be overwritten with the ones provided in the parameter.

## GET /users/{user_id}/filters

<span class="get"> GET</span> /users/{user_id}/filters

Returns all Speciality Filers user is associated with

## PUT /users/{user_id}/filters

> PUT /users/{user_id}/filters example request

```json
{
    "ids": [10, 11, 55, 132]
}
```

> PUT /users/{user_id}/filters example response

```json
{
    "success": true
}
```

<span class="put"> PUT</span> /users/{user_id}/filters

Update the list of Specialty Filters the user is associated with.
Pass the IDs in the `ids` parameter in the request body. The user's current Specialty Filters will be overwritten with the ones provided in the parameter.

## GET /users/{user_id}/groups

<span class="get"> GET</span> /users/{user_id}/groups

Returns all Groups user is associated with

> GET /users/{user_id}/groups

```json
[
    {
        "id": 2097,
        "name": "Platoon 1"
    },
    {
        "id": 2128,
        "name": "A Shift"
    }
]
```

## PUT /users/{user_id}/groups

> PUT /users/{user_id}/groups example request

```json
{
    "ids": [2097, 2128]
}
```

> PUT /users/{user_id}/groups example response

```json
{
    "success": true
}
```

<span class="put"> PUT</span> /users/{user_id}/groups

Update the list of Groups the user is associated with.
Pass the group IDS in the `ids` parameter in the request body. The user's current groups will be overwritten with the ones provided in the parameter.

## PUT /users

<span class="put">PUT</span> /users

> PUT /users example JSON body

```json
{
   "employee_id": "DEV123",
   "username": "olinagy",
   "first_name": "Oliver",
   "last_name": "Nagy",
   "full_name": "Oliver Nagy",
   "role": "Deputy",
   "date_of_hire": "2011-03-04",
   "date_of_birth": "1987-07-04",
   "phone": "5555555555",
   "mail": "info@example.com"
}
```
Create a new user. Send <code>JSON</code> data in the body

### Parameters

Field | Description | Type | Required?
--------- | ------- | ----------- | -----------
username | the username for the user | string | y
first_name | first name | string | y
last_name | last name | string | y
mail | email address | string | y
password | password for the user | string | y
employee_id | internal employee id # | integer | n
phone_numbers | phone number for user (format: 5555555555) | integer | n
role | permission role for user | string | integer
date_of_hire | Hire/promotion date in YYYY-MM-DD format | date | n
date_of_birth | Birthdate in YYYY-MM-DD format | date | n

## PATCH /users/{user_id}

> PATCH /users/{user_id} example:

```json
{
   "first_name": "Oliver",
   "last_name": "Nagy"
}
```

<span class="patch"> PATCH</span> /users/{user_id}

Update a user with the <code>user_id</code>. Send <code>JSON</code> data in the request body. You only need to send data that you want to update.

##  GET /users/{user_id}/time_offs

Retrieve time off entries for a user

<span class="get"> GET</span> /users/{user_id}/time_offs

> GET /users/{user_id}/time_offs example response:

```json
[
    {
        "id": "744837",
        "time_off_type": {
            "id": "6",
            "name": "Vacation",
            "work_code": "VAC001"
        },
        "user": {
            "id": "98",
            "name": "Brycen Doe",
            "href": "/users/98"
        },
        "admin": {
            "id": "34942",
            "name": "Oliver CrewSense",
            "href": "/users/34942"
        },
        "start": "2017-07-10 07:00:00",
        "end": "2017-07-11 07:00:00",
        "rrule": null,
        "status": "1",
        "request_date": "2017-07-10T08:19:57-07:00",
        "approval_date": "1969-12-31T16:00:00-08:00",
        "acknowledgement_date": "1969-12-31T16:00:00-08:00",
        "user_note": null,
        "traded_with": null
    }
]
```

### Optional Parameters

The start/end parameters filter the results to time off entries occurring inside the selected timeframe.

Field | Description | Type
--------- | ------- | -----------
start | Start date of query timeframe. Defaults to one month ago. | datetime
end | End date of query timeframe. Defaults to the current date. | datetime

## POST /users/{user_id}/time_offs

Request time off for user

<span class="post"> POST</span> /users/{user_id}/time_offs

## PATCH /users/{user_id}/time_offs/{id}

Edit time off request Data

<span class="patch"> PATCH</span> /users/{user_id}/time_offs/{id}

# Certifications

These endpoints allow you to create, update and delete certifications. You can also use them to 
add or remove them to users.

## GET /certifications

> Example request

```shell
curl -v https://api.crewsense.com/v1/certifications \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": 1,
        "name": "Emergency",
        "cert_id": "EMERG",
        "created_by": {
            "id": 98,
            "name": "Hass Brycen",
            "href": "https://api.crewsense.com/v1/users/98"
        }
    },
    {
        "id": 2,
        "name": "First Responder",
        "cert_id": "FRS",
        "created_by": {
            "id": 98,
            "name": "Hass Brycen",
            "href": "https://api.crewsense.com/v1/users/98"
        }
    }
]
```

Returns all non-deleted Certifications in system. 

## GET /certifications/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/certifications/2 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
{
    "id": 2,
    "name": "First Responder",
    "cert_id": "FRS",
    "created_by": {
        "id": 98,
        "name": "Hass Brycen",
        "href": "https://api.crewsense.com/v1/users/98"
    }
}
```

Returns a specific Certification by ID. 

## POST /certifications

> Example request

```shell
curl -v https://api.crewsense.com/v1/certifications \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -H "Content-Type: application/json" \
     -d '{ "name": "First Responder", "cert_id": "FRS", "created_by": 123 }'
```

> Example response

```json
{
    "success": true,
    "code": 201,
    "status": "created",
    "message": "Added new certification First Responder.",
    "id": 321
}
```

Create a new certification. The `id` received in the response can be used to add the certification to users.

### Request Parameters

Key | Description | Type | Required
----|-------------|------|---------
name | The name of the new certification | string | **Y**
cert_id | The shortcode for the new certification. | string | **Y**
created_by | The ID of the user that should be indicated as the creator of the certification. | integer | **Y**

## PUT /certifications/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/certifications/321 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -H "Content-Type: application/json" \
     --request PUT \
     -d '{ "name": "Second Responder", "cert_id": "SRS" }'
```

> Example response

```json
{
    "success": true,
    "code": 200,
    "status": "created",
    "message": "Updated certification Second Responder.",
    "id": 321
}
```

Updates a certification. Only provided fields will be updated, other fields will remain unchanged.

### Request Parameters

Key | Description | Type | Required
----|-------------|------|---------
name | The name of the certification | string | N
cert_id | The shortcode for the certification. | string | N

## DELETE /certifications/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/certifications/321 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -H "Content-Type: application/json" \
     --request DELETE 
```

> Example response

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Deleted certification First Responder."
}
```

> Example response (already deleted)

```json
{
    "error": "bad_request",
    "status": 400,
    "error_message": "Certification already deleted."
}
```

Deletes a certification. Certifications in the system are soft-deleted, which means they can be restored alongside their users in an event of accidental deletion. They won't show up in GET requests after deleting.

## GET /users/{user_id}/certifications

<span class="get"> GET</span> /users/{user_id}/certifications

Returns all certifications of a user in the system.

## GET /users/{user_id}/certifications/{id}

<span class="get"> GET</span> /users/{user_id}/certifications/{id}

Returns specific certification info for a user.

## POST /users/{user_id}/certifications

<span class="post"> POST</span> /users/{user_id}/certifications

Add a certification to a user in the system.

## DELETE /users/{user_id}/certifications/{id}

<span class="delete"> DELETE</span> /users/{user_id}/certifications/{id}

Deletes a certification of a user in the system.

## PUT /users/{user_id}/certifications/{id}

<span class="put"> PUT</span> /users/{user_id}/certifications/{id}

Updates a certification record for an employee.

## DELETE /users/{user_id}/certifications

<span class="get"> GET</span> /users/{user_id}/certifications

Deletes all certifications of a user

## POST /users/{user_id}/certifications/{id}/file

<span class="post"> POST</span> /users/{user_id}/certifications/{id}/file

Uploads file (pdf, jpeg or doc) of certification for user


# Logs

Query the system logs

Whenever any change is made in the system, we add a system log entry. The endpoints below allow access to these system logs.

We return 50 log entries per page. The <code>prev</code> and <code>next</code> links provide pagination through all of the system logs.

## GET /logs

> GET /logs example:

```json
{
  "data": [
    {
      "log_id": "3359725",
      "created": "2016-10-21T20:49:50-0700",
      "event_description": "Brandon Rigaud logged in.",
      "user": {
        "id": "965",
        "name": "Brandon Rigaud",
        "href": "https://api.crewsense.com/v1/users/965"
      }
    },
    {
      "log_id": "3359722",
      "created": "2016-10-21T20:49:08-0700",
      "event_description": "Brandon CrewSense changed shift for Tim Stacy: 7308 now from 10-23-16 7:00 am to 10-24-16 7:00 am, originally from 10-23-16 7:00 am to 10-24-16 7:00 am Work Type: Overtime",
      "user": {
        "id": "34941",
        "name": "Brandon CrewSense",
        "href": "https://api.crewsense.com/v1/users/34941"
      }
    }...
]
}
````
<span class="get"> GET</span> /logs

Returns system log data. 

Can pass optional <code>after</code> parameter to return only logs after specific date / time.

<span class="get">GET</span> /logs/after

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
after|	date / time to return logs from | n

# CallBacks

## GET /callbacks

> GET /callbacks sample response:

```json
{
    "data": [ 
        {
            "id": 123456,
            "title": {
                "id": 456,
                "name": "Firefighter"
            },
            "initiated_by": {
                "id": 848,
                "name": "John Doe"
            },
            "start": "2016-11-09 08:00:00",
            "end": "2016-11-10 08:00:00",
            "positions_to_fill": 2,
            "scheduled_to": "2016-11-05T06:30:00-07:00",
            "started_at": "2016-11-05T06:30:00-07:00",
            "ended_at": "2016-11-05T07:12:44-07:00",
            "deadline": "2016-11-05T10:00:00-07:00"
            "status": "filled"
        },{
            "id": 123456,
            "title": {
                "id": 456,
                "name": "Firefighter"
            },
            "initiated_by": {
                "id": 848,
                "name": "John Doe"
            },
            "start": "2016-11-09 08:00:00",
            "end": "2016-11-10 08:00:00",
            "positions_to_fill": 2,
            "scheduled_to": "2016-11-05T06:30:00-07:00",
            "started_at": "2016-11-05T06:30:00-07:00",
            "ended_at": "2016-11-05T07:12:44-07:00",
            "deadline": "2016-11-05T10:00:00-07:00"
            "status": "filled"
        }
    ],
    "pagination": {
        "prev": "https://api.crewsense.com/v1/callbacks?offset=50",
        "next": "https://api.crewsense.com/v1/callbacks?offset=150"
    }
}
```

<span class="get">GET</span> /callbacks

Returns CallBacks in the system 

### Optional Parameters

Passing the following parameters will return only callbacks that fall within the start and end date

Name | Description | Format | Required?
-----|-------------|--------|----------
start | Start date | 2017-11-28 07:00:00 | no
end | End date | 2017-11-29 07:00:00 | no


## GET /callbacks/{id}

> GET /callbacks/123456 example

```json 
      {
            "id": 123456,
            "title": {
                "id": 456,
                "name": "Firefighter"
            },
            "initiated_by": {
                "id": 848,
                "name": "John Doe"
            },
            "start": "2016-11-09 08:00:00",
            "end": "2016-11-10 08:00:00",
            "positions_to_fill": 2,
            "scheduled_to": "2016-11-05T06:30:00-07:00",
            "started_at": "2016-11-05T06:30:00-07:00",
            "ended_at": "2016-11-05T07:12:44-07:00",
            "deadline": "2016-11-05T10:00:00-07:00",
            "status": "filled"
        }
```

<span class="get">GET</span> /callbacks/{id}/

Returns data for a specific CallBack

## GET /callbacks/{id}/users

> GET /callbacks/123456/users example response

```json
[
  {
    "id": "972",
    "position": "1",
    "status": "denied",
    "contact_method": "Mobile App",
    "contact_time": "2016-11-22 12:52:21",
    "response_time": "2016-11-22 13:00:39",
    "start_time": null,
    "end_time": null,
    "uses_opportunity": "1",
    "do_not_disturb": "0",
    "auto_accepted": "0"
  },
  {
    "id": "973",
    "position": "2",
    "status": "no answer",
    "contact_method": "Text (SMS)",
    "contact_time": "2016-11-22 12:58:25",
    "response_time": null,
    "start_time": null,
    "end_time": null,
    "uses_opportunity": "1",
    "do_not_disturb": "0",
    "auto_accepted": "0"
  }
]
```
<span class="get">GET</span> /callbacks/{id}/users

Returns data for specific users of the CallBack

# Signup Board

## GET /signup_events

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example request (only active signup events)

```shell
curl -v https://api.crewsense.com/v1/signup_events -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "status=active"
```

> Example response (no filters applied)

```json
[
    {
        "id": 5,
        "work_type_id": 1,
        "work_subtype_id": null,
        "title": "May the 4th be with you",
        "start": "2018-05-04 08:00:00",
        "end": "2018-05-05 08:00:00",
        "description": "<p>Testing!</p>",
        "created_by": 34942,
        "created_at": "2018-11-01T09:34:45-07:00",
        "modified_by": 34942,
        "modified_at": "2018-11-01T09:34:45-07:00",
        "employees_needed": 2,
        "status": "partially_filled"
    },
    {
        "id": 6,
        "work_type_id": 2,
        "work_subtype_id": 8,
        "title": "Cinco de Cuatro",
        "start": "2018-05-04 08:00:00",
        "end": "2018-05-05 08:00:00",
        "description": "<p>Chickens don't CLAP!</p>",
        "created_by": 34942,
        "created_at": "2018-09-05T05:32:36-07:00",
        "modified_by": 34942,
        "modified_at": "2018-09-05T05:32:36-07:00",
        "employees_needed": 10,
        "status": "active"
    }
]
```

Retrieve Signup Events. 

### Parameters

Name | Description | Type
-----|-------------|-----
id | Unique identifier of the Signup Event. | integer
work_type_id | The ID of the work type used for signed up employees. | integer
work_subtype_id | The ID of the subtype of the work type (if any) | integer
title | A descriptive name for the Signup Event. | string
start | Event start | datetime
end | Event end | datetime
description | Longer description of the Signup Event in HTML | string
created_by | The user ID of the person creating the event. | integer
created_at | ISO8601 timestamp of the creation date | ISO datetime
modified_by | The user ID of the person who last modified the event. | integer
modified_at | ISO8601 timestamp of the last modification date | ISO datetime
employees_needed | The number of employees required to sign up before the event is closed. | integer
status | The current status of the Signup Event. One of `active`, `partially_filled`, `filled`, `not_filled`, `deleted`

By default, all but deleted events will be returned from the system. You can apply some parameters to refine the returned result set:

### Query Parameters

Name | Description | Type
-----|-------------|-----
deleted | If set and has a positive value (ie. `1`, `true`), we only return deleted events. | boolean
archived | If set and has a positive value (ie. `1`, `true`), we only return non-deleted, non-active events. | boolean
status | If set, we only return non-deleted events of the given status. One of `active`, `partially_filled`, `filled`, `not_filled`. | string
start | If set, we only return non-deleted Signup Events with a `start` value greater than or equal to the given timestamp | datetime
end | If set, we only return non-deleted Signup Events with an `end` value less than or equal to the given timestamp | datetime

`start` and `end` can be used together to query a time period for Signup Events.

## GET /signup_events/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
{
    "id": 5,
    "work_type_id": 1,
    "work_subtype_id": null,
    "title": "May the 4th be with you",
    "start": "2018-05-04 08:00:00",
    "end": "2018-05-05 08:00:00",
    "description": "<p>Testing!</p>",
    "created_by": 34942,
    "created_at": "2018-11-01T09:34:45-07:00",
    "modified_by": 34942,
    "modified_at": "2018-11-01T09:34:45-07:00",
    "deleted_by": 34942,
    "deleted_at": "2018-11-05T11:24:23-07:00",
    "employees_needed": 2,
    "status": "deleted"
}
```

Return a specific Signup Event of the company. This endpoint will return data even for deleted Signup Events for auditing reasons, in which case `deleted_by` and `deleted_at` will not be `null`. Other parameters match those of a single value from the [GET /signup_events](#get-signup_events) endpoint.

## POST /signup_events

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d '{ "admin_id": 848, "title": "My Signup Event", \ 
           "work_type_id": 123, "work_subtype_id": 222, \
           "start": "2019-01-01 08:00:00", "end": "2018-01-02 08:00:00", \ 
           "description": "First shift in New Year", "employees_needed": 10 }'
```

> Example response (Event created successfully)

```json
{
    "success": true,
    "code": 201,
    "status": "created",
    "message": "Added signup event My Signup Event",
    "id": 7
}
```

Create a new Signup Event in the system that employees can sign up for. The newly created event will have a status `active`, which means it's immediately available for signup.

### Parameters

Name | Description | Type | Required?
-----|-------------|------|----------
admin_id | The user ID of the admin creating the Signup Event. | integer | **Y**
title | A descriptive name for the Signup Event. | string | **Y**
start | Event start | datetime | **Y**
end | Event end | datetime | **Y**
employees_needed | The number of employees required to sign up before the event is closed. | integer | **Y**
work_type_id | The ID of the work type used for signed up employees. | integer | **Y**
work_subtype_id | The ID of the subtype of the work type (if any) | integer | N
description | Longer description of the Signup Event in HTML | string | N

## PUT /signup_events/{id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -X PUT
     -d '{ "admin_id": 848, "title": "My Changed Signup Event", \ 
           "work_type_id": 123, "work_subtype_id": 222, \
           "start": "2019-01-01 08:30:00", "end": "2018-01-02 08:00:00", \ 
           "description": "First shift in New Year", "employees_needed": 10 }'
```

> Example response (Event updated successfully)

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Updated signup event My Changed Signup Event",
    "id": 7
}
```

Update a Signup Event. This is a <span class="PUT">PUT</span> request that requires *all* fields to be sent in again. This way you can <span class="GET">GET</span> a Signup Event, make some modifications, and <span class="PUT">PUT</span> the entire document to apply changes.

### Parameters

See [POST /signup_events parameters](#post-signup_events).

## GET /signup_events/{id}/users

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5/users \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr"
```

> Example response

```json
[
    {
        "id": 2,
        "user": {
            "id": 138,
            "name": "Olivér Nagy",
            "href": "https://api.crewsense.com/v1/users/138"
        },
        "misc_hour_id": 12969
    },
    {
        "id": 3,
        "user": {
            "id": 848,
            "name": "Boss Doe",
            "href": "https://api.crewsense.com/v1/users/848"
        },
        "misc_hour_id": 12970
    }
]
```

Lists the users signed up for the Event.

### Parameters

Name | Description | Type
-----|-------------|-----
id | Unique identifier of the Signup Event User record. | integer
user | User who signed up for the Event. | [User](#get-users)
misc_hour_id | The ID of the Misc. hours entry that was created when the user signed up. | integer

## POST /signup_events/{id}/users

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5/users \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d '{ "user_id": 848 }'
```

> Example response

```json
{
    "success": true,
    "code": 201,
    "status": "created",
    "message": "Signed Hass Brycen up for event My Changed Signup Event",
    "id": 7
}
```

> Example response if user is already signed up

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "User #848 is already signed up for event #5"
    ]
}
```

Sign a user up for an event. A Misc. hour entry will automatically be created for the Signup Event duration with the selected work type / subtype.

## DELETE /signup_events/{id}/users/{user_id}

> Example request

```shell
curl -v https://api.crewsense.com/v1/signup_events/5/users/848 \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -X DELETE
```

> Example response

```json
{
    "success": true,
    "code": 200,
    "status": "ok",
    "message": "Removed Hass Brycen from event My Changed Signup Event",
    "id": 8
}
```

> Example response if user is not signed up

```json
{
    "error": "invalid_params",
    "status": 422,
    "error_message": [
        "User #848 not found in Signup Event."
    ]
}
```

Remove a user from a Signup Event. The auto-created Misc. hour entry will also be removed.

# Availability

## GET /availability_periods


> GET /availability_periods sample response:

```json
[   
    {
        "id": 47348,
        "user": {
            "id": 98,
            "name": "Boss Doe",
            "href": "https://api.crewsense.com/v1/users/98"
        },
        "start": "2017-09-04 08:00:00",
        "end": "2017-09-05 08:00:00",
        "admin": {
            "id": 34940,
            "name": "Oliver Nagy",
            "href": "https://api.crewsense.com/v1/users/34940"
        },
        "notes": "Travelling out of country",
        "unavailability_reason": {
            "id": 6,
            "name": "Default"
        }
    }
]
```

<span class="get">GET</span> /availability_periods

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
start | date / time to return availability periods from | y
end | date / time to return availability periods to | y

## POST /availability_periods


> POST /availability_periods sample JSON body:

```json
{
    "user_id": 98,
    "start": "2017-11-17 08:00:00",
    "end": "2017-11-30 08:00:00",
    "unavailability_reason_id": 6,
    "admin_id": 98,
    "notes": "Added via the API"
}
```

<span class="get">POST</span> /availability_periods

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
user_id | The ID of the user that is (un)available | y
start | date / time to return availability periods from | y
end | date / time to return availability periods to | y
unavailability_reason_id | The ID of the unavailability reason (if any) | n
admin_id | The User ID of the admin adding the period. If omitted, it will appear as if the user added the period themselves | n
notes | Notes about the period | n

# Announcements

## GET /announcements

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
<span class="get">GET</span> /announcements

Retrieve the latest, non-deleted announcements.

## POST /announcements

<span class="post">POST</span> /announcements

Create a new company announcement.

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
body|	The text of the announcement, in HTML. | Required
title|	Announcement title

## PUT /announcements/{id}

Update a company announcement identified by <code>id</code>.

<span class="put"> PUT</span> /announcements/{id}

Field | Description | Required?
--------- | ------- | -----------
body|	The text of the announcement, in HTML. | Required
title|	Announcement title


## DELETE /announcements/{id}

<span class="delete">DELETE</span> /announcements/{id}

Delete the announcement by the id <code>id</code>.

# Qualifiers

## GET /qualifiers

> GET /qualifiers

```json
[
   {
      "id": 7,
      "name": "Captain",
      "shortcode": "CPT",
      "color": "#d81b60",
      "text_color": "#FFFFFF",
      "modified_by": null,
      "modified_on": null
   },
   {
      "id": 8,
      "name": "Firefighter",
      "shortcode": "FF",
      "color": "#d81b60",
      "text_color": "#FFFFFF",
      "modified_by": null,
      "modified_on": null
   }
]
```

<span class="get">GET</span> /qualifiers

Retrieve all active CrewSense intelligence qualifiers in your system. 

## GET /qualifiers/{id}

> GET /qualifiers/{id}

```json
{
   "id": 8,
   "name": "Firefighter",
   "shortcode": "FF-01",
   "color": "#d81b60",
   "text_color": "#FFFFFF",
   "title_id": 23,
   "created_by": 848,
   "created_on": "2015-11-18 05:19:27",
   "modified_by": null,
   "modified_on": null,
   "deleted_at": null,
   "deleted_by": null
}
```

<span class="get">GET</span> /qualifiers/{id}

Retreive all information about a specific qualifer by <code>id</code>.

## POST /qualifers

<span class="post">POST</span> /qualifiers

Create a new qualifier.

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

## DELETE /qualifiers/{id}

<span class="delete">DELETE</span> /qualifiers/{id}

Delete a qualifier. The qualifier will be *soft-deleted*, which means we can manually restore it if you think you’ve made a mistake deleting it.

## GET /qualifiers/{id}/users

> GET /qualifiers/{id}/users

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

<span class="get">GET</span> /qualifiers/{id}/users

Retrieve all associated users of a specific qualifier. 

# Payroll

## GET /payroll

> Example request

```shell
curl -v https://api.crewsense.com/v1/payroll -G \
     -H "Authorization: Bearer CKRskOAU2tqYItxqlGnTt0VwXm4L0QABIvYrTBPr" \
     -d "start=2016-08-13 08:00:00" \
     -d "end=2016-08-14 08:00:00"
```

> Example response

```json
[
  {
    "type": "work shift",
    "user_id": "19218",
    "employee_id": "417",
    "name": "DAVID SMITH",
    "work_type": "Regular Time",
    "work_subtype": "Weekday",
    "work_code": "REG01",
    "start": "2016-08-13 08:00:00",
    "end": "2016-08-14 08:00:00",
    "assignment_id": 670,
    "assignment_name": "Engine 1",
    "labels": "CPT",
    "notes": "Worked as captain due to jones being sick",
    "total_hours": 24
  },
  ...
]
```

<span class="get">GET</span> /payroll

Returns all payroll data for date range / time. 

<span class="get">GET</span> /payroll/{user_id}

Returns payroll data for the User in the given date range.

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
start |	start date / time of payroll range| **Y**
end	| end date / time of payroll range | **Y**
user_id | user id of employee | N

# Forms

## GET /forms/

> Example response

```json
{
    "141": {
        "title": "Test form",
        "created_at": "2016-08-05 09:32:46",
        "created_by": "Brandon CrewSense"
    },
    "196": {
        "title": "Exposure Form",
        "created_at": "2016-08-06 16:49:33",
        "created_by": "Casey CrewSense"
    },
    "203": {
        "title": "Truck Check - Daily",
        "created_at": "2016-03-01 22:04:18",
        "created_by": "John Doe"
    }
}
```
<span class="get">GET</span> /forms/

Lists all forms in the system

## GET /forms/{id}/submissions

> example GET /forms/144/submissions:

```json
[
  {
    "user": {
      "id": "953",
      "name": "Casey McIntosh",
      "href": "https://api.crewsense.com/v1/users/953"
    },
    "submitted_at": "2015-09-23T19:27:28-0700",
    "submission": [
      {
        "title": "Date of Exposure",
        "value": "9/23/2015"
      },
      {
        "title": "Type of Exposure",
        "value": "Chemical"
      },
      {
        "title": "PPE Worn",
        "values": [
          "Gloves",
          "Eye Protection"
        ]
      },
      {
        "title": "Location of Exposure",
        "value": "Haz-Co"
      },
      {
        "title": "Employee Name",
        "value": "Joe Smith"
      },
      {
        "title": "Description of Exposure",
        "value": "While working in splash area; bleach was sprayed into the face of employee. No medical attention was required and standard cleanup processes were followed."
      },
      {
        "title": "Witnesse(s)",
        "value": "Jeff Davis"
      },...
]
```

<span class="get">GET</span> /forms/{id}/submissions

Returns submitted form data for a specific form

# ReadyAlert

## POST /ready_alerts

> POST /ready_alerts example JSON

```json
{
   "type": "all",
   "ids": null,
   "body": "2nd alarm fire in town. Please respond ASAP!",
   "user_id": "124",
   "email": 0
}
```
<span class="post"> POST</span> /ready_alerts

Send ReadyAlert

<aside class="info">note: Rate limits are in effect. Limited to 30 ReadyAlerts per day via API. To increase this limit, please contact us.</aside>

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
type |	(all, title, group, classification, user)| Y
ids | ids of type, array | Y unless ALL is used
body	| the body of the message| Y
user_id | user id of employee sending ReadyAlert| Y
email | (boolean) send email as well? | 

# Company

## GET /company
<span class="get">GET</span> /company

Returns company information

# Bulletin Board

## GET /bulletin_board

> GET /bulletin_board example JSON

```json
{
    "content": "<h1>Hey there!!!</h1><p>Please read the bulletin board</p>",
    "last_updated_by": {
        "id": 848,
        "name": "Boss Doe",
        "href": "https://api.crewsense.com/v1/users/848"
    },
    "last_updated_at": "2018-02-27T19:04:29+01:00"
}
```
<span class="get"> GET</span> /bulletin_board

Get the currently active Bulletin Board data. HTML is sanitized on the input side, it should be safe to output the `content` without further filtering.

## PUT /bulletin_board

> PUT /bulletin_board example request body

```json
{
    "admin_id": 848,
    "content": "<h1>Hey there!!!</h1><p>Please read the UPDATED bulletin board</p>"
}
```
<span class="put"> PUT</span> /bulletin_board

Replaces the existing bulletin board entry with the one you send in the `content` parameter.
Note that if the Bulletin Board is opened for editing in the web app, you'll get a `409 Conflict` response. In that case you should <span class="get">GET</span> the resource again, implement your changes on the updated data, and send the <span class="put">PUT</span> request again.

### Query Parameters

Field | Description | Required?
--------- | ------- | -----------
admin_id | User ID of the admin updating the Bulletin Board | Y
content | HTML of the Bulletin Board content. Some HTML is allowed. | Y