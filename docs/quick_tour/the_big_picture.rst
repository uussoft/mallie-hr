Symfony基本知识
===============

从零学习Symfony，只要10分钟！ 本文将带你贯穿框架中的一些重要概念，并通过简单的小项目来解释如何快速上手。

如果您以前用过Web框架，你对Symfony会有一种宾至如归的感觉。如果没有，欢迎使用Web开发的全新方式。
Symfony拥有最佳实践，并且保持向后兼容性（的确！升级始终是安全和方便！），并长期提供技术支持。


.. _installing-symfony2:

下载安装Symfony框架
-------------------

继续阅读本章之前，请确保你的终端已经安装了 `Composer`_ ，并且安装了PHP7.1.3或者更高版本

在终端中运行：

.. code-block:: terminal

    $ composer create-project symfony/skeleton quick_tour


这个命令可以创建一个新的 ``quick_tour/`` 目录，其中包含一个小型但功能强大的新 Symfony 应用程序：

.. code-block:: text

    quick_tour/
    ├─ .env
    ├─ bin/console
    ├─ composer.json
    ├─ composer.lock
    ├─ config/
    ├─ public/index.php
    ├─ src/
    ├─ symfony.lock
    ├─ var/
    └─ vendor/

我们可以在浏览器中加载项目吗？可以的! 您可以设置 :doc:`Nginx or Apache </setup/web_server_configuration>` 并将其文档根 ``public/`` 目
录配置为该目录。但是，对于开发而言，最好安装 :doc:`Symfony local web server </setup/symfony_server>` 并按以下方式运行它：

.. code-block:: terminal

    $ symfony server:start

您可以尝试在浏览器中访问您的新应用： ``http://localhost:8000`` 

.. image:: /_images/quick_tour/no_routes_page.png
   :align: center
   :class: with-browser


基础知识：路由、控制器、响应
-----------------------------------------

我们的项目只有大约15个文件，但已经准备好成为优美的API，强大的Web应用程序或者微服务。
Symfony开始很小，它会随着你的应用程序而改变。

接下来，让我们通过构建第一个页面来深入了解基础知识。

首先从 ``config/routes.yaml`` 开始：在这里可以定义新页面的URL。取消注释文件中已经存在的示例：

.. code-block:: yaml

    # config/routes.yaml
    index:
        path: /
        controller: 'App\Controller\DefaultController::index'


这个称为路由（Route）：它定义了当前页面 (``/``) 的URL和控制器（Controller）：每当有人访问此URL都会调用该 *函数*，
该功能尚能尚不存在，让我们按照下列步骤来创建它：

在  ``src/Controller`` 中，创建一个新的  ``DefaultController``  类和一个 ``index`` 方法：
::

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class DefaultController
    {
        public function index()
        {
            return new Response('Hello!');
        }
    }

就是这样，尝试转到主页 ``http://localhost:8000/`` 。Symfony将会看到URL与我们的路由匹配，然后执行  ``index()`` 方法

控制器只是一个普通函数，具有 *一个* 规则的：它必须返回 Symfony ``Response`` 对象。但是，该响应（Response）可以包含任何内容：简单文本、JSON 或完整的 HTML 页面。

但是路由系统是 *更* 强大的。因此，让我们使路由更有趣：

.. code-block:: diff

    # config/routes.yaml
    index:
    -     path: /
    +     path: /hello/{name}
          controller: 'App\Controller\DefaultController::index'

这页URL已经改变：它是 *现在* 这个样子 ``/hello/*``：这个 ``{name}`` 作用就像是匹配任何通配符：

.. code-block:: diff

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class DefaultController
    {
    -     public function index()
    +     public function index($name)
          {
    -         return new Response('Hello!');
    +         return new Response("Hello $name!");
          }
    }

尝试转到页面 ``http://localhost:8000/hello/Symfony``。您应该看到：Hello Symfony！URL中  ``{name}`` 的值可以作为一个控制器参数 ``$name``。

但这一切可以更简单！因此，让我们安装注释支持：

.. code-block:: terminal

    $ composer require annotations

现在，可以通过 ``#`` 字符来注释掉YAML中的路由：

.. code-block:: yaml

    # config/routes.yaml
    # index:
    #     path: /hello/{name}
    #     controller: 'App\Controller\DefaultController::index'

而是在Controller方法上方添加路由：

.. code-block:: diff

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;
    + use Symfony\Component\Routing\Annotation\Route;

    class DefaultController
    {
    +    /**
    +     * @Route("/hello/{name}")
    +     */
         public function index($name)
         {
             // ...
         }
    }

就像以前一样！但是，通过使用注释路由，可以让路由和控制器关系更紧密。如果需要其他页面？可以在  ``DefaultController`` 中添加其他路由和方法：
::
    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\Routing\Annotation\Route;

    class DefaultController
    {
        // ...

        /**
         * @Route("/simplicity")
         */
        public function simple()
        {
            return new Response('Simple! Easy! Great!');
        }
    }

路由可以做更多的事情，但是我们会把它留到下一次！现在，我们的应用需要更多的功能！像模板引擎、日志记录、调试工具等等。


继续阅读 :doc:`/quick_tour/flex_recipes`.

.. _`Composer`: https://getcomposer.org/
