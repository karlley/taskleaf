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

## エラーメッセージを日本語にする(i18n)

日本語翻訳ファイルをダウンロード

```
$ wget https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml --output-document=config/locales/ja.yml
```

config/initializers/local.rb 追加

```
Rails.application.config.i18n.default_locale = :ja
```

## Task model

- model: Task
- table: tasks

```
$ bin/rails g model Task name:string description:text 
$ bin/rails db:migrate
```

## tasks controller & views

```
$ bin/rails g controller tasks index show new edit
```

## resources

./config/routes.rb を修正

```
Rails.application.routes.draw do
  # get 'tasks/index'
  # get 'tasks/show'
  # get 'tasks/new'
  # get 'tasks/edit'
  resources :tasks
end
```

## new

### 新規投稿 ボタン 追加

apps/views/tasks/index.html.slim

```
h1 タスク一覧

= link_to '新規登録', new_task_path, class: 'btn btn-primary'
```

### model 翻訳情報追加

config/local/ja.yml

```
---
ja:
  activerecord:
    errors:
      messages:
        record_invalid: 'バリデーションに失敗しました: %{errors}'
        restrict_dependent_destroy:
          has_one: "%{record}が存在しているので削除できません"
          has_many: "%{record}が存在しているので削除できません"
    # ここから追加
    models:
      tasks: タスク
    attributes:
      tasks:
      id: ID
      name: 名称
      description: 詳しい説明
      created_at: 登録日時
      updated_at: 更新日時
      # ここまで
  date:
```

### new controller

app/controllers/tasks_controller.rb

```
  def new
    @task = Task.new
  end
```

### new view

app/views/tasks/new.html.slim

```
h1 タスクの新規登録 

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-ontrol', id: 'task_name'
  .form-group
    = f.label :description
    = f.text_area :description, rows: 5, class: 'form-control', id: 'task_descriptionl'
  = f.submit nil, class: 'btn btn-primary'

```

## create

### create controller

app/contorollers/tasks_contoroller.rb

```
  def create
    task = Task.new(task_params)
    task.save!
    redirect_to tasks_url, notice: "タスク「#{task.name}」を登録しました。"
  end

  private

    def task_params
      params.require(:task).permit(:name, :description)
    end
```

### create flash

app/views/layouts/application.html.slim

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
      - if flash.notice.present?
        .alert.alert-success= flash.notice
      = yield
```

## index

### index contoller

app/controllers/tasks_controller.rb

```
  def index
    @tasks = Task.all
  end
```

### index view

app/views/tasks/index.html.slim

```
h1 タスク一覧

= link_to '新規登録', new_task_path, class: 'btn btn-primary'

.mb-3
table.table.table-hover
  thead.thead-default
    tr
      th= Task.human_attribute_name(:name) 
      th= Task.human_attribute_name(:created_at) 
  tbody
    - @tasks.each do |task|
      tr
        td= task.name
        td= task.created_at
```

ロケールが効いてない
