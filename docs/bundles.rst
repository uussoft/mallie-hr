.. index::
   single: Bundles

.. _page-creation-bundles:

捆绑系统
=================

.. caution::

       在4.0之前的Symfony版本中，建议使用捆绑包组织自己的应用程序代码。不再建议这样做，捆绑软件只应用于在多个应用程序之间共享代码和功能。

捆绑包类似于其他软件中的插件，但更好。Symfony框架的核心功能是通过捆绑包（FrameworkBundle，SecurityBundle，DebugBundle等）实现的。还可以通过  `第三方捆绑包`_ 在您的应用程序中添加新功能。

在应用程序中必须按 ``config/bundles.php`` 文件中的每个 :ref:`环境 <configuration-environments>` 来启用应用程序捆绑软件::

    // config/bundles.php
    return [
        // 'all' means that the bundle is enabled for any Symfony environment
        Symfony\Bundle\FrameworkBundle\FrameworkBundle::class => ['all' => true],
        Symfony\Bundle\SecurityBundle\SecurityBundle::class => ['all' => true],
        Symfony\Bundle\TwigBundle\TwigBundle::class => ['all' => true],
        Symfony\Bundle\MonologBundle\MonologBundle::class => ['all' => true],
        Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle::class => ['all' => true],
        Doctrine\Bundle\DoctrineBundle\DoctrineBundle::class => ['all' => true],
        Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle::class => ['all' => true],
        // this bundle is enabled only in 'dev'  and 'test', so you can't use it in 'prod'
        Symfony\Bundle\WebProfilerBundle\WebProfilerBundle::class => ['dev' => true, 'test' => true],
    ];

.. tip::

       在使用了 :ref:`Symfony Flex <symfony-flex>` 的默认Symfony应用程序中，安装/删除捆绑包时会自动为您启用/禁用捆绑包，因此您无需查看或编辑此 ``bundles.php`` 文件。

创建一个捆绑包
-----------------

本节将教你如何创建并启用新捆绑包，仅需几个步骤。新的捆绑包名称为AcmeTestBundle，其中的 ``Acme`` 部分只是一个虚拟名称，实际开发中应替换为代表您或您的组织的 **供应商** 名称（例如，某些名为 ``ABC`` 公司的ABCTestBundle）。

首先创建一个 ``src/Acme/TestBundle/`` 目录并添加一个名为 ``AcmeTestBundle.php`` 新的文件::

    // src/Acme/TestBundle/AcmeTestBundle.php
    namespace App\Acme\TestBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AcmeTestBundle extends Bundle
    {
        //...
    }

.. tip::

       名称AcmeTestBundle遵循标准的 :ref:`Bundle命名约定 <bundles-naming-conventions>`。您还可以通过命名此类TestBundle（把文件命名为 ``TestBundle.php`` ）来选择将捆绑包的名称简称为TestBundle 。

这个空的类是创建新捆绑包所需的唯一部分。尽管通常是空的，但该类功能强大，可用于自定义捆绑软件的行为。既然已经创建了捆绑包，那就启用它::

    // config/bundles.php
    return [
        // ...
        App\Acme\TestBundle\AcmeTestBundle::class => ['all' => true],
    ];

虽然它还没做任何事情，但是AcmeTestBundle现在可以使用了。

捆绑包目录结构
--------------------------

捆绑软件的目录结构旨在帮助使所有Symfony捆绑软件之间的代码保持一致。它遵循一组约定，但是可以根据需要灵活调整：

``Controller/``
       包含捆绑软件的控制器（例如： ``RandomController.php``）。

``DependencyInjection/``
       包含某些 **依赖注入** 扩展类，这些类可能会导入服务配置，注册编译器传递或更多（不需要此目录）。

``Resources/config/``
       内部配置，包括路由配置（例如：  ``routing.yaml``）。

``Resources/views/``
       保存按控制器名称组织的模板（例如：  ``Random/index.html.twig``）。

``Resources/public/``
       包含web资产（图像、样式表等），并通过 ``assets:install`` 控制台命令复制或象征性地链接到项目的 ``public/`` 目录中。

``Tests/``
       包含捆绑软件的所有测试。

捆绑包可以像它实现的功能一样小或大。它只包含您需要的文件，而不包含任何其他文件。

在阅读指南时，您将学习如何将对象保存到数据库、创建和验证表单、为应用程序创建翻译、编写测试等等。这些在捆绑包中都有各自的位置和作用。

了解更多
----------

* :doc:`如何覆盖捆绑包的任何部分 </bundles/override>`
* :doc:`可重复使用捆绑软件的最佳做法 </bundles/best_practices>`
* :doc:`如何为捆绑包创建友好配置 </bundles/configuration>`
* :doc:`如何在捆绑包中加载服务配置 </bundles/extension>`
* :doc:`如何简化多个捆绑包的配置 </bundles/prepend_extension>`

.. _`第三方捆绑包`: https://github.com/search?q=topic%3Asymfony-bundle&type=Repositories
