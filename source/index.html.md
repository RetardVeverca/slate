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

All endpoints except `/signup`, `/login`, `/confirm_email` and `/password_reset` requires authentication token - `auth_token` to be presented in the header (Authorization: Bearer ...). `auth_token` has an expiration time and if expired, client needs to logout the user and acquire new token via the login process. On client side, authentication token can be stored without cookies (e.g. sessionStorage), but there must be no XSS vulnerabilities presented.

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
        "confirmation_url": "https://alpha.eventum.network/confirm?otk="
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
500 | otk_error | [varies, but otk was not created and there is a DB error info here]
500 | db_error | [varies]

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
400 | user_not_found | User with this email does not exist
400 | wrong_password | Password is incorrect
500 | auth_error | auth_token could not be generated
500 | db_error | [varies]
