---
title: "Azure Cosmos DB でテーブル データのクエリを実行する方法 | Microsoft Docs"
description: "Azure Cosmos DB でテーブル データのクエリを実行する方法を学習する"
services: cosmosdb
documentationcenter: 
author: kanshiG
manager: jhubbard
editor: 
tags: 
ms.assetid: 14bcb94e-583c-46f7-9ea8-db010eb2ab43
ms.service: cosmosdb
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 05/10/2017
ms.author: govindk
ms.translationtype: Human Translation
ms.sourcegitcommit: 5e92b1b234e4ceea5e0dd5d09ab3203c4a86f633
ms.openlocfilehash: cdd855aeac7dd30c52accb407289ca6db7dab4ae
ms.contentlocale: ja-jp
ms.lasthandoff: 05/10/2017


---

# <a name="azure-cosmos-db-how-to-query-with-the-table-api-preview"></a>Azure Cosmos DB: Table API (プレビュー) を使用してクエリを実行する方法

Azure Cosmos DB [Table API](table-introduction.md) (プレビュー) では、キー/値 (テーブル) データに対する OData クエリと [LINQ](https://docs.microsoft.com/rest/api/storageservices/fileservices/writing-linq-queries-against-the-table-service) クエリがサポートされます。  

この記事に含まれるタスクは次のとおりです。 

> [!div class="checklist"]
> * Table API を使用してデータのクエリを実行する

## <a name="sample-table"></a>サンプル テーブル

この記事のクエリは、次の `People` サンプル テーブルを使用します。

| PartitionKey | RowKey | 電子メール | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0101 |
| Smith | Walter | Ben@contoso.com| 425-555-0102 |
| Smith | Jeff | Jeff@contoso.com| 425-555-0104 | 

## <a name="about-the-table-api-preview"></a>Table API (プレビュー) について

Azure Cosmos DB は Azure Table Storage API との互換性があるため、Table API を使用してクエリを実行する方法の詳細については、「Querying Tables and Entities (テーブルとエンティティのクエリ)」(https://docs.microsoft.com/rest/api/storageservices/fileservices/querying-tables-and-entities) を参照してください。 

Azure Cosmos DB によって提供される Premium 機能の詳細については、[Azure Cosmos DB: Table API](table-introduction.md) や [.NET を使用した Table API での開発](tutorial-develop-table-dotnet.md)に関する記事を参照してください。 

## <a name="prerequisites"></a>前提条件

クエリを実行するには、Azure Cosmos DB アカウントがあり、コンテナーにエンティティ データがあることが必要です。 どちらもない場合には、 [5 分でできるクイックスタート](https://aka.ms/acdbtnetqs)か[開発者向けチュートリアル](https://aka.ms/acdbtabletut)を実行して、アカウントを作成し、データベースにデータを設定します。

## <a name="querying-on-partition-key-and-row-key"></a>パーティション キーと行キーにクエリを実行する
PartitionKey プロパティと RowKey プロパティによってエンティティのプライマリ キーが構成されるため、特別な構文を使用すると、次のようにエンティティを特定できます。 

**クエリ**

```
https://<mytableendpoint>/People(PartitionKey='Harp',RowKey='Walter')  
```
**結果**

| PartitionKey | RowKey | 電子メール | PhoneNumber |
| --- | --- | --- | --- |
| Harp | Walter | Walter@contoso.com| 425-555-0104 |

または、次のセクションで説明するように、これらのプロパティを `$filter` オプションに含めて指定することもできます。 キーのプロパティ名と定数値では大文字と小文字が区別されます。 PartitionKey プロパティと RowKey プロパティはいずれも文字列型です。 

## <a name="querying-with-an-odata-filter"></a>ODATA フィルターを使用してクエリを実行する
フィルター文字列を指定するときは、次のルールに注意してください。 

* OData プロトコル仕様で定義された論理演算子を使用して、プロパティと値を比較します。 プロパティを動的な値と比較することはできません。式の 1 つの辺は定数である必要があります。 
* プロパティ名、演算子、および定数値は、URL でエンコードされた空白で区切る必要があります。 空白は URL エンコードでは `%20` となります。 
* フィルター文字列のすべての要素は大文字と小文字が区別されます。 
* フィルターで有効な結果を得るためには、定数値をプロパティと同じデータ型にする必要があります。 サポートされているプロパティ型の詳細については、 [Table サービス データ モデル](https://docs.microsoft.com/rest/api/storageservices/understanding-the-table-service-data-model)に関するページを参照してください。 

ODATA `$filter` を使用して、PartitionKey と Email プロパティによってフィルター処理する方法を、次のサンプル クエリで示します。

**クエリ**

```
https://<mytableapi-endpoint>/People()?$filter=PartitionKey%20eq%20'Smith'%20and%20Email%20eq%20'Ben@contoso.com'
```

さまざまなデータ型のフィルター式の作成方法の詳細については、「[Querying Tables and Entities](https://docs.microsoft.com/rest/api/storageservices/querying-tables-and-entities)」(テーブルとエンティティのクエリ) を参照してください。

**結果**

| PartitionKey | RowKey | 電子メール | PhoneNumber |
| --- | --- | --- | --- |
| Ben |Smith | Ben@contoso.com| 425-555-0102 |

## <a name="querying-with-linq"></a>LINQ を使用してクエリを実行する 
LINQ を使用してクエリを実行することもできます。これは、対応する ODATA クエリ式に変換されます。 .NET SDK を使用してクエリを作成する方法の例を次に示します。

```csharp
CloudTableClient tableClient = account.CreateCloudTableClient();
CloudTable table = tableClient.GetTableReference("people");

TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>()
    .Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition(PartitionKey, QueryComparisons.Equal, "Smith"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition(Email, QueryComparisons.Equal,"Ben@contoso.com")
    ));

await table.ExecuteQuerySegmentedAsync<CustomerEntity>(query, null);
```

## <a name="next-steps"></a>次のステップ

このチュートリアルでは、次の手順を行いました。

> [!div class="checklist"]
> * Table API (プレビュー) を使用してクエリを実行する方法を学習しました。 

次のチュートリアルに進んで、データをグローバルに分散する方法について学習できます。

> [!div class="nextstepaction"]
> [データをグローバルに分散する](tutorial-global-distribution-documentdb.md)

