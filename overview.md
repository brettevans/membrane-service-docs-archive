---
id: overview
title: Overview
sidebar_label: Overview
---

Membrane Service solves the problem of being able to delegate authority in a decentralized manner. It does this by using capabilities and offering capability-based authorization at any scale in accordance with any policy via membranes.

Membrane Service provides all the primitives necessary for scalable capability use on the Internet. You can create a membrane and export capabilities through that membrane. Membrane Service is then ready to dispatch any capability invocations, and later, when the capabilites are no longer needed, the membrane and all included capabilities can be revoked.

<!--truncate-->

## What is a capability?

A capability, in the context of capability-based security used here, is an opaque, unique, and unguessable reference that grants the holder the authority to perform an action. The mere posession of a capability entitles the holder of the capability to use the referenced resource as specified by the rights reflected in the capability.

For example, if you posses a car key, the car key alone is a reference to the car (it designates a specific car) and it grants the authority to drive that particular car. A key property of why capabilities are useful, is that they can be delegated to others, much like you could delegate your car key to others, thus lending them your car.

### Capability format

A capability created by the Membrane Service has the form of a URI. For example:
```plaintext
cpblty://example.com/#CPBLTY1-aqp9nlT7a22dTGhks8vXMJNabKyIZ_kAES6U87Ljdg73xXiatBzgu5tImuWjFMXicgYb3Vpo0-C6mbm5_uFtAA
```
For more information, see the [Capability URI specification](https://github.com/capabilityio/capability-uri#capability-uri-scheme).

## What is a membrane?

A membrane is a way to organize many [capabilities](#what-is-a-capability) together. Membrane is what remembers what specific actions correspond to what opaque capabilities. Additionally, membranes are the unit of revocation when capabilities are no longer needed.
