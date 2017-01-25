# Bメッセ - サーバライブラリ[Ruby on Rails]

「Bメッセ - サーバライブラリ」はチャットライブラリ「Bメッセ」を利用する際に、サーバサイドに必要な機能を提供する Ruby gem 形式で提供するライブラリです。  

現在は以下の2つの機能を提供します。

- 認証トークンの発行
- Push通知の送信

## 動作環境

- Ruby  2.0.0 以上
- Rails 3.0.0 以上

## インストール

### gem モジュールのインストール

プロジェクトのルートディレクトリに gem ファイルを置き、下記コマンドでインストールしてください。

    gem install -l bmesse-1.2.0.gem

### Firebase（サービスアカウント）認証ファイルの取得

BメッセはFirebaseを利用してます。Bメッセの導入の前に下記手順にてFirebaseの認証ファイルを取得し保管してください。  
ファイル内の設定項目のいくつかをBメッセの設定で利用します。

- Firebase の対象プロジェクトコンソールにアクセスする。
- 設定アイコンをクリックし、「権限」を選択する。
- サイドメニューより「サービスアカウント」を選択する。
- ヘッダメニューより「サービスアカウントを作成」を選択する。
- サービスアカウント名に任意の名前を入力する。
- 役割にドロップダウンから「プロジェクト」「編集者」を選択する。
- 「新しい秘密鍵の提供」にチェックを入れ、キーのタイプに「JSON」を選択する。
- 「Google Apps のドメイン全体の委任を有効にする」は選択しない。
- 「作成」ボタンを押すと認証ファイルがダウンロードされる。

### bmesse.yml の設置

設定ファイル bmesse.yml を Rails の config ディレクトリに配置し、以下の項目を設定します。

- /notification/title  
  プッシュ通知のタイトル

- /notification/fcm-post-url
  FirebaseCloudMessaging における送信先URL。通常変更の必要はありません。

- /notification/gcm-post-url
  GoogleCloudMessaging における送信先URL。通常変更の必要はありません。

- /notification/server-key  
  プッシュ通知を行う際の Firebase サーバーキー。Firebaseのコンソール上の下記の場所から取得してください。  
  設定アイコン > プロジェクトの設定 > クラウドメッセージング > プロジェクト キー > サーバーキー

- /firebase/auth-token/service_account_email  
  Firebaseに接続するための設定項目。前項[「Firebase（サービスアカウント）認証ファイルの取得」](#Firebase（サービスアカウント）認証ファイルの取得)で取得したファイル内の同一項目の値を設定する。

- /firebase/auth-token/private_key  
  Firebaseに接続するための設定項目。前項[「Firebase（サービスアカウント）認証ファイルの取得」](#Firebase（サービスアカウント）認証ファイルの取得)で取得したファイル内の同一項目の値を設定する。  
  値は "-----BEGIN PRIVATE KEY-----..." で始まり、"...-----END PRIVATE KEY-----\n" で終わります。

- /firebase/auth-token/expires  
  Firebaeのカスタム認証により発行された認証トークンの有効期間（秒）。最大3600秒。

- /firebase/base-url
  リアルタイムデータベースへアクセスする際のベースURL。Firebaseのコンソール上の下記の場所から取得してください。
  Database > "データ" タブ > クリップアイコンの右側に表示されているURL

### Gemfile への追加

Rails プロジェクトの場合はプロジェクトの Gemfile へ下記の記述を追加してください。バージョンは該当するバージョンを指定します。

```
gem 'bmesse', '~> 1.2.0'
```

## ご利用方法

### 認証トークンの発行

既存のアプリケーション認証処理の一環として組込みます。ログインしたユーザのIDを指定して #generate_auth_token() を呼出し認証トークンを発行します。  
戻り値の token をクライアントへ返却し、以降の Bmesse の処理で使用します。

#### サンプルコード

```ruby
require 'rubygems'
require 'bmesse'

module Api

  class InitializeController < ApplicationController
    def index
      user_id = 'abc123456789'  # 実際にはログインしたユーザのID
      
      begin
        token = Bmesse::generate_auth_token(user_id)
      rescue => e
        render e
      else
        render json: {'user_id' => user_id, 'token' => token}
      end
    end
  end
end
```

### 匿名ユーザーからの投稿

不特定多数の方にチャットを公開する場合は Firebase の匿名認証を有効にする必要があります。
下記を選択し「保存」を行ってください。

  Auth > ログイン方法 > 匿名 > 有効にする  

### Push通知の送信

Bメッセの Web SDK から呼出されます。  
メッセージの送信時に、送信先ユーザへPush通知を行います。送信対象は iOSクライアントおよび Webクライアントです。  
リクエストパラメタのオブジェクト params をそのまま #push_notification() の引数に指定してください。  
内部では Firebase Cloud Messaging(FCM) または Google Cloud Messaging(GCM) サービスを利用して通知が行われ、戻り値には送信されたメッセージのIDが返りまが、その後の処理では特に必要ありません。
なお、メッセージIDは FCM および GCM の各メッセージIDが返る場合がありますので配列として処理してください。

#### サンプルコード

```ruby
require 'rubygems'
require 'bmesse'

module Api

  class SendController < ApplicationController

    def index
      begin
        message_ids = Bmesse::push_notification(params)
        render json: {'message_ids' => message_ids.to_json}
      rescue => e
        STDERR.puts e.backtrace.join("\n")
        render e
      else
      end
    end
  end
end
```

## 変更履歴

* v1.2.0  
	* Firebase3.3系に対応

* v1.1.0  
	* Webブラウザ向けPush通知への対応を追加

* v1.0.0  
	初回リリース  

---
? [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
