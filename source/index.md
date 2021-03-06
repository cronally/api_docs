---
title: Cronally API Reference

toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Cronally API Beta

The Cronally API supports account creation and management of scheduled jobs via a JSON REST-like interface.

A CLI is available for those who do not need direct access to the API.

<aside class="warning">
Help keep your account secure -- create a separate IAM user for use with Cronally and limit that user's privileges to publishing to a specific SNS topic.

In addition, *always* use our https:// endpoint when making requests. While we'll automatically redirect non-HTTPS requests to HTTPS, any requests that include confidential data (like an IAM user's secret access key) are vulnerable unless you begin the URL with https://.
</aside>

# Authentication

The Cronally API uses signed requests to authenticate for all actions except account creation. Please create an account before attempting to make other API calls.

Cronally expects the following HTTP headers to be included with each request:

`X-Cron-Key: [API KEY]`

`X-Cron-Date: [DATE_TIME]`

`X-Cron-Signature: [SIGNATURE]`

### Time format

`[DATE_TIME]` is the current UTC time formatted as `%Y%m%d%H%M` (24-hour clock)

Example: `200601021504`

### Signature calculation

`[SIGNATURE]` is calculated as follows:

1. Create `STRING_TO_SIGN`:

    Concatenate `URL_PATH+HTTP_METHOD+DATE_TIME`

    Example: `/account/GET200601021504`

2. Create `SIGNATURE`:

    Calculate an HMAC_256 of `STRING_TO_SIGN` using the base64-decoded `API_SECRET`:

    `hmac_256(base64_decode(API_SECRET), STRING_TO_SIGN)`

    Base64-encode the result to create the signature:

    `base64_encode(HMAC_256)`

3. Add `X-Cron-Signature` HTTP header to the request containing the encoded HMAC

<aside class="notice">
You must replace <code>[API KEY]</code>, <code>[DATE_TIME]</code>, and <code>[SIGNATURE]</code> with your API Key, current date/time, and calculated HMAC signature, respectively.
</aside>

# Account

## Create Account

```shell
curl -X POST -d email=username@example.com https://api.cronally.com/account/
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": {
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "email": "username@example.com",
    "api_key": "59ab5a9c-ce44-4446-9501-76f4f64be3c6",
    "api_secret": "h/Be+K0LvQxanvcabBL5Pjg3HL8g7CekNkG090he6rk=",
    "cronjob_count": 0
  }
}
```

This endpoint creates a Cronally account.

### HTTP Request

`POST https://api.cronally.com/account/`

### Parameters

Parameter | Required | Description
--------- | ------- | -----------
email | true | The email address to associate with the account. Must be unique.

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
Account ID | Account ID of newly created account
Email | Email address associated with newly created account
API Key | API key for newly created account
API Secret | Secret used to calculate signature for API requests
Cronjob Count | Total number of cron jobs associated with account


## Get Account Info

```shell
curl -X GET https://api.cronally.com/account/
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": {
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "email": "username@example.com",
    "api_key": "59ab5a9c-ce44-4446-9501-76f4f64be3c6",
    "api_secret": "h/Be+K0LvQxanvcabBL5Pjg3HL8g7CekNkG090he6rk=",
    "cronjob_count": 0
  }
}
```

This endpoint retrieves information for the account associated with the API key

### HTTP Request

`GET https://api.cronally.com/account/`

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
Account ID | Account ID of newly created account
Email | Email address associated with newly created account
API Key | API key for newly created account
API Secret | Secret used to calculate signature for API requests
Cronjob Count | Total number of cron jobs associated with account

# Cronjob

## Add cron job

```shell
curl -X POST \
-d "name=cronjob" \
-d "cron=0 * * * *" \ 
-d "sns_topic=arn:aws:sns:us-east-1:625900001012331:cron" \ 
-d "sns_message=hello from cronally" \ 
-d "sns_region=us-east-1" \ 
-d "sns_access_key_id=AKIAIOBAKZ88ALIMQPKBC3Q" \ 
-d "sns_secret_access_key=KdDja8..." \ 
https://api.cronally.com/cronjob/
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": {
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "cronjob_id": "37e545bd-fc2d-43e4-9b4a-e5eb4bc1f54e",
    "deleted": false,
    "name": "cronjob",
    "cron": "0 * * * *"
  }
}
```

This endpoint creates a new cron job

### HTTP Request

`POST https://api.cronally.com/cronjob/`

<aside class="warning">
This request requires the secret access key for an IAM user. *always* use our https:// endpoint when making this request to avoid exposing that user's secret key!
</aside>

### Parameters

Parameter | Required | Description
--------- | ------- | -----------
name | true | Name of the cron job to create.
cron | true | When to run the cron job in cron format (ex: `0 8 * * *`)
delivery_type | false | Set to `http` for HTTP delivery or `sns` (default) for SNS delivery
delivery_http_method | conditional | When `delivery_type` is `http`, set the HTTP method to `get` or `post`
delivery_http_url | conditional | When `delivery_type` is `http`, set the request URL
sns_topic | conditional | AWS SNS topic to associate with cron job
sns_message | conditional | Message to publish on SNS topic (if not needed, provide any string value)
sns_region | conditional | AWS region for SNS endpoint (ex: `us-east-1`)
sns_access_key_id | conditional | AWS IAM access key ID with appropriate permissions to publish on SNS topic
sns_secret_access_key | conditional | AWS IAM secret access key associated with IAM user

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
Name | Name of the cron job
Cron | Cron-formatted string
Cronjob ID | ID of the newly created cron job
Account ID | Account ID of newly created cron job
Deleted | Deleted status








## Update cron job

```shell
curl -X PATCH \
-d "name=updated cronjob name" \
-d "cron=10 5 * * *" \ 
https://api.cronally.com/cronjob/37e545bd-fc2d-43e4-9b4a-e5eb4bc1f54e
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": {
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "cronjob_id": "37e545bd-fc2d-43e4-9b4a-e5eb4bc1f54e",
    "deleted": false,
    "name": "updated cronjob name",
    "cron": "10 5 * * *"
  }
}
```

This endpoint updates an existing cron job's name or cron expression.

### HTTP Request

`PATCH https://api.cronally.com/cronjob/<cronjob_id>`

### Parameters

Parameter | Required | Description
--------- | ------- | -----------
name | false | Name of the cron job to create.
cron | false | When to run the cron job in cron format (ex: `0 8 * * *`)

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
Name | Name of the cron job
Cron | Cron-formatted string
Cronjob ID | ID of the newly created cron job
Account ID | Account ID of newly created cron job
Deleted | Deleted status










## List cron jobs

```shell
curl -X GET https://api.cronally.com/cronjobs/
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": [{
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "cronjob_id": "37e545bd-fc2d-43e4-9b4a-e5eb4bc1f54e",
    "deleted": false,
    "name": "cronjob",
    "cron": "0 * * * *"
  },
  {
    "account_id": "da1c2aa1-ca04-4faf-9e83-92eea9e58be8",
    "cronjob_id": "fc2d54e3-fc2d-43e4-9b4a-e5eb4bc1f54e",
    "deleted": false,
    "name": "cronjob 2",
    "cron": "5 * * * *"
  }]
}
```

This endpoint lists all cron jobs for an account

### HTTP Request

`GET https://api.cronally.com/cronjobs/`

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
Response | List of cron jobs associate with account

### Cron job

Key | Description
--------- | ------- 
Status | Success or error
Name | Name of the cron job
Cron | Cron-formatted string
Cronjob ID | ID of the newly created cron job
Account ID | Account ID of newly created cron job
Deleted | Deleted status

## Delete cron job

```shell
curl -X DELETE https://api.cronally.com/cronjob/<cronjob_id>/
```

> The above command returns JSON structured like this:

```json
{
  "status": "success",
  "response": {
    "account_id": "34f10c9d-ca04-4faf-9e83-92eea9e58be8",
    "cronjob_id": "37e545bd-fc2d-43e4-9b4a-e5eb4bc1f54e",
    "deleted": true,
    "name": "cronjob",
    "cron": "0 * * * *"
  }
}
```

This endpoint deletes a cron job

### HTTP Request

`DELETE https://api.cronally.com/cronjob/<cronjob_id>/`

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
