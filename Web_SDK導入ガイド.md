# Bメッセ V1.2 - Web SDK導入ガイド
本ガイドは、WebクライアントのオペレーターがBメッセを使えるようにするための導入ガイドです。  
別途、サーバアプリケーションおよびネイティブアプリへの導入も必要です。  
[RubyOnRails_サーバライブラリ導入ガイドはこちら](./RubyOnRails_サーバライブラリ導入ガイド.md)  
[iOS_SDK導入ガイドはこちら](./iOS_SDK導入ガイド.md)  

## 目次
[Bメッセとは](#Bメッセとは) 
[用語](#用語) 
[動作環境](#動作環境)  
[ファイル構成](#ファイル構成)  
[Firebaseの設定](#Firebaseの設定)  
[インストール](#インストール)  
[使用方法](#使用方法)  

<h2 id="Bメッセとは">Bメッセとは</h2>
![アプリ側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_app.png?raw=true)　　　　![Webサービス側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_web.png?raw=true)　　

[Firebase](https://firebase.google.com/)を利用したリアルタイムチャット部品です。  
あなたが運用するWebや、iOSアプリ、Androidアプリに、安定したチャット機能を簡単に組み込む事ができます。  
機能について、詳しくは[「Bメッセ」製品サイト](http://www.bmesse.com/?gh)をご覧ください。  

<h2 id="動作環境">動作環境</h2>
![Webサービス側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_web.png?raw=true)　　

__Chrome（for Windows）__  

※次期対応  
・Chrome(for Mac)  
・Safari(for Mac)  

<h2 id="用語">用語</h2>
### Webアプリケーション
特に断りの無い限り、チャット機能を組み込む対象のWebアプリケーションのことを指します。
サーバ上で稼働するサーバアプリケーション、ブラウザ上で動作するWebクライアント、スマートフォン上で動作するネイティブアプリから構成されると想定します。

### ユーザー
特に断りの無い限り、Webアプリケーションで認証されたユーザーの中で、チャット機能を使用できるようにされたユーザーを指します。
アプリを使うアプリユーザーと、Webクライアントを使うWebユーザーの2種類が存在します。
ユーザーはオンライン、オフラインのどちらかの状態にいるよう制御され、それぞれの状態によってできることや、チャット相手の表示が変わります。

### オペレーター
ユーザーからのメッセージにチャット機能で返信する利用者をオペレーターといい、Webクライアントを使用します。
オペレーターはオンラインの一種として、相談中という状態も持ちます。これはオンラインであるが、応対できるユーザーの上限に達している状態です。

### メッセージ
ユーザーとオペレーターの間で都度、送受信されるテキスト等をさします。

### スレッド
メッセージを送受信するために確立された特定のユーザーとオペレーターの繋がり、およびそこでやり取りされた一連のメッセージを指します。

### チャットルーム
特定のユーザーとオペレータの間でスレッドが共有され、メッセージの送受信ができる画面を指します。
また、ユーザーとオペレーターはそれぞれ、チャット中、書き込み中、退出中など、チャットルーム内での状態を持ち、相手方に表示されます。

<h2 id="ファイル構成">ファイル構成</h2>
BメッセSDKのファイル構成は以下の通りです  
<pre>
/web-sdk
	├─ api-doc		// APIリファレンスです。
	├─ css
	|	└─bmesse-バージョン番号.css		// Bメッセのスタイルシートです。
	└─ js
		├─bmesse-バージョン番号.js			// Bメッセ SDK本体です。
		├─bmesse-config.js				// Bメッセの設定ファイルです。
		├─manifest.json					// Webブラウザ向けPush通知用 manifest ファイルです。
		└─webpush.js					// Webブラウザ向けPush通知用 ServiceWorker です。
</pre>

<h2 id="Firebaseの設定">Firebaseの設定</h2>  

### Firebaseアプリケーション作成
まず、Bメッセを組み込むWebアプリケーション用のFirebaseアプリケーションを作成する必要があります。  
[Firebase](https://firebase.google.com/)にログインして、アプリケーションの作成を行ってください。  
ログインしたら、
`CREATE NEW PROJECT`し、  
`Project name`と`Country/region `を決めて  
`CREATE PROJECT`ボタンを押したら作成完了です。  
ここで作成したPROJECTは、Webアプリケーションで利用するFirebaseのデータベースやPushの管理に利用します。  

### セキュリティルールの設定

- Firebaseコンソールの左側のメニューより「Database」を選択します。
- ヘッダメニューより「RULES」タブを選択します。
- セキュリティルール定義が記述されているテキストを下記テキストに置き換えます。
- 「PUBLISH」ボタンのをおし適用します。

セキュリティルール定義）
```
{
  "rules": {
    "userassociations": {
      ".read": "auth != null",
      ".write": "auth != null"
    },
    "threads": {
      "$thread_id": {
        ".read": "data.child('allow_users/' + auth.uid).exists()",
      	".write": "auth != null"
      }
    },
    "read": {
      ".read": "auth != null",
      ".write": "auth != null"
    },
    "userinfos": {
      "$user_id": {
          ".read": "auth != null",
          ".write": "auth != null"
      }
    },
    "userstatus": {
      ".read": "auth != null",
      ".write": "auth != null"
    },
    "sharebox": {
      "messages": {
        "$thread_id": {
          ".read": "root.child('threads/' + $thread_id + '/allow_users/' + auth.uid).exists()",
          ".write": "root.child('threads/' + $thread_id + '/allow_users/' + auth.uid).exists()"
        }
      },
      "additionalinfos": {
        "$thread_id": {
          ".read": "root.child('threads/' + $thread_id + '/allow_users/' + auth.uid).exists()",
          ".write": "root.child('threads/' + $thread_id + '/allow_users/' + auth.uid).exists()"
        }
      }
    }
  }
}
```

### bmesse-config.jsの編集
__Bmesse.FB_URL__  
Firebaseで作成したプロジェクトのメニューにある`Database`をクリックしてください。  
このページに記載されているURLで`Bmesse.FB`を編集してください。  

__Bmesse.NOTIFICATION_POST_URL__  
Push通知を使用する際にはこの項目も編集してください。  
[RubyOnRails_サーバライブラリ導入ガイド](./RubyOnRails_サーバライブラリ導入ガイド.md)の__Push通知の送信__で実装したURLで編集してください。  

__Bmesse.WEB_NOTIFICATION_ENABLE__  
Webブラウザ向けPush通知を有効にするためのスイッチです。有効にする場合 true にします。  

__Bmesse.WEB_NOTIFICATION_TITLE__  
Webブラウザ向けPush通知のバナータイトル文字列です。  

__Bmesse.WEB_NOTIFICATION_MESSAGE__  
Webブラウザ向けPush通知のバナー本体文字列です。  

__Bmesse.WEB_NOTIFICATION_TRANSITION_URL__  
Webブラウザ向けPush通知のクリックした際の遷移先URLパスを設定します。  

__Bmesse.WEB_NOTIFICATION_ICON_URL__  
Webブラウザ向けPush通知のアイコン画像URLを設定します。  


<h2 id="インストール">インストール</h2>
以下のファイルをWebクライアントに設置してください。  
`bmesse-バージョン番号.css`、`bmesse-バージョン番号.js`、`bmesse-config.js`、`manifest.json`、`webpush.js`  

Bメッセを使用するページのhtmlファイルに以下のタグを入れてください  
```
<link rel="stylesheet" type="text/css" href="{bmesse-バージョン番号.cssへのパス}">
<script type="text/javascript" src="{bmesse-バージョン番号.jsへのパス}"></script>
<script type="text/javascript" src="{bmesse-config.jsへのパス}"></script>
<script type="text/javascript" src="https://cdn.firebase.com/js/client/2.4.2/firebase.js"></script>
```
※[firebaseのバージョンは2.4.2](https://cdn.firebase.com/js/client/2.4.2/firebase.js)を利用ください  

### Webブラウザ向けPush通知の設定### 
Webブラウザ向けのPush通知についての設定を行います。を利用しない場合は行う必要はありません。  

* Firebaseコンソールの設定アイコンをクリックし、「プロジェクトの設定」を選択します。
* ヘッダメニューより「クラウドメッセージング」を選択します。
* 「プロジェクト キー」セクションの「送信者ID」をコピーします。
* manifest.js 内の項目 "gcm_sender_id" の値にペーストします。
* Bメッセを使用する html ファイルの head 内に以下のタグを追加します。  

```
<link rel="manifest" href="manifest.json">
```

※ 現在 Push通知に対応している Webブラウザは Chrome(Win) のみです。

<h2 id="使用方法">使用方法</h2>
使用方法については、SDKに付随しているRailsの`sample`アプリケーションを例に説明をします。  
### Bメッセ 認証処理
Bメッセのインスタンス生成前に必ず実行してください  

```
Bmesse.authenticate = function(authToken, completeCallback, expiredCallback);
```
使用例）
```
Bmesse.authenticate(authToken,
	//completeCallback関数
	function(errorInfo,authInfo) {
		if(errorInfo){
			//認証時のエラー表示
			console.log(errorInfo['message']);
		}else{
			//Bメッセインスタンスを生成
			var bmesse = new Bmesse(myUserId);
			//認証が成功した時の処理をここに書きます
		},

	//expiredCallback関数　認証期間切れのコールバック
	function() {
		console.info("認証期間が切れました。");
	});
});

```
__`authToken`__： サーバアプリケーションで取得した認証トークンを渡します。  
__`completeCallback`__： 認証の返答が返ってきた時の処理を渡します。  
　・認証時にエラーがあった場合は`errorInfo`にその情報が入ります。  
　エラーが無い場合はBメッセのインスタンスをここで生成します。  
　認証成功時の処理もここに記述します。  
__`expiredCallback`__： 認証期限が切れた際の処理を渡します。  

### Bメッセインスタンスを生成する

Bメッセを使用するために必ず実行してください。  

```
var bmesse = new Bmesse(myUserId);
```
__`myUserId`__： WebアプリケーションにログインしたユーザーのIDを渡します。  

以降は、この`bmesse`インスタンスを用いて説明を行います。  

### オペレーターのオンライン・オフライン状態の制御
#### オペレーターをオンラインにする
オペレーターをチャット可能な状態にするためにこのメソッドを呼びます。  
```
bmesse.onLine();
```
#### オペレーターをオフラインにする
オペレーターがチャットを受け付けないようにするために、このメソッドを呼びます。  
```
bmesse.offLine();
```
表示中のページが更新される時にオフラインにする制御はSDK内では行っていません。
必要に応じて呼び出してください。

また、オフライン時には、自動返信やメールによる通知の機能が働きます。 
__・自動返信__  
オフライン時にユーザーがチャットをしてきた場合、ユーザーに以下の自動返信が行われます。  
_「～システムより自動返信～  
現在オフライン中、または他の方とご相談中のため、返信にお時間がかかります。  
ご容赦くださいませ」_  

__・メールによる通知__  
オフライン時にユーザーがチャットをしてきた場合、それを知らせる旨のメールがオペレーターのアドレスへ送信されます。  

#### ユーザーから見えるオペレーターの状態（チャットルーム）

| オンライン時 | オフライン時 |
|:------:|:------:|
| チャット中 |退出中  |
| 書込み中  |退出中 |
| 相談中   |退出中  |

※デフォルトは「オフライン」状態です。  

### 同時チャット可能なユーザー数の設定
オペレーターが、複数のユーザーと同時チャットできる人数の上限を設定します。  
```
bmesse.setAcceptableLimit(limit);
```
__`limit`__： 同時チャットできる数を渡します。  
人数の上限に達しているオペレーターに関して、ユーザー側で「相談中」という状態を受け取れるようになります。  
なお、`bmesse.getAcceptableLimit();`によって設定した値を取得することができます。

### チャットルームの表示
チャットルームは現在２種類あります。
複数人から相談を受けるオペレーター用と、ユーザー用です。
オペレーター用のチャットルームは、切断ボタンや、現在話しているユーザー数、同時対応可能なユーザーの数などが表示されます。

オペレーター用チャットルームの表示
```
bmesse.showExpertUsersMessageThread(selector, toUserId, title);
```
__`selector`__:　チャットルームをはめ込む箇所の、htmlタグid属性を指定します。  
　例えば　、チャットルームをはめ込みたい場所が`<div id="messages_container"></div>`だった場合、  
　`selector`の部分に`'#messages_container'`を渡します。  

__`toUserId`__： アプリ側のユーザーIDを渡します。  
__`title`__： チャットルームのヘッダー名として表示されます。  

指定したセレクタ部分に以下のようなチャットルーム表示されます。  
![img](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/chat_ui.PNG?raw=true)  

ユーザー用チャットルームの表示
```
bmesse.showCustomerUsersMessageThread(selector, toUserId, title);
```
引数の説明はオペレーター用と同等のため省略
指定したセレクタ部分に以下のようなチャットルーム表示されます。  
![img](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/chat_ui_web.PNG?raw=true)  


## ユーザーの状態を受け取る
例えばWebクライアントで、以下のようにユーザー一覧の表示する場合、ユーザーの__「状態」__やメッセージの__「新着」__などの情報を、Webクライアント内に表示する必要があります（赤枠）  
そのよう場合について説明します。  
![img](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/app_sample.jpg?raw=true)

### ユーザー一覧を取得する
```
//Webユーザーの、全ユーザーとの関係を取得し
var myAssociationWithAppUsers = bmesse.getUserAssociationModel();

//ユーザーが増加したときにリアルタイムに検知する
myAssociationWithAppUsers.addListenerAssociationAdded(function (userInfo) {
	//（１）ユーザーのIDを取得
	var appUsersId = userInfo['user_id'];

	//（２）ユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
	var appUsersSystemStatus = userInfo['systemUserStatus'];

	//（３）ユーザーの状態を取得（チャット中、書込み中、退出中などのチャットルームにおける状態）
	var appUsersthreadStatus = userInfo['threadUserStatus'];

	//（４）ユーザーの、全Webユーザーとの関係を取得
	var appUsersAssociationWithWebUsers = userInfo['userAssociation'];

	//（５）ユーザーと、自分との間でやりとりした最後のメッセージ情報を取得
	var appUserOrMyLastMessage = userInfo['latestMessage'];
});
```
上記の`function(userInfo){処理}`の部分をご覧ください。  
この`{処理}`の部分は、オペレーター（Webユーザー）と関係を持つユーザーの数分だけイベントが発生します。  
例えば、3人のユーザーとチャットをしたことのあるオペレーターの場合、3人分のuserInfoを取得するイベントが走ります。  
また、新たに4人目のユーザーがチャットを開始してきた時もこのイベントが走ります。  

なお、`{処理}`内で取得できる（１）～（５）の詳細は以下です。  

|     |取得できる値やオブジェクト |
|:------:|:------:|
| （１） |user_Id（文字列）|
| （２） |Bmesse.SystemUserStatusオブジェクト|
| （３） |Bmesse.ThreadUserStatusオブジェクト|
| （４） |Bmesse.UserAssociationオブジェクト|
| （５） |Bmesse.Messageオブジェクト|

それでは（１）～（５）の具体的な使用方法を説明します。  

### ユーザーの情報を表示する
あなたのサービスのDBからデータを取得して、ユーザーの情報を表示したい場合に、この（１）ユーザーのIDを使用ください。  

### ユーザーのオンライン、オフライン状態をリアルタイムに受け取る
（２）は以下のように使用します。  
```
//（２）ユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
var appUsersSystemStatus = userInfo['systemUserStatus'];
//（２）の状態変化をリアルタイムに検知する
appUsersSystemStatus.addListenerStatus(user_id, function(systemUserStatus) {

	//（２）-1 ユーザーのIDを取得
	var appUserId = systemUserStatus.userId;
	//（２）-2 変化したユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
	var appUserStatus = systemUserStatus.status;

	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	if (appUserStatus == Bmesse.SystemUserStatus.ONLINE){
		// 例）このユーザーの状態を「オンライン」と表示する処理
	}else{
		// 例）このユーザーの状態を「オフライン」と表示する処理
	}
});
```
`function (systemUserStatus){処理}`の部分をご覧ください。  
この`{処理}`の部分は、ユーザーの状態が変化するたびに発生します。  

なお、`{処理}`内で取得できるのは`SystemUserStatus`オブジェクトで、  
取得できる値としては上記コードに記載した（２）1～2に示した値です。  

__（２） - 2 取得できる状態の値__  

|状態     |Bメッセでの定数 (int) |
|:------:|:------:|
| オンライン |Bmesse.SystemUserStatus.ONLINE|
| オフライン |なし|


### ユーザーのチャットルームでの状態をリアルタイムに受け取る
（３）は以下のように使用します。  

```
//（３）ユーザーの状態を取得（チャット中、書込み中、退出中などのチャットルームにおける状態）
var appUsersthreadStatus = userInfo['threadUserStatus'];

appUsersthreadStatus.addListenerStatus(user_id, function(threadUserStatus) {

	//（３）-1 ユーザーとのチャットルームのIDを取得
	var ourThreadId = threadUserStatus.threadId;
	//（３）-2 ユーザーのIDを取得
	var appUserId = threadUserStatus.userId;
	//（３）-3 変化したユーザーの状態を取得（チャット中、書込み中、退出中など）
	var appUserStatus = threadUserStatus.status;

	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	switch (appUserStatus){
	case Bmesse.ThreadUserStatus.INACTIVE:
		//例）未使用のため、ACTIVE時と同じ処理をする
	case Bmesse.ThreadUserStatus.ACTIVE:
		//例）このユーザーの状態を「チャット中」と表示する処理
		break;
	case Bmesse.ThreadUserStatus.WRITING:
		//例）このユーザーの状態を「書込み中」と表示する処理
		break;
	default:
		//例）このユーザーの状態を「退出中」と表示する処理
		break;
	}
});
```
`function (threadUserStatus){処理}`の部分をご覧ください。  
この`{処理}`の部分は、ユーザーの状態が変化するたびに発生します。  
なお、`{処理}`内で取得できるのは`ThreadUserStatus`オブジェクトで、  
取得できる値としては上記コードに記載した（３）1～3に示した値です。  

__（３） - 3 取得できる状態の詳細__  

|状態                |Bメッセでの定数 (int) |
|:-----------------:|:------:|
| チャット中            |Bmesse.ThreadUserStatus.ACTIVE|
| (ユーザーは未使用) |Bmesse.ThreadUserStatus.INACTIVE|
| 書込み中            |Bmesse.ThreadUserStatus.WRITING|
| 退出中             |なし|

### ユーザーからの新着メッセージの情報をリアルタイムに受け取る
（４）は以下のように使用します。  

```
//（４）ユーザーの、全Webユーザーとの関係を取得
var appUsersAssociationWithWebUsers = userInfo['userAssociation'];
//リアルタイムに新着メッセージを取得
appUsersAssociationWithWebUsers.addListenerLatestMessageChanged(function (message) {
	message.isRead(myUserId, function(callbackIsRead, newMessage){

		//（４）-1 ユーザーとのチャットルームのIDを取得
		var threadId = newMessage.threadId;
		//（４）-2 メッセージIDを取得
		var messageId = newMessage.id;
		//（４）-3 メッセージ送信先のユーザーIDと名前を取得
		var toUserId = newMessage.toUserId;
		var toUserName = newMessage.toUserName;
		//（４）-4 メッセージ送信者のユーザーIDと名前を取得
		var fromUserId = newMessage.fromUserId;
		var fromUserName = newMessage.fromUserName;
		//（４）-5 送信したメッセージのテキストを取得
		var messageText = newMessage.message;
		//（４）-6 メッセージ送信時間を取得
		var timeStamp = new Date(newMessage.timestamp).toLocaleString();
		//（４）-7 メッセージがユーザーによって削除されているか否かを取得
		var delete = newMessage.delete;

		//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
		//例）既読でなければ
		if(!callbackIsRead){
			//例）ユーザーの「新着」マークや、新着メッセージの内容をリアルタイムに表示する処理
		}
	}
});
```
`function (callbackIsRead, newMessage) {処理}`の部分をご覧ください。  
この`{処理}`の部分は、新着メッセージがあるたびに発生します。  
なお、`{処理}`内で取得できるのは`Message`オブジェクトで、  
取得できる値は（４）の1～7に示した値です。

__新着を判別する値__  
`callbackIsRead`

|状態                |値 (論理値)   |
|:-----------------:|:------:|
| 既読            |true       |
| 未読            |false       |

### ユーザーからの新着メッセージの情報を受け取る
（５）は以下のように使用します。  
```
//（５）ユーザーと、自分との間でやりとりした最後のメッセージ情報を取得
var appUserOrMyLastMessage = userInfo['latestMessage'];
appUserOrMyLastMessage.isRead(myUserId,function (callbackIsRead, newMessage) {
	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	if(!callbackIsRead){
		//例）ユーザーの「新着」マークや、新着メッセージの内容を画面ロード時に表示する処理
	}
});
```
`function (callbackIsRead, newMessage) {処理}`で受け取れるのは`Message`オブジェクトです。  
取得できる値は前項__「ユーザーからの新着メッセージの情報をリアルタイムに受け取る」__と同様です。  

### ユーザーからのメッセージの付加情報を受け取る
このメソッドは、ユーザーからのメッセージに付加情報を付けた場合のみ使用できます。  
付加情報の設定の仕方について、詳しくは[iOS_SDK導入ガイド](./iOS_SDK導入ガイド.md)を参照ください。  
なお、取得できる付加情報は、最新のメッセージの付加情報のみです。  
最新のメッセージに付加情報が無い場合は、最後に送信された付加情報を取得できます。  

以下のように使用します。  
```
bmesse.getAdditionalInfo(appUserId, function(addtionalIntfo){
	//-1 メッセージのタイムスタンプを取得
	var timestamp = new Date(addtionalIntfo.infos.timestamp.toLocaleString();
	//-2 メッセージの付加情報を取得
	var infoEntitys = addtionalIntfo.infos.info;

	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	Object.keys(infoEntitys).forEach(function (key){
		//情報のkeyを表示
		console.log(key);
		//情報のvalueを表示
		console.log(infoEntitys[key]);
	});
});
```
## チャットルームの外観を変更する
`bmesse-バージョン番号.css`を編集してください。  
詳細はファイルのコメントを直接ご覧ください。  

## 変更履歴

* v1.2.0  
	以下の機能を追加
	* カスタマ向けWebチャット
	* Webブラウザ向けPush通知

* v1.0.0  
	初回リリース のため変更履歴なし  

---
© [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
