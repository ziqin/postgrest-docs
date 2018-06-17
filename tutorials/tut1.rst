.. _tut1:

Tutorial 1 - 金钥匙
===========================

在 :ref:`tut0` 中我们创建了一个获取 todos 数据只读的 API。使用单个端点列出 todos。 我们有很多办法可以使这个 API 更有趣，但一个好的开始是允许一些用户更改数据，而不仅仅是查询。

Step 1. 添加一个受信的用户
--------------------------

上一节中，进行匿名 Web 请求的事后在数据库中创建了一个 :code:`web_anon` 角色。让我们再创建一个叫做 :code:`todo_user` 的角色用于使用 API 进行身份验证的用户，这个角色将有权对 todo list 做任何事情。

.. code-block:: postgres

  -- run this in psql using the database created
  -- in the previous tutorial

  create role todo_user nologin;
  grant todo_user to postgres;

  grant usage on schema api to todo_user;
  grant all on api.todos to todo_user;
  grant usage, select on sequence api.todos_id_seq to todo_user;

Step 2. 生成一个密码
---------------------

客户端通过 API 使用 JSON Web Token 进行身份验证。JTW 是使用仅有我们和服务器知道的密码进行加密签名的 JSON 对象。 由于客户端不知道密码，所以不能篡改 token 的内容。 PostgREST 会检测伪造的 token 并拒绝它们。

我们来创建一个密码并提供给 PostgREST。最好想想一个复杂的长一点的，或使用一个工具来生成它。

.. note::

  `OpenSSL toolkit <https://www.openssl.org/>`_ 提个一个简单的方式来生成安全的密码。如果你有安装，运行

  .. code-block:: bash

    openssl rand -base64 32

打开 :code:`tutorial.conf` (在上一节中创建的) 并将密码添加在新的一行:

.. code-block:: ini

  # add this line to tutorial.conf

  jwt-secret = "<the password you created>"

如果 PostgREST server 仍旧在运行中，那么需要重启它以便加载最新的配置文件。

Step 3. 生成 token
--------------------

通常你自己的代码在数据库或其他服务器中将创建并签署身份验证 token，但是在本教程中，我们将“自己动手”。跳转到 `jwt.io <https://jwt.io/#debugger-io>`_，并填写如下字段：

.. figure:: ../_static/tuts/tut1-jwt-io.png
   :alt: jwt.io interface

   如何在 https://jwt.io 创建 Token

请记住您填写的密码，而不是图片里的 :code:`secret`。填写密码和 payload 之后，左侧的编码数据会刷新，该数据即 token 复制它。

.. note::

  虽然令牌可能看起来很模糊，但很容易逆向得到 payload。token 仅仅是被签名，没有加密，所以如果你有不想让客户端看到的信息，请不要将它们放在里面。

Step 4. 进行请求
----------------------

回到 terminal，我们来用 :code:`curl` 添加一个 todo。该请求将包括一个包含身份验证 token 的 HTTP 头。

.. code-block:: bash

  export TOKEN="<paste token here>"

  curl http://localhost:3000/todos -X POST \
       -H "Authorization: Bearer $TOKEN"   \
       -H "Content-Type: application/json" \
       -d '{"task": "learn how to auth"}'

现在我们已经完成了我们的 todo list 中的所有三个项目，所以我们通过 :code:`PATCH` 请求将他们全设置为 :code:`done`。

.. code-block:: bash

  curl http://localhost:3000/todos -X PATCH \
       -H "Authorization: Bearer $TOKEN"    \
       -H "Content-Type: application/json"  \
       -d '{"done": true}'

请求一下 todo 看看这三项，全部都已完成了.

.. code-block:: bash

  curl http://localhost:3000/todos

.. code-block:: json

  [
    {
      "id": 1,
      "done": true,
      "task": "finish tutorial 0",
      "due": null
    },
    {
      "id": 2,
      "done": true,
      "task": "pat self on back",
      "due": null
    },
    {
      "id": 3,
      "done": true,
      "task": "learn how to auth",
      "due": null
    }
  ]

Step 4. 添加过期时间
----------------------

目前，我们的认证 token 对于所有请求都是一致有效的。服务器只要继续使用相同的 JWT 密码，就会通过验证。

更好的策略是让 token 使用 :code:`exp` 声明一个过期时间戳。这是 PostgREST 特别对待的两个 JWT 声明之一。

+--------------+----------------------------------------------------------------+
| Claim        | Interpretation                                                 |
+==============+================================================================+
| :code:`role` | The database role under which to execute SQL for API request   |
+--------------+----------------------------------------------------------------+
| :code:`exp`  | Expiration timestamp for token, expressed in "Unix epoch time" |
+--------------+----------------------------------------------------------------+

.. note::

  Unix 时间戳 (Unix epoch time) 被定义为自 1970 年 1 月 1 日 00:00:00 协调世界时（UTC）以来到现在的总秒数，不考虑闰秒。

为了在行动中观察过期，我们将添加一个在 5min 之后过期的 :code:`exp` 声明。首先找到从当前时间算起到 5min 之后的时间戳。 在 psql 中运行：

.. code-block:: postgres

  select extract(epoch from now() + '5 minutes'::interval) :: integer;

回到 jwt.io 并修改 payload

.. code-block:: json

  {
    "role": "todo_user",
    "exp": "<computed epoch value>"
  }

拷贝新的 token，然后将其保存为一个新的环境变量。

.. code-block:: bash

  export NEW_TOKEN="<paste new token>"

尝试在过期时间的前后使用 curl 进行该请求:

.. code-block:: bash

  curl http://localhost:3000/todos \
       -H "Authorization: Bearer $NEW_TOKEN"

过期以后, 该 API 会返回一个 HTTP 401 Unauthorized:

.. code-block:: json

  {"message":"JWT expired"}

附加题: 立即撤销
---------------------------------

即使有 token 过期时间，有时你也可能想立即撤回某个 token 的权限，比方说你发现某个心怀怨气的员工不怀好意，而他的 token 仍然有效的时候。 

为了能撤销一个特定的 token，我们需要用某种方式将它和其他的 token 区分开来。让我们填加一个自定义的 :code:`email` 声明，它与发布 token 的客户的 email 相匹配。

继续用这个 payload 创建一个新的 token

.. code-block:: json

  {
    "role": "todo_user",
    "email": "disgruntled@mycompany.com"
  }

把它存到一个环境变量里：

.. code-block:: bash

  export WAYWARD_TOKEN="<paste new token>"

PostgREST 允许我们指定在尝试认证时运行的存储程序。这个存储程序（函数）可以为所欲为，包括通过引发异常来终止请求。

首先创建一个新的 schema，并添加下列函数：

.. code-block:: plpgsql

  create schema auth;
  grant usage on schema auth to web_anon, todo_user;

  create or replace function auth.check_token() returns void
    language plpgsql
    as $$
  begin
    if current_setting('request.jwt.claim.email', true) =
       'disgruntled@mycompany.com' then
      raise insufficient_privilege
        using hint = 'Nope, we are on to you';
    end if;
  end
  $$;

接着更新 :code:`tutorial.conf`，指明该函数：

.. code-block:: ini

  # add this line to tutorial.conf

  pre-request = "auth.check_token"

重启 PostgREST 使修改生效。接着，试试用我们原来的 token 创建一个请求，然后试试撤销掉的那个。

.. code-block:: bash

  # this request still works

  curl http://localhost:3000/todos \
       -H "Authorization: Bearer $TOKEN"

  # this one is rejected

  curl http://localhost:3000/todos \
       -H "Authorization: Bearer $WAYWARD_TOKEN"

服务器会响应 403 Forbidden：

.. code-block:: json

  {
    "hint": "Nope, we are on to you",
    "details": null,
    "code": "42501",
    "message": "insufficient_privilege"
  }
