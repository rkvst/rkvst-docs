---
title: "Verify DataTrails Receipts"
description: "Proof of Posting Receipts for DataTrails"
lead: "Proof of Posting Receipts for DataTrails"
date: 2021-05-18T14:52:25+01:00
lastmod: 2021-05-18T14:52:25+01:00
draft: false
images: []
menu: 
  archive:
    parent: "archive"
weight: 10
---
# This article is deprecated

## What are receipts?

Having a receipt for a DataTrails Event allows you to prove that you recorded the Event on the DataTrails Blockchain, independent of DataTrails.

Receipts can be retrieved for [Simple Hash](/platform/overview/advanced-concepts/#simple-hash) Events once they have been confirmed and [anchored](/glossary/common-datatrails-terms/).

A user may get a receipt for any Event they have recorded on the system. You must be an Administrator for your Tenancy to retrieve receipts for any Event within the Tenancy, including those shared by other organizations.

Receipts for Public Events can be obtained by any authenticated API request.

{{< note >}}
**Note:** Receipts are currently an API-only feature. In order to obtain a receipt, you will need an App Registration.
{{< / note >}}

## What is in a receipt?

This page discusses receipts in the context of verifying events recorded on the DataTrails platform.

For a discussion of how DataTrails supports SCITT Transparent Statements and their corresponding SCITT Receipts see [Supply Chain Integrity, Transparency, and Trust (SCITT)](https://www.datatrails.ai/what-is-scitt-and-how-does-rkvst-help/).

Any DataTrails receipt you obtain today will remain valid proof of posting for the Event.

{{< warning >}}
**Warning:** The complete contents of the Event are present in the receipt in clear text. If the Event information is sensitive, the receipt should be regarded as sensitive material as well.
{{< / warning >}}

<br>

The `/archivist/v1/notary/claims/events` API creates a claim for a DataTrails event.

A claim is a commitment signed by you attesting to the contents of the event.

The response is receipt which is signed by DataTrails using a key specific to your DataTrails tenancy. This receipt attests to the inclusion of your statement in a verifiable log.

In this instance the verifiable log is a private ledger and the signing key is an ECDSA ethereum wallet key. The log (ledger) can not be changed without visibly and immediately breaking the record of all DataTrails tenants.

Because DataTrails exposes an api to collect the block headers from the private ledger it is possible for anyone to hold DataTrails to account.

## How do I retrieve and verify a receipt?

Once retrieved, receipts are fully verifiable offline and without calls to the DataTrails system using independent OSS tooling in a manner consistent with the standards established by the ethereum ecosystem.

For your convenience DataTrails provides a Python script that can be used to retrieve and verify a receipt. For full details, please visit our [Python documentation](https://python-scitt.datatrails.ai/index.html).

Receipts can also be retrieved offline using curl commands. To get started, make sure you have an [Access Token](/developers/developer-patterns/getting-access-tokens-using-app-registrations/), [Event ID](/platform/overview/creating-an-event-against-an-asset/), and [jq](https://github.com/stedolan/jq/wiki/Installation) installed.

First, save the identity of an event in `EVENT_IDENTITY`.

1. Get the transaction_id from the Event.

```bash
EVENT_TRANSACTION_ID=$(curl -s \
        -X GET -H "Authorization: Bearer ${TOKEN}" \
        https://app.datatrails.ai/archivist/v2/${EVENT_IDENTITY} \
        | jq -r .transaction_id)
```

{{< warning >}}
The transaction_id is available once the event has been committed to the blockchain. For assets using the Simple Hash `proof_mechanisms` it is available once the event is included in an anchor.
{{< / warning>}}

1. Get a claim for the Event identity

    ```bash
    CLAIM=$(curl -s -d "{\"transaction_id\":\"${EVENT_TRANSACTION_ID}\"}" \
            -X POST -H "Authorization: Bearer ${TOKEN}" \
            https://app.datatrails.ai/archivist/v1/notary/claims/events \
            | jq -r .claim)
    ```

1. Next, get the corresponding receipt for the claim

    ```bash
    RECEIPT=$(curl -s -d "{\"claim\":\"${CLAIM}\"}" \
            -X POST -H "Authorization: Bearer ${TOKEN}" \
            https://app.datatrails.ai/archivist/v1/notary/receipts \
            | jq -r .receipt)
    ```

1. Get the block details
    Get the block number using:

    ```bash
    echo ${RECEIPT} | base64 -d | less
    ```

    Look for the first `"block":"<HEX-BLOCK-NUMBER>"` in the decoded output and set the value in the environment, for example: `BLOCK="0x1234"`.

    Next, get the appropriate state root field from the block details. To verify a Simple Hash receipt get the
    `stateRoot`:

    ```bash
    WORLDROOT=$(curl -s -X GET -H "Authorization: Bearer ${TOKEN}" \
                https://app.datatrails.ai/archivist/v1/archivistnode/block?number="${BLOCK}" \
                | jq -r .stateRoot)
    ```

1. Finally, use the `datatrails_receipt_scittv1` command to verify the receipt offline at any time.

    ```bash
    echo ${RECEIPT} | datatrails_receipt_scittv1 verify -d --worldroot ${WORLDROOT}
    ```