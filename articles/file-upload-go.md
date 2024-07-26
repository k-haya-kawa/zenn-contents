---
title: "Goのファイルアップロード処理"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Zenn"]
published: false
publication_name: "toccasystems"
---

# 1.アップロードとは
手元のPCなどの機器から、ネットワークを介して、別のPCやファイルサーバー、webサーバーなどに、画像やHTMLなどのファイルやデータなどを転送することです。
※今回は、画像（JPEG、PNG）をアップロードする実装方法の一つをご紹介します。

# 2.事前準備
ファイルアップロードを実装するにあたり、ファイルアップロードが出来るwebページを作成しましょう。

# 2-1.ワークスペースの構成
私は、ファイルアップロードの動作確認もしたいので、「C:\」に作成しました。

-----
> go-file-upload
> ├── file
> │   └── upload.go
> ├── main.go
> └── index.html
-----

# 2-2.プロジェクトの初期化
goモジュールを使用するために、「go-file-upload」プロジェクトを初期化します。
1. コマンドプロンプトで「go-file-upload」のディレクトリに移動する。
![](/images/go/go-file-upload/check_1.png)
2. 「go mod init upload」のコマンドを入力し、「go creating nre go mod: module upload」と表示されればOKです。
![](/images/go/go-file-upload/check_2.png)
3. 「go-file-upload」のディレクトリ直下に、「go.mod」ファイルが作成されます。
![](/images/go/go-file-upload/check_3.png)

# 2-3.ファイルアップロード用のwebページ作成
```go:index.html
<!-- index.html -->
<!DOCTYPE html>
<html lang="ja">
    <head>
        <meta charset="UTF-8">
        <title>Go ファイルアップロード</title>
        <style>
            body {
                width: 100%;
            text-align:center;
            }
        </style>
    </head>
    <body>
        <h1>ファイルアップロード</h1>
        <form id="form" enctype="multipart/form-data" action="/upload" method="POST">
            <input type="file" name="file" class="input file-input" multiple>
            <button class="button" type="submit">アップロード</button>
        </form>
    </body>
</html>
```

# 2-4.エンドポイントとwebページの表示
```go:main.go
// main.go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func indexHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Add("Content-Type", "text/html")
    http.ServeFile(w, r, "index.html")
}

func uploadHandler(w http.ResponseWriter, r *http.Request) {
}

func setupRoutes() {
    mux := http.NewServeMux()
    mux.HandleFunc("/", indexHandler)
    mux.HandleFunc("/upload", uploadHandler)

    if err := http.ListenAndServe(":8080", mux); err != nil {
        log.Fatal(err)
    }
}

func main() {
    fmt.Println("ファイルアップロード開始")
    setupRoutes()
}
```

# 2-5.動作確認
1. コマンドプロンプトで「go-file-upload」のディレクトリに移動する。
![](/images/go/go-file-upload/check_1.png)
2. 「go run main.go」のコマンドを入力する。
![](/images/go/go-file-upload/check_4.png)
3. 「http://localhost:8080」にアクセスする。
![](/images/go/go-file-upload/check_5.png)

# 3.ファイルアップロード処理
ファイルサイズの制限、ファイルの取得から保存、そして、ファイルタイプの制限について、実装してみましょう。

# 3-1.ファイルサイズ制限の実装
goでは、リクエストボディのサイズを制限するために「http.MaxByteReader()」を使用します。
また、フォームデータが「multipart/form-data」のため、「ParseMultipartForm()」を使用してフォームデータを取得します。
```go:upload.go
// upload.go
package file

import "net/http"

const MaxUploadSize = 1024 * 1024   // 最大ファイルサイズ

func UploadHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != "POST" {
        http.Error(w, "処理を終了します。", http.StatusMethodNotAllowed)
        return
    }

    r.Body = http.MaxBytesReader(w, r.Body, MaxUploadSize)
    if err := r.ParseMultipartForm(MaxUploadSize); err != nil {
        http.Error(w, "1MB以下のファイルを選択してください。", http.StatusBadRequest)
    }
}
```

# 3-2.アップロードしたファイルの取得と保存
```go:upload.go
// upload.go
package file

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"time"
)

const MaxUploadSize = 1024 * 1024 // 最大ファイルサイズ

func UploadHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "処理を終了します。", http.StatusMethodNotAllowed)
		return
	}

	r.Body = http.MaxBytesReader(w, r.Body, MaxUploadSize)
	if err := r.ParseMultipartForm(MaxUploadSize); err != nil {
		http.Error(w, "1MB以下のファイルを選択してください。", http.StatusBadRequest)
	}

	// ここから追加
	// 1. フォームのファイルを取得する
	file, fileHeader, err := r.FormFile("file")
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	defer file.Close()

	// 2. 保存用のディレクトリを作成する（存在していなければ、保存用のディレクトリを新規作成）
	err = os.MkdirAll("./uploadfiles", os.ModePerm)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	// 3. 保存するファイルを作成する
	dst, err := os.Create(fmt.Sprintf("./uploadfiles/%d%s", time.Now().UnixNano(), filepath.Ext(fileHeader.Filename)))
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	defer dst.Close()

	// アップロードしたファイルを保存用のディレクトリにコピーする
	_, err = io.Copy(dst, file)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	fmt.Fprintf(w, "アップロード成功！")
	// ここまで追加
}
```

# 3-3.ファイルタイプを「JPEG、PNG」のみに制限
goで、データ形式を識別するためのMIMEタイプを判定する方法として「DetectContentType()」を使用します。

:::details DetectContentType()
MIMEタイプを決定するために、最初の512バイトのデータを考慮します。
コンテンツタイプを判別するために、アップロードされたファイルの最初から512バイトを考慮すると、ファイルの位置が512バイト分進みます。
「io.Copy()」を実行した時、512バイト進んだ位置から読み取りを実行するため、アップロードした画像ファイルは破損します。
そのため、「file.Seek()」を使用し、ファイルの位置を「0」からに戻します。
:::
```go:upload.go
// upload.go
package file

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"time"
)

const MaxUploadSize = 1024 * 1024 // 最大ファイルサイズ

func UploadHandler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "処理を終了します。", http.StatusMethodNotAllowed)
		return
	}

	r.Body = http.MaxBytesReader(w, r.Body, MaxUploadSize)
	if err := r.ParseMultipartForm(MaxUploadSize); err != nil {
		http.Error(w, "1MB以下のファイルを選択してください。", http.StatusBadRequest)
	}

	// 1. フォームのファイルを取得する
	file, fileHeader, err := r.FormFile("file")
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	defer file.Close()

	// 2. 保存用のディレクトリを作成する（存在していなければ、保存用のディレクトリを新規作成）
	err = os.MkdirAll("./uploadfiles", os.ModePerm)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	// ここから追加
	buff := make([]byte, 512)
	_, err = file.Read(buff)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	filetype := http.DetectContentType(buff)
	if filetype != "image/jpeg" && filetype != "image/png" {
		http.Error(w, "JPEG、または、PNGでアップロードしてください。", http.StatusBadRequest)
	}

	_, err = file.Seek(0, io.SeekStart)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	// ここまで追加

	// 3. 保存するファイルを作成する
	dst, err := os.Create(fmt.Sprintf("./uploadfiles/%d%s", time.Now().UnixNano(), filepath.Ext(fileHeader.Filename)))
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	defer dst.Close()

	// 4. アップロードしたファイルを保存用のディレクトリにコピーする
	_, err = io.Copy(dst, file)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}

	fmt.Fprintf(w, "アップロード成功！")
}
```

# 3-4.ファイルアップロード処理をmain.goにimport
main.goにfileパッケージをimportします。
```go:main.go
// main.go
package main

import (
	"fmt"
	"log"
	"net/http"
	"upload/file" //追加
)

func indexHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Add("Content-Type", "text/html")
	http.ServeFile(w, r, "index.html")
}

func uploadHandler(w http.ResponseWriter, r *http.Request) {
	file.UploadHandler(w, r) //追加
}

func setupRoutes() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", indexHandler)
	mux.HandleFunc("/upload", uploadHandler)

	if err := http.ListenAndServe(":8080", mux); err != nil {
		log.Fatal(err)
	}
}

func main() {
	fmt.Println("ファイルアップロード開始")
	setupRoutes()
}
```

# 3-5.動作確認
1. コマンドプロンプトで「go-file-upload」のディレクトリに移動する。
![](/images/go/go-file-upload/check_1.png)
2. 「go run main.go」のコマンドを入力する。
![](/images/go/go-file-upload/check_4.png)
3. 「http://localhost:8080」にアクセスする。
![](/images/go/go-file-upload/check_5.png)
4. 「ファイル選択」ボタンを押下し、ファイルを選択する。（画面にファイル名が表示されること）
![](/images/go/go-file-upload/check_6.png)
![](/images/go/go-file-upload/check_7.png)
5. 「アップロード」ボタンを押下する。（画面に「アップロード成功！」が表示されること）
![](/images/go/go-file-upload/check_8.png)
6. ワークスペースに「uploadfiles」が作成される。
![](/images/go/go-file-upload/check_9.png)
7. 「uploadfiles」にファイルがアップロードされる。
![](/images/go/go-file-upload/check_10.png)

# 4.まとめ
今回は、ファイルアップロード処理の実装について、学びました。
他にも、パッケージを使用した実装方法やフレームワークを利用した方法などがあるので、機会があれば、勉強しようと思いました。

最後まで、ご覧いただきましてありがとうございます。
