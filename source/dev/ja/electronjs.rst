===========================================================
ElectronJSとReactを使い簡単なアプリを作成  
===========================================================

なぜ
=======================================

Visual Studio CodeやSlackなどよく使われるアプリケーションがElectronで作られたということで興味が湧いてくるから


目標
=======================================

簡単メモアプリを作成してみる


環境構築
=======================================

開発環境
---------------------------------------

Windows 10 Home








- Server Host : localhost
- Port : 3306
- Database : logger
- Username : root
- Password : maria
- Table : message_log

+------------+-----------+
| Name       | Type      |
+============+===========+
| registered | TIMESTAMP |
+------------+-----------+
| message    | TEXT      |
+------------+-----------+

テストソース
---------------------------------------

Tableに対してアクセスする簡単なUNITテストを用意

DbService.kt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: kotlin

    //~~~省略~~~
    class DbService() {
        init {
            Database.connect(
                "jdbc:mysql://localhost:3306/logger",
                driver = "com.mysql.cj.jdbc.Driver",
                user = "root",
                password = "maria"
            )
        }

        fun register(@NotNull registered: LocalDateTime, @NotNull message: String) {
            MessageLog.insert {
                it[MessageLog.registered] = registered
                it[MessageLog.message] = message
            }
        }
    }

DbServiceTest.kt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: kotlin

    //~~~省略~~~
    class DbServiceTest {
        @Test
        fun testInsert() {
            val ds = DbService()
            transaction {
                MessageLog.deleteAll()

                val expected = mapOf<Int, Pair<LocalDateTime, String>>(
                    0 to Pair(LocalDateTime.of(2021, 9, 1, 1, 1, 1), "first message"),
                    1 to Pair(LocalDateTime.of(2021, 10, 1, 2, 2, 2), "second message"),
                )

                // execute
                expected.values.forEach { ds.register(it.first, it.second) }

                // assert
                MessageLog.selectAll().orderBy(MessageLog.registered).forEachIndexed { index, resultRow ->
                    val col1 = resultRow[MessageLog.registered]
                    val col2 = resultRow[MessageLog.message]

                    assertEquals(
                        expected[index]!!.first, col1, "index %d : registered".format(index)
                    )
                    assertEquals(
                        expected[index]!!.second, col2, "index %d : message".format(index)
                    )
                }
            }
        }
    }

build.gradle.kts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: kotlin

    //~~~省略~~~
    tasks {
        compileKotlin {
            kotlinOptions.jvmTarget = "15"
        }
        compileTestKotlin {
            kotlinOptions.jvmTarget = "15"
        }
        test {
            this.testLogging {
                this.showStandardStreams = true
            }
        }
    }

MariaDBのimage作成
---------------------------------------

imageをダウンロード
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker pull mariadb
    Using default tag: latest
    latest: Pulling from library/mariadb
    ea362f368469: Pull complete
    adb9a1b1379d: Pull complete
    ac5c95406850: Pull complete
    fa48d8b47ec1: Pull complete
    bcf1feb44ac3: Pull complete
    8a5de7784a0f: Pull complete
    b8724b8a281a: Pull complete
    a8a7c3f612d6: Pull complete
    39b09b59e889: Pull complete
    14bc3a6b0a94: Pull complete
    Digest: sha256:5a37e65a6414d78f60d523c4ddcf93d715854337beb46f8beeb1a23d83262184
    Status: Downloaded newer image for mariadb:latest
    docker.io/library/mariadb:latest
    
    C:\Users\tjrdu\projects\action-study>docker images
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    mariadb      latest    d462573d8688   2 weeks ago   410MB
    
    C:\Users\tjrdu\projects\action-study>
        
MariaDB実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker run --name dbcontainer -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=maria mariadb
    586b26c89f12109e1ebd6c166ca5ff675f74427ad88bd5dc444805874910401e
    
    C:\Users\tjrdu\projects\action-study>

- -name : container name
- -d : daemon execute
- -p : local port / container port
- -e : environment variable
- mariadb : image name

Container実行確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker ps -a
    CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                    NAMES
    586b26c89f12   mariadb   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3306->3306/tcp   dbcontainer

Containerへ接続
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker exec -it dbcontainer /bin/bash
    root@586b26c89f12:/#

- -i : interactive
- -t : tty(teletpyewriter)

初期化スクリプト作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    root@586b26c89f12:/# cd docker-entrypoint-initdb.d/
    root@586b26c89f12:/docker-entrypoint-initdb.d# echo "create database logger;" >> init.sql
    root@586b26c89f12:/docker-entrypoint-initdb.d# echo "use logger;" >> init.sql
    root@586b26c89f12:/docker-entrypoint-initdb.d# echo "create table message_log (registered TIMESTAMP, message TEXT);" >> init.sql
    root@586b26c89f12:/docker-entrypoint-initdb.d# ll
    total 12
    drwxr-xr-x 1 root root 4096 Jan 24 15:53 ./
    drwxr-xr-x 1 root root 4096 Jan 24 15:41 ../
    -rw-r--r-- 1 root root   99 Jan 24 15:54 init.sql
    root@586b26c89f12:/docker-entrypoint-initdb.d# cat init.sql 
    create database logger;
    use logger;
    create table message_log (registered TIMESTAMP, message TEXT);
    root@586b26c89f12:/docker-entrypoint-initdb.d#

「docker-entrypoint-initdb.d」配下に存在するshやsqlファイルはContainer生成時にじっこうされる。

修正をCommit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker ps -a
    CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                    NAMES
    586b26c89f12   mariadb   "docker-entrypoint.s…"   21 minutes ago   Up 20 minutes   0.0.0.0:3306->3306/tcp   dbcontainer

    C:\Users\tjrdu\projects\action-study>docker images                   
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    mariadb      latest    d462573d8688   2 weeks ago   410MB
    
    C:\Users\tjrdu\projects\action-study>docker commit 586b26c89f12 db4ci
    sha256:28ca298e2b60c9f2fc9429be253b7b3098b1c357ff80cfdbd04b6b62fdb28964
    
    C:\Users\tjrdu\projects\action-study>docker images                    
    REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
    db4ci        latest    28ca298e2b60   9 seconds ago   410MB
    mariadb      latest    d462573d8688   2 weeks ago     410MB
    
    C:\Users\tjrdu\projects\action-study>

Imageを再実行
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker stop 586b26c89f12
    586b26c89f12
    
    C:\Users\tjrdu\projects\action-study>docker rm 586b26c89f12
    586b26c89f12
    
    C:\Users\tjrdu\projects\action-study>docker ps -a             
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    
    C:\Users\tjrdu\projects\action-study>docker images                         
    REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
    db4ci        latest    28ca298e2b60   About an hour ago   410MB
    mariadb      latest    d462573d8688   2 weeks ago         410MB
    
    C:\Users\tjrdu\projects\action-study>docker run --name dbcontainer -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=maria db4ci
    2fe351cc0417e2f87c70d421d6b24cd71e386c595f3fa1118ee114413a23d141

    C:\Users\tjrdu\projects\action-study>docker ps -a    
    CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                    NAMES
    2fe351cc0417   db4ci     "docker-entrypoint.s…"   4 minutes ago   Up 3 minutes   0.0.0.0:3306->3306/tcp   dbcontainer

Containerのみ再起動すると初期化Scriptは実行されない。 実行中のContainerを終了・削除して、Imageから再起動すると初期化スクリプトが実行される。

DatabaseとTable確認
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    root@2fe351cc0417:/# mysql -u root -p
    Enter password:
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 4
    Server version: 10.6.5-MariaDB-1:10.6.5+maria~focal mariadb.org binary distribution
    
    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
    
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | logger             |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    5 rows in set (0.001 sec)
    
    MariaDB [(none)]> use logger;
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A
    
    Database changed
    MariaDB [logger]> show tables;
    +------------------+
    | Tables_in_logger |
    +------------------+
    | message_log      |
    +------------------+
    1 row in set (0.000 sec)
    
    MariaDB [logger]> quit
    Bye
    root@2fe351cc0417:/#

作成したImageをGithubへアップロード
---------------------------------------

GithubのAccess Token作成
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. 「GitHub Profile > Settings > Developer settings > Personal access tokens > Generate new token」へ移動
#. repo, write:packages, read:packages, delete:packagesをチェックして生成
#. 「Repository Settings > secrets > Actions secrets > New repository secrets」へ移動
#. 「DOCKER_TOKEN」を名前で作成したPersonal access tokenを登録

GithubへImageをPush
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: shell

    C:\Users\tjrdu\projects\action-study>docker login ghcr.io -u ${account}
    Password:
    Login Succeeded
    
    C:\Users\tjrdu\projects\action-study>docker images
    REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
    db4ci        latest    28ca298e2b60   7 days ago    410MB
    mariadb      latest    d462573d8688   3 weeks ago   410MB
    
    C:\Users\tjrdu\projects\action-study>docker tag 28ca298e2b60 ghcr.io/${account}/db4ci:v0.1

    C:\Users\tjrdu\projects\action-study>docker images
    REPOSITORY             TAG       IMAGE ID       CREATED       SIZE
    db4ci                  latest    28ca298e2b60   7 days ago    410MB
    ghcr.io/${account}/db4ci   v0.1      28ca298e2b60   7 days ago    410MB
    mariadb                latest    d462573d8688   3 weeks ago   410MB
    
    C:\Users\tjrdu\projects\action-study>docker push ghcr.io/${account}/db4ci:v0.1
    The push refers to repository [ghcr.io/${account}/db4ci]
    dc4228fb7e5e: Pushed
    75d472854d2e: Pushed
    d683d712eeb9: Pushed
    8dd0937cd2ed: Pushed
    237f27be7be8: Pushed
    b9bd86e7f504: Pushed
    758d9b108a97: Pushed
    8742bad8d199: Pushed
    35552a6449c8: Pushed
    8eba7440063f: Pushed
    0eba131dffd0: Pushed
    v0.1: digest: sha256:6d9a93eb30457e9ae60c7175bcb63330ea4b0380fe0ca60e18c1babd1776d15c size: 2619
    
    C:\Users\tjrdu\projects\action-study>

「Your profile > Packages」でPushしたImageが確認できる。

Workflows作成
---------------------------------------
.. code-block:: yaml

    name: action for CI

    on: [ pull_request ]
   
    jobs:
      Unit-test-with-Docker:
        runs-on: ubuntu-latest
   
        services:
          mariadb:
            image: ghcr.io/nomdle/db4ci:v0.1
            credentials:
              username: $GITHUB_REPOSITORY_OWNER
              password: ${{ secrets.DOCKER_TOKEN }}
            ports:
              - 3306:3306
   
        steps:
          - name: setup jdk
            uses: actions/setup-java@v2
            with:
              java-version: 15
              distribution: 'zulu'
              architecture: x64
   
          - name: checkout
            uses: actions/checkout@v2
   
          - name: grant execute permission for gradlew
            run: chmod +x gradlew
   
          - name: Start build and test
            run: ./gradlew clean build

- name : GithubのActionsの左バーに表示される名称
- on : 実行トリガー
- jobs : 実行グループ
- Unit-test-with-Docker : Jobのラベル、GithubのActions詳細の左バーに表示される名称
- runs-on : Jobが実行されるOS
- services : Job実行中に必要なサービスをHostするためのDockerコンテナ
    - Jobの開始終了で生成破棄されて、Jobの中のすべてのStepから通信可能
- mariadb : サービスのラベル
- image : Dockerに使うイメージ。GithubのPackagesにあげたイメージのアドレス
    - Docker Hubのイメージならイメージ名のみでもOK
- credentials : GithubのPackagesへアクセスするための認証情報
- ports : コンテナが利用するポート

残課題
=======================================
- テーブル作成された状態のDocker Imageを作成
