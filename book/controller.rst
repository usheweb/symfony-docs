.. index::
   single: Controller

Controller
==========

控制器用于为HTTP请求创建并且返回HTTP响应信息(一个Symfony ``Response`` 对象)。
响应内容可以是一个HTML页面，一个XML文档，一个序列化的JSON数组，一张图片，
一个url重定向，一个404错误页面或者其它内容。控制器可以包含任意逻辑，让 *你的
应用* 程渲染出你想要的页面。

来看一个简单的Symfony控制器动作.
在页面重输出一个经典的 ``Hello world!``::

    use Symfony\Component\HttpFoundation\Response;

    public function helloAction()
    {
        return new Response('Hello world!');
    }

所有控制器的共同目标是: 创建并返回一个 ``Response``
对象. 在http请求过程中, 可能从request中读取信息, 加载数据库资源, 发送email, 或者在用户的session中设置信息.
任何情况下, 控制器都会返回一个 ``Response`` 对象给客户端.

There's no magic and no other requirements to worry about! 下面是一些常见的例子:

* *控制器A* 准备一个 ``Response`` 对象 作为站点的主页内容.

* *控制器B* 通过读取请求参数从数据库中加载日志并且创建 ``Response`` 对象显示这篇日志. 
如果请求的数据在数据库中没有找到, 将会返回一个带404状态码的 ``Response`` 对象.

* *控制器 C* 处理一提交的表单. 从请求中读取表单信息, 为你将电子邮箱等联系信息保存到数据库中. 
最后, 通过一个带浏览器地址跳转的 ``Response`` 对象到 "thank you" 页面.

.. index::
   single: Controller; Request-controller-response lifecycle

请求, 控制器, 响应的生命周期
----------------------------------------

Symfony 项目中的每一个请求处理都有着相同的简单生命周期.
框架负责所有重复的东西: 你只需要写一些自己的代码在控制器函数中:

#. 每一个请求都交给一个前端控制器文件来加载应用程序 (e.g. ``app.php`` 或者 ``app_dev.php``);

#. 路由从请求中读取信息 (e.g. the URI), 找到匹配的路由信息, 在路由中读取控制参数;

#. 被路由匹配的控制器中的代码将会被执行并且返回 ``Response`` 对象;

#. ``Response`` 对象把HTTP响应头和响应内容返回给客户端.

通过创建一个控制器和(#3) 制作一个路由URL连接到控制器 (#2)就可轻松的获得一个页面.

.. note::

    虽然命名类似, 前端控制器和本章节讨论的控制器是不一样的. 前端控制器是一个短的PHP文件,
    放在您的web根目录并且所有请求都直接通过它. 通常一个应用程序有一个生产环境 (e.g. ``app.php``) 
    和一个开发环境的前端控制器(e.g. ``app_dev.php``). 不需要为编辑应用程序中的前端控制器而担心.

.. index::
   single: Controller; Simple example

简单的控制器
-------------------

控制器可以是任意的PHP可执行方法 (函数, 对象方法,或者闭包), 控制器通常使用类文件中的方法.
控制器也被称为 *actions*.

.. code-block:: php

    // src/AppBundle/Controller/HelloController.php
    namespace AppBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

.. tip::

    这个控制器是一个 ``indexAction`` 方法, 放在控制器类中(``HelloController``). 
    不要为命而困惑: 一个控制器类把多个控制器方法集合在一起. 
    通常一个控制器类有多个这样的action (e.g. ``updateAction``, ``deleteAction``,
    etc).

这是一个相当简单的控制器:

* *第4行*: Symfony 的控制器类文件采用了PHP的命名空间功能. 
  use关键字加载了一个控制器用到的Response类.

* *第6行*: 类名由控制器名称(i.e. ``Hello``)和单词Controller组合而成. 这是一个约定,
为控制器提供一致性,并允许他们被引用,只有第一部分的名字(i.e. ``Hello``) 在路由中配置.

* *地8行*: 每一个控制器都包含一个后缀 ``Action``和路由配置对应的名字组合 (``index``).
  下一节中, 将会学习通过路由映射URL到action.学习如何把路由中的占位符 (``{name}``) 
  变成action 方法中的参数 (``$name``).

* *第10行*: 控制器返回一个``Response`` 对象.

.. index::
   single: Controller; Routes and controllers

映射URL到Controller
-----------------------------

新的控制器返回一个简单的HTML页面. 在您的浏览器中查看此页面, 你需要创建一个路由, 
把特定的URL映射到控制器:

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Controller/HelloController.php
        namespace AppBundle\Controller;

        use Symfony\Component\HttpFoundation\Response;
        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class HelloController
        {
            /**
             * @Route("/hello/{name}", name="hello")
             */
            public function indexAction($name)
            {
                return new Response('<html><body>Hello '.$name.'!</body></html>');
            }
        }

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            path:      /hello/{name}
            # uses a special syntax to point to the controller - see note below
            defaults:  { _controller: AppBundle:Hello:index }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" path="/hello/{name}">
                <!-- uses a special syntax to point to the controller - see note below -->
                <default key="_controller">AppBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\Route;
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            // uses a special syntax to point to the controller - see note below
            '_controller' => 'AppBundle:Hello:index',
        )));

        return $collection;

现在访问 ``/hello/ryan`` (e.g. ``http://localhost:8000/hello/ryan``
if you're using the :doc:`built-in web server </cookbook/web_server/built_in>`)
Symfony 将会执行 ``HelloController::indexAction()`` 控制器，
``ryan`` 被放到变量 ``$name`` 中. 通过一个简单的控制器方法并且路由联系起来就可创建一个页面。

Simple, right?

.. sidebar:: The AppBundle:Hello:index controller syntax

    If you use the YML or XML formats, you'll refer to the controller using
    a special shortcut syntax: ``AppBundle:Hello:index``. For more details
    on the controller format, see :ref:`controller-string-syntax`.

.. seealso::

    You can learn much more about the routing system in the
    :doc:`Routing chapter </book/routing>`.

.. index::
   single: Controller; Controller arguments

.. _route-parameters-controller-arguments:

Route Parameters as Controller Arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You already know that the route points to the
``HelloController::indexAction()`` method that lives inside AppBundle. What's
more interesting is the argument that is passed to that method::

    // src/AppBundle/Controller/HelloController.php
    // ...
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

    /**
     * @Route("/hello/{name}", name="hello")
     */
    public function indexAction($name)
    {
        // ...
    }

The controller has a single argument, ``$name``, which corresponds to the
``{name}`` parameter from the matched route (``ryan`` if you go to ``/hello/ryan``).
When executing your controller, Symfony matches each argument with a parameter
from the route. So the value for ``{name}`` is passed to ``$name``.

Take the following more-interesting example:

.. configuration-block::

    .. code-block:: php-annotations

        // src/AppBundle/Controller/HelloController.php
        // ...

        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class HelloController
        {
            /**
             * @Route("/hello/{firstName}/{lastName}", name="hello")
             */
            public function indexAction($firstName, $lastName)
            {
                // ...
            }
        }

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            path:      /hello/{firstName}/{lastName}
            defaults:  { _controller: AppBundle:Hello:index }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" path="/hello/{firstName}/{lastName}">
                <default key="_controller">AppBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\Route;
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{firstName}/{lastName}', array(
            '_controller' => 'AppBundle:Hello:index',
        )));

        return $collection;

Now, the controller can have two arguments::

    public function indexAction($firstName, $lastName)
    {
        // ...
    }

Mapping route parameters to controller arguments is easy and flexible. Keep
the following guidelines in mind while you develop.

* **The order of the controller arguments does not matter**

  Symfony matches the parameter **names** from the route to the variable
  **names** of the controller. The arguments of the controller could be totally
  reordered and still work perfectly::

      public function indexAction($lastName, $firstName)
      {
          // ...
      }

* **Each required controller argument must match up with a routing parameter**

  The following would throw a ``RuntimeException`` because there is no ``foo``
  parameter defined in the route::

      public function indexAction($firstName, $lastName, $foo)
      {
          // ...
      }

  Making the argument optional, however, is perfectly ok. The following
  example would not throw an exception::

      public function indexAction($firstName, $lastName, $foo = 'bar')
      {
          // ...
      }

* **Not all routing parameters need to be arguments on your controller**

  If, for example, the ``lastName`` weren't important for your controller,
  you could omit it entirely::

      public function indexAction($firstName)
      {
          // ...
      }

.. tip::

    Every route also has a special ``_route`` parameter, which is equal to
    the name of the route that was matched (e.g. ``hello``). Though not usually
    useful, this is also available as a controller argument. You can also
    pass other variables from your route to your controller arguments. See
    :doc:`/cookbook/routing/extra_information`.

.. _book-controller-request-argument:

The ``Request`` as a Controller Argument
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

What if you need to read query parameters, grab a request header or get access
to an uploaded file? All of that information is stored in Symfony's ``Request``
object. To get it in your controller, just add it as an argument and
**type-hint it with the Request class**::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction($firstName, $lastName, Request $request)
    {
        $page = $request->query->get('page', 1);

        // ...
    }

.. seealso::

    Want to know more about getting information from the request? See
    :ref:`Access Request Information <component-http-foundation-request>`.

.. index::
   single: Controller; Base controller class

The Base Controller Class
-------------------------

For convenience, Symfony comes with an optional base ``Controller`` class.
If you extend it, you'll get access to a number of helper methods and all
of your service objects via the container (see :ref:`controller-accessing-services`).

Add the ``use`` statement atop the ``Controller`` class and then modify the
``HelloController`` to extend it::

    // src/AppBundle/Controller/HelloController.php
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        // ...
    }

This doesn't actually change anything about how your controller works: it
just gives you access to helper methods that the base controller class makes
available. These are just shortcuts to using core Symfony functionality that's
available to you with or without the use of the base ``Controller`` class.
A great way to see the core functionality in action is to look in the
`Controller class`_.

.. seealso::

    If you're curious about how a controller would work that did *not* extend
    this base class, check out :doc:`Controllers as Services </cookbook/controller/service>`.
    This is optional, but can give you more control over the exact objects/dependencies
    that are injected into your controller.

.. index::
   single: Controller; Redirecting

Redirecting
~~~~~~~~~~~

If you want to redirect the user to another page, use the ``redirectToRoute()`` method::

    public function indexAction()
    {
        return $this->redirectToRoute('homepage');

        // redirectToRoute is equivalent to using redirect() and generateUrl() together:
        // return $this->redirect($this->generateUrl('homepage'), 301);
    }

.. versionadded:: 2.6
    The ``redirectToRoute()`` method was added in Symfony 2.6. Previously (and still now), you
    could use ``redirect()`` and ``generateUrl()`` together for this (see the example above).

Or, if you want to redirect externally, just use ``redirect()`` and pass it the URL::

    public function indexAction()
    {
        return $this->redirect('http://symfony.com/doc');
    }

By default, the ``redirectToRoute()`` method performs a 302 (temporary) redirect. To
perform a 301 (permanent) redirect, modify the third argument::

    public function indexAction()
    {
        return $this->redirectToRoute('homepage', array(), 301);
    }

.. tip::

    The ``redirectToRoute()`` method is simply a shortcut that creates a
    ``Response`` object that specializes in redirecting the user. It's
    equivalent to::

        use Symfony\Component\HttpFoundation\RedirectResponse;

        public function indexAction()
        {
            return new RedirectResponse($this->generateUrl('homepage'));
        }

.. index::
   single: Controller; Rendering templates

.. _controller-rendering-templates:

Rendering Templates
~~~~~~~~~~~~~~~~~~~

If you're serving HTML, you'll want to render a template. The ``render()``
method renders a template **and** puts that content into a ``Response``
object for you::

    // renders app/Resources/views/hello/index.html.twig
    return $this->render('hello/index.html.twig', array('name' => $name));

You can also put templates in deeper sub-directories. Just try to avoid creating
unnecessarily deep structures::

    // renders app/Resources/views/hello/greetings/index.html.twig
    return $this->render('hello/greetings/index.html.twig', array(
        'name' => $name
    ));

The Symfony templating engine is explained in great detail in the
:doc:`Templating </book/templating>` chapter.

.. sidebar:: Referencing Templates that Live inside the Bundle

    You can also put templates in the ``Resources/views`` directory of a
    bundle and reference them with a
    ``BundleName:DirectoryName:FileName`` syntax. For example,
    ``AppBundle:Hello:index.html.twig`` would refer to the template located in
    ``src/AppBundle/Resources/views/Hello/index.html.twig``. See :ref:`template-referencing-in-bundle`.

.. index::
   single: Controller; Accessing services

.. _controller-accessing-services:

Accessing other Services
~~~~~~~~~~~~~~~~~~~~~~~~

Symfony comes packed with a lot of useful objects, called services. These
are used for rendering templates, sending emails, querying the database and
any other "work" you can think of. When you install a new bundle, it probably
brings in even *more* services.

When extending the base controller class, you can access any Symfony service
via the ``get()`` method. Here are several common services you might need::

    $templating = $this->get('templating');

    $router = $this->get('router');

    $mailer = $this->get('mailer');

What other services exist? To list all services, use the ``debug:container``
console command:

.. code-block:: bash

    $ php app/console debug:container

.. versionadded:: 2.6
    Prior to Symfony 2.6, this command was called ``container:debug``.

For more information, see the :doc:`/book/service_container` chapter.

.. index::
   single: Controller; Managing errors
   single: Controller; 404 pages

Managing Errors and 404 Pages
-----------------------------

When things are not found, you should play well with the HTTP protocol and
return a 404 response. To do this, you'll throw a special type of exception.
If you're extending the base controller class, do the following::

    public function indexAction()
    {
        // retrieve the object from database
        $product = ...;
        if (!$product) {
            throw $this->createNotFoundException('The product does not exist');
        }

        return $this->render(...);
    }

The ``createNotFoundException()`` method is just a shortcut to create a
special :class:`Symfony\\Component\\HttpKernel\\Exception\\NotFoundHttpException`
object, which ultimately triggers a 404 HTTP response inside Symfony.

Of course, you're free to throw any ``Exception`` class in your controller -
Symfony will automatically return a 500 HTTP response code.

.. code-block:: php

    throw new \Exception('Something went wrong!');

In every case, an error page is shown to the end user and a full debug
error page is shown to the developer (i.e. when you're using ``app_dev.php`` -
see :ref:`page-creation-environments`).

You'll want to customize the error page your user sees. To do that, see the
":doc:`/cookbook/controller/error_pages`" cookbook recipe.

.. index::
   single: Controller; The session
   single: Session

Managing the Session
--------------------

Symfony provides a nice session object that you can use to store information
about the user (be it a real person using a browser, a bot, or a web service)
between requests. By default, Symfony stores the attributes in a cookie
by using the native PHP sessions.

Storing and retrieving information from the session can be easily achieved
from any controller::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
        $session = $request->getSession();

        // store an attribute for reuse during a later user request
        $session->set('foo', 'bar');

        // get the attribute set by another controller in another request
        $foobar = $session->get('foobar');

        // use a default value if the attribute doesn't exist
        $filters = $session->get('filters', array());
    }

These attributes will remain on the user for the remainder of that user's
session.

.. index::
   single: Session; Flash messages

Flash Messages
~~~~~~~~~~~~~~

You can also store small messages that will be stored on the user's session
for exactly one additional request. This is useful when processing a form:
you want to redirect and have a special message shown on the *next* page.
These types of messages are called "flash" messages.

For example, imagine you're processing a form submit::

    use Symfony\Component\HttpFoundation\Request;

    public function updateAction(Request $request)
    {
        $form = $this->createForm(...);

        $form->handleRequest($request);

        if ($form->isValid()) {
            // do some sort of processing

            $this->addFlash(
                'notice',
                'Your changes were saved!'
            );

            // $this->addFlash is equivalent to $this->get('session')->getFlashBag()->add

            return $this->redirectToRoute(...);
        }

        return $this->render(...);
    }

After processing the request, the controller sets a ``notice`` flash message
in the session and then redirects. The name (``notice``) isn't significant -
it's just something you invent and reference next.

In the template of the next action, the following code could be used to render
the ``notice`` message:

.. configuration-block::

    .. code-block:: html+jinja

        {% for flashMessage in app.session.flashbag.get('notice') %}
            <div class="flash-notice">
                {{ flashMessage }}
            </div>
        {% endfor %}

    .. code-block:: html+php

        <?php foreach ($view['session']->getFlash('notice') as $message): ?>
            <div class="flash-notice">
                <?php echo "<div class='flash-error'>$message</div>" ?>
            </div>
        <?php endforeach ?>

By design, flash messages are meant to live for exactly one request (they're
"gone in a flash"). They're designed to be used across redirects exactly as
you've done in this example.

.. index::
   single: Controller; Response object

The Response Object
-------------------

The only requirement for a controller is to return a ``Response`` object. The
:class:`Symfony\\Component\\HttpFoundation\\Response` class is an abstraction
around the HTTP response: the text-based message filled with headers and
content that's sent back to the client::

    use Symfony\Component\HttpFoundation\Response;

    // create a simple Response with a 200 status code (the default)
    $response = new Response('Hello '.$name, Response::HTTP_OK);

    // create a JSON-response with a 200 status code
    $response = new Response(json_encode(array('name' => $name)));
    $response->headers->set('Content-Type', 'application/json');

The ``headers`` property is a :class:`Symfony\\Component\\HttpFoundation\\HeaderBag`
object and has some nice methods for getting and setting the headers. The
header names are normalized so that using ``Content-Type`` is equivalent to
``content-type`` or even ``content_type``.

There are also special classes to make certain kinds of responses easier:

* For JSON, there is :class:`Symfony\\Component\\HttpFoundation\\JsonResponse`.
  See :ref:`component-http-foundation-json-response`.

* For files, there is :class:`Symfony\\Component\\HttpFoundation\\BinaryFileResponse`.
  See :ref:`component-http-foundation-serving-files`.

* For streamed responses, there is :class:`Symfony\\Component\\HttpFoundation\\StreamedResponse`.
  See :ref:`streaming-response`.

.. seealso::

    Don't worry! There is a lot more information about the Response object
    in the component documentation. See :ref:`component-http-foundation-response`.

.. index::
   single: Controller; Request object

The Request Object
------------------

Besides the values of the routing placeholders, the controller also has access
to the ``Request`` object. The framework injects the ``Request`` object in the
controller if a variable is type-hinted with
:class:`Symfony\\Component\\HttpFoundation\\Request`::

    use Symfony\Component\HttpFoundation\Request;

    public function indexAction(Request $request)
    {
        $request->isXmlHttpRequest(); // is it an Ajax request?

        $request->getPreferredLanguage(array('en', 'fr'));

        $request->query->get('page'); // get a $_GET parameter

        $request->request->get('page'); // get a $_POST parameter
    }

Like the ``Response`` object, the request headers are stored in a ``HeaderBag``
object and are easily accessible.

.. seealso::

    Don't worry! There is a lot more information about the Request object
    in the component documentation. See :ref:`component-http-foundation-request`.

Creating Static Pages
---------------------

You can create a static page without even creating a controller (only a route
and template are needed).

See :doc:`/cookbook/templating/render_without_controller`.

.. index::
   single: Controller; Forwarding

Forwarding to Another Controller
--------------------------------

Though not very common, you can also forward to another controller internally
with the :method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::forward`
method. Instead of redirecting the user's browser, it makes an internal sub-request,
and calls the controller. The ``forward()`` method returns the ``Response``
object that's returned from *that* controller::

    public function indexAction($name)
    {
        $response = $this->forward('AppBundle:Something:fancy', array(
            'name'  => $name,
            'color' => 'green',
        ));

        // ... further modify the response or return it directly

        return $response;
    }

Notice that the ``forward()`` method uses a special string representation
of the controller (see :ref:`controller-string-syntax`). In this case, the
target controller function will be ``SomethingController::fancyAction()``
inside the AppBundle. The array passed to the method becomes the arguments on
the resulting controller. This same idea is used when embedding controllers
into templates (see :ref:`templating-embedding-controller`). The target
controller method would look something like this::

    public function fancyAction($name, $color)
    {
        // ... create and return a Response object
    }

Just like when creating a controller for a route, the order of the arguments of
``fancyAction`` doesn't matter. Symfony matches the index key names (e.g.
``name``) with the method argument names (e.g. ``$name``). If you change the
order of the arguments, Symfony will still pass the correct value to each
variable.

Final Thoughts
--------------

Whenever you create a page, you'll ultimately need to write some code that
contains the logic for that page. In Symfony, this is called a controller,
and it's a PHP function where you can do anything in order to return the
final ``Response`` object that will be returned to the user.

To make life easier, you can choose to extend a base ``Controller`` class,
which contains shortcut methods for many common controller tasks. For example,
since you don't want to put HTML code in your controller, you can use
the ``render()`` method to render and return the content from a template.

In other chapters, you'll see how the controller can be used to persist and
fetch objects from a database, process form submissions, handle caching and
more.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/controller/error_pages`
* :doc:`/cookbook/controller/service`

.. _`Controller class`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php
