# simple-db

[Database Design and Implementation](https://link.springer.com/book/10.1007/978-3-030-33836-7) を読みながら DB の実装をする

# 本のまとめ

## 第１章 Database System

### データベースの 5 つの基本機能
* 永続的でなければならない
* 共有できること
* 正確に保たれること
* 非常に大きくなること
* 使いやすいこと

### データベースの機能まとめ
* ファイルシステムでただ保存するよりも効率的にアクセスできる形式を使用して、ファイルにレコードを保存する機能
* ファイル内のデータにインデックスを付けるためのアルゴリズムを使用することで、高速なアクセスを可能にする機能
* ネットワーク経由で複数のユーザーからの同時アクセスを処理し、必要に応じて排他制御を行う機能
* 変更のコミットやロールバックを実施する機能
* データベースのレコードをインメモリにキャッシュしながらも、システムがクラッシュした際にデータを失わない機能
* テーブルに対するユーザークエリをファイル上の実行可能なコードに変換する機能
* 非効率なクエリを効率的なクエリに変換するための最適化機能

参考文献
https://en.wikipedia.org/wiki/Database#History

## 第2章 JDBC

### JDBCの基本機能は以下の5つのインターフェースにより実装されている
* Driver・・クライアントがDBに接続するためのもの
* Connection・・クライアントがURLとプロパティを用いて実際にDBに接続するときのもの
* Statement・・SQLクエリを実行するためのもの
* ResultSet・・SQLクエリの結果を返すもの。ResultSetはリソースを拘束する事になるのでクローズをして解放する必要がある。
* ResultSetMetadata・・各フィールドの名前、タイプ、表示サイズを定義しており、このインターフェースを通じて利用可能

#### java の try-with-resources

tryブロックが終了した時に自動的にクローズする
```java
try (Connection conn = d.connect(url, null)) {  
  System.out.println("Database Created");  
} catch (SQLException e) {
  e.printStackTrace();  
}
```

#### DriverManager
DriverManagerを使用することでドライバをクライアントから隠蔽することが出来る。
ドライバをコレクションに追加しておき、必要な時に接続文字列を渡すことでドライバを取得できる。
JDBCではドライバ情報をJavaの設定ファイルに記述することで、クライアントコードからドライバ情報をなくすことが出来る。

#### DataSource
DataSourceを使用することで、ドライバと接続文字列をクライアントから隠蔽することが出来る。
各データベースベンダーが用意しているDataSourceを実装したクラスを使用することで、クライアントは必要な値を指定するだけでDBに接続できる。

### 4つのトランザクション分離レベル
* Read-Uncommitted・・トランザクションが全く分離されておらず、他のトランザクションで更新されたデータが見える。
* Read-Committed・・トランザクションがコミットされていない値にアクセスすることを禁止する。繰り返されない読み取りやファントムに関する問題は依然として起こる。
* Repeatable-Read・・Read-Committedを拡張し、一度読み取りを行うと常に同じ結果が繰り返されるようになる。ただし、ファントムの問題は残る。(mysqlではMVCCによりファントムの問題を防いでいる)
* Serializable・・トランザクションを順序付けて処理する。データを安全に操作できるが並列性が失われる。

#### クライアント vs SQL
ほとんどの場合、クライアント側でコードを書くよりSQLクエリで表現してDBに計算させる方が見た目もよく効率がいい。

## 第2章 ディスクとファイル管理
#### ディスクアクセスに必要な時間
シーク時間(ディスクヘッドを指定されたトラックに移動させる時間) + 
回転遅延時間(最初のバイトがディスクヘッドの下に来るまでにプラッターが回転する時間) +
転送時間(最初のバイトの読み出しから最後のバイトの読み出しまで)

シーク時間や回転遅延時間などの機械的な動きは電気的な動きよりかなり遅く、これがディスクドライブがフラッシュメモリより非常に遅い理由(HDD, SSDの違い)
ただしフラッシュメモリもまだRAMよりはかなり遅いためOSからはフラッシュメモリもディスクドライブも同じように扱っている

#### ディスクアクセスの改善
* ディスクキャッシュ・・あるセクタを読み込んだ時に、トラックにある情報をキャッシュしておく
* シリンダー・・関連する情報は近くのセクタに書き込んでおく
* ディスクストライピング・・複数のディスクドライブを使用した時に、システム側からは一つの大きなディスクであるように見せかける(仮想ディスク)

#### パリティ情報
ある情報のbit列の「1」の個数が奇数場合、パリティビットは「1」、偶数の場合は「0」となる
パリティ情報をディスクのバックアップにすると、バックアップ用のディスクが1つで済むようになる。ただし、複数のディスクが破損すると復元不可になる。

### ブロックとページ
OSはディスクドライブやフラッシュドライブの物理的な詳細をブロックベースのインタフェースを提供することでクライアントから隠蔽している
ページはメモリ上のブロックサイズの領域である。クライアントはブロックの内容をページに読み込んで変更し、そのページをブロックに書き戻すことでブロックを変更する。
(ハード側とはブロックと呼ばれる単位でやりとりをして、それをメモリに載せるとページと呼ばれる単位になるみたいなイメージ？)

