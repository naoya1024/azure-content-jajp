<properties
   pageTitle="Service Fabric のカスタム正常性レポートを追加する | Microsoft Azure"
   description="Azure Service Fabric の正常性エンティティにカスタム正常性レポートを送信する方法について説明します。品質正常性レポートの設計と実装の推奨事項を紹介します。"
   services="service-fabric"
   documentationCenter=".net"
   authors="oanapl"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/11/2016"
   ms.author="oanapl"/>

# Service Fabric のカスタム正常性レポートの追加
Azure Service Fabric では、特定のエンティティで、問題のあるクラスターおよびアプリケーションの条件にフラグを設定することを目的とする[正常性モデル](service-fabric-health-introduction.md)が導入されています。これは、**正常性レポーター** (システム コンポーネントおよびウォッチドッグ) を使用して実現されます。その目標は、簡単かつ迅速な診断および修復です。サービスの作成者は、正常性に関して事前に検討する必要があります。正常性に影響する可能性があるすべての条件は、特にルートに近い問題にフラグを設定するために役立つ場合に、レポートする必要があります。これにより、クラウド (プライベートまたは Azure) で、サービスが大規模に開始された後に、デバッグと調査にかかる時間と労力が大幅に削減されます。

Service Fabric レポーターは、識別された関心のある条件を監視します。このレポーターは、そのローカル ビューに基づいて、これらの条件をレポートします。[正常性ストア](service-fabric-health-introduction.md#Health-Store)は、すべてのレポーターによって送信された正常性データを集計して、エンティティがグローバルに正常な状態であるかどうかを判断します。このモデルは、機能豊富、柔軟で、使いやすいように設計されています。正常性レポートの品質によって、クラスターの正常性ビューがどの程度正確かが決定されます。正常性の問題が誤って示される誤検知があると、正常性データを使用するアップグレードや他のサービスに悪影響が及びます。たとえば、修理サービスやアラート メカニズムなどのサービスです。したがって、最適な方法で、関心のある条件をキャプチャするレポートを提供するには、ある程度の配慮が必要です。

正常性レポートを設計し、実装するには、ウォッチドックおよびシステム コンポーネントで、以下のことが必要です。

- 関心のある条件、監視する方法、およびクラスターまたはアプリケーション機能への影響を定義します。これにより、正常性レポートのプロパティと正常性状態を定義します。

- レポートの適用先の[エンティティ](service-fabric-health-introduction.md#health-entities-and-hierarchy)を決定します。

- レポートの作成元 (サービス内、内部または外部のウォッチドッグのいずれか) を決定します。

- レポーターを識別するために使用するソースを定義します。

- 定期的に、または遷移時に、レポート戦略を選択します。必要なコードが単純で、エラーが生じにくいため、定期的に行うことを推奨します。

- 異常状態のレポートを正常性ストアに保存する時間と消去する方法を決定します。これにより、レポートの有効期限と、有効期限切れ時の削除動作を定義します。

前述のように、レポートは、次のいずれかから作成できます。

- 監視対象の Service Fabric のサービス レプリカ。

- Service Fabric のサービスとしてデプロイされる内部ウォッチドッグ (条件および問題のレポートを監視する Service Fabric ステートレス サービスなど)。ウォッチドッグは、すべてのノードにデプロイするか、監視対象のサービスに関連付けることができます。

- Service Fabric のノードで実行されているが、Service Fabric のサービスとして実装されていない内部ウォッチドッグ。

- Service Fabric クラスター以外のリソースを調査する外部ウォッチドッグ (Gomez のような監視サービスなど)。

> [AZURE.NOTE] 既定では、クラスターには、システム コンポーネントによって送信される正常性レポートが事前設定されます。「[トラブルシューティングのためのシステム正常性レポートの使用](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)」を参照してください。ユーザー レポートは、システムによって作成済みの[正常性エンティティ](service-fabric-health-introduction.md#health-entities-and-hierarchy)に関して送信する必要があります。

正常性レポートの設計を決定したら、正常性レポートを簡単に送信できます。クラスターが[セキュリティで保護](service-fabric-cluster-security.md)されていない場合や、ファブリック クライアントに管理者特権がある場合は、`FabricClient` を使用して正常性をレポートできます。レポートの送信には、PowerShell または REST で API を介して [FabricClient.HealthManager.ReportHealth](https://msdn.microsoft.com/library/system.fabric.fabricclient.healthclient.reporthealth.aspx) を使用します。バッチ レポートで、パフォーマンスを向上することもできます。

> [AZURE.NOTE] 正常性レポートは同期的であり、クライアント側の検証作業のみを表します。レポートが正常性クライアントあるいは `Partition` または `CodePackageActivationContext` オブジェクトによって受け入れられても、そのレポートがストアに適用されるとは限りません。正常性レポートは非同期に送信されます。また、場合によっては他のレポートと一括して送信される可能性があります。サーバー上の処理が失敗する場合もあります (シーケンス番号が古い、レポートが適用されるエンティティが削除されているなど)。

## 正常性クライアント
正常性レポートは、ファブリック クライアント内で動作している正常性クライアントを使用して、正常性ストアに送信されます。正常性クライアントは、次の項目で構成できます。

- **HealthReportSendInterval**: レポートがクライアントに追加される時間から、正常性ストアに送信される時間までの遅延。1 つのレポートを 1 つのメッセージとして送信するのではなく、複数のレポートを 1 つのメッセージにまとめて送信するために使用されます。このパラメーターを使用すると、パフォーマンスが向上します。既定値: 30 秒。

- **HealthReportRetrySendInterval**: 正常性クライアントが累積した正常性レポートを正常性ストアに再送信する間隔。既定値: 30 秒。

- **HealthOperationTimeout**: 正常性ストアに送信されたレポート メッセージのタイムアウト時間。メッセージがタイムアウトした場合、正常性クライアントは、正常性ストアによってレポートが処理されたことが確認されるまで再試行します。既定値: 2 分。

> [AZURE.NOTE] レポートがバッチ処理される場合、ファブリック クライアントは、レポートが確実に送信されるように、少なくとも HealthReportSendInterval の間、動作している必要があります。メッセージが失われた場合、または正常性ストアが一時的なエラーのためにメッセージを適用できない場合、ファブリック クライアントは、再試行の機会を与えるために、さらに長い時間動作させる必要があります。

クライアントでのバッファリングでは、レポートの一意性を考慮に入れます。たとえば、特定の不良なレポーターが、同じエンティティの同じプロパティに関して、1 秒あたり 100 のレポートを発行している場合、レポートは直前のバージョンに置き換えられます。クライアント キューには、このようなレポートが 1 つのみ存在します。バッチ処理を構成すると、複数のレポートは 1 つにまとめられ、送信間隔ごとに 1 回、正常性ストアに送信されます。エンティティの最新の状態が反映された、最後に追加されたレポートが送信されます。すべての構成パラメーターを指定するには、`FabricClient` を作成するときに、正常性に関連するエントリの望ましい値を指定した [FabricClientSettings](https://msdn.microsoft.com/library/azure/system.fabric.fabricclientsettings.aspx) を渡します。

次に、ファブリック クライアントを作成し、追加された直後にレポートを送信することを指定します。タイムアウトや、再試行可能なエラーが発生した場合、40 秒間隔で再試行されます。

```csharp
var clientSettings = new FabricClientSettings()
{
    HealthOperationTimeout = TimeSpan.FromSeconds(120),
    HealthReportSendInterval = TimeSpan.FromSeconds(0),
    HealthReportRetrySendInterval = TimeSpan.FromSeconds(40),
};
var fabricClient = new FabricClient(clientSettings);
```

PowerShell を使用して、クラスターへの接続を確立するときに、同じパラメーターを指定できます。次に、ローカル クラスターへの接続を開始します。

```powershell
PS C:\> Connect-ServiceFabricCluster -HealthOperationTimeoutInSec 120 -HealthReportSendIntervalInSec 0 -HealthReportRetrySendIntervalInSec 40
True

ConnectionEndpoint   :
FabricClientSettings : {
                       ClientFriendlyName                   : PowerShell-1944858a-4c6d-465f-89c7-9021c12ac0bb
                       PartitionLocationCacheLimit          : 100000
                       PartitionLocationCacheBucketCount    : 1024
                       ServiceChangePollInterval            : 00:02:00
                       ConnectionInitializationTimeout      : 00:00:02
                       KeepAliveInterval                    : 00:00:20
                       HealthOperationTimeout               : 00:02:00
                       HealthReportSendInterval             : 00:00:00
                       HealthReportRetrySendInterval        : 00:00:40
                       NotificationGatewayConnectionTimeout : 00:00:00
                       NotificationCacheUpdateTimeout       : 00:00:00
                       }
GatewayInformation   : {
                       NodeAddress                          : localhost:19000
                       NodeId                               : 1880ec88a3187766a6da323399721f53
                       NodeInstanceId                       : 130729063464981219
                       NodeName                             : Node.1
                       }
```

> [AZURE.NOTE] 承認されていないサービスが、クラスター内のエンティティに対して正常性をレポートできないように、セキュリティで保護されたクライアントからの要求のみを受け入れるように、サーバーを構成できます。レポートは `FabricClient` を通じて実行されるため、`FabricClient` では、クラスターと通信するために (Kerberos や証明書の認証などによって) セキュリティが有効になっている必要があります。詳細については、[クラスター セキュリティ](service-fabric-cluster-security.md)に関する記述を参照してください。

## 低特権サービス内からのレポート
クラスターに対する管理アクセス権がない Service Fabric サービス内から、`Partition` または `CodePackageActivationContext` を使用して現在のコンテキストのエンティティに関する正常性をレポートすることができます。

- ステートレス サービスの場合は、[IStatelessServicePartition.ReportInstanceHealth](https://msdn.microsoft.com/library/system.fabric.istatelessservicepartition.reportinstancehealth.aspx) を使用して現在のサービス インスタンスについてレポートします。

- ステートフル サービス場合は、[IStatefulServicePartition.ReportReplicaHealth](https://msdn.microsoft.com/library/system.fabric.istatefulservicepartition.reportreplicahealth.aspx) を使用して現在のレプリカについてレポートします。

- 現在のパーティション エンティティについてレポートするには、[IServicePartition.ReportPartitionHealth](https://msdn.microsoft.com//library/system.fabric.iservicepartition.reportpartitionhealth.aspx) を使用します。

- 現在のアプリケーションについてレポートするには、[CodePackageActivationContext.ReportApplicationHealth](https://msdn.microsoft.com/library/system.fabric.codepackageactivationcontext.reportapplicationhealth.aspx) を使用します。

- 現在のノード上にデプロイされている現在のアプリケーションについてレポートするには、[CodePackageActivationContext.ReportDeployedApplicationHealth](https://msdn.microsoft.com/library/system.fabric.codepackageactivationcontext.reportdeployedapplicationhealth.aspx) を使用します。

- 現在のノード上にデプロイされている現在のアプリケーションのサービス パッケージについてレポートするには、[CodePackageActivationContext.ReportDeployedServicePackageHealth](https://msdn.microsoft.com/library/system.fabric.codepackageactivationcontext.reportdeployedservicepackagehealth.aspx) を使用します。

> [AZURE.NOTE] 内部的には、`Partition` と `CodePackageActivationContext` は、既定の設定で構成される正常性クライアントを保持します。[正常性クライアント](service-fabric-report-health.md#health-client)と同じ考慮事項が適用されます。つまり、レポートはバッチ処理されてからタイマーに従って送信されるため、レポートを送信できるようにオブジェクトを保持する必要があります。

## 正常性レポートの設計
高品質のレポートを生成するための最初の手順では、サービスの正常性に影響する可能性がある条件を特定します。開始時、またはできれば問題の発生前に、サービスまたはクラスターで問題にフラグを付けるために役立つことがある条件によって、数十億ドルのコストが削減される可能性があります。メリットとして、ダウン タイムの短縮、問題の調査と修復にかかる夜間の時間の短縮、顧客満足度の向上などがあります。

条件を識別したら、ウォッチドッグの作成者は、オーバーヘッドと有用性のバランスのために、条件を監視する最適な方法を見つける必要があります。たとえば、共有上の一部の一時ファイルを使用して複雑な計算を行うサービスがあるとします。ウォッチドッグは、共有を監視して、十分な領域が使用できるかどうかを確認できます。また、ファイルまたはディレクトリの変更に関する通知をリッスンできます。事前のしきい値に達した場合に警告をレポートし、共有に空き領域がなくなった場合にエラーをレポートすることもできます。警告の場合、修復システムは、共有上の古いファイルのクリーンアップを開始できます。エラーの場合、修復システムは、別のノードにサービスのレプリカを移動できます。正常性に関して、条件の状態 (正常または異常 (警告またはエラー) と見なすことができる条件の状態) を記述する方法に注意してください。

監視の詳細を設定したら、ウォッチドッグの作成者は、ウォッチドッグを実装する方法を決定する必要があります。サービス内から条件を決定できる場合、ウォッチドッグは、監視対象のサービス自体の一部にすることができます。たとえば、サービス コードは、ファイルを作成しようとするたびに、共有の使用状況を確認し、ローカルのファブリック クライアントを使用してレポートできます。この方法の利点は、レポートが簡単なことです。ウォッチドッグのバグが、サービスの機能に影響を与えないように、注意する必要があります。

監視対象のサービス内からのレポートは、常に適切なオプションであるとは限りません。原因として、サービス内のウォッチドッグで条件を検出できない場合、判断するロジックやデータがない場合、条件を監視するオーバーヘッドが高い場合、また、サービスに固有の条件ではなく、サービス間のやり取りに影響がある条件の場合があります。もう 1 つのオプションは、別のプロセスとして、クラスター内にウォッチドッグを用意することです。ウォッチドッグは、主なサービスにいかなる意味でも影響を与えずに、条件とレポートを単純に監視します。たとえば、このようなウォッチドッグは、すべてのノードに、またはサービスと同じノードにデプロイされる同じアプリケーションで、ステートレス サービスとして実装できます。

クラスター内で実行されるウォッチドッグも、適切なオプションではないことがあります。監視される条件が、ユーザーに表示されるサービスの可用性と機能である場合、ユーザー クライアントと同じ場所にウォッチドッグをデプロイすることをお勧めします。ローカルなので、ユーザーが操作を呼び出す場合と同じ方法で操作をテストできます。たとえば、クラスター以外の場所にウォッチドッグをデプロイし、要求をサービスに発行して、結果の待機時間と正確性を確認することができます(たとえば、計算機サービスの場合、2+2 の答え 4 が適切な時間で返されるかどうかを確認します)。

ウォッチドッグの詳細を決めたら、ウォッチドッグを一意に識別するソース ID を決定します。同じ種類の複数のウォッチドッグがクラスター内で動作している場合、これらのウォッチドッグは、異なるエンティティについてレポートする必要があります。同じエンティティについてレポートする場合は、ソース ID またはプロパティが異なることを確認します。こうすることで、複数のレポートが共存できます。正常性レポートのプロパティでは、監視対象の条件をキャプチャする必要があります。たとえば、上記の例で、プロパティは **ShareSize** にすることができます。 複数のレポートが同じ条件に適用される場合は、レポートが共存できるように、プロパティに動的な情報を含める必要があります。たとえば、監視する必要がある複数の共有がある場合、プロパティ名は、**ShareSize-sharename** にすることができます。

> [AZURE.NOTE] 正常性ストアは、状態情報を保持するために使用*しない*でください。正常性に関連する情報のみを、エンティティの正常性評価に影響を与える正常性情報としてレポートする必要があります。正常性ストアは、汎用目的のストアとして設計されていません。これは、すべてのデータを正常性状態に集計するために、正常性評価のロジックを使用します。正常性に関係しない情報 (正常性状態に問題のないレポート状態) を送信することによって、集約される正常性状態に影響は生じませんが、状態ストアのパフォーマンスに悪影響が生じることがあります。

次に、どのようなエンティティについてレポートするかを決定します。ほとんどの場合、これは条件に基づいて自明です。可能な限り詳細にエンティティを選択する必要があります。条件がパーティション内のすべてのレプリカに影響する場合は、サービスではなく、パーティションについてレポートします。さらに検討が必要な、まれなケースがあります。条件がエンティティ (レプリカなど) に影響するが、レプリカの有効期間よりも長く条件にフラグを付ける必要がある場合は、パーティションについてレポートする必要があります。そうしなければ、レプリカを削除すると、関連付けられているすべてのレポートは、ストアからクリーンアップされます。つまり、ウォッチドッグを作成する場合、エンティティとレポートの有効期間についても考慮する必要があります。また、レポートをストアからクリーンアップする時期を明確にする必要があります (エンティティでレポートされたエラーが該当しなくなった場合など)。

このようなすべての点についてまとめた例を見てみましょう。マスター ステートフル永続サービスと、すべてのノードにデプロイされたセカンダリ ステートレス サービス (タスクの種類ごとに 1 つのセカンダリ サービス) で構成される Service Fabric アプリケーションについて考えてみます。マスターには、セカンダリによって実行されるコマンドを格納する処理キューがあります。セカンダリは、受信要求を実行し、受信確認信号を返信します。監視できる 1 つの条件は、マスターの処理キューの長さです。マスターのキューの長さがしきい値に達すると、警告がレポートされます。これは、セカンダリが負荷を処理できないことを示します。キューが最大長に達して、コマンドが格納されなくなると、サービスが復旧できなくなり、エラーがレポートされます。**QueueStatus** プロパティについてレポートすることができます。ウォッチドッグはサービス内で動作し、マスター プライマリ レプリカに関して定期的に送信されます。有効期間は 2 分間です。30 秒間隔で定期的に送信されます。プライマリがダウンした場合、レポートはストアから自動的にクリーンアップされます。サービス レプリカが動作している状態でデッドロックまたはその他の問題がある場合、正常性ストアのレポートは有効期限が切れます。この場合は、エンティティはエラーと評価されます。

監視可能なその他の条件は、タスクの実行時間です。マスターは、タスクの種類に基づいて、タスクをセカンダリに配布します。設計によっては、マスターがタスクの状態についてセカンダリをポーリングすることができます。また、処理の完了時にセカンダリから受信確認信号が返信されるまで待機することもできます。この場合、セカンダリの異常終了やメッセージが失われた状況を検出するよう注意する必要があります。1 つの方法として、マスターが同じセカンダリに ping 要求を送信し、セカンダリがその状態を返信します。マスターに状態が返信されない場合、マスターはエラーと認識と、タスクのスケジュールを変更します。これは、タスクが独立していることを前提としています。

タスクが特定の時間 **t1** (10 分など) に実行されない場合は、監視対象の状態を警告とし、時間 **t2** (20 分など) でタスクが完了しない場合はエラーとすることができます。このレポートは、複数の方法で行うことができます。

- マスター プライマリ レプリカは、自身の状態に関して定期的にレポートします。キュー内にあるすべての保留中のタスクについて、1 つのプロパティを指定できます。1 つ以上のタスクの処理に時間がかかると、プロパティ **PendingTasks** のレポートの状態は、状況応じて警告またはエラーになります。保留中のタスクがないか、すべてが開始直後の場合、レポートの状態は OK です。タスクは継続されるため、プライマリがダウンした場合、新しい昇格されたプライマリが正しくレポートを続行できます。

- (クラウドまたは外部の) 別のウォッチドッグ プロセスは、(望ましいタスクの結果に基づいて外部から) タスクをチェックし、完了したかどうかを確認します。しきい値が反映されない場合、マスター サービスに関するレポートが送信されます。また、各タスクについて、タスク識別子を含むレポートも送信されます (例: **PendingTask+taskid**)。レポートは、正常ではない状態の場合にのみ送信されます。確実にレポートをクリーンアップするために、有効期間を数分に設定し、期限が切れたときに削除されるようにレポートをマークします。

- タスクを実行中のセカンダリは、実行時間が予想よりも長いかどうかをレポートします。プロパティ **PendingTasks** のサービス インスタンスに関してレポートされます。これは、問題のあるサービス インスタンスを示しますが、インスタンスが異常終了した状況をキャプチャしません。レポートは、その時点でクリーンアップされます。セカンダリ サービスについてレポートすることもできます。セカンダリでタスクが完了すると、セカンダリ インスタンスによってストアからレポートが消去されます。これは、確認応答メッセージが失われた状況をキャプチャせず、タスクは、マスターの観点では終了しません。

ただし、上記のケースではレポートが作成され、この状況は、正常性を評価する場合にアプリケーションの正常性に関してキャプチャされます。

## 定期的なレポートと遷移時のレポート
正常性レポート モデルを使用して、ウォッチドッグは、定期的に、または遷移時にレポートを送信できます。コードがはるかに単純で、エラーが生じにくいため、ウォッチドッグ レポートは定期的に行うことをお勧めします。ウォッチドッグは、正しくないレポートを生成させるバグを回避するために、できるだけ簡単にする必要があります。正しくない異常レポートは、正常性評価と、アップグレードなどの正常性に基づくシナリオに影響します。正しくない正常性レポートにより、望ましくないクラスター内の問題が隠されます。

定期的なレポートの場合、ウォッチドッグは、タイマーと共に実装できます。タイマーのコールバック時に、ウォッチドッグは状態を確認し、現在の状態に基づいて、レポートを送信できます。以前にどのようなレポートが送信されたかを確認したり、メッセージングに関する最適化を行う必要はありません。正常性クライアントには、これを支援するバッチ処理ロジックがあります。正常性クライアントは、動作している限り、レポートが正常性ストアによって確認応答されるか、ウォッチドッグが同じエンティティ、プロパティ、およびソースで新しいレポートを生成するまで、内部で再試行します。

遷移に関するレポートの場合、気を付けて状態を処理する必要があります。ウォッチドッグは、いくつかの条件を監視し、条件が変化した場合にのみレポートします。このアプローチの利点は、必要なレポート数が少ない点です。欠点は、ウォッチドッグのロジックが複雑なことです。状態の変化を判断するために調べることができるように、条件またはレポートも維持する必要があります。フェールオーバー時に、まだ送信されていない可能性があるレポートを送信することに注意する必要があります (キューに登録されたが、正常性ストアに送信されていないレポート)。シーケンス番号は増え続けるように指定する必要があります。そうしないと、古いレポートと判断され、拒否されます。データの損失が発生するまれなケースで、レポーターの状態と正常性ストアの状態間で、同期が必要になることがあります。

遷移に関するレポートは、`Partition` または `CodePackageActivationContext` を使用して、サービスでそれ自体についてレポートする場合に適しています。ローカル オブジェクト (レプリカまたはデプロイされたサービス パッケージ/デプロイされたアプリケーション) が削除されると、そのレポートもすべて削除されます。これにより、レポーターと正常性ストア間の同期の必要性が緩和されます。親パーティションまたは親アプリケーションに関するレポートの場合、正常性ストアの古いレポートを回避するフェールオーバーに注意する必要があります。正しい状態を維持し、必要なくなった場合にストアからレポートを消去するには、ロジックを追加する必要があります。

## 正常性レポートの実装
エンティティとレポートの詳細を明確にしたら、API、PowerShell または REST を使用して、正常性レポートを送信できます。

### API
API を使用してレポートするには、レポートする必要があるエンティティの種類に固有な正常性レポートを作成する必要があります。次に、正常性クライアントにレポートを送信します。あるいは、正常性の情報を作成し、それを `Partition` または `CodePackageActivationContext` を使用して適切なレポート メソッドに渡して現在のエンティティについてレポートする必要があります。

次の例は、クラスター内のウォッチドッグからの定期的なレポートを示していますウォッチドッグは、外部のリソースにノード内からアクセスできるかどうかを確認します。リソースは、アプリケーション内のサービス マニフェストで必要です。リソースを使用できない場合でも、アプリケーション内の他のサービスが正しく機能することがあります。したがって、レポートは、デプロイ済みのサービス パッケージのエンティティに、30 秒ごとに送信されます。

```csharp
private static Uri ApplicationName = new Uri("fabric:/WordCount");
private static string ServiceManifestName = "WordCount.Service";
private static string NodeName = FabricRuntime.GetNodeContext().NodeName;
private static Timer ReportTimer = new Timer(new TimerCallback(SendReport), null, 30 * 1000, 30 * 1000);
private static FabricClient Client = new FabricClient(new FabricClientSettings() { HealthReportSendInterval = TimeSpan.FromSeconds(0) });

public static void SendReport(object obj)
{
    // Test whether the resource can be accessed from the node
    HealthState healthState = this.TestConnectivityToExternalResource();

    // Send report on deployed service package, as the connectivity is needed by the specific service manifest
    // and can be different on different nodes
    var deployedServicePackageHealthReport = new DeployedServicePackageHealthReport(
        ApplicationName,
        ServiceManifestName,
        NodeName,
        new HealthInformation("ExternalSourceWatcher", "Connectivity", healthState));

    // TODO: handle exception. Code omitted for snippet brevity.
    // Possible exceptions: FabricException with error codes
    // FabricHealthStaleReport (non-retryable, the report is already queued on the health client),
    // FabricHealthMaxReportsReached (retryable; user should retry with exponential delay until the report is accepted).
    Client.HealthManager.ReportHealth(deployedServicePackageHealthReport);
}
```

### PowerShell
ユーザーは、**Send-ServiceFabric*EntityType*HealthReport** を使用して正常性レポートを送信できます。

次の例では、ノードの CPU の値に関する定期的なレポートを示しています。レポートは 30 秒間隔で送信されます。有効期間は 2 分間です。有効期限が切れた場合は、レポーターに問題があるため、ノードがエラーとして評価されます。CPU がしきい値を超えると、レポートの正常性状態は警告になります。CPU がしきい値を超えた状態のまま、構成した時間を過ぎると、エラーとレポートされます。それ以外の場合、OK の正常性状態が送信されます。

```powershell
PS C:\> Send-ServiceFabricNodeHealthReport -NodeName Node.1 -HealthState Warning -SourceId PowershellWatcher -HealthProperty CPU -Description "CPU is above 80% threshold" -TimeToLiveSec 120

PS C:\> Get-ServiceFabricNodeHealth -NodeName Node.1
NodeName              : Node.1
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='CPU', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 5
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : CPU
                        HealthState           : Warning
                        SequenceNumber        : 130741236814913394
                        SentAt                : 4/21/2015 9:01:21 PM
                        ReceivedAt            : 4/21/2015 9:01:21 PM
                        TTL                   : 00:02:00
                        Description           : CPU is above 80% threshold
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:01:21 PM
```

次の例は、レプリカに関して一時的な警告をレポートします。まずパーティション ID を取得し、次にそれに関連するサービスのレプリカ ID を取得します。次にプロパティ **ResourceDependency** で **PowershellWatcher** からレポートを送信します。レポートの有効期間はわずか 2 分間で、自動的にストアから削除されます。

```powershell
PS C:\> $partitionId = (Get-ServiceFabricPartition -ServiceName fabric:/WordCount/WordCount.Service).PartitionId

PS C:\> $replicaId = (Get-ServiceFabricReplica -PartitionId $partitionId | where {$_.ReplicaRole -eq "Primary"}).ReplicaId

PS C:\> Send-ServiceFabricReplicaHealthReport -PartitionId $partitionId -ReplicaId $replicaId -HealthState Warning -SourceId PowershellWatcher -HealthProperty ResourceDependency -Description "The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes." -TimeToLiveSec 120 -RemoveWhenExpired

PS C:\> Get-ServiceFabricReplicaHealth  -PartitionId $partitionId -ReplicaOrInstanceId $replicaId


PartitionId           : 8f82daff-eb68-4fd9-b631-7a37629e08c0
ReplicaId             : 130740415594605869
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='PowershellWatcher', Property='ResourceDependency', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 130740768777734943
                        SentAt                : 4/21/2015 8:01:17 AM
                        ReceivedAt            : 4/21/2015 8:02:12 AM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : ->Ok = 4/21/2015 8:02:12 AM

                        SourceId              : PowershellWatcher
                        Property              : ResourceDependency
                        HealthState           : Warning
                        SequenceNumber        : 130741243777723555
                        SentAt                : 4/21/2015 9:12:57 PM
                        ReceivedAt            : 4/21/2015 9:12:57 PM
                        TTL                   : 00:02:00
                        Description           : The external resource that the primary is using has been rebooted at 4/21/2015 9:01:21 PM. Expect processing delays for a few minutes.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : ->Warning = 4/21/2015 9:12:32 PM
```

## 次のステップ

正常性データに基づいて、サービスの作成者とクラスター/アプリケーション管理者は、情報の使用方法を検討できます。たとえば、障害が引き起こされる前に重大な問題を把握するために、正常性状態に基づいてアラートを設定できます。管理者は、問題を自動的に解決する修復システムを設定することもできます。

[Service Fabric の正常性モニタリングの概要](service-fabric-health-introduction.md)

[Service Fabric の正常性レポートの確認](service-fabric-view-entities-aggregated-health.md)

[サービス正常性のレポートとチェックの方法](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

[トラブルシューティングのためのシステム正常性レポートの使用](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[ローカルでのサービスの監視と診断](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Service Fabric アプリケーションのアップグレード](service-fabric-application-upgrade.md)

<!---HONumber=AcomDC_0713_2016-->