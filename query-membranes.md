---
id: query-membranes
title: Query Membranes
sidebar_label: Query
---

To query for membranes, you make a `GET` request using your account's `query` capability. The query parameters should be encoded as a query string in the path component of the URI.

You can use `query` capability to list your membranes as well as to retrieve the capabilities for a particular membrane you created previously.

## Request parameters

| Parameter | Description
| --------- | ----------- |
| `id`      | Optional id of the membrane to query.
| `lastId`  | Optional id of the last membrane from previous query, used to return more results if there are more results to retrieve.
| `limit`   | _(Default: 1)_ Optional Limit on the number of results. The number of results will be less than or equal to the limit.

## Capabilities

| Capability | Description
| ---------- | ----------- |
| `query`    | Capability to query membranes

## Response

Response is a query result. If something goes wrong, an error is returned.

## Example request

_NOTE: In these examples, `$QUERY_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `query` capability and `$QUERY_TOKEN` is the fragment part of `$QUERY_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane query --limit 2 --capability $QUERY_URI
```
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io/?limit=2 \
    -H "Authorization: Bearer $QUERY_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer $QUERY_TOKEN"
    },
    path: "/?limit=2"
};
const req = https.request(options, resp =>
    {
        console.log(resp.statusCode); // 201
        resp.on("data", data => process.stdout.write(data.toString()));
        resp.on("end", () => process.stdout.write("\n"));
    }
);
req.on("error", error => console.error(error, error.stack));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.query(
    "$QUERY_URI",
    {
        limit: 2
    },
    (error, resp) =>
    {
        if (error)
        {
            console.error(error, error.stack);
            return;
        }
        console.log(resp);
    }
);
```
<!--END_DOCUSAURUS_CODE_TABS-->

## Example response (with more results)

```json
{
    "membranes": [
        {
            "id": "my-first-membrane",
            "capabilities": {
                "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-v19zrBVcbtGvrhp99Pc8Egv_RXioH_J_20fFOZPVLAo3Ua9ZZBtIuZ9wN73UBizmXbY5oKVuOZnfpN2NiPz5cA",
                "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-lDrbF-v5odHjj-32QFQDm1fwSskKYCHT9P4aQsEsvrW5QAhzsk7aUVCdzI6kFzA55vVR12cQX352fZZ0Z4OFfA"
            }
        },
        {
            "id": "my-second-membrane",
            "capabilities": {
                "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-f-GGTWp-XnDdNSqcqE2FbCqZPKkFgre7zkhOtkjw0eNvgT8GRh5q4Uvc1GgjkQMhCpcYju8-9rGDXZ-Qh5KObw",
                "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-yMFg6M3qTMqGHsfQa8MZ-NblVmT7c7GlOCN33RNW2bZPoui0aTZ_uz2Zu8DEEA_yJtR2rCm3LazUH6ikDuoiyA"
            }
        }
    ],
    "completed": false
}
```

## Example continuation request

_NOTE: In these examples, `$QUERY_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `query` capability and `$QUERY_TOKEN` is the fragment part of `$QUERY_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane query \
    --last-id my-second-membrane \
    --limit 2 \
    --capability $QUERY_URI
```
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io/?lastId=my-second-membrane&limit=2 \
    -H "Authorization: Bearer $QUERY_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer $QUERY_TOKEN"
    },
    path: "/?limit=2&lastId=my-second-membrane"
};
const req = https.request(options, resp =>
    {
        console.log(resp.statusCode); // 201
        resp.on("data", data => process.stdout.write(data.toString()));
        resp.on("end", () => process.stdout.write("\n"));
    }
);
req.on("error", error => console.error(error, error.stack));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.export(
    "$QUERY_URI",
    {
        lastId: "my-second-membrane",
        limit: 2
    },
    (error, resp) =>
    {
        if (error)
        {
            console.error(error, error.stack);
            return;
        }
        console.log(resp);
    }
);
```
<!--END_DOCUSAURUS_CODE_TABS-->

## Example response (with no more results)

```json
{
    "membranes": [
        {
            "id": "my-third-membrane",
            "capabilities": {
                "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-ng-Z9ZN-ShiwfmpjfUJfglPK1RUsvgPMa-MoNgGgLz89imOaP_VPAjWqenFXNFMqnLOlLAzB3qaTtlps5l2LJQ",
                "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-lfkPzE8S6z-Y7StQLiv9u4TSvvDzqqy4Cc5tUrmP0jamQ16ByPCQFoiGGwOc2_IgUVX4QDWPagc68Xl_b9SJgw"
            }
        }
    ],
    "completed": true
}
```
