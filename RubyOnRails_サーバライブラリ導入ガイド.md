# Bメッセ V1.0 - Ruby on Rails サーバライブラリ導入ガイド


「Bメッセ - サーバライブラリ」はチャットライブラリ「Bメッセ」を利用する際に、サーバサイドに必要な機能を提供する Ruby gem 形式で提供するライブラリです。  

現在は以下の2つの機能を提供します。

- 認証トークンの発行
- Push通知の送信

## 目次  
[用語](#用語)    
[動作環境](#動作環境)    
[インストール](#インストール)  
[使用方法](#使用方法)  

<h2 id="用語">用語</h2>

### Webアプリケーション
特に断りの無い限り、チャット機能を組み込む対象のWebアプリケーションのことを指します。
サーバ上で稼働するサーバアプリケーション、ブラウザ上で動作するWebクライアント、スマートフォン上で動作するネイティブアプリから構成されると想定します。

### ユーザー
特に断りの無い限り、Webアプリケーションで認証されたユーザーの中で、チャット機能を使用できるようにされたユーザーを指します。
一般のユーザー（以降、アプリユーザー）と、それに応対するオペレーターの2種類が存在します。
アプリユーザーは、アプリからチャット機能を使用し、オペレーターはWebクライアントから使用します。
ユーザーはオンライン、オフラインのどちらかの状態にいるよう制御され、それぞれの状態によってできることや、チャット相手の表示が変わります。

### メッセージ
アプリユーザーとオペレーターの間で都度、送受信されるテキスト等をさします。

### スレッド
メッセージを送受信するために確立された特定のアプリユーザーとオペレーターの繋がり、およびそこでやり取りされた一連のメッセージを指します。

### チャットルーム
特定のアプリユーザーとオペレータの間でスレッドが共有され、メッセージの送受信ができる画面を指します。
また、アプリユーザーとオペレーターはそれぞれ、チャット中、書き込み中、退出中など、チャットルーム内での状態を持ち、相手方に表示されます。


<h2 id="動作環境">動作環境</h2>

- Ruby  2.0.0 以上
- Rails 3.0.0 以上

<h2 id="インストール">インストール</h2>

### gem モジュールのインストール

プロジェクトのルートディレクトリに gem ファイルを置き、下記コマンドでインストールしてください。

    gem install -l bmesse-1.0.0.gem --development

### bmesse.yml の設置

設定ファイル bmesse.yml を Rails の config ディレクトリに配置し、以下の項目を設定します。

- /notification/title  
  プッシュ通知のタイトル

- /notification/fcm-post-url  
  プッシュ通知を行う際の送信先。通常変更する必要はありません。

- /notification/server-key  
  プッシュ通知を行う際の Firebase サーバーキー。Firebaseのコンソール上の下記の場所から取得してください。  
  Setting アイコン > Project Setting > CLOUD MESSAGING > Project keys > Server key

- /firebase/secret  
  Firabaseのデータベースへ接続するための秘密鍵。Firebaseのコンソール上の下記の場所から取得してください。  
  Setting アイコン > Project Setting > DATABASE > Secrets > Database secrets  
  ※ マスクされている文字列にマウスカーソルを当てると "SHOW" ボタンが表示されます。

- /firebase/auth-token/expires  
  Firebaeのカスタム認証により発行された認証トークンの有効期間（秒）。

### Gemfile への追加

Rails プロジェクトの場合はプロジェクトの Gemfile へ下記の記述を追加してください。

```
gem 'bmesse'
```

<h2 id="使用方法">使用方法</h2>

### 認証トークンの発行

サーバアプリケーションの認証処理の一環として組込みます。ログインしたユーザのIDを指定して #generate_auth_token() を呼出し、認証トークンを発行します。  
戻り値の token をWebクライアントまたはネイティブアプリへ返却し、以降の Bメッセ の処理で使用します。

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

### Push通知の送信

Webクライアントに組み込まれた、Web SDKから呼出されます。  
メッセージの送信時に、アプリユーザへPush通知を行います。    
リクエストパラメタのオブジェクト params をそのまま #push_notification() の引数に指定してください。  
内部では Firebase Cloud Messaging サービスを利用して通知が行われます。戻り値は、その後の処理では特に必要ありません。

#### サンプルコード

```ruby
require 'rubygems'
require 'bmesse'

module Api

  class SendController < ApplicationController

    def index
      begin
        fcm_message_id = Bmesse::push_notification(params)
        render json: {'fcm_message_id' => fcm_message_id}
      rescue => e
        render e
      else
      end
    end
  end
end
```
