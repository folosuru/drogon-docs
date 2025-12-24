##### Other languages: [简体中文](/CHN/CHN-04-2-控制器-HttpController)

# Controller - HttpController

### 作成

`drogon_ctl`コマンドを使えば`HttpController`を使ったcontrollerを簡単に作成できます。次のように利用します。

```shell
drogon_ctl create controller -h <[namespace::]class_name>
```

We create one controller class named `User`, under namespace `demo v1`:

`demo v1`の名前空間内にあるcontrollerの`User`クラスを作成するには次のようにします。

```shell
drogon_ctl create controller -h demo::v1::User
```

実行することでカレントディレクトリに`demo_v1_User.h`と`demo_v1_User.cc`の2つのファイルが作成されます。内容を見ていきましょう。

demo_v1_User.h:

```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // use METHOD_ADD to add your custom processing function here;
    // METHOD_ADD(User::get, "/{2}/{1}", Get); // path is /demo/v1/User/{arg2}/{arg1}
    // METHOD_ADD(User::your_method_name, "/{1}/{2}/list", Get); // path is /demo/v1/User/{arg1}/{arg2}/list
    // ADD_METHOD_TO(User::your_method_name, "/absolute/path/{1}/{2}/list", Get); // path is /absolute/path/{arg1}/{arg2}/list

    METHOD_LIST_END
    // your declaration of processing function maybe like this:
    // void get(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, int p1, std::string p2);
    // void your_method_name(const HttpRequestPtr& req, std::function<void (const HttpResponsePtr &)> &&callback, double p1, int p2) const;
};
}
}
```

demo_v1_User.cc:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Add definition of your processing function here
```

### 使用法

これら2つのファイルを編集していきましょう:

demo_v1_User.hはこのように:

```c++
#pragma once

#include <drogon/HttpController.h>

using namespace drogon;

namespace demo
{
namespace v1
{
class User : public drogon::HttpController<User>
{
  public:
    METHOD_LIST_BEGIN
    // use METHOD_ADD to add your custom processing function here;
    METHOD_ADD(User::login,"/token?userId={1}&passwd={2}",Post);
    METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get);
    METHOD_LIST_END
    // your declaration of processing function maybe like this:
    void login(const HttpRequestPtr &req,
               std::function<void (const HttpResponsePtr &)> &&callback,
               std::string &&userId,
               const std::string &password);
    void getInfo(const HttpRequestPtr &req,
                 std::function<void (const HttpResponsePtr &)> &&callback,
                 std::string userId,
                 const std::string &token) const;
};
}
}
```

demo_v1_User.ccはこうします:

```c++
#include "demo_v1_User.h"

using namespace demo::v1;

// Add definition of your processing function here

void User::login(const HttpRequestPtr &req,
                 std::function<void (const HttpResponsePtr &)> &&callback,
                 std::string &&userId,
                 const std::string &password)
{
    LOG_DEBUG<<"User "<<userId<<" login";
    //Authentication algorithm, read database, verify, identify, etc...
    //...
    Json::Value ret;
    ret["result"]="ok";
    ret["token"]=drogon::utils::getUuid();
    auto resp=HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
void User::getInfo(const HttpRequestPtr &req,
                   std::function<void (const HttpResponsePtr &)> &&callback,
                   std::string userId,
                   const std::string &token) const
{
    LOG_DEBUG<<"User "<<userId<<" get his information";

    //Verify the validity of the token, etc.
    //Read the database or cache to get user information
    Json::Value ret;
    ret["result"]="ok";
    ret["user_name"]="Jack";
    ret["user_id"]=userId;
    ret["gender"]=1;
    auto resp=HttpResponse::newHttpJsonResponse(ret);
    callback(resp);
}
```
`HttpController`には複数のHTTPハンドラーを定義できます。ハンドラはいくらでも定義することができるので、仮想関数のオーバーライドでの実装は非現実的です。そのため、クラスではなくハンドラ関数そのものを登録します。

- #### Path Mapping

  URLからハンドラへのマッピングには`METHOD_ADD`マクロか、`ADD_METHOD_TO`マクロを使います。これらのマクロは全て`METHOD_LIST_BEGIN`から`METHOD_LIST_END`の間にあるようにしてください。

  `METHOD_ADD`マクロでは、名前空間およびクラス名がパスの先頭に自動的に付加されます。この例では、`login`関数は`/demo/v1/user/token`に、`getInfo`関数は`/demo/v1/user/xxx/info`にマッピングされます。なお、制約の記法は`PATH_ADD`と近いため、ここでは省略します。

  `ADD_METHOD_TO`は`METHOD_ADD`とほとんど同じですが、接頭辞は追加しません。つまり、絶対パスで登録されるということです。


  このように、`HttpController`は柔軟なパスのマッピングを複数のハンドラに行うことができます。


  さらに、マクロの中でパラメーターマッピングを使っていることも分かります。クエリパラメータをパスにマッピングして、関数の引数に渡すことができます。URLパラメータの数字は引数の順番に対応しています。引数には、文字列型から変換できる一般的な型(`std::string`, `int`, `float`, `double`など)が使用でき、Drogonが自動的に変換します。注意点として、左辺値参照の型は`const`でなければなりません。

  HTTPメソッドが違う、同じパスへの複数回のマッピングをする事も可能です。これはRestfulなAPIを作るには一般的な物です。

  ```c++
  METHOD_LIST_BEGIN
      METHOD_ADD(Book::getInfo,"/{1}?detail={2}",Get);
      METHOD_ADD(Book::newBook,"/{1}",Post);
      METHOD_ADD(Book::deleteOne,"/{1}",Delete);
  METHOD_LIST_END
  ```

  パスのプレースホルダにはいくつかの記法があります。

  - `{}`: パスでの位置が引数の位置になります。
  - `{1}, {2}`: パスのパラメーターに数字を入れた場合、番号で指定されたように引数に渡されます。
  - `{anystring}`: 括弧の中に文字を書いても動作への影響はありませんが、読みやすさを向上させることができます。`{}`と同等です。
  - `{1:anystring},{2:xxx}`: コロンより左側の数字は位置を表します。コロンより後ろ側には動作への影響はありませんが、読みやすさを向上させることができます。`{1}, {2}`と同等です。


  3番目および4番目の記法を推奨します。また、パスでの順番と引数での順番が同じであれば、3番目の記法で十分です。例えば次の4つの意味は全く同じになります。

  - "/users/{}/books/{}"
  - "/users/{}/books/{2}"
  - "/users/{user_id}/books/{book_id}"
  - "/users/{1:user_id}/books/{2}"

  > **Note: パスのマッチングには大文字小文字の区別がありませんが、パラメータでは区別します。パラメータの値は大文字小文字をそのままハンドラーの引数に渡します。**

  #### パラメータマッピング
  ここまでの説明で、パスおよび`?`の後のクエリパラメータをハンドラーの引数にマッピングすることがわかりました。その引数の型は次の条件を満たす必要があります。

  - 型は値型(`int`など)、constな左辺値参照、constでない右辺値参照のいずれかの必要があり、constでない左辺値参照は使用できません。ユーザー側で破棄できるため、右辺値参照を使用することを推奨します。

  - 基本型(`int`, `long`, `long long`, `unsigned long`, `unsigned long long`, `float`, `double`, `long double`など)

  - std::string

  - `stringstream >>`演算子で代入できる型

  > **また、Drogonでは`HttpRequestPtr`から他の型へ変換した値をマッピングすることもできます。**ハンドラの引数よりパスのパラメータの方が多い場合、残りのパラメータは`HttpRequestPtr`により変換されます。変換方法を定義するには、`fromRequest`テンプレート(HttpRequest.hで定義されています)を特殊化してください。例えばこの例では、新しいユーザーを作成するRestfulなインタフェースでユーザーの構造体を定義しています。

  ```c++
  namespace myapp{
  struct User{
      std::string userName;
      std::string email;
      std::string address;
  };
  }
  namespace drogon
  {
  template <>
  inline myapp::User fromRequest(const HttpRequest &req)
  {
      auto json = req.getJsonObject();
      myapp::User user;
      if(json)
      {
          user.userName = (*json)["name"].asString();
          user.email = (*json)["email"].asString();
          user.address = (*json)["address"].asString();
      }
      return user;
  }

  }
  ```

  上述の定義とテンプレート特殊化によって、ハンドラをこのように定義できるようになります。

  ```c++
  class UserController:public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          //use METHOD_ADD to add your custom processing function here;
          ADD_METHOD_TO(UserController::newUser,"/users",Post);
      METHOD_LIST_END
      //your declaration of processing function maybe like this:
      void newUser(const HttpRequestPtr &req,
                  std::function<void (const HttpResponsePtr &)> &&callback,
                  myapp::User &&pNewUser) const;
  };
  ```

  It can be seen that the third parameter of `myapp::User` type has no corresponding placeholder on the mapping path, and the framework regards it as a parameter converted from the `req` object and obtains this parameter through the user-specialized function template. This is very convenient for users.

  3番目の引数の`myapp::User`はパスのマッピングに対応していないため、Drogonはこれを`req`からの変換と見なしてユーザー定義の関数テンプレートを利用して引数を作成します。

  Further, some users do not need to access the HttpRequestPtr object except for their custom type data. They can put the custom object in the position of the first parameter, and the framework will correctly complete the mapping such as the above example. It can also be written as follows:

  また、カスタムデータ以外ではHttpRequestPtrオブジェクトが不要になる場合もあるでしょう。その場合、第一引数にカスタムオブジェクトを指定することでもマッピングを行えます。例えば、次のように書くことができます。

  ```c++
  class UserController:public drogon::HttpController<UserController>
  {
  public:
      METHOD_LIST_BEGIN
          //use METHOD_ADD to add your custom processing function here;
          ADD_METHOD_TO(UserController::newUser,"/users",Post);
      METHOD_LIST_END
      //your declaration of processing function maybe like this:
      void newUser(myapp::User &&pNewUser,
                  std::function<void (const HttpResponsePtr &)> &&callback) const;
  };
  ```

- #### Multiple Path Mapping

  Drogonは正規表現によるパスマッピングにも対応しており、波括弧`{}`の外側で使用することができます。

  ```c++
  ADD_METHOD_TO(UserController::handler1,"/users/.*",Post); /// `/users/`で始まるすべてのパスにマッチ
  ADD_METHOD_TO(UserController::handler2,"/{name}/[0-9]+",Post); /// 名前の文字列と数字で構成されたパスにマッチ
  ```

- #### 正規表現によるマッピング

  ここまでに示した方法では正規表現の機能に制限があります。より柔軟に正規表現を扱うために、`ADD_METHOD_VIA_REGIX`を使うことができます。

  ```c++
  ADD_METHOD_VIA_REGEX(UserController::handler1,"/users/(.*)",Post); /// `/users/`から始まるパスにマッチし、それ以降のパスをhandler1の引数に渡す
  ADD_METHOD_VIA_REGEX(UserController::handler2,"/.*([0-9]*)",Post); /// 数字で終わるパスにマッチし、その数字をhandler2の引数に渡す
  ADD_METHOD_VIA_REGEX(UserController::handler3,"/(?!data).*",Post); /// `/data`で始まらない全てのパスにマッチ
  ```

  As can be seen, parameter mapping can also be done using regular expressions, and all strings matched by subexpressions will be mapped to the parameters of the handler in order.

  > **It should be noted that when using regular expressions, you should pay attention to matching conflicts (multiple different handlers are matched). When conflicts happen in the same controller, drogon will only execute the first handler (the one registered in the framework first). When conflicts happen between different controllers, it is uncertain which handler will be executed. Therefore, users need to avoid these conflicts.**

# Next: [WebSocketController](/ENG/ENG-04-3-Controller-WebSocketController)
