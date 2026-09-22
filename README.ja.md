<p align="center">
  <img src="src/App/Assets/iPhoneMirror.png" width="112" alt="iPhoneMirror icon">
</p>

<h1 align="center">iPhoneMirror</h1>

<p align="center">
  Windows で USB または AirPlay により iPhone の画面とシステム音声を低遅延でキャプチャ。<br>
  Low-latency USB and AirPlay iPhone mirroring for Windows.
</p>

<p align="center"><a href="README.md">简体中文</a> · <a href="README.en.md">English</a> · <strong>日本語</strong></p>

<p align="center">
  <a href="https://github.com/RayrenSX/iPhoneMirror/releases"><img alt="GitHub Release" src="https://img.shields.io/github/v/release/RayrenSX/iPhoneMirror?include_prereleases&sort=semver"></a>
  <a href="https://github.com/RayrenSX/iPhoneMirror/actions/workflows/windows-build.yml"><img alt="Windows build" src="https://github.com/RayrenSX/iPhoneMirror/actions/workflows/windows-build.yml/badge.svg"></a>
  <a href="LICENSE"><img alt="GPL v3 License" src="https://img.shields.io/badge/license-GPL--3.0--only-3DA639.svg"></a>
  <img alt="Windows 10 and 11 x64" src="https://img.shields.io/badge/Windows-10%20%7C%2011-0078D4">
</p>

> [!IMPORTANT]
> 現在は公開プレビュー版で、商用の Authenticode 署名はまだ行っていません。Windows で SmartScreen や
> 「発行元不明」の警告が表示される場合があります。Apple Screen Capture は非公開プロトコルを使用しているため、
> 今後の iOS アップデートで対応が必要になる可能性があります。公式ビルドは現在 Windows x64 のみ対応です。
> Windows ARM64 は、USB カーネルドライバーとワイヤレスランタイムに ARM64 版がないため非対応です。

> [!TIP]
> ### Special Thanks: Linux 移植
>
> 本プロジェクトをもとに Linux ネイティブ移植を進め、独立した
> [Linux 対応ブランチ](https://github.com/furruka/iPhoneMirror)を維持している
> **[@furruka](https://github.com/furruka)** に深く感謝します。この取り組みは iPhoneMirror の
> USB・AirPlay ミラーリング経路を Linux へ広げるもので、GUI・描画・音声・映像デコード・USB 通信・
> デバイス検出といった Windows 固有の部分をネイティブに置き換えつつ、上流のプロトコル層・ポリシー層の
> 動作をできる限り一致させています。
>
> この移植は現在も活発に開発中で、利用可能な Linux 向けリリースパッケージはまだありません。
> 進捗・ビルド方法・既知の制限については
> [Linux 対応の説明](https://github.com/furruka/iPhoneMirror/blob/linux-port/docs/LINUX_PORT.md)
> を参照してください。furruka さんの尽力と貢献に改めて感謝するとともに、Linux ユーザーの皆さんの
> 応援をお待ちしています。

## コミュニティ

QQ グループで、ほかのユーザーとの情報交換、不具合の報告、機能の提案などができます。

QQ グループ番号: **1050045279**

## 概要

現在の最新正式版は `v1.8.3` です。iPhoneMirror は Windows 10/11 x64 向けのローカル
iPhone/iPad ミラーリング＆Bluetooth リバースコントロールツールです。クラウド中継に頼らず、
USB 有線キャプチャと LAN 経由の AirPlay 受信を、共通のプレビュー・音声・スクリーンショット・
独立ウインドウ・OBS・複数デバイスセッションの仕組みに統合することを目指しています。

プロジェクトは明確に分かれた 4 つの部分で構成されています。C++ コアは Apple の非公開 USB
プロトコル、QuickTime/CoreMedia の解析、H.264 デコード、D3D11 描画、WASAPI 音声を担当し、
WPF メインアプリはデバイス一覧・セッション制御・ユーザーインターフェイスを担当します。
独立したワイヤレスホストは AirPlay プロトコルとデコードを担当し、上限付きの名前付きパイプで
メディアフレームを受け渡します。ドライバーのインストール・修復・アンインストールは別プロセスの
`iPhoneMirror.Driver.exe` が担当し、メインアプリは有線デバイスの状態を読み取るだけで、
ミラーリング中のプロセスからシステムのドライバーを変更することはありません。

> [!TIP]
> ドライバーのインストールから USB・AirPlay・複数デバイス・独立ウインドウ・OBS までの詳しい手順は、
> [使い方ガイド（中国語）](docs/USER_GUIDE.md)を参照してください。

## 主な特長

### 実際の iPhone/iPad の形状に合わせた表示

iPhoneMirror は、すべてのデバイスに同じ角丸を適用することはしません。Apple の `ProductType`
から iPhone X、ノッチ付きモデル、mini、標準/Max、Dynamic Island、および iPad Pro、Air、mini、
全画面の無印 iPad などの機種ファミリーを判別し、デバイスごとに画面の角丸・曲線・切り抜きの
設定を割り当てます。ホームボタン付きや角が直角と分かっている機種は直角のまま表示し、未知の新機種は
画面比率から控えめにフォールバックして、iPad を iPhone の形で切り抜いてしまうことを防ぎます。

この形状調整はネイティブ描画と独立ウインドウの輪郭の両方に適用されます。ドラッグ、縦横比を保った
拡大縮小、縦横の切り替え、全画面表示でも、ウインドウはそのデバイスの見た目を保ちます。右クリック
メニューから角丸を手動で解除・復元することもできます。角丸のパラメーターは公開されている外観と
画面比率をもとにした視覚的な近似であり、Apple が公表している工業寸法ではありません。

### 2 つの接続経路・複数デバイス・ローカルのネイティブ描画

- USB の非公開 Screen Capture と LAN の AirPlay で、同じデバイス管理・プレビュー・音声・OBS のワークフローを使用できます。
- 各デバイスが独立したセッションと独立ウインドウを持ち、同時に動作します。切り替えのたびに前のデバイスを停止する必要はありません。
- デバイスカードは長押しドラッグで並べ替えられ、ワイヤレスデバイスは接続直後に一度だけ自動で切り替わります。
- H.264/CoreMedia と AirPlay 画面ミラーリングのフレームはローカルでデコードされ、D3D11/DirectComposition でネイティブ表示されます。
- メディアが iPhoneMirror のクラウドを経由することはなく、USB 接続ではネットワークも不要です。
- UI を含まない独立ウインドウを OBS でそのまま使用でき、スクリーンショットはデコード済みフレームから直接取得します。
- オプションの BLE HID マウス/キーボード操作: iOS の AssistiveTouch と組み合わせて使い、iPhone への App のインストールや脱獄は不要です。

## 画面イメージ

![メイン画面とミラーリング設定](docs/images/user-guide/settings-workspace1.png)

### 一般的なミラーリングツールとの違い

| 比較項目 | iPhoneMirror | 一般的なツール |
|---|---|---|
| 接続方式 | USB と AirPlay を統合 | 通常は 1 種類のみ、または別々のアプリ |
| デバイスの形状 | iPhone/iPad の機種ファミリーごとに角丸と曲線を適用 | 共通の角丸、長方形のウインドウ、または余分な黒枠 |
| 複数デバイス | 独立セッション・独立ウインドウ・並べ替え・同時ミラーリング | 多くは 1 台ずつの切り替えが前提 |
| 有線の互換性 | A デモ、B AirPlay（実験的）、C Aisi モードをデバイスごとに選択 | 通常は固定のプロトコルパラメーター |
| OBS | UI のないネイティブ独立ウインドウ | 操作パネルの切り取りやデスクトップキャプチャが必要なことが多い |
| ドライバー | 独立した管理ツールでデバイスごとに確認・修復・アンインストール | メインアプリに混在し、診断情報が少ないことが多い |
| データ経路 | ローカル/LAN 内で処理し、プロジェクトのクラウド中継なし | ログイン・ネット接続・クラウドサービスが必要なものもある |

iPhoneMirror の Bluetooth 操作は iOS の AssistiveTouch と Windows の Bluetooth ペリフェラルモードの
制約を受けるため、マルチタッチの入力はできません。動画編集機能も内蔵していません。Apple の非公開
プロトコルや AirPlay 互換の実装は、今後の iOS アップデートで対応が必要になる可能性があります。

## ダウンロード

[Releases](https://github.com/RayrenSX/iPhoneMirror/releases) から、
`iPhoneMirror-Setup-v*-x64.exe` のダウンロードをおすすめします。インストールウィザードは
簡体字中国語、繁体字中国語（香港）、English、日本語に対応し、インストール先を選択できます。
管理者としてインストールした場合の既定のインストール先は `C:\Program Files\iPhoneMirror` で、
スタートメニューに項目が作成されます。デスクトップのショートカットは任意です。現在のユーザーのみに
インストールする場合、Inno Setup は Windows のユーザー単位のプログラムフォルダーを使用します。
インストール不要版が必要な場合は、`iPhoneMirror-v*-win-x64.zip` をダウンロードしてすべて展開し、
`iPhoneMirror.exe` を実行してください。ZIP 版でワイヤレス用 DLL について「不正なイメージ」や
エラー `0xc0e90002` が表示される場合は Setup 版を使用してください。どうしても ZIP 版を使う場合は、
ダウンロードしたファイルの「プロパティ」で「許可する」にチェックを入れてから展開し直してください。
展開後の DLL を個別に処理するだけでは解決しません。

どちらのパッケージにも .NET ランタイムと独立したドライバー管理ツール `iPhoneMirror.Driver.exe` が
含まれているため、.NET Desktop Runtime やドライバーツールを別途用意する必要はありません。同じ
Release にある `SHA256SUMS.txt` でダウンロードしたファイルの整合性を確認できます。

アプリは既定で、起動後にバックグラウンドで GitHub Release を確認します。更新が見つかると、
バージョン・公開日・Markdown 形式の更新内容が表示され、「今すぐ更新」をクリックすると
ダウンロード・検証・上書きインストールを行い、完了後にアプリが自動的に再起動します。
「バージョン情報」ページでは手動で更新を確認できるほか、起動時の確認、自動ダウンロード、
正式版/Beta の通知をそれぞれ設定できます。アプリのテーマはメイン画面の「設定 → アプリ設定」で
変更します。ネットワークが利用できない場合や確認がタイムアウトした場合でも、通常どおり起動します。

詳しい操作方法は[使い方ガイド（中国語）](docs/USER_GUIDE.md)を参照してください。

使用する PC には Apple USB サポートが必要です。通常は次のいずれかの Apple 公式コンポーネントに
含まれています（ない場合はドライバー管理ツールが後述の手順で取得できます）。

- Microsoft Store 版 **Apple デバイス**
- Apple Mobile Device Support を含むデスクトップ版 iTunes

ワイヤレスの検出には Windows 10/11 標準の DNS-SD を使用し、Bonjour などのシステムサービスは
インストールしません。検出自体に管理者権限は不要です。インストーラー版を管理者モードで
インストールすると、ワイヤレスホスト用に `iPhoneMirror Wireless AirPlay` という名前の
ローカルサブネット限定のファイアウォール規則が作成されます。この規則は `WirelessHost.exe` と
ローカルサブネットに限定して TCP/UDP の受信を許可し、AirPlay の `SETUP` でセッションごとに
動的にネゴシエートされるメディアポートも許可します。アンインストール時に削除されます。
ポータブル版はファイアウォールを自動では変更しないため、必要に応じて
[ガイドのコマンド](docs/USER_GUIDE.md#在-iphoneipad-上连接)で手動追加してください。

## 機能一覧

| 機能 | 現在の実装 |
|---|---|
| 有線ミラーリング | USB 直結。デバイスごとにデモ、AirPlay（実験的）、Aisi 互換モードを選択 |
| ワイヤレスミラーリング | ローカルネットワークの AirPlay。メインプレビューとすべての出力機能に直接対応 |
| ビデオ App からのキャスト | AirPlay/DLNA の HTTP(S)/HLS 再生、再生操作、メディアソースの音声 |
| 映像 | CoreMedia/AVCC H.264、HEVC 記述子の解析、Media Foundation による自動/ハードウェア優先/ソフトウェア互換デコード |
| 描画 | D3D11/DirectComposition によるネイティブプレビュー。BT.709 フルレンジの色メタデータを保持 |
| 音声 | USB 48 kHz PCM と AirPlay PCM。WASAPI 再生、ミュート、音量調整 |
| デバイス | iPhone/iPad、UDID、ProductType、OS バージョン、信頼状態、安定した複数デバイスの切り替え |
| 画面 | ネイティブ/1080p/720p/540p のローカル描画上限、24/30/60/120 FPS の上限 |
| プレビュー | メインウインドウ、タイトルバーなしの独立ウインドウ、全画面、縦横切り替え、縦横比を保った拡大縮小、機種に合わせた角丸 |
| OBS | 独立ウインドウをそのままウィンドウキャプチャで使用可能。専用ウインドウの重複入口なし |
| Bluetooth リバースコントロール | デバイスごとにバインドする BLE HID マウス/キーボード、システムナビゲーション、設定可能なグローバルショートカット |
| 画像調整 | ローカルプレビューのみの明るさ・コントラスト・彩度・ガンマ |
| ツール | スクリーンショット、強制再描画、ショートカット、リアルタイムログ、簡体字中国語・繁体字中国語（香港）・英語・日本語の UI |
| ドライバー | 有線ミラーリング開始前に現在のデバイスを厳密にチェックし、異常時は独立したドライバー管理ツールを起動 |

解像度と FPS の設定はローカル描画のみを制限し、USB で転送される元の画質を下げることはありません。

有線デバイスの既定は **A デモモード（推奨）** です。`Valeria=true` とネイティブの `DisplaySize` を
送信して iPhone の画面全体をミラーリングしますが、ステータスバーの日付・時刻・バッテリーは Apple の
デモ用の値で表示されます。**B AirPlay（実験的）** はネイティブサイズと縦横の自動調整を使用し、
ビデオ App を外部再生に切り替えられますが、映像が切り取られたり全体が表示されなかったりする場合が
あります。**C Aisi モード** は 1565×1565 に固定され、互換性を管理しやすい反面、ソースの鮮明さが
制限されます。各モードの右にある感嘆符アイコンで長所と短所の詳細を確認できます。選択は現在の
有線デバイスにのみ適用されます。

## クイックスタート

1. Release の Setup インストーラーを実行し、スタートメニューから iPhoneMirror を起動します。
   ZIP 版の場合は、すべて展開してから `iPhoneMirror.exe` を実行します。
2. ケーブルで iPhone または iPad を接続し、ロックを解除したまま、デバイスで「このコンピュータを信頼」を選択します。
3. 左側の「ドライバー」をクリックし、対象デバイスに対してワンクリックでインストールします。
   Apple USB サポートとキャプチャフィルタードライバーが必要に応じて補われます。
4. 左側の「ソース」でデバイスを選択し、上部の「ミラーリング開始」をクリックします。現在の有線デバイスの
   ドライバーが見つからない、または異常がある場合、メインアプリは開始を取り消してドライバー管理ツールを自動で開きます。

別のデバイスに切り替えるときは、前のデバイスに QuickTime の終了制御を送信して通常の USB 構成に
戻してから切り替えます。メインウインドウを閉じたときも同じクリーンアップ処理が行われます。

> [!WARNING]
> Zadig で Apple の親デバイスを WinUSB/libusb に置き換えないでください。iPhoneMirror は対象の
> Apple `usbccgp` デバイスインスタンス上の `libusb0` UpperFilter のみを確認します。メインアプリには
> 起動に必要な `libusb0.dll` ユーザーモードランタイムだけが含まれ、カーネルフィルタードライバーを
> 自らインストール・有効化することはありません。ドライバーの変更は独立したドライバーツールで行ってください。

## 有線ドライバーの管理

`iPhoneMirror.exe` はドライバーの状態を読み取るだけで、インストール・修復・アンインストールは
すべて同じフォルダーにある独立した `iPhoneMirror.Driver.exe` が行います。メインウインドウ左側の
「ドライバー」からいつでもこのツールを開けます。すでに開いている場合は既存のウインドウが
アクティブになり、同じパスのツールが重複して起動することはありません。

有線デバイスで「ミラーリング開始」をクリックすると、メインアプリは選択中のデバイスについて次の点を確認します。

- Apple USB の親デバイスが `usbccgp` のままか
- 現在のデバイスに `libusb0` UpperFilter が登録されているか
- `libusb0.sys` のファイルとサービスが正常か
- `libusb0` が現在のデバイスのシリアル番号で正確に列挙できるか

いずれかの確認に失敗すると有線ミラーリングの開始を中止し、ドライバー管理ツールを自動で開きます。
修復を完了し、案内に従ってデバイスを差し直してから、メインアプリに戻って「ミラーリング開始」を
クリックしてください。ドライバー管理ツールの UI ログは
`%LOCALAPPDATA%\iPhoneMirror.Driver\Logs\driver-ui.log`、管理者操作のログは
`%ProgramData%\iPhoneMirror.Driver` にあります。

PC に Apple USB サポートがまったくない場合、ドライバー管理ツールはまず同じフォルダーまたは
キャッシュにある信頼済みの `AppleMobileDeviceSupport64.msi` を使用します。ローカルにない場合は
Apple Software Update Catalog から単体の MSI をダウンロードして検証し、それも取得できない場合に
限り Apple 公式の HTTPS からデスクトップ版 iTunes のインストーラーをダウンロード・検証して、
Apple Mobile Device Support を取り出します。Microsoft Store 版の Apple デバイスも手動で
インストールできます。本プロジェクトが Apple 独自のバイナリを再配布することはありません。
ドライバーの依存関係の一覧は [`docs/DRIVER_DEPENDENCIES.md`](docs/DRIVER_DEPENDENCIES.md) を参照してください。

## 診断ログ

メインアプリは、マネージド UI と業務処理のエラーを
`%LOCALAPPDATA%\iPhoneMirror\Logs\application.log` に、USB・デコード・描画のコアログを同じフォルダーの
`capture.log` に書き込みます。起動に失敗した場合は `startup.log` にも記録され、ワンクリック更新で
インストーラーを起動したときは `installer-update-日時.log` が作成されます。LocalAppData に一時的に
書き込めない場合、重要なマネージドエラーは `%TEMP%\iPhoneMirror-fallback.log` に記録されます。

「バージョン情報 → 診断」から、ログフォルダーを開いたり、ログとダウンロード済みの更新キャッシュを
すぐに削除したりできます。ログは自動でローテーションされ、14 日を過ぎたファイルは削除され、
メインのログフォルダーの合計は 64 MB に制限されます。削除時に使用中のファイルは安全にスキップされ、
ミラーリングが中断されることはありません。

> [!NOTE]
> 上記の自動チェックは USB 有線デバイスのみが対象です。ワイヤレス AirPlay のソースは `libusb0` を
> 読み取ったり必要としたりせず、ドライバーの状態によってドライバー管理ツールが自動で開くこともありません。

## ワイヤレス AirPlay

AirPlay 受信サービスはメインアプリとともに自動で起動するため、「ミラーリング開始」をクリックしなくても
iPhone から検出できます。未接続のときは左側に空の AirPlay デバイスは表示されず、接続が成功した時点で
ワイヤレスデバイスのカードが作成され、接続直後に一度だけ自動で切り替わります。

1. Windows PC と iPhone/iPad を同じ信頼できる LAN に接続します。
2. 左側の「設定」をクリックし、「ワイヤレス AirPlay」でレシーバー名と接続解像度を設定します。
3. 最大 5120×2880 60 fps、1080p 60 fps（既定）、720p 30 fps、540p 30 fps から選択します。
4. 「適用」をクリックします。名前と解像度の変更内容は 1 つの確認ダイアログにまとめて表示されます。
5. iOS のコントロールセンターで「画面ミラーリング」を開き、設定したレシーバー名を選択します。
6. 接続後は右上の「ミラーリング停止」で現在のワイヤレスセッションを終了できます。受信サービスは常駐し続けます。

ワイヤレスのタブには有線用のローカル解像度/FPS 上限は表示されません。ワイヤレスの画質は接続前に
AirPlay のアドバタイズ仕様として宣言する必要があり、名前や仕様を変更するとレシーバーが再起動して
現在のワイヤレスセッションがすべて切断されるため、iPhone でレシーバーを選び直す必要があります。
ワイヤレスの画面ミラーリングでも、メインプレビュー、音量、スクリーンショット、独立ウインドウ、
全画面、複数ウインドウ、OBS を使用できます。

現在の実装にはワイヤレスデバイス数の固定上限はありません。実際に同時接続できる台数は、CPU/GPU、
メモリ、LAN の帯域によって決まります。同じレシーバーでビデオ App からの AirPlay/DLNA キャストも
受け付けます。固定の制御/検出ポートは RAOP `5001`、AirPlay `7001`、DLNA `8090`、SSDP `1900` で、
ミラーリングと RAOP のメディアポートはセッションごとに動的にネゴシエートされます。接続後に約 10 秒
黒い画面が続いて切断される場合、たいていはファイアウォールで固定ポートしか許可されていないことが
原因です。この修正を含むインストーラーに更新するか、ガイドに従って WirelessHost プロセスの TCP/UDP
受信ポートを `Any`（ローカルサブネットに限定したまま）に設定してください。

## AirPlay で音楽を再生

「画面ミラーリング」を開始しなくても、iPhone/iPad の音楽だけを AirPlay で Windows に再生できます。

1. PC と iPhone/iPad を同じ信頼できる LAN に接続し、iPhoneMirror を起動します。
2. ミュージック App またはコントロールセンターの「再生中」パネルで AirPlay オーディオボタンをタップします。
3. 画面ミラーリングと同じレシーバー名を選択します。
4. 接続するとワイヤレスソースが自動で作成され、PCM 音声が再生されます。メイン画面で音量調整、ミュート、キャストの停止ができます。

音楽のみのセッションでは映像は転送されません。メインプレビューには「AirPlay ミュージック」の状態が
表示され、スクリーンショット、独立プレビュー、全画面などの映像ツールは一時的に無効になります。
送信側がその後画面ミラーリングを始めた場合は、映像が届いた時点でこれらのツールが自動的に復帰します。

## ビデオ App からのキャスト

「ビデオ App からのキャスト」と「ワイヤレス AirPlay 画面ミラーリング」は同じネットワークレシーバーを
共有しています。これにより、デバイスの識別情報、ペアリング情報、サービスポートの不一致によって iOS が
レシーバーを表示しなくなる問題を防いでいます。アプリは要求の種類で処理を振り分け、画面ミラーリングの
フレームは通常のプレビューへ、ビデオ App から送られた再生 URL はメインウインドウ内の専用再生画面へ
送られます。2 つの再生処理が混ざることはありません。

1. PC と iPhone/iPad を同じ信頼できる LAN に接続し、iPhoneMirror を起動します。
2. AirPlay 対応のビデオ App でキャストボタンをタップします。
3. ビデオ App で「画面ミラーリング」と同じ AirPlay レシーバーを選択します。アプリは動画再生の要求として、メインウインドウ内の専用再生画面に切り替えます（画面ミラーリングとしては扱いません）。
4. 再生画面を閉じるか、iPhone 側でキャストを停止すると終了します。既存の画面ミラーリングセッションには影響しません。

DRM、ログイン状態、独自の再生プロトコルを使用する App では、他社製のレシーバーに再生可能な URL が
提供されない場合があります。

## サードパーティの依存関係とライセンス

| 依存関係 | 用途 | ライセンス/入手元 |
|---|---|---|
| .NET 10、WPF、Windows SDK | メイン UI、Windows API、配布用ランタイム | Microsoft 公式ランタイム |
| libusb 1.0.29 | オプションの USB 転送互換レイヤー | LGPL-2.1-or-later、`third_party/libusb/` を参照 |
| libusb-win32 1.2.6.0 | 独立したドライバー管理ツールの `libusb0` フィルタードライバー | LGPL-3.0 および上流のライセンス、`src/DriverInstaller/Assets/` を参照 |
| AirPlayServer 1.1.2 | ワイヤレス AirPlay 受信、FairPlay/映像/音声のデコード | GPL-3.0、LGPL-2.1-or-later および上流のライセンス、`third_party/airplay-server/` を参照 |
| FFmpeg 4.4.2 runtime | AirPlayServer 内蔵の H.264/音声ランタイム | LGPL-2.1-or-later、AirPlayServer の配布物に同梱 |
| FFmpeg 8.1.2 runtime | 録画、ライブ配信、ビデオキャストの HLS ブリッジ | GPL-3.0、既定で `tools/ffmpeg/` に同梱 |
| iUsbBridge | 現在のビルドとリリースで使用する USBMux 有線/ワイヤレスリバースコントロールブリッジ | [RayrenSX/iUsbBridge](https://github.com/RayrenSX/iUsbBridge) の iUsbBridge 非商用利用ライセンス（OSI 定義のオープンソースライセンスではありません） |
| quicktime_video_hack fixtures | QuickTime プロトコルの回帰テスト用データ | MIT、`src/Core/tests/fixtures/` でのみ使用 |

Apple デバイス、Apple Mobile Device Support、iTunes、Windows のシステムコンポーネントは、いずれも
本プロジェクトが再配布するサードパーティソフトウェアではありません。著作権、入手元、バージョン、
ハッシュ、ライセンスの詳細は [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) と AirPlayServer の
`SOURCE.md` を参照してください。

### USBMux 有線ミラーリングと有線リバースコントロールの参考プロジェクト

USBMux で有線ミラーリングと有線リバースコントロールを同時に扱えるかを初期に検証した際、AI ツールを
使って [xiaozai-van-liu/iPhoneUsbTouch](https://gitee.com/xiaozai-van-liu/iPhoneUsbTouch) を見つけました。
このプロジェクトは本プロジェクトの USBMux 有線ミラーリング経路の実際の出所の一つであり、本リポジトリの
`tools/iostouch/qt/usb.py`、`tools/iostouch/qt/usbmux_usb.py`、`tools/iostouch/qt/usbmuxd_server.py` に
ある対応する実装は同プロジェクトに由来します。これらのファイルは上流の `iUsbBridge` コンポーネントの
一部ではありません。同プロジェクトの公開リポジトリには、本説明の更新時点でルートの `LICENSE` や本プロジェクトが
採用できる完全なサードパーティライセンス表記がありません。本プロジェクトはこの実装を暫定的に使用しており、
独自ドライバーの開発完了後に完全に廃止する予定です。

実装の出所について、本プロジェクトの作者は次の主張を明確に留保します。iPhoneUsbTouch の USBMux・
有線ミラーリング・有線リバースコントロールの手法の一部または全部は、当時未公開だった作者の
[RayrenSX/iUsbBridge](https://github.com/RayrenSX/iUsbBridge) の手法を、許可なく、当時公開されていた
アーキテクチャ図をもとに再現したもの、あるいは部分的または完全にリバースエンジニアリングしたものであると
考えています。作者が [RayrenSX/iUsbBridge](https://github.com/RayrenSX/iUsbBridge) を公開したことを
踏まえ、xiaozai-van-liu 氏には iUsbBridge 非商用利用ライセンスを同プロジェクトのライセンスに
追記していただくことを希望しています。

現在の iPhoneMirror が使用している USBMux リバースコントロールブリッジは、作者自身による上流プロジェクト
[RayrenSX/iUsbBridge](https://github.com/RayrenSX/iUsbBridge) のものです。ブリッジは独立したコンポーネント
としてビルド・配布されており、ソースコード、バイナリ、非商用利用の制限、表示義務、サードパーティの依存関係は
すべて上流リポジトリとそのライセンスファイルに従います。商用利用には上流の作者から別途許諾を得る必要があります。

## OBS

1. iPhoneMirror で「独立ウインドウ」を開きます。
2. OBS → ソース → ウィンドウキャプチャ を追加します。
3. タイトルに `iPhoneMirror` と対象デバイス名を含む独立ウインドウを選択します。
4. Windows 11 では Windows Graphics Capture の使用をおすすめします。

OBS 30.1 以降では、「アプリケーション音声キャプチャ」で `iPhoneMirror.exe` を選択することもできます。
詳しくは [OBS 出力のドキュメント（中国語）](docs/OBS_OUTPUT.md)を参照してください。

## ショートカットキー

| ショートカット | 操作 |
|---|---|
| `F11` / `Esc` | 全画面表示の開始/終了 |
| `F5` | デバイスを更新 |
| `Ctrl+R` | 強制再描画 |
| `Ctrl+Shift+P` | 独立プレビューを開く |
| `Ctrl+L` | リアルタイムログの表示/非表示 |
| `Ctrl+M` | ミュート/解除 |
| `Ctrl+S` | スクリーンショット |

## 動作確認済みデバイス

| ProductType / iOS | ネイティブ画面 | 実機での結果 |
|---|---:|---|
| `iPhone18,3` / iOS 26.5.2 | 1206×2622 | 約 58.6 FPS、通常のデコード 3–5 ms、48 kHz ステレオ PCM |
| `iPhone13,1` / iOS 18.7.8 | 1082×2340 | 約 58.9 FPS、通常のデコード 3–6 ms、48 kHz ステレオ PCM |

これらの結果は上記の実機の組み合わせで確認したことを示すものであり、すべての iPhone/iOS バージョンでの
互換性を保証するものではありません。

## ソースからのビルド

### GitHub Actions と USB ブリッジ

Windows のワークフローは GitHub がホストする `windows-latest` ランナーを使用します。ビルド時に
[RayrenSX/iUsbBridge](https://github.com/RayrenSX/iUsbBridge) を自動で取得してビルドするため、
ランナーにあらかじめ隣接ディレクトリを用意する必要はありません。ローカルに作業コピーがある場合は、
`IPHONE_MIRROR_USB_BRIDGE_ROOT` を設定して既定のパスを上書きできます。

Actions が失敗した場合は、ログで `Cloning USB touch bridge`、`USB touch bridge build failed`、
`USB touch bridge runtime` を検索すると、それぞれダウンロード、コンパイル、ランタイムのペイロード検証の
段階を特定できます。`Set up MSYS2 UCRT64 for UxPlay` で `Operation too slow` と表示されて失敗する場合、
たいていは MSYS2 ミラーの一時的なタイムアウトです。ワークフローでは全体更新を無効にし、ビルドに必要な
パッケージのみをインストールしているので、再実行してください。

必要なもの:

- Windows 10/11 x64
- Visual Studio Build Tools（MSVC、Windows SDK、CMake を含む）
- .NET 10 SDK と Windows デスクトップのワークロード
- MSYS2 UCRT64: 同梱の UxPlay 予備レシーバーのビルド用に、CMake、Ninja、UCRT64 ツールチェーン、
  GStreamer（base、good、bad、libav）、libplist、OpenSSL

```powershell
git clone https://github.com/RayrenSX/iPhoneMirror.git
cd iPhoneMirror
./build.ps1 -Configuration Release
```

スクリプトは C++20 コアをビルドし、プロトコルテストを実行して、自己完結型の WPF アプリを発行します。

```text
outputs/iPhoneMirror/iPhoneMirror.exe
outputs/iPhoneMirror/iPhoneMirror.Driver.exe
outputs/iPhoneMirror/iPhoneMirror.Core.dll
outputs/iPhoneMirror/iPhoneMirror.VirtualCamera.dll
outputs/iPhoneMirror/iPhoneMirror.VirtualCamera.Admin.exe
outputs/iPhoneMirror/tools/ffmpeg/ffmpeg.exe
outputs/iPhoneMirror/Wireless/iPhoneMirror.WirelessHost.exe
outputs/iPhoneMirror/Wireless/UxPlay/iPhoneMirror.UxPlayHost.exe
outputs/iPhoneMirror/Wireless/UxPlay/uxplay.exe
```

`outputs/iPhoneMirror` は .NET/WPF の依存関係を内蔵した単一ファイルのポータブル版です。インストーラーは
`outputs/iPhoneMirror.Installer` を使用し、メインアプリとドライバー管理ツールで外部のランタイム DLL を
共有することで、インストーラーのダウンロードサイズを抑えています。

既定のビルドには FFmpeg 8.1.2 のメディア出力ランタイムが含まれ、録画と RTMP/SRT/WHIP 配信をそのまま
利用できます。サイズを最小限にしたい場合で、システムの FFmpeg への依存を受け入れられるときに限り、
軽量版を生成してください。

```powershell
.\build.ps1 -Configuration Release -OmitMediaOutputRuntime
```

軽量版のリリース資産を作成する場合も、リリーススクリプトに `-OmitMediaOutputRuntime` を渡してください。

完全な Release 資産（Setup、ZIP、SHA256 一覧、SBOM）を生成するには:

```powershell
./scripts/package_release.ps1 -Version 1.8.3 -GenerateSbom
```

アップロード用の資産を正式に生成する場合は `-UpdateReleaseManifest` を指定してください。リリース
スクリプトが `updates/releases.json` 内の該当バージョンのファイルサイズと SHA256 を同期し、GitHub API が
利用できない場合でも予備の更新元で検証できるようにします。通常のローカルパッケージングでは、オンラインの
リリース一覧は変更されません。

Inno Setup 6.7.3 と簡体字・繁体字中国語の翻訳ファイルは、固定の SHA256 で `work/tools` にダウンロード
されるため、グローバルなインストールは不要です。日本語の翻訳ファイルは Inno Setup 本体に同梱されている
公式のものを使用します。

自己完結型アプリを発行せずに、ビルドと全テストを実行するには:

```powershell
./build.ps1 -Configuration Debug -NoPublish
```

## アーキテクチャ

```text
iPhone/iPad
  ├─ USB / QuickTime ─► H.264 / PCM decode ─┐
  └─ AirPlay ─► WirelessHost ─► I420 / PCM ─┤
                                              └─► native session
                                                   ├─► D3D11 main/detached/fullscreen preview
                                                   ├─► screenshot
                                                   ├─► WASAPI audio
                                                   ├─► FFmpeg MP4 / RTMP / SRT / WHIP
                                                   ├─► Windows 11 Virtual Camera
                                                   └─► OBS Window Capture
```

以下のドキュメントは中国語です。

- [プロトコルの説明](docs/PROTOCOL.md)
- [ソフトウェアアーキテクチャ](docs/ARCHITECTURE.md)
- [D3D11 描画](docs/D3D11_RENDERING.md)
- [デバイスの角丸設定](docs/DEVICE_CORNER_PROFILES.md)
- [音声出力](docs/WASAPI_AUDIO.md)
- [今後のアップグレード計画](docs/ROADMAP.md)

## 現在の制限事項

- 内蔵の録画と RTMP、SRT、WebRTC/WHIP 配信は、ソース音声と対応するエンコーダーが利用できる場合は映像と音声を同時に出力します。音声やエンコーダーが利用できない場合でも、映像のみの出力はすぐに開始できます。MP4、RTMP、SRT は AAC、WHIP は Opus を使用します。
- メインアプリはまだ商用の署名を行っていません。
- 外部キャプチャドライバーについて、クリーンな Win10/Win11 環境でのインストール検証をさらに広く行う必要があります。
- QuickTime Screen Capture は、Apple が公開している安定したサードパーティ向け API ではありません。
- AirPlay 互換の実装は Apple 公式のインターフェイスではないため、今後の iOS アップデートで対応が必要になる可能性があります。
- Bluetooth リバースコントロールはアダプターの BLE ペリフェラルモードと iOS の AssistiveTouch に依存し、ポインターによる 1 本指の操作のみに対応します。
- ハードウェアデコードを利用できるかは Windows の MFT、GPU ドライバー、デバイスによって異なります。非対応の場合は自動的にソフトウェアデコードにフォールバックし、プレビューには引き続き D3D11 を使用します。
- SDR 出力はすべてフルレンジ BT.709 としてタグ付けされます。HDR の表示や下流のプレーヤーでの見え方は、デバイスとディスプレイの性能によって異なります。

## プロジェクトへの参加

問題を報告する前に[サポートについて](SUPPORT.md)をお読みください。開発への貢献については
[CONTRIBUTING.md](CONTRIBUTING.md) を参照してください。セキュリティの問題はリポジトリの
[非公開の脆弱性報告](https://github.com/RayrenSX/iPhoneMirror/security/advisories/new)を使用し、
UDID、ペアリング記録、USB の完全なキャプチャデータを公開の場に貼り付けないでください。

## ライセンスと謝辞

iPhoneMirror 独自のコードは [GNU General Public License v3.0 only](LICENSE) で提供されています。
改変版や派生物を配布する場合は、ソースコードの提供、著作権表示、同一ライセンスでの配布という GPLv3 の
要件に従う必要があります。ソースコードやリリースパッケージとともに配布されるサードパーティの
コンポーネントは、それぞれのライセンスに従います。詳しくは
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) を参照してください。

ワイヤレスレシーバーは独立した GPLv3 プロセスとして配布され、名前付きパイプを通じて GPLv3 の
メインアプリにデコード済みのメディアフレームを渡します。リリースパッケージの `Wireless/licenses` には、
固定されたバージョン、ソースコードへのリンク、バイナリのハッシュ、ライセンスの全文が含まれています。

プロトコル研究の参考:

- [danielpaulus/quicktime_video_hack](https://github.com/danielpaulus/quicktime_video_hack)
- [chotgpt/quicktime_video_hack_windows](https://github.com/chotgpt/quicktime_video_hack_windows)

Apple、iPhone、iOS、QuickTime は Apple Inc. の商標です。本プロジェクトは Apple Inc. と提携関係になく、
Apple Inc. による後援や承認を受けたものでもありません。
