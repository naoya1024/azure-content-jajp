<properties
	pageTitle="Ruby から Azure Table Storage を使用する方法 | Microsoft Azure"
	description="NoSQL データ ストアの Azure Table Storage を使用して、構造化データをクラウドに格納します。"
	services="storage"
	documentationCenter="ruby"
	authors="rmcmurray"
	manager="wpickett"
	editor=""/>
<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="ruby"
	ms.topic="article"
	ms.date="08/11/2016"
	ms.author="robmcm"/>


# Ruby から Azure Table Storage を使用する方法

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
<br/>
[AZURE.INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-tables.md)]

## Overview

このガイドでは、Azure Table サービスを使用して一般的なシナリオを実行する方法について説明します。サンプルは Ruby API を使用して記述されています。紹介するシナリオは、**テーブルの作成と削除、テーブルのエンティティの挿入とクエリ実行**などです。

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## Ruby アプリケーションの作成

Ruby アプリケーションの作成方法については、「[Azure VM での Ruby on Rails Web アプリケーション](../virtual-machines/virtual-machines-linux-classic-ruby-rails-web-app.md)」をご覧ください。


## アプリケーションのストレージへのアクセスの構成

Azure Storage を使用するには、Ruby azure パッケージをダウンロードして使用する必要があります。このパッケージには、Storage REST サービスと通信するための便利なライブラリのセットが含まれています。

### RubyGems を使用してパッケージを取得する

1. **PowerShell** (Windows)、**ターミナル** (Mac)、**Bash** (Unix) などのコマンド ライン インターフェイスを使用します。

2. コマンド ウィンドウに「**gem install azure**」と入力して、gem と依存関係をインストールします。

### パッケージをインポートする

任意のテキスト エディターを使用して、Storage を使用する Ruby ファイルの先頭に次のコードを追加します。

	require "azure"

## Azure Storage の接続文字列の設定

azure モジュールは、Azure Storage アカウントに接続するために必要な情報として、環境変数 **AZURE\_STORAGE\_ACCOUNT** と **AZURE\_STORAGE\_ACCESS\_KEY** を読み取ります。これらの環境変数が設定されていない場合は、**Azure::TableService** を使用する前に、次のコードを使用してアカウント情報を指定する必要があります。

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your azure storage access key>"

Azure ポータルでクラシックまたは Resource Manager ストレージ アカウントからこれらの値を取得するには:

1. [Azure ポータル](https://portal.azure.com)にログインします。
2. 使用するストレージ アカウントを表示します。
3. 右側の [設定] ブレードで、**[アクセス キー]** をクリックします。
4. 表示される [アクセス キー] ブレードに、アクセス キー 1 とアクセス キー 2 が表示されます。このいずれかを使用できます。
5. コピー アイコンをクリックしてキーをクリップボードにコピーします。

Azure ポータルでクラシック ストレージ アカウントからこれらの値を取得するには:

1. [従来の Azure ポータル](https://manage.windowsazure.com)にログインします。
2. 使用するストレージ アカウントを表示します。
3. ナビゲーション ウィンドウの下部にある **[アクセス キーの管理]** をクリックします。
4. ポップアップ ダイアログに、ストレージ アカウント名、プライマリ アクセス キー、およびセカンダリ アクセス キーが表示されます。アクセス キーには、プライマリとセカンダリのどちらでも使用できます。
5. コピー アイコンをクリックしてキーをクリップボードにコピーします。

## テーブルを作成する

**Azure::TableService** オブジェクトを使用すると、テーブルとエンティティを操作できます。テーブルを作成するには、**create\_table()** メソッドを使用します。次の例では、テーブルを作成し、既に存在している場合はエラーを出力します。

	azure_table_service = Azure::TableService.new
	begin
	  azure_table_service.create_table("testtable")
	rescue
	  puts $!
	end

## エンティティをテーブルに追加する

エンティティを追加するには、エンティティのプロパティを定義するハッシュ オブジェクトを最初に作成します。すべてのエンティティについて、**PartitionKey** と **RowKey** を指定する必要があることに注意してください。これらはエンティティの一意の識別子であり、他のエンティティのプロパティよりはるかに高速に照会できる値です。Azure Storage では、テーブルのエンティティを多数のストレージ ノードに自動的に配布するために **PartitionKey** を使用します。**PartitionKey** が同じエンティティは同じノードに格納されます。**RowKey** は、エンティティが属するパーティション内のエンティティの一意の ID です。

	entity = { "content" => "test entity",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.insert_entity("testtable", entity)

## エンティティを更新する

既存のエンティティを更新するには、複数のメソッドがあります。

* **update\_entity():** 既存のエンティティを、置換することで更新します。
* **merge\_entity():** 既存のエンティティを、新しいプロパティ値を既存のエンティティにマージすることで更新します。
* **insert\_or\_merge\_entity():** 既存のエンティティを、置換することで更新します。エンティティが存在しない場合は、新しいエンティティが挿入されます。
* **insert\_or\_replace\_entity():** 既存のエンティティを、新しいプロパティ値を既存のエンティティにマージすることで更新します。エンティティが存在しない場合は、新しいエンティティが挿入されます。

次の例に、**update\_entity()** を使用してエンティティを更新する方法を示します。

	entity = { "content" => "test entity with updated content",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.update_entity("testtable", entity)

**update\_entity()** と **merge\_entity()** では、更新されるエンティティが存在しない場合、更新操作は失敗します。したがって、既に存在しているかどうかに関係なくエンティティを格納するには、代わりに **insert\_or\_replace\_entity()** または **insert\_or\_merge\_entity()** を使用する必要があります。

## エンティティのグループを操作する

状況によって、複数の操作をバッチとして送信し、サーバーによるアトミック処理を行うことが合理的である場合があります。このためには、まず **Batch** オブジェクトを作成し、次に **TableService** の **execute\_batch()** メソッドを使用します。次の例では、RowKey が 2 および 3 である 2 つのエンティティをバッチで送信する方法を示します。これは、同じ PartitionKey を持つエンティティでのみ機能することに注意してください。

	azure_table_service = Azure::TableService.new
	batch = Azure::Storage::Table::Batch.new("testtable",
	  "test-partition-key") do
	  insert "2", { "content" => "new content 2" }
	  insert "3", { "content" => "new content 3" }
	end
	results = azure_table_service.execute_batch(batch)

## エンティティを照会する

テーブル内のエンティティを照会するには、**get\_entity()** メソッドを使用して、テーブル名、**PartitionKey**、および **RowKey** を渡します。

	result = azure_table_service.get_entity("testtable", "test-partition-key",
	  "1")

## エンティティのセットを照会する

テーブル内のエンティティのセットを照会するには、クエリ ハッシュ オブジェクトを作成し、**query\_entities()** メソッドを使用します。次の例では、同じ **PartitionKey** を持つエンティティをすべて取得します。

	query = { :filter => "PartitionKey eq 'test-partition-key'" }
	result, token = azure_table_service.query_entities("testtable", query)

> [AZURE.NOTE] 結果セットが大きすぎて単一のクエリで返すことができない場合は、継続トークンが返されます。この継続トークンを使用して、後続のページを取得できます。

## エンティティ プロパティのサブセットを照会する

テーブルに対するクエリでは、ごくわずかのプロパティだけをエンティティから取得できます。"プロジェクション" と呼ばれるこの方法では、帯域幅の使用が削減され、クエリのパフォーマンスが向上します。特に、大量のエンティティがある場合に役立ちます。select 句を使用して、クライアントに渡すプロパティの名前を指定します。

	query = { :filter => "PartitionKey eq 'test-partition-key'",
	  :select => ["content"] }
	result, token = azure_table_service.query_entities("testtable", query)

## エンティティを削除する

エンティティを削除するには、**delete\_entity()** メソッドを使用します。目的のエンティティを含んでいるテーブルの名前と、エンティティの PartitionKey および RowKey を渡す必要があります。

		azure_table_service.delete_entity("testtable", "test-partition-key", "1")

## テーブルを削除する

テーブルを削除するには、**delete\_table()** メソッドを使用して、削除するテーブルの名前を渡します。

		azure_table_service.delete_table("testtable")

## 次のステップ

さらに複雑なストレージ タスクの詳細については、次のリンク先をご覧ください。

- [Azure Storage チーム ブログ](http://blogs.msdn.com/b/windowsazurestorage/)
- GitHub の [Azure SDK for Ruby](http://github.com/WindowsAzure/azure-sdk-for-ruby) リポジトリ

<!---HONumber=AcomDC_0817_2016--->