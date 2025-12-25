##### Other languages: [简体中文](/CHN/CHN-08-1-数据库-DbClient)

# Database - DbClient

### DbClient オブジェクトの作成

DbClientの作成には2つ方法があります。一つはDbClientクラスの静的メソッドを呼ぶ方法です。その定義についてはDbClient.hで次のように書かれています。

```c++
#if USE_POSTGRESQL
    static std::shared_ptr<DbClient> newPgClient(const std::string &connInfo, const size_t connNum);
#endif
#if USE_MYSQL
    static std::shared_ptr<DbClient> newMysqlClient(const std::string &connInfo, const size_t connNum);
#endif
```

上記のインターフェースを使うと、DbClientを実装したオブジェクトへのスマートポインタを取得できます。connInfo引数はコネクションの設定で、key=valueの形式で列挙する事で設定できます。詳しくはヘッダーファイルにあるコメントを参照してください。connNum引数はDbClientのデータベースへのコネクション数で、並行性に大きく影響を与えます。状況に合わせて設定してください。

上記の方法で取得されたオブジェクトは、ユーザー側がそれを保持する方法を用意する必要があります。例えばグローバル変数に保存するなどです。以下に示す理由のため、**一時的にオブジェクトを作成し、使い終わったら開放する手法はしないでください**

- 接続と切断の分だけ時間を無駄に使い、システムが遅延します
- このインターフェース自体もノンブロッキングです。つまり、DbClientのオブジェクトを取得してもそれが管理するコネクションはまだ確立されていません。drogonは(意図的に)接続が確立した時のコールバックを提供していません。つまりクエリを行うために待機する必要があり、これは非同期フレームワークという本来の目的に反しています。

これらの理由から、DbClientオブジェクトはプログラムの開始時に構築され、プログラムが停止するまでは保持・利用され続けるべきです。これはフレームワーク側でできる事です。なので、drogonではコンフィグファイルか `createDbClient()`によって作成される方法を提供しています。コンフィグについては[db_clients](/ENG/ENG-11-Configuration-File?id=db_clients)を参照してください。


必要に応じて、DbClientのスマートポインタはフレームワークのインターフェースを介して取得できます。インターフェースは次のとおりです。

```c++
orm::DbClientPtr getDbClient(const std::string &name = "default");
```


引数のnameはコンフィグファイルのオプション名を示し、これは1つのアプリケーションで複数のDbClientを利用する必要がある時に使われます。コネクションはDbClientによって自動的に再接続されるため、ユーザーは接続の状態について考える必要がありません。また、接続はほぼ常に維持されます。**Note**: app.run()よりも前でこの関数を呼び出すことはできません。その場合、空のshared_ptrが返ります。

### 実行用インターフェース

DbClientでは次のようにいくつかのインターフェースを提供しています。

```c++
/// 非同期メソッド
template <
        typename FUNCTION1,
        typename FUNCTION2,
        typename... Arguments>
void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;

/// futureによる非同期メソッド
template <typename... Arguments>
std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                             Arguments &&... args) noexcept;

/// 同期メソッド
template <typename... Arguments>
const Result execSqlSync(const std::string &sql,
                         Arguments &&... args) noexcept(false);

/// ストリームを使うメソッド
internal::SqlBinder operator<<(const std::string &sql);
```


バインディングする引数の数はあらかじめ決まっているわけではなく、関数テンプレートを利用しています。


それぞれのメソッドの特徴については以下の表の通りです:

| Methods                                       | 同期/非同期 | ブロッキング/ノンブロッキング     | 例外                                                        |
| :-------------------------------------------- | :----------------------- | :---------------------------------------------- | :--------------------------------------------------------------- |
| void execSqlAsync                             | 非同期 | Non-blocking                                    | 投げない                              |
| std::future<const Result\> execSqlAsyncFuture | 非同期  | futureのgetメソッドを呼んだ時にブロック | futureのgetメソッドを呼んだときに投げる |
| const Result execSqlSync                      | 同期          | Blocking                                        | 投げる                                |
| internal::SqlBinder operator<<                | 非同期    | Default non-blocking                            | 投げない                              |


非同期とブロッキングの組み合わせに混乱しているかもしれません。基本的に、同期メソッドはネットワークIOのブロッキングを起こし、非同期メソッドはノンブロッキングです。ただ、非同期メソッドをブロッキングとして動作させることもできます。つまり、コールバックが終わるまでブロックするという事です。DbClientの非同期メソッドがブロッキングとして動作する際は、関数を呼び出したスレッドでコールバックが実行され、メソッドが返ります。

アプリケーションで高い平行性が必要な場合は、ノンブロッキングの非同期メソッドを利用してください。一方、あまり平行性は必要でない場合（ネットワーク上のデバイスの管理用ページなど）では書きやすさや理解しやすさのために同期メソッドを利用することもできます。

- #### execSqlAsync

  ```c++
  template <typename FUNCTION1,
          typename FUNCTION2,
          typename... Arguments>
  void execSqlAsync(const std::string &sql,
                  FUNCTION1 &&rCallback,
                  FUNCTION2 &&exceptCallback,
                  Arguments &&... args) noexcept;
  ```

  これは最も一般的な非同期インターフェースで、ノンブロッキングで動作します。

  `sql`引数は、SQL文の文字列です。バインディングのためのプレースホルダはデータベースに対応します。例えば、PostgreSQLでは`$1`,`$2`...、MySQLでは数字を含まない`?`です。

  `args`で示される引数はバインディングする値で、0以上の好きな数だけ指定できます。個数はSQL文内のプレースホルダの数に一致する必要があります。使用できる型は以下の通りです。

  - 整数型: 様々なサイズの整数型を使用でき、データベースのフィールドのサイズと一致させるべきです。
  - 浮動小数点型: `float`または`double`で、データベースのフィールドの型に一致させるべきです
  - 文字列型: `std::string`か`const char[]`で、データベースでの文字列型に対応します
  - 時刻型: `trantor::Date`型で、 データベースのdate, datetime, timestampに対応します。
  - Binary type: `std::vector<char>`で、PostgreSQLのバイナリ列型、MySQLのBlob型に対応します。


  これらは左辺値や右辺値、変数やリテラルにすることができ、自由に指定することができます。

  rCallbackおよびexceptCallback引数はそれぞれ結果用と例外時のコールバックです。それぞれ次のような形式である必要があります。

  - 結果用のコールバック: 関数シグネチャはvoid(const Result &)である必要があり、これに対応するstd::functionやラムダ式などのcallableなオブジェクトを使用できます。
  - 例外用のコールバック: 関数シグネチャはvoid(const DrogonDbException &)で、これに対応するstd::functionやラムダ式などのcallableなオブジェクトを使用できます。

  SQL文の実行が成功した場合、実行結果はResult型を使って結果用のコールバックに渡されます。なんらかの例外が発生した場合は例外用のコールバックが実行され、DrogonDbExceptionの値から例外についての情報を取得できます。

  一つ例を見てみましょう。

  ```c++
  auto clientPtr = drogon::app().getDbClient();
  clientPtr->execSqlAsync("select * from users where org_name=$1",
                              [](const drogon::orm::Result &result) {
                                  std::cout << result.size() << " rows selected!" << std::endl;
                                  int i = 0;
                                  for (auto row : result)
                                  {
                                      std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                                  }
                              },
                              [](const DrogonDbException &e) {
                                  std::cerr << "error:" << e.base().what() << std::endl;
                              },
                              "default");
  ```

  プログラム例からResult型がstdのコンテナ型と互換性があることがわかります。イテレータをサポートしているので、範囲forで各行を取得する事ができます。ResultやRow, Field型の詳しいインターフェースについてはソースコードを参照してください。

  DrogonDbException型はデータベースでのすべての例外の基底型です。ソースコードのコメントを参照してください。


- #### execSqlAsyncFuture

  ```c++
  template <typename... Arguments>
  std::future<const Result> execSqlAsyncFuture(const std::string &sql,
                                              Arguments &&... args) noexcept;
  ```

  非同期のfutureを使うインターフェースでは、先に示した関数にあったコールバックの引数がありません。この関数を呼び出すとfutureオブジェクトをすぐに返します。ユーザーはfutureオブジェクトのget()メソッドを呼び出して結果を取得する必要があります。例外についてはtry/catchの方式で行われるので、get()メソッドが`try/catch`ブロックに含まれておらずコールスタックにもない場合、SQLの実行で例外が発生するとプログラムは終了します。

  For example:

  ```c++
  auto f = clientPtr->execSqlAsyncFuture("select * from users where org_name=$1",
                                      "default");
  try
  {
      auto result = f.get(); // Block until we get the result or catch the exception;
      std::cout << result.size() << " rows selected!" << std::endl;
      int i = 0;
      for (auto row : result)
      {
          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
      }
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### execSqlSync

  ```c++
  template <typename... Arguments>
  const Result execSqlSync(const std::string &sql,
                          Arguments &&... args) noexcept(false);
  ```

  同期インターフェースはシンプルで直感的です。引数はSQL文とバインディングするパラメータで、Resultオブジェクトを返し、現在のスレッドをブロックします。問題が発生した場合は例外を投げるので、`try/catch`でキャッチするように注意してください。

  E.g:

  ```c++
  try
  {
      auto result = clientPtr->execSqlSync("update users set user_name=$1 where user_id=$2",
                                          "test",
                                          1); // 結果が取得できるか例外が出るまでブロック
      std::cout << result.affectedRows() << " rows updated!" << std::endl;
  }
  catch (const DrogonDbException &e)
  {
      std::cerr << "error:" << e.base().what() << std::endl;
  }
  ```

- #### operator<<

  ```c++
  internal::SqlBinder operator<<(const std::string &sql);
  ```

  ストリームを使うインターフェースは少し特殊です。SQL文とパラメータを`<<`演算子で入力し、結果と例外のコールバックは`>>`演算子で指定します。例えば、先述の値を取得する例はストリームを使って書くとこのようになります。

  ```c++
  *clientPtr  << "select * from users where org_name=$1"
              << "default"
              >> [](const drogon::orm::Result &result)
                  {
                      std::cout << result.size() << " rows selected!" << std::endl;
                      int i = 0;
                      for (auto row : result)
                      {
                          std::cout << i++ << ": user name is " << row["user_name"].as<std::string>() << std::endl;
                      }
                  }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  この使い方では最初の非同期かつノンブロッキングなインターフェースと全く同じで、どちらを使うかはスタイルによります。また、ブロッキングで動作させたい場合、`<<`に`Mode::Blocking`を渡すことで行うことができますが、詳細は省きます。

  加えて、ストリームを使う場合には特殊な使い方があります。特殊なコールバックを使用する事で、フレームワークは実行結果を行ごとに渡すようになります。その関数シグネチャは次の通りです。

  ```c++
  void (bool,Arguments...);
  ```


  1つ目のboolの引数がtrueだった場合、結果が空行である事、つまり全ての行を返し終わったことを意味し、これが最後に呼び出されます。
  それ以降はパラメータの列で、各行のカラムの値に対応します。フレームワークは型変換を行いますが、当然ながら対応する型になるよう気を付けてください。型はconstな左辺値参照か右辺値参照、そして値型にすることができます。

  先ほどの例をこの方式で書き直してみると、このようになります。

  ```c++
  int i = 0;
  *clientPtr  << "select user_name, user_id from users where org_name=$1"
              << "default"
              >> [&i](bool isNull, const std::string &name, int64_t id)
                      {
                      if (!isNull)
                          std::cout << i++ << ": user name is " << name << ", user id is " << id << std::endl;
                      else
                          std::cout << i << " rows selected!" << std::endl;
                      }
              >> [](const DrogonDbException &e)
                  {
                      std::cerr << "error:" << e.base().what() << std::endl;
                  };
  ```

  この例ではSELECT文によるuser_nameとuser_idの値はnameとid引数に代入されています。型変換などのコードを書く必要がないため利便性があり、また柔軟に使用できます。

> **Note: 非同期プログラミングにおいて注意してほしい点として、上の例の変数iがあります。変数iは参照としてキャプチャしているので、コールバックが呼ばれた時点で変数iの参照先が有効である必要があります。コールバックは別のスレッドで呼ばれる事もあるので、すでに無効になっていることがあります。よく使われる手法としては変数を保持するためにスマートポインタを使ってそれをコールバックでキャプチャする事で保証するものがあります。**

### まとめ

それぞれのDbClientは複数のデータベースへ接続するイベントループ用のスレッドを持っており、リクエストを同期および非同期に処理し、結果をコールバックで返す事ができます。

ブロッキングインターフェースは呼び出し側のスレッドのみをブロックし、イベントループのスレッドで呼び出さない限りはイベントループのスレッドには影響しません。コールバックが呼ばれるときは、コールバックの内容はイベントループのスレッドで実行されます。なので、コールバックの中でスレッドをブロックするような操作をするとデータベースの読み書きの平行性に影響を与えるため、しないでください。ノンブロッキングIOを扱う場合は、必ずこのことを頭に入れておいてください。

# Next: [Transaction](/ENG/ENG-08-2-Database-Transaction)
