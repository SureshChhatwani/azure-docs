---
# Mandatory fields. See more on aka.ms/skyeye/meta.
title: Assets in Media Services - Azure | Microsoft Docs
description: This article gives an explanation of what assets are, and how they are used by Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''

ms.service: media-services
ms.workload: 
ms.topic: article
ms.date: 12/08/2018
ms.author: juliako
ms.custom: seodec18

---

# Assets

An **Asset** contains digital files (including video, audio, images, thumbnail collections, text tracks and closed caption files) and the metadata about these files. After the digital files are uploaded into an asset, they could be used in the Media Services encoding and streaming workflows.

An asset is mapped to a blob container in the [Azure Storage account](storage-account-concept.md) and the files in the asset are stored as block blobs in that container. You can interact with the Asset files in the containers using the Storage SDK clients.

Azure Media Services supports Blob tiers when the account uses General-purpose v2 (GPv2) storage. With GPv2, you can move files to cool or cold storage. Cold storage is suitable for archiving source files when no longer needed (for example, after they have been encoded.)

In Media Services v3, the job input can be created from assets or from HTTP(s) URLs. To create an asset that can be used as an input for your job, see [Create a job input from a local file](job-input-from-local-file-how-to.md).

Also, read about [storage accounts in Media Services](storage-account-concept.md) and [transforms and jobs](transform-concept.md).

## Asset definition

The following table shows the Asset's properties and gives their definitions.

|Name|Description|
|---|---|
|id|Fully qualified resource ID for the resource.|
|name|The name of the resource.|
|properties.alternateId |The alternate ID of the Asset.|
|properties.assetId |The Asset ID.|
|properties.container |The name of the asset blob container.|
|properties.created |The creation date of the Asset.|
|properties.description|The Asset description.|
|properties.lastModified |The last modified date of the Asset.|
|properties.storageAccountName |The name of the storage account.|
|properties.storageEncryptionFormat |The Asset encryption format. One of None or MediaStorageEncryption.|
|type|The type of the resource.|

For the full definition, see [Assets](https://docs.microsoft.com/rest/api/media/assets).

## Filtering, ordering, paging

Media Services supports the following OData query options for Assets: 

* $filter 
* $orderby 
* $top 
* $skiptoken 

Operator description:

* Eq = equal to
* Ne = not equal to
* Ge = Greater than or equal to
* Le = Less than or equal to
* Gt = Greater than
* Lt = Less than

### Filtering/ordering

The following table shows how these options may be applied to the Asset properties: 

|Name|Filter|Order|
|---|---|---|
|id|||
|name|Supports: Eq, Gt, Lt|Supports:Ascending and Descending|
|properties.alternateId |Supports: Eq||
|properties.assetId |Supports: Eq||
|properties.container |||
|properties.created|Supports: Eq, Gt, Lt| Supports: Ascending and Descending|
|properties.description |||
|properties.lastModified |||
|properties.storageAccountName |||
|properties.storageEncryptionFormat | ||
|type|||

The following C# example filters on the created date:

```csharp
var odataQuery = new ODataQuery<Asset>("properties/created lt 2018-05-11T17:39:08.387Z");
var firstPage = await MediaServicesArmClient.Assets.ListAsync(CustomerResourceGroup, CustomerAccountName, odataQuery);
```

### Pagination

Pagination is supported for each of the four enabled sort orders. Currently, the page size is 1000.

> [!TIP]
> You should always use the next link to enumerate the collection and not depend on a particular page size.

If a query response contains many items, the service returns an "\@odata.nextLink" property to get the next page of results. This can be used to page through the entire result set. You cannot configure the page size. 

If Assets are created or deleted while paging through the collection, the changes are reflected in the returned results (if those changes are in the part of the collection that has not been downloaded.) 

The following C# example shows how to enumerate through all the assets in the account.

```csharp
var firstPage = await MediaServicesArmClient.Assets.ListAsync(CustomerResourceGroup, CustomerAccountName);

var currentPage = firstPage;
while (currentPage.NextPageLink != null)
{
    currentPage = await MediaServicesArmClient.Assets.ListNextAsync(currentPage.NextPageLink);
}
```

For REST examples, see [Assets - List](https://docs.microsoft.com/rest/api/media/assets/list)

## Storage side encryption

To protect your Assets at rest, the assets should be encrypted by the storage side encryption. The following table shows how the storage side encryption works in Media Services:

|Encryption option|Description|Media Services v2|Media Services v3|
|---|---|---|---|
|Media Services Storage Encryption|AES-256 encryption, key managed by Media Services|Supported<sup>(1)</sup>|Not supported<sup>(2)</sup>|
|[Storage Service Encryption for Data at Rest](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)|Server-side encryption offered by Azure Storage, key managed by Azure or by customer|Supported|Supported|
|[Storage Client-Side Encryption](https://docs.microsoft.com/azure/storage/common/storage-client-side-encryption)|Client-side encryption offered by Azure storage, key managed by customer in Key Vault|Not supported|Not supported|

<sup>1</sup> While Media Services does support handling of content in the clear/without any form of encryption, doing so is not recommended.

<sup>2</sup> In Media Services v3, storage encryption (AES-256 encryption) is only supported for backwards compatibility when your Assets were created with Media Services v2. Meaning v3 works with existing storage encrypted assets but will not allow creation of new ones.

## Next steps

[Stream a file](stream-files-dotnet-quickstart.md)
