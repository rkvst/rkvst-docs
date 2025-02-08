---
title: "Assets API"
description: "Assets API Reference"
lead: "Assets API Reference"
date: 2021-06-09T11:39:03+01:00
lastmod: 2021-06-09T11:39:03+01:00
draft: false
images: []
menu: 
  developers:
    parent: "api-reference"
weight: 103
toc: true
aliases: 
  - /docs/api-reference/assets-api/
---
DataTrails provides two mechanisms for persisting provenance metadata:

1. [Asset based Events](/developers/api-reference/asset-events-api): where a series of Events are grounded to a specific Asset.
1. [Asset-free Events](/developers/api-reference/events-api) (preview) : where events can be correlated across pre-defined Trails.

The Asset-free Events implementation is the future focus of the DataTrails platform providing the capabilities of Asset based events, with broader flexibility, performance and scalability.
Asset-free Events are currently in preview, inviting early developer feedback.

The transition to Asset-free Events involves removing the dependency to anchoring Events in an Asset and shifting from mutable Asset Attributes to immutable Event Attributes.
To minimize the impact, prior to switching to Asset-free Events, it is recommended to use Event Attributes, rather than Asset Attributes.

{{< note >}}
**Note:** For more information on Assets and Asset creation, visit [Core Concepts](/platform/overview/core-concepts/#assets) and the [Creating an Asset](/platform/overview/creating-an-asset/) guide.
{{< /note >}}

## Asset API Examples

{{< note >}}
**Note:** If you are looking for a simple way to test DataTrails APIs you might prefer the [Postman collection](https://www.postman.com/datatrails-inc/workspace/datatrails-public/overview), the [YAML runner](/developers/yaml-reference/story-runner-components/) or the [Developers](https://app.datatrails.ai) section of the web UI.

Additional YAML examples can be found in the articles in the [Overview](/platform/overview/introduction/) section.
{{< /note >}}

### Asset Record Creation

- Create the [bearer_token](/developers/developer-patterns/getting-access-tokens-using-app-registrations) and store in a file in a secure local directory with 0600 permissions.
- Define the asset:

  ```bash
  cat > /tmp/asset.json <<EOF
  {
    "behaviours": ["RecordEvidence"],
    "attributes": {
      "arc_display_type": "Cat",
      "arc_display_name": "My Cat",
      "weight": "3.6kg"
    },
    "public": false
  }
  EOF
  ```

- Create the Asset:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "Content-type: application/json" \
      -d "@/tmp/asset.json" \
      https://app.datatrails.ai/archivist/v2/assets \
      | jq
  ```

  The response:

  ```json
  {
    "identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "behaviours": [
      "RecordEvidence",
      "AssetCreator",
      "Builtin"
    ],
    "attributes": {
      "arc_display_type": "Cat",
      "arc_display_name": "My Cat",
      "weight": "3.6kg"
    },
    "confirmation_status": "PENDING",
    "tracked": "TRACKED",
    "owner": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "at_time": "2024-09-04T23:35:13Z",
    "proof_mechanism": "MERKLE_LOG",
    "chain_id": "xxxxxxxxxx",
    "public": false,
    "tenant_identity": "tenant/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
  ```

### Primary Image

An Asset can have a primary image, displayed in the DataTrails Application.
The image must first be uploaded with the [Blobs API](/developers/api-reference/blobs-api/), with the BLOB_ID, BLOB_HASH and BLOB_FILE captured for uploading the asset.

- Define the asset parameters, with the image information from the uploaded Blob:

  ```bash
  BLOB_ID=<blob-id>
  BLOB_FILE=cat.jpg
  BLOB_HASH=<hash-value>
  ```

  Example:

  ASSET_ID=assets/a1234567-890a  
  BLOB_ID=blobs/b1234567-890b  
  BLOB_FILE=cat.jpg  
  BLOB_HASH=h1234567h  

  ```bash
  cat > /tmp/asset.json <<EOF
  {
    "behaviours": ["RecordEvidence"],
    "attributes": {
      "arc_display_type": "Cat",
      "arc_display_name": "My Cat",
      "weight": "3.6kg",
      "arc_primary_image": {
        "arc_attribute_type": "arc_attachment",
        "arc_blob_identity": "$BLOB_ID",
        "arc_blob_hash_alg": "SHA256",
        "arc_blob_hash_value": "$BLOB_HASH",
        "arc_display_name": "arc_primary_image",
        "arc_file_name": "cat.jpeg"
      }
    },
    "public": false
  }
  EOF
  ```

- Create the Asset With a Primary Image:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "Content-type: application/json" \
      -d "@/tmp/asset.json" \
      https://app.datatrails.ai/archivist/v2/assets \
      | jq
  ```

  The response:

  ```json
  {
    "identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "behaviours": [
      "RecordEvidence",
      "AssetCreator",
      "Builtin"
    ],
    "attributes": {
      "arc_display_type": "Cat",
      "arc_primary_image": {
        "arc_attribute_type": "arc_attachment",
        "arc_blob_identity": "blobs/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "arc_blob_hash_alg": "SHA256",
        "arc_blob_hash_value": "xxxxxxxxxxxx",
        "arc_display_name": "arc_primary_image",
        "arc_file_name": "cat.jpeg"
      },
      "arc_display_name": "My Cat",
      "weight": "3.6kg"
    },
    "confirmation_status": "PENDING",
    "tracked": "TRACKED",
    "owner": "yyyyyyyyyy",
    "at_time": "2025-01-28T21:58:34Z",
    "proof_mechanism": "MERKLE_LOG",
    "chain_id": "123456",
    "public": false,
    "tenant_identity": "tenant/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }
  ```

#### Creating a Public Asset

{{< warning >}}
**Warning**: Assets can only be made public at Asset creation and cannot be made private afterwards.
{{< /warning >}}

In most cases it is appropriate to create a standard Asset.
These Assets can only be shared externally using Access Policies as described in [Sharing Assets with OBAC](/platform/administration/sharing-access-outside-your-tenant/) or the [IAM Policies API Reference](../iam-policies-api/).

However, it is also possible to create a Public Asset which can be shared with a read-only public url; similar to the link sharing you may have seen in file sharing services like Google Drive or DropBox.

Public Assets can be used for Public Attestation, where you may wish to publicly assert data you have published.

For example, the vulnerability reports against an OpenSource software package, or perhaps the maintenance records for a public building.

Creating a Public Asset just requires flipping the `public` value in the above request to `true`.
From then on, **only** the creating Tenancy may update the Asset and Events on a Public Asset through their private, signed-in interface.

All Public Assets are then given a read-only public URL that can be retrieved using [Fetch a Public Asset's URL](./#fetch-a-public-assets-url), any Events added to that Public Asset will also get their own unique Public URL that can be retrieved with [Fetch a Public Asset's Event URL](./#fetch-a-public-assets-event-url).

This link can be shared with anyone to give them read-only access to the Asset or Event without the need to sign in.

To interact with the unauthenticated Public Interface for a Public Asset see the [Public Assets API Reference](../public-assets-api/).
To update the Assets and Events as the creating Tenant on a Public Asset's authenticated Private Interface, you would still use the standard Assets and Events API as normal.

#### Creating a Document Profile Asset

This class of Asset conforms to the [Document Profile Developer Pattern](/developers/developer-patterns/document-profile/), which allows you to trace the lifecycle of a document.

- Define the asset:

  ```bash
  cat > /tmp/asset.json <<EOF
  {
      "attributes": {
          "arc_description":"Test Document",
          "arc_display_type":"Marketing glossy",
          "arc_display_name":"Test Document Profile Asset",
          "arc_profile":"Document",
          "document_hash_value":"xxxxxxxxxxxxxxxxxxxx",
          "document_hash_alg":"sha256",
          "document_version":"1",
          "document_status":"Published",
          "some_custom_attribute":"anything you like"
      },
      "behaviours": [
          "Builtin",
          "RecordEvidence"
      ],
      "public": true
  }
  EOF
  ```

  {{< note >}}
  **Note**: Document Profile Assets must be set to `public` to be compatible with [Instaproof](/platform/overview/instaproof/) verification.
  {{< /note >}}

- Create the Asset:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "Content-type: application/json" \
      -d "@/tmp/asset.json" \
      https://app.datatrails.ai/archivist/v2/assets \
      | jq
  ```

  The response is:

  ```json
  {
      "identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "behaviours": [
          "Builtin",
          "RecordEvidence"
      ],
      "attributes": {
          "arc_profile": "Document",
          "document_version": "1",
          "some_custom_attribute": "anything you like",
          "document_status": "Published",
          "arc_description": "Test Document",
          "arc_display_type": "Marketing glossy",
          "document_hash_value": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
          "arc_display_name": "Test Document Profile Asset",
          "document_hash_alg": "sha256"
      },
      "confirmation_status": "STORED",
      "tracked": "TRACKED",
      "owner": "",
      "at_time": "2023-09-27T11:32:22Z",
      "chain_id": "8275868384",
      "public": false,
      "tenant_identity": ""
  }
  ```

### Asset Record Retrieval

Asset records in DataTrails are tokenized at creation time and referred to in all API calls and smart contracts throughout the system by a unique identity of the form:

```bash
assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

If you do not know the Asset’s identity you can fetch Asset records using other information you do know.

#### Fetch All Assets

- To fetch all Asset records, `GET` the Assets resource:

  ```bash
  curl -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      https://app.datatrails.ai/archivist/v2/assets?page_size=5 \
      | jq
  ```

#### Fetch Specific Asset by Identity

- If you know the unique identity of the Asset record `GET` the resource:

  ```bash
  curl -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID \
      | jq
  ```

#### Fetch Specific Asset at Given Point in Time by Identity

- If you know the unique identity of an Asset record and want to show its state at any given point in the past, simply `GET` with the following query parameter:

  ```bash
  curl -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID?at_time=2021-01-13T12:34:21Z" \
      | jq
  ```

  This will return the Asset record with the values it had on `2021-01-13T12:34:21Z`.

#### Fetch Assets by Name

- To fetch all Assets with a specific name, GET the Assets resource and filter on `arc_display_name`:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets?attributes.arc_display_name=tcl.ccj.003" \
      | jq
  ```

#### Fetch Assets by Type

- To fetch all Assets of a specific type, `GET` the Assets resource and filter on `arc_display_type`:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets?attributes.arc_display_type=Traffic%20light" \
      | jq
  ```

#### Fetch Assets by Proof Mechanism

- To fetch all Assets that use a specific Proof Mechanism, `GET` the Assets resource and filter on `proof_mechanism`:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets?proof_mechanism=MERKLE_LOG&page_size=5" \
      | jq
  ```

#### Fetch Events Ordered for SIMPLEHASHV1 Schema

- To fetch Simple Hash Events in the order needed for the [SIMPLEHASHV1 schema](https://github.com/datatrails/datatrails-simplehash-python), `GET` the Assets resource, specifying a specific Asset ID or using `assets/-/events` to fetch Events for all Assets:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets/-/events?order_by=SIMPLEHASHV1&page_size=5" \
      | jq
  ```

#### Fetch Assets by Filtering for Presence of a Field

- To fetch all Assets with a field set to any value, `GET` the Assets resource and filter on most available fields.

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets?attributes.arc_display_name=*&page_size=5" \
      | jq
  ```

  Returns all Assets which have `arc_display_name` that is not empty.

#### Fetch Assets Which are Missing a Field

- To fetch all Assets with a field which is not set to any value, `GET` the Assets resource and filter on most available fields.

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      "https://app.datatrails.ai/archivist/v2/assets?attributes.arc_display_name!=*&page_size=5" \
      | jq
  ```

  Returns all Assets which do not have `arc_display_name` or in which `arc_display_name` is empty.

#### Fetch a Public Asset's URL

- Fetch the Public URL of a Public Asset:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID:publicurl \
      | jq
  ```

  The response:

  ```json
  {
    "publicurl":"https://app.datatrails.ai/archivist/publicassets/xxxxxxxx"
  }
  ```

#### Fetch a Public Asset's Event URL

- Fetch the Public URL of an Event on a Public Asset:

  ```bash
  curl -g -X GET \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID/events/$EVENT_ID:publicurl \
      | jq
  ```

  The response:

  ```json
  {
    "publicurl":"https://app.datatrails.ai/archivist/publicassets/xxxx/events/xxxx"
  }
  ```

### Tracking and Untracking Assets

While deleting Assets is not possible, it is possible to hide them from default searches so that old or obsolete records don't crowd out the tenant estate or show up in partner tenancies when they shouldn't.

#### Untracking an Asset

Untracking is actually an Event in the Asset lifecycle, so it is necessary to know the Asset identity and POST to it directly.
Here we assume we are working with an Asset with identity `assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

- Define the Event attributes:

  ```bash
  cat > /tmp/event.json <<EOF
  {
    "operation": "StopTracking",
    "behaviour": "Builtin"
  }
  EOF
  ```

- Untrack the Asset:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "Content-type: application/json" \
      -d "@/tmp/event.json" \
      https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID/events \
      | jq
  ```

  The response:

  ```json
  {
    "identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/events/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "asset_identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "event_attributes": {},
    "asset_attributes": {},
    "operation": "StopTracking",
    "behaviour": "Builtin",
    "timestamp_declared": "2023-02-23T19:55:44Z",
    "timestamp_accepted": "2023-02-23T19:55:44Z",
    "timestamp_committed": "1970-01-01T00:00:00Z",
    "principal_declared": {
      "issuer": "idp.synsation.io/1234",
      "subject": "phil.b",
      "email": "phil.b@synsation.io"
    },
    "principal_accepted": {
      "issuer": "job.idp.server/1234",
      "subject": "bob@job"
    },
    "confirmation_status": "STORED",
    "transaction_id": "",
    "block_number": 0,
    "transaction_index": 0,
    "from": "",
    "tenant_identity": ""
  }
  ```

#### (Re-)Tracking an Asset

It is possible to reverse an untracking Event by tracking the Asset again, assuming you know the Asset identity.
Here we assume we are working with an Asset with identity `assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

- Define the Event attributes:

  ```bash
  cat > /tmp/event.json <<EOF
  {
    "operation": "StartTracking",
    "behaviour": "Builtin"
  }
  EOF
  ```

- Track the Asset:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "Content-type: application/json" \
      -d "@/tmp/event.json" \
      https://app.datatrails.ai/archivist/v2/assets/$ASSET_ID/events \
      | jq
  ```

  The response:

  ```json
  {
    "identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/events/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "asset_identity": "assets/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "event_attributes": {},
    "asset_attributes": {},
    "operation": "StartTracking",
    "behaviour": "Builtin",
    "timestamp_declared": "2023-02-23T19:55:44Z",
    "timestamp_accepted": "2023-02-23T19:55:44Z",
    "timestamp_committed": "1970-01-01T00:00:00Z",
    "principal_declared": {
      "issuer": "idp.synsation.io/1234",
      "subject": "phil.b",
      "email": "phil.b@synsation.io"
    },
    "principal_accepted": {
      "issuer": "job.idp.server/1234",
      "subject": "bob@job"
    },
    "confirmation_status": "PENDING",
    "transaction_id": "",
    "block_number": 0,
    "transaction_index": 0,
    "from": "",
    "tenant_identity": ""
  }
  ```

## Asset OpenAPI Docs

{{< openapi url="https://raw.githubusercontent.com/datatrails/datatrails-openapi/main/doc/assetsv2.swagger.json" >}}
