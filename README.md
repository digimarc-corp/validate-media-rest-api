# Validate Media REST API

The validate-media-rest-api repository contains information and examples, for 
using the Digimarc Validate Media REST API to work with protected images on 
the Illuminate Platform.

* [Getting Started](#getting-started)
* [Output Parameters](#output-parameters)
* [Requests](#requests)
* [Examples](#examples)
* [Help](#help)


## Getting Started

An Account Administrator can use the Illuminate UI to get the account ID and 
create Validate Media API keys for the account you want to work with. The API 
keys can have read-only or writable access with the `VALIDATE_READ_ONLY` and 
`VALIDATE_EDITOR` roles, respectively.

Include the API key in the authorization header for every request. See the 
[Examples](#examples).

The base API URL is:

```
https://api.dmrc.app/rest
```

For each [request](#requests) detailed below, append the path and any 
parameters to the base URL.

### Notes

> You might have access to multiple Illuminate accounts, but each has one unique
account ID and one or more unique API keys. In this document, "your account" 
means the account with the ID and API key you used. Likewise, "another account" 
means an account with a different ID and API key. 

> URLs are temporary and expire after one hour.

## Output Parameters

To help you understand responses, the various output parameters are detailed 
below.

| Field | Type | Description |
|-------|------|-------------|
| [assetData](#assetdata) | Object | Contains information about an image |
| assetId | String | The asset ID returned by `/assets/check/file/start` or `/assets/protect/file/start` |
| projectId | String | The project ID returned by `/assets/check/file/start` or `/assets/protect/file/start` |
| status | String | The status of the protect or check operation: [ `ALLOCATED`, `AVAILABLE`, `DELETED`, `INVALID`, `VIRTUAL` ] |
| uploadSetId | String | Used internally |
| url | String | The temporary secure URL to which you're authorized to upload the image file |
| assetType | String | The type of asset: [ `DOCUMENT_ASSET`, `IMAGE_ASSET` ] |
| created | String | The date and time the image record was created |
| createdById | String | The ID of the user who created the image record |
| id | String | The internal ID of the image record |
| location | String | The internal file name of the image |
| modified | String | The date and time the image was changed |
| modifiedById | String | The ID of the user who last changed the image record |
| name | String | The original image file name |
| publicAssetUrl | String | The temporary image file location in cloud storage |
| protectedAssetUrl | String | The temporary protected image file location in cloud storage |
| thumbnailUrl | String | The temporary thumbnail image file location in cloud storage |
| errors | Object | Lists the error messages, if any, such as an invalid or missing account ID or asset ID |
| customAttributes | Object | Lists the key/value pairs that were added to the image |


### assetData 

This object holds information about an image. Not all fields are present for 
every instance of assetData.

| Field | Type | Description |
|-------|------|-------------|
| assetFileError | String | File processing error, if any |
| assetTypeString | String | The type of asset: [ `DOCUMENT_ASSET`, `IMAGE_ASSET` ] |
| dateProtected | String | The protection date of the original file |
| detectionStatus | String | The detection status result: [ `DETECTED`, `NOT_DETECTED` ] |
| dwProtected | Boolean | Whether the checked image is protected with a digital watermark |
| fileSizeBytes | Integer | The size of the file in bytes |
| originalAssetId | String | The original asset ID of a protected image |
| originalFileName | String | The original file name of a protected image |
| originalProjectId | String | The original project ID of a protected image |
| pixelHeight | Integer | The height of the image in pixels |
| pixelWidth | Integer | The width of the image in pixels |
| protectedAsset | String | The protected image name as stored |
| protectionStatus | String | Protection status result: [ `ALREADY_PROTECTED`, `PROTECTION_APPLIED` ] |


## Requests

This section describes how to use the available endpoints. To protect an image 
or add custom attributes, your API key must have create, read, and update 
permissions (`VALIDATE_EDITOR`).

* [Protect an Image](#protect-an-image)
  * [Upload an Image to Protect](#upload-an-image-to-protect)
    * [Start Protection](#protect-file-start)
    * [Finalize](#end)
  * [Protect an Image from a URL](#protect-an-image-from-a-url)
* [Check an Image](#check-an-image)
  * [Upload an Image to Check](#upload-an-image-to-check)
    * [Start Checking](#check-file-start)
    * [Finalize](#end)
  * [Check an Image from a URL](#check-an-image-from-a-url)
* [Get Protection Status](#get-protection-status)
* [Download the Protected Image](#download-the-protected-image)
* [Add Custom Attributes](#add-custom-attributes)

### Protect an Image

You protect an image by adding a digital watermark that contains information 
about the copyright holder. You can [upload an image](#upload-an-image-to-protect) 
from your computer or [provide a URL](#protect-an-image-from-a-url) from which 
to fetch the image.

Validate Media can protect images that meet the following criteria:

* Image format is JPG, PNG, or WebP
* The maximum file size is 25 MB
* The minimum image size is 256x256 pixels
* The maximum image size is 5000x5000 pixels

### Upload an Image to Protect

To protect an image that you upload:

1. Specify the file to protect using the [/protect/file/start](#protect-file-start) endpoint. 
1. Upload the image to the signed URL using the method you prefer. Be sure that:
    * you set the appropriate Content-Type in the header for the image you're uploading
    * you get a 200 HTTP response code before you continue
1. Send the asset ID to the [/end](#end) endpoint. 
1. [Get the protection status](#get-protection-status).
1. [Download the Protected Image](#download-the-protected-image).

#### Protect File Start

Allocates resources for the image you want to protect. Provide the image file 
name and your Illuminate account ID.

```
POST /assets/protect/file/start
```

**Example**
```
https://api.dmrc.app/rest/assets/protect/file/start?input={"accountId": "{{AccountId}}", "fileName": "{{File_Name}}"}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID and file name of the image to protect |


**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account  |
| `fileName` | String | The file name of the image to protect including the file name extension |
| `watermarkSettings` | Object | The optional watermark settings to use |

**Watermark Settings**

If you provide watermarkSettings, both fields are required.

| Name | Type | Description |
|------|------|-------------|
| `protectionMode` | String | Choose `CHROMA_COMMERCIAL` to adjust colors (for colorful images) or `LUMA_COMMERCIAL` to adjust brightness (for low-color images). The default is `CHROMA_COMMERCIAL` |
| `strength` | Integer | The strength of the watermark to use: [1-100]. The default is 90 |


**Sample Output**

The output includes the `assetId` and `projectId` that you send to the 
`/assets/end` endpoint. The `url` is the signed URL where you upload the 
file. For more information, see [Output Parameters](#output-parameters).

```json
{
    "assetId": "14d31e13-e270-499f-1234-3b8c92810585",
    "projectId": "9003c622-44f2-45fb-5678-9f0da3bcfd4f",
    "status": "ALLOCATED",
    "uploadSetId": "78326019-123a-4567-a12b-15978d997916",
    "url": "https://storage.googleapis.com/illuminate-complete-url"
}
```

#### End

Finalizes the resource allocation for the image asset you uploaded. Use the 
`assetId` from the `/protect/file/start` output.

> This endpoint is used for both protecting and checking images.

After running `/end`, you can [Get the Protection Status](#get-protection-status) 
and [download](#download-the-protected-image) the image.


```
POST /assets/end
```

**Example**
```
https://api.dmrc.app/rest/assets/end?input={"accountId": "{{AccountID}}", "assetId": "{{AssetID}}"}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID and asset ID for the uploaded image |


**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account  |
| `assetId` | String | The asset ID of the image you uploaded |

**Sample Output**

The status property tells you what stage of the protection operation the image 
is in. For more information, see [Output Parameters](#output-parameters).

```json
{
    "assetData": {
        "assetTypeString": "IMAGE_ASSET",
        "dateProtected": "Thu Jul 18 2024 13:12:13 GMT+0000 (Coordinated Universal Time)"
    },
    "assetType": "IMAGE_ASSET",
    "created": "2024-07-18T13:12:13.742Z",
    "createdById": "8d430ea1-1a23-456b-cdef-6b04ffb30cd3",
    "id": "14d31e13-e270-499f-1234-3b8c92810585",
    "location": "14d31e13-e270-499f-1234-3b8c92810585.webp",
    "modified": "2024-07-18T13:12:13.755Z",
    "modifiedById": "8d430ea1-1a23-xXXx-cdef6b04ffb30cd3",
    "name": "imagesample.webp",
    "publicAssetUrl": "https://storage.googleapis.com/illuminate-complete-url",
    "status": "ALLOCATED"
}
```

### Protect an Image from a URL

Creates an asset for the image you want to protect. The API needs the file 
name, URL where it can be found, and the watermark settings to use.

To protect an image from a URL:

1. Specify the file to protect using the [/url](#provide-the-url) endpoint. You receive an 
assetId and a signed URL.
1. [Check the protection status](#get-protection-status) for the image.
1. [Download the Protected Image](#download-the-protected-image).

```
POST /assets/protect/url
```

**Example**
```
https://api.dmrc.app/rest/assets/protect/url?input={"accountId": "{{AccountId}}", "fileName": "{{File_Name}}", "sourceURL": "{{Image_Location_URL}}"} 
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID, file name of the image, and its Internet location |

**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The Illuminate account ID |
| `fileName` | String | The file name of the image to protect including the file name extension |
| `sourceURL` | String | The URL where the source image is located |
| `watermarkSettings` | Object | The settings you want to use to apply the watermark. If not provided, default values are used |

**Watermark Settings**

| Name | Type | Description |
|------|------|-------------|
| `protectionMode` | String | The mode you want to use for protecting the image: [ `CHROMA_COMMERCIAL`, `LUMA_COMMERCIAL` ] |
| `strength` | integer | The strength of the watermark to use: [1-100]. The default is 90 |


**Sample Output**

For information about the fields, see [Output Parameters](#output-parameters).

```json
{
    "assetId": "f3182686-1d9e-4d42-baca-3000ddf01f8a",
    "projectId": "db2de294-33ea-4bdb-989f-13225daf3e5f",
    "status": "ALLOCATED",
    "uploadSetId": "72cafa61-5ba9-491c-a1a0-291e080970d2"
}
```

You can now [Get the Protection Status](#get-protection-status) and [download](#download-the-protected-image) the image.

### Get Protection Status

You can call this endpoint after protecting an image or checking an image for
an existing watermark. The contents of the `assetData` object vary 
depending on whether the operation was to protect an image or check an image.

* After protecting an image, inspect the value of `assetData.protectionStatus`. 
A value of `PROTECTION_APPLIED` means the image was protected.
* After checking an image, inspect the value of `assetData.dwProtected` and 
`assetData.originalProjectId`. If `assetData.dwProtected` is `true` and 
`assetData.originalProjectId` is an empty string, the image is owned by another 
account.

```
GET /assets
Authorization: ApiKey $API_KEY
```

**Example**
```
https://api.dmrc.app/rest/assets?accountId={{AccountID}}&first=10&projectId={{ProjectID}}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account |
| `first` | Integer | The number of assets matching the Project ID to return |
| `projectId` | String | The projectId returned from the first step in the protect or check operation  |
| `filter` | Object | An optional array of assets |


**Sample Output (Protect)**

When the `assetData.protectionStatus` is `PROTECTION_APPLIED`, the file is 
protected and ready to [download](#ddownload-the-protected-image). You can also
[add custom data](#add-custom-attributes).

When the `assetData.protectionStatus` is `ALREADY_PROTECTED`, the image already 
has a watermark and can't be watermarked again. 

For more information, see [Output Parameters](#output-parameters). 

```json
[
    {
        "assetData": {
            "assetFileError": "",
            "assetTypeString": "IMAGE_ASSET",
            "dateProtected": "Thu Jul 18 2024 13:12:13 GMT+0000 (Coordinated Universal Time)",
            "fileSizeBytes": 361208,
            "pixelHeight": 1680,
            "pixelWidth": 3200,
            "protectedAsset": "14d31e13-e270-499f-1234-3b8c92810585-protected.webp",
            "protectionStatus": "PROTECTION_APPLIED"
        },
        "assetType": "IMAGE_ASSET",
        "created": "2024-07-18T13:12:13.742Z",
        "createdById": "8d430ea1-1a23-xXXx-cdef6b04ffb30cd3",
        "id": "14d31e13-e270-499f-1234-3b8c92810585",
        "location": "14d31e13-e270-499f-1234-3b8c92810585.webp",
        "modified": "2024-07-18T13:13:50.575Z",
        "modifiedById": "validate-workers",
        "name": "imagesample.webp",
        "protectedAssetUrl": "https://storage.googleapis.com/illuminate-complete-protectedAssetUrl",
        "publicAssetUrl": "https://storage.googleapis.com/illuminate-complete-publicAssetUrl",
        "status": "AVAILABLE",
        "thumbnailUrl": "https://storage.googleapis.com/illuminate-complete-thumbnailUrl"
    }
]
```

**Sample Output (Check)**

When the `assetData.dwProtected` is `true` and `assetData.originalProjectId` 
contains a projectId, the image was protected by your account. You can 
[add custom data](#add-custom-attributes) to this image.

When the `assetData.dwProtected` is `true` and `assetData.originalProjectId` is 
an empty string, the image was watermarked by another account. 

When the `assetData.dwProtected` is `false` and `asset.detectionStatus` is `DETECTED`, 
the image has a watermark but isn't protected, such as images 
watermarked using another Digimarc application. This image can't be protected.

When the `asset.detectionStatus` is `NOT_DETECTED`, the image hasn't been 
protected. 

For more information, see [Output Parameters](#output-parameters).

```json
[
    {
        "assetData": {
            "assetTypeString": "IMAGE_ASSET",
            "dateProtected": "Thu Jul 18 2024 14:38:47 GMT+0000 (Coordinated Universal Time)",
            "detectionStatus": "DETECTED",
            "dwProtected": true,
            "fileSizeBytes": 4844093,
            "originalAssetId": "",
            "originalFileName": "",
            "originalProjectId": "",
            "pixelHeight": 3024,
            "pixelWidth": 4032
        },
        "assetType": "IMAGE_ASSET",
        "created": "2024-07-18T14:38:47.005Z",
        "createdById": "8d430ea1-1a23-xXXx-cdef6b04ffb30cd3",
        "id": "b8652ec8-6c54-42ba-84ea-fb0bc269c1bf",
        "location": "b8652ec8-6c54-42ba-84ea-fb0bc269c1bf.jpg",
        "modified": "2024-07-18T14:40:55.826Z",
        "modifiedById": "validate-workers",
        "name": "sampleimage_otheraccount.JPG",
        "publicAssetUrl": "https://storage.googleapis.com/illuminate-complete-publicAssetUrl",
        "status": "AVAILABLE",
        "thumbnailUrl": "https://storage.googleapis.com/illuminate-complete-thumbnailUrl"
    }
]
```

### Download the Protected Image

Gets a signed URL to a ZIP file that contains the protected images. The URL expires after one hour.

```
GET /assets/download
Authorization: ApiKey $API_KEY
```

**Example**
```
https://api.dmrc.app/rest/assets/download?accountId={{AccountID}}&ids={{AssetID}}
```


**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account |
| `ids` | `IDs` list | An array of protected image asset IDs to download |

**IDs Parameters**

| Name | Type | Description |
|------|------|-------------|
| `ids` | String | The assets to return |

**Sample Output**

The URL expires at the time indicated by the `Expires` property (in Unix time 
format). For more information, see [Output Parameters](#output-parameters).

```json
{
    "url": "https://storage.googleapis.com/illuminate-complete-url"
}
```

### Check an Image

You can have the Validate Media API scan an image for the presence of a digital 
watermark by [uploading an image](#upload-an-image-to-check) from your 
computer or [providing a URL](#check-an-image-from-url) from which it will 
fetch the image.

Validate Media can check images that meet the following criteria:

Image format is JPG, PNG, or WebP
The maximum file size is 25 MB
The minimum image size is 256x256 pixels
The maximum image size is 5000x5000 pixels

### Upload an Image to Check

To check an image that you upload:

1. Specify the file to check using the [/check/file/start](#check-file-start) endpoint. You receive
an assetId and a signed URL.
1. Upload the image to the signed URL using the method you prefer. Be sure that:
    * you set the appropriate Content-Type in the header for the image you're uploading
    * you get a 200 HTTP response code before you continue
1. Finalize the resource allocation by sending the assetId to the [/end](#end) 
endpoint. You receive a projectId.
1. [Get the protection status](#get-protection-status).

#### Check File Start

First, you allocate the resource for the image to check.

```
POST /assets/check/file/start
```

**Example**
```
https://api.dmrc.app/rest/assets/check/file/start?input={"accountId": "{{AccountID}}", "fileName": "{{File_Name}}"}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID and file name of the image to check |


**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account  |
| `fileName` | String | The file name of the image to check including the file name extension |

**Sample Output**

For information about the output fields, see [Output Parameters](#output-parameters).

```json
{
    "assetId": "14892ea8-3eb6-48a3-be19-acac926de25b",
    "projectId": "8c4dfa17-0b4e-422b-bb40-76c98000766a",
    "status": "ALLOCATED",
    "uploadSetId": "da4e70e3-6c08-4477-9fae-641fc7238850",
    "url": "https://storage.googleapis.com/illuminate-complete-url"
}
```

### Check an Image from a URL

Enables you to specify the URL of the image you want to check for the presence 
of a digital watermark. Be sure the URL includes the file name extension 
(.jpg, .png, or .webp).

To check an image from a URL:

1. Specify the file to check using the [/assets/check/url](#provide-URL-to-check) 
endpoint. You receive an asset ID and project ID.
1. [Get the image's protection status](#get-protection-status).

```
POST /assets/check/url
```

**Example**
```
https://api.dmrc.app/rest/assets/check/url?input={"accountId": "{{AccountID}}", "fileName": "{{File_Name}}", "sourceUrl": "{{Image_Location_URL}}"}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID, the file name of the image to check, and the source URL |

**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account  |
| `fileName` | String | The file name of the image to check including the file name extension |
| `sourceUrl` | String | The complete URL of the image to check |

After running, you can [Get the Protection Status](#get-protection-status).

**Sample Output**

For information about the output fields, see [Output Parameters](#output-parameters).

```json
{
    "assetId": "a0d361d4-12ab-34c5-6d78-2cd630a70e18",
    "projectId": "4c2922c2-456a-7890-841d-d1ee3e6eca83",
    "status": "ALLOCATED",
    "uploadSetId": "ea05d6c3-2345-6a7b-8880-31e3a96c9bdc"
}
```

### Add Custom Attributes

For any protected image in your account, you can add up to 12 custom attributes 
as key/value pairs. You'll need the image's asset ID. If you didn't recently 
protect it, first check the image and set `assetId` to the `originalAssetId` 
returned from [GET /assets](#get-protection-status) to add the 
custom data.

Each time you PUT /asset, you overwrite the existing attributes. This means
you can update the value of existing attributes by reusing an attribute name 
for the new value. You can also delete unwanted attributes by not including 
them in the `patch`.

> The custom attributes aren't returned by GET /assets. Use the Illuminate UI or
the Mobile API to read them.

```
PUT /asset
```

**Example**
```
https://api.dmrc.app/rest/asset?input={"accountId": "{{AccountID}}", "assetId": "{{AssetId}}", "patch": "{"customAttributes": {"firstAttribute": "firstValue", "secondAttribute": "secondValue"}}"}
```

**Query Parameters**

| Name | Type | Description |
|------|------|-------------|
| input | Object | Contains the Illuminate account ID, image's assetId, and a patch object containing the custom attributes |

**Input Parameters**

| Name | Type | Description |
|------|------|-------------|
| `accountId` | String | The ID of the Illuminate account  |
| `assetId` | String | The asset ID of the image you want to update |
| `patch` | Object | Contains a customAttributes object that lists the key/value pairs to add |

**Patch Parameters**

| Name | Type | Description |
|------|------|-------------|
| `customAttributes` | Object | Contains a comma-separated list of `key`: "value" pairs |

**Sample Output**

For information about the output fields, see [Output Parameters](#output-parameters).

```json
{
    "accoundId": "abc123de-0a1a-497b-bc4d-a564a928f30b",
    "assetId": "a0d361d4-12ab-34c5-6d78-2cd630a70e18",
    "created": "2024-07-31T12:00:00.001Z",
    "createdById": "userA",
    "customAttributes": {
        "firstAttr": "one",
        "secondAttr": "two",
        "thirdAttr": "three"
    },
    "id": "1abcdef7-a2c3-4e5b-df67-ab1c23d45678",
    "modified": "2024-07-31T12:00:00.100Z",
    "modifiedById": "1abcdef7-a2b3-4c5d-ef67-ab1c23d54321"
}
```

## Examples

The following examples will help you understand what you can do with the 
Validate Media API.

* [Protect an Uploaded Image Example](#protect-an-uploaded-image-example)
* [Check a Hosted Image Example](#check-a-hosted-image-example)
* [Add Three Custom Attributes](#add-three-custom-attributes)
* [Update and Delete a Custom Attribute](#update-and-delete-a-custom-attribute)

### Protect an Uploaded Image Example

To protect an image you upload from your computer:

**Step 1** -  Start by allocating resources for the image
```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X POST '$API_URL/rest/assets/protect/file/start?input={"accountId":"abc123de-0a1a-497b-bc4d-a564a928f30b", "fileName": "sampleimage.png"}'
```

**Step 2** - Upload the image
```json
curl --location --request PUT 'https://storage.googleapis.com/complete-signed-URL' \
--header 'Content-Type: application/octet-stream' \
--header 'Authentication: ApiKey 8d430ea1-1a23-xXXx-cdef6b04ffb30cd3_zp1aBc2d3UnfhsTVgpXGgmQMFE7iO5xW_5x8e12ABcDgCBtM' \
--data '<image uploaded>'
```

**Step 3** - Finalize the resource allocation for the image

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X POST '$API_URL/rest/assets/protect/file/end?input={"accountId":"abc123de-0a1a-497b-bc4d-a564a928f30b","assetId":"73d915dc-1abc-40e9-a558-4eebe8a0536f"}'
```

**Step 4** - Check that the image was protected
```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X GET '$API_URL/rest/assets?accountId=abc123de-0a1a-497b-bc4d-a564a928f30b&first=10&projectId=09d859b1-1a2b-3cd4-9c93-7d7a88dce560'
```

**Step 5** - Download the protected image
```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X GET '$API_URL/rest/assets/download?accountId=abc123de-0a1a-497b-bc4d-a564a928f30b&ids=73d915dc-1abc-40e9-a558-4eebe8a0536f'
```

### Check a Hosted Image Example

To check an image on the internet for the presence of a digital watermark:

**Step 1** - Provide the URL for the image to check
```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X POST '$API_URL/rest/assets/check/url?input={"accountId":"abc123de-0a1a-497b-bc4d-a564a928f30b","fileName": "sampleimage.png","sourceUrl":"http://imagelocation.com/sampleimage.png"}'
```

**Step 2** - Check the hosted image
```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -X GET '$API_URL/rest/assets?accountId=abc123de-0a1a-497b-bc4d-a564a928f30b&first=10&projectId=09d859b1-1a2b-3cd4-9c93-7d7a88dce560'
```

### Add Three Custom Attributes

To get the assetId, perform all the steps for checking the image and set the 
assetId to the value of originalAssetId returned by 
[GET /assets](#get-protection-status).

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H Content-Type:application/json \
  -X PUT '$API_URL/asset' \
  -d '{
    "accountId": "$ACCOUNT_ID",
    "assetId": "$ASSET_ID",
    "patch": {
        "customAttributes": {
            "firstAttribute": "one",
            "secondAttribute": "two",
            "thirdAttribute": "three"
        }
    }
}'
```

### Update and Delete a Custom Attribute

This example sets one of the previously added attribute keys to a new value
and removes an unwanted attribute (secondAttribute).

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H Content-Type:application/json \
  -X PUT '$API_URL/asset' \
  -d '{
    "accountId": "$ACCOUNT_ID",
    "assetId": "$ASSET_ID",
    "patch": {
        "customAttributes": {
            "firstAttribute": "won",
            "thirdAttribute": "three"
        }
    }
}'
```

## Help

You can request assistance by creating a support ticket in the Illuminate UI. 
Log into Illuminate and click the Help icon to get started.
