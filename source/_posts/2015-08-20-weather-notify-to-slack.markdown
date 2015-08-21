---
layout: post
title: "天気予報をHerokuを使ってSlackに通知してみた"
date: 2015-08-20 17:11:56 +0900
comments: true
categories: 
---

### 天気情報を取得する
天気情報の取得は[weather_hacks](http://weather.livedoor.com/weather_hacks/webservice)のAPIを利用しました。  
APIはjsonでレスポンスが返されます。  

以下は、APIから天気情報を取得して、その内容を解析して、必要な情報を取得しています。
weather-notifier.rbを作成して、以下のコードを記入します。

```ruby
require 'json'
require 'open-uri'
require 'slack-notifier'

uri = 'http://weather.livedoor.com/forecast/webservice/json/v1?city=400010'

res     = JSON.load(open(uri).read)
title   = res['title']
link    = res['link']
weather = res['forecasts'].first
message = "[#{weather['date']}の#{title}](#{link})は「#{weather['telop']}」です。"
puts message
```

ターミナルで`ruby weather-notifier.rb` と実行すると、今日の天気の情報が表示されます。

### Slackに通知する
天気情報の取得までできたので、次はSlackに通知してみましょう。  
Slackの通知にはIncoming WebHooksを使います。  

Slackの設定画面にアクセスします。  
`https://example.slack.com/services/new/incoming-webhook`  
`example.slack.com`は自分のSlackTeamのドメインをいれてください。  

通知するチャンネルを選んで、ボタンを押せば作成されます。  

設定画面でWebhook URLが確認できます。  
このURLにPOSTすると、Slackに通知できるようになります。  

Customize NameやCustomize Iconを変更すると、通知するbotの名前やアイコンが変更できます。  

では、通知をしてみましょう。  
通知はslack-notifierというgemを使うと簡単にできるので、今回はそれを使います。  

slack-notifier gemをインストールする
```
$ gem install slack-notifier
```

先ほどのスクリプトにslack通知の設定を追加します。  
```ruby
require 'json'
require 'open-uri'
require 'slack-notifier'

uri = 'http://weather.livedoor.com/forecast/webservice/json/v1?city=400010'
webhook_url = 'ここにWebhook URLを設定してください'

res     = JSON.load(open(uri).read)
title   = res['title']
link    = res['link']
weather = res['forecasts'].first

notifier = Slack::Notifier.new webhook_url
message = "[#{weather['date']}の#{title}](#{link})は「#{weather['telop']}」です。"
notifier.ping Slack::Notifier::LinkFormatter.format(message)
```

ターミナルで`ruby weather-notifier.rb` と実行してみましょう。  
Slackに通知されれば成功です。  

### HerokuでAppを作成
さて、天気情報とSlack通知ができるようになりました。
次は定期的に実行できるようにすると便利ですね。  

Herokuを使えば無料でスクリプトを定期実行できます。  
Herokuでアカウントを作成し、Toolbeltをインストールします。  
Toolbeltは、Herokuをコマンドラインから利用できるようになるツールです。  

次に認証を行います。  
ターミナルに`heroku login`とコマンドを入力します。  
メールアドレスとパスワードが聞かれるので、先ほどherokuで登録したメールアドレスとパスワードを入力してください。  

#### アプリを作成
アプリケーションを登録しましょう。

Herokuでは、Gitを使ってリモートリポジトリへプッシュする必要があるので、Gitの登録を行います。

ここでは、weather-notifierというディレクトリを作成し、そこに先ほどのファイルを作成し、Gitで管理します。

また、Herokuでslack-notifier gemを使うためにGemfileでgemを管理します。  
weather-notifierディレクトリの直下にGemfileを作成します。  

Gemfile
```
source 'https://rubygems.org'

gem 'slack-notifier'
```

ターミナルで `bundle install`と実行すると、gemがインストールされ、Gemfile.lockファイルが作成されます。

以下のような構成になります。
```
weather-notifier
├── Gemfile
├── Gemfile.lock
└── weather-notifier.rb
```

Webhook Urlは外部に公開すべきではないので、Herokuの環境変数に登録して、それを使うようにしましょう。


```ruby
require 'json'
require 'open-uri'
require 'slack-notifier'

uri = 'http://weather.livedoor.com/forecast/webservice/json/v1?city=400010'

res     = JSON.load(open(uri).read)
title   = res['title']
link    = res['link']
weather = res['forecasts'].first

notifier = Slack::Notifier.new ENV['SLACK_WEBHOOK_URL']
message = "[#{weather['date']}の#{title}](#{link})は「#{weather['telop']}」です。"
notifier.ping Slack::Notifier::LinkFormatter.format(message)

```


Gitに登録しましょう。  
```
$ git init
$ git add .
$ git commit -m "first commit"
```

Herokuにpushします。
```
$ git push heroku master
```


### カード登録

### スケジューラーを登録
### 時間を設定
