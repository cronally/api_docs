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

3. Add `X-Cron-Signature` HTTP header to the request containing the HMAC

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

## Add cron job

This endpoint creates a new cron job

### HTTP Request

`POST https://api.cronally.com/cronjob/`

### Parameters

Parameter | Required | Description
--------- | ------- | -----------
name | true | Name of the cron job to create.
cron | true | When to run the cron job in cron format (ex: `0 8 * * *`)
sns_topic | true | AWS SNS topic to associate with cron job
sns_message | true | Message to publish on SNS topic (if not needed, provide any string value)
sns_region | true | AWS region for SNS endpoint (ex: `us-east-1`)
sns_access_key_id | true | AWS IAM access key ID with appropriate permissions to publish on SNS topic
sns_secret_access_key | true | AWS IAM secret access key associated with IAM user

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

This endpoint deletes a cron job

### HTTP Request

`DELETE https://api.cronally.com/cronjob/<cronjob_id>/`

### HTTP Response

Key | Description
--------- | ------- 
Status | Success or error
