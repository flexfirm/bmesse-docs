# Bメッセ - サーバライブラリ[ruby]

## What is Bメッセ - サーバライブラリ?

「Bメッセ - サーバライブラリ」はチャットライブラリ「Bメッセ」を利用する際に、サーバサイドに必要な機能を提供する Ruby gem 形式で提供するライブラリです。

## Installation

**Ruby 2.0.0 以上** に対応します。 プロジェクトのルートディレクトリに gem ファイルを置き、下記コマンドでインストールしてください。

    gem install -l bmesse-1.0.0.gem --save-dev

## Getting Started

以下の2つの機能を提供します。

- 認証トークンの発行
- Push通知の送信

### 認証トークンの発行

アプリケーションにログインしているユーザのIDを指定して #generate_auth_token() を呼出し認証トークンを発行します。  
戻り値の token をクライアントへ返却し、以降の Bmesse の処理で使用します。

```ruby
require 'rubygems'
require 'bmesse'

module Api

  class InitializeController < ApplicationController
    def index
      user_id = 'abc123456789'  # 実際にはログインしているユーザのID
      
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

メッセージの送信先ユーザへPush通知を送信します。  
リクエストパラメタオブジェクト params をそのまま #push_notification() の引数に指定してください。  
内部では Firebase Cloud Messaging サービスを利用して通知が送信され、戻り値には送信されたメッセージのIDが返りまが、その後の処理では特に必要ありません。

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
