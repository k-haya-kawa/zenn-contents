---
title: "Go言語のバリデーションチェック"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Zenn"]
published: false
---
# 目次

1. バリデーションチェックとは
2. 事前準備
3. 必須チェックの実装
4. 文字数チェックの実装
5. まとめ

# 1.バリデーションとは
バリデーションとは、「入力内容や記述内容が仕様に沿って、適切に記述されているかどうかを検証すること」と記載されています。
簡単に言うと、「入力チェック」になります。
入力チェックの実装方法にもいろいろな方法がありますので、その一つをご紹介します。

# 2.事前準備
　バリデーションチェックを実装するにあたり、簡単なwebページを作成しましょう。

:::details 2-1.ワークスペースの構成を下記のように作成する。
私は、バリデーションの動作確認もしたかったので、「C:\」に作成しました。

-----
> ★ワークスペースの構成
> go-validation
> ├── domain
> │   └── models.go
> ├── go.mod
> ├── internal
> │   └── forms
> │       ├── errors.go
> │       └── forms.go
> ├── main.go
> └── templates
>     ├── form.tmpl
>     └── result.tmpl
-----
:::

:::details 2-2.送信データを格納する構造体を定義する
```go:models.go
// models.go
package domain

type FormModel struct {
	Id string `json:"id"`
	Password string `json:"password"`
}

type TemplateData struct {
	Form map[string]interface{}
	Data map[string]interface{}
}
```
:::

:::details 2-3.Webサーバーとテンプレートファイルをブラウザへ返す処理を実装する
```go:main.go
// main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
	"validation/domain"
)

const portNumber = ":8080"

func Home(w http.ResponseWriter, r *http.Request) {
	if r.Method == "GET" {

		// 初期値は空を設定
		var emptyFormModel domain.FormModel
		data := make(map[string]interface{})
		data["validation"] = emptyFormModel

		renderTemplate(w, r, "form.tmpl", &domain.TemplateData{
			Form : nil,
			Data : data,
		})

	} else if r.Method == "POST" {
		err := r.ParseForm()
		if err != nil {
			log.Fatal(err)
			return
		}

		renderTemplate(w, r, "result.tmpl", &domain.TemplateData{})
	}
}

func main() {
	http.HandleFunc("/", Home)

	fmt.Println(fmt.Printf("Starting application on port %s", portNumber))
	log.Fatal(http.ListenAndServe(portNumber, nil))
}

// templateを返すメソッド
func renderTemplate(
	w http.ResponseWriter,
	r *http.Request,
	tmpl string,
	data *domain.TemplateData) {

	parsedTemplate, _ := template.ParseFiles("./templates/" + tmpl)

	// dataを格納
	err := parsedTemplate.Execute(w, data)

	if err != nil {
		fmt.Println("Error parsing template:", err)
		return
	}
}
```
:::

:::details 2-4.webページのフォームを作成する
```go:form.tmpl
<!-- form.tmpl -->
<!doctype html>
<html lang="ja">
    <head>
        <meta charset="utf-8">
        <title>GO言語 バリデーション</title>
        <style>
        body {
            width: 100%;
            text-align:center;
        }
        </style>
    </head>
    <body>
        <div class="container">

            {{$res := index .Data "validation"}}

             <h1>必須・桁数入力チェック</h1>
            <form method="post" novalidate>
                <div>
                    <label>チェックID</label>
                    <input
                     class="form-control
                     {{with.Form.Errors.Get "id"}}
                     is-invalid {{end}}"
                     type="text"
                     name="id"
                     required
                     >
                    {{with.Form.Errors.Get "id"}}
                        <label><font color="red">※チェックIDは{{.}}</font></label>
                    {{end}}
                </div>
                <div>
                    <label>パスワード</label>
                    <input
                     class="form-control
                     {{with .Form.Errors.Get "password"}}
                     is-invalid {{end}}"
                     type="password"
                     name="password"
                     required
                    >
                    {{with.Form.Errors.Get "password"}}
                        <label><font color="red">※パスワードは{{.}}</font></label>
                    {{end}}
                </div>
                <br>
                <button type="submit">入力チェック開始</button>
            </form>
        </div>
    </body>
</html>
```
:::

:::details 2-5.バリデーションチェックを通過した時に表示されるwebページを作成する
```go:result.tmpl
<!doctype html>
<html lang="ja">
    <head>
       <title>GO言語 バリデーション</title>
    </head>
    <body>
        <div class="container">
            <h1>入力チェック問題なし</h1>
        </div>
    </body>
</html>
```
:::

:::details 2-6.バリデーションチェックのエラー情報を格納するモジュールを実装する
errorsには、バリデーションチェック結果でNGの場合、フォームのフィールド名(id,password)とエラーメッセージを格納します。
エラーメッセージは複数格納される場合があるので、最初にNGになったエラーのみを返すようにする。
```go:errors.go
// errors.go
package forms

type errors map[string][]string

/*
	エラーを格納する
	fieldはform inputのname属性の値
	messageはエラーメッセージ
*/
func (e errors) Add(field, message string) {
	e[field] = append(e[field], message)
}

// エラーを取得する
func (e errors) Get(field string) string {
	es := e[field]
	if len(es) == 0 {
		return ""
	}

	return es[0]
}
```
:::

:::details 2-7.カスタムフォーム構造体とバリデーションメソッドを定義する
url.Valuesは、フォームからデータを送られてきた時、PostForm関数で取得できる戻り値の型です。
パースされたフォームの情報が入り、引数にはinput要素のname(id,password)がkeyとして入ります。
```go:forms.go
package forms

import (
	"net/url"
)

type Form struct {
	url.Values
	Errors errors
}

// コンストラクタ
func New(data url.Values) *Form {
	return &Form{
		data,
		// errors.goに設定したerrors
		errors(map[string][]string{}),
	}
}

// エラーの存在チェック
func (f *Form) Valid() bool {
	return len(f.Errors) == 0
}
```
:::

:::details 2-8.作成したwebページを確認する
1. コマンドプロンプトで「go-validation」のディレクトリに移動する。
![](/images/go/go-validation/check_1.png)
2. 「go run main.go」のコマンドを入力する。
![](/images/go/go-validation/check_2.png)
3. 「http://localhost:8080」にアクセスします。
![](/images/go/go-validation/check_3.png)
4. 何も入力しないで【入力チェック開始】ボタンを押下する。
5. 「result.tmpl」のページが表示される。
![](/images/go/go-validation/check_4.png)
:::

# 3.必須チェック
必須チェックの実装をしてみましょう。
:::details 3-1.フォームを作成したので、models.goのTemplateDataの「Form」を書き換える
```go:models.go
// models.go
package domain

import "validation/internal/forms"	// 追加

type FormModel struct {
	Id string `json:"id"`
	Password string `json:"password"`
}

type TemplateData struct {
	Form *forms.Form	// 修正
	Data map[string]interface{}
}
```
:::

:::details 3-2.必須チェックをforms.goに追加する
fieldsには、idやpasswordなどのname属性の値が入ります。

```go:forms.go
// forms.go
package forms

import (
	"net/url"
	"strings"	// 追加
)

type Form struct {
	url.Values
	Errors errors
}

// コンストラクタ
func New(data url.Values) *Form {
	return &Form{
		data,
		// errors.goに設定したerrors
		errors(map[string][]string{}),
	}
}

// エラーの存在チェック
func (f *Form) Valid() bool {
	return len(f.Errors) == 0
}

// 追加開始
/*
 必須チェック
*/
func (f *Form) Required(fields ...string) {
	for _, field := range fields {
		// url.Valuesオブジェクトを持っているので、
		// Getメソッドが使える
		value := f.Get(field)
		if strings.TrimSpace(value) == "" {
			// エラーを格納
			f.Errors.Add(field, "必須項目です")
		}
	}
}
// 追加終了
```
:::

:::details 3-3.バリデーションメソッドをmain.goに実装する
GET処理でrenderTemplateのFormの引数を変えています。


```go:main.go
// main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
	"validation/domain"
	"validation/internal/forms"	// 追加
)

const portNumber = ":8080"

func Home(w http.ResponseWriter, r *http.Request) {
	if r.Method == "GET" {

		// 初期値は空を設定
		var emptyFormModel domain.FormModel
		data := make(map[string]interface{})
		data["validation"] = emptyFormModel

		renderTemplate(w, r, "form.tmpl", &domain.TemplateData{
			Form: forms.New(nil), // 修正
			Data: data,
		})

	} else if r.Method == "POST" {
		err := r.ParseForm()
		if err != nil {
			log.Fatal(err)
			//return	削除
		}

		//renderTemplate(w, r, "result.tmpl", &domain.TemplateData{})	削除

		// 追加開始
		// Get関数でデータを取得する
		validation := domain.FormModel{
			Id:       r.Form.Get("id"),
			Password: r.Form.Get("password"),
		}

		// ログに出力
		fmt.Println("チェックID:", r.Form.Get("id"))
		fmt.Println("パスワード:", r.Form.Get("password"))

		// 追加
		// PostFormの戻り値はurl.url.Values
		// Postで送信されたデータをパースしている
		form := forms.New(r.PostForm)
		// formのinput要素のnameに指定した情報を送る
		form.Required("id", "password")
 
		// エラーかチェック
		if !form.Valid() {
			data := make(map[string]interface{})
			data["validation"] = validation
			renderTemplate(w, r, "form.tmpl", &domain.TemplateData{
				Form: form,
				Data: data,
			})
			return
		} else {
			renderTemplate(w, r, "result.tmpl", &domain.TemplateData{})
			return
		}
		// 追加終了
	}
}

func main() {
	http.HandleFunc("/", Home)

	fmt.Println(fmt.Printf("Starting application on port %s", portNumber))
	log.Fatal(http.ListenAndServe(portNumber, nil))
}

// templateを返すメソッド
func renderTemplate(
	w http.ResponseWriter,
	r *http.Request,
	tmpl string,
	data *domain.TemplateData) {

	parsedTemplate, _ := template.ParseFiles("./templates/" + tmpl)

	// dataを格納
	err := parsedTemplate.Execute(w, data)

	if err != nil {
		fmt.Println("Error parsing template:", err)
		return
	}
}
```
:::

:::details 3-4.必須チェックの実装を確認する
1. コマンドプロンプトで「go-validation」のディレクトリに移動する。
![](/images/go/go-validation/check_1.png)
2. 「go run main.go」のコマンドを入力する。
![](/images/go/go-validation/check_2.png)
3. 「http://localhost:8080」にアクセスします。
4. 何も入力しないで【入力チェック開始】ボタンを押下する。
![](/images/go/go-validation/check_3.png)
5. テキストボックスの横にエラーメッセージが表示されました。
![](/images/go/go-validation/required_1.png)
※ログで、入力値の確認ができます。
![](/images/go/go-validation/required_2.png)
6. 「ログインID：abcde、パスワード：12345」を入力し、【入力チェック開始】ボタンを押下する。
![](/images/go/go-validation/required_3.png)
7. 「result.tmpl」のページが表示される。
![](/images/go/go-validation/check_4.png)
※ログで、入力値の確認ができます。
![](/images/go/go-validation/required_4.png)
:::

# 4.文字数チェック
文字数チェックを実装してみましょう。

:::details 4-1.文字数チェックをforms.goに追加する
```go:forms.go
// forms.go
package forms

import (
	"net/url"
	"strings"
	"fmt"	// 追加
	"net/http"	// 追加

)

type Form struct {
	url.Values
	Errors errors
}

// コンストラクタ
func New(data url.Values) *Form {
	return &Form{
		data,
		// errors.goに設定したerrors
		errors(map[string][]string{}),
	}
}

// エラーの存在チェック
func (f *Form) Valid() bool {
	return len(f.Errors) == 0
}

/*
 必須チェック
*/
func (f *Form) Required(fields ...string) {
	for _, field := range fields {
		// url.Valuesオブジェクトを持っているので、
		//  Getメソッドが使える
		value := f.Get(field)
		if strings.TrimSpace(value) == "" {
			// エラーを格納
			f.Errors.Add(field, "必須項目です")
		}
	}
}

// 追加開始
/*
 最小文字数チェック
*/
func (f *Form) MinLength(field string, length int, r *http.Request) bool {
	x := r.Form.Get(field)
	if len(x) < length {
		f.Errors.Add(field, fmt.Sprintf(
			"%d文字以上で入力してください", length))
		return false
	}
	return true
}

/*
 最大文字数チェック
*/
func (f *Form) MaxLength(field string, length int, r *http.Request) bool {
	x := r.Form.Get(field)
	if len(x) > length {
		f.Errors.Add(field, fmt.Sprintf(
			"%d文字以内で入力してください", length))
		return false
	}
	return true
}
// 追加終了
```
:::

:::details 4-2.文字数チェックをmain.goに実装する
必須チェックと同じように、POST処理からメソッドを呼び出せるようにします。

```go:main.go
// main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"text/template"
	"validation/domain"
	"validation/internal/forms"
)

const portNumber = ":8080"

func Home(w http.ResponseWriter, r *http.Request) {
	if r.Method == "GET" {

		// 初期値は空を設定
		var emptyFormModel domain.FormModel
		data := make(map[string]interface{})
		data["validation"] = emptyFormModel

		renderTemplate(w, r, "form.tmpl", &domain.TemplateData{
			Form: forms.New(nil),
			Data: data,
		})

	} else if r.Method == "POST" {
		err := r.ParseForm()
		if err != nil {
			log.Fatal(err)
		}

		// Get関数でデータを取得する
		validation := domain.FormModel{
			Id:       r.Form.Get("id"),
			Password: r.Form.Get("password"),
		}

		// ログに出力
		fmt.Println("チェックID:", r.Form.Get("id"))
		fmt.Println("パスワード:", r.Form.Get("password"))

		// PostFormの戻り値はurl.url.Values
		// Postで送信されたデータをパースしている
		form := forms.New(r.PostForm)
		// formのinput要素のnameに指定した情報を送る
		form.Required("id", "password")
		// 追加開始
 		// idの最小文字数は5
		form.MinLength("id", 5, r)
		// passwordの最大文字数は10
		form.MaxLength("password", 10, r)
		// 追加終了

		// エラーかチェック
		if !form.Valid() {
			data := make(map[string]interface{})
			data["validation"] = validation
			renderTemplate(w, r, "form.tmpl", &domain.TemplateData{
				Form: form,
				Data: data,
			})
			return
		} else {
			renderTemplate(w, r, "result.tmpl", &domain.TemplateData{})
			return
		}
	}
}

func main() {
	http.HandleFunc("/", Home)

	fmt.Println(fmt.Printf("Starting application on port %s", portNumber))
	log.Fatal(http.ListenAndServe(portNumber, nil))
}

// templateを返すメソッド
func renderTemplate(
	w http.ResponseWriter,
	r *http.Request,
	tmpl string,
	data *domain.TemplateData) {

	parsedTemplate, _ := template.ParseFiles("./templates/" + tmpl)

	// dataを格納
	err := parsedTemplate.Execute(w, data)

	if err != nil {
		fmt.Println("Error parsing template:", err)
		return
	}
}
```
:::

:::details 4-3.文字数チェックの実装を確認する
1. コマンドプロンプトで「go-validation」のディレクトリに移動する。
![](/images/go/go-validation/check_1.png)
2. 「go run main.go」のコマンドを入力する。
![](/images/go/go-validation/check_2.png)
3. 「http://localhost:8080」にアクセスします。
![](/images/go/go-validation/check_3.png)
4. 「ログインID：abc、パスワード：12345678901」を入力し、【入力チェック開始】ボタンを押下する。
![](/images/go/go-validation/count_1.png)
5. テキストボックスの横にエラーメッセージが表示されました。
![](/images/go/go-validation/count_2.png)
※ログで、入力値の確認ができます。
![](/images/go/go-validation/count_3.png)
6. 「ログインID：abcde、パスワード：123456789」を入力し、【入力チェック開始】ボタンを押下する。
![](/images/go/go-validation/count_4.png)
7. 「result.tmpl」のページが表示される。
![](/images/go/go-validation/check_4.png)
※ログで、入力値の確認ができます。
![](/images/go/go-validation/count_5.png)
:::

# 5.まとめ
今回は、バリデーションチェック（必須チェックと文字数チェック）について、学びました。
入力チェックには、他にもいろいろ種類や実装方法があります。
これをきっかけに、いろいろなバリデーションチェックの方法について学んでいこうと思います。
