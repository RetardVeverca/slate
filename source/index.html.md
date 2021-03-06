---
title: Eventum API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: cURL

toc_footers:
  - <a href="mailto:martin.m@thekiwifactory.com">Contact admin</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Eventum API!

Use this API to authenticate users, log them in, get their account details, show demo event information, retrieve information about consensus results etc.

## Design and authentication

> Sample request with the authentication token:

```shell
curl "https://api.eventum.network/events"
  -H "Authorization: Bearer drevutEbr*qA=_#ruYasp+bruva3!aBr"
  -X "GET"
```

All responses return JSON object in the body, successful responses return `data` object and error responses return `error` object. Both objects contain message (error message), id (message id) and code (HTTP code). HTTP codes are always returned according to the RFC 2616 (see <a href="https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html">this</a>). All 400 error codes mean the request was malformed and you should not repeat the request without modifications first. All 500 errors mean that something went wrong on the server side, so users can receive general "Something went wrong, please try again later" or similar warning. If detected please contact admin <a href="mailto:martin.m@thekiwifactory.com">here</a>.

All endpoints except `/signup`, `/login`, `/confirm_otk` and `/password_reset` requires authentication token - `auth_token` to be presented in the header (Authorization: Bearer ...). `auth_token` has an expiration time and if expired, client needs to logout the user and acquire new token via the login process. On client side, authentication token can be stored without cookies (e.g. sessionStorage), but there must be no XSS vulnerabilities presented.

Default rate limiting on all endpoints is: 30/minute and 200/hour per IP address. This is overwritten by specific values mentioned for each endpoint.

All times are defined in UNIX timestamp (UNIX epoch time measured in seconds since Jan 01 1970 (UTC))

<aside class="notice">
Watch for XSS vulnerabilities when storing authorization token into sessionStorage
</aside>

# Users

## Signup

> Sample request:

```shell
curl "https://api.eventum.network/users"
  -X "POST"
  -d '{
        "data": {
        "email": "janez@gmail.com",
        "password": "super_secret_password",
        "confirmation_url": "https://alpha.eventum.network/confirm?otk=",
        "ref": "8hv1p2hc1p2v"
        }
      }'
```

> Sample success response:

```json
{
    "data": {
        "message": "User added and email was sent",
        "code": 201,
        "id": "user_inserted"
    }
}
```

> Sample error response:

```json
{
    "error": {
        "message": "Duplicate entry 'mikelnmartin@yahoo.com' for key 'email'",
        "code": 400,
        "id": "duplicate"
    }
}
```

Signup new user to the demo app.

Rate limiting on this endpoint is 10/minute.

### HTTP Request

`POST https://api.eventum.network/users`

### Query Parameters

Query parameters must be send in a JSON format inside `data` object!

Parameter | Default | Format | Description
--------- | ------- | ------ | -----------
email | NULL | string(max=190), UNIQUE | Email which will be used for all communications and the whitelist
password | NULL | string | Password
confirmation_url | NULL | string | URL to which otk value is appended and send to the user to confirm his email
ref | NULL, OPTIONAL | string | Referral parameter

### Success response

User is added to the database, otk (one-time-token) is generated and user receives email with the confirmation link where otk is appended

code | id | message
---- | -- | -------
201 | user_inserted | User added and email was sent

### Error response

400 errors could be returned because `email` field is already used or JSON format is incorrect

code | id | message
---- | -- | -------
400 | duplicate | Duplicate entry '`email`' for key 'email'
400 | json_error | [varies, but usually the structure is wrong or there is a missing field]
500 | email_error | Email couldn't be send to the user. User deleted from DB.
500 | db_error | Unknown DB error

## Email verification

> Sample request:

```shell
curl "https://api.eventum.network/confirm_otk"
  -X "POST"
  -d '{
        "data": {
        "otk": "bJMc90E7HvdXFqml8mSkoT6s0rBhM6upCXFr76jbki"
        }
      }'
```

> Sample success response:

```json
{
    "data": {
        "message": "Email verified, auth_token returned",
        "code": 200,
        "id": "email_verified",
        "auth_token": "vJoXFg48bsRbHOuNkQ5WtrafIgJ2GeKN3Fo9jIugnC",
        "user_id": "42"
    }
}
```

> Sample error response:

```json
{
    "error": {
        "message": "One time token could not be found",
        "code": 500,
        "id": "otk_error"
    }
}
```

Confirm email address with one time token

### HTTP Request

`POST https://api.eventum.network/confirm_otk`

### Query Parameters

Query parameters must be send in a JSON format inside `data` object!

Parameter | Default | Format | Description
--------- | ------- | ------ | -----------
otk | NULL | string(fixed=42) | One time token user receives in the email link

### Success response

One time token is validated, expiration date is checked, email becomes verified (user can now login via the /login) and auth_token is returned (which
  is used for other endpoints that need verification)

code | id | message | auth_token | user_id
---- | -- | ------- | ---------- | -------
201 | email_verified | Email verified, auth_token returned | Auth token for further authentication | User's ID used for other operations, such as /users/{id}

### Error response

400 errors could be returned because otk is expired or JSON format is incorrect

code | id | message
---- | -- | -------
400 | otk_expired | One time token is expired
400 | json_error | [varies, but usually the structure is wrong or there is a missing field]
500 | auth_error | Error confirming the email and generating auth_token
500 | otk_not_found_error | One time token could not be found


## Login

> Sample request:

```shell
curl "https://api.eventum.network/login"
  -X "POST"
  -d '{
        "data": {
        "email": "janez@gmail.com",
        "password": "super_secret_password"
        }
      }'
```

> Sample success response:

```json
{
    "data": {
        "code": 200,
        "user_id": 42,
        "auth_token": "3PBTqxXE4RPDJtRIh9tJMrZmNGUKQNN6YLMvJgjZen",
        "message": "User successfuly logged in",
        "user_surname": null,
        "user_name": null,
        "id": "user_logged_in"
    }
}
```

> Sample error response:

```json
{
    "error": {
        "message": "Password is incorrect",
        "code": 400,
        "id": "wrong_password"
    }
}
```

Login user. Email needs to be verified before calling this endpoint.

Rate limiting on this endpoint is 10/minute.

### HTTP Request

`POST https://api.eventum.network/login`

### Query Parameters

Query parameters must be send in a JSON format inside `data` object!

Parameter | Default | Format | Description
--------- | ------- | ------ | -----------
email | NULL | string(max=190), UNIQUE | Email
password | NULL | string | Password

### Success response

Successful response returns auth_token, if this
token already exists (because of the signup process), then the existing auth_token is returned.

code | id | message | auth_token | user_id | user_name | user_surname
---- | -- | ------- | ---------- | ------- | --------- | ------------
201 | user_logged_in | User successfuly logged in | Auth token for further authentication | User's ID used for other operations, such as /users/{id} | Name | Surname

### Error response

400 errors could be returned because email is unverified, user does not exist, password is wrong or JSON format is incorrect

code | id | message
---- | -- | -------
400 | email_unverified | Email is not verified
400 | wrong_credentials | Wrong login credentials
500 | auth_error | auth_token could not be generated
500 | db_error | Unknown DB error




## Update user information

> Sample request:

```shell
curl "https://api.eventum.network/users/{user_id}"
  -X "PUT"
  -d '{
        "data": {
          "name": "Martin",
          "surname": "Mikeln",
          "password": "",
          "eth_address": "0xOP403834046d64AAc2F98BA9CD29A84D48DBFIOD"
        }
      }'
```

> Sample success response:

```json
{
  "data": {
      "message": "User's information was updated'",
      "code": 200,
      "id": "user_updated"
  }
}
```

> Sample error response:

```json
{
  "error": {
      "message": "Could not find name field in the JSON",
      "code": 400,
      "id": "json_error"
  }
}
```

Update user's information. <br />
All fields must be included. If the value is an empty string the field is not updated on the backend. <br />
The update is atomic in nature - it can succeed entirely or it can fail entirely, but it cannot partly succeed.

The following fields can be updated: `name`, `surname`, `password` and `eth_address`

This endpoint requires authentication!

### HTTP Request

`PUT https://api.eventum.network/users/{user_id}`

### Query Parameters

Query parameters must be send in a JSON format inside `data` object!

Parameter | Default | Format | Description
--------- | ------- | ------ | -----------
name | previous value | string(max=255) | First name
surname | previous value | string(max=255) | Last name
password | previous value | string | Password
eth_address | previous value | string(fixed=42) | Last name

### Success response

code | id | message
---- | -- | -------
200 | user_updated | User's information was updated

### Error response

400 errors could be returned because JSON format is incorrect

code | id | message
---- | -- | -------
400 | auth_expired | Authorization token is expired
400 | auth_invalid | Authorization token is not valid for this user
400 | auth_not_found | Auth_token was not found
400 | json_error | [varies, but usually the structure is wrong or there is a missing field]
500 | db_error | Unknown DB error


## Get user information

> Sample request:

```shell
curl "https://api.eventum.network/users/{user_id}"
  -X "GET"
  -H "Authorization: Bearer drevuty9r*qA=_#ruYasp+bruva3yiP"
```

> Sample success response:

```json
{
  "data": {
      "code": 200,
      "id": "user_info",
      "message": "User's info successfuly retrieved",
      "user_id": 120,
      "user_name": "Janez",
      "user_surname": "Novak",
      "user_email": "mikelnmartin@yahoo.comm",
      "user_eth_address": "0xOP403834046d64AAc2F98BA9CD29A84D48DBFIOD",
      "user_signup_timestamp": 1507041664,
      "user_email_verified": 1,
      "user_reputation": 0,
      "events": [
          {
              "user_voted": true,
              "event_id": 1,
              "reward_claimed": 1,
              "answers": [
                  {
                      "field_value": "first_value",
                      "field_id": 4
                  },
                  {
                      "field_value": "first_value",
                      "field_id": 5
                  }
              ],
              "user_reward_ETH": "523809523809523840",
              "user_reward_EVT": 52,
              "vote_timestamp": 1510672356,
              "vote_position": 3,
              "user_joined": true
          },
          {
                "event_id": 2,
                "user_reward": 0,
                "user_joined": false,
                "user_voted": false,
                "answers": []
            }
      ]
  }
}
```

> Sample error response:

```json
{
  "error": {
      "message": "Auth_token was not found",
      "code": 400,
      "id": "auth_not_found"
  }
}
```

Get user's information including events he joined and his voting results. <br />

This endpoint requires authentication!

### HTTP Request

`GET https://api.eventum.network/users/{user_id}`

### Query Parameters

There are no query parameters (do NOT send Content-Type headers to a GET request!)

### Success response

code | id | message
---- | -- | -------
200 | user_info | User's info successfuly retrieved

### Error response

400 errors could be returned because JSON format is incorrect

code | id | message
---- | -- | -------
400 | auth_expired | Authorization token is expired
400 | auth_invalid | Authorization token is not valid for this user
400 | auth_not_found | Auth_token was not found
500 | db_error | Unknown DB error

# Events

## Get events information

To get statistics for the events use "statistics" in "include" (https://api.eventum.network/events?include=statistics)
Statistics will only be included on the events that have already ended and have reached consensus

States go as follows:
<br>
-1: upcoming
<br>
0: joinable
<br>
1: pending
<br>
2: live
<br>
3: done
<br>
4: claimable


> Sample request:

```shell
curl "https://api.eventum.network/events?include=statistics"
  -X "GET"
```

> Sample success response:

```json
{
    "data": {
        "message": "Events info successfuly retrieved",
        "code": 200,
        "id": "events_info",
        "events": [
            {
                "category_image": "https://spartanoracle.com/wp-content/uploads/2016/11/testing.jpg",
                "image": "https://google.com/somethingsomething",
                "min_reputation": 0,
                "reward_claimable": 1,
                "claim_topic": "0x63e32091e4445d16e29c33a6b264577c2d86694021aa4e6f4dd590048f5792e8",
                "subject": "Testing",
                "time_to_join": 1510244446,
                "title": "9. November Test",
                "event_id": 1,
                "max_users": 5,
                "reward_ETH": "1000000000000000000",
                "state": 4,
                "join_start": 1510319429,
                "reward_EVT": 100,
                "description": "This is a test event",
                "end_flag": 1,
                "start_time": 1510244506,
                "contract_address": "0x77d85a9d992f6260734e30e4564151e609c3a2bf",
                "category_name": "Testing",
                "resource": "https://coleridgeprimary.net/wp-content/uploads/2016/02/Calculator.jpg",
                "fields": [
                    {
                        "field_id": 16,
                        "label": "What is (4+4-12+25)*2"
                    }
                ],
                "join_topic": "0x0940fd5e528ae9fa6c086f42387c791c3854e0febdf0717c2da155c9dbf1e86c",
                "end_time": 1520241993,
                "category_description": "This is a bullshit category",
                "input_methods": [
                    {
                        "field_id": 16,
                        "values": [
                            {
                                "value": "21",
                                "label": "21"
                            },
                            {
                                "value": "42",
                                "label": "42"
                            },
                            {
                                "value": "43",
                                "label": "43"
                            }
                        ],
                        "validation": [
                            "required",
                            "dropdown"
                        ],
                        "type": "dropdown"
                    }
                ]
            },
            {
                "category_image": "https://spartanoracle.com/wp-content/uploads/2016/11/testing.jpg",
                "min_reputation": 0,
                "reward_claimable": 0,
                "claim_topic": "0x63e32091e4445d16e29c33a6b264577c2d86694021aa4e6f4dd590048f5792e8",
                "subject": "Testing",
                "time_to_join": 1600000001,
                "title": "Upcoming event",
                "event_id": 2,
                "max_users": 1,
                "reward_ETH": "100000",
                "state": -1,
                "join_start": 1600000000,
                "reward_EVT": 2424,
                "description": "This is a test event",
                "end_flag": 0,
                "start_time": 1600000002,
                "contract_address": "0x77d85a9e992f6260734e30e4564151e609c3a2bf",
                "category_name": "Testing",
                "join_topic": "0x0940fd5e528ae9fa6c086f42387c791c3854e0febdf0717c2da155c9dbf1e86c",
                "end_time": 1600000003,
                "category_description": "This is a bullshit category"
            }
        ]
    }
}
```

> Sample error response:

```json
{
    "error": {
        "message": "Unknown DB error",
        "code": 500,
        "id": "db_error"
    }
}
```

### HTTP Request

`GET https://api.eventum.network/events?include=:options`


### Success response

code | id | message
---- | -- | -------
201 | events_info | Events info successfuly retrieved

### Error response

400 errors could be returned because JSON format is incorrect

code | id | message
---- | -- | -------
500 | db_error | Unknown DB error


## Vote

> Sample request:

```shell
curl "https://api.eventum.network/vote"
  -X "POST"
  -d '{
        "data": {
          "user_id": 45,
          "event_id": 1,
          "answers": [{"field_id": 4, "field_value": "first_value"}, {"field_id": 5, "field_value": "first_value"}]
        }
      }'
```

> Sample success response:

```json
{
    "data": {
        "message": "Vote cast, consensus reached",
        "code": 201,
        "id": "vote_cast",
        "answers": [
            {
                "field_value": "first_value",
                "field_id": 4
            },
            {
                "field_value": "first_value",
                "field_id": 5
            }
        ]
    }
}
```

> Sample error response:

```json
{
    "error": {
        "message": "Event already ended",
        "code": 400,
        "id": "event_end_error"
    }
}
```

Vote on an event

### HTTP Request

`POST https://api.eventum.network/vote`

### Query Parameters

Query parameters must be send in a JSON format inside `data` object!

Parameter | Default | Format | Description
--------- | ------- | ------ | -----------
user_id | NULL | int | user's ID
event_id | NULL | int | event's ID
answers | NULL | array | list answers

### Success response

User "voted" on the event, votes added to the database table "votes"

code | id | message
---- | -- | -------
201 | vote_cast | Vote cast, consensus not yet reached/consensus reached/consensus reached beforehand

### Error response

400 errors could be returned because JSON format is incorrect

code | id | message
---- | -- | -------
400 | not_joined_error | User is not joined on this event
400 | event_start_error | Event not started
400 | event_end_error | Event already ended
400 | already_voted_error | User already voted on this event
500 | event_error | Unknown event parameters error
500 | db_error | Unknown DB error

# Socket

## Status

Socket event to update "terminal widget" on a specific event (field event_id) with a specific text (field text)


> Sample success response:

```json
{
  "event": {
    "event_id": 15,
    "text": "User 1231fab4618c joined the event"
  }
}
```
