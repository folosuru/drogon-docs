[English](/ENG/ENG-01-Overview) , [简体中文](/CHN/CHN-01-概述)


# 概要

**Drogon** は C++17/20-based HTTPアプリケーションフレームワークです. Drogonを使う事で、様々なWebアプリケーションをC++で簡単に作成することができます.

<!--
**Drogon** is the name of a dragon in the American TV series "Game of Thrones" that I really like.

Drogonの名前の由来は、アメリカのテレビ番組である"Game of Thrones"に登場するドラゴンです。

-->
Drogonの主なプラットフォームLinuxですが、Mac OSやFreeBSD、Windowsにも対応しています.

Drogonの持つ主な特徴としては次のようなものがあります。

* epoll(macOS/FreeBSDではkqueue)を利用したノンブロッキングIOによる、高い並列性と高効率なネットワークIO。詳細は[TFB Tests Results](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite)をご覧ください。
* 完全な非同期プログラミング
* Support Http1.0/1.1 (server side and client side);
* テンプレートによるシンプルなリフレクションにより、内部機構,コントローラー,ビューを分離.
* cookie対応と組み込みのセッション管理機能。<!--Support cookies and built-in sessions;>
* バックエンドでのレンダリングに対応。controllerによってデータを作成、viewでHTMLを生成することがでます。viewはCSPファイルで記述し、CSPタグによってC++コードをHTML内に埋め込む事が出来ます。drogonのプログラムによって、CSPファイルはC++ソースに変換されコンパイルされます。
* viewの動的ロードに対応
* 使いやすく柔軟なcontrollerへのパス割り当て
* filter機能によって、controllerより前に統一された機能(ログイン認証、httpメゾットの制約など)を容易に作成可能
* OpenSSLによるHTTPSサポート
* サーバー・クライアント双方のWebsocketサポート
* JSONによるリクエスト/レスポンス。Restful APIの実装に最適
* ファイルのアップロード/ダウンロード
* gzipやbrotilによる送受信
* Support pipelining; <!--???-->
* 軽量なコマンドラインツールであるdrogon_ctlを提供: drogonでの様々なclassや、viewの生成を簡単に行えます<!--Provide a lightweight command line tool, drogon_ctl, to simplify the creation of various classes in Drogon and the generation of view code;-->
* ノンブロッキングI/Oによる、非同期なデータベースへの読み書き。(PostgreSQL、MySQL(MariaDB)に対応)<!--Support non-blocking I/O based asynchronously reading and writing database (PostgreSQL and MySQL(MariaDB) database);-->
* スレッドプールによる非同期なsqlite3データベースへの読み書き。<!--Support asynchronously reading and writing sqlite3 database based on thread pool;-->
* ARMアーキテクチャへのサポート。<!--Support ARM Architecture;-->
* 軽量なORMによるオブジェクトとデータベース間の双方向マッピング。
* コンフィグファイルによるロード時のプラグイン読み込み。<!--Support plugins which can be installed by the configuration file at load time;-->
* 組み込みのジョインポイントによるAOP<!-- AOP with build-in joinpoints. --> <!--なにもわからん。-->

# Next: [drogonの導入](/JPN/JPN-02-Installation)
