# taskleaf

- CRUD
- postgreSQL
- slim 

## setup

```
$ rails new taskleaf -d postgresql
$ brew services start postgresql
$ bin/rails db:create
$ bin/rails s
```

http:localhost:3000

## git

```
$ git add .
$ git commit -m "first commit"
$ git remote add origin https://github.com/karlley/taskleaf.git
$ git push -u origin master
```

## slim

- slim-rails
- html2slim

Gemfile 追記

```
# slim
gem 'slim-rails'
gem 'html2slim'
```

Gem インストール

```
$ bundle
```

## ERB > slim

- app/views/layouts のERB をslim に変換
- Gemfile で指定した環境でコマンドを使うので`bundle exec` を使用

```
$ bundle exec erb2slim app/views/layouts/ --delete
```

## Bootstrap

### Gem

Gemfile 追記

```
# Bootstrap
gem 'bootstrap'
```

Gem インストール

```
$ bundle
```

### CSS > SCSS

application.css 削除, application.scss 作成

```
$ rm app/assets/stylesheets/application.css
$ touch rm app/assets/stylesheets/application.scss
```

application.scss bootstrap 読み込み

```
@import "bootstrap";
```

### application.html.slim

bootstrap のクラスを追加

```
doctype html
html
  head
    title
      | Taskleaf
    = csrf_meta_tags
    = csp_meta_tag
    = stylesheet_link_tag 'application', media: 'all', 'data-turbolinks-track': 'reload'
    = javascript_pack_tag 'application', 'data-turbolinks-track': 'reload'
  body
    .app-title.navbar.navbar-expand-md.navbar-light.bg-light
      .navbar-brand Taskleaf
    .container
    = yield
```