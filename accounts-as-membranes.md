---
id: accounts-as-membranes
title: Accounts as membranes
sidebar_label: Accounts as membranes
---

Managing accounts is a part of almost every software project. In this guide, we'll learn common patterns for managing accounts represented as membranes. We'll use the very same patterns that are being used by Capability Services.

At first glance, these patterns will resemble any other ways of managing accounts. However, in the [Delegation](#delegation) section, we'll highlight the ability to delegate permissions to others without having to create accounts for them.

**Contents**:
* [Create a new account](#create-a-new-account)
* [Grant account its capabilities](#grant-account-its-capabilities)
* [Delegation](#delegation)
* [Revoke the account](#revoke-the-account)

## Create a new account

Lets assume that you want to create a new account for Antoine, who's email address is `antoine@example.com`. One way to represent an account is to create a membrane that will represent that account in our system.

### A note on unique identifiers

To create a membrane we'll need a unique identifier. A common mistake would be to use Antoine's email for this purpose. The problem with using something like an email address is that we are giving the control over unique identifiers to someone outside of the system that we control.

For example, imagine you have a company as a customer, with thousands of people, each with a unique email like `antoine@example.io`. Now, the company changes it's email domain to `example.com`. In this scenario, everything inside your system that referenced things by `example.io` needs to be migrated to `example.com`. You may not have designed for this, you may not have the tools to do the migration, and it may create a significant amount of unplanned work to change how a user refers to themselves.

A design that avoids this problem, while still providing us the benefit of unique identifiers, is to generate a unique identifier internally within our system and never expose it outside the system.

For our example, we will randomly generate a unique identifier: `8a56db8f-d735-444b-bfd8-b71301f3b708` and then maintain a database where we know that `antoine@example.com` is an email for the account identified by `8a56db8f-d735-444b-bfd8-b71301f3b708`.

_For the rest of this guide, we'll assume that the database mapping email accounts to unique identifiers exists somewhere. We won't mention it further._

### Let's do it

Now that we have our unique identifier for Antoine, let's create a membrane for Antoine's account. Our membrane configuration could look like:

```json
{
    "id": "8a56db8f-d735-444b-bfd8-b71301f3b708"
}
```

If all we'll ever use membranes for is accounts, the above membrane configuration would be sufficient. However, in case we'll use membranes to represent other things in the future, let's keep our account membranes separate from other membrane types. We'll add a namespace to the id, and we'll call it `accounts`. We'll use `/` to seperate the account namespace from account ids. Our membrane configuration now looks like:

```json
{
    "id": "accounts/8a56db8f-d735-444b-bfd8-b71301f3b708"
}
```

Ok, let's create the membrane.

_NOTE: You'll need your Membrane Service `create` capability for this part. When used in examples below, replace `$CREATE_URI` with your actual `create` capability (the full URI starting with `cpblty://...`) and `$CREATE_TOKEN` with the part of your `create` capability that starts with `CPBLTY1-...`._

### Example create request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane create --id accounts/8a56db8f-d735-444b-bfd8-b71301f3b708 --capability $CREATE_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CREATE_TOKEN" \
    -d '{"id":"accounts/8a56db8f-d735-444b-bfd8-b71301f3b708"}'
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
        id: "accounts/8a56db8f-d735-444b-bfd8-b71301f3b708"
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
        id: "accounts/8a56db8f-d735-444b-bfd8-b71301f3b708"
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
    "id": "accounts/8a56db8f-d735-444b-bfd8-b71301f3b708",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-72tmHiZTXdK3gGsf1albhJw1Tq-uTREXXY7N89gL6p2e6Z4oCJYYVPKX8bF6hF653WK85GnqzuSSVA-i8DM8Ug",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-48LzJ4lS5oVcMwY9ZCXzsZjuy9V7NH1ks5kjsC1bKzimlc02H4NjU1WdPST-iejhmFYtu2cUO9OKjGMSCizYhg"
    }
}
```

Great! We created a membrane to represent Antoine's account. Now, lets grant Antoine some capabilities.

## Grant account its capabilities

Now that we have a membrane representing Antoine's account we can grant it capabilities. For this example, our service will maintain an inbox of messages for Antoine, that can be accessed in JSON format from at this URI:
```plaintext
https://gist.githubusercontent.com/tristanls/6703ffd0fd3755cbed0b/raw/8b4db63e9ea276b0d86b7417246c9c2ce456317f/managing-accounts-as-membranes-guide-example-20160205.json
```

We want Antoine to be able to access their inbox, and we want to associate Antoine's access with their account. To do that, we will export the configuration for accessing the inbox through Antoine's account membrane.

_NOTE: You'll need your membrane `export` capability for this part. The `export` capability was returned in [the response](#example-create-response) when you created the membrane. When used in examples below, replace `$EXPORT_URI` with your actual `export` capability and `$EXPORT_TOKEN` with the part of your `export` capability that starts with `CPBLTY-1...`._

### Example export request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export \
    --uri "https://gist.githubusercontent.com/tristanls/6703ffd0fd3755cbed0b/raw/8b4db63e9ea276b0d86b7417246c9c2ce456317f/managing-accounts-as-membranes-guide-example-20160205.json" \
    --capability $EXPORT_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $EXPORT_TOKEN" \
    -d '{"uri":"https://gist.githubusercontent.com/tristanls/6703ffd0fd3755cbed0b/raw/8b4db63e9ea276b0d86b7417246c9c2ce456317f/managing-accounts-as-membranes-guide-example-20160205.json"}'
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
        uri: "https://gist.githubusercontent.com/tristanls/6703ffd0fd3755cbed0b/raw/8b4db63e9ea276b0d86b7417246c9c2ce456317f/managing-accounts-as-membranes-guide-example-20160205.json"
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
        uri: "https://gist.githubusercontent.com/tristanls/6703ffd0fd3755cbed0b/raw/8b4db63e9ea276b0d86b7417246c9c2ce456317f/managing-accounts-as-membranes-guide-example-20160205.json"
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
    "capability": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-EQRxOx5xY_I0nZwKkk0O_XH_nYbDV6ShaTSwufSRPoxrtReC2-EMp1gyJMqutbLkwpLNgRrE-O0Xws_Icvq59Q"
}
```

We now have a capability to view Antoine's inbox that is associated with Antoine's account. We can now give this capability to Antoine, who can now use our service and retrieve messages at any point by using the capability.

_NOTE: You'll need your exported `capability` for this part. The `capability` was returned in [the response](#example-export-response) when you exported it through the membrane. When used in examples below, replace `$CAPABILITY_URI` with your actual `capability` and `$CAPABILITY_TOKEN` with the part of your `capability` that starts with `CPBLTY1-...`._

### Example view inbox request

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

### Example view inbox response

```json
{
    "messages":
    [
        {
            "id": 2,
            "subject": "Follow up",
            "body": "Hey, did you see this? http://25.media.tumblr.com/tumblr_m322b2jQHF1qfsqhmo1_400.gif"
        },
        {
            "id": 1,
            "subject": "Important message for you",
            "body": "You can see this because you have the capability for it."
        }
    ]
}
```

We have created an account for Antoine and represented it as a membrane. We then gave Antoine's account the capability to view Antoine's inbox by exporting the "view Antoine's inbox" capability through Antoine's account membrane.

## Delegation

What we've demonstrated so far, isn't much different from typical account management, so why use membranes when we have other solutions available to us?

**Membranes allow the principals (also known as "users", "account holders", etc.) to delegate authority to others without having to coordinate with the issuer of the authority.** That's a pretty dense statement, so let's illustrate it by example.

Say, you own a car, you have a friend named Ayana, and you want to lend your car to them. The way this is typically done, is:

1. You give Ayana the key.
1. Ayana drives your car.

It is such a natural behavior for us to delegate authority that I find it difficult to think of it as a "thing" that happens. In step 1, we delegated our authority to drive the car by giving Ayana the key. Because Ayana now has the key, they have the authority to drive the car. If Ayana gets in a car wreck, you resolve any conflict you have with Ayana, because you know that you delegated the authority to them and you hold them responsible.

Here is how you would not lend a car to Ayana:

1. You call the "driver's license issuing" government agency.
2. Tell the agency that Ayana is now authorized to drive your car.
3. Ayana creates an account with the "driver's license issuing" government agency.
4. Ayana is recognized as authorized to drive your car.
5. Ayana drives your car.

Membranes allow the principals to delegate authority the first way. Let's demonstrate.

Antoine is in possession of `$CAPABILITY_URI` that we created for them. If Antoine now wants to grant Ayana the ability to read their inbox, they can give Ayana the `$CAPABILITY_URI`. However, the problem with this approach is that once Antoine wants to revoke Ayana's access, the only means to do that is to revoke Antoine's own `$CAPABILITY_URI`. This is inconvenient. Membranes allow Antoine to delegate `$CAPABILITY_URI` and revoke it, without affecting Antoine's own capability.

Antoine creates a new membrane representing their delegation of "view Antoine's inbox" capability to Ayana. Let's call this membrane "Temporary access for Ayana".

_NOTE: You'll need your Membrane Service `create` capability for this part. When used in examples below, replace `$CREATE_URI` with your actual `create` capability (the full URI starting with `cpblty://...`) and `$CREATE_TOKEN` with the part of your `create` capability that starts with `CPBLTY1-...`._

### Example create another membrane request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane create --id "Temporary access for Ayana" --capability $CREATE_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $CREATE_TOKEN" \
    -d '{"id":"Temporary access for Ayana"}'
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
        id: "Temporary access for Ayana"
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
        id: "Temporary access for Ayana"
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

### Example create another membrane response

```json
{
    "id": "Temporary access for Ayana",
    "capabilities":
    {
        "export": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-AXk_4nTUj5f_gyE8_lS-QLLcZTqmF4ox2NU-nODqcaLoHo8Y2ilfbtUxWCP6TN-NIs1042P7xvc01PqgWrptXw",
        "revoke": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-NaawGbDAQKIyWJWOMVhXzr0fMOXhNzszEhf6gkUyKXwtOB05AoSh7nPcQy84pEx7KFkb5EprMZ_DwMHNGFAQFA"
    }
}
```

Now, with this new membrane, Antoine can export their inbox viewing capability.

_NOTE: You'll need Antoine's view inbox `capability` for this part, as well as the membrane `export` capability from the membrane we just created ("Temporary access for Ayana"). We'll refer to `capability` as `$CAPABILITY_URI` and `$CAPABILITY_TOKEN` respectively, and to the `export` capability for the "Temporary access for Ayana" membrane as `$AYANA_EXPORT_URI` and `$AYANA_EXPORT_TOKEN`. As before, `$..._URI` is the full URI and `$..._TOKEN` is the part starting with `CBLTY1-...`._

### Example re-export request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane export \
    --capability-to-export $CAPABILITY_URI \
    --capability $AYANA_EXPORT_URI
```
<!--curl-->
```plaintext
$ curl -XPOST \
    https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $AYANA_EXPORT_TOKEN" \
    -d '{"capability":"$CAPABILITY_URI"}'
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
        authorization: "Bearer $AYANA_EXPORT_TOKEN"
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
        capability: "$CAPABILITY_URI"
    }
));
req.end();
```
<!--capability-sdk-js-->
```javascript
const CapabilitySDK = require("capability-sdk");
const membrane = new CapabilitySDK.Membrane();
membrane.export(
    "$AYANA_EXPORT_URI",
    {
        capability: "$CAPABILITY_URI"
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

### Example re-export response

```json
{
    "capability": "cpblty://membrane.amzn-us-east-1.capability.io/#CPBLTY1-_SoVXcWBr9A0YUIHP_YBJmvybm5nR4QdWziewHA4RVXgkXldqi7qXJ5zKEsVVqwBj9G_Fn_ahERJBoJ8xJgQ5w"
}
```

Now, we have two capabilities that can view Antoine's inbox. The first capability was exported through Antoine's membrane, and the capability we just created was exported through Ayana's membrane.

Go ahead and verify that both capabilities show you Antoine's inbox.

Now, the whole reason we created these membranes was so that we could revoke Ayana's access while keeping Antoine's original access. Let's now demonstrate this by revoking Ayana's membrane, the one we named "Temporary access for Ayana".

## Revoke the delegation membrane

To revoke Ayana's access, we revoke the temporary membrane we created for Ayana.

_NOTE: You'll need "Temporary access for Ayana" membrane `revoke` capability (from when you created the membrane) for this part. When used in examples below, replace `$AYANA_REVOKE_URI` with the actual `revoke` capability and `$AYANA_REVOKE_TOKEN` with the part of the `revoke` capability that starts with `CPBLTY1-...`._

### Example revoke delegation membrane request

<!--DOCUSAURUS_CODE_TABS-->
<!--capi-->
```plaintext
$ capi membrane revoke --capability "$AYANA_REVOKE_URI"
```
<!--curl-->
```plaintext
$ curl https://membrane.amzn-us-east-1.capability.io \
    -H "Authorization: Bearer $AYANA_REVOKE_TOKEN"
```
<!--node.js-->
```javascript
const https = require("https");
const options =
{
    hostname: "membrane.amzn-us-east-1.capability.io",
    headers:
    {
        authorization: "Bearer $AYANA_REVOKE_TOKEN"
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
membrane.revoke("$AYANA_REVOKE_URI", error =>
    {
        if (error)
        {
            console.error(error, error.stack);
        }
    }
);
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Example revoke delegation membrane response

```json
{
    "statusCode": 202,
    "message": "Accepted"
}
```

Once the asynchronouos membrane revocation is complete, you should be able to verify that Ayana's capability no longer works, while the original Antoine's capability still does. Attempting to use Ayana's capability will result in `401 Unauthorized` response.

## Revoke the account

While Antoine is able to delegate their capability to read their inbox to others, we can revoke Antoine's account completely by revoking their membrane. Whether or not you revoked Ayana's membrane in the example above, any capability that Antoine re-exports through other membranes will be revoked if Antoine's membrane is revoked. This gives us a nice security property, that while everyone has the freedom to delegate the capabilities in their posession to others, we can be sure that we revoked every capability that is a descendant of Antoine's capability by revoking Antoine's membrane.

_NOTE: You'll need Antoine's membrane `revoke` capability (from when you created the membrane) for this part. When used in examples below, replace `$REVOKE_URI` with your actual `revoke` capability and `$REVOKE_TOKEN` with the part of your `revoke` capability that starts with `CPBLTY1-...`._

### Example revoke account request

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

### Example revoke account response

```json
{
    "statusCode": 202,
    "message": "Accepted"
}
```

The revoke request is asynchronous. Additionally, once the revoke completes, all descendant capabilites (capabilities from this membrane that were re-exported through other membranes) are guaranteed to be revoked as well. This provides a great balance between giving your principals the freedom to organize and delegate their capabilities as needed by the business needs of the day, while providing the security guarantee that no matter what delegations took place, when a membrane is revoked, all capabilities descendant from those exported through that membrane are revoked as well.

## Summary

In this guide, we created an account for Antoine and enabled them to use our service by giving them the capability to read their inbox. Then, we demonstrated how Antoine can delegate their capability by re-exporting it through other membranes. Finally, we demonstrated that revoking a membrane revokes all capabilities exported through it as well as any descendant capabilities.

It may be of interest to go through this guide again, but then revoke Antoine's account membrane before revoking Ayana's temporary access membrane. You'll see that neither Antoine nor Ayana will have access to Antoine's inbox even though Ayana's membrane has not been revoked yet.
