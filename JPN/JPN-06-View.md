##### Other languages: [简体中文](/CHN/CHN-06-视图)

# View

### Viewsについて

最近ではフロントエンド技術も広まり、バックエンド側ではフロントエンド側にデータだけを渡せばいいようになりました。しかし、よいWebフレームワークというのはバックエンドレンダリング、つまり動的にHTMLを生成できる機能を備えておくべきでしょう。Viewはそれに役立ちます。Viewは名前の通り表示についてのみ担当し、複雑なビジネスロジックについてはcontrollerに担当させるべきです。

初期のWebアプリケーションでは、HTMLを直接ソースコードに埋め込んで動的なHTMLページを生成していました。これは非効率的でわかりにくいので、JSPのような逆にHTMLの中にソースコードを埋め込む形式の言語が登場しました。もちろんdrogonでは後者を採用しています。ご存じの通りC++はコンパイル型の言語なので、まずHTMLに埋め込まれたC++をC++のソースコードに変換し、その後コンパイルする必要があります。drogonでは専用のCSP(C++ Server Pages)言語を使い、drogon_ctlコマンドでC++コードへと変換します。

### Drogon's CSP

drogonのCSPは、特殊な記号でC++をHTMLの中に埋め込むシンプルな形式で、具体的には以下に示す通りです。

- `<%inc` と `%>` に挟まれた部分では、必要なヘッダーファイルを定義します。ここには`#include`文のみを書きます。例えば、`<%inc#include "xx.h" %>`のような形です。ただ、多くの一般的なヘッダーはdrogonによって自動的にインクルードされます。なので、このタグを使う機会はあまりありません。

- `<%c++` と `%>` に挟まれた部分は、C++コードとして解釈されます。例えば、`<c++ std:string name="drogon"; %>`のように書きます。
- C++コードは通常そのままC++ソースファイルに書き込まれますが、2つの例外があります。
  - `@@` はControllerから渡されたデータを表し、例えばそこから表示すべきデータを取得できます。
  - `$$` はページの内容を表すストリームオブジェクトで、ページに表示するコンテンツを`<<`演算子で出力できます。

- `[[` と `]]`に挟まれた部分は変数名と解釈されます。これはControllerから渡されたデータのキーとして解釈され、その値がページに出力されます。`[[` と `]]`の前後にあるスペースは無視されます。`[[` と `]]`は同じ行に書く必要があります。パフォーマンス上の理由から、`const char *` `std::string` `const std::string`の3つの型のみに対応しています。他の型のデータを出力する場合は上述の`$$`を使った方法を利用してください。
- `{%` と `%}`に挟まれた部分は、C++の式や変数名（Controllerからの値ではない）として解釈され、その値がページに出力されます。つまるところ、 `{%val.xx%}` と `<%c++$$<<val.xx;%>`は全く同じ意味です。ですが、こちらの方がシンプルでわかりやすいでしょう。上に同じく、タグは行をまたいではいけません。
- `<%view` と `%>`の間にある文字列は、sub-viewの名前として解釈されます。drogonは対応するviewを探し、このタグの位置に内容を書き込みます。名前の前後にあるスペースは無視されます。`<%view` と `%>` が行をまたいではいけません。sub-viewは複数回ネストする事ができますが、再帰はしないようにしてください。
- `<%layout` と `%>`の間にある文字列は、レイアウト名として解釈されます。drogonは対応するレイアウトを探し、viewの内容をレイアウトの対応する位置（`[[]]`の位置）に埋め込みます。レイアウト名の前後のスペースは無視され、改行が含まれてはなりません。複数回のネストをすることができますが、再帰はしないようにしてください。また、1つのファイルで継承できるレイアウトは1つのみで、複数個のレイアウトを利用する事はできません。

### viewの使用例

drogonのHTTPレスポンスはControllerのハンドラで作成するので、viewによるレスポンスも同様にハンドラで行い、以下のようなインターフェースで利用できます。

```c++
static HttpResponsePtr newHttpViewResponse(const std::string &viewName,
                                           const HttpViewData &data);
```

これはHttpResponseクラスのstatic関数で、以下に示す2つの引数を取ります。

- **viewName**: Viewの名前。CSPのファイル名です (**拡張子は省略可能**);
- **data**: controllerからviewへ渡すデータ。型は`HttpViewData`型で、これは特殊なMapで、自由な型を保存・取得できます。 詳細は [HttpViewData API] (API-HttpViewData) を参照してください。

見ての通り、controllerではviewのヘッダーファイルをインクルードしたりする必要はなく、両者は分離されています。唯一の接点がdata変数で、これについては双方で合わせる必要があります

### 簡単な例

それでは、ブラウザから送られたHTTPリクエストのパラメータを表示するHTMLページを返すviewを作ってみましょう。

今回は直接HttpAppFrameworkによるハンドラを定義します。mainファイルのrun()よりも前にに以下のようなコードを追加してください。

```c++
drogon::HttpAppFramework::instance()
        .registerHandler("/list_para",
                        [=](const HttpRequestPtr &req,
                            std::function<void (const HttpResponsePtr &)> &&callback)
                        {
                            auto para=req->getParameters();
                            HttpViewData data;
                            data.insert("title","ListParameters");
                            data.insert("parameters",para);
                            auto resp=HttpResponse::newHttpViewResponse("ListParameters.csp",data);
                            callback(resp);
                        });
```

上のコードはラムダ式を利用してパスが`/list_para`に対してのハンドラを登録し、リクエストのパラメータをviewに渡して表示しています。
次に、viewsフォルダに移動してListParameters.cspを作成し、次のように書き込みます。

```html
<!DOCTYPE html>
<html>
<%c++
    auto para=@@.get<std::unordered_map<std::string,std::string,utils::internal::SafeStringHash>>("parameters");
%>
<head>
    <meta charset="UTF-8">
    <title>[[ title ]]</title>
</head>
<body>
    <%c++ if(para.size()>0){%>
    <H1>Parameters</H1>
    <table border="1">
      <tr>
        <th>name</th>
        <th>value</th>
      </tr>
      <%c++ for(auto iter:para){%>
      <tr>
        <td>{%iter.first%}</td>
        <td><%c++ $$<<iter.second;%></td>
      </tr>
      <%c++}%>
    </table>
    <%c++ }else{%>
    <H1>no parameter</H1>
    <%c++}%>
</body>
</html>
```

次のように`drogon_ctl`コマンドを利用してListParameters.cspをC++ソースファイルへ変換できます。

```shell
drogon_ctl create view ListParameters.csp
```

変換が終わると、カレントディレクトリにListParameters.h と ListParameters.ccの2つのファイルが出力されます。このファイルはアプリケーションにコンパイルして組み込むことができます。

プロジェクトをCMakeで再コンパイルして実行し、ブラウザで`http://localhost/list_para?p1=a&p2=b&p3=c`にアクセスしてみると次のように表示されます。

![view page](https://drogonframework.github.io/drogon-docs/images/viewdemo.png)

これでバックエンドによるHTMLレンダリングを追加する事ができました。

### CSPファイルの処理の自動化

**Note: プロジェクトが `drogon_ctl` コマンドで作成された場合、 この項で説明している事はすでに`drogon_ctl`によって行われています。**

Obviously, it is too inconvenient to manually run the drogon_ctl command every time you modify the csp file. We can put the processing of drogon_ctl into the CMakeLists.txt file. Still use the previous example as an example. Let's assume that we put all the csp files In the views folder, CMakeLists.txt can be added as follows:

```cmake
FILE(GLOB SCP_LIST ${CMAKE_CURRENT_SOURCE_DIR}/views/*.csp)
foreach(cspFile ${SCP_LIST})
  message(STATUS "cspFile:" ${cspFile})
  execute_process(COMMAND basename ARGS "-s .csp ${cspFile}" OUTPUT_VARIABLE classname)
  message(STATUS "view classname:" ${classname})
  add_custom_command(
    OUTPUT ${classname}.h ${classname}.cc
    COMMAND drogon_ctl ARGS create view ${cspFile}
    DEPENDS ${cspFile}
    VERBATIM)
  set(VIEWSRC ${VIEWSRC} ${classname}.cc)
endforeach()
```

Then add a new source file collection ${VIEWSRC} to the add_executable statement as follows:

```cmake
Add_executable(webapp ${SRC_DIR} ${VIEWSRC})
```

### viewの動的コンパイル・ロード

drogonは以下の関数によって実行時にCSPを動的にコンパイル・ロードする機能を提供しています。

```c++
void enableDynamicViewsLoading(const std::vector<std::string> &libPaths);
```

The interface is a member method of `HttpAppFramework`, and the parameter is an array of strings representing a list of directories in which the view csp file is located. After calling this interface, drogon will automatically search for csp files in these directories. After discovering new or modified csp files, the source files will be automatically generated, compiled into dynamic library files and loaded into the application. The application process does not need to be restarted. Users can experiment on their own and observe the page changes caused by the modification of csp file.

Obviously, this function depends on the development environment. If both drogon and webapp are compiled on this server, there should be no problem in dynamically loading the csp page.

> **Note: 動的Viewをアプリケーションに静的にリンクしないでください。viewが静的リンクされている場合、動的viewによって更新する事ができません。開発時はコンパイル対象外のディレクトリにViewのファイルを移動させてください。**

> **Note: この機能はHTMLページを開発しているときに最適な物です。セキュリティと安全のために、本番環境においては直接CSPファイルをターゲットに指定する事をおすすめします。**

> **Note: If a `symbol not found` error occurs while loading a dynamic view, please use the `cmake .. -DCMAKE_ENABLE_EXPORTS=on` to configure your project, or uncomment the last line (`set_property(TARGET ${PROJECT_NAME} PROPERTY ENABLE_EXPORTS ON)`) in your project's CMakeLists.txt, and then rebuild the project**

# Next: [Session](/ENG/ENG-07-Session)
