---
id: your-first-membrane
title: Your first membrane
sidebar_label: Your first membrane
---

Let's create your first membrane! 🙌

If you haven't signed up for Capability Services then [Sign Up](sign-up.md).

Here's what we'll accomplish:
- [Create a membrane](#create-a-membrane)
- [Export a capability through the membrane](#export-a-capability)
- [Use the capability](#use-the-capability)
- [Revoke the membrane](#revoke-the-membrane)

## Create a membrane

A membrane enables you to group capabilities together in any way that is useful to you. Let's assume that we have a file we want to restrict access to. For this example, [this is the file](https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json), and these are the contents:

```json
{
  "message": "hello there",
  "status": "200 OK"
}
```

Now, let's create a membrane through which we'll control access to this file.

_NOTE: You'll need your Membrane Service `create` capability for this part. When used in examples below, replace `$CREATE_URI` with your actual `create` capability (the full URI starting with `cpblty://...`) and `$CREATE_TOKEN` with the part of your `create` capability that starts with `CPBLTY1-...`._

### Example create request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane create --id my-first-membrane --capability $CREATE_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CREATE_TOKEN" \
    -d '{"id":"my-first-membrane"}'
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
        id: "my-first-membrane"
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
        id: "my-first-membrane"
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

### Example create response

```json
{
    "id": "my-first-membrane",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-K8SZox9Gjk9Go8qAhkJNDsgIvQhqZGe2eG8yEGjvVGc3RN_xHJQrcOoOR-99aK3rqg3MiLevoxEaSe8rCSUFtQ",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-rS5iE85AE-_UJ_mrQf_wTwQTg25eX3U8HObfQiKOvpezTld-ZrxWkTQx9yoBjYFRazEmyqk7iNdIsJhBTeptfw"
    }
}
```

We now have our first membrane.

## Export a capability

Now that we have a membrane, we will export our first capability through it. As we mentioned in the [Create a membrane](#create-a-membrane) section above, we will export a capability to access [this file](https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json) with contents:

```json
{
  "message": "hello there",
  "status": "200 OK"
}
```

_NOTE: You'll need your membrane `export` capability for this part. The `export` capability was returned in [the response](#example-create-response) when you created the membrane. When used in examples below, replace `$EXPORT_URI` with your actual `export` capability and `$EXPORT_TOKEN` with the part of your `export` capability that starts with `CPBLTY-1...`._

### Example export request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export \
    --uri "https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json" --capability $EXPORT_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $EXPORT_TOKEN" \
    -d '{"uri":"https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json"}'
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
        uri: "https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json"
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
        uri: "https://gist.githubusercontent.com/tristanls/b47430985858978669b7/raw/d5c221dd2b9959c2d271369bf50e8cef6aa2b2d9/gistfile1.json"
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

### Example export response

```json
{
    "capability": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-hzH3Rb91MKMm658Dn9V7weUjU5gmg6U094EjcEht-quWVrJrPOWqpIMYA24B11yPpjLyq0vrsEgY8RT8yP-3Wg"
}
```

With the created capability, we can now retrieve our file.

## Use the capability

Now that we have created the capability above, we can give it to someone in order to grant them access to our file. Let's test it ourselves first.

_NOTE: You'll need your exported `capability` for this part. The `capability` was returned in [the response](#example-export-response) when you exported it through the membrane. When used in examples below, replace `$CAPABILITY_URI` with your actual `capability` and `$CAPABILITY_TOKEN` with the part of your `capability` that starts with `CPBLTY1-...`._

### Example request

<!--DOCUSAURUS_CODE_TABS-->
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CAPABILITY_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer $CAPABILITY_TOKEN"
    }
};
const req = https.request(options, resp =>
    {
        console.log(resp.statusCode); // 200
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
const req = CapabilitySDK.request("$CAPABILITY_URI");
req.on("response", resp =>
    {
        console.log(resp.statusCode); // 200
        resp.on("data", data => process.stdout.write(data.toString()));
        resp.on("end", () => process.stdout.write("\n"));
    }
);
req.on("error", error => console.error(error, error.stack));
req.end();
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Example response

And the response is the file we expect:

```json
{
  "message": "hello there",
  "status": "200 OK"
}
```

We now have a capability we can give to others. Now, let's revoke it.

## Revoke the membrane

In order to get rid of existing capabilities, like our capability to view the file, we must revoke the membrane through which the capability was exported. No matter how many capabilities were exported through the membrane, when the membrane is revoked, all of those capabilities are guaranteed to be revoked.

Let's revoke the membrane we created.

_NOTE: You'll need your membrane `revoke` capability (from when you created the membrane) for this part. When used in examples below, replace `$REVOKE_URI` with your actual `revoke` capability and `$REVOKE_TOKEN` with the part of your `revoke` capability that starts with `CPBLTY1-...`._

### Example revoke request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane revoke --capability "$REVOKE_URI"
```
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $REVOKE_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer $REVOKE_TOKEN"
    }
};
const req = https.request(options, resp =>
    {
        console.log(resp.statusCode); // 202
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
membrane.revoke("$REVOKE_URI", error =>
    {
        if (error)
        {
            console.error(error, error.stack);
        }
    }
);
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Example revoke response

```json
{
    "statusCode": 202,
    "message": "Accepted"
}
```

Notice that the status code of a successful response is `202`. This means that revocation of a membrane is asynchronous and will complete after the response is returned to you.

## Summary

You now executed a full membrane life cycle of creation, capability exports, and revocation. With this foundation, you are ready to start setting up delegatable authorization for your resources.
