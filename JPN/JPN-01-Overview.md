[English](/ENG/ENG-01-Overview) , [简体中文](/CHN/CHN-01-概述)


# 概要

**Drogon** は C++17/20-based HTTPアプリケーションフレームワークです. Drogonを使う事で、様々なWebアプリケーションをC++で簡単に作成することができます.

<!--
**Drogon** is the name of a dragon in the American TV series "Game of Thrones" that I really like.

Drogonの名前の由来は、アメリカのテレビ番組である"Game of Thrones"に登場するドラゴンです。

-->
Drogonの主なプラットフォームLinuxですが、Mac OSやFreeBSD、Windowsにも対応しています.

Drogonの持つ主な特徴としては次のようなものがあります。

* Use a non-blocking I/O network lib based on epoll (kqueue under macOS/FreeBSD) to provide high-concurrency, high-performance network IO, please visit the [TFB Tests Results](https://www.techempower.com/benchmarks/#section=data-r19&hw=ph&test=composite) for more details;
* 完全な非同期プログラミング; Provide a completely asynchronous programming mode;
* Support Http1.0/1.1 (server side and client side);
* テンプレートによるシンプルなリフレクションにより、内部機構,コントローラー,ビューを分離.<!-- Based on template, a simple reflection mechanism is implemented to completely decouple the main program framework, controllers and views.-->
* cookie対応と組み込みのセッション管理機能。<!--Support cookies and built-in sessions;>
* back-end rendering(?)に対応。controllerによってデータを作成、viewでHTMLを生成することがでます。viewはCSPファイルで記述し、CSPタグによってC++コードをHTML内に埋め込む事が出来ます。drogonのプログラムによって、CSPファイルはC++ソースに変換されコンパイルされます。<!--Support back-end rendering, the controller generates the data to the view to generate the Html page. Views are described by CSP template files, C++ codes are embedded into Html pages through CSP tags. And the drogon command-line tool automatically generates the C++ code files for compilation;-->
* viewの動的ロードに対応<!--Support view page dynamic loading (dynamic compilation and loading at runtime);-->
* (便利的な言葉)かつ柔軟なcontrollerへのパス割り当て（？） Provide a convenient and flexible routing solution from the path to the controller handler;
* filter機能によって、controllerより前に統一された機能(ログイン認証、httpメゾットの制約など)を容易に作成できる<!--Support filter chains to facilitate the execution of unified logic (such as login verification, Http Method constraint verification, etc.) before handling HTTP requests;-->
* OpenSSLによるHTTPSサポート。<!--Support https (based on OpenSSL);-->
* Websocketサポート(サーバー・クライアント両方) <!--Support WebSocket (server side and client side);--> <!--文面に違和感-->
* JSONによるリクエスト/レスポンス。Restful APIの実装に最適 <!--Support JSON format request and response, very friendly to the Restful API application development;-->
* ファイルのアップロード/ダウンロード。<!--Support file download and upload;-->
* gzipやbrotilによる送受信。<!--Support gzip, brotli compression transmission;-->
* Support pipelining; <!--???-->
* 軽量なコマンドラインツールであるdrogon_ctlを提供: drogonでの様々なclassや、viewの生成を簡単に行えます<!--Provide a lightweight command line tool, drogon_ctl, to simplify the creation of various classes in Drogon and the generation of view code;-->
* 呑んブロッキングI/Oによる、非同期なデータベースへの読み書き。(PostgreSQL、MySQL(MariaDB)に対応)<!--Support non-blocking I/O based asynchronously reading and writing database (PostgreSQL and MySQL(MariaDB) database);-->
* スレッドプールによる非同期なsqlite3データベースへの読み書き。<!--Support asynchronously reading and writing sqlite3 database based on thread pool;-->
* ARMアーキテクチャへのサポート。<!--Support ARM Architecture;-->
* 軽量なORMによるオブジェクトとデータベース間の双方向マッピング。<!--Provide a convenient lightweight ORM implementation that supports for regular object-to-database bidirectional mapping;-->
* ロード時のプラグイン読み込み。<!--Support plugins which can be installed by the configuration file at load time;-->
* 組み込みのジョイントポイントによるAOP<!-- AOP with build-in joinpoints. --> <!--なにもわからん。-->

# Next: [drogonの導入](/JPN/JPN-02-Installation)
