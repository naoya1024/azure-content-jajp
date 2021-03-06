<properties
   pageTitle="Azure Storage Blob のデータを Data Lake Store にコピーする | Microsoft Azure"
   description="AdlCopy ツールを使用して Azure Storage Blob のデータを Data Lake Store にコピーします"
   services="data-lake-store"
   documentationCenter=""
   authors="nitinme"
   manager="jhubbard"
   editor="cgronlun"/>

<tags
   ms.service="data-lake-store"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="07/19/2016"
   ms.author="nitinme"/>

# Azure Storage BLOB から Data Lake Store へのデータのコピー

> [AZURE.SELECTOR]
- [DistCp を使用](data-lake-store-copy-data-wasb-distcp.md)
- [AdlCopy を使用](data-lake-store-copy-data-azure-storage-blob.md)

Azure Data Lake Store には、[AdlCopy](http://aka.ms/downloadadlcopy) というコマンド ライン ツールがあります。このツールを使用すると、次のソースからデータをコピーできます。

* Azure Storage BLOB から Data Lake Store へのデータのコピー。AdlCopy を使用して Data Lake Store のデータを Azure Storage BLOB にコピーすることはできません。

* 2 つの Azure Data Lake Store アカウント間でのデータのコピー。

AdlCopy ツールは 2 つの異なるモードで使用できます。

* **スタンドアロン**。Data Lake Store リソースを使用してタスクを実行する方法です。
* **Data Lake Analytics アカウントの使用**。Data Lake Analytics アカウントに割り当てられているユニットを使用して、コピー操作を実行する方法です。予測可能な方法でコピー操作を実行したい場合は、このオプションを利用できます。

##前提条件

この記事を読み始める前に、次の項目を用意する必要があります。

- **Azure サブスクリプション**。[Azure 無料試用版の取得](https://azure.microsoft.com/pricing/free-trial/)に関するページを参照してください。
- Data Lake Store パブリック プレビューに対して、**Azure サブスクリプションを有効にする**。[手順](data-lake-store-get-started-portal.md#signup)を参照してください。
- データが含まれる **Azure Storage Blobs** コンテナー。
- **Azure Data Lake Analytics アカウント (省略可能)** - Data Lake Store アカウントの作成手順については、「[チュートリアル: Azure ポータルで Azure Data Lake Analytics の使用を開始する](../data-lake-analytics/data-lake-analytics-get-started-portal.md)」を参照してください。
- **AdlCopy ツール**。[http://aka.ms/downloadadlcopy](http://aka.ms/downloadadlcopy) の AdlCopy ツールをインストールします。

## AdlCopy ツールの構文

AdlCopy ツールを操作するには、次の構文を使用します。

	AdlCopy /Source <Blob or Data Lake Store source> /Dest <Data Lake Store destination> /SourceKey <Key for Blob account> /Account <Data Lake Analytics account> /Unit <Number of Analytics units> /Pattern 

構文のパラメーターを次に示します。

| オプション | Description |
|-----------|------------|
| から | Azure Storage Blob のソース データの場所を指定します。ソースには BLOB コンテナー、BLOB、または別の Data Lake Store アカウントを指定できます。 |
| Dest | Data Lake Store のコピー先を指定します。 |
| SourceKey | Azure Storage Blob ソースのストレージ アクセス キーを指定します。これが必要なのは、ソースが BLOB コンテナーかBLOB の場合のみです。 |
| Account | **省略可能**。Azure Data Lake Analytics アカウントを使用してコピー ジョブを実行する場合に、このオプションを使用します。構文に /Account オプションを使用して、Data Lake Analytics アカウントを指定しない場合、AdlCopy は既定のアカウントを使用してジョブを実行します。また、このオプションを使用する場合、ソース (Azure Storage Blob) とコピー先 (Azure Data Lake Store) を Data Lake Analytics アカウントのデータ ソースとして追加する必要があります。 |
| Units | コピー ジョブに使用する Data Lake Analytics ユニット数を指定します。 **/Account** オプションを使用して Data Lake Analytics アカウントを指定する場合、このオプションは必須です。
| パターン | コピーするBLOB やファイルを示す regex パターンを指定します。AdlCopy は、大文字と小文字を区別する照合を使用します。すべての項目のコピーにパターンが指定されていない場合は、既定のパターンが使用されます。 複数のファイル パターンを指定することはサポートされていません。                                                                                                                                                                                                                                                                                                                                               



## AdlCopy を (スタンドアロンとして) 使用して Azure Storage Blob からデータをコピーする

1. コマンド プロンプトを開き、AdlCopy がインストールされているディレクトリ (通常は `%HOMEPATH%\Documents\adlcopy`) に移動します。

2. 次のコマンドを実行して、ソース コンテナーの特定の BLOB を Data Lake Store にコピーします。

		AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adls_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>

	For example:

		AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log /dest swebhdfs://mydatalakestore.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==

	
	>[AZURE.NOTE] 上の構文は、Data Lake Store アカウントのフォルダーにコピーするファイルを指定しています。指定されたフォルダー名が存在しない場合は、AdlCopy ツールがフォルダーを作成します。

	Data Lake Store アカウントがある Azure サブスクリプションの資格情報を入力するように求められます。次のような出力が表示されます。

		Initializing Copy.
		Copy Started.
		100% data copied.
		Finishing Copy.
		Copy Completed. 1 file copied.

3. また、あるコンテナーのすべての BLOB を Data Lake Store アカウントにコピーするには、次のコマンドも使用できます。

		AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/ /dest swebhdfs://<dest_adls_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>		

	次に例を示します。

		AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest adl://mydatalakestore.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==

## AdlCopyを (スタンドアロンとして) 使用して別の Data Lake Store アカウントからデータをコピーする

AdlCopy を使用して、2 つの Data Lake Store アカウント間でデータをコピーすることもできます。

1. コマンド プロンプトを開き、AdlCopy がインストールされているディレクトリ (通常は `%HOMEPATH%\Documents\adlcopy`) に移動します。

2. 次のコマンドを実行して、Data Lake Store アカウントからもう一方の Data Lake Store アカウントに特定のファイルをコピーします。

		AdlCopy /Source adl://<source_adls_account>.azuredatalakestore.net/<path_to_file> /dest adl://<dest_adls_account>.azuredatalakestore.net/<path>/

	For example:

		AdlCopy /Source adl://mydatastore.azuredatalakestore.net/mynewfolder/909f2b.log /dest adl://mynewdatalakestore.azuredatalakestore.net/mynewfolder/

	>[AZURE.NOTE] 上の構文は、コピー先の Data Lake Store アカウントのフォルダーにコピーするファイルを指定しています。指定されたフォルダー名が存在しない場合は、AdlCopy ツールがフォルダーを作成します。

	Data Lake Store アカウントがある Azure サブスクリプションの資格情報を入力するように求められます。次のような出力が表示されます。

		Initializing Copy.
		Copy Started.|
		100% data copied.
		Finishing Copy.
		Copy Completed. 1 file copied.

3. 次のコマンドは、ソースの Data Lake Store アカウントの特定のフォルダーから、コピー先の Data Lake Store アカウントのフォルダーにすべてのファイルをコピーします。

		AdlCopy /Source adl://mydatastore.azuredatalakestore.net/mynewfolder/ /dest adl://mynewdatalakestore.azuredatalakestore.net/mynewfolder/

## AdlCopy を (Data Lake Analytics アカウントで) 使用してデータをコピーする

Data Lake Analytics アカウントを使用して、AdlCopy ジョブを実行し、Azure Storage Blob のデータを Data Lake Store にコピーすることもできます。通常、このオプションを使用するのは、移動対象のデータが GB および TB の範囲で、パフォーマンスのスループットを向上し、予測可能にする場合です。

Data Lake Analytics アカウントと AdlCopy を使用して、Azure Storage Blob からコピーするには、ソース (Azure Storage Blob) を Data Lake Analytics アカウントのデータ ソースとして追加する必要があります。データ ソースを Data Lake Analytics アカウントに追加する手順については、[Data Lake Analytics の「アカウント データ ソースの管理」](..//data-lake-analytics/data-lake-analytics-manage-use-portal.md#manage-account-data-sources)を参照してください。

>[AZURE.NOTE] Data Lake Analytics アカウントを使用して、ソースとして Azure Data Lake Store アカウントからコピーする場合は、Data Lake Store アカウントを Data Lake Analytics アカウントに関連付ける必要はありません。Data Lake Analytics アカウントにソース ストアを関連付ける必要があるのは、ソースが Azure Storage アカウントの場合のみです。

次のコマンドを実行して、 Data Lake Analytics アカウントを使用して、Azure Storage Blob から Data Lake Store アカウントにコピーします。

	AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adls_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Account <data_lake_analytics_account> /Unit <number_of_data_lake_analytics_units_to_be_used>

次に例を示します。

	AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest swebhdfs://mydatalakestore.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Account mydatalakeanalyticaccount /Units 2


同様に、次のコマンドを実行して、Data Lake Analyticsアカウントを使用して、Azure Storage Blob から Data Lake Store アカウントにコピーします。

	AdlCopy /Source adl://mysourcedatalakestore.azuredatalakestore.net/mynewfolder/ /dest adl://mydestdatastore.azuredatalakestore.net/mynewfolder/ /Account mydatalakeanalyticaccount /Units 2

## AdlCopy を使用したパターン一致によるデータのコピー

このセクションでは、AdlCopy を使用して、ソース (次の例では Azure Storage Blob を使用) からコピー先のData Lake Store アカウントに、パターン一致によるデータのコピーを行う方法について説明します たとえば、次の手順を使用して、 .csv 拡張子が付いたすべてのファイルをソース BLOB からコピー先にコピーすることができます。

1. コマンド プロンプトを開き、AdlCopy がインストールされているディレクトリ (通常は `%HOMEPATH%\Documents\adlcopy`) に移動します。

2. 次のコマンドを実行して、*.csv 拡張子が付いたすべてのファイルを、ソース コンテナーの特定の BLOB から Data Lake Store にコピーします。

		AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adls_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Pattern *.csv

	For example:

		AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/FoodInspectionData/ /dest adl://mydatalakestore.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Pattern *.csv

	
## 課金

* AdlCopy ツールをスタンドアロンとして使用する場合、ソースの Azure Storage アカウントが Data Lake Store と同じリージョン内になければ、データの移動に関する送信コストがかかります。

* Data Lake Analytics アカウントで AdlCopy ツールを使用する場合、標準の [Data Lake Analytics 料金](https://azure.microsoft.com/pricing/details/data-lake-analytics/)が適用されます。

## AdlCopy の使用に関する注意点

* AdlCopy (バージョン 1.0.5) では、合計で数千個を超えるファイルとフォルダーがあるソースからのデータのコピーをサポートしています。ただし、大規模なデータセットをコピーする際に問題が発生する場合は、ファイルやフォルダーを異なるサブフォルダーに分散し、代わりにこれらのサブフォルダーへのパスをソースとして使用します。

## 次のステップ

- [Data Lake Store のデータをセキュリティで保護する](data-lake-store-secure-data.md)
- [Data Lake Store で Azure Data Lake Analytics を使用する](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
- [Data Lake Store で Azure HDInsight を使用する](data-lake-store-hdinsight-hadoop-use-portal.md)

<!---HONumber=AcomDC_0914_2016-->