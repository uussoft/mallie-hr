Flex: 编写您的应用程序
==============================

阅读了本教程的第一部分后，您已经确定Symfony值得您再花10分钟。你的选择是正确的！在第二部分中，
您将学习Symfony Flex：这是一款了不起的工具，它使添加新功能就像运行一个命令一样简单。这
也是Symfony理想用于小型微服务或大型应用程序的原因。是否感到很好奇？

Symfony: 启动Micro!
---------------------

除非您要构建一个纯API（稍后会有详细介绍！），你可能想要呈现HTML。要做到这一点，您将使用 `Twig`_。
Twig是一个灵活，快速且安全的PHP模板引擎。它使您的模板更具可读性和简洁性；使它们对网页设计师更加友好。

Twig默认是不安装的，当你开始一个Symfony项目时，它的体积很小： ``composer.json`` 文件中仅包含关键的依赖项：

.. code-block:: text

    "require": {
        "...",
        "symfony/console": "^4.1",
        "symfony/flex": "^1.0",
        "symfony/framework-bundle": "^4.1",
        "symfony/yaml": "^4.1"
    }

这使得Symfony不同于任何其他PHP框架！Symfony是小而简单且快速的应用程序，而不是从开始就带给您可能需要的每一个功能的 *笨重* 应用程序。你可以完全掌控添加的内容。

Flex Recipes and Aliases
------------------------

那我们该如何安装和配置Twig？通过运行下面的命令：

.. code-block:: terminal

    $ composer require twig

由于Symfony Flex：一个已经安装在项目中的Composer插件，两件非常有意思的事情在幕后发生了。

首先， ``twig`` 不是一个Composer软件包的名称：它是一个Flex别名，指向 ``symfony/twig-bundle``。Flex会为Composer解析该别名。

其次，Flex为  ``symfony/twig-bundle`` 安装了一个 ``recipe`` 。什么是 ``recipe`` ？这是库通过添加和修改文件自动配置自身的一种方式。辛亏有了 ``recipe`` ，添加功能是无缝和自动化的：安装软件包即可完成！

您可以通过转到 `https://flex.symfony.com`_ 找到 ``Recipes`` 和  ``Aliases`` 的完整列表。

``Recipes`` 做了哪些事情？除了在 ``config/bundles.php`` 中自动启用功能外。它还添加了三项功能：

``config/packages/twig.yaml``
       一个配置文件，使用合理的默认值设置Twig。

``config/routes/dev/twig.yaml``
    一种可以帮助您调试错误页面的路由。

``templates/``
    这是模板所在的目录， ``Recipe`` 还添加了 ``base.html.twig`` 布局模板。

Twig: 渲染模板
--------------------------

多亏了Flex，您只需要执行一个命令，就可以立即开始使用Twig：

.. code-block:: diff

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Component\Routing\Annotation\Route;
    - use Symfony\Component\HttpFoundation\Response;
    + use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

    -class DefaultController
    +class DefaultController extends AbstractController
     {
         /**
          * @Route("/hello/{name}")
          */
         public function index($name)
         {
    -        return new Response("Hello $name!");
    +        return $this->render('default/index.html.twig', [
    +            'name' => $name,
    +        ]);
         }
    }

通过扩展 ``AbstractController`` ，您现在可以访问许多快捷方法和工具，如 ``render()`` 。创建新模板如下：

.. code-block:: html+twig

    {# templates/default/index.html.twig #}
    <h1>Hello {{ name }}</h1>

就是这样！该 ``{{ name }}`` 语法将打印从控制器传入的 ``name`` 变量。如果您不熟悉Twig，稍后，您将了解有关其语法和功能的更多信息。

但是，现在，该页面仅包含 ``h1`` 标签。要为其提供HTML布局，请扩展 ``base.html.twig`` ：

.. code-block:: html+twig

    {# templates/default/index.html.twig #}
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Hello {{ name }}</h1>
    {% endblock %}

这称为模板继承：现在页面从 ``base.html.twig`` 继承了HTML结构。

探查器：调试的好助手
----------------------------

Symfony最酷的功能之一还没有安装！我们来解决这个问题：

.. code-block:: terminal

    $ composer require profiler

这是另一个别名！Flex还安装了另一个 ``Recipe`` ，该 ``Recipe`` 可以自动配置Symfony的探查器（Symfony's Profiler），结果怎么样呢？刷新一下！

看到底部的黑条了吗？那是Web调试工具栏，也是您的新好朋友。通过将鼠标悬停在每个图标上，您可以获得有关执行了什么控制器的信息，性能信息，高速缓存命中和未命中等等。单击任何图标进入事件探查器，您将在其中获得更详细的调试和性能数据！

随着您安装更多库或者软件包，您将获得更多工具（例如显示数据库查询的Web调试工具栏图标）。

现在，您可以直接使用探查器，因为它已经通过Recipe自动配置完成了。我们还能安装什么？

丰富的API支持
----------------

您正在构建API吗？您可以通过任何控制器返回JSON::

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;

    class DefaultController extends AbstractController
    {
        // ...

        /**
         * @Route("/api/hello/{name}")
         */
        public function apiExample($name)
        {
            return $this->json([
                'name' => $name,
                'symfony' => 'rocks',
            ]);
        }
    }

但是对于创建真正丰富的API，请尝试安装  `API Platform`_ ：

.. code-block:: terminal

    $ composer require api

这是 ``api-platform/api-pack`` :ref:`Symfony pack <symfony-packs>` 的别名，该软件包依赖于其他几个软件包，例如Symfony的Validator和Security组件以及Doctrine ORM。实际上，Flex安装了5个 ``Recipe`` ！

像往常一样，我们可以立即开始使用新库。是否要为 ``product`` 表格创建丰富的API？创建一个 ``product`` 实体并为其添加 ``@ApiResource()`` 注释::

    // src/Entity/Product.php
    namespace App\Entity;

    use ApiPlatform\Core\Annotation\ApiResource;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity()
     * @ApiResource()
     */
    class Product
    {
        /**
         * @ORM\Id
         * @ORM\GeneratedValue(strategy="AUTO")
         * @ORM\Column(type="integer")
         */
        private $id;

        /**
         * @ORM\Column(type="string")
         */
        private $name;

        /**
         * @ORM\Column(type="integer")
         */
        private $price;

        // ...
    }

很好！现在，您可以使用端点来列出，添加，更新和删除产品！不相信我吗？通过运行以下命令列出您的路由：

.. code-block:: terminal

    $ php bin/console debug:router

    ------------------------------ -------- -------------------------------------
     Name                           Method   Path
    ------------------------------ -------- -------------------------------------
     api_products_get_collection    GET      /api/products.{_format}
     api_products_post_collection   POST     /api/products.{_format}
     api_products_get_item          GET      /api/products/{id}.{_format}
     api_products_put_item          PUT      /api/products/{id}.{_format}
     api_products_delete_item       DELETE   /api/products/{id}.{_format}
     ...
    ------------------------------ -------- -------------------------------------

.. _ easily-remove-recipes:

卸载Recipes
----------------

执行下面的命令来删除库：

.. code-block:: terminal

    $ composer remove api

Flex将卸载配方：删除文件并撤消更改以使您的应用恢复到之前状态。让您毫无后顾之忧的进行实验。

更多功能、架构和速度
-------------------------------------

希望您和我一样对Flex感到兴奋！但是我们还有一章，这是迄今为止最重要的一章。我想向您展示Symfony如何使您能够快速构建功能而
不牺牲代码质量或性能。一切都与服务容器有关，这是Symfony的超能力。阅读：关于 :doc:`架构 </quick_tour/the_architecture>` 。

.. _`https://flex.symfony.com`: https://flex.symfony.com
.. _`API Platform`: https://api-platform.com/
.. _`Twig`: https://twig.symfony.com/
