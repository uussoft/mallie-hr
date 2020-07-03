.. index::
   single: Security

安全（Security）
========

Symfony的安全系统非常强大，但设置起来也会让人感到困惑。别担心！在本文中，您将学习如何逐步设置应用程序的安全系统：

#. :ref:`如何安装安全支持 <security-installation>`;

#. :ref:`创建您的用户类 <create-user-class>`;

#. :ref:`认证和防火墙 <security-yaml-firewalls>`;

#. :ref:`拒绝访问您的应用程序（授权） <security-authorization>`;

#. :ref:`获取当前的User对象 <retrieving-the-user-object>`.

之后讨论了其他一些重要主题。

.. _security-installation:

1) 安装
---------------

在使用  :ref:`Symfony Flex <symfony-flex>` 的应用程序中，在使用安全功能之前运行此命令以安装该功能：

.. code-block:: terminal

    $ composer require symfony/security-bundle

.. _initial-security-yml-setup-authentication:
.. _initial-security-yaml-setup-authentication:
.. _create-user-class:

2a) 创建用户类
--------------------------

无论您 *如何* 进行身份验证（例如，登录表单或API令​​牌）或将用户数据存储在 *何处* （数据库，单点登录），
下一步始终是相同的：创建 ``User`` 类。最简单的方法是使用  `MakerBundle`_。

假设您要使用Doctrine将用户数据存储在数据库中：

.. code-block:: terminal

    $ php bin/console make:user

    The name of the security user class (e.g. User) [User]:
    > User

    Do you want to store user data in the database (via Doctrine)? (yes/no) [yes]:
    > yes

    Enter a property name that will be the unique "display" name for the user (e.g.
    email, username, uuid [email]
    > email

    Does this app need to hash/check user passwords? (yes/no) [yes]:
    > yes

    created: src/Entity/User.php
    created: src/Repository/UserRepository.php
    updated: src/Entity/User.php
    updated: config/packages/security.yaml

执行该命令会提出几个问题，以便它能够准确地生成你需要的内容。最重要的是 ``User.php`` 文件。 ``User`` 类唯一的规则就是它 *必须* 实现的：类  :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`。随意添加 *任何* 您需要的其他字段或逻辑。如果您的 ``User`` 类是一个实体（如本例中所示），您可以使用： :ref:`make:entity <doctrine-add-more-fields>` 命令添加更多字段。另外，请确保为新实体进行并运行迁移：

.. code-block:: terminal

    $ php bin/console make:migration
    $ php bin/console doctrine:migrations:migrate

.. _security-user-providers:
.. _where-do-users-come-from-user-providers:

2b) 用户提供者
-----------------------

除了 ``User`` 类之外，您还需要一个用户提供程序  ``User provider`` 类：它可以帮助完成一些其他事情，
例如从会话中重新加载用户数据以及一些可选功能，例如 :doc:`记住我 </security/remember_me>` 和 :doc:`模拟用户 </security/impersonating_user>`。

然而幸运的是，  ``make:user`` 命令已经在 ``security.yaml`` 配置文件中为您配置了 ``providers`` 键 。

如果您的 ``User`` 是一个实体，则无需执行其他任何操作。但是，如果您的 ``User`` 类 *不是* 一个实体，
那么 ``make:user`` 命令还会生成一个 ``UserProvider`` 您需要完成的类。在此处了解有关用户提供者的更多信息： :doc:`用户提供者（User Providers） </security/user_provider>` 。

.. _security-encoding-user-password:
.. _encoding-the-user-s-password:

2c) 密码编码
----------------------

并非所有的应用程序都有需要密码的 “用户”。 *如果* 您的用户有密码，则可以在 ``security.yaml`` 配置文件中
控制这些密码的编码方式。 ``make:user`` 命令将为您预先配置了：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            # ...

            encoders:
                # use your user class name here
                App\Entity\User:
                    # Use native password encoder
                    # This value auto-selects the best possible hashing algorithm
                    # (i.e. Sodium when available).
                    algorithm: auto

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <!-- ... -->

                <encoder class="App\Entity\User"
                    algorithm="auto"
                    cost="12"/>

                <!-- ... -->
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            // ...

            'encoders' => [
                'App\Entity\User' => [
                    'algorithm' => 'auto',
                    'cost' => 12,
                ]
            ],

            // ...
        ]);

现在，Symfony知道了 *如何* 对密码进行编码，可以使用 ``UserPasswordEncoderInterface`` 服务在将用户保存
到数据库之前执行此操作。

例如，通过 :ref:`DoctrineFixturesBundle <doctrine-fixtures>`，您可以创建虚拟数据库用户：

.. code-block:: terminal

    $ php bin/console make:fixtures

    The class name of the fixtures to create (e.g. AppFixtures):
    > UserFixtures

使用此服务对密码进行编码：

.. code-block:: diff

    // src/DataFixtures/UserFixtures.php

    + use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
    // ...

    class UserFixtures extends Fixture
    {
    +     private $passwordEncoder;

    +     public function __construct(UserPasswordEncoderInterface $passwordEncoder)
    +     {
    +         $this->passwordEncoder = $passwordEncoder;
    +     }

          public function load(ObjectManager $manager)
          {
              $user = new User();
              // ...

    +         $user->setPassword($this->passwordEncoder->encodePassword(
    +             $user,
    +             'the_new_password'
    +         ));

              // ...
          }
    }

您可以通过运行以下命令来手动编码密码：

.. code-block:: terminal

    $ php bin/console security:encode-password

.. _security-yaml-firewalls:
.. _security-firewalls:
.. _firewalls-authentication:

3a) 验证和防火墙
------------------------------

安全系统是在 ``config/packages/security.yaml`` 中配置。 最 *重要* 的部分是防火墙 ``firewalls``：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            firewalls:
                dev:
                    pattern: ^/(_(profiler|wdt)|css|images|js)/
                    security: false
                main:
                    anonymous: lazy

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="dev"
                    pattern="^/(_(profiler|wdt)|css|images|js)/"
                    security="false"/>

                <firewall name="main">
                    <anonymous/>
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            'firewalls' => [
                'dev' => [
                    'pattern'   => '^/(_(profiler|wdt)|css|images|js)/',
                    'security'  => false,
                ),
                'main' => [
                    'anonymous' => null,
                ],
            ],
        ]);

“防火墙”是应用程序的身份验证系统：其下面的配置定义了应用程序该如何对用户进行身份验证（例如，登录表单，API令牌等）。

每个请求上只有一个防火墙处于活动状态：Symfony使用 ``pattern`` 键查找第一个匹配项（您页可以按 :doc:`主机或其他方式进行匹配 </security/firewall_restriction>`）。
该 ``dev`` 防火墙实际上是假的防火墙：它只是确保您不会意外地阻止了Symfony的开发工具—这些工具位于URL下，类似于 ``/_profiler`` 和 ``/_wdt`` 。

所有 *真实* 的URL均由 ``main`` 防火墙处理（没有 ``pattern`` 键表示它匹配 *所有* 的URL）。
但这并不意味着每一个URL都需要验证。多亏了 ``anonymous`` 键，该防火墙可以匿名访问。

实际上，如果您现在转到主页，则可以访问，并且会看到您已被“认证”为 ``anon.``。不要被Authenticated旁边的“是”所欺骗。
防火墙已验证它不知道您的身份，因此您是匿名的：       

.. image:: /_images/security/anonymous_wdt.png
   :align: center

稍后将学习如何拒绝对某些URL或控制器的访问。

.. note::

       如果看不到工具栏，请使用以下命令安装 :doc:`profiler </profiler>` 探查器：

    .. code-block:: terminal

        $ composer require --dev symfony/profiler-pack


现在我们了解了防火墙，下一步就是为应用程序用户创建一种认证方式！

.. _security-form-login:

3b) 验证用户
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony中的身份验证一开始会感觉有点“神奇”。这是因为，您无需构建路由和控制器来处理登录，而是要激活 *身份验证提供程序* ：有些代码会在调用控制器之前自动运行。

Symfony具有几个 :doc:`内置的身份验证提供程序 </security/auth_providers>`。如果您的应用程序 *完全* 符合其中的一个，那就太好了！但是，在大多数情况下（包括登录表单）， *我们建议您构建一个Guard Authenticator* ：允许您可以控制身份验证过程中各个部分的类（请参阅下一节）。

.. tip::

       如果您的应用程序通过第三方服务（例如新浪微博，QQ或Google（社交登录））登录用户，请安装 `HWIOAuthBundle`_ 社区捆绑包。

Guard 身份验证器
....................

Guard身份验证器是一个可让您 *完全* 控制身份验证过程的类。构建身份验证器有 *很多* 不同的方法，下面是一些常见的用例：

* :doc:`如何建立登入表格</security/form_login_setup>`
* :doc:`带Guard的自定义身份验证系统（API令牌示例）</security/guard_authentication>`

有关身份验证器及其工作方式的最详细说明，请参阅 :doc:`带Guard的自定义身份验证系统（API令牌示例）</security/guard_authentication>`。

.. _`security-authorization`:
.. _denying-access-roles-and-other-authorization:

4) 拒绝访问，角色和其他授权
------------------------------------------------

用户现在可以使用您的登录表单登录到您的应用程序。很好！现在，您需要学习如何拒绝访问和使用User对象。这称为 **授权**，其任务是决定用户是否可以访问某些资源（URL，模型对象，方法调用等）。

授权过程有两个不同方面：

#. 用户在登录时会接收一组特定的角色（例如 ``ROLE_ADMIN``）。

#. 添加一些代码，以便资源（例如URL，控制器）需要特定的“属性”（最常见是像 ``ROLE_ADMIN`` 这样一个角色）才能被访问。

角色
~~~~~

当用户登录时，Symfony会通过调用 ``User`` 对象的 ``getRoles()`` 方法以确定该用户具有哪些角色。在之前生成的 ``User`` 类中，角色是存储在数据库中的一个数组，并且每个用户始终被赋予至少一个角色： ``ROLE_USER`` ：
::

    // src/Entity/User.php
    // ...

    /**
     * @ORM\Column(type="json")
     */
    private $roles = [];

    public function getRoles(): array
    {
        $roles = $this->roles;
        // guarantee every user at least has ROLE_USER
        $roles[] = 'ROLE_USER';

        return array_unique($roles);
    }

这是一个很好的默认设置，但是你可以做 *任何* 你想决定用户应该扮演的角色。以下是一些指导原则：

* 每个角色 **都必须以**  ``ROLE_`` 开头（否则，事情将无法按预期的那样工作）

* 除了上述规则，角色只是一个字符串，您可以按照具体需求创造它们（例如 ``ROLE_PRODUCT_ADMIN``）。

接下来，您将使用这些角色授予对应用程序特定部分的访问权限。您还可以使用：:ref:`角色层次结构 <security-role-hierarchy>` 其中某些角色会自动为您提供其他角色。

.. _security-role-authorization:

添加代码以拒绝访问
~~~~~~~~~~~~~~~~~~~~~~~

有 *两种* 方法可以拒绝访问某些内容：

#. security.yaml中的 :ref:`access_control <security-authorization-access-control>` 允许您开启保护URL模式（例如 ``/admin/*``）。很简单，但是不太灵活；

#. :ref:`在您的控制器（或其他代码中）中 <security-securing-controller>`.

.. _security-authorization-access-control:

保护URL模式（access_control）
......................................

保护应用程序安全部分的最基本方法是在 ``security.yaml`` 中保护整个URL模式。例如，要求以 ``/admin`` 开头的所有url都使用 ``ROLE_ADMIN`` ，您可以在 ``security.yaml`` 中添加如下配置代码：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            # ...

            firewalls:
                # ...
                main:
                    # ...

            access_control:
                # require ROLE_ADMIN for /admin*
                - { path: '^/admin', roles: ROLE_ADMIN }

                # or require ROLE_ADMIN or IS_AUTHENTICATED_FULLY for /admin*
                - { path: '^/admin', roles: [IS_AUTHENTICATED_FULLY, ROLE_ADMIN] }

                # the 'path' value can be any valid regular expression
                # (this one will match URLs like /api/post/7298 and /api/comment/528491)
                - { path: ^/api/(post|comment)/\d+$, roles: ROLE_USER }

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <!-- ... -->

                <firewall name="main">
                    <!-- ... -->
                </firewall>

                <!-- require ROLE_ADMIN for /admin* -->
                <rule path="^/admin" role="ROLE_ADMIN"/>

                <!-- require ROLE_ADMIN or IS_AUTHENTICATED_FULLY for /admin* -->
                <rule path="^/admin">
                    <role>ROLE_ADMIN</role>
                    <role>IS_AUTHENTICATED_FULLY</role>
                </rule>

                <!-- the 'path' value can be any valid regular expression
                     (this one will match URLs like /api/post/7298 and /api/comment/528491) -->
                <rule path="^/api/(post|comment)/\d+$" role="ROLE_USER"/>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            // ...

            'firewalls' => [
                // ...
                'main' => [
                    // ...
                ],
            ],
            'access_control' => [
                // require ROLE_ADMIN for /admin*
                ['path' => '^/admin', 'roles' => 'ROLE_ADMIN'],

                // require ROLE_ADMIN or IS_AUTHENTICATED_FULLY for /admin*
                ['path' => '^/admin', 'roles' => ['ROLE_ADMIN', 'IS_AUTHENTICATED_FULLY']],

                // the 'path' value can be any valid regular expression
                // (this one will match URLs like /api/post/7298 and /api/comment/528491)
                ['path' => '^/api/(post|comment)/\d+$', 'roles' => 'ROLE_USER'],
            ],
        ]);

您可以根据需要定义任意数量的URL模式-每个模式都是正则表达式。 **但是** ，每个请求只匹配 **一个** 模式：Symfony将会从列表顶部开始，在找到第一个匹配项时停止：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            # ...

            access_control:
                # matches /admin/users/*
                - { path: '^/admin/users', roles: ROLE_SUPER_ADMIN }

                # matches /admin/* except for anything matching the above rule
                - { path: '^/admin', roles: ROLE_ADMIN }

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <!-- ... -->

                <rule path="^/admin/users" role="ROLE_SUPER_ADMIN"/>
                <rule path="^/admin" role="ROLE_ADMIN"/>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            // ...

            'access_control' => [
                ['path' => '^/admin/users', 'roles' => 'ROLE_SUPER_ADMIN'],
                ['path' => '^/admin', 'roles' => 'ROLE_ADMIN'],
            ],
        ]);

在路径前面加上 ``^`` 表示仅匹配以该路径 *开头* 的URL模式 。但是如果，路径  ``/admin`` （不含 ``^`` ）此时它将会匹配 ``/admin/foo`` 但同时也会匹配其他URL，如 ``/foo/admin`` 。

每个 ``access_control`` 还可以匹配IP地址，主机名和HTTP方法。它还可以用于将用户重定向到URL模式的 ``https`` 版本。请参阅 :doc:`安全性access_control如何工作 </security/access_control>` ？。

.. _security-securing-controller:

保护控制器和其他代码
...................................

您可以从控制器内部拒绝访问::

    // src/Controller/AdminController.php
    // ...

    public function adminDashboard()
    {
        $this->denyAccessUnlessGranted('ROLE_ADMIN');

        // or add an optional message - seen by developers
        $this->denyAccessUnlessGranted('ROLE_ADMIN', null, 'User tried to access a page without having ROLE_ADMIN');
    }

就这样！如果用户未授予访问权限，则抛出一个特殊的 :class:`Symfony\\Component\\Security\\Core\\Exception\\AccessDeniedException` 
异常，并且不在执行控制器中的其他代码。然后，将发生以下两种情况之一：

1) 如果用户尚未登录，将要求他们登录（例如，重定向到登录页面）。

2) 如果用户正在登录，但 **没有** ``ROLE_ADMIN`` 角色，则会显示 **403** 拒绝访问页面（可以 :ref:`自定义 <controller-error-pages-by-status-code>`）。

.. _security-securing-controller-annotations:

多亏了SensioFrameworkExtraBundle，您还可以使用注释保护控制器，代码如下：

.. code-block:: diff

    // src/Controller/AdminController.php
    // ...

    + use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

    + /**
    +  * Require ROLE_ADMIN for *every* controller method in this class.
    +  *
    +  * @IsGranted("ROLE_ADMIN")
    +  */
    class AdminController extends AbstractController
    {
    +     /**
    +      * Require ROLE_ADMIN for only this controller method.
    +      *
    +      * @IsGranted("ROLE_ADMIN")
    +      */
        public function adminDashboard()
        {
            // ...
        }
    }

有关更多信息，请参见 `FrameworkExtraBundle`_ 文档。

.. _security-template:

模板中的访问控制
...........................

如果要检查当前用户是否具有特定角色，您可以在Twig模板中任何地方使用内置的 ``is_granted()`` 辅助方法：

.. code-block:: html+twig

    {% if is_granted('ROLE_ADMIN') %}
        <a href="...">Delete</a>
    {% endif %}

保护其他服务
.......................

请参阅 :doc:`如何保护应用程序中的任何服务或方法 </security/securing_services>` 。

检查用户是否已经登录 (IS_AUTHENTICATED_FULLY)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果你 **只是** 想检查用户是否登录（你不关心用户角色），你有两个选择。首先，如果您给了 **每个** 用户 ``ROLE_USER`` 角色，
则只需检查该角色即可。否则，您可以使用特殊的“属性”来代替角色::

    // ...

    public function adminDashboard()
    {
        $this->denyAccessUnlessGranted('IS_AUTHENTICATED_FULLY');

        // ...
    }

您可以在任何需要角色检查的地方使用 ``IS_AUTHENTICATED_FULLY`` ：像 ``access_control`` 或 ``Twig`` 模板中。

实际上 ``IS_AUTHENTICATED_FULLY`` 不是一个角色，但是有点像角色，并且每个登录的用户都会有。实际上，它有3种特殊属性，例如：

* ``IS_AUTHENTICATED_REMEMBERED`` ：所有登录的用户都具有此功能，即使他们是因为 **记住我的cookie** 而登录的。即使您不使用 :doc:`记住我 </security/remember_me>` 的功能，同样也可以使用它来检查用户是否已登录。

* ``IS_AUTHENTICATED_FULLY``：这类似于 ``IS_AUTHENTICATED_REMEMBERED`` ，但效果更强。仅由于 **记住我的cookie** 功能而登录的用户将具有 ``IS_AUTHENTICATED_REMEMBERED`` 而不是 ``IS_AUTHENTICATED_FULLY``。

* ``IS_AUTHENTICATED_ANONYMOUSLY``： **所有** 用户（甚至是匿名用户）都具有此功能，在将URL列入 **白名单** 以确保访问权限时非常有用-有关某些详细信息，请参见 :doc:`安全性access_control如何工作？ </security/access_control>` 。

.. _security-secure-objects:

访问控制列表（ACL）：保护单个数据库对象
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

假设您正在设计一个博客，用户可以在其中评论您的帖子。您还希望用户能够编辑自己的评论，但不能编辑其他用户的注释。另外，作为管理员用户，您希望能够编辑 **所有** 评论。

:doc:`Voters </security/voters>` 允许您编写自己的 **任何** 业务逻辑（例如，用户可以编辑这篇文章，因为他们是创建人）来决定访问权限。这就是Symfony正式推荐 **Voters** 创建类似ACL的安全系统的原因。
如果您仍然喜欢使用传统ACL，请参阅 `Symfony ACL`_ 捆绑包。

.. _retrieving-the-user-object:

5a) 获取用户对象
----------------------------

身份认证之后，就可以通过 ``getUser()`` 快捷方式来访问当前用户的 ``User`` 对象::

    public function index()
    {
        // usually you'll want to make sure the user is authenticated first
        $this->denyAccessUnlessGranted('IS_AUTHENTICATED_FULLY');

        // returns your User object, or null if the user is not authenticated
        // use inline documentation to tell your editor your exact User class
        /** @var \App\Entity\User $user */
        $user = $this->getUser();

        // Call whatever methods you've added to your User class
        // For example, if you added a getFirstName() method, you can use that.
        return new Response('Well hi there '.$user->getFirstName());
    }

5b) 从服务中获取用户
------------------------------------

如果需要从服务中获取登录用户，请使用该 :class:`Symfony\\Component\\Security\\Core\\Security` 服务::

    // src/Service/ExampleService.php
    // ...

    use Symfony\Component\Security\Core\Security;

    class ExampleService
    {
        private $security;

        public function __construct(Security $security)
        {
            // Avoid calling getUser() in the constructor: auth may not
            // be complete yet. Instead, store the entire Security object.
            $this->security = $security;
        }

        public function someMethod()
        {
            // returns User object or null if not authenticated
            $user = $this->security->getUser();
        }
    }

在模板中获取用户
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在Twig模板中，用户对象可通过 ``app.user`` 变量获得，这要归功于 :ref:`Twig全局应用程序变量 <twig-app-variable>` ：

.. code-block:: html+twig

    {% if is_granted('IS_AUTHENTICATED_FULLY') %}
        <p>Email: {{ app.user.email }}</p>
    {% endif %}

.. _security-logging-out:

注销
-----------

要启用注销，请激活防火墙下的  ``logout`` 配置参数：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            # ...

            firewalls:
                main:
                    # ...
                    logout:
                        path: app_logout

                        # where to redirect after logout
                        # target: app_any_route

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <!-- ... -->

                <firewall name="secured_area">
                    <!-- ... -->
                    <logout path="app_logout"/>
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            // ...

            'firewalls' => [
                'secured_area' => [
                    // ...
                    'logout' => ['path' => 'app_logout'],
                ],
            ],
        ]);

接下来，您需要为此URL创建一个路由（而不是控制器）：

.. configuration-block::

    .. code-block:: php-annotations

        // src/Controller/SecurityController.php
        namespace App\Controller;

        use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
        use Symfony\Component\Routing\Annotation\Route;

        class SecurityController extends AbstractController
        {
            /**
             * @Route("/logout", name="app_logout", methods={"GET"})
             */
            public function logout()
            {
                // controller can be blank: it will never be executed!
                throw new \Exception('Don\'t forget to activate logout in security.yaml');
            }
        }

    .. code-block:: yaml

        # config/routes.yaml
        app_logout:
            path: /logout
            methods: GET

    .. code-block:: xml

        <!-- config/routes.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                https://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="app_logout" path="/logout" methods="GET"/>
        </routes>

    ..  code-block:: php

        // config/routes.php
        use Symfony\Component\Routing\Loader\Configurator\RoutingConfigurator;

        return function (RoutingConfigurator $routes) {
            $routes->add('logout', '/logout')
                ->methods(['GET'])
            ;
        };

就这样！通过将用户发送到该 ``app_logout`` 路由（即 ``/logout`` ），Symfony将取消对当前用户的身份验证并将其重定向。

.. tip::

       需要对注销后发生的事情进行更多控制吗？在 ``logout`` 下添加一个 ``success_handler`` 键，并将其指向到实现 :class:`Symfony\\Component\\Security\\Http\\Logout\\LogoutSuccessHandlerInterface` 接口的类的服务 **ID**

.. _security-role-hierarchy:

层次角色
------------------

您可以通过创建角色层次结构来定义角色继承规则，而不是为每个用户分配许多角色：

.. configuration-block::

    .. code-block:: yaml

        # config/packages/security.yaml
        security:
            # ...

            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

    .. code-block:: xml

        <!-- config/packages/security.xml -->
        <?xml version="1.0" encoding="UTF-8"?>
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                https://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <!-- ... -->

                <role id="ROLE_ADMIN">ROLE_USER</role>
                <role id="ROLE_SUPER_ADMIN">ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH</role>
            </config>
        </srv:container>

    .. code-block:: php

        // config/packages/security.php
        $container->loadFromExtension('security', [
            // ...

            'role_hierarchy' => [
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => [
                    'ROLE_ADMIN',
                    'ROLE_ALLOWED_TO_SWITCH',
                ],
            ],
        ]);

具有 ``ROLE_ADMIN`` 角色的用户也将具有 ``ROLE_USER`` 角色。
而具有 ``ROLE_SUPER_ADMIN`` 角色的用户将会自动拥有 ``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH`` 和 ``ROLE_USER`` 角色（从 ``ROLE_ADMIN`` 继承）。

要使角色层次结构正常工作，请勿尝试从 ``$user->getRoles()`` 手动调用。例如，在从 :ref:`主控制器 <the-base-controller-class-services>` 扩展的控制器中::

    // BAD - $user->getRoles() will not know about the role hierarchy
    $hasAccess = in_array('ROLE_ADMIN', $user->getRoles());

    // GOOD - use of the normal security methods
    $hasAccess = $this->isGranted('ROLE_ADMIN');
    $this->denyAccessUnlessGranted('ROLE_ADMIN');

.. note::

    ``role_hierarchy`` 值是静态的。例如，您不能将角色层次结构存储在数据库中。如果需要，请创建一个自定义 :doc:`security voter </security/voters>`，以便在数据库中查找用户角色。

常见问题
--------------------------

**我可以有多个防火墙吗？**
       可以的! 但这通常不是必需的。每个防火墙就像一个独立的安全系统。因此，除非您有非常不同的认证需求，否则一个防火墙通常可以正常工作。使用 :doc:`Guard 身份认证器 </security/guard_authentication>` ，您可以在同一防火墙下创建各种不同的允许身份的验证方法（例如，表单登录，API密钥身份验证和LDAP）都在同一防火墙下。

**我可以在防火墙之间共享身份验证吗？**
       是的，但只有某些配置。如果您使用多个防火墙，并且针对一个防火墙进行身份验证，则不会自动对任何其他防火墙进行身份验证。不同的防火墙就像不同的安全系统。为此，您必须为不同的防火墙明确指定相同的 :ref:`防火墙上下文 <reference-security-firewall-context>` 。但通常对于大多数应用程序而言，拥有一个主防火墙就足够了。

**在我的错误页面上安全性似乎不起作用**
       由于路由是在 **安全认证** 之前完成的，因此任何防火墙都不覆盖404个错误页。这意味着您无法检查安全性，甚至无法访问这些页面上的用户对象。有关详细信息，请参见： :doc:`自定义错误页面 </controller/error_pages>`。

**我的身份验证似乎不起作用：没有错误，但是我从未登录**
       有时，身份验证可能会成功，但是在重定向之后，由于从会话加载 ``用户`` 时出现问题，用户将被立即注销。要查看此问题，请在日志文件（ ``var/log/dev.log`` ）中查看日志消息。

**由于用户已更改，因此无法刷新令牌**
       如果看到此现象，可能有两个原因。首先，从会话加载您的用户可能存在问题。请参阅  :ref:`了解如何从会话刷新用户 <user_session_refresh>` 。其次，如果自上次刷新页面以来数据库中某些用户信息已更改，Symfony出于安全原因将立即注销该用户。

了解更多
----------

身份认证（识别/登录用户）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

       如何建立一个登录表单 <security/form_login_setup>
       如何建立一个JSON认证端点 <security/json_login_setup>
       带Guard的自定义身份认证系统（API令牌示例） <security/guard_authentication>
       如何迁移密码哈希 <security/password_migration>
       内置身份验证提供程序 <security/auth_providers>
       安全用户提供者 <security/user_provider>
       针对LDAP服务器进行身份验证 <security/ldap>
       如何添加“记住我”登录功能 <security/remember_me>
       如何模拟用户 <security/impersonating_user>
       如何创建和启用自定义用户检查器 <security/user_checkers>
       如何为每个用户使用不同的密码编码器算法 <security/named_encoders>
       如何使用多个Guard认证器 <security/multiple_guard_authenticators>
       如何将防火墙限制在请求范围内 <security/firewall_restriction>
       如何实施CSRF保护 <security/csrf>
       如何创建自定义身份验证提供程序 <security/custom_authentication_provider>

授权（拒绝访问）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

       如何使用Voters检查用户权限 <security/voters>
       如何保护应用程序中的任何服务或方法 <security/securing_services>
       安全access_control如何工作？ <security/access_control>
       如何创建自定义访问拒绝处理程序 <security/access_denied_handler>
       如何使用访问控制列表（ACL） <security/acl>
       如何对不同的URL强制使用HTTPS或HTTP <security/force_https>

.. _`FrameworkExtraBundle`: https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html
.. _`HWIOAuthBundle`: https://github.com/hwi/HWIOAuthBundle
.. _`Symfony ACL`: https://github.com/symfony/acl-bundle
.. _`Symfony Security screencast series`: https://symfonycasts.com/screencast/symfony-security
.. _`MakerBundle`: https://symfony.com/doc/current/bundles/SymfonyMakerBundle/index.html
