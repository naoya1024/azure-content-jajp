<properties
  pageTitle="Azure Backup Server を使用してワークロードをバックアップする環境の準備方法 | Microsoft Azure"
  description="Azure Backup Server を使用して、ワークロードをバックアップする環境を適切に準備します"
  services="backup"
  documentationCenter=""
  authors="pvrk"
  manager="shivamg"
  editor=""
  keywords="Azure Backup Server, バックアップ コンテナー"/>

<tags
  ms.service="backup"
  ms.workload="storage-backup-recovery"
  ms.tgt_pltfrm="na"
  ms.devlang="na"
  ms.topic="article"
  ms.date="08/22/2016"
  ms.author="jimpark;trinadhk;pullabhk; markgal"/>

# Azure Backup Server を使用してワークロードをバックアップするための準備

> [AZURE.SELECTOR]
- [Azure Backup Server](backup-azure-microsoft-azure-backup.md)
- [SCDPM](backup-azure-dpm-introduction.md)
- [Azure Backup Server (クラシック)](backup-azure-microsoft-azure-backup-classic.md)
- [SCDPM (クラシック)](backup-azure-dpm-introduction-classic.md)


この記事では、Azure Backup Server を使用してワークロードをバックアップする環境の準備方法について説明します。Azure Backup Server を使用すると、単一のコンソールから Hyper-V VM、Microsoft SQL Server、SharePoint Server、Microsoft Exchange、Windows クライアントなどのアプリケーションのワークロードを保護することができます。

>[AZURE.WARNING] Azure Backup Server は、Data Protection Manager (DPM) のワークロード バックアップの機能を継承しています。これらの機能の一部については、DPM ドキュメントへのポインターがあります。ただし、Azure Backup Server は、テープ上の保護や System Center との統合の機能は提供していません。

## 1\.Windows Server マシン

![step1](./media/backup-azure-microsoft-azure-backup/step1.png)

Azure Backup Server を準備して実行するための最初の手順は、Windows Server マシンを用意することです。

| Location (場所) | 最小要件 | 追加説明 |
| -------- | -------------------- | ----------------------- |
| Azure | Azure IaaS 仮想マシン<br><br>A2 Standard: 2 コア、3.5 GB RAM | 手始めに、Windows Server 2012 R2 Datacenter の単純なギャラリー イメージを使用することができます。[Azure Backup Server (DPM) を使用した IaaS ワークロードの保護](https://technet.microsoft.com/library/jj852163.aspx)には、数多くの注意点があります。マシンのデプロイ前に、必ずこの記事によく目を通してください。 |
| オンプレミスの | Hyper-V、<br> VMWare VM、<br>または物理ホスト<br><br>2 コア、4 GB RAM | Windows Server の重複除去を使用して DPM ストレージの重複を除去することができます。Hyper-V VM にデプロイするときは、[DPM と重複除去](https://technet.microsoft.com/library/dn891438.aspx)が連携するしくみの詳細を確認してください。 |

> [AZURE.NOTE] Azure Backup Server は、Windows Server 2012 R2 Datacenter を設定済みのマシンにインストールすることをお勧めします。前提条件の多くは、最新バージョンの Windows オペレーティング システムを使用すると自動的に満たされます。

このサーバーをドメインに参加させる予定がある場合は、Azure Backup Server をインストールする前にドメインへの参加作業を完了することをお勧めします。デプロイ後の、新しいドメインへの既存の Azure Backup Server マシンの移動は*サポートされていません*。

## 2\.バックアップ資格情報コンテナー

![step2](./media/backup-azure-microsoft-azure-backup/step2.png)

バックアップ データを Azure に送信する場合でも、ローカルに保存する場合でも、ソフトウェアを Azure に接続する必要があります。具体的には、Azure Backup Server マシンをバックアップ資格情報コンテナーに登録する必要があります。

バックアップ資格情報コンテナーを作成するには:

1. [管理ポータル](http://manage.windowsazure.com/)にサインインします。

2. **[新規]** > **[Data Services]** > **[Recovery Services]** > **[バックアップ資格情報コンテナー]** > **[簡易作成]** の順にクリックします。組織のアカウントに複数のサブスクリプションが関連付けられている場合は、バックアップ コンテナーに関連付ける適切なサブスクリプションを選択してください。

3. **[名前]** ボックスに、コンテナーを識別する表示名を入力します。これは、サブスクリプションごとに一意である必要があります。

4. **[リージョン]** ボックスで、コンテナーのリージョンを選択します。通常、資格情報コンテナーのリージョンは、データの主権またはネットワーク待ち時間の制約に応じて選択します。

    ![バックアップ資格情報コンテナーの作成](./media/backup-azure-microsoft-azure-backup/backup_vaultcreate.png)

5. **[資格情報コンテナーの作成]** をクリックします。バックアップ資格情報コンテナーが作成されるまで時間がかかることがあります。ポータルの下部にある状態通知を監視します。

    ![資格情報コンテナーのトースト通知の作成](./media/backup-azure-microsoft-azure-backup/creating-vault.png)

6. 資格情報コンテナーが正常に作成されたことを示すメッセージが表示され、[復旧サービス] ページに [アクティブ] と表示されます。![バックアップ資格情報コンテナーの一覧](./media/backup-azure-microsoft-azure-backup/backup_vaultslist.png)

  > [AZURE.IMPORTANT] コンテナーを作成したら、適切なストレージの冗長オプションが選択されていることを確認してください。この[概要](../storage/storage-redundancy.md)に記載されている [geo 冗長](../storage/storage-redundancy.md#geo-redundant-storage)オプションと[ローカル冗長](../storage/storage-redundancy.md#locally-redundant-storage)オプションの詳細をご覧ください。


## 3\.ソフトウェア パッケージ

![step3](./media/backup-azure-microsoft-azure-backup/step3.png)

### ソフトウェア パッケージのダウンロード

コンテナー資格情報と同様に、アプリケーション ワークロード用の Microsoft Azure Backup をバックアップ コンテナーの**クイック スタート ページ**からダウンロードすることができます。

1. **[アプリケーション ワークロード用 (Disk to Disk to Cloud)]** をクリックします。ソフトウェア パッケージをダウンロード可能なダウンロード センターのページが表示されます。

    ![Microsoft Azure Backup Welcome Screen](./media/backup-azure-microsoft-azure-backup/dpm-venus1.png)

2. **[Download]** をクリックします。

    ![Download center 1](./media/backup-azure-microsoft-azure-backup/downloadcenter1.png)

3. すべてのファイルを選択し、**[次へ]** をクリックします。Microsoft Azure Backup のダウンロード ページからすべてのファイルをダウンロードし、すべてのファイルを同じフォルダーに配置します。![Download center 1](./media/backup-azure-microsoft-azure-backup/downloadcenter.png)

    すべてのファイルをまとめてダウンロードするとサイズが 3 G を超えるため、10 Mbps のダウンロード リンクでは、ダウンロードが完了するまでに最大で 60 分かかることがあります。


### ソフトウェア パッケージの抽出

すべてのファイルをダウンロードしたら、**MicrosoftAzureBackupInstaller.exe** をクリックします。**Microsoft Azure Backup セットアップ ウィザード**が開始され、指定した場所にセットアップ ファイルが抽出されます。ウィザードの手順を続行し、**[抽出]** ボタンをクリックして抽出プロセスを開始します。

> [AZURE.WARNING] セットアップ ファイルを抽出するには、少なくとも 4 GB の空き領域が必要です。


![Microsoft Azure Backup Setup Wizard](./media/backup-azure-microsoft-azure-backup/extract/03.png)

抽出プロセスが完了したら、Microsoft Azure Backup Server のインストールを開始するために、抽出された *setup.exe* を起動するチェック ボックスをオンにし、**[完了]** をクリックします。

### ソフトウェア パッケージのインストール

1. **[Microsoft Azure Backup]** をクリックしてセットアップ ウィザードを起動します。

    ![Microsoft Azure Backup Setup Wizard](./media/backup-azure-microsoft-azure-backup/launch-screen2.png)

2. ようこそ画面で **[次へ]** をクリックします。*[前提条件の確認]* セクションが表示されます。この画面で **[確認]** をクリックして、Azure Backup Server のハードウェアとソフトウェアの前提条件が満たされているかどうかを確認します。前提条件がすべて正常に満たされている場合は、マシンが要件を満たしていることを示すメッセージが表示されます。**[次へ]** をクリックします。

    ![Azure Backup Server - Welcome and Prerequisites check](./media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)

3. Microsoft Azure Backup Server には SQL Server Standard が必要であり、Azure Backup Server のインストール パッケージには、適切な SQL Server バイナリがバンドルされています。新しい Azure Backup Server のインストールを開始するには、**[このセットアップを使用して、SQL Server の新しいインスタンスをインストールします]** をオンにし、**[確認してインストール]** をクリックします。前提条件が正常にインストールされたら、**[次へ]** をクリックします。

    ![Azure Backup Server - SQL check](./media/backup-azure-microsoft-azure-backup/sql/01.png)

    エラーが発生し、コンピューターの再起動を推奨された場合は、コンピューターを再起動して **[再確認]** をクリックします。

    > [AZURE.NOTE] Azure Backup Server は、リモートの SQL Server インスタンスでは動作しません。Azure Backup Server に使用されるインスタンスは、ローカルに存在する必要があります。

4. Microsoft Azure Backup サーバーのファイルをインストールする場所を指定し、**[次へ]** をクリックします。

    ![Microsoft Azure Backup PreReq2](./media/backup-azure-microsoft-azure-backup/space-screen.png)

    スクラッチ場所は、Azure へのバックアップの要件です。スクラッチ場所が、クラウドにバックアップする予定のデータの 5% 以上であることを確認します。ディスクを保護するために、インストールが完了した後で個別のディスクを構成する必要があります。記憶域プールの詳細については、「[記憶域プールおよびディスク記憶域の構成](https://technet.microsoft.com/library/hh758075.aspx)」を参照してください。

5. 制限付きのローカル ユーザー アカウント用に強力なパスワードを指定し、**[次へ]** をクリックします。

    ![Microsoft Azure Backup PreReq2](./media/backup-azure-microsoft-azure-backup/security-screen.png)

6. *Microsoft Update* を使用するかどうかを選択し、更新プログラムを確認して **[次へ]** をクリックします。

    >[AZURE.NOTE] Windows Update を Microsoft Update にリダイレクトすることをお勧めします。Microsoft Update は、Windows と Microsoft Azure Backup Server などのその他の製品のセキュリティ更新プログラムと重要な更新プログラムを提供しています。

    ![Microsoft Azure Backup PreReq2](./media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)

7. *[設定の概要]* を確認し、**[インストール]** をクリックします。

    ![Microsoft Azure Backup PreReq2](./media/backup-azure-microsoft-azure-backup/summary-screen.png)

8. インストールは段階的に実施します。最初の段階では、Microsoft Azure Recovery Services エージェントをサーバーにインストールします。インターネットに接続できるかどうかも、ウィザードによって確認されます。インターネット接続を利用可能な場合は、インストールを続行できます。利用可能でない場合は、プロキシの詳細を指定してインターネットに接続する必要があります。

    次の手順では、Microsoft Azure Recovery Services エージェントを構成します。構成を行う際、マシンをバックアップ資格情報コンテナーに登録するために、コンテナーの資格情報を提供する必要があります。また、Azure とお客様の環境間で送信されるデータを暗号化/復号化するためのパスフレーズも提供します。自動的にパスフレーズを生成するか、最小で 16 文字の独自のパスフレーズを指定することができます。エージェントの構成が完了するまで、ウィザードを続行します。

    ![Azure Backup Serer PreReq2](./media/backup-azure-microsoft-azure-backup/mars/04.png)

9. Microsoft Azure Backup Server の登録が正常に完了すると、セットアップ ウィザードは、SQL Server と Azure Backup Server コンポーネントのインストールおよび構成を開始します。SQL Server コンポーネントのインストールが完了すると、Microsoft Azure Backup Server コンポーネントがインストールされます。

    ![Azure Backup Server](./media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)


インストール手順が正常に完了すると、製品のデスクトップ アイコンも作成されます。アイコンをダブルクリックして、製品を起動します。

### Backup ストレージの追加

一次バックアップ コピーは、Azure Backup Server マシンに接続されているストレージに保持されます。ディスクを追加する方法の詳細については、「[記憶域プールおよびディスク記憶域の構成](https://technet.microsoft.com/library/hh758075.aspx)」を参照してください。

> [AZURE.NOTE] Azure にデータを送信する場合でも、Backup ストレージを追加する必要があります。Azure Backup Server の現在のアーキテクチャでは、Azure Backup コンテナーにはデータの *2 番目の*コピーが保持され、最初の (必須の) バックアップ コピーはローカル ストレージに保持されます。

## 4\.ネットワーク接続

![step4](./media/backup-azure-microsoft-azure-backup/step4.png)

Azure Backup Server が正常に動作するためには、Azure Backup サービスに接続されている必要があります。マシンが Azure に接続されているかどうかを確認するには、Azure Backup Server PowerShell コンソールで ```Get-DPMCloudConnection``` コマンドレットを使用します。コマンドレットの出力が TRUE の場合、マシンは接続されていますが、それ以外の場合は接続されていません。

同時に、Azure のサブスクリプションが正常な状態である必要があります。サブスクリプションの状態を確認および管理するには、[サブスクリプション ポータル](https://account.windowsazure.com/Subscriptions)にログインします。

Azure への接続と Azure サブスクリプションの状態がわかれば、以下の表から、提供されるバックアップ/復元機能に対する影響を確認することができます。

| 接続状態 | Azure サブスクリプション | Azure へのバックアップ| ディスクへのバックアップ | Azure からの復元 | ディスクからの復元 |
| -------- | ------- | --------------------- | ------------------- | --------------------------- | ----------------------- |
| 接続中 | アクティブ | 許可 | 許可 | 許可 | 許可 |
| 接続中 | 有効期限切れ | 停止済み | 停止済み | 許可 | 許可 |
| 接続中 | プロビジョニング解除済み | 停止済み | 停止済み | 停止され、Azure の回復ポイントが削除される | 停止済み |
| 切断されている期間が 15 日を越える | アクティブ | 停止済み | 停止済み | 許可 | 許可 |
| 切断されている期間が 15 日を越える | 有効期限切れ | 停止済み | 停止済み | 許可 | 許可 |
| 切断されている期間が 15 日を越える | プロビジョニング解除済み | 停止済み | 停止済み | 停止され、Azure の回復ポイントが削除される | 停止済み |

### 接続の切断からの回復
ファイアウォールまたはプロキシにより Azure へのアクセスが妨げられている場合、ファイアウォール/プロキシのプロファイルのホワイトリストに以下のドメイン アドレスを追加する必要があります。

- www.msftncsi.com
- *.Microsoft.com
- *.WindowsAzure.com
- *.microsoftonline.com
- *.windows.net

Azure Backup Server マシンが Azure に接続できるようになると、実行可能な操作が Azure サブスクリプションの状態に応じて決まります。マシンが "接続中" になった場合に許可される操作の詳細は、上記の表に記載されています。

### サブスクリプションの状態の処理

Azure サブスクリプションの状態が "*有効期限切れ*" または "*プロビジョニング解除済み*" である場合、"*アクティブ*" 状態にすることができます。ただし、状態が*アクティブ*でない間は、製品の動作に次のような影響があります。

- サブスクリプションが "*プロビジョニング解除済み*" の場合、プロビジョニングが解除されている期間は機能を使用できません。*アクティブ*になると、製品のバックアップ/復元機能を使用できるようになります。ローカル ディスクのバックアップ データが十分に長い間保持されている場合は、それらのデータも回復できます。ただし、Azure に保持されるバックアップ データは、サブスクリプションが "*プロビジョニング解除済み*" 状態になると失われ、回復できなくなります。
- サブスクリプションが "*有効期限切れ*" になった場合は、再び "*アクティブ*" になるまで機能を使用できなくなるだけです。サブスクリプションが "*有効期限切れ*" になった期間に予定されていたバックアップは実行されません。


## トラブルシューティング

Microsoft Azure Backup サーバーがセットアップ段階 (またはバックアップや復元) でエラーのため失敗した場合、詳細については、この[エラー コードのドキュメント](https://support.microsoft.com/kb/3041338)を参照してください。[Azure Backup 関連の FAQ](backup-azure-backup-faq.md) に関するページも参照してください。


## 次のステップ

Microsoft TechNet サイトの「[System Center 2012 R2 Data Protection Manager (DPM) 用の環境の準備](https://technet.microsoft.com/library/hh758176.aspx)」で詳細を確認します。このページには、Azure Backup Server のデプロイと使用が可能なサポートされる構成も記載されています。

以下の記事により、Microsoft Azure Backup Server を使用したワークロードの保護について理解を深めてください。

- [SQL Server のバックアップ](backup-azure-backup-sql.md)
- [SharePoint サーバーのバックアップ](backup-azure-backup-sharepoint.md)
- [代替サーバーのバックアップ](backup-azure-alternate-dpm-server.md)

<!---HONumber=AcomDC_0831_2016-->