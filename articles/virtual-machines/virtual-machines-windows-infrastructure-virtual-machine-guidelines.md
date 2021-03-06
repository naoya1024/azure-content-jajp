<properties
	pageTitle="Windows 仮想マシンのガイドライン | Microsoft Azure"
	description="Azure への Windows 仮想マシンのデプロイに関する主な設計と実装のガイドラインについて説明します"
	documentationCenter=""
	services="virtual-machines-windows"
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/08/2016"
	ms.author="iainfou"/>

# 仮想マシンのガイドライン

[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-intro](../../includes/virtual-machines-windows-infrastructure-guidelines-intro.md)]

この記事は、ご使用の Azure 環境内で仮想マシン (VM) を作成および管理するために必要な計画について説明します。

## VM に関する実装ガイドライン
決めること:

- インフラストラクチャのさまざまなアプリケーション層とコンポーネントに対して、いくつのVMが必要か
- 各 VM が必要とする CPU とメモリ リソース、およびストレージの要件

タスク:

- アプリケーションのワークロードと、VM が必要とするリソースを定義する
- 各 VM に対するリソースの需要を、適切な VM サイズとストレージの種類により調整する
- インフラストラクチャのさまざまな階層とコンポーネントに対し、リソース グループを定義する
- VM の名前付け規則を定義する
- Azure PowerShell、Web ポータル、または Resource Manager テンプレートを使用して VM を作成する。

## 仮想マシン

Azure 環境内の主要なコンポーネントの 1 つは VM です。ここでアプリケーション、データベース、認証サービスなどを実行します。

パフォーマンスとコストの観点から環境のサイズを設定するには、[さまざまなサイズの VM](virtual-machines-windows-sizes.md) について理解しておくことが重要です。VM に十分な量のメモリや CPU コアがない場合、どれほど適切に設計、開発してもアプリケーションのパフォーマンスは低下します。インフラストラクチャの各コンポーネントに対して使用する VM のサイズを決定する際の出発点として、各 VM シリーズの推奨ワークロードを確認します。デプロイ後、[VM のサイズを変更](https://azure.microsoft.com/blog/resize-virtual-machines/)できます。

ストレージは、VM のパフォーマンスにおいて重要な役割を果たします。通常の回転ディスクを使った Standard Storage か、高い I/O ワークロードとピーク パフォーマンス用の SSD ディスクを使った Premium Storage を使用できます。VM サイズと同様に、ストレージ メディア選択についてはコストに関する考慮事項があります。VM の最適なパフォーマンス実現に適したストレージ設計の詳細については、「[Storage infrastructure guidelines (ストレージ インフラストラクチャのガイドライン)](virtual-machines-windows-infrastructure-storage-solutions-guidelines.md)」を参照してください。


## リソース グループ
[Azure リソース グループ](../resource-group-overview.md)を使った管理と保守を容易にするため、VM などのコンポーネントは論理的にグループ化されます。リソース グループを使用すると、特定のアプリケーションを構成するすべてのリソースを作成、管理、および監視することができます。また、チーム内の他者に必要なリソースのみへのアクセスを許可するため、[ロールベースのアクセス制御](../active-directory/role-based-access-control-what-is.md)を実装することもできます。リソース グループとロール割り当ての計画に時間をとってください。リソース グループの実際の設計と実装にはさまざまな方法があるため、[リソース グループのガイドラインに関する記事](virtual-machines-windows-infrastructure-resource-groups-guidelines.md)をお読みになり、最適な VM の設計を理解してください。


## テンプレート 
宣言型の JSON ファイルで定義されたテンプレートを構築して、VM を作成できます。テンプレートは通常、VM 自体とともに、必要となるストレージ、ネットワーク、ネットワーク インターフェイス、IP アドレス指定なども作成します。テンプレートを使って、一貫性があり再現可能な、開発とテストを目的とする環境を作成し、容易に運用環境をレプリケートすることができ、またその逆のこともできます。テンプレートを使った VM の作成とデプロイについての詳細は、[テンプレートの構築と使用](../resource-group-overview.md#template-deployment)についての記事を参照してください。


## 次のステップ
[AZURE.INCLUDE [virtual-machines-windows-infrastructure-guidelines-next-steps](../../includes/virtual-machines-windows-infrastructure-guidelines-next-steps.md)]

<!---HONumber=AcomDC_0914_2016-->