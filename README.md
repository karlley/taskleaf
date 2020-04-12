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

## Git

```
$ git add .
$ git commit -m "first commit"
$ git remote add origin https://github.com/karlley/taskleaf.git
$ git push -u origin master
```

## slim 

### slim gem 追加

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

###  ERB > slim

- app/views/layouts のERB をslim に変換
- Gemfile で指定した環境でコマンドを使うので`bundle exec` を使用

```
$ bundle exec erb2slim app/views/layouts/ --delete
```

## Bootstrap

### Bootstrap Gem

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

bootstrap 読み込み

app/assets/stylesheets/application.scss

```
@import "bootstrap";
```

### views 読み込み

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
  #bootstrap のクラスを追加
    .app-title.navbar.navbar-expand-md.navbar-light.bg-light
      .navbar-brand Taskleaf
    .container
      = yield
```

## ローカライズ(i18n)

### i18n ダウンロード

```
$ wget https://raw.githubusercontent.com/svenfuchs/rails-i18n/master/rails/locale/ja.yml --output-document=config/locales/ja.yml
```

### 日本語にローカライズ設定

config/initializers/local.rb 

```
Rails.application.config.i18n.default_locale = :ja
```


## Task model

Task モデル作成

- model: Task
- table: tasks

```
$ bin/rails g model Task name:string description:text 
$ bin/rails db:migrate
```

## tasks controller & views

tasks コントローラ作成

```
$ bin/rails g controller tasks index show new edit
```

## resources

resouces を使用したルートに変更

./config/routes.rb

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

### link for new

apps/views/tasks/index.html.slim

```
h1 タスク一覧

= link_to '新規登録', new_task_path, class: 'btn btn-primary'
```

### ローカライズ追加

モデル名, 属性名を表示できるように追加

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

**`Task.model_name.human` で翻訳を取得できる**

### ローカライズが効かない

index.html.slim

下記2行だけlocale が効いておらず"Name" と"Created_at" と表示されていた

```
th= Task.human_attribute_name(:name)  # 正常値: 名前
th= Task.human_attribute_name(:created_at)  #正常値: 登録日時
```

#### ローカライズ確認事項

config/locale/initializers/locale.rb

```
# locale を:ja に設定
Rails.application.config.i18n.default_locale = :ja
```

config/locale/ja.yml

```
# :ja ロケールファイルの記述の確認
---
ja:
  activerecord:
    errors:
      messages:
        record_invalid: 'バリデーションに失敗しました: %{errors}'
        restrict_dependent_destroy:
          has_one: "%{record}が存在しているので削除できません"
          has_many: "%{record}が存在しているので削除できません"
    models:
      task: タスク
    attributes:
      task:
        id: ID
        name: 名称
        description: 詳しい説明
        created_at: 登録日時
        updated_at: 更新日時
```

app/views/tasks/index.html.slim

```
h1 タスク一覧

= link_to '新規登録', new_task_path, class: 'btn btn-primary'

.mb-3
table.table.table-hover
  thead.thead-default
    tr
    # model の:name属性と:created_at属性を表示
      th= Task.human_attribute_name(:name) 
      th= Task.human_attribute_name(:created_at) 
  tbody
    - @tasks.each do |task|
      tr
        td= task.name
        td= task.created_at
```

#### ローカライズが効かない原因

config/locale/ja.yml のインデント不良

```
    models:
      task: タスク
    attributes:
      task:
      # ここから下を一段インデントしていなかった
        id: ID
        name: 名称
        description: 詳しい説明
        created_at: 登録日時
        updated_at: 更新日時
```

rails console で確認

```
$ rails c
>> I18n.locale #locale 確認
=> :ja
>> Task.model_name.human #モデルネームが取得可能か確認
=> "タスク”
>> Task.human_attribute_name(:name) #:name属性が取得可能か確認
=> "名称"
>> Task.human_attribute_name(:created_at) #:name属性が取得可能か確認
=> "登録日時"
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

### flash

共通view にflash を表示できるように修正

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

## show 

### link for show

show ページヘのリンク追加

app/views/tasks/index.html.slim

```
.
      tr
      #link_to を追加
        td= link_to task.name, task
        td= task.created_at
```

### show action

app/controllers/tasks_controller.rb

```
  def show
    @task = Task.find(params[:id])
  end
```

### show view

app/views/tasks/show.html.slim

```
h1 タスクの詳細

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'
table.table.table-hover
  tbody
    tr
      th= Task.human_attribute_name(:id)
      td= @task.id
    tr
      th= Task.human_attribute_name(:name)
      td= @task.name
    tr
      th= Task.human_attribute_name(:description)
      td= simple_format(h(@task.description), {}, sanitize: false, wrapper_tag: "div")
    tr
      th= Task.human_attribute_name(:created_at)
      td= @task.created_at
    tr
      th= Task.human_attribute_name(:updated_at)
      td= @task.updated_at
```

## edit & update

- index, show ページにedit へのリンク追加
- edit, update アクション追加
- edit ページ追加

### link for edit

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
      #列を追加
      th
  tbody
    - @tasks.each do |task|
      tr
        td= link_to task.name, task
        td= task.created_at
        #リンク追加
        td
          = link_to '編集', edit_task_path(task), class: 'btn btn-primary mr-3'
```

app/views/tasks/show.html.slim

```
.
      th= Task.human_attribute_name(:updated_at)
      td= @task.updated_at
#リンク追加
= link_to '編集', edit_task_path, class: 'btn btn-primary mr-3'
```

### edit & update action

app/controllers/tasks_controller.rb

```
  def edit
    @task = Task.find(params[:id])
  end

  def update
    task = Task.find(params[:id])
    task.update!(task_params)
    redirect_to tasks_url, notice: "タスク「#{task.name}」を更新しました。"
  end
```

### edit view

app/views/tasks/edit.html.slim

```
h1 タスクの編集

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

= form_with model: @task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .from-group 
    = f.label :description
    = f.text_area :description, row: 5, class: 'form-control', id: 'task_description'
  = f.submit nil, class: 'btn btn-primary'
```

## 共通部分のパーシャル化

new, edit のview のフォーム部分をパーシャル化

### パーシャル 作成

app/views/tasks/_form.html.slim

```
#@task ではなくtask と表記する事に注意する
= form_with model: task, local: true do |f|
  .form-group
    = f.label :name
    = f.text_field :name, class: 'form-control', id: 'task_name'
  .from-group 
    = f.label :description
    = f.text_area :description, row: 5, class: 'form-control', id: 'task_description'
  = f.submit nil, class: 'btn btn-primary'
```

### new edit パーシャル呼び出し

app/views/tasks/new.html.slim

```
h1 タスクの新規登録 

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

#locals: { task: @task } で_form.html.slim のローカル変数(task) を@task に指定する
= render partial: 'form', locals: { task: @task }
```

app/views/tasks/edit.html.slim

```
h1 タスクの編集

.nav.justify-content-end
  = link_to '一覧', tasks_path, class: 'nav-link'

= render partiial: 'form', locals: { task: @task }
```

## delete

### link for delete

app/views/tasks/index.html.slim

```
.
= link_to '編集', edit_task_path(task), class: 'btn btn-primary mr-3'
#method: :delete でdelete メソッドでリクエストする
= link_to '削除', task, method: :delete, data: { confirm: "タスク「#{task.name}」を削除します。よろしいですか?" }, class: 'btn btn-danger'
```

app/views/tasks/show.html.slim

```
= link_to '編集', edit_task_path, class: 'btn btn-primary mr-3'
= link_to '削除', @task, method: :delete, data: { confirm: "タスク「#{@task.name}」を削除します。よろしいですか?" }, class: 'btn btn-danger'
```

### delete action

app/controllers/tasks_controller.rb

```
.
def destroy
  task = Task.find(params[:id])
  task.destroy
  redirect_to tasks_url, notice: "タスク「#{task.name}」を削除しました。"
end
```