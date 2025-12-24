##### Other languages: [简体中文](/CHN/CHN-04-控制器-简介)

# Controller - はじめに

controllerはWeb開発においてとても重要です。URLやHTTPメゾットの指定、処理するfilterの設定も行います。Drogonはネットワーク通信やHTTPプロトコルの分析を行えるため、開発者はcontrollerのロジックだけに注力できます。

それぞれのcontrollerは複数の"ハンドラー"と呼ばれる処理用の関数を持つことができ、次のように定義します。

```c++
Void handlerName(const HttpRequestPtr &req,
                  std::function<void (const HttpResponsePtr &)> &&callback,
                 ...);
```

`req`はHTTPリクエストに関するオブジェクトで、スマートポインタにより管理されています。
`callback`はコールバック用の関数オブジェクトです。controllerはレスポンス内容のオブジェクト(これもスマートポインタで管理しています)を作成し、コールバック関数を使って結果を返します。Drogonはそれを受け取り、ブラウザにレスポンスを送信します。
最後の`...`はパラメーターのリストで、Drogonがマッピングのルールに従ってHTTPリクエストのパラメータをマッピングします。


お気づきかもしれませんが、これは非同期なインタフェースです。例えば、時間のかかる処理を別のスレッドで行い、完了したらコールバックを呼び出す、といったことができます。

Drogonには`HttpSimpleController`、`HttpController`、`WebSocketController`の3種類のcontrollerがあります。これらを使うためには対応するクラスを継承する必要があります。例えば、`MyClass`という`HttpSimpleController`の実装を作成するには次のようにします。


```c++

class MyClass:public drogon::HttpSimpleController<MyClass>
{
public:
    //TestController(){}
    virtual void asyncHandleHttpRequest(const HttpRequestPtr &req,
                                         std::function<void (const HttpResponsePtr &)> &&callback) override;

    PATH_LIST_BEGIN
    PATH_ADD("/json");
    PATH_LIST_END
};
```

### Controllerのライフサイクル

Drogonに登録されたcontrollerはインスタンスを1つだけ持ち、アプリケーションの実行中は破壊されません。そのため、controllerのクラスにメンバ変数を追加して利用することができます。
注意すべき点として、controllerのハンドラーが呼ばれるとき、IO用スレッドが複数に設定された場合はマルチスレッド環境になります。そのため、ローカル変数以外の変数を利用するときは同時アクセスからの保護が必要です。

# Next: [HttpSimpleController](/JPN/JPN-04-1-Controller-HttpSimpleController)
