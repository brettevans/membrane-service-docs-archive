---
id: aws-lambda-capability
title: AWS Lambda capability
sidebar_label: AWS Lambda capability
---

Membrane Service supports creating capabilities for AWS services by signing requests using `AWS4-HMAC-SHA256` signature scheme. This means that you can use capabilities to access all AWS resources and securely delegate that access to others.

In this example, we'll demonstrate what it takes to create a capability that invokes an AWS Lambda function.

**Contents**:
* [Create IAM user](#create-iam-user)
* [Create a Lambda function](#create-a-lambda-function)
* [Create a membrane](#create-a-membrane)
* [Configure capability for AWS Lambda](#configure-capability-for-aws-lambda)
* [Cleanup](#cleanup)
* [Summary](#summary)

## Create IAM user

You will want to create a separate IAM user (perhaps more than one depending on your use case) for use with Membrane Service capabilities in order to control what permissions capabilities managed by the Membrane Service can have. **Do not use any of your existing user credentials via Membrane Service.**

Please reference the [AWS User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) for multiple ways for creating an IAM user via Console, AWS CLI, or AWS API.

Once you created an IAM user, you'll need to create an Access Key for that user. Please reference the [AWS Managing Access Keys for IAM Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).

We will use the generated `Access Key ID` and `Secret Access Key` to configure our capability for AWS Lambda, so store them for reference.

## Create a Lambda function

In order to call a Lambda function, we need to create one. Please reference the [AWS Create a Lambda Function](https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html) getting started documentation.

## Create a membrane

Next, we'll create a new membrane to manage our AWS Lambda capability.

_NOTE: You'll need your Membrane Service `create` capability for this part. When used in examples below, replace `$CREATE_URI` with your actual `create` capability (the full URI starting with `cpblty://...`) and `$CREATE_TOKEN` with the part of your `create` capability that starts with `CPBLTY1-...`._

### Example create request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane create --id my-aws-lambda-membrane --capability $CREATE_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CREATE_TOKEN" \
    -d '{"id":"my-aws-lambda-membrane"}'
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
        id: "my-aws-lambda-membrane"
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
        id: "my-aws-lambda-membrane"
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
    "id": "my-aws-lambda-membrane",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-Vf2Q5NBTFtn-57umfGU3anHy9JkLbpZQddRD13oMAE0M3nfabXD2wKg_C3WHgP56oThB7GlvZZ_IbX5avhia4Q",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-xScyMk1m7W4cXTNyoZxdrjFUtH-oC6rR4nefoPqWMoqIx0Xh8jkWYjCJ-xUPpstrpBBvyBRPDrsSUcDBA9KA0A"
    }
}
```

We will use the created membrane `export` functionality in our next step to export a capability for invoking our AWS Lambda function.

## Configure capability for AWS Lambda

We finally have all the pieces to create our capability.

What we are doing is configuring our capability to call the [AWS HTTP Lambda Invoke API](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html) and we will sign the request using the `AWS4-HMAC-SHA256` signature scheme with the credentials of the IAM user we created earlier. Here's what that looks like.

First, we need to construct a valid URI to invoke the Lambda function. As per AWS [endpoint](https://docs.aws.amazon.com/general/latest/gr/rande.html) and [API](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html) Documentation, the HTTP API URI for a Lambda function invocation has the format:
```plaintext
https://lambda.${AWS_REGION}.amazonaws.com/2015-03-31/functions/${AWS_LAMBDA_FUNCTION_NAME}/invocations
```
where `${AWS_REGION}` is the region you created your Lambda function in (for example `us-east-1`), and `{AWS_LAMBDA_FUNCTION_NAME}` is the name of the Lambda function you created.

Additionally, in the example below, we will use `${ACCESS_KEY_ID}` to stand for the access key ID of the IAM user you created and `${SECRET_ACCESS_KEY}` to stand for the secret access key corresponding to that access key ID and IAM user.

The capability configuration is as follows:

### Example export request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export \
    --uri "https://lambda.${AWS_REGION}.amazonaws.com/2015-03-31/functions/${AWS_LAMBDA_FUNCTION_NAME}/invocations" \
    --method POST \
    --allow-query false \
    --header "X-Amz-Invocation-Type: RequestResponse" \
    --header "X-Amz-Log-Type: None" \
    --aws4-hmac-sha256-aws-access-key-id ${ACCESS_KEY_ID} \
    --aws4-hmac-sha256-region ${AWS_REGION} \
    --aws4-hmac-sha256-service lambda \
    --aws4-hmac-sha256-secret-access-key ${SECRET_ACCESS_KEY} \
    --capability cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-Vf2Q5NBTFtn-57umfGU3anHy9JkLbpZQddRD13oMAE0M3nfabXD2wKg_C3WHgP56oThB7GlvZZ_IbX5avhia4Q
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer CPBLTY1-Vf2Q5NBTFtn-57umfGU3anHy9JkLbpZQddRD13oMAE0M3nfabXD2wKg_C3WHgP56oThB7GlvZZ_IbX5avhia4Q" \
    -d '{"uri":"https://lambda.${AWS_REGION}.amazonaws.com/2015-03-31/functions/${AWS_LAMBDA_FUNCTION_NAME}/invocations","method":"POST","allowQuery":false,"headers":{"X-Amz-Invocation-Type": "RequestResponse","X-Amz-Log-Type":"None"},"hmac":{"aws4-hmac-sha256":{"awsAccessKeyId":"${ACCESS_KEY_ID}","region":"${AWS_REGION}","service":"lambda","secretAccessKey":"${SECRET_ACCESS_KEY}"}}}'
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
        authorization: "Bearer CPBLTY1-Vf2Q5NBTFtn-57umfGU3anHy9JkLbpZQddRD13oMAE0M3nfabXD2wKg_C3WHgP56oThB7GlvZZ_IbX5avhia4Q"
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
        uri: "https://lambda.${AWS_REGION}.amazonaws.com/2015-03-31/functions/${AWS_LAMBDA_FUNCTION_NAME}/invocations",
        method: "POST",
        allowQuery: false,
        headers:
        {
            "X-Amz-Invocation-Type": "RequestResponse",
            "X-Amz-Log-Type": "None"
        },
        hmac:
        {
            "aws4-hmac-sha256":
            {
                awsAccessKeyId: "${ACCESS_KEY_ID}",
                region: "${AWS_REGION}",
                service: "lambda",
                secretAccessKey: "${SECRET_ACCESS_KEY}"
            }
        }
    }
));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.export(
    "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-Vf2Q5NBTFtn-57umfGU3anHy9JkLbpZQddRD13oMAE0M3nfabXD2wKg_C3WHgP56oThB7GlvZZ_IbX5avhia4Q",
    {
        uri: "https://lambda.${AWS_REGION}.amazonaws.com/2015-03-31/functions/${AWS_LAMBDA_FUNCTION_NAME}/invocations",
        method: "POST",
        allowQuery: false,
        headers:
        {
            "X-Amz-Invocation-Type": "RequestResponse",
            "X-Amz-Log-Type": "None"
        },
        hmac:
        {
            "aws4-hmac-sha256":
            {
                awsAccessKeyId: "${ACCESS_KEY_ID}",
                region: "${AWS_REGION}",
                service: "lambda",
                secretAccessKey: "${SECRET_ACCESS_KEY}"
            }
        }
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
    "capability": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-vnFy3qq382OQkvNVbErpzUhqIEmTtEL494b7BrQTnc8gbod3k0kpOVUXJiHieSqciH9tfzQgrsSGTP8W4gdSFA"
}
```

And now, with the exported capability, you can invoke your AWS Lambda function and verify that it works.

More importantly, you now have a capability that you can delegate through other membranes and whomever posesses this or a delegated capability, can invoke the AWS Lambda function.

### Example request

<!--DOCUSAURUS_CODE_TABS-->
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer CPBLTY1-vnFy3qq382OQkvNVbErpzUhqIEmTtEL494b7BrQTnc8gbod3k0kpOVUXJiHieSqciH9tfzQgrsSGTP8W4gdSFA"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer CPBLTY1-vnFy3qq382OQkvNVbErpzUhqIEmTtEL494b7BrQTnc8gbod3k0kpOVUXJiHieSqciH9tfzQgrsSGTP8W4gdSFA"
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
const req = CapabilitySDK.request("cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-vnFy3qq382OQkvNVbErpzUhqIEmTtEL494b7BrQTnc8gbod3k0kpOVUXJiHieSqciH9tfzQgrsSGTP8W4gdSFA");
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

And the response is whatever your AWS Lambda function returns.

## Cleanup

To clean up, first, revoke the `my-aws-lambda-membrane` membrane we created (all capabilities exported through the membrane are revoked when the membrane is revoked):

### Example revoke request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane revoke --capability cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-xScyMk1m7W4cXTNyoZxdrjFUtH-oC6rR4nefoPqWMoqIx0Xh8jkWYjCJ-xUPpstrpBBvyBRPDrsSUcDBA9KA0A
```
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer CPBLTY1-xScyMk1m7W4cXTNyoZxdrjFUtH-oC6rR4nefoPqWMoqIx0Xh8jkWYjCJ-xUPpstrpBBvyBRPDrsSUcDBA9KA0A"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer CPBLTY1-xScyMk1m7W4cXTNyoZxdrjFUtH-oC6rR4nefoPqWMoqIx0Xh8jkWYjCJ-xUPpstrpBBvyBRPDrsSUcDBA9KA0A"
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
membrane.revoke("cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-xScyMk1m7W4cXTNyoZxdrjFUtH-oC6rR4nefoPqWMoqIx0Xh8jkWYjCJ-xUPpstrpBBvyBRPDrsSUcDBA9KA0A", error =>
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

Delete the AWS Lambda function you created.

Delete the IAM user you created (user credentials will be automatically deleted).

## Summary

In this example, we covered how to create a capability that invokes an AWS Lambda Function. To do so, we created an IAM user for use with Membrane Service. We then created credentials for that user. Then, we created a new membrane and exported a capability configured to call the AWS HTTP API and invoke the Lambda function.

It is worth noting that you can use this pattern to invoke any AWS HTTP API endpoint that accepts `AWS4-HMAC-SHA256` signature scheme, which is most, if not all, of AWS functionality.
