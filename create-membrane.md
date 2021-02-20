---
id: create-membrane
title: Create Membrane
sidebar_label: Create
---

To create a membrane, you make a `POST` request using an available `create` capability. The request must include a JSON body that contains the unique `id` of the membrane to create.

## Request parameters

| Parameter | Description
| --------- | ----------- |
| `id`      | A unique membrane id

## Capabilities

| Capability | Description
| ---------- | ----------- |
| `create`   | Capability to create a membrane

## Response

Response is a newly created [membrane](membrane.md). If something goes wrong, an error is returned.

## Example request

_NOTE: In these examples, `$CREATE_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `create` capability and `$CREATE_TOKEN` is the fragment part of `$CREATE_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane create --id a-membrane --capability $CREATE_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CREATE_TOKEN" \
    -d '{"id":"a-membrane"}'
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    method: "POST",
    headers:
    {
        authorization: "Bearer $CREATE_TOKEN"
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
req.write(JSON.stringify(
    {
        id: "a-membrane"
    }
));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.create(
    "$CREATE_URI",
    {
        id: "a-membrane"
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

## Example response

```json
{
    "id": "a-membrane",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-JEoyZGkxQOq8u2RMPf8cwMZ9rNSUCSg0JzrWTp31E9cW5rvaOC8bkdkwpKstOBxfOHa-nV1cQ6ggog9m0tt8Jw",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-U5XhCSmAOE97XoJboKCTB500w2BPSW_1dstjsbl2z7HprewryLSp0Dze40LQOm7Nfa2whFkUDJWCjjXI4VfNTw"
    }
}
```
