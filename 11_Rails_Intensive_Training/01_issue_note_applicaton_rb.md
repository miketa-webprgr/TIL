# Issue01 application.rb

## generateコマンド時に生成されるファイルを制限するには

`rails g`すると色々なファイルが自動生成される。  
便利だが、開発プロジェクトによっては無駄なファイルが生成されるので、  
`application.rb`にて事前に設定しておくとよい。  

```rb: config/application.rb
puts class Application < Rails::Application

 #以下のように、generateコマンド時に生成されるファイルに制限をかける
   config.generators do |g|
     g.assets  false
     g.test_framework  false
     g.skip_routes  true
   end
 end
```

## タイムゾーンの設定

```rb: config/application.rb
puts class Application < Rails::Application

    config.time_zone = 'Tokyo'
    config.active_record.default_timezone = :local

    config.i18n.default_locale = :ja
    config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}').to_s]
 end
```

## 参考

RUNTEQのryotaさんに助けられました 笑
> [【Rails】rails generate controllerで生成されるファイルに制限をかける【config\.generatorsの設定】 \- Qiita](https://qiita.com/ryota21/items/643737b54f331b0aaa72)

タイムゾーンの設定はこちらを参照
あと、現場Railsも参照した
>[\[初学者\]Railsのi18nによる日本語化対応 \- Qiita](https://qiita.com/shimadama/items/7e5c3d75c9a9f51abdd5)
