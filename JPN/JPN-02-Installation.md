##### Other languages: [English](/ENG/ENG-02-Installation) , [简体中文](/CHN/CHN-02-安装)

# 導入

この章ではUbuntu 24.04, CentOS 7.5, MacOS 12.2を例として用います。他の環境でも同様です。
<!--This section takes Ubuntu 24.04, CentOS 7.5, MacOS 12.2 as an example to introduce the installation process. Other systems are similar;-->

## 前提要件

* Linuxカーネルは64-bitかつ2.6.9以上
* gccは5.4.0以降、バージョン11以上を推奨
* cmake 3.5以上
* git

## 依存するライブラリ

* ビルドイン
  * trantor, ノンブロッキングI/O ネットワークライブラリ。drogonの作者によって開発されており、gitのsubmoduleとして提供されているため追加の操作は不要です。<!-- a non-blocking I/O C++ network library, also developed by the author of Drogon, has been used as  a git repository submodule, no need to install in advance;-->
* 必須
  * jsoncpp, C++用JSONライブラリ。バージョン1.7以降を使用してください。<!--JSON's c++ library, the version should be **no less than 1.7**;-->
  * libuuid, uuidを生成するCライブラリ。<!--generating c library of uuid;-->
  * zlib, 圧縮送信(?)に用います。<!--used to support compressed transmission;-->
* 任意
  * boost, 1.61以降を使用してください。C++コンパイラがC++17をサポートしておらず、`std::filesystem`がサポートされていない時のみ使用されます。<!--the version should be **no less than 1.61**, is required only if the C++ compiler does not support C++ 17 and if the STL doesn't fully support `std::filesystem`.-->
  * OpenSSL, 導入することで、drogonがHTTPS通信を扱えるようになります。導入されていない場合、HTTP通信のみを扱います。<!--after installed, drogon will support HTTPS as well, otherwise drogon only supports HTTP.-->
  * c-ares, 導入することでdrogonのDNSがより効率的になります<!--after installed, drogon will be more efficient with DNS;-->
  * libbrotli, 導入することでHTTPレスポンスでのbrotliによる圧縮をサポートします。<!--after installed, drogon will support brotli compression when sending HTTP responses;-->

  * postgreSQLやmariadb、sqlite3のライブラリが導入されている場合、それらのデータベースへのアクセスをサポートします<!--the client development libraries of postgreSQL, mariadb and sqlite3, if one or more of them is installed, drogon will support access to the according database.-->
  * hiredis, redisに接続できるようになります<!--after installed, drogon will support access to redis.-->
  * gtest, ユニットテストがコンパイル去れるようになります。<!--after installed, the unit tests can be compiled.-->
  * yaml-cpp, コンフィグをyaml形式で書くことができるようになります。<!--after installed, drogon will support config file with yaml format.-->

## システムの準備の例<!--直訳過ぎない？-->

#### Ubuntu 24.04

* 環境

  ```shell
  sudo apt install git gcc g++ cmake
  ```

* jsoncpp

  ```shell
  sudo apt install libjsoncpp-dev
  ```

* uuid

  ```shell
  sudo apt install uuid-dev
  ```

* zlib

  ```shell
  sudo apt install zlib1g-dev
  ```

* OpenSSL (任意, HTTPS対応させる場合)

  ```shell
  sudo apt install openssl libssl-dev
  ```

#### Arch Linux

* Environment

  ```shell
  sudo pacman -S git gcc make cmake
  ```

* jsoncpp

  ```shell
  sudo pacman -S jsoncpp
  ```

* uuid

  ```shell
  sudo pacman -S uuid
  ```

* zlib

  ```shell
  sudo pacman -S zlib
  ```

* OpenSSL (任意, HTTPS対応させる場合)

  ```shell
  sudo pacman -S openssl libssl
  ```

#### CentOS 7.5

* Environment

  ```shell
  yum install git
  yum install gcc
  yum install gcc-c++
  ```

  <!--The default installed cmake version is too low, use source installation-->
  ```shell
  # デフォルトのcmakeのバーフォンが古いため、ソースからビルド
  git clone https://github.com/Kitware/CMake
  cd CMake/
  ./bootstrap && make && make install
  ```

  ```shell
  # Upgrade gcc
  yum install centos-release-scl
  yum install devtoolset-11
  scl enable devtoolset-11 bash
  ```

  > **Note: Command `scl enable devtoolset-11 bash` only activate the new gcc temporarily until the session is end. If you want to always use the new gcc, you could run command `echo "scl enable devtoolset-11 bash" >> ~/.bash_profile`, system will automatically activate the new gcc after restarting.**

* jsoncpp

  ```shell
  git clone https://github.com/open-source-parsers/jsoncpp
  cd jsoncpp/
  mkdir build
  cd build
  cmake ..
  make && make install
  ```

* uuid

  ```shell
  yum install libuuid-devel
  ```

* zlib

  ```shell
  yum install zlib-devel
  ```

* OpenSSL (Optional, if you want to support HTTPS)

  ```shell
  yum install openssl-devel
  ```

#### MacOS 12.2

* Environment

  <!--
  All the essentials are inherent in MacOS, you only need to upgrade it.
-->
必要なものは既にMacOSにあるため、アップグレードするのみです

  ```shell
  # upgrade gcc
  brew upgrade
  ```

* jsoncpp

  ```shell
  brew install jsoncpp
  ```

* uuid

  ```shell
  brew install ossp-uuid
  ```

* zlib

  ```shell
  yum install zlib-devel
  ```

* OpenSSL (Optional, if you want to support HTTPS)

  ```shell
  brew install openssl
  ```

#### Windows

* 環境 (Visual Studio 2019)
  <!--Install Visual Studio 2019 professional 2019, at least included these options:-->
  Visual Studio 2019 professional 2019を導入してください。また、次のオプションを含めて下さい。
  * MSVC C++ building tools
  * Windows 10 SDK
  * C++ CMake tools for windows
  * Test adaptor for Google Test

  パッケージマネージャの`conan`はDrogonに必要な物を全て導入することができます。Pythonが導入されている場合、pipを使って`conan`をインストールすることができます。
  <!--
  `conan` package manager could provide all dependencies that Drogon projector needs。If python   is installed on system, you could install `conan` package manager via pip. 
  -->
  ```
  pip install conan
  ```
  <!--
  > of course you can download the installation file from `connan` [official website](https://  conan.io/) to install it also.
  -->

  > (もちろん)`conan`を[公式サイト](https://conan.io/)からダウンロードしてインストールすることもできます。

  <!--
  Create`conanfile.txt`and add the following content to it:
  -->
  `conanfile.txt`を作成し、次の内容を書き込んでください。

  * jsoncpp

    ```txt
    [requires]
    jsoncpp/1.9.4
    ```

  * uuid
    <!--
    No installation is required, the Windows 10 SDK already includes the uuid library.
    -->
    Windows 10 SDKにuuidのライブラリが含まれているため、追加の導入は不要です。

  * zlib

    ```txt
    [requires]
    zlib/1.2.11
    ```

  * OpenSSL (任意, HTTPS対応させる場合)

    ```txt
    [requires]
    openssl/1.1.1t
    ```

## データベース (任意)

<!--
> **Note: These libraries below are not mandatory. You could choose to install one or more database according to your actual needs.**

> **Note: If you want to develop your webapp with database, please install the database develop environment first, then install drogon, otherwise you will encounter a `NO DATABASE FOUND` issue.**
-->

> **Note: これらのライブラリは必須ではありません。必要な物のみをインストールしてください。**

> **Note: データベースを使うつもりの場合、まずデータベースをインストールしてから導入してからdrogonを導入して下さい。**

* #### PostgreSQL
  <!--
  PostgreSQL's native C library libpq needs to be installed. The installation is as follows:
  -->
  PostgreSQLのネイティブCライブラリの libpqが必要です。次のようにインストールします:

  * `ubuntu 16`: `sudo apt-get install postgresql-server-dev-all`
  * `ubuntu 18`: `sudo apt-get install postgresql-all`
  * `ubuntu 24`: `sudo apt-get install postgresql-all`
  * `arch`: `sudo pacman -S postgresql`
  * `centOS 7`: `yum install postgresql-devel`
  * `MacOS`: `brew install postgresql`
  * `Windows conanfile`: `libpq/13.4`

* #### MySQL

  MySQLのネイティブライブラリは非同期な読み書きに対応していません。MySQLの元の開発者がメンテナンスするMariaDBではMySQLと互換性があり、そのライブラリでは非同期な読み書きに対応しています。そのため、DrogonではMariaDBのライブラリを使ってMySQLをサポートします。なお、ベストプラクティスとして、MariaDBとMySQLを同じシステムに導入しないでください。

  MariaDBの導入は次のように行います
<!--
  MySQL's native library does not support asynchronous read and write. Fortunately, MySQL also has a version of MariaDB maintained by the original developer community. This version is compatible with MySQL, and its development library supports asynchronous read and write. Therefore, Drogon uses the MariaDB development library to provide the right MySQL support, as a best practice，your operating system shouldn't install both Mysql and MariaDB at the same time.

  MariaDB installation is as follows：
-->
  * `ubuntu 18.04`: `sudo apt install libmariadbclient-dev`
  * `ubuntu 24.04`: `sudo apt install libmariadb-dev-compat libmariadb-dev`
  * `arch`: `sudo pacman -S mariadb`
  * `centOS 7`: `yum install mariadb-devel`
  * `MacOS`: `brew install mariadb`
  * `Windows conanfile`: `libmariadb/3.1.13`

* #### Sqlite3

  * `ubuntu`: `sudo apt-get install libsqlite3-dev`
  * `arch`: `sudo pacman -S sqlite3`
  * `centOS`: `yum install sqlite-devel`
  * `MacOS`: `brew install sqlite3`
  * `Windows conanfile`: `sqlite3/3.36.0`

* #### Redis
  * `ubuntu`: `sudo apt-get install libhiredis-dev`
  * `ubuntu`: `sudo pacman -S redis`
  * `centOS`: `yum install hiredis-devel`
  * `MacOS`: `brew install hiredis`
  * `Windows conanfile`: `hiredis/1.0.0`

> **Note: 上記のコマンドの中には開発用ライブラリのみがインストールされる物があります。サーバーを導入したい場合は、ｇｇｒｋｓ。(please use Google search yourself.)**
<!--
> **Note: Some of the above commands only install the development library. If you want to install a server also, please use Google search yourself.**
-->
## Drogonの導入

<!--
Assuming that the above environment and library dependencies are all ready, the installation process is very simple;
-->
上述の開発環境や依存するライブラリが導入されていれば、導入はとても簡単です。

* #### ソースからインストール(Linux)

  ```shell
  cd $WORK_PATH
  git clone https://github.com/drogonframework/drogon
  cd drogon
  git submodule update --init
  mkdir build
  cd build
  cmake ..
  make && sudo make install
  ```

  > 既定ではデバッグ用でビルドされるようになっています。リリース版にする場合、cmakeコマンドに次のようなパラメータを追加してください。

  ```shell
  cmake -DCMAKE_BUILD_TYPE=Release ..
  ```

  After the installation is complete, the following files will be installed in the system（One can change the installation location with the CMAKE_INSTALL_PREFIX option）:

  インストールが完了すると、これらのファイルがインストールされます。（`CMAKE_INSTALL_PREFIX`オプションで変更することができます。）

  * drogon のヘッダーファイルは /usr/local/include/drogon
  * drogonのライブラリのlibdrogon.a は/usr/local/lib 
  * コマンドラインツールの drogon_ctl は /usr/local/bin
  * trantor のヘッダーファイルは /usr/local/include/trantor
  * trantor のライブラリの libtrantor.a は /usr/local/lib
* #### ソースからインストール(Windows)

  1. ソースファイルをダウンロード

      ```dos
      cd $WORK_PATH
      git clone https://github.com/drogonframework/drogon
      cd drogon
      git submodule update --init
      ```

  2. 依存するライブラリをインストール

    `conan`を使ってインストールする場合
    ```dos
    mkdir build
    cd build
    conan profile detect --force
    conan install .. -s compiler="msvc" -s compiler.version=193  -s compiler.cppstd=17 -s build_type=Debug  --output-folder . --build=missing
    ```

    > `conanfile.txt`を使う事で導入する物のバージョンを変更できます
  3. コンパイル・インストール
    ```dos
    cmake ..  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_TOOLCHAIN_FILE="conan_toolchain.cmake" -DCMAKE_POLICY_DEFAULT_CMP0091=NEW -DCMAKE_INSTALL_PREFIX="D:"
      cmake --build . --parallel --target install
    ```
    > **Note: conanとcmakeでのビルドタイプは同じものにしてください**

    インストールが完了すると、これらのファイルがインストールされます。（`CMAKE_INSTALL_PREFIX`オプションで変更することができます。）

    * drogon のヘッダーファイルは `D:/include/drogon`
    * drogonのライブラリのlibdrogon.a は`D:/bin`
    * コマンドラインツールの drogon_ctl は `D:/bin`
    * trantor のヘッダーファイルは `D:/include/trantor`
    * trantor のライブラリの libtrantor.a は `D:/include/trantor`
  
    `bin`と`cmake`ディレクトリを`path`に追加
    ```
    D:\bin
    ```
    ```
    D:\lib\cmake\Drogon
    ```
    ```
    D:\lib\cmake\Trantor
    ```

* #### vcpkgを使って導入する
  
  [読むのがめんどくさい人向け(英語)](https://www.youtube.com/watch?v=0ojHvu0Is6A)

  **vcpkgをインストール:**

  1. `vcpkg`を`git`を使って導入.

     > note: vcpkgを更新する場合、`git pull`を実行してください。

  2. 環境変数**_path_**に`vpckg`を追加.
  3. インストールできているか確認するには`vcpkg`あるいは`vcpkg.exe`と入力してください。

  **Drogonをインストール:**

  1. Drogonをインストールするためには、次のコマンドを入力してください
     * 32Bit: `vcpkg install drogon`
     * 64Bit: `vcpkg install drogon:x64-windows`
     * 任意 : `vcpkg install jsoncpp:x64-windows zlib:x64-windows openssl:x64-windows sqlite3:x64-windows libpq:x64-windows libpqxx:x64-windows drogon[core,ctl,sqlite3,postgres,orm]:x64-windows`  
     > Note: `drogon_ctl`はデフォルトでは導入されません。`vcpkg install drogon[core,ctl]`で導入できます。

     note:

     * もしパッケージがインストールされておらず、エラーが表示された場合はそれらをインストールして下さい。次のように:
      
        zlib : `vcpkg install zlib`, `vcpkg install zlib:x64-windows`(64bit)

     * インストールされているパッケージを確認するには:
        `vcpkg list`

     * `drogon_ctl`を使うには、`vcpkg install drogon[ctl]`(32bit) あるいは `vcpkg install drogon[ctl]:x64-windows`(64bit)を使用してください。他のインストールオプションは`vcpkg search drogon`で閲覧できます。

  2. **_drogon_ctl_** コマンドや依存パッケージを使用するため, 環境変数PATHを設定する必要があります. ここまでの説明通りに進めてきた場合、追加する必要があるのは次の通りです。 <!--2. To add **_drogon_ctl_** command and dependencies, you need to add some variables. By following this guide, you just need to add:-->

       - `C:\dev\vcpkg\installed\x64-windows\tools\drogon`
       - `C:\dev\vcpkg\installed\x64-windows\bin`
       - `C:\dev\vcpkg\installed\x64-windows\lib`
       - `C:\dev\vcpkg\installed\x64-windows\include`
       - `C:\dev\vcpkg\installed\x64-windows\share`
       - `C:\dev\vcpkg\installed\x64-windows\debug\bin`
       - `C:\dev\vcpkg\installed\x64-windows\debug\lib`
       <!--to your windows **_environment variables_**. Then restart/re-open your **_powershell_**.-->
      これらを追加した後、**_powershell_** を再起動して下さい。

  3. **_powershell_** を再起動させたら, `drogon_ctl`あるいは `drogon_ctl.exe`を入力してください。
     ```
     usage: drogon_ctl [-v | --version] [-h | --help] <command> [<args>]
     commands list:
     create                  create some source files(Use 'drogon_ctl help create' for more information)
     help                    display this message
     press                   Do stress testing(Use 'drogon_ctl help press' for more information)
     version                 display version of this tool
     ```
     のように表示されれば、インストールは成功です。

   > Note:
   > you need to be familiar with building cpp libraries by using: independent `gcc` or `g++` (**_[msys2](https://www.msys2.org/), [mingw-w64](https://www.mingw-w64.org/), [tdm-gcc](https://jmeubank.github.io/tdm-gcc/download/)_**) or Microsoft Visual Studio compiler

   > **_make.exe/nmake.exe/ninja.exe_** をcmakeのジェネレータに使うことを検討してください。設定方法が LinuxのMakeと同じなので、誰かがLinux/Windows を使っていて、Linux環境でデプロイしようとしている場合、システムの切り替え時のエラーを発生しにくくできます。

* #### Use Docker Image
  <!--We also provide a pre-build docker image on the [docker hub](https://hub.docker.com/r/drogonframework/drogon). All dependencies of Drogon and Drogon itself are already installed in the docker environment, where users can build Drogon-based applications directly.-->
  Dockerイメージを[docker hub](https://hub.docker.com/r/drogonframework/drogon)で提供しています. 必要なライブラリやdrogonの設定が完了してるため、直接アプリケーションを作ることができます。

* #### Use Nix Package

  There is a Nix package for Drogon which was released in version 21.11.

  > **if you haven't installed Nix:** You can follow the instructions on the [NixOS website](https://nixos.org/download.html).

  You can use the package by adding the following `shell.nix` to your project root:

  ```
  { pkgs ? import <nixpkgs> {} }:
  pkgs.mkShell {
    nativeBuildInputs = with pkgs; [
      cmake
    ];

    buildInputs = with pkgs; [
      drogon
    ];
  
  }
  ```

  Enter the shell by running `nix-shell`. This will install Drogon and enter you into an environment with all its dependencies.

  The Nix package has a few options which you can configure according to your needs:

  | option          | default value |
  | --------------- | ------------- |
  | sqliteSupport   | true          |
  | postgresSupport | false         |
  | redisSupport    | false         |
  | mysqlSupport    | false         |

  Here is an example of how you can change their values:

  ```
    buildInputs = with pkgs; [
      (drogon.override {
        sqliteSupport = false;
      })
    ];
  ```

* #### Use CPM.cmake

  You can use [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) to include the drogon source code:

  ```cmake
  include(cmake/CPM.cmake)

  CPMAddPackage(
      NAME drogon
      VERSION 1.7.5
      GITHUB_REPOSITORY drogonframework/drogon
      GIT_TAG v1.7.5
  )

  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

* #### Include drogon source code locally

  Of course, you can also include the drogon source in your project. Suppose you put the drogon under the third_party of your project directory (don't forget to update submodule in the drogon source directory). Then, you only need to add the following two lines to your project's cmake file:

  ```cmake
  add_subdirectory(third_party/drogon)
  target_link_libraries(${PROJECT_NAME} PRIVATE drogon)
  ```

# Next: [Quick Start](/JPN/JPN-03-Quick-Start)
