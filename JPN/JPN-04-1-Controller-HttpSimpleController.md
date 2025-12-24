##### Other languages: [简体中文](/CHN/CHN-04-1-控制器-HttpSimpleController)

# Controller - HttpSimpleController

`drogon_ctl`コマンドを使えば`HttpSimpleController`を使ったcontrollerを簡単に作成できます。次のように利用します。

```shell
drogon_ctl create controller <[namespace::]class_name>
```

例えば、`TestCtrl`という名前のcontrollerを作成するにはこうします。

```shell
drogon_ctl create controller TestCtrl
```


実行することでカレントディレクトリに`TestCtrl.h`と`TestCtrl.cc`の2つのファイルが作成されます。内容を見ていきましょう。

TestCtrl.h：

```c++
#pragma once
#include <drogon/HttpSimpleController.h>
using namespace drogon;
class TestCtrl:public drogon::HttpSimpleController<TestCtrl>
{
public:
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                        std::function<void (const HttpResponsePtr &)> &&callback)override;
    PATH_LIST_BEGIN
    //list path definitions here;
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);
    PATH_LIST_END
};
```

TestCtrl.cc:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
}
```
`HttpSimpleController`を使ったcontrollerでは1つだけHTTPハンドラーを定義できます。また、ハンドラーは仮想関数として定義されます。

URLからのルーティング(あるいはマッピング)には`PATH_ADD`マクロを使います。複数個記述することで、複数のURLを指定できます。全ての`PATH_ADD`は`PATH_LIST_BEGIN`から`PATH_LIST_END`までの中に入るようにしてください。

第一引数は割り当てるパスを指定します。それ以降の引数はマッピングの制約を指定します。現状では2種類の制約を指定できます。1つは列挙型の`HttpMethod`で、これは許可するメゾットを表します。もう1つは`HttpFilter`クラスの名前を指定するものです。これらの制約には個数や順序に制限はありません。filterについては[Middleware and Filter](/ENG//ENG/ENG-05-Middleware-and-Filter)を参照してください。

同じコントローラに複数のパスをマッピングしたり、HTTPメゾットを指定することで同じパスに複数のcontrollerを割り当てることができます。

`HttpResponse`クラスの変数を作成し、`cakkback()`を呼び出すことでレスポンスを返すことができます。

```c++
    //write your application logic here
    auto resp=HttpResponse::newHttpResponse();
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Your Page Contents");
    callback(resp);
```


> **上記のパスからハンドラへのマッピングはコンパイル時に行われます。なお、Drogonは実行時にマッピングを設定する方法も提供しています。これは、ユーザーがマッピングの設定を再コンパイルをせずにコンフィグの編集などの手段でできるようにするためです。(パフォーマンス上の理由から、`app().run()`の実行後のcontrollerへのマッピングはできません)**

# Next: [HttpController](/ENG/ENG-04-2-Controller-HttpController)
