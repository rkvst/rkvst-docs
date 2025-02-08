---
title: "Blobs API"
description: "Blobs API Reference"
lead: "Blobs API Reference"
date: 2021-06-09T13:32:57+01:00
lastmod: 2021-06-09T13:32:57+01:00
draft: false
images: []
menu: 
  developers:
    parent: "api-reference"
weight: 109
toc: true
aliases: 
  - /docs/api-reference/blobs-api/
---
The Blobs API enables uploading Binary Large OBjects (BLOBs) such as documents, process artifacts and images, [attaching](/developers/api-reference/attachments-api/) them to [Assets](/developers/api-reference/assets-api/) and [Events (preview)](/developers/api-reference/events-api/).

{{< note >}}
**Note:** Blobs cannot be searched or listed as a collection using the blobs resource.
BLobs must be associated with an Asset or Event through an Attachment Attribute and can only be downloaded by users with appropriate access rights to that Attachment.
Take note of the Blob ID returned in the API response, it will be needed for use with Assets and Events.<br>
For information on Attachments and how to implement them, please refer to [the Events API Reference](../events-api/#adding-attachments).
{{< /note >}}

## Blob API Examples

- Create the [bearer_token](/developers/developer-patterns/getting-access-tokens-using-app-registrations) and store in a file in a secure local directory with 0600 permissions.

### Reference a Sample File

An attachment can be any type of file, from media files to code files.
The sample uses Fyodor (cat.jpg), but the `BLOB_FILE` can be replaced with any content you desire:

- Download a picture of Fyodor, or select another file for upload:

  ```bash
  curl https://app.datatrails.ai/archivist/v2/attachments/publicassets/208c5282-750e-4302-86f8-eb751de89005/events/4161673f-efa4-4391-bf06-347edd53024e/dae5a430-7d2e-4b88-b753-c09bdcc48c33 \
    -o cat.jpg
  ```

### Upload a Blob

- Identify the file to upload:

  ```bash
  BLOB_FILE=./cat.jpg
  ```

- Upload the blob:

  ```bash
  curl -X POST \
      -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "content_type=image/jpg" \
      -F "file=@$BLOB_FILE" \
      https://app.datatrails.ai/archivist/v1/blobs \
      | jq
  ```

  The response:

  ```json
  {
    "hash": {
      "alg": "SHA256",
      "value": "h1234567h"
    },
    "identity": "blobs/b123467-890b",
    "mime_type": "image/jpeg",
    "size": 21779,
    "tenantid": "t1234567-890t",
    "timestamp_accepted": "2025-01-27T23:45:29Z"
  }
  ```

### Retrieve a Blob

- Capture the Blob identity from the above POST.  
  Note, the `<blob-id>` combines `blobs/` and the value.  
  example:`"identity": "blobs/b123467-890b"` becomes `BLOB_ID=blobs/b123467-890b`:

  ```bash
  BLOB_ID=blobs/<blob-id>
  ```

- Retrieve a specific Blob, downloading the `cat.jpg` file:

  ```bash
  curl -H "@$HOME/.datatrails/bearer-token.txt" \
      -H "content_type=image/jpg" \
      --output "$BLOB_FILE" \
      https://app.datatrails.ai/archivist/v1/$BLOB_ID
  ```

### Finding Blobs

The Blobs API does not support a discovery or query API that lists all possible blobs.
Blobs are discovered through their usage within the DataTrails platform, such as the [Attachments API](/developers/api-reference/attachments-api/) or the [Asset Primary Image](/developers/api-reference/assets-api/#primary-image), [Asset-event Primary Image](/developers/api-reference/asset-events-api/#asset-event-primary-image), or the [Event (preview) Primary Image](/developers/api-reference/events-api/#event-primary-image).

Through the above APIs, capture the value of the `"arc_blob_identity"`, nested below a named attribute with an `"arc_attribute_type": "arc_attachment"`, then use the [Attachments API](/developers/api-reference/attachments-api/) to get metadata about the attachment.

## Blobs OpenAPI Docs

{{< openapi url="https://raw.githubusercontent.com/datatrails/datatrails-openapi/main/doc/blobsv1.swagger.json" >}}
