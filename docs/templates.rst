.. index::
   single: Templating

创建和使用模板
============================

模板是从应用程序内部组织和呈现HTML的最佳方式，无论您是需要从 :doc:`控制器 </controller>` 呈现HTML，还是生成 :doc:`电子邮件内容 </mailer>`。Symfony中的模板是用 ``Twig`` 创建的：一个灵活、快速、安全的模板引擎。

.. _twig-language:

Twig模板语言
------------------------

 `Twig`_ 模板语言允许您编写简洁、易读的模板，这些模板对网页设计师更友好，并且在某些方面比PHP模板功能更强大。请看下面的 ``Twig`` 模板示例。即使这是你第一次看到 ``Twig`` ，你可能也能理解其中的大部分内容：

.. code-block:: html+twig

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            {% if user.isLoggedIn %}
                Hello {{ user.name }}!
            {% endif %}

            {# ... #}
        </body>
    </html>

Twig语法基于以下三种结构：

* ``{{ ... }}``, 用于显示变量的内容或对表达式求值的结果；
* ``{% ... %}``, 用于运行某些逻辑，例如条件或循环；
* ``{# ... #}``, 用于向模板添加注释（与HTML注释不同，这些注释不包含在呈现的页面中）。

您无法在 ``Twig`` 模板中运行PHP代码，但是 ``Twig`` 提供了实用程序用来在模板中运行某些逻辑。例如， **过滤器** 会在渲染模板内容之前对其进行修改，如通过 ``upper`` 过滤器将内容修改为大写：

.. code-block:: twig

    {{ title|upper }}

 ``Twig`` 附带了很多的 `tags`_， `filters`_ 和 `functions`_ ，默认情况下可用。在Symfony应用程序中，您还可以使用 :doc:`Symfony定义的Twig过滤器和函数 </reference/twig_reference>` ，还可以 :doc:`创建自己的Twig过滤器和函数 </templating/twig_extension>`。

 ``Twig`` 在 ``prod`` :ref:`环境 <configuration-environments>` 中速度很快 （因为模板已编译到PHP中并自动缓存），但在 ``dev`` 环境中使用很方便（因为更改模板后它们会自动重新编译）。

Twig配置
~~~~~~~~~~~~~~~~~~

 ``Twig`` 有几个配置选项来定义用于显示数字和日期的格式、模板缓存等。请阅读 :doc:`Twig配置参考 </reference/configuration/twig>` 以了解他们。

创建模板
------------------

在详细解释如何创建和渲染模板之前，请查看下面的示例快速了解整个过程。首先，需要在 ``templates/`` 目录中创建一个新文件来存储模板内容：

.. code-block:: html+twig

    {# templates/user/notifications.html.twig #}
    <h1>Hello {{ user_first_name }}!</h1>
    <p>You have {{ notifications|length }} new notifications.</p>

然后，创建一个 :doc:`控制器 </controller>` 来呈现此模板并将所需的变量传递给它：
::

    // src/Controller/UserController.php
    namespace App\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

    class UserController extends AbstractController
    {
        // ...

        public function notifications()
        {
            // get the user information and notifications somehow
            $userFirstName = '...';
            $userNotifications = ['...', '...'];

            // the template path is the relative file path from `templates/`
            return $this->render('user/notifications.html.twig', [
                // this array defines the variables passed to the template,
                // where the key is the variable name and the value is the variable value
                // (Twig recommends using snake_case variable names: 'foo_bar' instead of 'fooBar')
                'user_first_name' => $userFirstName,
                'notifications' => $userNotifications,
            ]);
        }
    }

模板命名
~~~~~~~~~~~~~~~

Symfony建议模板名称如下：

* 对文件名和目录使用 `snake case`_ 方式命名（例如 ``blog_posts.twig`` ， ``admin/default_theme/blog/index.twig`` 等等）；
* 为文件定义两个扩展名（例如 ``index.html.twig`` 或 ``blog_posts.xml.twig`` ），作为模板将生成的最终格式的第一个扩展名（ ``html`` ， ``xml`` ），等等。

尽管模板通常生成HTML内容，但是它们可以生成任何基于文本的格式。这就是为什么两个扩展约定简化了为多种格式创建和呈现模板的方式。

模板位置
~~~~~~~~~~~~~~~~~

Templates are stored by default in the ``templates/`` directory. When a service
or controller renders the ``product/index.html.twig`` template, they are actually
referring to the ``<your-project>/templates/product/index.html.twig`` file.
模板默认情况下存储在根目录下的 ``templates/`` 目录中。当服务或控制器需要呈现 ``product/index.html.twig`` 模板时，它们实际上是在引用 ``<your-project>/templates/product/index.html.twig`` 文件。

默认模板目录可通过 :ref:`twig.default_path <config-twig-default-path>` 选项进行配置，您可以添加更多模板目录，如本文 :ref:`后面所述 <templates-namespaces>` 。

模板变量
~~~~~~~~~~~~~~~~~~

模板的一个常见需求是打印存储在从控制器或服务传递给模板的值。变量通常存储对象和数组中，而不是字符串，数字和布尔值。这就是为什么 ``Twig`` 提供了对复杂PHP变量的快速访问。考虑以下模板：

.. code-block:: html+twig

    <p>{{ user.name }} added this comment on {{ comment.publishedAt|date }}</p>

该 ``user.name`` 符号表示您要显示存储在变量（ ``user`` ）中的某些信息（ ``name`` ）。 ``user`` 是数组还是对象？ ``name`` 是属性还是方法？在Twig中，这些都不重要。

使用 ``foo.bar`` 语法， ``Twig`` 会尝试按以下顺序获取变量的值：

#. ``$foo['bar']`` (array and element);
#. ``$foo->bar`` (object and public property);
#. ``$foo->bar()`` (object and public method);
#. ``$foo->getBar()`` (object and *getter* method);
#. ``$foo->isBar()`` (object and *isser* method);
#. ``$foo->hasBar()`` (object and *hasser* method);
#. 如果以上都不存在，请使用 ``null`` 。

这允许在不更改模板代码的情况下演化您的应用程序代码（您可以从数组变量开始进行应用程序概念验证，然后再使用方法移动到对象等）。

.. _templates-link-to-pages:

链接到页面
~~~~~~~~~~~~~~~~

不用手动编写URL链接，而是使用 ``path()`` 函数根据 :ref:`路由配置 <routing-creating-routes>` 自动生成URL 。

以后，如果您要修改特定页面的URL，则只需更改路由配置即可：模板将自动生成新的URL

考虑以下路由配置：

.. configuration-block::

    .. code-block:: php-annotations

        // src/Controller/BlogController.php
        namespace App\Controller;

        // ...
        use Symfony\Component\Routing\Annotation\Route;

        class BlogController extends AbstractController
        {
            /**
             * @Route("/", name="blog_index")
             */
            public function index()
            {
                // ...
            }

            /**
             * @Route("/article/{slug}", name="blog_post")
             */
            public function show(string $slug)
            {
                // ...
            }
        }

    .. code-block:: yaml

        # config/routes.yaml
        blog_index:
            path:       /
            controller: App\Controller\BlogController::index

        blog_post:
            path:       /article/{slug}
            controller: App\Controller\BlogController::show

    .. code-block:: xml

        <!-- config/routes.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                https://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_index"
                path="/"
                controller="App\Controller\BlogController::index"/>

            <route id="blog_post"
                path="/article/{slug}"
                controller="App\Controller\BlogController::show"/>
        </routes>

    .. code-block:: php

        // config/routes.php
        use App\Controller\BlogController;
        use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

        return function (RoutingConfigurator $routes) {
            $routes->add('blog_index', '/')
                ->controller([BlogController::class, 'index'])
            ;

            $routes->add('blog_post', '/articles/{slug}')
                ->controller([BlogController::class, 'show'])
            ;
        };

使用 ``path()`` Twig函数链接到这些页面，并将 **路由名称** 作为第一个参数传递，将 **路由参数** 作为第二个可选参数传递。

.. code-block:: html+twig

    <a href="{{ path('blog_index') }}">Homepage</a>

    {# ... #}

    {% for post in blog_posts %}
        <h1>
            <a href="{{ path('blog_post', {slug: post.slug}) }}">{{ post.title }}</a>
        </h1>

        <p>{{ post.excerpt }}</p>
    {% endfor %}

该 ``path()`` 函数生成相对URL。如果您需要生成绝对URL（例如，在呈现电子邮件或RSS feed的模板时），请使用 ``url()`` 函数，该函数的参数与 ``path()`` （例如 ``<a href="{{ url('blog_index') }}"> ... </a>`` ）相同。

.. _templates-link-to-assets:

链接到CSS、JavaScript和图像资产
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果模板需要链接到静态资产（例如图像），Symfony提供了 ``asset()`` Twig函数来帮助生成该URL。首先，安装 ``asset`` 软件包：

.. code-block:: terminal

    $ composer require symfony/asset

您现在可以使用 ``asset()`` 函数：

.. code-block:: html+twig

    {# the image lives at "public/images/logo.png" #}
    <img src="{{ asset('images/logo.png') }}" alt="Symfony!"/>

    {# the CSS file lives at "public/css/blog.css" #}
    <link href="{{ asset('css/blog.css') }}" rel="stylesheet"/>

    {# the JS file lives at "public/bundles/acme/js/loader.js" #}
    <script src="{{ asset('bundles/acme/js/loader.js') }}"></script>

该 ``asset()`` 函数功能的主要目的是使应用程序更具可移植性。如果应用程序位于主机的根目录下（例如 ``https://example.com`` ），则呈现的路径应为 ``/images/logo.png`` 。但是，如果应用程序位于子目录中（例如 ``https://example.com/my_app`` ），则所有资产路径都应使用子目录来呈现（例如 ``/my_app/images/logo.png`` ）。 ``asset()`` 函数通过确定应用程序的使用方式并相应地生成正确的路径来处理这个问题。

.. tip::

    ``asset()`` 函数通过 
    :ref:`version <reference-framework-assets-version>` ，
    :ref:`version_format <reference-assets-version-format>` 和
    :ref:`json_manifest_path <reference-assets-json-manifest-path>` 配置选项支持各种缓存清除技术 。

.. tip::

       如果您想以现代方式帮助打包，版本控制和最小化JavaScript和CSS资产，请阅读 :doc:`Symfony's Webpack Encore </frontend>` 。

如果您需要资产的绝对URL，请使用 ``absolute_url()`` Twig函数按以下方式使用：

.. code-block:: html+twig

    <img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!"/>

    <link rel="shortcut icon" href="{{ absolute_url('favicon.png') }}">

.. _twig-app-variable:

App全局变量
~~~~~~~~~~~~~~~~~~~~~~~

Symfony创建一个上下文对象，该对象作为名为 ``app`` 的变量自动注入到每个Twig模板中。它提供了对某些应用程序信息的访问：

.. code-block:: html+twig

    <p>Username: {{ app.user.username ?? 'Anonymous user' }}</p>
    {% if app.debug %}
        <p>Request method: {{ app.request.method }}</p>
        <p>Application Environment: {{ app.environment }}</p>
    {% endif %}

通过 ``app`` 变量（这是一个 :class:`Symfony\\Bridge\\Twig\\AppVariable` 实例），您可以访问这些变量：

``app.user``
     :ref:`当前用户对象 <create-user-class>` 或者  ``null`` （如果用户没有通过认证）
``app.request``
     :class:`Symfony\\Component\\HttpFoundation\\Request` 对象存储了 :ref:`请求数据 <accessing-request-data>` （取决于应用程序，这个可以是 :ref:`子请求 <http-kernel-sub-requests>` 或者常规请求）。
``app.session``
    The :class:`Symfony\\Component\\HttpFoundation\\Session\\Session` object that
    represents the current :doc:`user's session </session>` or ``null`` if there is none.
``app.flashes``
    An array of all the :ref:`flash messages <flash-messages>` stored in the session.
    You can also get only the messages of some type (e.g. ``app.flashes('notice')``).
``app.environment``
    The name of the current :ref:`configuration environment <configuration-environments>`
    (``dev``, ``prod``, etc).
``app.debug``
    True if in :ref:`debug mode <debug-mode>`. False otherwise.
``app.token``
    A :class:`Symfony\\Component\\Security\\Core\\Authentication\\Token\\TokenInterface`
    object representing the security token.

In addition to the global ``app`` variable injected by Symfony, you can also
:doc:`inject variables automatically to all Twig templates </templating/global_variables>`.

.. _templates-rendering:

Rendering Templates
-------------------

Rendering a Template in Controllers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your controller extends from the :ref:`AbstractController <the-base-controller-class-services>`,
use the ``render()`` helper::

    // src/Controller/ProductController.php
    namespace App\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

    class ProductController extends AbstractController
    {
        public function index()
        {
            // ...

            return $this->render('product/index.html.twig', [
                'category' => '...',
                'promotions' => ['...', '...'],
            ]);
        }
    }

If your controller does not extend from ``AbstractController``, you'll need to
:ref:`fetch services in your controller <controller-accessing-services>` and
use the ``render()`` method of the ``twig`` service.

Rendering a Template in Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Inject the ``twig`` Symfony service into your own services and use its
``render()`` method. When using :doc:`service autowiring </service_container/autowiring>`
you only need to add an argument in the service constructor and type-hint it with
the :class:`Twig\\Environment` class::

    // src/Service/SomeService.php
    namespace App\Service;

    use Twig\Environment;

    class SomeService
    {
        private $twig;

        public function __construct(Environment $twig)
        {
            $this->twig = $twig;
        }

        public function someMethod()
        {
            // ...

            $htmlContents = $this->twig->render('product/index.html.twig', [
                'category' => '...',
                'promotions' => ['...', '...'],
            ]);
        }
    }

Rendering a Template in Emails
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Read the docs about the :ref:`mailer and Twig integration <mailer-twig>`.

.. _templates-render-from-route:

Rendering a Template Directly from a Route
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Although templates are usually rendered in controllers and services, you can
render static pages that don't need any variables directly from the route
definition. Use the special :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\TemplateController`
provided by Symfony:

.. configuration-block::

    .. code-block:: yaml

        # config/routes.yaml
        acme_privacy:
            path:          /privacy
            controller:    Symfony\Bundle\FrameworkBundle\Controller\TemplateController
            defaults:
                # the path of the template to render
                template:  'static/privacy.html.twig'

                # special options defined by Symfony to set the page cache
                maxAge:    86400
                sharedAge: 86400

                # optionally you can define some arguments passed to the template
                context:
                    site_name: 'ACME'
                    theme: 'dark'

    .. code-block:: xml

        <!-- config/routes.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing https://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="acme_privacy"
                path="/privacy"
                controller="Symfony\Bundle\FrameworkBundle\Controller\TemplateController">
                <!-- the path of the template to render -->
                <default key="template">static/privacy.html.twig</default>

                <!-- special options defined by Symfony to set the page cache -->
                <default key="maxAge">86400</default>
                <default key="sharedAge">86400</default>

                <!-- optionally you can define some arguments passed to the template -->
                <default key="context">
                    <default key="site_name">ACME</default>
                    <default key="theme">dark</default>
                </default>
            </route>
        </routes>

    .. code-block:: php

        // config/routes.php
        use Symfony\Bundle\FrameworkBundle\Controller\TemplateController;
        use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

        return function (RoutingConfigurator $routes) {
            $routes->add('acme_privacy', '/privacy')
                ->controller(TemplateController::class)
                ->defaults([
                    // the path of the template to render
                    'template'  => 'static/privacy.html.twig',

                    // special options defined by Symfony to set the page cache
                    'maxAge'    => 86400,
                    'sharedAge' => 86400,

                    // optionally you can define some arguments passed to the template
                    'context' => [
                        'site_name' => 'ACME',
                        'theme' => 'dark',
                    ]
                ])
            ;
        };

.. versionadded:: 5.1

    The ``context`` option was introduced in Symfony 5.1.

Checking if a Template Exists
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Templates are loaded in the application using a `Twig template loader`_, which
also provides a method to check for template existence. First, get the loader::

    // in a controller extending from AbstractController
    $loader = $this->get('twig')->getLoader();

    // in a service using autowiring
    use Twig\Environment;

    public function __construct(Environment $twig)
    {
        $loader = $twig->getLoader();
    }

Then, pass the path of the Twig template to the ``exists()`` method of the loader::

    if ($loader->exists('theme/layout_responsive.html.twig')) {
        // the template exists, do something
        // ...
    }

Debugging Templates
-------------------

Symfony provides several utilities to help you debug issues in your templates.

Linting Twig Templates
~~~~~~~~~~~~~~~~~~~~~~

The ``lint:twig`` command checks that your Twig templates don't have any syntax
errors. It's useful to run it before deploying your application to production
(e.g. in your continuous integration server):

.. code-block:: terminal

    # check all the application templates
    $ php bin/console lint:twig

    # you can also check directories and individual templates
    $ php bin/console lint:twig templates/email/
    $ php bin/console lint:twig templates/article/recent_list.html.twig

    # you can also show the deprecated features used in your templates
    $ php bin/console lint:twig --show-deprecations templates/email/

Inspecting Twig Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``debug:twig`` command lists all the information available about Twig
(functions, filters, global variables, etc.). It's useful to check if your
:doc:`custom Twig extensions </templating/twig_extension>` are working properly
and also to check the Twig features added when :ref:`installing packages <symfony-flex>`:

.. code-block:: terminal

    # list general information
    $ php bin/console debug:twig

    # filter output by any keyword
    $ php bin/console debug:twig --filter=date

    # pass a template path to show the physical file which will be loaded
    $ php bin/console debug:twig @Twig/Exception/error.html.twig

The Dump Twig Utilities
~~~~~~~~~~~~~~~~~~~~~~~

Symfony provides a :ref:`dump() function <components-var-dumper-dump>` as an
improved alternative to PHP's ``var_dump()`` function. This function is useful
to inspect the contents of any variable and you can use it in Twig templates too.

First, make sure that the VarDumper component is installed in the application:

.. code-block:: terminal

    $ composer require symfony/var-dumper

Then, use either the ``{% dump %}`` tag or the ``{{ dump() }}`` function
depending on your needs:

.. code-block:: html+twig

    {# templates/article/recent_list.html.twig #}
    {# the contents of this variable are sent to the Web Debug Toolbar
       instead of dumping them inside the page contents #}
    {% dump articles %}

    {% for article in articles %}
        {# the contents of this variable are dumped inside the page contents
           and they are visible on the web page #}
        {{ dump(article) }}

        <a href="/article/{{ article.slug }}">
            {{ article.title }}
        </a>
    {% endfor %}

To avoid leaking sensitive information, the ``dump()`` function/tag is only
available in the ``dev`` and ``test`` :ref:`configuration environments <configuration-environments>`.
If you try to use it in the ``prod`` environment, you will see a PHP error.

.. _templates-reuse-contents:

Reusing Template Contents
-------------------------

.. _templates-include:

Including Templates
~~~~~~~~~~~~~~~~~~~

If certain Twig code is repeated in several templates, you can extract it into a
single "template fragment" and include it in other templates. Imagine that the
following code to display the user information is repeated in several places:

.. code-block:: html+twig

    {# templates/blog/index.html.twig #}

    {# ... #}
    <div class="user-profile">
        <img src="{{ user.profileImageUrl }}"/>
        <p>{{ user.fullName }} - {{ user.email }}</p>
    </div>

First, create a new Twig template called ``blog/_user_profile.html.twig`` (the
``_`` prefix is optional, but it's a convention used to better differentiate
between full templates and template fragments).

Then, remove that content from the original ``blog/index.html.twig`` template
and add the following to include the template fragment:

.. code-block:: twig

    {# templates/blog/index.html.twig #}

    {# ... #}
    {{ include('blog/_user_profile.html.twig') }}

The ``include()`` Twig function takes as argument the path of the template to
include. The included template has access to all the variables of the template
that includes it (use the `with_context`_ option to control this).

You can also pass variables to the included template. This is useful for example
to rename variables. Imagine that your template stores the user information in a
variable called ``blog_post.author`` instead of the ``user`` variable that the
template fragment expects. Use the following to *rename* the variable:

.. code-block:: twig

    {# templates/blog/index.html.twig #}

    {# ... #}
    {{ include('blog/_user_profile.html.twig', {user: blog_post.author}) }}

.. _templates-embed-controllers:

Embedding Controllers
~~~~~~~~~~~~~~~~~~~~~

:ref:`Including template fragments <templates-include>` is useful to reuse the
same content on several pages. However, this technique is not the best solution
in some cases.

Imagine that the template fragment displays the three most recent blog articles.
To do that, it needs to make a database query to get those articles. When using
the ``include()`` function, you'd need to do the same database query in every
page that includes the fragment. This is not very convenient.

A better alternative is to **embed the result of executing some controller**
with the ``render()`` and ``controller()`` Twig functions.

First, create the controller that renders a certain number of recent articles::

    // src/Controller/BlogController.php
    namespace App\Controller;

    // ...

    class BlogController extends AbstractController
    {
        public function recentArticles($max = 3)
        {
            // get the recent articles somehow (e.g. making a database query)
            $articles = ['...', '...', '...'];

            return $this->render('blog/_recent_articles.html.twig', [
                'articles' => $articles
            ]);
        }
    }

Then, create the ``blog/_recent_articles.html.twig`` template fragment (the
``_`` prefix in the template name is optional, but it's a convention used to
better differentiate between full templates and template fragments):

.. code-block:: html+twig

    {# templates/blog/_recent_articles.html.twig #}
    {% for article in articles %}
        <a href="{{ path('blog_show', {slug: article.slug}) }}">
            {{ article.title }}
        </a>
    {% endfor %}

Now you can call to this controller from any template to embed its result:

.. code-block:: html+twig

    {# templates/base.html.twig #}

    {# ... #}
    <div id="sidebar">
        {# if the controller is associated with a route, use the path() or url() functions #}
        {{ render(path('latest_articles', {max: 3})) }}
        {{ render(url('latest_articles', {max: 3})) }}

        {# if you don't want to expose the controller with a public URL,
           use the controller() function to define the controller to execute #}
        {{ render(controller(
            'App\\Controller\\BlogController::recentArticles', {max: 3}
        )) }}
    </div>

.. _fragments-path-config:

When using the ``controller()`` function, controllers are not accessed using a
regular Symfony route but through a special URL used exclusively to serve those
template fragments. Configure that special URL in the ``fragments`` option:

.. configuration-block::

    .. code-block:: yaml

        # config/packages/framework.yaml
        framework:
            # ...
            fragments: { path: /_fragment }

    .. code-block:: xml

        <!-- config/packages/framework.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:framework="http://symfony.com/schema/dic/symfony"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/symfony https://symfony.com/schema/dic/symfony/symfony-1.0.xsd">

            <!-- ... -->
            <framework:config>
                <framework:fragment path="/_fragment"/>
            </framework:config>
        </container>

    .. code-block:: php

        // config/packages/framework.php
        $container->loadFromExtension('framework', [
            // ...
            'fragments' => ['path' => '/_fragment'],
        ]);

.. caution::

    Embedding controllers require making requests to those controllers and
    rendering some templates as result. This can have a significant impact in
    the application performance if you embed lots of controllers. If possible,
    :doc:`cache the template fragment </http_cache/esi>`.

.. seealso::

    Templates can also :doc:`embed contents asynchronously </templating/hinclude>`
    with the ``hinclude.js`` JavaScript library.

Template Inheritance and Layouts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As your application grows you'll find more and more repeated elements between
pages, such as headers, footers, sidebars, etc. :ref:`Including templates <templates-include>`
and :ref:`embedding controllers <templates-embed-controllers>` can help, but
when pages share a common structure, it's better to use **inheritance**.

The concept of `Twig template inheritance`_ is similar to PHP class inheritance.
You define a parent template that other templates can extend from and child
templates can override parts of the parent template.

Symfony recommends the following three-level template inheritance for medium and
complex applications:

* ``templates/base.html.twig``, defines the common elements of all application
  templates, such as ``<head>``, ``<header>``, ``<footer>``, etc.;
* ``templates/layout.html.twig``, extends from ``base.html.twig`` and defines
  the content structure used in all or most of the pages, such as a two-column
  content + sidebar layout. Some sections of the application can define their
  own layouts (e.g. ``templates/blog/layout.html.twig``);
* ``templates/*.html.twig``, the application pages which extend from the main
  ``layout.html.twig`` template or any other section layout.

In practice, the ``base.html.twig`` template would look like this:

.. code-block:: html+twig

    {# templates/base.html.twig #}
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="UTF-8">
            <title>{% block title %}My Application{% endblock %}</title>
        </head>
        <body>
            <div id="sidebar">
                {% block sidebar %}
                    <ul>
                        <li><a href="{{ path('homepage') }}">Home</a></li>
                        <li><a href="{{ path('blog_index') }}">Blog</a></li>
                    </ul>
                {% endblock %}
            </div>

            <div id="content">
                {% block body %}{% endblock %}
            </div>
        </body>
    </html>

The `Twig block tag`_ defines the page sections that can be overridden in the
child templates. They can be empty, like the ``body`` block or define a default
content, like the ``title`` block, which is displayed when child templates don't
override them.

The ``blog/layout.html.twig`` template could be like this:

.. code-block:: html+twig

    {# templates/blog/layout.html.twig #}
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Blog</h1>

        {% block content %}{% endblock %}
    {% endblock %}

The template extends from ``base.html.twig`` and only defines the contents of
the ``body`` block. The rest of the parent template blocks will display their
default contents. However, they can be overridden by the third-level inheritance
template, such as ``blog/index.html.twig``, which displays the blog index:

.. code-block:: html+twig

    {# templates/blog/index.html.twig #}
    {% extends 'blog/layout.html.twig' %}

    {% block title %}Blog Index{% endblock %}

    {% block content %}
        {% for article in articles %}
            <h2>{{ article.title }}</h2>
            <p>{{ article.body }}</p>
        {% endfor %}
    {% endblock %}

This template extends from the second-level template (``blog/layout.html.twig``)
but overrides blocks of different parent templates: ``content`` from
``blog/layout.html.twig`` and ``title`` from ``base.html.twig``.

When you render the ``blog/index.html.twig`` template, Symfony uses three
different templates to create the final contents. This inheritance mechanism
boosts your productivity because each template includes only its unique contents
and leaves the repeated contents and HTML structure to some parent templates.

Read the `Twig template inheritance`_ docs to learn more about how to reuse
parent block contents when overriding templates and other advanced features.

Output Escaping
---------------

Imagine that your template includes the ``Hello {{ name }}`` code to display the
user name. If a malicious user sets ``<script>alert('hello!')</script>`` as
their name and you output that value unchanged, the application will display a
JavaScript popup window.

This is known as a `Cross-Site Scripting`_ (XSS) attack. And while the previous
example seems harmless, the attacker could write more advanced JavaScript code
to performs malicious actions.

To prevent this attack, use *"output escaping"* to transform the characters
which have special meaning (e.g. replace ``<`` by the ``&lt;`` HTML entity).
Symfony applications are safe by default because they perform automatic output
escaping thanks to the :ref:`Twig autoescape option <config-twig-autoescape>`:

.. code-block:: html+twig

    <p>Hello {{ name }}</p>
    {# if 'name' is '<script>alert('hello!')</script>', Twig will output this:
       '<p>Hello &lt;script&gt;alert(&#39;hello!&#39;)&lt;/script&gt;</p>' #}

If you are rendering a variable that is trusted and contains HTML contents,
use the `Twig raw filter`_ to disable the output escaping for that variable:

.. code-block:: html+twig

    <h1>{{ product.title|raw }}</h1>
    {# if 'product.title' is 'Lorem <strong>Ipsum</strong>', Twig will output
       exactly that instead of 'Lorem &lt;strong&gt;Ipsum&lt;/strong&gt;' #}

Read the `Twig output escaping docs`_ to learn more about how to disable output
escaping for a block or even an entire template.

.. _templates-namespaces:

Template Namespaces
-------------------

Although most applications store their templates in the default ``templates/``
directory, you may need to store some or all of them in different directories.
Use the ``twig.paths`` option to configure those extra directories. Each path is
defined as a ``key: value`` pair where the ``key`` is the template directory and
the ``value`` is the Twig namespace, which is explained later:

.. configuration-block::

    .. code-block:: yaml

        # config/packages/twig.yaml
        twig:
            # ...
            paths:
                # directories are relative to the project root dir (but you
                # can also use absolute directories)
                'email/default/templates': ~
                'backend/templates': ~

    .. code-block:: xml

        <!-- config/packages/twig.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:twig="http://symfony.com/schema/dic/twig"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/twig https://symfony.com/schema/dic/twig/twig-1.0.xsd">

            <twig:config>
                <!-- ... -->
                <!-- directories are relative to the project root dir (but you
                     can also use absolute directories -->
                <twig:path>email/default/templates</twig:path>
                <twig:path>backend/templates</twig:path>
            </twig:config>
        </container>

    .. code-block:: php

        // config/packages/twig.php
        $container->loadFromExtension('twig', [
            // ...
            'paths' => [
                // directories are relative to the project root dir (but you
                // can also use absolute directories)
                'email/default/templates' => null,
                'backend/templates' => null,
            ],
        ]);

When rendering a template, Symfony looks for it first in the ``twig.paths``
directories that don't define a namespace and then falls back to the default
template directory (usually, ``templates/``).

Using the above configuration, if your application renders for example the
``layout.html.twig`` template, Symfony will first look for
``email/default/templates/layout.html.twig`` and ``backend/templates/layout.html.twig``.
If any of those templates exists, Symfony will use it instead of using
``templates/layout.html.twig``, which is probably the template you wanted to use.

Twig solves this problem with **namespaces**, which group several templates
under a logic name unrelated to their actual location. Update the previous
configuration to define a namespace for each template directory:

.. configuration-block::

    .. code-block:: yaml

        # config/packages/twig.yaml
        twig:
            # ...
            paths:
                'email/default/templates': 'email'
                'backend/templates': 'admin'

    .. code-block:: xml

        <!-- config/packages/twig.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:twig="http://symfony.com/schema/dic/twig"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd
                http://symfony.com/schema/dic/twig https://symfony.com/schema/dic/twig/twig-1.0.xsd">

            <twig:config>
                <!-- ... -->
                <twig:path namespace="email">email/default/templates</twig:path>
                <twig:path namespace="admin">backend/templates</twig:path>
            </twig:config>
        </container>

    .. code-block:: php

        // config/packages/twig.php
        $container->loadFromExtension('twig', [
            // ...
            'paths' => [
                'email/default/templates' => 'email',
                'backend/templates' => 'admin',
            ],
        ]);

Now, if you render the ``layout.html.twig`` template, Symfony will render the
``templates/layout.html.twig`` file. Use the special syntax ``@`` + namespace to
refer to the other namespaced templates (e.g. ``@email/layout.html.twig`` and
``@admin/layout.html.twig``).

.. note::

    A single Twig namespace can be associated with more than one template
    directory. In that case, the order in which paths are added is important
    because Twig will start looking for templates from the first defined path.

Bundle Templates
~~~~~~~~~~~~~~~~

If you :ref:`install packages/bundles <symfony-flex>` in your application, they
may include their own Twig templates (in the ``Resources/views/`` directory of
each bundle). To avoid messing with your own templates, Symfony adds bundle
templates under an automatic namespace created after the bundle name.

For example, the templates of a bundle called ``AcmeFooBundle`` are available
under the ``AcmeFoo`` namespace. If this bundle includes the template
``<your-project>/vendor/acmefoo-bundle/Resources/views/user/profile.html.twig``,
you can refer to it as ``@AcmeFoo/user/profile.html.twig``.

.. tip::

    You can also :ref:`override bundle templates <override-templates>` in case
    you want to change some parts of the original bundle templates.

Learn more
----------

.. toctree::
    :maxdepth: 1
    :glob:

    /templating/*

.. _`Twig`: https://twig.symfony.com
.. _`tags`: https://twig.symfony.com/doc/2.x/tags/index.html
.. _`filters`: https://twig.symfony.com/doc/2.x/filters/index.html
.. _`functions`: https://twig.symfony.com/doc/2.x/functions/index.html
.. _`with_context`: https://twig.symfony.com/doc/2.x/functions/include.html
.. _`Twig template loader`: https://twig.symfony.com/doc/2.x/api.html#loaders
.. _`Twig raw filter`: https://twig.symfony.com/doc/2.x/filters/raw.html
.. _`Twig output escaping docs`: https://twig.symfony.com/doc/2.x/api.html#escaper-extension
.. _`snake case`: https://en.wikipedia.org/wiki/Snake_case
.. _`Twig template inheritance`: https://twig.symfony.com/doc/2.x/tags/extends.html
.. _`Twig block tag`: https://twig.symfony.com/doc/2.x/tags/block.html
.. _`Cross-Site Scripting`: https://en.wikipedia.org/wiki/Cross-site_scripting
