# Bメッセ V1.0 - サーバライブラリ導入ガイド（Ruby on Rails版）


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

### Push通知の送信

Bメッセのクライアントライブラリから呼出されます。  
メッセージの送信時に、送信先ユーザへPush通知を行います。    
リクエストパラメタのオブジェクト params をそのまま #push_notification() の引数に指定してください。  
内部では Firebase Cloud Messaging サービスを利用して通知が行われ、戻り値には送信されたメッセージのIDが返りまが、その後の処理では特に必要ありません。

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
