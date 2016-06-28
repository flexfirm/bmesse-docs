# Bメッセ - SDK導入ガイド[javascript]

## 目次
1. [Bメッセとは](#Bメッセとは)  
1. [動作環境](#動作環境)  
1. [ファイル構成](#ファイル構成)  
1. [Firebaseの設定](#Firebaseの設定)  
1. [インストール](#インストール)  
1. [導入方法](#導入方法)  

<h2 id="Bメッセとは">Bメッセとは</h2>
![アプリ側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_app.png?raw=true)　　　　![Webサービス側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_web.png?raw=true)　　

[Firebase](https://firebase.google.com/)を利用したリアルタイムチャット部品です。  
あなたが運用するWebや、iOSアプリ、Androidアプリに、  
安定したチャット機能を簡単に組み込む事ができます。  
機能について、詳しくは[「Bメッセ」製品サイト](http://www.bmesse.com/?gh)をご覧ください。  

<h2 id="動作環境">動作環境</h2>
![Webサービス側](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/for_web.png?raw=true)　　

__Chrome（for Windows）__  

※次期対応  
・Chrome(for Mac)  
・Safari(for Mac)  


【注意】  
本ガイドは、Webサービス側の利用者がBメッセを使えるようにするための導入ガイドとなります。  
別途サーバー用の設定と、アプリ用の設定が必要となります。  
[サーバ用のガイドはこちら](../サーバライブラリ導入ガイド_for_Ruby.md)  
[アプリ用のガイドはこちら](../SDK導入ガイド_for_iOS.md)  

<h2 id="ファイル構成">ファイル構成</h2>
BメッセSDKのファイル構成は以下の通りです  
<pre>
/web-sdk
	├─ api-doc		// APIリファレンスです。
	├─ css
	|	└─bmesse-バージョン番号.css		// Bメッセのスタイルシートです。
	└─ js
		├─bmesse-バージョン番号.js			// Bメッセ SDK本体です。
		└─bmesse-config.js				// Bメッセの設定ファイルです。
</pre>

<h2 id="Firebaseの設定">Firebaseの設定</h2>
BメッセはGoogle社が提供している[Firebase](https://www.firebase.com/)を利用しています。  

### Firebaseアプリケーション作成
まず、Bメッセを組み込むあなたのサービス用のFirebaseアプリケーションを作成する必要があります。  
[ログイン画面](https://www.firebase.com/login/)よりログインして、アプリケーションの作成を行ってください。  
ログインできたら、
`APP NAME`、`APP URL`を決め、`CRETE NEW APP`ボタンを押したら作成完了です。  
（※`APP URL`は後ほど利用します）  
「アプリケーション」という名前ですが、主にあなたのサービスで利用するFirebaseのデータベースやPushの管理を行います。  

### bmesse-config.jsの編集
__Bmesse.FB__  
URL：前の項で作成した`APP URL`で編集してください。  
Push通知を利用しない場合は以上で編集は終了です。  

__Bmesse.NOTIFICATION_POST_URL__  
Push通知を利用する際にはこの項目も編集してください。  
[サーバライブラリ導入ガイド_for_Ruby](../サーバライブラリ導入ガイド_for_Ruby.md)の__Push通知の送信__で実装したURLで編集してください。  

<h2 id="インストール">インストール</h2>
Bメッセを利用するページのhtmlファイルに以下のタグを入れてください  
```
<script type="text/javascript" src="/bmesse.js"></script>
<script type="text/javascript" src="/bmesse-config.js"></script>
<script type="text/javascript" src="https://cdn.firebase.com/js/client/2.4.2/firebase.js"></script>
```
※firebaseのバージョンは随時変更してください  

<h2 id="導入方法">導入方法</h2>
導入方法については、SDKに付随しているRailsの`sample`アプリケーションを元に説明をします。  
### Bメッセ 認証処理
Bメッセのインスタンス生成前に必ず実行してください  

```
Bmesse.authenticate = function(authToken, completeCallback, expiredCallback);
```
利用例）
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
__`authToken`__： あなたのサービスで利用している認証トークンを渡します。  
__`completeCallback`__： 認証の返答が返ってきた時の処理を渡します。  
　・認証時にエラーがあった場合は`errorInfo`にその情報が入ります。  
　エラーが無い場合はBメッセのインスタンスをここで生成します。  
　認証成功時の処理もここに記述します。  
__`expiredCallback`__： 認証期限が切れた際の処理を渡します。  

### Bメッセインスタンスを生成する

Bメッセを利用するために必ず実行してください。  

```
var bmesse = new Bmesse(myUserId);
```
__`myUserId`__： あなたのサービスにおけるWebユーザーのIDを渡します。  

以降は、この`bmesse`インスタンスを用いて説明を行います。  

### Bメッセのオンライン・オフライン
#### ユーザーをオンラインにする
```
bmesse.onLine();
```
#### ユーザーをオフラインにする
```
bmesse.offLine();
```

Bメッセには「オンライン」状態と、「オフライン」状態があります。  
これは、あなたのサービスの利用者の状態（例えばサービスの「ログイン」と「ログオフ」）と、  
Bメッセの状態を同期させるための機能です。  
使える機能に大きな違いはありませんが、アプリユーザーからの見え方が異なります。  

あるアプリユーザーとのチャット画面を開いても、  
「オフライン」時ではアプリユーザーからは全て「退出中」だと認識されます。  

__アプリユーザーから見える状態の違い__  

| オンライン | オフライン |
|:------:|:------:|
| チャット中 |退出中  |
| 書込み中  |退出中 |
| 相談中   |退出中  |

※デフォルトは「オフライン」状態です。  

### 同時チャット可能な人数の設定
あなたのサービスの利用者が、複数のアプリユーザーと同時チャットできる人数を設定します。  
```
bmesse.setAcceptableLimit(limit);
```
__`limit`__： 同時チャットできる数を渡します。  
この数以上の人数とチャットした場合、アプリユーザー側で「相談中」という状態を受け取れるようになります。  

### チャット窓の表示
```
bmesse.showMessageThread(selector,appUserId,appUserName);
```
__`selector`__:　チャット窓（以下チャットルーム）をはめ込む箇所の、htmlタグid属性を指定します。  
　例えば　、チャットルームをはめ込みたい場所が`<div id="messages_container"></div>`だった場合、  
　`selector`の部分に`'#messages_container'`を渡します。  

__`appUserId`__： アプリ側のユーザーIDを渡します。  
__`appUserName`__： アプリ側のユーザー名を渡します。これはチャットルームのヘッダーに表示されます。  

指定したセレクタ部分に以下のようなチャットルーム表示されます。  
![img](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/chat_ui.PNG?raw=true)  

## アプリユーザーの状態を受け取る
例えばあなたのサービスが、以下のようにアプリユーザー一覧の表示が必要なサービスだったとします。  
その場合、アプリユーザーの__「ステータス」__やメッセージの__「新着」__などの情報を、あなたのサービス内に表示する必要があります（赤枠）  
そのよう場合についてご説明します。  
![img](https://github.com/flexfirm/bmesse-docs/blob/img_branch/img/app_sample.jpg?raw=true)

### ユーザー一覧を取得する
```
//Webユーザーの、全アプリユーザーとの関係を取得し
var myAssociationWithAppUsers = bmesse.getUserAssociationModel();

//アプリユーザーが増加したときにリアルタイムに検知する
myAssociationWithAppUsers.addListenerAssociationAdded(function (userInfo) {
	//（１）アプリユーザーのIDを取得
	var appUsersId = userInfo['user_id'];

	//（２）アプリユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
	var appUsersSystemStatus = userInfo['systemUserStatus'];

	//（３）アプリユーザーの状態を取得（チャット中、書込み中、退出中などのチャットルームにおける状態）
	var appUsersthreadStatus = userInfo['threadUserStatus'];

	//（４）アプリユーザーの、全Webユーザーとの関係を取得
	var appUsersAssociationWithWebUsers = userInfo['userAssociation'];

	//（５）アプリユーザーと、自分との間でやりとりした最後のメッセージ情報を取得
	var appUserOrMyLastMessage = userInfo['latestMessage'];
});
```
上記の`function(userInfo){処理}`の部分をご覧ください。  
この`{処理}`の部分は、Webユーザーと関係を持つアプリユーザーの数分だけイベントが発生します。  
例えば、3人のアプリユーザーと会話をしたWebユーザーの場合、3人分のuserInfoを取得するイベントが走ります。  
また、新たに4人目のアプリユーザーが会話をしてきた時もこのイベントが走ります。  

なお、`{処理}`内で取得できる（１）～（５）の詳細は以下です。  

|     |取得できる値やオブジェクト |
|:------:|:------:|
| （１） |user_Id（文字列）|
| （２） |Bmesse.SystemUserStatusオブジェクト|
| （３） |Bmesse.ThreadUserStatusオブジェクト|
| （４） |Bmesse.UserAssociationオブジェクト|
| （５） |Bmesse.Messageオブジェクト|

それでは（１）～（５）の具体的な利用方法を説明します。  

### アプリユーザーの情報を表示する
あなたのサービスのDBからデータを取得して、アプリユーザーの情報を表示したい倍に、この（１）アプリユーザーのIDをご利用ください。  

### アプリユーザーのオンライン、オフライン状態をリアルタイムに受け取る
（２）は以下のように利用します。  
```
//（２）アプリユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
var appUsersSystemStatus = userInfo['systemUserStatus'];
//（２）の状態変化をリアルタイムに検知する
appUsersSystemStatus.addListenerStatus(user_id, function(systemUserStatus) {

	//（２）-1 アプリユーザーのIDを取得
	var appUserId = systemUserStatus.userId;
	//（２）-2 変化したアプリユーザーの状態を取得（オンライン・オフライン状態などのアプリにおける状態）
	var appUserStatus = systemUserStatus.status;

	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	if (appUserStatus == Bmesse.SystemUserStatus.ONLINE){
		// 例）このアプリユーザーの状態を「オンライン」と表示する処理
	}else{
		// 例）このアプリユーザーの状態を「オフライン」と表示する処理
	}
});
```
`function (systemUserStatus){処理}`の部分をご覧ください。  
この`{処理}`の部分は、アプリユーザーの状態が変化するたびに発生します。  

なお、`{処理}`内で取得できるのは`SystemUserStatus`オブジェクトで、  
取得できる値としては上記コードに記載した（２）1～2に示した値です。  

__（２） - 2 取得できる状態の値__  

|状態     |Bメッセでの定数 (int) |
|:------:|:------:|
| オンライン |Bmesse.SystemUserStatus.ONLINE|
| オフライン |なし|


### アプリユーザーのチャットルームでの状態をリアルタイムに受け取る
（３）は以下のように利用します。  

```
//（３）アプリユーザーの状態を取得（チャット中、書込み中、退出中などのチャットルームにおける状態）
var appUsersthreadStatus = userInfo['threadUserStatus'];

appUsersthreadStatus.addListenerStatus(user_id, function(threadUserStatus) {

	//（３）-1 ユーザーとのチャットルームのIDを取得
	var ourThreadId = threadUserStatus.threadId;
	//（３）-2 アプリユーザーのIDを取得
	var appUserId = threadUserStatus.userId;
	//（３）-3 変化したアプリユーザーの状態を取得（チャット中、書込み中、退出中など）
	var appUserStatus = threadUserStatus.status;

	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	switch (appUserStatus){
	case Bmesse.ThreadUserStatus.INACTIVE:
		//例）未使用のため、ACTIVE時と同じ処理をする
	case Bmesse.ThreadUserStatus.ACTIVE:
		//例）このアプリユーザーの状態を「チャット中」と表示する処理
		break;
	case Bmesse.ThreadUserStatus.WRITING:
		//例）このアプリユーザーの状態を「書込み中」と表示する処理
		break;
	default:
		//例）このアプリユーザーの状態を「退出中」と表示する処理
		break;
	}
});
```
`function (threadUserStatus){処理}`の部分をご覧ください。  
この`{処理}`の部分は、アプリユーザーの状態が変化するたびに発生します。  
なお、`{処理}`内で取得できるのは`ThreadUserStatus`オブジェクトで、  
取得できる値としては上記コードに記載した（３）1～3に示した値です。  

__（３） - 3 取得できる状態の詳細__  

|状態                |Bメッセでの定数 (int) |
|:-----------------:|:------:|
| チャット中            |Bmesse.ThreadUserStatus.ACTIVE|
| (アプリユーザーは未使用) |Bmesse.ThreadUserStatus.INACTIVE|
| 書込み中            |Bmesse.ThreadUserStatus.WRITING|
| 退出中             |なし|

### アプリユーザーからの新着メッセージの情報をリアルタイムに受け取る
（４）は以下のように利用します。  

```
//（４）アプリユーザーの、全Webユーザーとの関係を取得
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
		//（４）-7 メッセージがアプリユーザーによって削除されているか否かを取得
		var delete = newMessage.delete;

		//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
		//例）既読でなければ
		if(!callbackIsRead){
			//例）アプリユーザーの「新着」マークや、新着メッセージの内容をリアルタイムに表示する処理
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

### アプリユーザーからの新着メッセージの情報を受け取る
（５）は以下のように利用します。  
```
//（５）アプリユーザーと、自分との間でやりとりした最後のメッセージ情報を取得
var appUserOrMyLastMessage = userInfo['latestMessage'];
appUserOrMyLastMessage.isRead(myUserId,function (callbackIsRead, newMessage) {
	//これ以降にあなたの行いたい処理を記述します。例えば以下のように記述します。
	if(!callbackIsRead){
		//例）アプリユーザーの「新着」マークや、新着メッセージの内容を画面ロード時に表示する処理
	}
});
```
`function (callbackIsRead, newMessage) {処理}`で受け取れるのは`Message`オブジェクトです。  
取得できる値は前項__「アプリユーザーからの新着メッセージの情報をリアルタイムに受け取る」__と同様です。  

### アプリユーザーからのメッセージの付加情報を受け取る
このメソッドは、アプリユーザーからのメッセージに付加情報を付けた場合のみ利用できます。  
付加情報の設定の仕方について、詳しくは[アプリのSDK導入ガイド](../SDK導入ガイド_for_iOS.md)をご参照ください。  
なお、取得できる付加情報は、最新のメッセージの付加情報のみです。  
最新のメッセージに付加情報が無い場合は、最後に送信された付加情報を取得できます。  

以下のように利用します。  
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
`bmesse.css`を編集してください。  
詳細はファイルのコメントを直接ご覧ください。  

## 変更履歴
v1.0.0  
　初回リリース のため変更履歴なし  



---
© [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.