# ① Dockerfileをダウンロード
<br>

docker_go-master.zipを解凍し、docker_go-masterディレクトリをユーザー直下に配置します
![キャプチャ](https://user-images.githubusercontent.com/66953939/84660963-94fddc00-af54-11ea-90ff-e8fcc0af3e5f.png)

<br>

![キャプチャ](https://user-images.githubusercontent.com/66953939/84664878-23289100-af5a-11ea-9818-8b3dd7beb8ca.png)
<br><br>
# ② コマンドプロントで解凍したdocker_go-masterまで移動
```
cd C:\Users\docker_go-master
```
<br><br>
# ③ ビルド実行
```
docker-compose build
```
→「Successfully built ●●●●●●●●●」が出たら完了！
<br><br>

# ④ ビルドしたイメージからコンテナを起動します 
```
docker-compose up -d
```
<br><br>
# ★チェックポイント★
正常に起動しているかを確認
```
docker ps -a
```
Creating docker_go-master_app_1 ... done 
↑のように「docker_go-master_app_1」のコンテナがupしていればOK！
<br><br>

# ⑤ \docker_go-master配下にmain.goファイルを作成
<br><br>

# ⑥ main.goファイルを編集し、Hello Worldを出力させる
```
package main

import "fmt"

func main() {
  fmt.Printf("Hello World\n")
}
```
<br><br>

# ⑦ WEBのコンテナに接続します
```
docker exec -it docker_go-master_app_1 bash
```

<br><br>
# ⑧ コンテナ内でgoファイルを実行
```
go run main.go
```
「Hello World」が出力されていれば成功
![キャプチャ](https://user-images.githubusercontent.com/66953939/84671123-f11b2d00-af61-11ea-8f95-3050b48159de.png)
<br><br>
# ⑯ myapp\config　配下の以下のファイルを編集します
対象ファイル：routes.rb
```
Rails.application.routes.draw do
  root to: 'employees#index'
  resources :employees
end
```
<br><br>
# ⑰ 以下のコマンドで画面を作成します
```
touch app/views/employees/index.html.erb
```

<br><br>
# ⑱ app/views/employees　配下の以下のファイルを編集します
対象ファイル：index.html.erb
```
<h1>List of employees</h1>

<table border="1" width="700" cellspacing="0" cellpadding="5" bordercolor="#333333" class="table table-hover">
  <thead class="thead-dark">
    <tr>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="100"><font size="+1" color="#FFFFFF">Employee no</font></th>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="400"><font size="+1" color="#FFFFFF">Name</font></th>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="150"><font size="+1" color="#FFFFFF">Hire date</font></th>
    </tr>
  </thead>


  <tbody>
    <% @employees.each do |employee| %>
      <tr scope="row">
        <td><%= employee.number %></td>
        <td><%= employee.name %></td>
        <td><%= employee.date %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```
<br><br>
# ⑲ 以下に接続して確認！
```
http://localhost:3000/
```
![image-20191121003744232](https://user-images.githubusercontent.com/53431136/69325948-53db6d00-0c8e-11ea-982e-650f9e31d71d.png)

<br>
このような画面が表示されたら完成！


<br><br>
<br>
# ★クラウド上でRailsを起動しよう★
使用するクラウド・・・Azure
<br><br>
# ⑳ DBの向き先をAzure上のDBに変更します
C:\Users\azure_db\database.yml　をC:\Users\myapp\config\database.yml　にファイルごと上書きします

<br><br>
# ㉑ Dockerファイルの最後に以下を追記します
```
CMD /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 8080 -b '0.0.0.0'"
```
<br><br>
# ㉒ 以下のコマンドでDockerのビルドします
```
docker-compose build --no-cache
```
<br><br>
# ㉓ 以下のコマンドでAzureに接続します
※パスワードが求められます
```
docker login azureContainerRegistryName1.azurecr.io --username azureContainerRegistryName1
```
<br><br>
# ㉔ 以下のコマンドでDockerイメージにタグ付けをします

```
docker tag myapp_web azureContainerRegistryName1.azurecr.io/myapp_web:v1.0.0
```
<br><br>
# ㉒ 以下のコマンドでDockerイメージをAzureにデプロイします
```
docker push azureContainerRegistryName1.azurecr.io/myapp_web:v1.0.0
```
<br><br>
# ㉓ 以下に接続してみましょう！
```
https://webapl1001.azurewebsites.net/
```
