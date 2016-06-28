# Bメッセ - SDK導入ガイド[iOS]

## 目次
[動作環境](#動作環境)  
[ファイル構成](#ファイル構成)  
[インストール](#インストール)  
[導入方法](#導入方法)  
[変更履歴](#変更履歴)  

<h2 id="動作環境">動作環境</h2>

- iOS 7.1.2 以上
- iOS SDK 8.1 以上
- Xcode 7.3 以上

### 注意

本SDKの利用には別途サーバ側の構築が必要です。詳しくは[サーバライブラリ導入ガイドイド](https://github.com/flexfirm/bmesse-docs/blob/master/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E5%B0%8E%E5%85%A5%E3%82%AC%E3%82%A4%E3%83%89_for_Ruby.md)をご覧ください。  

<h2 id="ファイル構成">ファイル構成</h2>
BメッセSDKのファイル構成は以下の通りです。  

<pre>
/ios-sdk
    ├─api-docs         // APIリファレンス
    ├─sample           // Bメッセを導入したサンプルアプリケーション
    └─Bmesse.framework // BメッセFrameworkライブラリ
</pre>

<h2 id="インストール">インストール</h2>

### Firebase のインストール
Bメッセは Firebase をバックエンドサービスとして利用しているため、まずは Firebase のライブラリをインストールします。  
プロジェクトの Podfile に下記を追加してください。

<pre>
pod 'Firebase/Messaging'
</pre>

下記コマンドでインストールを実行します。

<pre>
$ pod install
</pre>


### GoogleService-Info.plist の設置

Firebase を使うための設定が定義された "GoogleService-Info.plist" をプロジェクトのルートディレクトリに置きます。ファイルの取得方法は Firebase のドキュメント [Add Firebase to your iOS Project](https://firebase.google.com/docs/ios/setup) をご覧ください。

### Bメッセ Framework の追加

アプリケーションプロジェクトに Bmesse.framework を Embedded Framework として追加します。手順は下記の通りです。

1. Fileメニューより Bmesse.framework をファイルとしてプロジェクトに追加する。
1. TARGET設定の "General" - "Linked Frameworks and Libraries" に追加した Bmesse.framework が含まれているので削除する。
1. TARGET設定の "General" - "Embedded Binaries" に Bmesse.framework を追加する。
1. TARGET設定の "Build Phases" - "Link Binary With Libraries" から Bmesse.framework を削除する。

### リソースファイル配置

/Bmesse.framework/BmesseResources に含まれる全てのリソースファイルをアプリケーションプロジェクトのビルド設定 Copy Bundle Resources に追加してください。  
SDKが使用する画像および言語ファイルが含まれています。 ファイル名は変更しないでください。

### iOS 9における設定

Xcode 7.0以降では Info.plist に以下の設定を追加してください。 {your server domain} には連携させるサーバのドメイン名を指定します。  
また、http でアクセスさせる場合は "NSAllowsArbitraryLoads" も追加します。

	<key>NSAppTransportSecurity</key>
	<dict>
		<key>NSExceptionDomains</key>
		<dict>
			<key>{your server domain}</key>
			<dict>
				<key>NSIncludesSubdomains</key>
				<true/>
				<key>NSTemporaryExceptionRequiresForwardSecrecy</key>
				<false/>
				<key>NSExceptionMinimumTLSVersion</key>
				<string>TLSv1.0</string>
			</dict>
		</dict>
		<!-- http アクセスを許可する場合 -->
		<key>NSAllowsArbitraryLoads</key>
		<true/>
	</dict>

<h2 id="導入方法">導入方法</h2>

導入方法については、SDKに付随しているサンプルアプリケーションを元に説明をします。
サンプルコードは全て Swift です。

下記コードにてBメッセのモジュールをインポートします。

	import Bmesse


### 認証処理

Bメッセのインスタンス生成前に必ず実行してください

<strong style="color:red;">TODO: 修正中のため後日更新</strong>


### ログインユーザの設定

現在ログインしているユーザ情報を設定します。  


	Bmesse.userName = "のび尾さん"
	Bmesse.userId = "user-00002"


### Bメッセのオンライン・オフライン

#### ユーザーをオンラインにする

	Bmesse.publishUserStatusOnline()

#### ユーザーをオフラインにする

	Bmesse.publishUserStatusOffline()

※デフォルトは「オフライン」状態です。  


### チャット窓の表示

チャット相手および表示する際のスレッド名をセットしてから表示メソッドを呼び出します。

	Bmesse.toUserId = user.userId
	Bmesse.toUserName = user.userName
	Bmesse.threadName = "\(user.userName)に相談"
	Bmesse.showMessageThread()


### チャット相手の状態を受け取る

例えばあなたのサービスでチャット相手の一覧表示が存在し、それぞれの相手の__「ステータス」__を表示する場合をご説明します。

	var bmesseListener = { (userId, systemStatus, threadStatus) in
		if targetUserId == userId {
			// ログインの状態を表すのローカライゼーション文字列の取得
			var systemStatusLabelText = systemStatus.localizedDescription()

			// チャットルームの状態を表すのローカライゼーション文字列の取得
			var threadStatusLabelText = threadStatus.localizedDescription()

			// TODO:ステータスに応じた描画を行う
		}
	}
	
	// ユーザーステータスの変更を感知するブロックを設定
	Bmesse.setUserStatusListener(bmesseListener)
	// ユーザーステータスの変更感知を開始
	Bmesse.startUserStatusListener()

##### ログインの状態を表すステータス

ログイン状態は下記クラスにより表されます。詳細はAPIリファレンスを参照してください。

- /bmesse/Database/SystemUserStatusEnum

__取得できる状態__  

|状態     |Bメッセでの定数 (int) |
|:------:|:------:|
| オンライン |SystemUserStatusEnum.Online|
| オフライン |SystemUserStatusEnum.Offline|



#### チャットルームでの状態を表すステータス

チャットルームでの状態は下記クラスにより表されます。詳細はAPIリファレンスを参照してください。

- /bmesse/Database/ThreadUserStatusEntity

__取得できる状態__  

|状態                |Bメッセでの定数 (int) |
|:-----------------:|:------:|
| チャット中            |ThreadUserStatusEnum.Active|
| 書込み中            |ThreadUserStatusEnum.Writing|
| 退出中             |ThreadUserStatusEnum.Left|


### アプリユーザーの情報を送信する

チャットメッセージを送信する際に、アプリ側の情報を合わせてサーバへ送信し、サーバ側でのチャット時に利用することができます。  

	Bmesse.addAdditionalInfo(key: "相談ポイント", value: "1000")
	Bmesse.addAdditionalInfo(key: "誕生日", value: "1988/01/10")
	Bmesse.addAdditionalInfo(key: "性別", value: "男")


上記でセットされた情報はメッセージの送信毎にサーバへ送信されます。送信が不要になったタイミングで下記メソッドを呼び出すことで以降は送信されなくなります。

	Bmesse.clearAllAdditionalInfos()



### チャットルームの外観を変更する

各箇所のテキストや背景の色を変更する場合は以下のファイルを編集してください。

	/Bmesse.framework/BmesseResources/themes/BMSTheme.plist


### ローカライゼーション文字列を変更する

各テキストラベルのローカライゼーション文字列を変更する場合は以下のファイルを編集してください。

	/Bmesse.framework/BmesseResources/languages/en.lproj
	/Bmesse.framework/BmesseResources/languages/ja.lproj


<h2 id="変更履歴">変更履歴</h2>

v1.0.0  
　初回リリース のため変更履歴なし  



---
© [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
