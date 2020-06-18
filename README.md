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
docker ps
```
Creating docker_go-master_app_1 ... done <br>
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
④と同じ階層で実行する。
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
# ⑨ main.goファイルを編集し、googleのtopページのスクリーンショットを撮る
※/docker_go-master配下にscreenshotフォルダを作成しておく。<br>
![キャプチャ](https://user-images.githubusercontent.com/66953939/84674597-1dd14380-af66-11ea-8147-bc97fa0dc019.png)
<br>
```
package main

import (
	"github.com/sclevine/agouti"
	"log"
)

func main() {
	// Chromeを利用することを宣言
	driver := agouti.ChromeDriver(
		agouti.ChromeOptions("args", []string{
			"--headless",
			"--disable-gpu",
			"--window-size=1280,1024",
			"--disable-dev-shm-usage",
			"--no-sandbox",
		}),
		agouti.Debug,
	)

	if err := driver.Start(); err != nil {
		log.Printf("Failed to start driver: %v", err)
	}
	defer driver.Stop()

	page, err := driver.NewPage(agouti.Browser("chrome"))
	if err != nil {
		log.Printf("Failed to open page: %v", err)
	}

	// Access to a target page
	url := "https://www.google.co.jp/"
	err = page.Navigate(url)
	if err != nil {
		log.Printf("Failed to navigate: %v", err)
	}
	// Get screen shot
	page.Screenshot("screenshot/Google.png")
}
```
<br><br>
# ⑩ コンテナ内でgoファイルを実行
```
go run main.go
```
下記のようなエラーが発生した場合はmain.goファイルの文字コードをUTF-8に変更してください。<br>
./main.go:9:11: invalid UTF-8 encoding
<br><br>

# ⑪ スクリーンショットの確認
/docker_go-master配下に作成したscreenshotフォルダにGoogle.pngが作成されていることを確認<br>
![キャプチャ](https://user-images.githubusercontent.com/66953939/84678989-b1594300-af6b-11ea-808f-6f32a12a66fa.png)
<br><br>

# ⑫ main.goファイルを編集し、ANAのページで予約を自動化していく・・・1
⑨で編集した「page.Screenshot("screenshot/Google.png")」に続けてコードを記載する。<br>
```
	// 自動操作
	//ANA日本語ページ遷移
	page.Navigate("https://www.ana.co.jp/ja/jp")
	log.Printf(page.Title())
        page.Screenshot("screenshot/ana-top.png")
	//検索ボタン押下
	page.FirstByName("arrivalAirport").Submit()
	page.Screenshot("screenshot/ana-1.png")
	//区間検索「片道」押下
	page.FindByID("hoge1").Click()
	page.Screenshot("screenshot/ana-2.png")
```
<br><br>

# ⑬ ブラウザでANAの区間検索ページで要素を検索して、片道のIDを取得し"hoge1"を書き換える
URL:https://www.ana.co.jp/ja/jp <br>
上記URLで検索ボタンを押下すると、区間ページへ遷移できる。
<br><br>

![キャプチャ](https://user-images.githubusercontent.com/66953939/84682510-b240a380-af70-11ea-9aaa-381d7f67df82.png)
```
page.FindByID("hoge1").Click()
↓
page.FindByID("buttonOneWay").Click()
```
<br><br>
# ⑭ main.goファイルを編集し、ANAのページで予約を自動化していく・・・2
⑫で編集した「page.Screenshot("screenshot/ana-2.png")」に続けてコードを記載する。<br>
```
	//到着地「札幌」選択
	page.FindByID("arrivalAirport").Select("札幌(千歳)")
	page.Screenshot("screenshot/ana-3.png")
	//カレンダーテキスト押下
	page.FindByID("outwardEmbarkationDate").Click()
	page.Screenshot("screenshot/ana-4.png")
	//カレンダーでXpathを指定して8月10日を指定
	page.FirstByXPath("hoge2").Click()
	page.Screenshot("screenshot/ana-5.png")
	//最安値指定
	page.FirstByLabel("最安運賃を検索").Click()
	page.Screenshot("screenshot/ana-6.png")
	//検索ボタン押下
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[2]/div[4]/p/input").Click()
	page.Screenshot("screenshot/ana-7.png")
	//値段を押下
	page.FirstByLabel("hoge3").Click()
	page.Screenshot("screenshot/ana-8.png")
	//確認ボタン押下
	page.FirstByName("j_idt331").Click()
	page.Screenshot("screenshot/ana-9.png")
	//一般の方押下
	page.FirstByName("j_idt318").Click()
	page.Screenshot("screenshot/ana-10.png")
```
# ⑮ 要素を検索して、搭乗日8/10のXpathと値段のラベルを取得し"hoge2"と"hoge3"を書き換える
Xpathの要素検索<br>
下記図の①→②→③の順番で取得する。<br>
![キャプチャ](https://user-images.githubusercontent.com/66953939/84685535-6f34ff00-af75-11ea-8d6e-4ff2b8d3893e.png)
<br>
```
page.FirstByXPath("hoge2").Click()
↓
page.FirstByXPath("/html/body/div[9]/div/div/div/div/div[3]/table/tbody/tr[3]/td[2]/a").Click()
```
<br>
ラベルの検索  

![a](https://user-images.githubusercontent.com/66953939/84686408-d8694200-af76-11ea-8e3b-551d611ba86d.png)
<br>

```
page.FirstByLabel("hoge3").Click()
↓
page.FirstByLabel("18,860円").Click()
```
<br><br>

# ⑯ main.goファイルを編集し、ANAのページで予約を自動化していく・・・3
⑭で編集した「page.Screenshot("screenshot/ana-10.png")」に続けてコードを記載する。<br>
```
	//お客様情報入力
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[2]/input").Fill("ソラノ")
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[3]/input").Fill("タロウ")
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[4]/input").Fill("25")
	page.FirstByLabel("男性").Click()
	page.FirstByName("hoge4").Fill()
	page.FirstByName("assistMailAddress").Fill("test@test.test.test")
	page.FirstByName("assistConfirmMailAddress").Fill("test@test.test.test")
	page.Screenshot("screenshot/ana-11.png")
```
# ⑰ 要素を検索して、電話番号のName属性を取得し"hoge4"を書き換え、Fill()内に電話番号を記載する。
```
page.FirstByName("hoge4").Fill()
↓
page.FirstByName("telNo").Fill("123-456-7890")
```

<br><br>
# ⑱ ⑰の作業まで完了後再度コンテナ内でgoファイルを実行する。
```
go run main.go
```
<br><br>
# ⑲ screenshotフォルダ内にana-11.pngまでのキャプチャが撮れていることを確認する。
<br><br>
# ⑳ 完成コード全量
```
package main

import (
	"github.com/sclevine/agouti"
	"log"
)

func main() {
	// Chromeを利用することを宣言
	driver := agouti.ChromeDriver(
		agouti.ChromeOptions("args", []string{
			"--headless",
			"--disable-gpu",
			"--window-size=1280,1024",
			"--disable-dev-shm-usage",
			"--no-sandbox",
		}),
		agouti.Debug,
	)

	if err := driver.Start(); err != nil {
		log.Printf("Failed to start driver: %v", err)
	}
	defer driver.Stop()

	page, err := driver.NewPage(agouti.Browser("chrome"))
	if err != nil {
		log.Printf("Failed to open page: %v", err)
	}

	// Access to a target page
	url := "https://www.google.co.jp/"
	err = page.Navigate(url)
	if err != nil {
		log.Printf("Failed to navigate: %v", err)
	}
	// Get screen shot
	page.Screenshot("screenshot/Google.png")

	// 自動操作
	//ANA日本語ページ遷移
	page.Navigate("https://www.ana.co.jp/ja/jp")
	log.Printf(page.Title())
	page.Screenshot("screenshot/ana-top.png")
	//検索ボタン押下
	page.FirstByName("arrivalAirport").Submit()
	page.Screenshot("screenshot/ana-1.png")
	//区間検索「片道」押下
	page.FindByID("buttonOneWay").Click()
	page.Screenshot("screenshot/ana-2.png")
	//到着地「札幌」選択
	page.FindByID("arrivalAirport").Select("札幌(千歳)")
	page.Screenshot("screenshot/ana-3.png")
	//カレンダーテキスト押下
	page.FindByID("outwardEmbarkationDate").Click()
	page.Screenshot("screenshot/ana-4.png")
	//搭乗日カレンダーで8月10日を指定
	page.FirstByXPath("/html/body/div[9]/div/div/div/div/div[3]/table/tbody/tr[3]/td[2]/a").Click()
	page.Screenshot("screenshot/ana-5.png")
	//最安値指定
	page.FirstByLabel("最安運賃を検索").Click()
	page.Screenshot("screenshot/ana-6.png")
	//検索ボタン押下
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[2]/div[4]/p/input").Click()
	page.Screenshot("screenshot/ana-7.png")
	//値段を押下
	page.FirstByLabel("18,860円").Click()
	page.Screenshot("screenshot/ana-8.png")
	//確認ボタン押下
	page.FirstByName("j_idt331").Click()
	page.Screenshot("screenshot/ana-9.png")
	//一般の方押下
	page.FirstByName("j_idt318").Click()
	page.Screenshot("screenshot/ana-10.png")

	//お客様情報入力
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[2]/input").Fill("ソラノ")
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[3]/input").Fill("タロウ")
	page.FirstByXPath("/html/body/div[4]/div/div[1]/form/div[1]/table/tbody/tr/td[4]/input").Fill("25")
	page.FirstByLabel("男性").Click()
	page.FirstByName("telNo").Fill("123-456-7890")
	page.FirstByName("assistMailAddress").Fill("test@test.test.test")
	page.FirstByName("assistConfirmMailAddress").Fill("test@test.test.test")
	page.Screenshot("screenshot/ana-11.png")
	//  page.FindByID("m_ticket02").Click()
	// log.Printf(page.Title())
}
```
