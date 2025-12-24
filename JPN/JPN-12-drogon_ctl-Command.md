##### Other languages: [简体中文](/CHN/CHN-12-drogon_ctl命令)

# drogon_ctl - Command

**Drogon**のコンパイルとインストールができた後の最初のプロジェクトの作成には、フレームワークと共にインストールされているコマンドラインツールの`drogon_ctl`の使用をお勧めします。また、短縮版の`gd_ctl`もあるので、好みに応じて選択してください。


主要な機能はdrogonの様々なプロジェクト用のファイルを作成する事です。`dg_ctl help`を実行して対応している機能を確認してみましょう・

```console
$ dg_ctl help
usage: drogon_ctl <command> [<args>]
commands list:
create                  create some source files(Use 'drogon_ctl help create' for more information)
help                    display this message
version                 display version of this tool
press                   Do stress testing(Use 'drogon_ctl help press' for more information)
```

### Version サブコマンド

`version`サブコマンドは、以下のように現在インストールされているdrogonのバージョンを表示します。

```console
$ dg_ctl version
     _
  __| |_ __ ___   __ _  ___  _ __
 / _` | '__/ _ \ / _` |/ _ \| '_ \
| (_| | | | (_) | (_| | (_) | | | |
 \__,_|_|  \___/ \__, |\___/|_| |_|
                 |___/

drogon ctl tools
version:0.9.30.771
git commit:d4710d3da7ca9e73b881cbae3149c3a570da8de4
compile config:-O3 -DNDEBUG -Wall -std=c++17 -I/root/drogon/trantor -I/root/drogon/lib/inc -I/root/drogon/orm_lib/inc -I/usr/local/include -I/usr/include/uuid -I/usr/include -I/usr/include/mysql
```

### Create サブコマンド

`create`サブコマンドは、様々な物を作成するのに用いる、drogon_ctlの主要機能です。`dg_ctl help create` を実行すると、下に示すように詳細な情報が表示されます。

```console
$ dg_ctl help create
Use create command to create some source files of drogon webapp

Usage:drogon_ctl create <view|controller|filter|project|model> [-options] <object name>

drogon_ctl create view <csp file name> [-o <output path>] [-n <namespace>]|[--path-to-namespace] //create HttpView source files from csp file

drogon_ctl create controller [-s] <[namespace::]class_name> //create HttpSimpleController source files

drogon_ctl create controller -h <[namespace::]class_name> //create HttpController source files

drogon_ctl create controller -w <[namespace::]class_name> //create WebSocketController source files

drogon_ctl create filter <[namespace::]class_name> //create a filter named class_name

drogon_ctl create project <project_name> //create a project named project_name

drogon_ctl create model <model_path> //create model classes in model_path
```

- #### Viewの作成
  `dg_ctl create view`はCSPファイルからソースファイルを生成するのに利用します。詳しくは[View](/ENG/ENG-06-View) の項を参照してください。

  基本的には、このコマンドを直接使う必要はありません。CMakeファイルの設定を行ってこのコマンドを自動的に実行する方が望ましいためです。CSPファイルが`UsersList.csp`だった場合の使用例は以下のようになります。

  ```shell
  dg_ctl create view UsersList.csp
  ```

- #### Controllerの作成
  `dg_ctl create controller`は、controllerのソースファイルを作成するのに役立ちます。3種類のcontrollerの作成に対応しています。

  - HttpSimpleControllerを作成する場合は以下のようにコマンドを実行します。

  ```shell
  dg_ctl create controller SimpleControllerTest
  dg_ctl create controller webapp::v1::SimpleControllerTest
  ```

  最後のパラメータはcontrollerのクラス名で、名前空間名を先頭に着けることができます。

  - HttpControllerを作成する場合は以下のようにコマンドを実行します。


  ```shell
  dg_ctl create controller -h ControllerTest
  dg_ctl create controller -h api::v1::ControllerTest
  ```

  - WebSocketController を作成する場合は以下の通りです。

  ```shell
  dg_ctl create controller -w WsControllerTest
  dg_ctl create controller -w api::v1::WsControllerTest
  ```

- #### Filterの作成

  `dg_ctl create filter`はfilter用のソースファイルを作成するのに役立ちます。詳しくは[Middleware and Filter](/ENG/ENG-05-Middleware-and-Filter) の項目を参照してください。

  ```shell
  dg_ctl create filter LoginFilter
  dg_ctl create filter webapp::v1::LoginFilter
  ```

- #### プロジェクトの作成

  新しくdrogonを使ったアプリケーションを作成するには、以下のようにdrogon_ctlを使うのが最善の方法です。

  ```shell
  dg_ctl create project ProjectName
  ```

  After the command is executed, a complete project directory will be created in the current directory. The directory name is `ProjectName`, and the user can directly compile the project in the build directory (cmake .. && make). Of course, it does not have any business logic.

  コマンドが実行し終わると、カレントディレクトリにプロジェクト用のディレクトリにが用意されます。この例ではディレクトリ名は`ProjectName`で、ビルドディレクトリですぐにコンパイルする事ができます(cmake .. && make)。もちろん、まだビジネスロジックは何も含まれていません。

  ディレクトリ構造は次のようになります:

  ```console
  ├── build                         ビルド用のディレクトリ
  ├── CMakeLists.txt                CMakeの設定ファイル
  ├── cmake_modules                 サードパーティーのライブラリの探索用のスクリプト
  │   ├── FindJsoncpp.cmake
  │   ├── FindMySQL.cmake
  │   ├── FindSQLite3.cmake
  │   └── FindUUID.cmake
  ├── config.json                   アプリケーション用の設定ファイル。コンフィグファイルの項を参照。
  ├── controllers                   controllerのソースファイル用のディレクトリ
  ├── filters                       filterのソースファイル用のディレクトリ
  ├── main.cc                       main
  ├── models                        データベースのmodel用。11.2.5参照。
  │   └── model.json
  ├── tests                         単体/統合テスト用のディレクトリ
  │   └── test_main.cc              テスト用のエントリポイント
  └── views                         CSPファイル用のディレクトリ。コンパイル時に自動で変換されるので主導での変換は不要。
  ```
- #### modelの作成

  Use the `dg_ctl create model` command to create database model source files. The last parameter is the directory where models is stored. This directory must contain a model configuration file named `model.json` to tell dg_ctl how to connect to the database and which tables to be mapped.

  For example, if you want to create models in the project directory mentioned above, execute the following command in the project directory:

  ```shell
  dg_ctl create model models
  ```

  This command will prompt the user that the file will be overwritten directly. After the user enters `y`, it will generate all the model files.

  Other source files need to reference model classes should include model header files, such as:

  ```c++
  #include "models/User.h"
  ```

  Note that the models directory name is included to distinguish between multiple data sources in the same project. See [ORM](/ENG/ENG-08-3-Database-ORM).

### 負荷テスト

`dg_ctl press`コマンドを利用する事で負荷テストをすることができます。以下に利用できるオプションを示します。

- `-n 数値` リクエストの個数(default : 1)
- `-t 数値` スレッドの数(default : 1), CPUの数と同じに設定する事で、最大のパフォーマンスを発揮できます。
- `-c 数値` 同時に行う接続の数(default : 1)
- `-q` 途中経過を表示しない(default: no)

次のようにすることで、HTTPサーバーの負荷テストを行うことができます。

```shell
dg_ctl press -n1000000 -t4 -c1000 -q http://localhost:8080/
dg_ctl press -n 1000000 -t 4 -c 1000 https://www.domain.com/path/to/be/tested
```

# Next: [AOP Aspect-Oriented-Programming](/ENG/ENG-13-AOP-Aspect-Oriented-Programming)
