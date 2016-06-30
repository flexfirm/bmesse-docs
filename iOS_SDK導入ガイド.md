# Bメッセ V1.0 - iOS SDK導入ガイド
## 目次
[用語](#用語)    
[動作環境](#動作環境)  
[ファイル構成](#ファイル構成)  
[インストール](#インストール)  
[使用方法](#使用方法)  
[変更履歴](#変更履歴)  

<h2 id="用語">用語</h2>
### Webアプリケーション
特に断りの無い限り、チャット機能を組み込む対象のWebアプリケーションのことを指します。
サーバ上で稼働するサーバアプリケーション、ブラウザ上で動作するWebクライアント、スマートフォン上で動作するネイティブアプリから構成されると想定します。

### ユーザー
特に断りの無い限り、Webアプリケーションで認証されたユーザーの中で、チャット機能を使用できるようにされたユーザーを指します。
一般のユーザー（以降、アプリユーザー）と、それに応対するオペレーターの2種類が存在します。
アプリユーザーは、アプリからチャット機能を使用し、オペレーターはWebクライアントから使用します。
ユーザーはオンライン、オフラインのどちらかの状態にいるよう制御され、それぞれの状態によってできることや、チャット相手の表示が変わります。
オペレーターはオンラインの一種として、相談中という状態も持ちます。これはオンラインであるが、応対できるアプリユーザーの上限に達している状態です。

### メッセージ
アプリユーザーとオペレーターの間で都度、送受信されるテキスト等をさします。

### スレッド
メッセージを送受信するために確立された特定のアプリユーザーとオペレーターの繋がり、およびそこでやり取りされた一連のメッセージを指します。

### チャットルーム
特定のアプリユーザーとオペレータの間でスレッドが共有され、メッセージの送受信ができる画面を指します。
また、アプリユーザーとオペレーターはそれぞれ、チャット中、書き込み中、退出中など、チャットルーム内での状態を持ち、相手方に表示されます。


<h2 id="動作環境">動作環境</h2>

- iOS 7.0 以上
- iOS SDK 9.3 以上
- Xcode 7.3.1 以上
- Swiftのみ

### 注意

本SDKの利用には別途サーバ側の構築が必要です。詳しくは[サーバライブラリ導入ガイド](https://github.com/flexfirm/bmesse-docs/blob/master/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%A9%E3%82%A4%E3%83%96%E3%83%A9%E3%83%AA%E5%B0%8E%E5%85%A5%E3%82%AC%E3%82%A4%E3%83%89_for_Ruby.md)をご覧ください。  

<h2 id="ファイル構成">ファイル構成</h2>
BメッセSDKのファイル構成は以下の通りです。  

<pre>
/ios-sdk
    ├─api-docs         // APIリファレンス
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

Firebase を使うための設定が定義された "GoogleService-Info.plist" をプロジェクトのルートディレクトリに置きます。ファイルの取得方法は Firebase のドキュメント [Firebaseヘルプ - 設定ファイルをダウンロードする](https://support.google.com/firebase/answer/7015592) をご覧ください。

### Bメッセ Framework の追加

アプリケーションプロジェクトに Bmesse.framework を Embedded Framework として追加します。手順は下記の通りです。

1. Fileメニューより Bmesse.framework をファイルとしてプロジェクトに追加する。
1. TARGET設定の "General" - "Linked Frameworks and Libraries" に追加した Bmesse.framework が含まれているので削除する。
1. TARGET設定の "General" - "Embedded Binaries" に Bmesse.framework を追加する。
1. TARGET設定の "Build Phases" - "Link Binary With Libraries" から Bmesse.framework を削除する。

### iOS 9における設定

Info.plist に以下の設定を追加してください。 {your server domain} には連携させるサーバのドメイン名を指定します。  
今回新たに認証サーバへアクセスする場合も同様です。  
また、http でアクセスさせる場合は "NSAllowsArbitraryLoads" も追加します。  
__※__ 2017年1月よりhttpsでのアクセスが必須になりhttpでアクセスするアプリはリジェクト対象になるためご注意ください。

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

<h2 id="使用方法">使用方法</h2>

使用方法については、SDKに付随しているサンプルアプリケーションを例に説明をします。
サンプルコードは全て Swift です。

下記コードにてBメッセのモジュールをインポートします。

	import Bmesse


### 認証処理

Bメッセを利用するためには、まず最初に認証処理が必要です。サーバアプリケーションの認証処理において認証トークンを発行し、そのトークンを使って認証を行います。  
Bmesse インスタンスを生成する前に必ず実行してください。

    Bmesse.authenticate(
        // Firebaseトークン
        authToken:authToken,
        // 認証完了
        completeCallback: { (error, authInfo) in
            if error != nil {
                print("認証でエラーが発生")
            }
            else {
                // 認証情報
                print("認証に成功")
                print(authInfo)
                // ユーザーステータスの変更感知を開始
                Bmesse.startUserStatusListener()
            }
        },
        // トークンの期限切れ
        expiredCallback: {
            print("トークンの有効期限が切れました")
            // TODO: アプリケーションで実装している認証処理を呼び出すなど。
        }
    )

### ユーザの設定

チャット機能を使うユーザの情報を設定します。  


	Bmesse.userName = "のび尾さん"
	Bmesse.userId = "user-00002"


### ユーザーのオンライン・オフラインの制御

#### ユーザーをオンラインにする

	Bmesse.publishUserStatusOnline()

#### ユーザーをオフラインにする

	Bmesse.publishUserStatusOffline()

※デフォルトは「オフライン」状態です。  


### チャットルームの表示

チャット相手および表示する際のスレッド名をセットしてから表示メソッドを呼び出します。

	Bmesse.toUserId = user.userId
	Bmesse.toUserName = user.userName
	Bmesse.threadName = "\(user.userName)に相談"
	Bmesse.showMessageThread()


### チャット相手の状態を受け取る

例えば、チャット相手の一覧に、それぞれの__「状態」__を表示する場合で説明します。

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

#### オペレーターの状態を取得する

オペレーターの状態は下記クラスにより表されます。詳細はAPIリファレンスを参照してください。

- /bmesse/Database/SystemUserStatusEnum

__取得できる状態__  

|オペレーターの状態     |Bメッセでの定数 (int) |
|:------:|:------:|
| オンライン |SystemUserStatusEnum.Online|
| オフライン |SystemUserStatusEnum.Offline|
| 相談中 |SystemUserStatusEnum.InConsultation|



#### チャットルームでの状態を取得する

チャットルームにおけるオペレーターの状態は下記クラスにより表されます。詳細はAPIリファレンスを参照してください。

- /bmesse/Database/ThreadUserStatusEntity

__取得できる状態__  

|チャットルームでの状態 |Bメッセでの定数 (int) |
|:-----------------:|:------:|
| チャット中（アクティブ）            |ThreadUserStatusEnum.Active|
| チャット中（非アクティブ）            |ThreadUserStatusEnum.Inactive|
| 書込み中            |ThreadUserStatusEnum.Writing|
| 退出中             |ThreadUserStatusEnum.Left|


### アプリユーザーの情報を送信する

メッセージを送信する際に、アプリ側の情報を合わせてサーバアプリケーションへ送信し、オペレーターが利用することができます。  

	Bmesse.addAdditionalInfo(key: "相談ポイント", value: "1000")
	Bmesse.addAdditionalInfo(key: "誕生日", value: "1988/01/10")
	Bmesse.addAdditionalInfo(key: "性別", value: "男")


上記でセットされた情報はメッセージの送信毎にサーバアプリケーションへ送信されます。送信が不要になったタイミングで下記メソッドを呼び出すことで以降は送信されなくなります。

	Bmesse.clearAllAdditionalInfos()



### チャットルームの外観を変更する

各箇所のテキストや背景の色を変更する場合は以下のファイルを編集してください。

	/Bmesse.framework/BmesseResources/themes/BMSTheme.plist


### ローカライゼーション文字列を変更する

各テキストラベルのローカライゼーション文字列を変更する場合は以下のファイルを編集してください。

	/Bmesse.framework/BmesseResources/languages/en.lproj/BMSLocalizable.strings
	/Bmesse.framework/BmesseResources/languages/ja.lproj/BMSLocalizable.strings


<h2 id="変更履歴">変更履歴</h2>

v1.0.0  
　初回リリース のため変更履歴なし  



---
© [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
