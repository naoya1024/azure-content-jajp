<properties
   pageTitle="Azure リソース マネージャーの概要 | Microsoft Azure"
   description="Azure リソース マネージャーを使用して、Azure のリソースをデプロイ、管理、およびのアクセス制御する方法について説明します。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/16/2016"
   ms.author="tomfitz"/>

# Azure リソース マネージャーの概要

アプリケーションのインフラストラクチャは通常、仮想マシン、ストレージ アカウント、仮想ネットワーク、Web アプリ、データベース、データベース サーバー、サード パーティのサービスなど、複数のコンポーネントで構成されます。これらのコンポーネントは別々のエンティティではなく、1 つのエンティティの中で互いに関連付けられ相互依存しています。これらのコンポーネントを、1 つのグループとしてデプロイ、管理、および監視するのが好ましいです。Azure リソース マネージャーを使用すると、ソリューション内の複数のリソースを 1 つのグループとして作業できます。ソリューションのこれらすべてのリソースを、1 回の連携した操作でデプロイ、更新、または削除できます。デプロイにはテンプレートを使用しますが、このテンプレートは、テスト、ステージング、運用環境などのさまざまな環境に使用できます。リソース マネージャーには、デプロイ後のリソースの管理に役立つ、セキュリティ、監査、タグ付けの機能が用意されています。

## 用語集

Azure Resource Manager には、初めて使う方にとって、あまり馴染みのない用語がいくつか存在します。

- **リソース** - Azure を通じて管理できる要素。一般的なリソースとしては、仮想マシン、ストレージ アカウント、Web アプリ、データベース、仮想ネットワークなどがありますが、他にもさまざまなリソースが存在します。
- **リソース グループ** - Azure ソリューションの関連するリソースを保持するコンテナー。リソース グループには、ソリューションのすべてのリソースか、グループとして管理したいリソースのみを含めることができます。組織のニーズに合わせてリソースをリソース グループに割り当てる方法を指定してください。「[リソース グループ](#resource-groups)」を参照してください。
- **リソース プロバイダー** - Resource Manager を使用してデプロイおよび管理できるリソースを提供するサービスです。各リソース プロバイダーは、デプロイされたリソースを利用するための操作を提供します。一般的なリソース プロバイダーとしては、仮想マシン リソースを提供する Microsoft.Compute や、ストレージ アカウント リソースを提供する Microsoft.Storage、Web アプリに関連したリソースを提供する Microsoft.Web などがあります。「[リソース プロバイダー](#resource-providers)」を参照してください。
- **Resource Manager テンプレート** - リソース グループへのデプロイ対象となるリソースを定義する JavaScript Object Notation (JSON) ファイルです。デプロイ対象リソース間の依存関係も、このテンプレートによって定義されます。このテンプレートを使えば、リソースを一貫性のある形で繰り返しデプロイできます。「[テンプレートのデプロイ](#template-deployment)」を参照してください。
- **宣言型構文** - 一連のプログラミング コマンドを記述しなくても、"作成しようとしているもの" を明確に宣言することのできる構文です。宣言型構文の例として、Resource Manager テンプレートがあります。このファイルで、Azure にデプロイするインフラストラクチャのプロパティを定義します。

## リソース マネージャーを使用する利点

リソース マネージャーには、いくつかの利点があります。

- ソリューションのリソースを個別に処理するのではなく、すべてのリソースをグループとしてデプロイ、管理、監視できます。
- ソリューションを開発のライフサイクル全体で繰り返しデプロイできます。また、常にリソースが一貫した状態でデプロイされます。
- スクリプトではなく宣言型のテンプレートを使用してインフラストラクチャを管理できます。
- 正しい順序でデプロイされるようにリソース間の依存性を定義できます。
- ロールベースの Access Control (RBAC) が管理プラットフォームにネイティブ統合されるため、リソース グループのすべてのサービスにアクセス制御を適用できます。
- タグをリソースに適用し、サブスクリプションのすべてのリソースを論理的に整理できます。
- 同じタグを共有するリソース グループのコストを表示することで、組織の課金をわかりやすくすることができます。

リソース マネージャーには、ソリューションをデプロイして管理するための新しい方法が用意されています。以前のデプロイメント モデルを使用していて、変更の詳細を確認する場合は、「[リソース マネージャー デプロイと従来のデプロイを理解する](resource-manager-deployment-model.md)」を参照してください。

## ガイダンス

次の推奨事項は、ソリューションを操作する際に Resource Manager を最大限に活用するのに役立ちます。

1. インフラストラクチャを定義してデプロイする場合は、命令型コマンドではなく、リソース マネージャーのテンプレートにある宣言型構文を使用します。
2. デプロイと構成の手順すべてをこのテンプレートで定義します。ソリューションの設定に手動操作は必要ありません。
3. アプリやコンピューターの開始または停止など、リソースの管理には命令型コマンドを実行します。
4. リソース グループ内の同じライフサイクルでリソースを調整します。リソースのその他すべての整理には、タグを使用します。

その他の推奨事項については、「[Azure Resource Manager テンプレートを作成するためのベスト プラクティス](resource-manager-template-best-practices.md)」を参照してください。

## リソース グループ

リソース グループを定義する際、次のような考慮すべき要素があります。

1. グループ内のすべてのリソースで、同じライフサイクルが共有される必要がある。そのため、これらのリソースは一緒にデプロイ、更新、削除されます。データベース サーバーなどの 1 つのリソースが、別のデプロイ サイクル上に存在する必要がある場合は、別のリソース グループに含めなければなりません。
2. 各リソースは、1 つのリソース グループにのみ存在できる。
3. リソースは、いつでもリソース グループに追加したり、削除できる。
4. あるリソース グループから別のリソース グループへリソースを移動できる。詳細については、「[新しいリソース グループまたはサブスクリプションへのリソースの移動](resource-group-move-resources.md)」を参照してください。
4. リソース グループには、別のリージョンに存在するリソースを含めることができる。
5. リソース グループを使用すると、管理操作のアクセス制御のスコープを設定できる。
6. リソースは、他のリソース グループ内のリソースとやり取りできる。このやり取りは、2 つの関連するリソースで同じライフサイクルが共有されていない場合によく見られます (データベースに接続する Web アプリなど)。

リソース グループを作成するとき、そのリソース グループの場所を指定する必要があります。"なぜリソース グループに場所が必要なのか。 リソースがリソース グループとは異なる場所に存在してよいとしたら、いったいなぜリソース グループの場所が問題になるのか" と、疑問に思われるかもしれません。 リソース グループには、リソースについてのメタデータが格納されます。そのため、リソース グループの場所を指定するとき、このメタデータが格納される場所を指定することになります。コンプライアンス上の理由から、データは特定のリージョンに格納されるようにする必要があります。

## リソース プロバイダー

技術的側面を扱うための一連のリソースと操作は、それぞれのリソース プロバイダーから得られます。たとえば、キーとシークレットを格納するためには、**Microsoft.KeyVault** リソース プロバイダーを使用します。このリソース プロバイダーは、キー コンテナーを作成するための **vaults** というリソースの種類、キー コンテナーにシークレットを作成するための **vaults/secrets** というリソースの種類を提供します。また一連の操作は、[Key Vault REST API 操作](https://msdn.microsoft.com/library/azure/dn903609.aspx)を通じて利用できます。REST API は直接呼び出すことができるほか、[Key Vault PowerShell コマンドレット](https://msdn.microsoft.com/library/dn868052.aspx)と [Key Vault Azure CLI](./key-vault/key-vault-manage-with-cli.md) を使用して Key Vault を管理することもできます。また、ほとんどのリソースは、いくつかのプログラミング言語を使用して操作することもできます。詳細については、[SDK とサンプル](#sdks-and-samples)を参照してください。

インフラストラクチャをデプロイして管理するには、リソース プロバイダーに関する詳細を確認する必要があります。リソースの種類、REST API 操作のバージョン番号、サポートされている操作、リソースの作成に使用するスキーマを確認する必要があります。サポートされているリソース プロバイダーについては、「[Resource Manager のプロバイダー、リージョン、API のバージョン、およびスキーマ](resource-manager-supported-services.md)」を参照してください。

## テンプレートのデプロイ

Resource Manager では、アプリケーションのデプロイと構成を定義する JSON 形式のテンプレートを作成できます。テンプレートを使用すると、アプリケーションをアプリのライフサイクルを通して繰り返しデプロイできるほか、常にリソースが一貫した状態でデプロイされます。依存関係は Azure Resource Manager によって分析され、確実に正しい順序でリソースが作成されます。詳細については、「[Azure リソース マネージャーのテンプレートでの依存関係の定義](resource-group-define-dependencies.md)」を参照してください。

ポータルからソリューションを作成すると、ソリューションには自動的にデプロイ テンプレートが含まれます。最初からテンプレートを作成する必要はありません。はじめにソリューション向けのテンプレートを使用して、それを特定のニーズに合わせてカスタマイズできます。リソース グループの現在の状態をエクスポートするか、特定のデプロイに使用されたテンプレートを表示することで、既存のリソース グループのテンプレートを取得できます。エクスポートしたテンプレートを表示すると、テンプレートの構文について理解するのに役立ちます。エクスポートしたテンプレートの操作の詳細については、「[既存のリソースから Azure Resource Manager テンプレートをエクスポートする](resource-manager-export-template.md)」を参照してください。

インフラストラクチャ全体を 1 つのテンプレートで定義する必要はありません。多くの場合、対象を絞って目的を特化した一連のテンプレートにデプロイの要件を分類することが合理的です。これらのテンプレートは、さまざまなソリューションで簡単に再利用できます。特定のソリューションをデプロイするには、必要なすべてのテンプレートをリンクするマスター テンプレートを作成します。詳細については、「[Azure リソース マネージャーでのリンクされたテンプレートの使用](resource-group-linked-templates.md)」を参照してください。

インフラストラクチャの更新にも、テンプレートを使用することができます。たとえば、ソリューションにリソースを追加したり、既にデプロイされたリソースに構成ルールを追加したりできます。テンプレートでリソースの作成を指定した際、そのリソースが既に存在する場合は、Azure Resource Manager では、新しい資産を作成する代わりに更新が実行されます。Azure リソース マネージャーでは、既存の資産が、新しい資産と同じ状態になるよう更新されます。または、テンプレートで指定されていないリソースをリソース マネージャーで削除するように指定することができます。デプロイ時の各種のオプションについては、「[Azure リソース マネージャーのテンプレートを使用したアプリケーションのデプロイ](resource-group-template-deploy.md)」を参照してください。

テンプレートでパラメーターを指定すると、デプロイをカスタマイズでき、柔軟性が高まります。たとえば、テスト環境に合わせてデプロイを調整するためのパラメーター値を渡すことができます。パラメーターを指定すると、同じテンプレートを、さまざまな環境へのソリューションのデプロイに使用できます。

リソース マネージャーでは、セットアップ時に含まれていなかった特定ソフトウェアのインストールなど、追加の操作が必要なシナリオのための拡張機能を使用できます。DSC、Chef、または Puppet などの構成管理サービスをすでにご利用の場合は、拡張機能を使用すれば、引き続きそのサービスで作業ができます。

最後に、テンプレートは、アプリのソース コードの一部になります。テンプレートはソース コード リポジトリにチェックインして、アプリの変更に合わせて更新できます。テンプレートは Visual Studio から編集できます。

テンプレートの定義の詳細については、[Azure リソース マネージャーのテンプレートの作成](resource-group-authoring-templates.md)を参照してください。

テンプレートの作成手順については、「[Resource Manager テンプレートのチュートリアル](resource-manager-template-walkthrough.md)」を参照してください。

ソリューションを別の環境にデプロイする方法については、「[Microsoft Azure の開発環境とテスト環境](solution-dev-test-environments.md)」を参照してください。

## タグ

リソース マネージャーでは、管理や課金の要件に合わせてリソースを分類できる、タグ付け機能を使用できます。リソース グループとリソースが複雑に絡み合っており、これらの資産をわかりやすく視覚化する必要がある場合は、タグを使用してください。たとえば、組織内で同じロールを果たしている複数リソース、または同じ部門に属している複数リソースにタグを付けることができます。タグがないと、組織内のユーザーは複数のリソースを作成できるものの、作成したリソースを後で特定して管理することが困難になる場合があります。たとえば、特定のプロジェクトに関するすべてのリソースを削除したい場合があります。これらのリソースに対してプロジェクトに関するタグが設定されていない場合、手動で探す必要があります。タグ付けは、サブスクリプションの不要なコストを削減するための重要な方法です。

タグを共有するために、リソースが同じリソース グループ内に格納されている必要はありません。組織内のユーザーが類似したタグを誤って適用しないよう (「department」の代わりに「dep」など)、組織内のすべてのユーザーが共通のタグを使用できる独自のタグ分類法を作成することができます。

タグの詳細については、[タグを使用した Azure リソースの整理](resource-group-using-tags.md)を参照してください。デプロイ中にリソースへのタグの追加を要求する、[カスタマイズしたポリシー](#manage-resources-with-customized-policies)を作成することができます。

## Access control

リソース マネージャーを使用すると、組織に対する特定アクションにアクセスできるユーザーを制御できます。Azure リソース マネージャーでは、OAuth およびロールベースのアクセス制御 (RBAC) が管理プラットフォームにネイティブに統合されているため、そのアクセス制御がリソース グループ内のすべてのサービスに適用されます。事前定義されたプラットフォームとリソース固有のロールにユーザーを追加し、これらのロールをサブスクリプション、リソース グループ、またはリソースに適用することで、アクセスを制限できます。たとえば、ユーザーにデータベース管理は許可するが、データベース サーバーやセキュリティ ポリシーの管理は許可しない SQL DB 共同作業者と呼ばれる事前定義済みのロールを利用するとします。このようなアクセスを必要とする組織内のユーザーを SQL DB 共同作成者ロールに追加し、このロールをサブスクリプション、リソース グループ、またはリソースに適用します。

リソース マネージャーでは、監査のために、自動的にユーザー操作が記録されます。監査ログの使用については、「[Resource Manager の監査操作](resource-group-audit.md)」を参照してください。

ロールベースのアクセス制御の詳細については、「[Azure のロールベースのアクセス制御](./active-directory/role-based-access-control-configure.md)」を参照してください。「[RBAC: 組み込みのロール](./active-directory/role-based-access-built-in-roles.md)」トピックには、組み込みのロールと許可されているアクションの一覧が記載されています。組み込みのロールには、所有者、閲覧者、共同作業者などの一般的なロールに加え、仮想マシンの共同作業者、仮想ネットワークの共同作業者、SQL セキュリティ管理者など、サービス固有のロールがあります (使用可能なロールの数例にすぎません)。

重要なリソースを、ユーザーによって削除または変更されないように、明示的にロックすることもできます。詳細については、「[Azure リソース マネージャーによるリソースのロック](resource-group-lock-resources.md)」を参照してください。

ベスト プラクティスについては、「[Azure リソース マネージャーのセキュリティに関する考慮事項](best-practices-resource-manager-security.md)」を参照してください。

## カスタマイズしたポリシーによるリソースの管理

リソース マネージャーでは、リソースを管理するためにカスタマイズされたポリシーを作成できます。多様なシナリオに対応した各種ポリシーを作成できます。リソースに対して名前付け規則を適用できるほか、デプロイできるリソースの種類とインスタンスや、特定の種類のリソースをホストできるリージョンを制限できます。また、部門別に課金を整理するために、リソースへのタグ値の設定を義務付けることができます。コストを削減し、サブスクリプション内での一貫性を維持できるようにポリシーを作成します。詳細については、「[ポリシーを使用したリソース管理とアクセス制御](resource-manager-policy.md)」を参照してください。

## 一貫性のある管理レイヤー

Resource Manager での操作は、Azure PowerShell、Mac、Linux、および Windows 用の Azure CLI、Azure Portal、または REST API の操作と互換性があります。ユーザーに最適なインターフェイスを使用できるほか、インターフェイス間を混乱せずにすばやく移動できます。

PowerShell に関する詳細については、[リソース マネージャーでの Windows PowerShell の使用](powershell-azure-resource-manager.md)および [Azure Resource Manager Cmdlets (Azure リソース マネージャのコマンドレッド)](https://msdn.microsoft.com/library/azure/dn757692.aspx) を参照してください。

Azure CLI の詳細については、[Azure リソース管理での、Mac、Linux、および Windows 用 Azure CLI の使用](xplat-cli-azure-resource-manager.md)を参照してください。

REST API の詳細については、[Azure リソース マネージャー REST API リファレンス](https://msdn.microsoft.com/library/azure/dn790568.aspx)を参照してください。デプロイしたリソースの REST 操作を表示する方法については、「[Azure リソース エクスプローラーを使用してリソースを表示および変更する](resource-manager-resource-explorer.md)」を参照してください。

ポータルの使用については、「[Resource Manager テンプレートと Azure ポータルを使用したリソースのデプロイ](resource-group-template-deploy-portal.md)」を参照してください。

Azure リソース マネージャーでは、クロス オリジン リソース共有 (CORS) がサポートされています。CORS を使用すると、別のドメインに存在する Web アプリケーションからリソース マネージャー REST API または Azure サービス REST API を呼び出すことができます。CORS がサポートされていない場合は、Web ブラウザーによって、あるドメイン内のアプリは別のドメイン内のリソースにアクセスできないようになっています。リソース マネージャーを使用すると、有効な認証の資格情報を持つすべての要求で CORS が有効になります。

## SDK

Azure の SDK は、複数の言語とプラットフォームで利用できます。これらの言語実装はそれぞれ、エコシステム パッケージ マネージャーと GitHub を介して入手できます。

これらの SDK のコードはそれぞれ、Azure REST ベースの API 仕様から生成されます。これらの仕様はオープン ソースであり、Swagger 2.0 仕様に基づいています。SDK コードは、AutoRest と呼ばれるオープン ソース プロジェクトを使用して生成されています。AutoRest では、これらの REST ベースの API 仕様が、複数の言語でクライアント ライブラリに変換されます。SDK で生成されたコードに改善する箇所がある場合、広範に適用されている API 仕様形式に基づく、SDK を作成するためのツール全体が自由に使用できます。

Microsoft のオープン ソース SDK リポジトリを次に示します。フィードバック、問題、プル要求をお待ちしております。

[.NET](https://github.com/Azure/azure-sdk-for-net) | [Java](https://github.com/Azure/azure-sdk-for-java) | [Node.js](https://github.com/Azure/azure-sdk-for-node) | [PHP](https://github.com/Azure/azure-sdk-for-php) | [Python](https://github.com/Azure/azure-sdk-for-python) | [Ruby](https://github.com/Azure/azure-sdk-ruby)

> [AZURE.NOTE] 必要な機能が SDK に備わっていない場合は、[Azure REST API](https://msdn.microsoft.com/library/azure/dn790568.aspx) を直接呼び出すこともできます。

## サンプル

### .NET

- [Azure のリソースとリソース グループを管理する](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-resources-and-groups/)
- [テンプレートを使用して SSH 対応 VM をデプロイする](https://azure.microsoft.com/documentation/samples/resource-manager-dotnet-template-deployment/)

### Java

- [Azure のリソースを管理する](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource/)
- [Azure のリソース グループを管理する](https://azure.microsoft.com/documentation/samples/resources-java-manage-resource-group/)
- [テンプレートを使用して SSH 対応 VM をデプロイする](https://azure.microsoft.com/documentation/samples/resources-java-deploy-using-arm-template/)

### Node.js

- [Azure のリソースとリソース グループを管理する](https://azure.microsoft.com/documentation/samples/resource-manager-node-resources-and-groups/)
- [テンプレートを使用して SSH 対応 VM をデプロイする](https://azure.microsoft.com/documentation/samples/resource-manager-node-template-deployment/)

### Python

- [Azure のリソースとリソース グループを管理する](https://azure.microsoft.com/documentation/samples/resource-manager-python-resources-and-groups/)
- [テンプレートを使用して SSH 対応 VM をデプロイする](https://azure.microsoft.com/documentation/samples/resource-manager-python-template-deployment/)

### Ruby

- [Azure のリソースとリソース グループを管理する](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-resources-and-groups/)
- [テンプレートを使用して SSH 対応 VM をデプロイする](https://azure.microsoft.com/documentation/samples/resource-manager-ruby-template-deployment/)


これらのサンプルだけでなく、ギャラリー サンプルを検索することもできます。

[.NET](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=dotnet) | [Java](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=java) | [Node.js](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=nodejs) | [Python](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=python) | [Ruby](https://azure.microsoft.com/documentation/samples/?service=azure-resource-manager&platform=ruby)

## 次のステップ

- テンプレートの簡単な操作手順については、「[既存のリソースから Azure Resource Manager テンプレートをエクスポートする](resource-manager-export-template.md)」を参照してください。
- さらに詳しくテンプレートの作成方法を解説したチュートリアルについては、「[Resource Manager テンプレートのチュートリアル](resource-manager-template-walkthrough.md)」を参照してください。
- テンプレートで使用できる関数については、[テンプレートの関数](resource-group-template-functions.md)に関するページを参照してください。
- Visual Studio と Resource Manager の使用については、「[Visual Studio での Azure リソース グループの作成とデプロイ](vs-azure-tools-resource-groups-deployment-projects-create-deploy.md)」を参照してください。
- Resource Manager での VS Code の使用については、「[Visual Studio Code で Azure Resource Manager テンプレートを操作する](resource-manager-vs-code.md)」を参照してください。

この概要のデモンストレーション ビデオを次に示します。

[AZURE.VIDEO azure-resource-manager-overview]

<!---HONumber=AcomDC_0921_2016-->