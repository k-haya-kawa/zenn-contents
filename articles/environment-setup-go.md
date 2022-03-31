---
title: "Go言語の開発環境を構築してみた（Windows編）"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Zenn"]
published: false
---
:::details 1.GO言語とは
　Go言語とは、検索エンジンとして馴染み深いGoogleが開発したプログラミング言語のこと。

「シンプルかつ高速な処理が可能なプログラミング言語」で、有名なWebサービスの開発や
サーバの構築などにも用いられています。

シンプルな構文である分「誰が読んでも分かるプログラムを書きやすい」という特徴があります。

そのため、複数人のエンジニアで並行してコーディングが行いやすく、
用途を問わず「規模の大きいシステム開発に最適な言語」と言えます。

処理速度の速さも相まって、同時にコーディングをしても作業効率が落ちにくい点は、
Go言語の大きな利点になります。
:::

:::details 2.Go言語の開発環境を構築するにあたってインストールするツール
・Go SDK 1.14.4
・Visual Studio Code 1.65.2
・Git
:::

:::details 3.Go SDK 1.14.4のインストール
　1. Go SDKをダウンロードする。（今回はWindows）
　 ※ダウンロード先：https://go.dev/dl/

![](/images/go/go-install/go-install_1.png)

　2. ダウンロードした「go1.17.8.windows-amd64.msi」をインストールする。

![](/images/go/go-install/go-install_2.png)

　3. 「I accept the terms in the License Agreement」にチェックがされていることを確認し、
　 「Next」を押下する。

![](/images/go/go-install/go-install_3.png)

　4. インストール先のディレクトリを確認し、「Next」を押下する。

![](/images/go/go-install/go-install_4.png)

　5. 「Install」を押下する。（管理者権限を求められることもあります。）

![](/images/go/go-install/go-install_5.png)

　6. 「Finish」を押下する。

![](/images/go/go-install/go-install_6.png)

 　7. Go SDKがインストールされたかを確認するため、コマンドプロンプトを起動し、
　「go version」と入力する。
 　（コンソールに「go version go1.17.8 windows/amd64」と表示されれば、OK！）

 ![](/images/go/go-install/go-install_7.png)
:::

:::details 4.Visual Studio Codeの設定
　1. Visual Studio Codeは、下記のサイトを参考にインストールまで行ってください。
　 ※参考：https://sukkiri.jp/technologies/devtools/vscode_win.html

 　2. Visual Studio Code（Code.exe）を起動する。

![](/images/go/go-edit/go-edit_1.png)

　3. 下記の手順でGoを検索する。
　　①. Extensionの検索画面を開く。
　　②. 「go」で検索する。
　　③. [Go Team at Google]のGoエクステンションを選択する。

![](/images/go/go-edit/go-edit_2.png)

　4. 「インストール」を押下する。

![](/images/go/go-edit/go-edit_3.png)

　5. インストールが完了すると「無効にする」「アンインストール」が表示される。

![](/images/go/go-edit/go-edit_4.png)

　6. 今回は、Cドライブ配下に「go_test」フォルダを作成し、「hello.go」ファイルを作成する。
 
![](/images/go/go-edit/go-edit_5.png)

　7. Visual Studio Codeで、hello.goを下記のように編集する。

![](/images/go/go-edit/go-edit_6.png)

　8. コマンドプロンプトで「go_test」に移動して、 
　「go run hello.go」コマンドを入力して実行する。
 　（「Hello!」と表示されれば成功です。）

![](/images/go/go-edit/go-edit_7.png)
:::

:::details 5.Gitの設定
　1. Gitは、下記のサイトを参考にインストールまで行ってください。
　 ※参考：https://sukkiri.jp/technologies/devtools/git/git_win.html
 
　2. 今回は、Cドライブ配下に「go_local」フォルダを作成し、「go.mod」ファイルを作成する。
　（※先程の「go_test」のフォルダとは別にすること。）

![](/images/go/go-git/go-git_1.png)

　3. Visual Studio Codeで、go.modを下記のように編集する。

![](/images/go/go-git/go-git_2.png)

　4. コマンドプロンプトでgo_localに移動して、
　「go get github.com/labstack/echo/v4@v4.1.16」コマンドを実行する。
　 下記のように表示されれば成功です。

![](/images/go/go-git/go-git_3.png)

　5. 「go.mod」ファイルを確認する。 
　 「require github.com/labstack/echo/v4 v4.1.16 // indirect」の記述が追加されている。
 
![](/images/go/go-git/go-git_4.png)

　6. 「go_local」フォルダに「server.go」ファイルを作成する。

![](/images/go/go-git/go-git_5.png)

　7. Visual Studio Codeで、「server.go」を下記のように編集する。

![](/images/go/go-git/go-git_6.png)

　8. コマンドプロンプトで「go_local」に移動して、 「go run server.go」コマンドを実行する。
　 Windowsセキュリティの重要な警告が表示された場合は、[アクセスを許可する]を押下する。
　 次のように表示されれば成功です。

![](/images/go/go-git/go-git_7.png)

　9. ブラウザで「 http://localhost:1323 」にアクセスする。
 　「Hello!」と表示されれば成功です。

![](/images/go/go-git/go-git_8.png)

　10. 停止させる場合は、実行しているコマンドプロンプトで「Ctrl + C」を押下する。
 　　Echoが停止します。

![](/images/go/go-git/go-git_9.png)
:::

:::details 6.まとめ
Goの開発環境について、実際にやってみたところ、詰まったりすることなく、構築出来ました。
次回は、Goの統合開発環境の「Lite IDE」の環境構築をしていこうと思います。
最後まで、読んでいただきまして、ありがとうございます。
:::
