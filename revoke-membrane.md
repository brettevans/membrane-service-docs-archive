---
id: revoke-membrane
title: Revoke Membrane
sidebar_label: Revoke
---

Revoking a membrane deletes all of the capabilities that were exported through the membrane. **This is the only mechanism for deleting capabilities.**

To revoke a membrane, you make a `DELETE` request using the membrane's `revoke` capability. The revoke request is asynchronous. A successful response will be `202 Accepted`, indicating that the request was received and will be completed soon.

To verify that the request has completed, you can query for the membrane that you revoked. When the query returns an empty result, the membrane has been fully revoked.

## Request parameters

None

## Capabilities

| Capability | Description
| ---------- | ----------- |
| `revoke`   | Capability to revoke the membrane

## Response

Response is a `202 Accepted`. If something goes wrong, an error is returned.

## Example request

_NOTE: In these examples, `$REVOKE_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `revoke` capability and `$REVOKE_TOKEN` is the fragment part of `$REVOKE_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane revoke --capability $REVOKE_URI
```
<!--curl-->
```plaintext
$ curl -XDELETE \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $REVOKE_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    method: "DELETE",
    headers:
    {
        authorization: "Bearer $REVOKE_TOKEN"
    }
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
membrane.revoke(
    "$REVOKE_URI",
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

## Example resopnse

```json
{
    "statusCode": 202,
    "message": "Accepted"
}
```
