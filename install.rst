可执行文件
==============

[ `下载页 <https://github.com/begriffs/postgrest/releases/latest>`_ ]

下载页面具有 Mac OS X、Windows 和几个 Linux 发行版的预编译文件。解压之后可以运行可执行文件加 :code:`--help` 标志来查看使用说明:

.. code-block:: bash

  # 解压 tar 包 (available at https://github.com/begriffs/postgrest/releases/latest)

  $ tar Jxf postgrest-[version]-[platform].tar.xz

  # 尝试运行
  $ ./postgrest --help

  # You should see a usage help message

Homebrew
========

在 Mac 上你可以使用 Homebrew 来安装 PostgREST

.. code-block:: bash

  # 确保 brew 是最新的
  brew update

  # 检查 brew 的 setup 有没问题
  brew doctor

  # 安装 postgrest
  brew install postgrest

该命令会自动将 PostgreSQL 当做依赖安装. 该过程往往需要长达15分钟才能安装软件包及其依赖。

安装完成后，该工具会被添加到 $PATH 中，你可以在任意位置使用：

.. code-block:: bash

  postgrest --help

PostgreSQL 依赖
=====================

要使用 PostgREST 您将需要安装数据库（PostgreSQL 9.3 或更高版本）。 您可以使用像 Amazon `RDS <https://aws.amazon.com/rds/>`_ 这样的东西，但是在本地安装本身比较便宜，更便于开发。

* `OS X 说明 <http://exponential.io/blog/2015/02/21/install-postgresql-on-mac-os-x-via-brew/>`_
* `Ubuntu 14.04 说明 <https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04>`_
* `Windows 安装包 <http://www.enterprisedb.com/products-services-training/pgdownload#windows>`_

.. _build_source:

源代码编译
=================

.. note::

  我们不建议在 **Alpine Linux** 上构建和使用 PostgREST，因为在该平台有过 GHC 内存泄漏的报告。

当您的系统没有预构建的可执行文件时，可以从源代码构建项目。如果您想帮助开发，您还需要这种操作操作：`安装 Stack <https://github.com/commercialhaskell/stack>`_ 。它将在您的系统上安装任何必要的 Haskell 依赖。

* `安装 Stack <http://docs.haskellstack.org/en/stable/README.html#how-to-install>`_
* 安装依赖库

  =====================  =======================================
  Operating System       Dependencies
  =====================  =======================================
  Ubuntu/Debian          libpq-dev, libgmp-dev
  CentOS/Fedora/Red Hat  postgresql-devel, zlib-devel, gmp-devel
  BSD                    postgresql95-server
  OS X                   postgresql, gmp
  =====================  =======================================

* 构建并安装

  .. code-block:: bash

    git clone https://github.com/begriffs/postgrest.git
    cd postgrest

    # adjust local-bin-path to taste
    stack build --install-ghc --copy-bins --local-bin-path /usr/local/bin

.. note::

   如果你构建失败，而且你的系统只有不到 1GB 内存，尝试添加一个 swap 文件。

* 检查安装是否成功: :code:`postgrest --help`.

PostgREST 测试套件
--------------------

创建测试库
~~~~~~~~~~~~~~~~~~~~~~~~~~

To properly run postgrest tests one needs to create a database. To do so, use the test creation script :code:`create_test_database` in the :code:`test/` folder.

The script expects the following parameters:

.. code:: bash

  test/create_test_db connection_uri database_name [test_db_user] [test_db_user_password]

Use the `connection URI <https://www.postgresql.org/docs/current/static/libpq-connect.html#AEN45347>`_ to specify the user, password, host, and port. Do not provide the database in the connection URI. The Postgres role you are using to connect must be capable of creating new databases.

The :code:`database_name` is the name of the database that :code:`stack test` will connect to. If the database of the same name already exists on the server, the script will first drop it and then re-create it.

Optionally, specify the database user :code:`stack test` will use. The user will be given necessary permissions to reset the database after every test run.

If the user is not specified, the script will generate the role name :code:`postgrest_test_` suffixed by the chosen database name, and will generate a random password for it.

Optionally, if specifying an existing user to be used for the test connection, one can specify the password the user has.

The script will return the db uri to use in the tests--this uri corresponds to the :code:`db-uri` parameter in the configuration file that one would use in production.

Generating the user and the password allows one to create the database and run the tests against any postgres server without any modifications to the server. (Such as allowing accounts without a passoword or setting up trust authentication, or requiring the server to be on the same localhost the tests are run from).

运行测试
~~~~~~~~~~~~~~~~~

To run the tests, one must supply the database uri in the environment variable :code:`POSTGREST_TEST_CONNECTION`.

Typically, one would create the database and run the test in the same command line, using the `postgres` superuser:

.. code:: bash

  POSTGREST_TEST_CONNECTION=$(test/create_test_db "postgres://postgres:pwd@database-host" test_db) stack test

For repeated runs on the same database, one should export the connection variable:

.. code:: bash

  export POSTGREST_TEST_CONNECTION=$(test/create_test_db "postgres://postgres:pwd@database-host" test_db)
  stack test
  stack test
  ...

If the environment variable is empty or not specified, then the test runner will default to connection uri

.. code:: bash

  postgres://postgrest_test@localhost/postgrest_test

This connection assumes the test server on the :code:`localhost:code:` with the user `postgrest_test` without the password and the database of the same name.

销毁数据库
~~~~~~~~~~~~~~~~~~~~~~~

The test database will remain after the test, together with four new roles created on the postgres server. To permanently erase the created database and the roles, run the script :code:`test/delete_test_database`, using the same superuser role used for creating the database:

.. code:: bash

  test/destroy_test_db connection_uri database_name

使用 Docker 测试
~~~~~~~~~~~~~~~~~~~

The ability to connect to non-local PostgreSQL simplifies the test setup. One elegant way of testing is to use a disposable PostgreSQL in docker.

For example, if local development is on a mac with Docker for Mac installed:

.. code:: bash

  $ docker run --name db-scripting-test -e POSTGRES_PASSWORD=pwd -p 5434:5432 -d postgres
  $ POSTGREST_TEST_CONNECTION=$(test/create_test_db "postgres://postgres:pwd@localhost:5434" test_db) stack test

Additionally, if one creates a docker container to run stack test (this is necessary on MacOS Sierra with GHC below 8.0.1, where :code:`stack test` fails), one can run PostgreSQL in a separate linked container, or use the locally installed Postgres.app.

Build the test container with :code:`test/Dockerfile.test`:

.. code:: bash

  $ docker build -t pgst-test - < test/Dockerfile.test
  $ mkdir .stack-work-docker ~/.stack-linux

The first run of the test container will take a long time while the dependencies get cached. Creating the :code:`~/.stack-linux` folder and mapping it as a volume into the container ensures that we can run the container in disposable mode and not worry about subsequent runs being slow. :code:`.stack-work-docker` is also mapped into the container and must be specified when using stack from Linux, not to interfere with the :code:`.stack-work` for local development. (On Sierra, :code:`stack build` works, while :code:`stack test` fails with GHC 8.0.1).

Linked containers:

.. code:: bash

  $ docker run --name pg -e POSTGRES_PASSWORD=pwd  -d postgres
  $ docker run --rm -it -v `pwd`:`pwd` -v ~/.stack-linux:/root/.stack --link pg:pg -w="`pwd`" -v `pwd`/.stack-work-docker:`pwd`/.stack-work pgst-test bash -c "POSTGREST_TEST_CONNECTION=$(test/create_test_db "postgres://postgres:pwd@pg" test_db) stack test"

Stack test in Docker for Mac, Postgres.app on mac:

.. code:: bash

  $ host_ip=$(ifconfig en0 | grep 'inet ' | cut -f 2 -d' ')
  $ export POSTGREST_TEST_CONNECTION=$(test/create_test_db "postgres://postgres@$HOST" test_db)
  $ docker run --rm -it -v `pwd`:`pwd` -v ~/.stack-linux:/root/.stack -v `pwd`/.stack-work-docker:`pwd`/.stack-work -e "HOST=$host_ip" -e "POSTGREST_TEST_CONNECTION=$POSTGREST_TEST_CONNECTION" -w="`pwd`" pgst-test bash -c "stack test"
  $ test/destroy_test_db "postgres://postgres@localhost" test_db
