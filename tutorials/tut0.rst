.. _tut0:

Tutorial 0 - 让它跑起来
===========================

欢迎使用 PostgREST！该前言是一个 Quick start，帮助你快速创建第一个简单的 API。

PostgREST 是一个独立的 Web 服务器，为 PostgreSQL 数据库生成 RESTful API。 它提供基于底层数据库结构定制的 API。

.. image:: ../_static/tuts/tut0-request-flow.png

想要生成 API，我们只需要创建一个数据库。所有端点和权限都来自数据库对象，如表、视图、角色和存储过程。这些教程将涵盖许多常见的情况以及如何在数据库中对它们进行建模。

在本教程结束的时候，您将拥有一个能用的数据库，PostgREST 服务器和一个简单的单用户 todo list API。

Step 1. 放轻松老铁, 我们会帮你的
--------------------------------

在你开始这个教程时, Ctrl+T 一下在新标签中打开项目 `聊天室 <https://gitter.im/begriffs/postgrest>`_ . 有一群很 nice 的人在聊天室中活跃，如果你卡住了我们会帮你的。

Step 2. 安装 PostgreSQL
--------------------------

您将需要在本机或 Docker 实例中运行一个数据库的现代副本（modern copy）。我们需要 PostgreSQL 9.3 或更高版本，但建议至少 9.5 用于 row-level 安全性功能（在后面的教程中会用到）。

如果您已经熟悉 PostgreSQL 的使用并在本地有安装，可以直接使用现有的数据库。在本教程中，我们将介绍如何在 Docker 中使用数据库，否则数据库配置对于简单的教程来说太复杂了。

如果你没有安装 Docker 可以点 `这里 <https://www.docker.com/community-edition#download>`_. 安装好了之后，让我们来拉去并启动数据库的镜像:

.. code-block:: bash

  sudo docker run --name tutorial -p 5432:5432 \
                  -e POSTGRES_PASSWORD=mysecretpassword \
                  -d postgres

以上操作会以守护进程方式运行 Docker 实例并且暴露一个 5432 端来供你访问 PostgreSQL server。

Step 3. 安装 PostgREST
-------------------------

PostgREST 是作为一个单独的二进制文件发布的，支持 Linux/BSD/Windows 的主要发行版。访问 `最新版本 <https://github.com/begriffs/postgrest/releases/latest>`_ 以获取下载列表。如果您的平台不是预先构建的平台，请参阅 :ref:`build_source` 以获取有关如何自行构建的说明。也可以通知我们在下一个版本中添加您的平台。

用于下载的二进制文件是 :code:`.tar.xz` 压缩文件（除了 Windows 是 zip 文件）。 要提取二进制文件，进入终端并运行

.. code-block:: bash

  # download from https://github.com/begriffs/postgrest/releases/latest

  tar xfJ postgrest-<version>-<platform>.tar.xz

你会得到一个名为 :code:`postgrest` (Windows 上是 :code:`postgrest.exe`) 的文件. 到了这一步，可以尝试运行

.. code-block:: bash

  ./postgrest

如果一切正常，它将打印出有关配置的版本和信息。您可以继续从您下载的文件目录运行它，也可以将其复制到系统目录，如 Linux上 的 :code:`/usr/local/bin` ，以便您可以从任何目录运行它。

.. note::

  PostgREST 依赖 libpq （PostgreSQL 的 C 语言库）的安装。没有这个库的话你会获得一个形如 "error while loading shared libraries: libpq.so.5." 的报错，以下是解决方案:

  .. raw:: html

    <p>
    <details>
      <summary>Ubuntu or Debian</summary>
      <div class="highlight-bash"><div class="highlight">
        <pre>sudo apt-get install libpq-dev</pre>
      </div></div>
    </details>
    <details>
      <summary>Fedora, CentOS, or Red Hat</summary>
      <div class="highlight-bash"><div class="highlight">
        <pre>sudo yum install postgresql-libs</pre>
      </div></div>
    </details>
    <details>
      <summary>OS X</summary>
      <div class="highlight-bash"><div class="highlight">
        <pre>brew install postgresql</pre>
      </div></div>
    </details>
    <details>
      <summary>Windows</summary>
      <p>It isn't fun. Learn more <a href="https://stackoverflow.com/questions/38341725/how-to-install-libpq-dev-package-for-windows">here</a>.</p>
      <p>It might be easier to execute PostgREST in its own Docker image as well.</p>
    </details>
    </p>

Step 4. 为 API 创建数据库
-------------------------------

为了连上容器内的 SQL 控制台 (psql)，你需要运行如下命令:

.. code-block:: bash

  sudo docker exec -it tutorial psql -U postgres

你应该看到了 psql 的命令行提示:

::

  psql (9.6.3)
  Type "help" for help.

  postgres=#

我们要做的第一件事是为要暴露在 API 中的数据库对象创建一个 `命名的 schema <https://www.postgresql.org/docs/current/static/ddl-schemas.html>`_。我们可以使用任何我们喜欢的名称，那么就叫 "api" 怎么样。在你刚刚启动的命令行工具内执行该操作：

.. code-block:: postgres

  create schema api;

我们的 API 准备通过表来设置一个端点 :code:`/todos`。

.. code-block:: postgres

  create table api.todos (
    id serial primary key,
    done boolean not null default false,
    task text not null,
    due timestamptz
  );

  insert into api.todos (task) values
    ('finish tutorial 0'), ('pat self on back');

接下来，创建一个角色来用于进行匿名的 web 请求。当一个请求进来，PostgREST 会在数据库中切换到该角色进行查询。

.. code-block:: postgres

  create role web_anon nologin;
  grant web_anon to postgres;

  grant usage on schema api to web_anon;
  grant select on api.todos to web_anon;

:code:`web_anon` 角色拥有访问 :code:`api` schema 的权限，可以读取 :code:`todos` 表中的数据（rows）。

现在可以退出 psql， 是时候开始使用 API 了！

.. code-block:: psql

  \q

Step 5. 运行 PostgREST
----------------------

PostgREST 使用一个配置文件来确定如何连接数据库。创建一个文件 :code:`tutorial.conf` 并加上如下内容:

.. code-block:: ini

  db-uri = "postgres://postgres:mysecretpassword@localhost/postgres"
  db-schema = "api"
  db-anon-role = "web_anon"

详细配置内容参见 :ref:`options <configuration>`。现在可以运行服务器:

.. code-block:: bash

  ./postgrest tutorial.conf

你应该看到

.. code-block:: text

  Listening on port 3000
  Attempting to connect to the database...
  Connection successful

现在可以进行 web 请求了。市面上有很多可以用的图形化 API 请求工具，不过在本教程内我们使用 :code:`curl`。打开一个新的 terminal (保持 PostgREST 依旧运行)。尝试对 todos 做一个 HTTP 请求。

.. code-block:: bash

  curl http://localhost:3000/todos

API 返回:

.. code-block:: json

  [
    {
      "id": 1,
      "done": false,
      "task": "finish tutorial 0",
      "due": null
    },
    {
      "id": 2,
      "done": false,
      "task": "pat self on back",
      "due": null
    }
  ]

通过当前的角色权限，匿名请求有 :code:`todos` 表的只读权限。如果我们试图添加一个新的 todo 会被拒绝。

.. code-block:: bash

  curl http://localhost:3000/todos -X POST \
       -H "Content-Type: application/json" \
       -d '{"task": "do bad thing"}'

响应是 401 Unauthorized:

.. code-block:: json

  {
    "hint": null,
    "details": null,
    "code": "42501",
    "message": "permission denied for relation todos"
  }

There we have it, a basic API on top of the database! 在下一篇教程中，我们将会看到如何拓展这个例子，使用更复杂的用户访问控制，以及更多的表和查询。
