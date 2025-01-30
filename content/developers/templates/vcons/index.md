---
title: "DataTrails Event vCon Template"
description: "Creating DataTrails Events for vCons"
lead: "Securing vCons with DataTrails Events"
date: 2021-06-09T13:49:35+01:00
lastmod: 2021-06-09T13:49:35+01:00
draft: false
images: []
menu:
  developers:
    parent: "templates"
weight: 20
toc: true
aliases:
---
vCons safely and securely carry conversations from the network elements that create them to the applications that analyze them, enabling responsible management of the most personal of data.
Recording a cryptographic hash of a vCon on DataTrails secures the integrity and inclusion of the vCon from tampering or deletion.

## Version

Template Version `0.3.0`

## vCon Event Example

The following provides a template for how to secure a vCon on DataTrails.

The following example highlights a typical [DataTrails Asset-Event](../../api-reference/asset-events-api/), based on a vCon:

```json
{
  "operation": "Record",
  "behaviour": "RecordEvidence",
  "event_attributes": {
    "arc_display_type": "vcon_created",
    "conserver_link": "DataTrails",
    "conserver_link_name":  "datatrails_created",
    "conserver_link_version": "0.3.0",
    "payload_hash_alg": "SHA-256",
    "payload_preimage_content_type": "application/vcon+json",
    "payload": "5cdc3d525e...bfac2e948f31b61",
    "subject": "vcon://bbba043b-xxxx-xxxx-xxxx-ac3ddd0303af",
    "timestamp_declared": "2024-05-07T16:33:29.004994",
    "vcon_operation": "vcon_create",
    "vcon_draft_version": "01",
    "scitt_draft_version": "10"
  }
}
```

### DataTrails Event to SCITT Mapping

The following DataTrails Event attributes map to a SCITT Signed Statement:

| DataTrails Attribute | SCITT |
| -                    | -    |
| `arc_display_type` | |
| `conserver_link` | `metamap.conserver_link` |
| `conserver_link_name` | `metamap.conserver_link_name` |
| `conserver_link_version` | `metamap.conserver_link_version` |
| `payload` | `protected-header.payload` |
| `payload_hash_alg` | `protected-header.payload_hash_alg` |
| `payload_pre_image_content_type` | `protected-header.payload_pre_image_content_type` |
| `subject` | `protected-header.cwt-claims.subject` |
| `timestamp_declared` | `metamap.timestamp_declared`|
| `vcon_draft_version` | `metamap.vcon_draft_version` |
| `vcon_operation` | `metamap.vcon_operation` |
| `scitt_draft_version` | `metamap.scitt_draft_version` |

### arc_display_type (REQUIRED)

Default within DataTrails to categorize events.
`arc_display_type` is also the default means to [configure permissions](https://docs.datatrails.ai/platform/administration/sharing-access-inside-your-tenant/) for which type of events a client may view and/or edit.

For simplicity in configuring permissions, this property is a duplicate of the [vcon_operation](#vcon_operation) but could vary in advanced scenarios.

### conserver_link (OPTIONAL)

The link `type` as named under the conserver links folder.

While optional, this value is useful for tracing and debugging, knowing the source of the statement, long after troubleshooting or auditing may be needed.

For the [DataTrails Conserver Link](https://github.com/vcon-dev/vcon-server/tree/main/server/links/datatrails), this value would be `DataTrails`

### conserver_link_name (OPTIONAL)

vCons are processed by workflow pipelines that run multiple steps.
In the [conserver model](https://www.conserver.io/), these are called chains which run one or more links.
The `conserver_link_name` is the link as instanced and executed, different from the `conserver_link` which is the type name, that may be instanced 1 or more times.

Based on the conserver link implementation, this value is likely sourced from a [conserver configuration](https://github.com/vcon-dev/vcon-server/tree/main/server/links/datatrails#configuration):
For debugging purposes, the vCon pipeline may wish to store the name of the pipeline.
This property may prove to be redundant to the [vcon_operation](#vcon_operation), however it's proven helpful for tracing and debugging as the `vcon_operation` will likely turn into a standard set of lifetime values, while configuration will be unique to each instance.

```yaml
links:
  datatrails-created:
    module: links.datatrails
    options:
      api_url: "https://app.datatrails.ai/archivist"
      vcon_operation: "vcon_created"
      auth:
        type: "OIDC-client-credentials"
        token_endpoint: "https://app.datatrails.ai/archivist/iam/v1/appidp/token"
        client_id: "<your_client_id>"
        client_secret: "<your_client_secret>"
  datatrails_consent_revoked:
    module: links.datatrails
    options:
      api_url: "https://app.datatrails.ai/archivist"
      vcon_operation: "vcon_consent_revoked"
      auth:
        type: "OIDC-client-credentials"
        token_endpoint: "https://app.datatrails.ai/archivist/iam/v1/appidp/token"
        client_id: "<your_client_id>"
        client_secret: "<your_client_secret>"

chains:
  create_chain:
    links:
      - datatrails_created
    ingress_lists:
      - create_ingress
    egress_lists:
      - default_egress
    enabled: 1
  consent_revoked_chain:
    links:
      - datatrails_consent_revoked
    ingress_lists:
      - consent_ingress
    egress_lists:
      - default_egress
    enabled: 1
```

In the above configuration, depending on whether the `create_chain` or the `consent_revoked_chain` chain was instanced, `conserver_link_name` would equal: `datatrails_created` or `datatrails_consent_revoked`.
In both instances, the `conserver_link` would be `DataTrails`.

### conserver_link_version (OPTIONAL)

The version of the `conserver_link`.
This template applies to version `0.3.0`

### payload (REQUIRED)

The hash of the vCon as it's recorded on the SCITT Transparency Service.
Setting the `payload_hash_alg` indicates the payload is a hash of content in `payload_preimage_content_type` format, using the `payload_hash_alg` algorithm.

`payload`, `payload_hash_alg` and `payload_preimage_content_type` originate from the IETF Draft: [COSE Hash Envelope](https://datatracker.ietf.org/doc/draft-ietf-cose-hash-envelope/).

### payload_hash_alg (REQUIRED)

The hash algorithm used to hash the vCon.
Currently, this is `SHA-256`, but should be sourced by the vCon object to support agility.

### payload_preimage_content_type (REQUIRED)

The property name comes from [draft-ietf-cose-hash-envelope](https://datatracker.ietf.org/doc/draft-steele-cose-hash-envelope/), representing the `content-type` of the vCon, prior to hashing.

[Section 5.3.1 of vCon 01](https://www.ietf.org/archive/id/draft-ietf-vcon-vcon-container-01.html#section-5.3.1) specifies `application/vcon`.
There is [vcon issue](https://github.com/ietf-wg-vcon/draft-ietf-vcon-vcon-container/issues/7), and discussion for using `application/vcon+json`

### subject (REQUIRED)

The [vCon unique identifier (uuid)](https://www.ietf.org/archive/id/draft-ietf-vcon-vcon-container-01.html#section-4.1.2).

### subject (REQUIRED)

The [vCon unique identifier (uuid)](https://www.ietf.org/archive/id/draft-ietf-vcon-vcon-container-01.html#section-4.1.2).
Subject is used to align with the [SCITT Protected Header](https://www.ietf.org/archive/id/draft-ietf-scitt-architecture-08.html#:~:text=Subject:)

### timestamp_declared (REQUIRED)

Set to [vCon updated_at](https://www.ietf.org/archive/id/draft-ietf-vcon-vcon-container-01.html#section-4.1.4), capturing the datetime the vCon was updated.
As vCon processing may take time, and the processing of various steps (Conserver links & chains), may create out of order entries to the ledger, capturing the updated time creates consistency across a set of independent operations.
All entries related to the same vCon version (`updated_at` | `hash`), should likely be considered equal in timing.

### vcon_operation (REQUIRED)

A DataTrails Event should be created for each completed vCon operation.
For every creation and update to a vCon, a SCITT Statement would seal the vCon, recording it on the ledger for inclusion and verification.
The defined lifecycle events of a vCon will likely evolve with the standard.
For now, the `vcon_operation` (`string`) is the placeholder.

### vcon_draft_version (REQUIRED)

IETF vCon Draft version, providing interoperable stability within a draft version.
This document is aligned with [draft version 01](https://datatracker.ietf.org/doc/draft-ietf-vcon-vcon-container/history/)

### scitt_draft_version (REQUIRED)

IETF SCITT Draft version, providing interoperable stability within a draft version.
This document is aligned with [draft version 01](https://datatracker.ietf.org/doc/draft-ietf-scitt-architecture/history/)

## Verifying vCons

DataTrails provides several APIs for verifying the integrity and inclusion of changes to a vCons history.

We'll also explore specific vCon scenarios, such as consent and revocation validation.

### Retrieving All vCon Events

For each important operation performed on a vCon, a DataTrails Event (SCITT Signed Statement) should be recorded.

To align with SCITT semantics, the vcon_uuid is set to the DataTrails `subject` event attribute. (`event_attributes.subject`)

To query the history of DataTrails Events for a given vCon, use the following:

- For bash/curl commands, configure the `.datatrails/bearer-token.txt` using the DataTrails [Creating Access Tokens](https://docs.datatrails.ai/developers/developer-patterns/getting-access-tokens-using-app-registrations/) developer docs.
- Query the collection of DataTrails Events, using the `subject` attribute.
  Set the `VCON` env variable to the `vcon_uuid`

   ```bash
   DATATRAILS_EVENTS_URL="https://app.datatrails.ai/archivist/v2/assets/-/events"
   VCON="bbba043b-d1aa-4691-8739-ac3ddd0303af"
   curl -g -X GET -H "@$HOME/.datatrails/bearer-token.txt" \
     "$DATATRAILS_EVENTS_URL?event_attributes.subject=vcon://$VCON" \
     | jq
   ```

- Verify Inclusions of a Specific vCon Hash

   ```bash
   DATATRAILS_EVENTS_URL="https://app.datatrails.ai/archivist/v2/assets/-/events"
   VCON="bbba043b-d1aa-4691-8739-ac3ddd0303af"
   VCON_HASH="eae12ce2ae12c7b1280921236857d2dc1332babd311ae0fbcab620bdb148fd0d"
  curl -g -X GET -H "@$HOME/.datatrails/bearer-token.txt" \
     "$DATATRAILS_EVENTS_URL?event_attributes.subject=vcon://$VCON&event_attributes.payload_hash_alg=SHA-256&event_attributes.payload_hash_value=$VCON_HASH" \
     | jq
   ```

- Query Events for a Specific vCon for a Specific Operation

   ```bash
   DATATRAILS_EVENTS_URL="https://app.datatrails.ai/archivist/v2/assets/-/events"
   VCON="bbba043b-d1aa-4691-8739-ac3ddd0303af"
   VCON_OPERATION="vcon_created"
   curl -g -X GET -H "@$HOME/.datatrails/bearer-token.txt" \
     "$DATATRAILS_EVENTS_URL?event_attributes.subject=vcon://$VCON&event_attributes.vcon_operation=$VCON_OPERATION" \
     | jq
   ```

- Query All Events for a Specific Operations

   ```bash
   DATATRAILS_EVENTS_URL="https://app.datatrails.ai/archivist/v2/assets/-/events"
   VCON_OPERATION="vcon_created"
   curl -g -X GET -H "@$HOME/.datatrails/bearer-token.txt" \
     "$DATATRAILS_EVENTS_URL?event_attributes.vcon_operation=$VCON_OPERATION" \
     | jq
   ```

- Limit Events Created by a Specific DataTrails Identity

   ```bash
   DATATRAILS_EVENTS_URL="https://app.datatrails.ai/archivist/v2/assets/-/events"
   VCON="bbba043b-d1aa-4691-8739-ac3ddd0303af"
   PRINCIPAL="b5cfacfd-b918-4338-ad61-f4947477f874"
   curl -g -X GET -H "@$HOME/.datatrails/bearer-token.txt" \
     "$DATATRAILS_EVENTS_URL?event_attributes.subject=vcon://$VCON&principal_declared.issuer=https://app.datatrails.ai/appidpv1&principal_declared.subject=$PRINCIPAL" \
     | jq
   ```

### More Info:

- [DataTrails Quickstart: SCITT Statements (Preview)](../../developer-patterns/scitt-api/)
- [SCITT.io](https://scitt.io)
- [vCons and Conserver.io](https://www.conserver.io/)
