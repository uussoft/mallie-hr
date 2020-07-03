架构
================

没想到前两部分结束后你还会在这里？你的努力很快就会得到回报。前两部分并没有对框架的体系结构做太深入的研究。现在让我们深入到架构中。

添加日志系统
-----------

一个新的Symfony应用程序是微型的：它基本上只是一个路由和控制器系统构成。但是多亏了Flex，这样就很容易安装其他更多的功能。

执行下面的命令，安装日志系统：

.. code-block:: terminal

    $ composer require logger

安装和配置（通过 ``Recipe`` ）功能强大的 `Monolog`_ 库。要在控制器中使用日志功能，请添加 ``LoggerInterface`` 提示新的参数类型::

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use Psr\Log\LoggerInterface;
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;

    class DefaultController extends AbstractController
    {
        /**
         * @Route("/hello/{name}")
         */
        public function index($name, LoggerInterface $logger)
        {
            $logger->info("Saying hello to $name!");

            // ...
        }
    }

就这样，新的日志消息将被写入 ``var/log/dev.log`` 。日志文件路径以及不同的日志记录方法都可以通过更新 ``Recipe`` 添加的配置文件来进行配置。

服务和自动装配
---------------------

然而，刚刚发生了件很酷的事。Symfony框架读取了 ``LoggerInterface`` 类型提示，并自动发现它应该向我们传递Logger对象！这叫做自动装配（ ``autowiring`` ）。

使用Symfony框架开发的应用程序中完成的每一项工作都是由 ``对象`` 完成的： ``Logger`` 对象记录事情，``Twig`` 对象呈现模板。我们将这些对象称为 ``服务`` ，它们是帮助您构建丰富功能的工具。

您可以使用类型提示要求Symfony框架向您传递服务。您还可以使用其他哪些类或接口？通过运行下面的命令了解并查找：

.. code-block:: terminal

    $ php bin/console debug:autowiring

      # this is just a *small* sample of the output...

      Describes a logger instance.
      Psr\Log\LoggerInterface (monolog.logger)

      Request stack that controls the lifecycle of requests.
      Symfony\Component\HttpFoundation\RequestStack (request_stack)

      Interface for the session.
      Symfony\Component\HttpFoundation\Session\SessionInterface (session)

      RouterInterface is the interface that all Router classes must implement.
      Symfony\Component\Routing\RouterInterface (router.default)

      [...]

这只是一个完整列表的简短摘要！随着你添加更多的包，这个工具列表将会增长！

创建服务
-----------------

为了使应用程序代码井井有条，您甚至可以创建自己的服务！假设您要生成随机问候（例如 ``Hello`` ，``Yo`` 等）。无需将这些代码直接放在您的控制器中，而是创建一个新服务类::

    // src/GreetingGenerator.php
    namespace App;

    class GreetingGenerator
    {
        public function getRandomGreeting()
        {
            $greetings = ['Hey', 'Yo', 'Aloha'];
            $greeting = $greetings[array_rand($greetings)];

            return $greeting;
        }
    }

很好，您可以在控制器中使用它::

    // src/Controller/DefaultController.php
    namespace App\Controller;

    use App\GreetingGenerator;
    use Psr\Log\LoggerInterface;
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;

    class DefaultController extends AbstractController
    {
        /**
         * @Route("/hello/{name}")
         */
        public function index($name, LoggerInterface $logger, GreetingGenerator $generator)
        {
            $greeting = $generator->getRandomGreeting();

            $logger->info("Saying $greeting to $name!");

            // ...
        }
    }

就这样！Symfony框架将自动实例化 ``GreetingGenerrator`` 并把它作为参数进行传递。但是，我们是否可以将日志功能逻辑移动到 ``GreetingGenerator`` 吗？可以的，您可以使用服务内部的自动装配（ ``autowiring`` ）来访问其他服务。唯一的区别是它是在构造函数中完成的：

.. code-block:: diff

    // src/GreetingGenerator.php
    + use Psr\Log\LoggerInterface;

    class GreetingGenerator
    {
    +     private $logger;
    +
    +     public function __construct(LoggerInterface $logger)
    +     {
    +         $this->logger = $logger;
    +     }

        public function getRandomGreeting()
        {
            // ...

     +        $this->logger->info('Using the greeting: '.$greeting);

             return $greeting;
        }
    }

这样也是可以的：不需要配置，节约时间。

Twig扩展和自动配置
----------------------------------

由于Symfony框架的服务处理能力，您可以通过多种方式扩展Symfony框架，比如为复杂的授权规则创建事件订阅服务器或安全投票者。让我们为Twig添加一个名为 ``greet`` 的新过滤器。怎样处理呢？创建扩展类 ``AbstractExtension``::

    // src/Twig/GreetExtension.php
    namespace App\Twig;

    use App\GreetingGenerator;
    use Twig\Extension\AbstractExtension;
    use Twig\TwigFilter;

    class GreetExtension extends AbstractExtension
    {
        private $greetingGenerator;

        public function __construct(GreetingGenerator $greetingGenerator)
        {
            $this->greetingGenerator = $greetingGenerator;
        }

        public function getFilters()
        {
            return [
                new TwigFilter('greet', [$this, 'greetUser']),
            ];
        }

        public function greetUser($name)
        {
            $greeting =  $this->greetingGenerator->getRandomGreeting();

            return "$greeting $name!";
        }
    }

仅创建一个文件后，就可以立即使用：

.. code-block:: html+twig

    {# templates/default/index.html.twig #}
    {# Will print something like "Hey Symfony!" #}
    <h1>{{ name|greet }}</h1>

它是如何工作的呢？Symfony框架注意到您的类扩展了 ``AbstractExtension`` ，因此会自动将其注册为 ``Twig`` 扩展。这就是所谓的自动配置，它适用于做很多事情。创建一个类，然后扩展一个基类（或实现一个接口）。其他的工作由Symfony框架负责。

极速：缓存容器
-----------------------------------

在看到Symfony框架自动处理了很多工作之后，您可能会想：“这不会影响性能吗？” 其实不！Symfony框架的速度很快。

现在您可能想知道当更新文件并且需要重建缓存时会发生什么情况？我喜欢你的想法！它足够聪明，可以在加载下一页时重建。但这确实是下一节的主题。

开发与生产：环境
-------------------------------------------

框架的主要工作之一是使调试变得更加容易！我们的应用程序有很多很棒的工具：web调试工具栏显示在页面的底部，错误很大，漂亮且明确，任何配置缓存都会在需要时自动重建。

但是，当你部署到生产环境时，我们需要隐藏这些工具并优化速度！

这是由Symfony框架的 **环境** 系统解决的，它有三种环境分别为： ``dev``, ``prod`` 和  ``test`` 。根据环境，Symfony框架根据环境在 ``config/``目录中加载不同的文件：

.. code-block:: text

    config/
    ├─ services.yaml
    ├─ ...
    └─ packages/
        ├─ framework.yaml
        ├─ ...
        ├─ **dev/**
            ├─ monolog.yaml
            └─ ...
        ├─ **prod/**
            └─ monolog.yaml
        └─ **test/**
            ├─ framework.yaml
            └─ ...
    └─ routes/
        ├─ annotations.yaml
        └─ **dev/**
            ├─ twig.yaml
            └─ web_profiler.yaml

这是一个很不错的想法：通过修改一个配置（环境），您的应用程序将从调试友好的体验转变为针对速度进行了优化的体验。

那我们将如何来切换环境呢？将 ``APP_ENV`` 环境变量从更改 ``dev`` 为 ``prod`` ，这样就会完成了环境的切换：

.. code-block:: diff

    # .env
    - APP_ENV=dev
    + APP_ENV=prod

接下来我想多谈谈环境变量的问题。所以将值改回 ``dev`` ：在本地工作时，调试工具非常有用。

环境变量
---------------------

每个应用程序在每个服务器上都包含不同的环境配置，比如数据库连接信息或密码。这些东西应该怎么存放？在文件里？还是在其他地方？

Symfony框架遵循行业最佳实践，将基于服务器的配置存储为 **环境** 变量。这意味着Symfony框架可以与平台即服务（Platform as a Service，PaaS）部署系统以及Docker完美地协同工作。

但是在开发过程中设置环境变量可能会很痛苦。这就是为什么你的应用程序会自动加载一个 ``.env`` 文件。然后，此文件中的键将成为环境变量，并由应用程序读取：

.. code-block:: bash

    # .env
    ###> symfony/framework-bundle ###
    APP_ENV=dev
    APP_SECRET=cc86c7ca937636d5ddf1b754beb22a10
    ###< symfony/framework-bundle ###

一开始，该文件不会包含太多内容。但是随着应用程序的增长，您将根据需要添加更多的配置。但是，事实上，它变得更有趣了！假设你的应用程序需要数据库ORM。让我们安装 ``Doctrine ORM`` ：

.. code-block:: terminal

    $ composer require doctrine

由于Flex安装了新 ``Recipe`` ，请再次查看 ``.env`` 文件：

.. code-block:: diff

    ###> symfony/framework-bundle ###
    APP_ENV=dev
    APP_SECRET=cc86c7ca937636d5ddf1b754beb22a10
    ###< symfony/framework-bundle ###

    + ###> doctrine/doctrine-bundle ###
    + # ...
    + DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
    + ###< doctrine/doctrine-bundle ###

新的 ``DATABASE_URL`` 环境变量已自动添加，并且已由新的 ``doctrine.yaml`` 配置文件引用。通过将环境变量和Flex结合使用，您将无需任何额外工作即可体验行业最佳实践。

继续！
-----------

说我疯了，但是在阅读完这一部分之后，你应该对Symfony框架最重要的部分感到满意。Symfony框架中的所有内容均旨在摆脱你的束缚，因此您可以继续编码和添加功能，并以所需的速度和质量进行操作。

这就是快速浏览的全部内容。从身份验证到表单，再到缓存，还有更多的发现。
准备好深入研究这些话题了吗？转到文档 ：:doc:`首页 </index>` ，选择您想要了解的任何指南。

.. _`Monolog`: https://github.com/Seldaek/monolog
