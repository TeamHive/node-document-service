# node-document-service
A TypeScript NodeJS client that interfaces with the Maestro Document Service

# Getting Started

Run this to install the latest stable version.

```
npm i @teammaestro/node-document-service
```

Import `@teammaestro/node-document-service` and the client will be exported into your script from the latest build.

`import { DocumentService } from 'node-document-service';`

# Documentation
## Overview
### new DocumentService({ host: string, apiKey: string, apiSecret: string, apiCustomer: string, logging: boolean })
**Parameters:**

|Name|Type|Required|Description|
|---|---|---|---|
|host|string|False (_default: https://dms.meetmaestro.com_)| The domain that the Maestro Document Service is located at.|
|apiKey|string|True| The API Key you are given to communication with DMS|
|apiSecret|string|True| The API Secret you are given to communication with DMS|
|apiCustomer|string|True| The API Customer you are given to communication with DMS|
|logging|boolean|False (_default: false_)| Use this to turn on logging of all requests|


**Request Example:**
```javascript
new DocumentService({
    host: 'https://dev-dms.meetmaestro.com';
    apiKey: '432432',
    apiSecret: '432432',
    apiCustomer: '432432',
    logging: true
})
```

### .catch()
Whenever the API makes a call and runs into a catch block, you will get an error block that looks consistently like this:

|Name|Type|Description|
|---|---|---|
|error|Error|The exception that was thrown|
|status|number|The HTTP Status Code|
|message|string|The HTTP error message (some are custom mapped)|
|duration|number|How long the response took in miliseconds|

**Response Example:**
```
{
    error: Error
    status: 500,
    message: 'Internal Server Error',
    duration: 300
}
```

### getPreSignedData({ fileName: string, acl: string,  expiration: number }) [GET /api/v1/pre-sign](https://dev-dms.meetmaestro.com/docs/development/index.html#api-Signing-Pre_Sign_Url)

This endpoint is used for creating policies in order to upload content to your S3 bucket Note: You must send the payload to S3 in the order that we send them back.

You can append anything you want to the `key` property (including the file extension).

You can also update the `Content-Type` to the real mime-type.

**Parameters**

|Name|Type|Required|Description|
|---|---|---|---|
|fileName|string | False (_default: UUID_)| Set this if you want to name the file.|
|acl|string | False (_default: private_)| This is the permissions for the file. Options are `private|public`|
|expiration|number | False (_default: 1800_) | This is expiration time for the signature in seconds|

**Request Example:**
```javascript
dms.getPreSignedConfig({
    acl: 'public',
    fileName: 'test.pdf',
    expiration: 120
})
```

**Response Example:**
```
{
    "url": "https://new-media-test-bucket.s3.amazonaws.com",
    "key": "a8231d87-c327-4cfc-a225-d1567de732ce",
    "Content-Type": "binary/octet-stream",
    "AWSAccessKeyId": "AKIAIF4CQIVFLH2VGVNA",
    "acl": "public-read",
    "policy": "eyJleHBpcmF0aW9uIjoiMjAxNy0wMi0xM1QwN...",
    "signature": "mmgVVFG6swkWvm3AmWZ9FB71R8s=",
    "expiration": "2017-04-06T14:49:16.267Z"
}
```

### getSignedUrl(url: string, expiration: number) [GET /api/v1/sign](https://dev-dms.meetmaestro.com/docs/development/index.html#api-Signing-Sign_Url)
This endpoint is used for signing your S3 private content

**Parameters**

|Name|Type|Required|Description|
|---|---|---|---|
|url|string|True| The url of the private S3 content you want to view|
|expiration|number| False (_default: 1800_)| This is expiration time for the signature in seconds|

**Request Example:**
```javascript
dms.getSignedUrl('https://new-media-test-bucket.s3.amazonaws.com/test.pdf', 2000)
```
**Response Example:**
```
{
  "url": "https://bucket.s3.amazonaws.com/73aff5ee-a986-4af...",
  "expiration": "2017-04-06T14:49:16.267Z"
}
```

### register(options: DocumentServiceOptions.RegistrationData) [POST /api/v1/register] (https://dev-dms.meetmaestro.com/docs/development/index.html#api-Signing-Sign_Url)
This endpoint is used to register your content with the document service.

**Parameters**

|Name|Type|Required|Description|
|---|---|---|---|
|options.title|string|True| The title of the content|
|options.identity|string|True| The identity that DMS will use for callbacks|
|options.path|string|True| The location of the content in S3|
|options.mediaType|MediaType|True| The contents media type used for determining all the registration requirements|
|options.shouldConvert|boolean|False| If the file should be converted|
|options.shouldGenerateThumbnail|boolean|False| If a thumbnail should be generated|

**Request Example:**
```javascript
dms.register({
    title: 'Training Intro',
    identity: 'ad9991a8-ab82-4521-befe-a8f2f956ce12',
    path: 'https://new-media-test-bucket.s3.amazonaws.com/test.pdf',
    mediaType: 'PDF',
    shouldGenerateThumbnail: true
})
```
**Response Example:**
```
{
  "code": "MEDIA_PROCESSING"
}
```

### view(options: DocumentServiceOptions.RegistrationData) [POST /api/v1/view] (https://dev-dms.meetmaestro.com/docs/development/index.html#api-Signing-Sign_Url)
This endpoint is used for generating the information you need to view the content

The payload will be a little dynamic based on the content type

**Parameters**

|Name|Type|Required|Description|
|---|---|---|---|
|options.title|string|True| The title of the content|
|options.identity|string|True| The identity that DMS will use for callbacks|
|options.path|string|True| The location of the content in S3|
|options.mediaType|MediaType|True| The contents media type used for determining all the registration requirements|

**Request Example:**
```javascript
dms.view({
    title: 'Training Intro',
    identity: 'ad9991a8-ab82-4521-befe-a8f2f956ce12',
    path: 'https://new-media-test-bucket.s3.amazonaws.com/test.pdf',
    mediaType: 'PDF'
})
```

**Response Example:**
```
{
  "url": "https://bucket.s3.amazonaws.com/73aff5ee-a986-4af...",
  "downloadUrl": "https://bucket.s3.amazonaws.com/73aff5ee-a986-4af...",
  "expiration": "2017-04-06T14:49:16.267Z",
  "jwt": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2V..", (PDF only - The JWT needed for opening the pdf with PSPDFKit)
  "documentId": 3 (PDF only - The documentId needed for opening the pdf with PSPDFKit)
  "convertedContent": {
    "url": "https://bucket.s3.amazonaws.com/73aff5ee-a986-4af...",
    "downloadUrl": "https://bucket.s3.amazonaws.com/73aff5ee-a986-4af...",
    "expiration": "2017-04-06T14:49:16.267Z",
    "jwt": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2V..", (PDF only - The JWT needed for opening the pdf with PSPDFKit)
    "documentId": 3 (PDF only - The documentId needed for opening the pdf with PSPDFKit)
  }
}
```

# Contributors

[<img alt="John Pinkster" src="https://avatars1.githubusercontent.com/u/5350861?v=3&s=460" width="117">](https://github.com/jpinkster)|
:---: |
[John Pinkster](https://github.com/jpinkster)|
