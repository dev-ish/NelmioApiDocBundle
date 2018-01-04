NelmioApiDocBundle
==================

The **NelmioApiDocBundle** bundle allows you to generate documentation in the
OpenAPI (Swagger) format and provides a sandbox to interactively browse the API documentation.

What's supported?
-----------------

This bundle supports _Symfony_ route requirements, PHP annotations,
[_Swagger-Php_](https://github.com/zircote/swagger-php) annotations,
[_FOSRestBundle_](https://github.com/FriendsOfSymfony/FOSRestBundle) annotations
and apps using [_Api-Platform_](https://github.com/api-platform/api-platform).

For models, it supports the Symfony serializer and the JMS serializer.

Migrate from 2.x to 3.0
-----------------------

[To migrate from 2.x to 3.0, just follow our guide.](https://github.com/nelmio/NelmioApiDocBundle/blob/master/UPGRADE-3.0.md)

Installation
------------

Open a command console, enter your project directory and execute the following command to download the latest version of this bundle:

.. code-block:: bash

    $ composer require nelmio/api-doc-bundle

.. note::

    If you're not using Flex, then add the bundle to your kernel::

        class AppKernel extends Kernel
        {
            public function registerBundles()
            {
                $bundles = [
                    // ...

                    new Nelmio\ApiDocBundle\NelmioApiDocBundle(),
                ];

                // ...
            }
        }

    To browse your documentation with Swagger UI, register the following route:

    .. code-block:: yaml

        # app/config/routing.yml
        app.swagger_ui:
            path: /api/doc
            methods: GET
            defaults: { _controller: nelmio_api_doc.controller.swagger_ui }

    If you also want to expose it in JSON, register this route:

    .. code-block:: yaml

        # app/config/routing.yml
        app.swagger:
            path: /api/doc.json
            methods: GET
            defaults: { _controller: nelmio_api_doc.controller.swagger }

    As you just installed the bundle, you'll likely see routes you don't want in
    your documentation such as `/_profiler/`. To fix this, you can filter the
    routes that are documented by configuring the bundle:

    .. code-block:: yaml

        # app/config/config.yml
        nelmio_api_doc:
            areas:
                path_patterns: # an array of regexps
                    - ^/api(?!/doc$)

How does this bundle work?
--------------------------

It generates you a swagger documentation from your symfony app thanks to
_Describers_. Each of these _Describers_ extract infos from various sources.
For instance, one extract data from SwaggerPHP annotations, one from your
routes, etc.

If you configured the ``app.swagger_ui`` route above, you can browse your
documentation at `http://example.org/api/doc`.

Using the bundle
----------------

You can configure global information in the bundle configuration ``documentation.info`` section (take a look at
[the Swagger specification](http://swagger.io/specification/) to know the fields
available):

.. code-block:: yaml

    nelmio_api_doc:
        documentation:
            info:
                title: My App
                description: This is an awesome app!
                version: 1.0.0

.. note::

    If you're using Flex, this config is there by default. Don't forget to adapt it to your app!

To document your routes, you can use the SwaggerPHP annotations and the
``Nelmio\ApiDocBundle\Annotation\Model`` annotation in your controllers::

    namespace AppBundle\Controller;

    use AppBundle\Entity\User;
    use AppBundle\Entity\Reward;
    use Nelmio\ApiDocBundle\Annotation\Model;
    use Swagger\Annotations as SWG;
    use Symfony\Component\Routing\Annotation\Route;

    class UserController
    {
        /*
         * @Route("/api/{user}/rewards", methods={"GET"})
         * @SWG\Response(
         *     response=200,
         *     description="Returns the rewards of an user",
         *     @SWG\Schema(
         *         type="array",
         *         @Model(type=Reward::class, groups={"full"})
         *     )
         * )
         * @SWG\Parameter(
         *     name="order",
         *     in="query",
         *     type="string",
         *     description="The field used to order rewards"
         * )
         * @SWG\Tag(name="rewards")
         */
        public function fetchUserRewardsAction(User $user)
        {
            // ...
        }
    }

Use models
----------

As shown in the example above, the bundle provides the ``@Model`` annotation.
When you use it, the bundle will deduce your model properties.

It has two options:

* ``type`` to specify your model's type::

    /**
     * @Model(type=User::class)
     */

* ``groups`` to specify the serialization groups used to (de)serialize your model::

    /**
     * @Model(type=User::class, groups={"non_sensitive_data"})
     */

.. warning::

    The ``@Model`` annotation acts like a ``@Schema`` annotation. If you nest it with a ``@Schema`` annotation, the bundle will consider that
    you're documenting an array of models.

    For instance, the following example::

        /**
         * @SWG\Response(
         *   response="200",
         *   description="Success",
         *   @SWG\Schema(@Model(type=User::class))
         * )
         */
        public function getUserAction()
        {
        }

    will produce:

    .. code-block:: yaml

        # ...
        responses:
            200:
                schema:
                    items: { $ref: '#/definitions/MyModel' }

    while you probably expected:

    .. code-block:: yaml

        # ...
        responses:
            200:
                schema: { $ref: '#/definitions/MyModel' }

    To obtain the output you expected, remove the ``@Schema`` annotation::

        /**
         * @SWG\Response(
         *   response="200",
         *   description="Success",
         *   @Model(type=MyModel::class)
         * )
         */
        public function myAction()
        {
        }

If you're not using the JMS Serializer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The [Symfony PropertyInfo component](https://symfony.com/doc/current/components/property_info.html)
is used to describe your models. It supports doctrine annotations, type hints,
and even PHP doc blocks as long as you required the
``phpdocumentor/reflection-docblock`` library. It does also support
serialization groups when using the Symfony serializer.

If you're using the JMS Serializer
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The metadata of the JMS serializer are used by default to describe your
models. Additional information is extracted from the PHP doc block comment,
but the property types must be specified in the JMS annotations.

In case you prefer using the [Symfony PropertyInfo component](https://symfony.com/doc/current/components/property_info.html) (you
won't be able to use JMS serialization groups), you can disable JMS serializer
support in your config:

.. code-block:: yaml

    nelmio_api_doc:
        models: { use_jms: false }

Learn more
----------

If you need more complex features, take a look at:

.. toctree::
    :maxdepth: 1

    areas
