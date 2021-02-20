---
id: export-capability
title: Export Capability
sidebar_label: Export
---

Creating new capabilities happens by exporting a capability configuration through a membrane. You provide the configuration for the capability as input, and you get back the new capability as output.

To export a capability through a membrane, you make a `POST` request using the membrane's `export` capability. The request should include a JSON body that contains the `uri` or `capability`, along with other optional attributes.

## Request parameters

| Parameter | Description
| --------- | ----------- |
| `capability` | _(mutually exclusive with `uri`)_ An already existing Capability URI to re-export through this particular membrane. If the membrane is revoked the original capability will not be revoked, only the capability created during this re-export and any of its descendants will be revoked.
| `uri` | _(mutually exclusive with `capability`)_ Fully qualified URI, for example `https://example.com/path/to/something`
| `allowQuery` | _(`uri` option)_ _(Default: false)_ Optionally allow requester's URI query string to be appended to the `uri` in membrane request.
| `headers` | _(`uri` option)_ Optional headers to include with the membrane request to the URI. [Hop-by-hop headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#hbh) will be ignored.
| `hmac` | _(`uri` option)_ Optional selector for which signature scheme to use to sign membrane request to URI.<br/>- `aws4-hmac-sha256`: Use `AWS4-HMAC-SHA256` signature.<br/>-- `awsAccessKeyId`: AWS Access Key Id to sign requests with.<br/>-- `region`: AWS region capability is in.<br/>-- `service`: AWS service capability is in.<br/>-- `secretAccessKey`: AWS Secret Access Key to sign requests with.<br/>- `cap1-hmac-sha512`: Use `CAP1-HMAC-SHA512` signature.<br/>-- `key`: Base64url encoded secret key bytes.<br/>-- `keyId`: Secret key id.
| `method` | _(`uri` option)_ Optional HTTP method to use in the membrane request to the URI. This overrides the method specified by the requester.
| `timeoutMs` | _(`uri` option)_ Optional timeout in milliseconds to end idle connection between membrane and URI. Will be ignored if greater than membrane's configured internal timeout.
| `tls` | _(`uri` option)_ TLS options.<br/>- `ca`: Optionally, override default trusted Certificate Authorities (CAs). Default is to trust the well-known CAs curated by Mozilla. Mozilla's CAs are completely replaced when CA is explicitly specified using this option.<br/>- `cert`: Optional certificate chain in PEM format.<br/>- `key`: Optional private key in PEM format.<br/>- `rejectUnauthorized` If not `false`, membrane request verifies responding server against the list of default trusted Certificate Authorities or supplied Certificate Authorities, if any.

## Capabilities

| Capability | Description
| ---------- | ----------- |
| `export`   | Capability to export through a membrane

## Response

Response is a newly created capability. If something goes wrong, an error is returned.

## Example request (capability)

_NOTE: In these examples, `$EXPORT_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `export` capability and `$EXPORT_TOKEN` is the fragment part of `$EXPORT_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export --capability-to-export cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-GoqpcTT_GRVSyH-qYBGur2KnhZp7Tc06uEuFDk8Ma2e_XX1tUjLgtASB_I8SkVA4VZYcTkozC6PDdNkpIfMIjA --capability $EXPORT_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $EXPORT_TOKEN" \
    -d '{"capability":"cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-GoqpcTT_GRVSyH-qYBGur2KnhZp7Tc06uEuFDk8Ma2e_XX1tUjLgtASB_I8SkVA4VZYcTkozC6PDdNkpIfMIjA"}'
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
        authorization: "Bearer $EXPORT_TOKEN"
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
        capability: "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-GoqpcTT_GRVSyH-qYBGur2KnhZp7Tc06uEuFDk8Ma2e_XX1tUjLgtASB_I8SkVA4VZYcTkozC6PDdNkpIfMIjA"
    }
));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.export(
    "$EXPORT_URI",
    {
        capability: "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-GoqpcTT_GRVSyH-qYBGur2KnhZp7Tc06uEuFDk8Ma2e_XX1tUjLgtASB_I8SkVA4VZYcTkozC6PDdNkpIfMIjA"
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

## Example request (uri)

_NOTE: In these examples, `$EXPORT_URI` is the full [Capability URI](https://github.com/capabilityio/capability-uri#capability-uri-scheme) format of the `export` capability and `$EXPORT_TOKEN` is the fragment part of `$EXPORT_URI` that begins with `"CBLTY-1"`._

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export \
    --uri https://www.example.com \
    --header "TenantID: tenant17" \
    --method DELETE \
    --capability $EXPORT_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $EXPORT_TOKEN" \
    -d '{"uri":"https://www.example.com","headers":{"TenantID":"tenant17"},"method":"DELETE"}'
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
        authorization: "Bearer $EXPORT_TOKEN"
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
        uri: "https://www.example.com",
        headers:
        {
            TenantID: "tenant17"
        },
        method: "DELETE"
    }
));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.export(
    "$EXPORT_URI",
    {
        uri: "https://www.example.com",
        headers:
        {
            TenantID: "tenant17"
        },
        method: "DELETE"
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
    "capability": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-Vs8axPY96csXT6f7Jx_YhsyxjqsDxnIzytgtfmOHDoyIm6gDf-sjWgv-9OwufgDZAeLIGfHA2X8F-bkEXMuyMA"
}
```
