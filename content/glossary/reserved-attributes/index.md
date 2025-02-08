---
title: "Reserved Attributes"
description: ""
lead: ""
date: 2022-10-19T07:39:44-07:00
lastmod: 2022-10-19T07:39:44-07:00
draft: false
images: []
menu: 
  glossary:
    parent: "glossary"
weight: 52
toc: true
aliases: 
  - /docs/glossary/reserved-attributes/
---

Reserved attributes are asset attributes that are used by the DataTrails platform and have a specific purpose. All reserved attributes have the `arc_` prefix.

Select an attribute to see an example of it in use.

## Asset Attributes

| **Attribute**                                                                | **Meaning**  |
|------------------------------------------------------------------------------|--------------|
| [arc_description](/developers/api-reference/assets-api/)                     | brief description of Asset or Event being recorded |
| [arc_display_name](/developers/api-reference/assets-api/)                    | friendly name identifier for Assets, Events, and policies |
| [arc_display_type](/developers/api-reference/assets-api/)                    | classification of the type of Asset being traced that can be used for grouping or access control |
| arc_home_location_identity                                                   | physical location to which an Asset nominally 'belongs'. NOT related to the Asset's position in space. For that, use `arc_gis_*` (below) |
| [arc_primary_image](/platform/overview/advanced-concepts/#the-primary-image) | an image attachment that will display as the thumbnail of an Asset |

## Asset-Event Attributes

| **Attribute**                                                            | **Meaning** |
|--------------------------------------------------------------------------|-------------|
| [arc_attribute_type](/developers/api-reference/blobs-api/)                              | When set within a nested attribute, the value of `"arc_attachment"` identifies a reference to a [DataTrails Blob](/developers/api-reference/blobs-api/)<br>See `arc_blob*` attributes for more info |
| [arc_blob_hash_value](/developers/api-reference/blobs-api/)                             | When `arc_attribute_type` = `"arc_attachment"`, the value must equal the hash value within the associated `arc_blob_identity`|
| [arc_blob_identity](/developers/api-reference/blobs-api/)                               | A reference to a [Blob](/developers/api-reference/blobs-api/) |
| [arc_blob_hash_alg](/developers/api-reference/blobs-api/)                               | The algorithm of the `arc_blob_hash_value` (eg: "SHA256") |
| [arc_correlation_value](/platform/administration/compliance-policies/#creating-a-compliance-policy) | links Events together for evaluation in Compliance Policies |
| [arc_description](/developers/api-reference/asset-events-api/#event-creation)  | brief description of the Event being recorded |
| [arc_display_type](/developers/api-reference/asset-events-api/)          | classification of the type of Event being performed that can be used for grouping or access control |
| [arc_file_name](/developers/api-reference/blobs-api/)                                   | When `arc_attribute_type` = `"arc_attachment"`, the file name of the blob. |
| [arc_gis_lat](/platform/overview/advanced-concepts/#geolocation)          | tags the Event as having happened at a particular latitude. Used in the DataTrails UI for mapping |
| [arc_gis_lng](/platform/overview/advanced-concepts/#geolocation)          | tags the Event as having happened at a particular longitude. Used in the DataTrails UI for mapping |
| [arc_primary_image](/developers/api-reference/asset-events-api/#event-primary-image) | an image that displays as the thumbnail of the Event |
| timestamp_declared                                                       | a user provided value for when the _Asset Event was declared_.<br>The value is recorded and integrity protected but not validated as the time is declared outside the scope of DataTrails.<br>The timestamp_declared can be useful when corelating with `timestamp_accepted` and `timestamp_committed`|
| [document_hash_value](/overview/registering-an-event-against-a-document-profile-asset/) | |
| [document_hash_alg](/overview/registering-an-event-against-a-document-profile-asset/)   | |
| [document_version](/overview/registering-an-event-against-a-document-profile-asset/)    | |
| [document_status](/overview/registering-an-event-against-a-document-profile-asset/)     | |
