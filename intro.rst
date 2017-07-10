动机
##########

PostgREST 是一个独立的 Web 服务器，可为您的 PostgreSQL 数据库直接生成 RESTful API。 数据库中的结构约束和权限决定了 API 的端点（endpoints）和操作。

PostgREST 是手动编写 CRUD 的替代方案。通常的 API 服务器普遍遇到遇到一个问题。那就是编写业务逻辑往往会重复劳动，并且对于数据结构存在疏漏。对象关系（Object-relational）映射是一种存疑（leaky）的抽象，可能导致产生低效的代码。PostgREST 哲学建立了一个单一的声明性真理来源————数据本身。

声明性编程
-----------------------

比起对着查询结果重复劳动，让查询计划理清细节使 PostgreSQL 为您添加数据是一件更容易的事。为数据库对象分配权限比在控制器（controllers）中添加保护更容易（这对于数据依赖关系中的级联权限尤其如此）。并且设置约束比编写重复代码进行健全检查更容易。

Leak-proof 抽象
----------------------

没有 ORM 参与。创建 SQL 性能提示的视图。数据库管理员（DA）现在可以自行创建 API，无需开发参与。

拥抱关系模型
------------------------------

1970年，E. F. Codd 在他的文章 “大型共享数据库数据关系模型” 中批评了当时主导的数据库分层模型。于都文章会发现层次数据库和嵌套 http 路由之间存在惊人相似性。而在 PostgREST 中，我们尝试使用灵活的过滤和嵌入，而不是嵌套路由。

一个重点
--------------

PostgREST 有一个重要的特点。它适用于如 Nginx 的工具。这可以强行将以数据为中心的 CRUD 操作与其他问题进行干净分离。

改进共享
-------------------

与任何开源项目一样，我们都可以从工具中的功能和修复中获益。这比在各自私有库中耦合的改进更有益处。

生态系统
#########

PostgREST 具有不断增长的生态系统，示例、库、实验和用户。这是一个选择。

.. _clientside_libraries:

客户端库
---------------------

* `tomberek/aor-postgrest-client <https://github.com/tomberek/aor-postgrest-client>`_ - JS, admin-on-rest
* `hugomrdias/postgrest-url <https://github.com/hugomrdias/postgrest-url>`_ - JS, just for generating query URLs
* `john-kelly/elm-postgrest <https://github.com/john-kelly/elm-postgrest>`_ - Elm
* `mithril.postgrest <https://github.com/catarse/mithril.postgrest>`_ - JS, Mithril
* `lewisjared/postgrest-request <https://github.com/lewisjared/postgrest-request>`_ - JS, SuperAgent
* `JarvusInnovations/jarvus-postgrest-apikit <https://github.com/JarvusInnovations/jarvus-postgrest-apikit>`_ - JS, Sencha framework
* `davidthewatson/postgrest_python_requests_client <https://github.com/davidthewatson/postgrest_python_requests_client>`_ - Python
* `calebmer/postgrest-client <https://github.com/calebmer/postgrest-client>`_ - JS
* `clesiemo3/postgrestR <https://github.com/clesiemo3/postgrestR>`_ - R
* `PierreRochard/postgrest-angular <https://github.com/PierreRochard/postgrest-angular>`_ - TypeScript, generate UI from API description
* `thejettdurham/postgrest-sharp-client <https://github.com/thejettdurham/postgrest-sharp-client>`_ (needs maintainer) - C#, RestSharp

额外通知
---------------------

在于外部交互上（LISTEN/NOTIFY）PostgreSQL 有拓展到外部队列进行进一步处理的网桥。这允许存储过程在数据库外发起动作，例如发送电子邮件。

* `frafra/postgresql2websocket <https://github.com/frafra/postgresql2websocket>`_ - Websockets
* `matthewmueller/pg-bridge <https://github.com/matthewmueller/pg-bridge>`_ - Amazon SNS
* `aweber/pgsql-listen-exchange <https://github.com/aweber/pgsql-listen-exchange>`_ - RabbitMQ
* `SpiderOak/skeeter <https://github.com/SpiderOak/skeeter>`_ - ZeroMQ
* `FGRibreau/postgresql-to-amqp <https://github.com/FGRibreau/postgresql-to-amqp>`_ - AMQP

示例应用
------------

* `subzerocloud/postgrest-starter-kit <https://github.com/subzerocloud/postgrest-starter-kit>`_ - Boilerplate for new project
* `NikolayS/postgrest-google-translate <https://github.com/NikolayS/postgrest-google-translate>`_ - Calling to external translation service
* `CodeforAustralia/heritage-near-me <https://github.com/CodeforAustralia/heritage-near-me>`_ - Elm and PostgREST with PostGIS
* `timwis/handsontable-postgrest <https://github.com/timwis/handsontable-postgrest>`_ - An excel-like database table editor
* `Recmo/PostgrestSkeleton <https://github.com/Recmo/PostgrestSkeleton>`_ - Docker Compose, PostgREST, Nginx and Auth0
* `benoror/ember-postgrest-dynamic-ui <https://github.com/benoror/ember-postgrest-dynamic-ui>`_ - generating Ember forms to edit data
* `ruslantalpa/blogdemo <https://github.com/ruslantalpa/blogdemo>`_ - blog api demo in a vagrant image
* `timwis/ext-postgrest-crud <https://github.com/timwis/ext-postgrest-crud>`_ - browser-based spreadsheet
* `srid/chronicle <https://github.com/srid/chronicle>`_ - tracking a tree of personal memories
* `diogob/elm-workshop <https://github.com/diogob/elm-workshop>`_ - building a simple database query UI
* `marmelab/ng-admin-postgrest <https://github.com/marmelab/ng-admin-postgrest>`_ - automatic database admin panel
* `myfreeweb/moneylog <https://github.com/myfreeweb/moneylog>`_ - accounting web app in Polymer + PostgREST
* `tyrchen/goodfilm <https://github.com/tyrchen/goodfilm>`_ - example film api
* `begriffs/postgrest-example <https://github.com/begriffs/postgrest-example>`_ - sqitch versioning for API
* `SMRxT/postgrest-demo <https://github.com/SMRxT/postgrest-demo>`_ - multi-tenant logging system
* `PierreRochard/postgrest-boilerplate <https://github.com/PierreRochard/postgrest-boilerplate>`_ - example auth backend

Production
-------------

* `Catarse <https://www.catarse.me/>`_
* `iAdvize <http://iadvize.com/>`_
* `Redsmin <https://www.redsmin.com/>`_
* `Image-charts <https://image-charts.com/>`_
* `Drip Depot <https://www.dripdepot.com/>`_
* `OpenBooking <http://openbooking.ch>`_
* `Convene <https://info.convene.thomsonreuters.com/en.html>`_ by Thomson-Reuters
* `eGull <http://www.egull.co>`_

拓展
----------

* `ppKrauss/PostgREST-writeAPI <https://github.com/ppKrauss/PostgREST-writeAPI>`_ - generate Nginx rewrite rules to fit an OpenAPI spec
* `diogob/postgrest-ws <https://github.com/diogob/postgrest-ws>`_ - expose web sockets for PostgreSQL's LISTEN/NOTIFY
* `pg-safeupdate <https://bitbucket.org/eradman/pg-safeupdate/>`_ - Prevent full-table updates or deletes
* `srid/spas <https://github.com/srid/spas>`_ - allow file uploads and basic auth
* `svmnotn/postgrest-auth <https://github.com/svmnotn/postgrest-auth>`_ - OAuth2-inspired external auth server
* `nblumoe/postgrest-oauth <https://github.com/nblumoe/postgrest-oauth>`_ - OAuth2 WAI middleware

广告
---------------

* `subZero <https://subzero.cloud/>`_ - Automated GraphQL & REST API with built-in caching (powered in part by PostgREST)

赞誉
############

  "开发起来太快了, 感觉就像在作弊!"

  -- François-G. Ribreau

  "我不得不说, 与 Node.js/Waterline ORM 构建的 API 对比
   CPU/Memory usage 简直是难以置信. 当我们在 6 个示例 （dynos）
   持续求情 1GB 数据是它甚至只有 60/70 MB 大小."

  -- Louis Brauer

  "我非常喜欢这样一个事实，偶然使用 SQL DDL（和 V8 javascript）开发微服务。
  我们在 6 个月内完全重写了一个 Spring + MySQL 遗留应用程序。 
  速度快 10 倍，代码很简洁。而之前的人用了 4 个人花了 3 年时间。"

  -- Simone Scarduzio

获得支持
################

该项目有一个友好且不断成长的社区。加入我们的 `聊天室 <https://gitter.im/begriffs/postgrest>`_ 来讨论和求助。同时你也可以在 Github 的 `issues <https://github.com/begriffs/postgrest/issues>`_ 上搜索 bugs/features。
