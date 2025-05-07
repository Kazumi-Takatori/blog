---
title: Exchange 2016/2019 でトランスポート キューにメールが滞留する事象について
date: 2022-01-04
lastupdate: 2022-01-11
tags:
  - Exchange
---

※ この記事は、[Email Stuck in Transport Queues](https://techcommunity.microsoft.com/t5/exchange-team-blog/email-stuck-in-transport-queues/ba-p/3049447) の抄訳です。

こんにちは。日本マイクロソフト Exchange & Outlook サポート チームです。

Exchange 2016 および、Exchange 2019 のマルウェア対策機能の定義ファイルのバージョン確認処理において不具合が確認され、2022 年以降の定義ファイルの処理に限り問題が発生し、メールの送受信に支障をきたしました。

弊社開発部門より以下の公開情報にて事象概要と対処手順を公開致しました。

Title: Email Stuck in Transport Queues  
URL: https://techcommunity.microsoft.com/t5/exchange-team-blog/email-stuck-in-transport-queues/ba-p/3049447

今回は上記公開情報を抄訳し、手順に注意点など一部補足を追加してご案内いたします。  
今後、Update 等により内容に差異が生じた場合はオリジナル (英文) の内容を正とさせて頂きますことを予めご了承ください。

本抄訳は、以下の項目を順にご案内いたします。

- [事象概要](#事象概要)
- [影響を受ける対象のサーバー](#影響を受ける対象のサーバー)
- [対処方法 1 (自動化された対処方法)](#対処方法-1-自動化された対処方法)
  - [事前準備](#事前準備)
  - [スクリプトの実行](#スクリプトの実行)
- [対処方法 2 (手動の対処方法)](#対処方法-2-手動の対処方法)
  - [既存のエンジンとメタデータの削除](#既存のエンジンとメタデータの削除)
  - [最新エンジンへの更新](#最新エンジンへの更新)
  - [エンジン更新情報の確認](#エンジン更新情報の確認)
- [補足情報: 上記手順の実行時のユーザー影響について](#補足情報-上記手順の実行時のユーザー影響について)
- [FAQ: よくお寄せいただくご質問](#FAQ-よくお寄せいただくご質問)

---

## 事象概要

問題の定義ファイルのバージョン確認処理においてマルウェア エンジンがクラッシュすることで、メール配送処理ができず、Exchange サーバー上のキュー内にメールが滞留します。  
本事象はあくまでも定義ファイルのバージョン確認処理に依存した問題であり、マルウェア エンジン自体や、セキュリティ関連の問題ではございません。

Exchange 2016 または Exchange 2019 を経由したメールが送受信できない可能性がございますのでご利用のサーバーにて影響を受けているかどうかを、ご確認いただきますようお願いいたします。

なお、事象の報告を受け、弊社開発部門側では 1 月 2 日に定義ファイルを置き換えての配布を開始しておりますが、すでに問題となった定義ファイルがダウンロードされ事象が発生しているサーバーにおいては、一度定義ファイルを削除し、改めて最新の定義ファイル取得を実施頂く必要がございます。
影響を受けているサーバーでは最新の定義ファイルに更新されると解消する問題ではなく、解消するためにはお客様側の操作が必須となります。

本事象の影響受けている場合、以下のようにソースが "FIPFS" のエラー イベント 5300/1106 がイベント ログ (Application) に記録されます。

> Log Name: Application
> Source: FIPFS
> Logged: 1/1/2022 1:03:42 AM
> Event ID: 5300
> Level: Error
> Computer: server1.contoso.com
> Description: The FIP-FS "Microsoft" Scan Engine failed to load. PID: 23092, Error Code: 0x80004005. Error Description: Can't convert "2201010001" to long.

> Log Name: Application
> Source: FIPFS
> Logged: 1/1/2022 11:47:16 AM
> Event ID: 1106
> Level: Error
> Computer: server1.contoso.com
> Description: The FIP-FS Scan Process failed initialization. Error: 0x80004005. Error Details: Unspecified error.

対処方法としては、上記公開情報に記載の通り 「対処方法 1 (自動化された対処方法)」 と 「対処方法 2 (手動の対処方法)」 の 2 つの方法がございます。それぞれ後述します。

## 影響を受ける対象のサーバー

本事象によりキュー内にメールが滞留する問題は以下種類のサーバーにて影響を受けます。

**影響を受ける:**

- Exchange 2016 メールボックス サーバー
- Exchange 2019 メールボックス サーバー

**影響を受けない:**

- Exchange 2013 (クライアント アクセス サーバー、メールボックス サーバー、エッジ トランスポート サーバー)
- Edge Transport サーバー (すべてのバージョン)
- Exchange Online

## 対処方法 1 (自動化された対処方法)

※ 以下の手順を影響を受けているサーバーすべてで実行いただく必要がございます。

### 事前準備

1. https://aka.ms/ResetScanEngineVersion よりスクリプトをダウンロードし、事象の発生している各 Exchange サーバーに配置します。
2. Exchange 管理シェルを起動します。
3. Execution Policy が RemoteSigned ではない場合は、以下のように変更します。

   ```PowerShell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
   ```

### スクリプトの実行

1. Exchange 管理シェルにて cd でカレント ディレクトリをスクリプトの配置場所に移動します。

   例: cd C:\Scripts

2. 以下のようにスクリプトを実行します。

   ```PowerShell
   .\Reset-ScanEngineVersion.ps1
   ```

   実行結果例:

   ```PowerShell
   [PS] C:\Scripts>.\Reset-ScanEngineVersion.ps1

   EXCH1 Stopping services...
   EXCH1 Removing Microsoft engine folder...
   EXCH1 Emptying metadata folder...
   EXCH1 Starting services...
   WARNING: Waiting for service 'Microsoft Filtering Management Service (FMS)' to start...
   WARNING: Waiting for service 'Microsoft Filtering Management Service (FMS)' to start...
   WARNING: Waiting for service 'Microsoft Filtering Management Service (FMS)' to start...
   WARNING: Waiting for service 'Microsoft Filtering Management Service (FMS)' to start...
   WARNING: Waiting for service 'Microsoft Exchange Transport (MSExchangeTransport)' to start...
   EXCH1 Starting engine update...
   Running as EXCH1-DOM\Administrator.
   --------
   Connecting to EXCH1.CONTOSO.com.
   Dispatched remote command. Start-EngineUpdate -UpdatePath http://amupdatedl.microsoft.com/server/amupdate
   ```

3. 上記 2 のスクリプト実行完了後、最新の定義ファイルのダウンロードが成功し、変更が適用されたかどうかを確認するため、Exchange 管理シェルにて以下 2 つのコマンドを順に実行します。

   ```PowerShell
   Add-PSSnapin Microsoft.Forefront.Filtering.Management.Powershell
   Get-EngineUpdateInformation
   ```

   実行結果例:

   ```PowerShell
   [PS] C:\Scripts> Add-PSSnapin Microsoft.Forefront.Filtering.Management.Powershell
   [PS] C:\Scripts> Get-EngineUpdateInformation

   Engine                : Microsoft
   LastChecked           : 01/01/2022 08:58:22 PM -08:00
   LastUpdated           : 01/01/2022 08:58:31 PM -08:00
   EngineVersion         : 1.1.18800.4
   SignatureVersion      : 1.355.1227.0
   SignatureDateTime     : 01/01/2022 03:29:06 AM -08:00
   UpdateVersion         : 2112330001 (note: higher version number starting with 211233... is also OK)
   UpdateStatus          : UpdateAttemptSuccessful
   ```

   上記 Get-EngineUpdateInformation 実行後に、UpdateVersion 値が事象の発生する連番 2201010001 ではなく、2112330001 や 2112330002 などの 211233\*\*\*\* から始まる連番であることを確認します。なお、スクリプト実行完了までには 1 時間ほどお時間を要する場合がございます。

4. 上記手順の実行完了後、イベント ログ (Application) よりソースが FIPFS のエラー イベント (例: Event ID 5300 や 1106) が記録されないことを確認し、滞留していたメールがはけることをご確認ください。

## 対処方法 2 (手動の対処方法)

※ 以下の手順を影響を受けているサーバーすべてで実行いただく必要がございます。

### 既存のエンジンとメタデータの削除

1. Windows + R で Services.msc を入力し Enter を押下して "サービス" を起動します。
2. Microsoft Filtering Management サービスを停止します。このとき Microsoft Exchange Transport サービスの停止について可否が確認されますため許可します。
3. タスク マネージャーの詳細タブより updateservice.exe が実行されていないか確認します。実行されている場合は停止ください。
4. 以下のフォルダー自体を削除します。もしフォルダー自体の削除に失敗した場合は、その配下のファイルをそれぞれ削除ください。

   %ExchangeInstallPath%FIP-FS\Data\Engines\amd64\Microsoft

5. 以下のフォルダー配下のファイルをすべて削除します。

   %ExchangeInstallPath%FIP-FS\Data\Engines\metadata

### 最新エンジンへの更新

1. Windows + R で Services.msc を入力し Enter を押下して "サービス" を起動します。
2. Microsoft Filtering Management サービスと Microsoft Exchange Transport サービスを開始します。
3. Exchange 管理シェルを起動し、cd コマンドでカレント ディレクトリを %ExchangeInstallPath%Scripts に変更します。例えば以下のように実行します。

   ```PowerShell
   cd $env:ExchangeInstallPath\Scripts
   ```

4. 以下の通り、エンジンの更新スクリプトを実行します。このコマンドの実行からエンジンの更新完了まで 25 分以上かかる可能性がございます。

   ```PowerShell
   .\Update-MalwareFilteringServer.ps1 <対象サーバーの FQDN>
   ```

   実行例:

   ```PowerShell
    .\Update-MalwareFilteringServer.ps1 server01.contoso.local
   ```

### エンジン更新情報の確認

1. Exchange 管理シェルを起動し、以下の 2 つのコマンドを順に実行します。

   ```PowerShell
   Add-PSSnapin Microsoft.Forefront.Filtering.Management.Powershell
   Get-EngineUpdateInformation
   ```

   実行結果例:

   ```PowerShell
   [PS] C:\Scripts> Add-PSSnapin Microsoft.Forefront.Filtering.Management.Powershell
   [PS] C:\Scripts> Get-EngineUpdateInformation

   Engine                : Microsoft
   LastChecked           : 01/01/2022 08:58:22 PM -08:00
   LastUpdated           : 01/01/2022 08:58:31 PM -08:00
   EngineVersion         : 1.1.18800.4
   SignatureVersion      : 1.355.1227.0
   SignatureDateTime     : 01/01/2022 03:29:06 AM -08:00
   UpdateVersion         : 2112330001 (note: higher version number starting with 211233... is also OK)
   UpdateStatus          : UpdateAttemptSuccessful
   ```

   上記 Get-EngineUpdateInformation 実行後に、UpdateVersion が 事象が発生する連番 2201010001 ではなく、2112330001 や 2112330002 などの 211233\*\*\*\* から始まる連番であることを確認します。なお、スクリプト実行完了までには 1 時間ほどお時間を要する場合がございます。

2. 上記手順の実行完了後、イベント ログ (Application) よりソースが FIPFS のエラー イベント (例: Event ID 5300 や 1106) が記録されないことを確認し、滞留していたメールがはけることをご確認ください。

## 補足情報: 上記手順の実行時のユーザー影響について

「既存のエンジンとメタデータの削除」 において、Microsoft Filtering Management サービスと Microsoft Exchange Transport サービスの停止が必要となりますが、当該サーバー上でこの間メールの配送処理が行われなくなりますためご注意ください。
その後、「最新エンジンへの更新」 でサービスを再開するタイミングでサービスが再開され、該当サーバー上でメールの配送処理が再開します。

## FAQ: よくお寄せいただくご質問

**Q. この問題が私の組織に影響するかどのように確認できますか?**  
A. 最新の [HealthChecker スクリプト](https://aka.ms/ExchangeHealthChecker)を組織のすべての Exchange サーバーで実行し、FIP-FS に関する警告の有無で確認できます。

**Q. 対処手順は自動的に行われますか？**  
A. 今回の対処手順はお客様側にて実施いただく必要があり、更新された定義ファイルのダウンロードや、キュー内に滞留したメッセージがはけることが確認できるまでに、お時間を要する作業となります。https://aka.ms/ResetScanEngineVersion よりスクリプトを取得し、実行をすることで自動的に対処を実施することもできますが、手動での対処も実行いただけます。どちらを手段を利用する場合でも、各 Exchange 2016 または Exchange 2019 サーバー上で実行いただく必要がございます。

**Q. 対処のためのスクリプト実行の作業時間はどの程度かかるのか。**  
A. 組織内の Exchange サーバー台数にも依存しますが、1 台あたり対処の手順が完了するまで 30 分 ~ 60 分ほど時間を要することが予想されます。

**Q. キュー内に滞留したメッセージが配信されるまでどの程度時間がかかりますか。**  
A. キュー内に滞留したメッセージの数、滞留が解消するまでに新たに受信するメッセージの数、および、サーバー・ネットワーク等のパフォーマンスに依存するため、具体的な時間はご案内出来かねます。Get-Queue -Server <サーバー名> コマンドを利用してキュー内のメール滞留数を適宜確認いただき、キュー内のメッセージ数の減少していく推移をご確認いただけますようお願いいたします。

**Q. Exchange Hybrid 構成を組んでいる環境ですが、何をすればよいですか。**  
A. オンプレミス Exchange サーバーをメール送付のためにご利用の場合は、Blog に記載の対処を実施ください。
メール送受信には使用されず、オンプレミス Exchange サーバーは受信者の管理の用途のみでご利用の場合は、対処は不要です。

**Q. スクリプトによって停止されるサービスは何ですか。**  
A. Microsoft フィルタリング マネージメント サービス、および、Microsoft Exchange トランスポートサービスの 2 つのサービスです。

**Q. 本事象を考慮して一時的にマルウェア エージェントを無効化しました。Blog の対処策を実施後に、再度有効化すべきですか?**  
A. 一時的にマルウェア エージェントを無効化した場合は、Blog に記載の対処策を実施後に再度有効化ください。
Blog に記載の対処手順を実施いただくことで、マルウェア エージェントも正常に動作し、新たなキューの滞留も発生しません。

**Q. 更新された定義ファイルは 2112330001 という名前になっています。これで良いのですか? 存在しない日付を指しているようですが、これで良いのですか?**  
A. はい、問題ありません。開発部門での対処の結果として、このような存在しない日付の命名規則で定義ファイルがダウンロードされます。存在しない日付を含む定義ファイルであっても Microsoft でサポートしております。
今後リリースされる定義ファイルも同様の形式で各サーバーにて取得されていきます。

**Q. Exchange サーバーがインターネットへのアクセスがない場合、どうすれば良いですか。**  
A. Exchange サーバーがマルウェア対策の更新情報をインターネットより取得しない場合、対処いただく必要はございません。このような構成の場合、マルウェア対策機能による問題となった定義ファイルの更新が行われておりませんので、影響を受けることはございません。

**Q. Exchange 2013 サーバーにて今回の事象は発生していないものの、問題の定義ファイルが取得されていました。どのように対応すれば良いですか?**  
A. Exchange 2013 は今回の事象によりサービスがクラッシュしてメールが滞留することはございません。しかし、もし Exchange 2013 側で問題のマルウェア定義ファイル バージョン "2201010001" を取得していた場合は、Exchange 2013 メールボックス サーバー上にて「対処方法 1 (自動化された対処方法)」または「対処方法 2 (手動の対処方法)」の手順を実施ください。対処を実施しない場合、今後最新のマルウェア定義ファイルの取得が行われません。
ご利用の Exchange 2013 メールボックス サーバーにて "%ExchangeInstallPath%FIP-FS\Data\Engines\amd64\Microsoft" 配下に "2201010001" のフォルダーが存在する場合は、問題のマルウェア定義ファイルに更新されたことを指します。

2022/01/11 更新: Exchange 2013 をご利用の場合、スクリプトを実行する前にバージョンが "21..." に更新されているかご確認ください。異なる自動更新の仕組みにより、既に問題のない最新バージョンへ更新されている可能性があります。

**Q. 「対処方法 1 (自動化された対処方法)」よりスクリプトを実行した際、警告「WARNING: WARNING: Unable to process update request because engine metadata is not available. Attempting to synchronize」が表示されます。**  
A. Exchange サーバーがプロキシ経由で Internet に接続している場合、以下の手順でプロキシを設定します。

- Exchange 管理シェルを起動します。
- `Add-PSSnapin Microsoft.Forefront.Filtering.Management.Powershell` を実行します。
- Set-ProxySettings -Enabled \$true -Server <ProxyServer> -Port <proxy.port> を実行します。

  例:
  `Set-ProxySettings -Enabled $true -Server proxy.contoso.com -Port 8080`

- スクリプトを再実行します。

問題が継続する場合は以下を実行します:

- C:\Windows\System32 より msvcr110.dll をコピーし、以下ディレクトリに配置します。

  %ExchangeInstallPath%FIP-FS\Bin  
  %ExchangeInstallPath%Bin

- Exchange サーバーを再起動します。
- スクリプトを再実行します。

**Q. 組織内に多数のサーバーがあるため、ローカルで定義ファイルを配布することはできないか。**

- こちらの[公開情報](https://docs.microsoft.com/ja-jp/exchange/troubleshoot/setup/manually-update-scan-engines)をもとに定義ファイルを取得します。

  例: `Update-Engine.ps1 -EngineDirPath C:\ScanEngineUpdates\`

- ダウンロードしたファイルを Exchange サーバーがアクセス可能なパスに配置します。

  例: `\\server1\amware`

- 定義ファイルが必要である各 Exchange サーバーにて、次のコマンドを実行します。

  例: `Set-MalwareFilteringServer -PrimaryUpdatePath \\server1\amware -Identity mail1.contoso.com`

- https://aka.ms/ResetScanEngineVersion よりスクリプトをダウンロードし、各サーバーにて以下のように実行します。

  `.\Reset-ScanEngineVersion.ps1`
