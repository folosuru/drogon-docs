##### Other languages: [English](/ENG/ENG-03-Quick-Start) , [简体中文](/CHN/CHN-03-快速开始) 

# Quick Start

## 静的サイト

drogonの使い方を説明する簡単な例から始めましょう。この例では、プロジェクトを`drogon_ctl`で作成します。

```shell
drogon_ctl create project your_project_name
```

プロジェクトのディレクトリに移動すると、以下のようなファイルが生成されています。

```console
├── build                         ビルド用
├── CMakeLists.txt                CMakeの設定ファイル
├── config.json                   Drogonの設定ファイル
├── controllers                   controllerのファイル用のディレクトリ
├── filters                       filterのファイル用のディレクトリ
├── main.cc                       Main program
├── models                        database modelのファイル
│   └── model.json
└── views                         ViewのCSPファイル用のディレクトリ
```

各種のファイル (controllerやfilter, viewなど) を対応するディレクトリに作成することができます。より便利にかつ安全に扱うため、プロジェクトを作成するときには`drogon_ctl`コマンドを使用する事を強くお勧めします。コマンドについての詳細は、 [drogon_ctl](/ENG//ENG/ENG-11-drogon_ctl-Command) を参照して下さい。

それでは、`main.cc`ファイルを見ていきましょう。

```c++
#include <drogon/HttpAppFramework.h>
int main() {
    //Set HTTP listener address and port
    drogon::app().addListener("0.0.0.0",5555);
    //Load config file
    //drogon::app().loadConfigFile("../config.json");
    //Run HTTP framework,the method will block in the internal event loop
    drogon::app().run();
    return 0;
}
```

そして、これをビルドするには

```shell
cd build
cmake ..
make
```

を実行します。コンパイルが完了したら、`./your_project_name`で成果物を動かしましょう。

では、とてもシンプルな静的サイトである index.htmlを httpのルートパスに置いてみましょう:

```shell
echo '<h1>Hello Drogon!</h1>' >>index.html
```

デフォルトのルートパスは `"./"`です。この値はconfig.jsonで編集することができます。config.jsonについては、[Configuration File](/ENG/ENG/ENG-10-Configuration-File) を参照して下さい。

では、このページにアクセスしてみましょう。 `"http://localhost:5555"` 
あるいは`"http://localhost:5555/index.html"`(あるいは、サーバーの動いているIPアドレス)でアクセスできます。

![Hello Drogon!](images/hellodrogon.png)

もしリクエストされたページが見当たらなかった場合、404ページを返します。
![404 page](images/notfound.png)

> **Note: ファイヤーウォールがサーバーの80番ポートの通信を許可している事を確認してください。開いていない場合、ページを見る事ができません。また、以下のようなエラーが出る場合、ポートを80から1024以降に変更してみてください。

```console
FATAL Permission denied (errno=13) , Bind address failed at 0.0.0.0:80 - Socket.cc:67
```

We could copy the directory and files of a static website to the startup directory of this running webapp, then we can access them from the browser. The file types supported by drogon by default are

静的サイトのファイルやディレクトリをアプリケーションが動作しているディレクトリにコピーすることで、ブラウザからアクセスできるようになります。Drogonではデフォルトで次のファイルを扱う事が出来ます。

- html
- js
- css
- xml
- xsl
- txt
- svg
- ttf
- otf
- woff2
- woff
- eot
- png
- jpg
- jpeg
- gif
- bmp
- ico
- icns

また、Drogonではファイルタイプを変更することもできます。詳細はHttpAppFramework APIを参照してください

## 動的サイト

この説ではアプリケーションにcontrollerを追加し、それを使ってコンテンツを返す方法を見ていきます。

controllerを作成するには、`drogon_ctl`を使います。`controllers`ディレクトリで実行してみましょう。

```shell
drogon_ctl create controller TestCtrl
```

As you can see, there are two new files, TestCtrl.h and TestCtrl.cc：

見てわかる通り、TestCtrl.h と TestCtrl.ccの2つのファイルが生成されています。

TestCtrl.hは以下の通り:

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

TestCtrl.ccは以下の通り:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
}
```

では、これらのファイルを編集してcontrollerが"Hello World!"とレスポンスを返すようにしてみましょう。

TestCtrl.h はこうします:

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
    //list path definitions here

    //example
    //PATH_ADD("/path","filter1","filter2",HttpMethod1,HttpMethod2...);

    PATH_ADD("/",Get,Post);
    PATH_ADD("/test",Get);
    PATH_LIST_END
};
```

`PATH_ADD`を使ってパスのマッピングを行います。ここでは、`/`と`/test`の2つを登録し、さらにその後ろで許可するHTTPメゾットを指定しています。


TestCtrl.ccは次のようにします:

```c++
#include "TestCtrl.h"
void TestCtrl::asyncHandleHttpRequest(const HttpRequestPtr &req,
                                      std::function<void (const HttpResponsePtr &)> &&callback)
{
    //write your application logic here
    auto resp=HttpResponse::newHttpResponse();
    //NOTE: The enum constant below is named "k200OK" (as in 200 OK), not "k2000K".
    resp->setStatusCode(k200OK);
    resp->setContentTypeCode(CT_TEXT_HTML);
    resp->setBody("Hello World!");
    callback(resp);
}
```

再びCMakeでプロジェクトをコンパイルし、`./your_project_name`を実行しましょう。

```shell
cd ../build
cmake ..
make
./your_project_name
```

`"http://localhost/"` か `"http://localhost/test"` をブラウザに入力してアクセスすると、"Hello World!"と表示されるはずです。

> **Note: 静的リソースと動的リソースの両方が存在する場合、Drogonは動的なリソースを優先します。この例の場合、`http://localhost`にアクセスすると静的リソースである`index.html`ではなく、動的リソースであるcontrollerの`TestCtrl`が優先され、`Hello World!`と表示されます。**

We see that adding a controller to an application is very simple. You only need to add the corresponding source file. Even the main file does not need to be modified. This loosely coupled design is very effective for web application development.

controllerの作成がとてもシンプルな事がおわかりいただけたでしょうか。編集が必要なのは対応するソースファイルだけで、`main.cc`を編集する必要はありません。この疎結合な設計は開発を効率的にしてくれるでしょう。

> **Note: Drogonでは、controllerのファイルの配置場所に制約はありません。プロジェクトのルート(`./`)に保存したり、`CMakeLists.txt`で指定した別のディレクトリに保存することも可能です。ただ、管理のしやすさのために`controllers`ディレクトリに保存する事をおすすめします。

# Next: [Controller - Introduction](/ENG/ENG-04-0-Controller-Introduction)
