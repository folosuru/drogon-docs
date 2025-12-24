##### Other languages: [简体中文](/CHN/CHN-17-协程)

# コルーチン

drogonではバージョン1.4以降で[C++ コルーチン][1]に対応しています。コルーチンでは非同期呼び出しの流れを平坦にでき、いわゆるコールバック地獄を回避できます。コルーチンを使う事で、非同期プログラミングは同期プログラミングと同程度に簡単になります。

### 技術

このページはコルーチンが何なのかやどう動作するのかではなく、drogonでどのようにコルーチンを使うのかについて説明します。コルーチンでも多くの用語は通常の関数（サブルーチン）と同様ですが、若干意味合いが異なる物もあります。混乱を避けるため、以下に用語を示します。

**Coroutine(コルーチン)** は実行の中断と再開が可能な関数。<br/>
**Return** は関数の実行を終了し、呼び出し元へ値を返します。コルーチンにおいては、再開のために使う _resumable_ オブジェクトを返す。<br/>
**Yield(生成)** は、コルーチンが呼び出し元へ値を生成して中断すること。<br/>
**co-return** はコルーチンが値を生成して終了すること。<br/>
**(co-)await** は、（ネットワーク通信などの）結果が完了するまで待機している状態。中断している間はコルーチンを実行していたスレッドは別の事を行い、準備ができたらコルーチンが再開する。<br/>

### コルーチンの有効か

The coroutine feature in Drogon is header-only. This means the application can use coroutines even if Drogon is built without coroutine support. How to enable coroutines depends on the compiler used. In GCC >= 10 it can be enabled by setting `-std=c++20 -fcoroutines` while with MSVC (tested on MSVC 19.25) it can be enabled with `/std:c++latest` and `/await` must not be set.

drogonでのコルーチンはヘッダーオンリーです。なので、drogon自体がコルーチンなしでビルドされていてもアプリケーションではコルーチンを利用する事ができます。コルーチンを有効化する方法は使用するコンパイラーによります。GCC >= 10 では`-std=c++20 -fcoroutines`を、MSVC（バージョニング19.25で確認）では`/std:c++latest` と `/await` を設定する必要があります。

注意点として、drogonのコルーチン実装はclang (12.0現在) では動作しません。GCC11ではC++20が有効化されていれば自動的に有効化されます。また、GCC 10ではコルーチンをコンパイルできますが、コンパイラにバグがあり、ネストされたコルーチンフレームのメモリが解放されずメモリリークを引き起こします。

### Using coroutines
drogonにおけるコルーチンは、末尾に`Coro`がつきます。`db->execSqlSync()` は`db->execSqlCoro()`に、`client->sendRequest()` は `client->sendRequestCoro()`にといった具合です。コルーチンは _awaitable_ な値を返し、それに`co_await`を行う事で結果を得ることができます。フレームワークは結果が得られるまでのawaitしている間、スレッドをIOの処理や別のタスクに使う事ができます。これがコルーチンの素晴らしい所で、見た目上は同期的なコードに見えますが、実際には非同期に動いているのです。

例として、データベースに存在するユーザー数を取得するコードはこのようになります。

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req, std::function<void(const HttpResponsePtr&)> callback) -> Task<>
    //                                                          戻り値は _resumable_ ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        callback(resp);
    }
    catch(const DrogonDbException &err)
    {
        // 例外も同期的なインターフェースと同様に扱えます
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        callback(resp);
    }
    // 返す必要のあるものはありません。なので、このコルーチンはvoidを返します。
    // これは、Task<void>で示されています
    co_return; // したい場合（必須ではない） co_returnする
}
```

重要なポイント：

1.  コルーチンを呼び出すハンドラは _resumable_ を返す事
    - これでハンドラがコルーチンになります
2.  `co_return` で `return` を置き換える 
3.  大半の引数はパラメータ渡しにする

_resumable_ はコルーチンの標準的な方式に従っている型です。これについてあまり深く考える必要はありません。知っておくべきことは、コルーチンで`T`型を生成するときには戻り値の型が`Task<T>`になるという事です。

引数をパラメータ渡しにするのは、コルーチンが非同期であるためです。コルーチンが待機している間に参照先がスコープから外れて破壊されたりするのを知るのは不可能です。また、参照先は別のスレッドである可能性もあり、コルーチンの実行中に参照先が破壊される可能性もあります。

コールバックではなく、`co_return`を使って結果を返す方がわかりやすいかもしれません。この方式もサポートされていますが、状況によりコールバックと比べて最大で8%程スループットが下がる場合があります。使用目的に対してパフォーマンスの低下が許容できる程度かによって使い分けてください。下記のコードは上で示した処理と同じ処理を行うものです。

```c++
app.registerHandler("/num_users",
    [](HttpRequestPtr req) -> Task<HttpResponsePtr>)
    //          レスポンスを返している ^^^
{
    auto sql = app().getDbClient();
    try
    {
        auto result = co_await sql->execSqlCoro("SELECT COUNT(*) FROM users;");
        size_t num_users = result[0][0].as<size_t>();
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(std::to_string(num_users));
        co_return resp;
    }
    catch(const DrogonDbException &err)
    {
        // 同期インターフェースのように例外処理ができる
        auto resp = HttpResponse::newHttpResponse();
        resp->setBody(err.base().what());
        co_return resp;
    }
}
```

websocket controllerでのコルーチンには現状対応していません。この機能が必要であれば、お気軽にissueを開いてください。

### よくある落とし穴

コルーチンを使うとき、いくつかの落とし穴に出くわすかもしれません。

- #### Launching coroutines with lambda capture from a function

  Lambda captures and coroutines have separate lifetimes. A coroutine lives until the coroutine frame is destructed. While lambdas commonly destruct right after being called. Thus, due to the asynchronous nature of coroutines, the coroutines's lifetime can be much longer than the lambda, for example in SQL execution. The lambda destructs right after awaiting for SQL to complete (and returns to the event loop to process other events), while the coroutine frame is awaiting SQL. Thus the lambda will have been destructed when SQL has finished.

  Instead of

  ```c++
  app().getLoop()->queueInLoop([num] -> AsyncTask {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY". std::to_string(num));
      // The lambda object, thus captures destruct right at awaiting. They are destructed at this point
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days"; // use-after-free
  });
  // BAD, This will crash
  ```

  Drogon provides `async_func` that wraps around the lambda to ensure its lifetime

  ```c++
  app().getLoop()->queueInLoop(async_func([num] -> Task<void> {
  //                             ^^^^^^^^^^^^^^^^^^^^^^^^^ wrap with async_func and return a Task<>
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE last_login < CURRENT_TIMESTAMP - INTERVAL $1 DAY". std::to_string(num));
      LOG_INFO << "Remove old customers that have no activity for more than " << num << "days";
  }));
  // Good
  ```

- #### Passing/capturing references into coroutines from function

  It's a good practice in C++ to pass objects by reference to reduce unnecessary copy. However passing by reference into a coroutine from a function commonly causes issues. This is caused by the the coroutine is in fact asynchronous and can have a much longer lifetime compared to a regular function. For example, the following code crashes

  ```cpp
  void removeCustomers(const std::string& customer_id)
  {
      async_run([&customer_id] {
          //      ^^^^ DO NOT pass/capture objects by reference into a coroutine
          // Unless you are sure the object has a longer lifetime than the coroutine

          auto db = app().getDbClient();
          co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
          // `customer_id` goes out of scope right at awaiting SQL. Crashes here
          co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
      }
  }
  ```

  However passing objects as reference from a coroutine is considered a good practice

  ```cpp
  Task<> removeCustomers(const std::string& customer_id)
  {
      auto db = app().getDbClient();
      co_await db->execSqlCoro("DELETE FROM customers WHERE customer_id = $1", customer_id);
      co_await db->execSqlCoro("DELETE FROM orders WHERE customer_id = $1", customer_id);
  }

  Task<> findUnwantedCustomers()
  {
      auto db = app().getDbClient();
      auto list = co_await db->execSqlCoro("SELECT customer_id from customers "
          "WHERE customer_score < 5;");
      for(const auto& customer : list)
          co_await removeCustomers(customer["customer_id"].as<std::string>());
          //                               ^^^^^^^^^^^^^^^^^
          // This is perfectly fine and preferred although it's a const reference
          // since we are calling it from a coroutine
  }
  ```

[1]: https://en.cppreference.com/w/cpp/language/coroutines

# Next: [Redis](/ENG/ENG-18-Redis)
