---
title: "Webセキュリティへの対応（SQLインジェクション攻撃）"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Zenn"]
published: false
---

# 1.Webセキュリティへの対応（SQLインジェクション攻撃）
今回は、「SQLインジェクション攻撃」への対応方法について考えてみましょう。
代表的な対応方法としては、下記のようなものがあります。

 - バリデーションチェックの実施
 - プレ－スホルダの利用
 - エスケープ処理、サニタイジングの利用
 - OSやアプリケーションは常に最新化
 - WAF（Web Application Firewall）の導入

それでは、対応方法について、見ていきましょう！

# 2.バリデーションチェックの実施
バリデーションチェックとは、簡単に言うと入力チェックのこと。
バリデーションチェックを実装することで、予期しないスクリプトの実行を防ぐことが出来ます。

 - **バリデーションチェックとしては、下記のような方法があります。**
:::message
・文字種チェック
　実用例：入力フィールド（年齢）は、数字のみ可能。
・桁数チェック
　実用例：メールアドレスは、300文字まで可能。
:::

 - **Javaの場合、下記のようなチェックメソッドを実装したりします。**
```java
// 整数チェックメソッド
if (value != null) {
  Pattern pattern = Pattern.compile("^[0-9]+$|-[0-9]+$");
  result = pattern.matcher(value).matches();
}
```

# 3.プレ－スホルダの利用
プレースホルダ（place holder）とは、SQL文の中で値を入れたい個所に記号を置き、後にその記号に実際の値を割り当てることです。
後から実際に扱う値を入れる前に、一時的に場所を確保するために入れられる仮の値のこと。

 - **プレースホルダの利用としては、下記のような方法があります。**
:::message
下記の「?」がプレースホルダになる。
select user_id, pass from Account where user_id=?
:::

 - **Javaの場合、「PreparedStatement」を使用し、下記のように実装します。**
```java
Connection conn = null;

try
{
    conn = DriverManager.getConnection(DBのURL, ユーザID, パスワード);

    //実行するSQL文にプレースホルダを設定
    String sql = "SELECT * FROM テーブル名 WHERE id = ?"

    //実行したいSQLからPreparedStatement型のSQL文を取得する
    PreparedStatement ps = conn.prepareStatement(sql);

    //「?」の部分に、String型の値を設定する
    ps.setString(1, "test");

    //SQL文の実行
    ps.executeQuery();
}
catch (SQLException e)
{
  System.out.println("SQLException:" + e.getMessage());
}
```


# 4.エスケープ処理、サニタイジングの利用
エスケープ処理とは、特別な意味を持つ文字列を、無害な別の文字列に変換すること。
エスケープ処理は、エスケープシーケンスと呼ばれる「\(バックスラッシュ)」と何らかの記号や文字で表現します。

サニタイジングとは、害悪な文字列がないかをチェックし、特定の文字に置き換えて無害化すること。

 - **エスケープ処理としては、下記のような方法があります。**
:::message
| エスケープ文字 | エスケープの意味 |
| ---- | ---- |
| \b | バックスペース |
| \t | 水平タブ |
| \n | 改行 |
| \r | 復帰 |
| \' | シングルクォーテーション |
:::

 - **Javaの場合、下記のように実装します。**
```java
// エスケープシーケンスを使用しているため「改行のエスケープシーケンスは¥nになる」と出力する
System.out.println("改行のエスケープシーケンスは¥¥nになる");
```

**サニタイジングの利用としては、下記のような方法があります。**
:::message
| 無害化前 | 無害化後 |
| ---- | ---- |
| < | &lt; |
| > | &gt; |
:::

- **Javaの場合、下記のように実装します。**
```java
private String sanitizing( String string )
{
    // 変換処理
    string = string.replaceAll( "<", "&lt;" );
    string = string.replaceAll( ">", "&gt;" );
    return string;
}
```

# 5.OSやアプリケーションは常に最新化
OSやアプリケーションに最新のバージョンが公開されたら、最新のバージョンへ更新すること。
また、ツールやプラグインなども同様に、最新の状態を保つようにしましょう。

OSやアプリケーションのバージョンアップの中身は、ほとんどが脆弱性対策です。
最新化することで、リスクを下げることが出来ます。

# 6.WAF（Web Application Firewall）の導入
WAFはファイアーウォールやIPS/IDSでは防ぐことができない、Webアプリケーションの脆弱性を悪用したサイバー攻撃を防ぐことができます。
特徴としては、通信内容をアプリケーションレベルで検査できること。

※IPS：侵入防止システム（Intrusion Prevention System）
　IDS：侵入検知システム（Intrusion Detection System）

# 7.まとめ
SQLインジェクションの被害としては、「情報漏洩」や「Webサイトの改ざん」があります。
被害に遭わないように、しっかりと対応していきましょう！！
