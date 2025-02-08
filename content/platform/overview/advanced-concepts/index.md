---
title: "Advanced Concepts"
description: "DataTrails Advanced Concepts"
lead: "This section goes into more detail on subjects covered in Core Concepts, introducing some additional concepts."
date: 2021-06-14T10:57:58+01:00
lastmod: 2024-03-19T10:57:58+01:00
draft: false
images: []
menu: 
  platform:
    parent: "overview"
weight: 32
toc: true
aliases: 
  - /docs/overview/advanced-concepts/
---

## Events

The principal objects in the DataTrails platform are *Events*.
These are the records that represent the collective ['Golden Thread'](/platform/overview/core-concepts/#the-golden-thread) of evidence contributed by all stakeholders about a particular *thing*.

That *thing* can represent anything: a file, a piece of data, a physical thing or even a business process.
All that's needed is an identifier of the *thing* to corelate across the Events recorded about the *thing*.
As long as shared accountability needs to be traced and trustworthy, Events can be recorded about it.
If there are *moments that matter* they can be committed to the DataTrails immutable audit log.

What defines a moment that matters? It's all about the use case: if you think you might need to prove something in a multi-party dispute later, chances are you can save a lot of time and stress by committing it to the ledger.
Simply looking and knowing the current state of things isn't enough: sure, it has software version 3.0 now but when was that released? Before the major incident? After the major incident? This morning before the support call? By recording events into an immutable audit trail, questions relating to that fact can be answered.

DataTrails ensures complete and tamper-proof lineage and provenance for all Asset attributes by enforcing a simple rule:
*Events cannot be modified once published! No shredding, tampering or backdating is possible without leaving an undeniable trace in the log.*

### Event Attributes

DataTrails Events are essentially very simple: a collection of *attributes* that describe the Event.
The power of the system comes from the fact that those attributes come with complete traceable provenance and are guaranteed to appear the same to every stakeholder, creating a single source of truth for shared business processes.

DataTrails is not opinionated about Event content, meaning attributes can trace anything deemed important to participants.
Much like #hashtags on social media platforms, Event Attributes can be invented by anyone at any time, but once an attribute has been recorded, it will be fully traced from that point on.

### Asset Container

{{< note >}}
**Note:** Assets will soon be deprecated for a more flexible and powerful concept of *Trails*.

The change is subtle, but where Assets only allow Events to be registered against a single thing or theme, Trails will enable Events to be free of any such restriction leading to more natural expression of *what happened when* or *who said what about what*.

To prepare for this change, when writing code integrations be sure to focus on Event attributes and not mutable `asset_attributes`.
This will ensure best performance and minimal code changes to take advantage of the newer API.
Trails will still support simple properties like types, descriptions and thumbnails for search and grouping purposes.
{{< /note >}}

In the current platform, Events are registered into collections called *Assets*.
[Assets](/developers/api-reference/assets-api/) may represent an individual thing, a class of things, or something more abstract like 'all Events for this day'.
As Assets are retired, Events can still be correlated by Trails or Event Attributes.

Some care should be taken in designing the scheme for this as it will aid in simplifying sharing policies.

See the [Templates](/developers/templates/) section for domain specific examples.

### Deleting Assets (untracking)

An essential value of storing evidence in DataTrails is that data is always available to stakeholders and cannot be shredded or manipulated later.
Given this, it is not possible to actually delete Assets from the system, but there will be cases where it is desirable to hide Assets in the UI or omit them from default searches or compliance queries (for instance as a result of decommissioning or disposal of the corresponding physical asset).

To accommodate this need DataTrails separates the Asset estate into 2 classes: tracked Assets (those that are interesting to the system and actively recording events) and untracked Assets (those that are no longer actively interesting).
When for any reason it becomes desirable to remove an Asset, the Asset owner can make it *untracked* so that it does not appear in lists or searches.

{{< caution >}}
**Caution:** Untracking an Asset does not remove it or its Event history from the system; all stakeholders who previously had access to the record will continue to have access to the Event history, *including* the untracking event, if they look for it.
{{< /caution >}}

### Timestamps on Events

Lifecycle events in DataTrails give stakeholders a shared view of “Who did What When to an Artifact".
The “What” is quite straightforward, but the "Artifact", “When” and “Who” can be more nuanced.
The Artifact may be referenced by a Trail (preview) or an Event Attribute.

Once committed to the DataTrails system, each lifecycle Event record carries 3 separate timestamps:

* `timestamp_declared` - an optional user-supplied value that tells when an Event happened.
This is useful for cases where the client system is off-line for a period but the user still wishes to record the accurate time and order of activities (eg inspection rounds in an air-gapped facility).
If unspecified, the system sets timestamp_declared equal to timestamp_accepted (see below).
* `timestamp_accepted` - the time the event was actually received on the DataTrails REST interface.
Set by the system, cannot be changed by the client.
* `timestamp_committed` - the time the event was confirmed distributed to all nodes in the value chain.
Set by the system, cannot be changed by the client.

Having these 3 fields enables users of DataTrails to accurately reflect what is claimed, while preventing tampering and backdating of entries.

### User Principals on Events

Just as with the "When", the "Who" of “Who Did What When to an Artifact" is potentially complicated.
For example, an application or gateway may be acting on behalf of some other real-world user.

Once committed to the DataTrails system, each lifecycle Event record carries 2 separate user identities:

* `principal_declared` - an optional user-supplied value that tells who performed an Event.
This is useful for cases where the user principal/credential used to authorize the Event does not accurately or usefully reflect the real-world agent (eg a multi-user application with device-based credentials).
* `principal_accepted` - the actual user principal information belonging to the credential used to access the DataTrails REST interface.
Set by the system and retrieved from the authorizing IDP, cannot be changed by the client.

For more detailed information on Events, and how to implement them, please refer to the [Events API Reference](/developers/api-reference/events-api/).

## Access Policies

Sharing the right amount of information with your value chain partners is critical to creating a trustworthy shared lineage.
It is important that every participant be able to see and contribute to the management of Events without compromising security, commercial, or private personal information.
For example, competing vendors should not see each other’s information, but both should be able to freely collaborate with their mutual customer or industry regulator.

In other scenarios, it is desirable to share basic maintenance information with a vendor or external maintenance company, whilst restricting critical operating information such as run cycles and cyber SLAs to a much smaller group.

DataTrails Access Policies are the method through which this access is defined, allowing Asset owners to collaborate with just the right partners at the right time, sharing as much or as little access to Assets as the needs of the value chain partners dictate.
All transactions are private by default, meaning that only the Asset owner can see and update Asset histories until a sharing policy has been set up.
This ensures ready compliance with important regimes such as GDPR and antitrust regulations, as well as allowing safe and ready collaboration with a large and diverse range of value chain partners in the DataTrails network when required.

{{< note >}}
**Note:** To collaborate with a value chain partner you first need to enroll them as a partner in your DataTrails Tenancy by exchanging your public DataTrails Subject Identity with each other, a little like making a new LinkedIn connection or Facebook friend.

This one-time manual process helps to underpin trust and security in your DataTrails Access Policies by ensuring that the partners represented in them are the ones you expect.
{{< /note >}}

### Public Attestations

While a strict, 1-to-1 relationship might be desirable for some use cases, it is also possible that a recorded asset and associated events are recorded in a more widely accessible way.
With the use of the *Public* setting for an asset, you can create an access policy which enables anyone to view that asset record.
A viewer of that asset does not have to be registered with DataTrails, and can anonymously use our [Instaproof](/platform/overview/instaproof) service to check the thing they have against the public record in DataTrails.

{{< note >}}
**Note:** Instaproof uses the hash of a file, piece of data or digital artifact to check for associated records with that hash value.
This enables users to quickly check if the thing they have is the correct, unaltered version they are expecting.
{{< /note >}}

### Considerations

As with any system handling large amounts of important data, one must carefully consider the design and scope of Access Policy rules in DataTrails.
Every situation is different, and the DataTrails Access Policy system is flexible and powerful enough to support most situations, but in general it is recommended to follow some basic rules:

* Aim for fewest possible number of policies: This makes it much easier to review and manage access rights.
* Balance complex, highly-specific policies with simple, broad ones: Remember rights granted by policy are additive.
* A single Access Policy can contain several permission groups, so it is possible to define a single filter to cover a particular population of Assets, then apply different rights to different sets of users and partner organizations.
This is often a simpler way to manage access than to create separate Access Policies for each set of users.
* Remember attributes can change: ABAC policies are applied at time of access request, not at time of creation, so changing attributes on an asset may change which access policies apply to it.
This is one of the primary advantages to an ABAC system, but needs to be kept in mind when designing access control processes.

### Access Policy Configuration

DataTrails employs a principle called Attribute-Based Access Control (ABAC) for users within an organization who are given [internal access to your tenant](/platform/administration/sharing-access-inside-your-tenant/).
A related concept called Organization-Based Access Control (OBAC) is provided to mediate data sharing between value chain participants who will share [external access to their tenants](/platform/administration/sharing-access-outside-your-tenant/).

Rather than applying a specific fixed policy to each Asset, or grouping them into rigid hierarchies, Access Policies are defined in terms of the observable properties (or attributes) of Assets and users, and if both match, the policy is applied.
This enables much greater flexibility and expressivity than traditional hierarchical or role-based methods, whilst at the same time reducing complexity in defining sharing in large-scale systems.

DataTrails Access Policies comprise of 2 main parts:

* Filters: A list of attributes to match on Assets (this defines the scope of the policy)
* Access Permissions: A list of access rights to be granted to users on those matching Assets

A simple Access Policy may look like this:

```json

    {
      // identity and tenant are non-modifiable system info
      "identity": "access_policies/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "tenant/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",

      // User-friendly identifiers
      "display_name": "Sample Policy",
      "description": "An Access Policy created for DataTrails user docs"

      // Filters define which Assets (sets of Events) this Policy applies to
      "filters": [
        {
          // Any image, video, or whitepaper...
          "or": [
            "attributes.arc_display_type=Image",
            "attributes.arc_display_type=Video",
            "attributes.arc_display_type=Whitepaper"
          ]
        },
        {
          // ... which is marked either CONFIDENTIAL or SECRET
          "or": [
            "attributes.classification=CONFIDENTIAL",
            "attributes.classification=SECRET"
          ]
        }
      ],

      // Permissions define what can be done with the matching Assets
      "access_permissions": [
        {
          // Grant access to partner organizations by Subject Identity
          "subjects": [
            // The Subject Identity for a known partner eg ACME Co
            "subjects/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" 
          ],

          // Grant access to local users by attribute (eg email address)
          "user_attributes": [ 
            { "or": [ "email=bill@synsation.com" ] } 
          ],
          
          // Select which Asset attributes these users can see
          "asset_attributes_read": [
            "arc_display_name",
            "arc_display_type"
          ],

          // Select which Asset attributes these users can modify
          // Note modifying Asset attributes in this way is deprecated
          // and not recommended
          "asset_attributes_write": [],

          // Select which types of Events these users can see
          "event_arc_display_type_read": [
            "Pre-release",
            "Watermark",
            "Approve"
          ],

          // Select which types of Events these users can contribute to the Trail
          "event_arc_display_type_write": [
            "Approve"
          ],

          // Note the include_attributes field is deprecated
          "include_attributes": []
        }
      ]
    }
```

{{< note >}}
**Note:** Observe that there are 2 lists in the `filters` which concern different attributes.
The effect of this is to say that an Asset matches the filters if it matches *at least one* entry from *every list_.
Or in other words, inner lists are `OR`, while outer lists are `AND`.

For example:

```json
Filters = [
  { "or": [type=Pump, type=Valve] },
  { "or": [vendor=SynsationIndustries] },
  { "or": [location=ChicagoWest,location=ChicagoEast] }
]
```

In the above simplified example, the policy would apply to any Pump or Valve from SynsationIndustries installed in either the ChicagoWest or ChicagoEast sites, but it would not match:

* Other device types from SynsationIndustries;
* Pumps or Valves from any other vendor;
* SynsationIndustries Valves installed in a different location
{{< /note >}}

### Revoking Access

DataTrails adopts a ‘default deny’ approach so access to an Asset Record is only possible if an Access Policy explicitly allows it.

Revoking access can therefore be achieved in a number of ways, any of which may be more or less appropriate for the circumstances:

* Remove the whole Access Policy
* Change the attributes of the Asset so that it no longer matches the Access Policy (eg change location)
* Change the attributes of the user or subject so that they no longer match the Access Policy (eg change IDP group)
* Turn off the user’s login at the IDP

{{< note >}}
**Note:** As with any fair decentralized system it is not possible to 'unsee' information.
Any change in OBAC access policies *including revoking OBAC access to a value chain partner* only applies to new information contributed *after* the policy change.
This ensures continued fair access to the historic evidence base for all legitimate participants whilst also maintaining control of future operations with the Asset owner.
{{< /note >}}

## Attachments & Content Integrity Protection

Creating provenance of artifacts may involve integrity protecting content.
The content may be the subject Artifact, such as a document, an image, media file, an AI Model, or a Virtualized Conversation (vCon).
Or, the content may be supporting evidence to an artifact, such as a Bill of Materials, a security scan result, an x-ray image of a critical pipeline or an AI Model Card.

In these scenarios, the intent is to record integrity protection of the content to the DataTrails immutable audit log.
In most cases, the content is already stored in an existing content storage system, and shouldn't be duplicated or moved to another location, disrupting the existing workflow.

Integrity protecting content with DataTrails is achieved by storying cryptographic hashes of the content on the  immutable audit log.
By storing cryptographic hashes, the integrity of the content is protected without having to duplicate the content, and avoiding persisting any Personally Identifiable Information that may require removal or redaction.

In other cases where new storage is needed, DataTrails provides a [Blobs API](/developers/api-reference/blobs-api/) to upload content, which can then be associated with an event through the [Attachments API](/developers/api-reference/attachments-api/).

Whether DataTrails Blob storage is used, or existing storage is used, storing cryptographic hashes of the content is achieved by creating Event Attributes for the Hash Algorithm, Content Type and the hash of the content.
For more information, see the the DataTrails [Templates](/developers/templates/) section.

### Asset/Event Primary Image

Attachments on Assets and Events are named in the `arc_display_name` property, so that they can be searched and indexed.
Names are arbitrary and may be defined according to the needs of the application, but one name is reserved and interpreted by the DataTrails services: `arc_primary_image`.
If an Asset has an attachment attribute named `arc_primary_image`, it will be used by the DataTrails web user interface and other tools as a thumbnail to represent the Asset or Event being viewed.

{{< note >}}
**Note:** Blobs and Attachments cannot be searched or listed as a collection in their own right: they must always be associated with an Asset or Event through an Attachment Attribute and can only be downloaded by users with appropriate access rights to that Attachment.
{{< /note >}}

{{< note >}}
**Note:** While Attachments are generally expected to be unique, the same attachment can be applied to multiple assets, such as the case of a process manual PDF.
{{< /note >}}

## Geolocation

DataTrails supports *GIS coordinates* on Events, which enable recording of exactly where an event took place, and when analyzed together can show the movement of an Asset ("This was scanned in London, and later sold in Manchester"):

{{< img src="gis_tracking.png" alt="Rectangle" caption="<em>Tracking Assets with GIS coordinates</em>" class="border-0" >}}

If you're wanting to track the movement of an Asset, or record an audit trail of *where* a particular Event happens, you can add `arc_gis_lat` and `arc_gis_lng` attributes to the `event_attributes`.

For example:

```json
    {
      "asset_identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "behaviour": "RecordEvidence",
      "operation": "Record",
      "event_attributes": {
        "arc_display_type": "Sighting Report",
        "arc_description": "Observed at Fort Mason, San Francisco",
        "arc_gis_lat": "37.807338",
        "arc_gis_lng" : "-122.429286"
      }
    }
```

Once applied the GIS coordinates on Events are immutable.

## That's It

These are all the basics of DataTrails.
With this knowledge you can now [jump straight into the API](/developers/api-reference/) or try other topics on the [DataTrails Platform](/platform/overview/introduction).
