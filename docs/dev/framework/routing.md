---
title: "Routing"
description: "Routing in Contao"
alias:
  - /framework/routing/
---


This section covers "routing" in Contao, e.g. how to implement your on routes in
the Managed Edition, Contao-specific route attributes and Page controllers.


## Implementing Custom Routes

Routing and controllers are a core concept of any Symfony application. Regardless
of whether you are using Contao within your own Symfony application or the Managed
Edition, the same principles apply. This subsection provides a short introduction
on how to implement your own routes and controllers in the Contao Managed Edition.
Have a look at the [Symfony routing documentation][SymfonyRouting] for the full
range of possibilities.

Routes can be defined either in XML/PHP/YAML files or via annotations. For simplicity
this guide will only show the latter. So to start off, we first need to tell Symfony
that our routes will be defined via annotations:

{{% expand "Defining routes in Contao 4.4" %}}
In Contao **4.4** you need to create a YAML file with the following definition:

```yaml
# app/config/routes.yaml
app.controller:
    resource: ../src/Controller
    type: annotation
```

This definition will not be automatically loaded however. In order to load this
YAML file within your Contao Managed Edition, you first need to create an
[Application-Specific Manager Plugin](/framework/manager-plugin/#the-application-specific-manager-plugin)
and implement the [`RoutingPluginInterface`](/framework/manager-plugin/#the-routingplugininterface):

```php
// src/ContaoManager/Plugin.php
namespace App\ContaoManager;

use Contao\ManagerPlugin\Routing\RoutingPluginInterface;
use Symfony\Component\Config\Loader\LoaderResolverInterface;
use Symfony\Component\HttpKernel\KernelInterface;

class Plugin implements RoutingPluginInterface
{
    public function getRouteCollection(LoaderResolverInterface $resolver, KernelInterface $kernel)
    {
        return $resolver
            ->resolve(__DIR__.'/../../app/config/routes.yaml')
            ->load(__DIR__.'/../../app/config/routes.yaml')
        ;
    }
}
```
{{% /expand %}}

{{% expand "Defining routes in Contao 4.9 and up" %}}
Starting with Contao **4.9** this is already defined by default in the Contao Managed
Edition, along with automatically registering any class within the `App\` namespace 
as services. Therefore you can skip this part and start implementing your controller
right away! If you still need to customize the routes configuration, the default 
would look like this:

```yaml
# config/routes.yaml
app.controller:
    resource: ../src/Controller
    type: annotation
```
{{% /expand %}}

This will tell Symfony that any controller defined under `src/Controller` within
your application (i.e. the `App\Controller\` namespace) will use PHP annotations
for defining routes.

Now we can go right ahead and create a simple controller and define its route via
the `Symfony\Component\Routing\Annotation\Route` annotation:

```php
// src/Controller/ExampleController.php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

/**
 * @Route("/example", name=ExampleController::class)
 */
class ExampleController
{
    public function __invoke(): Response
    {
        return new Response('Hello World!');
    }
}
```

This is the most bare bones controller you can build. In this case it is implemented
as an [invokable controller][InvokableController]. The route itself is registered
with two parameters:

* The actual route: `/example`. Our controller will be reachable under this path
  in the front end.
* The name of the route: in this case we chose the [_Fully Qualified Class Name_][FQCN]
  (FQCN) of the controller as the
  name of the route. This takes advantage of your IDE's auto-complete feature, if
  you want to reference the route for the `UrlGenerator` for example.

We can use the `debug:router` command to confirm its succesful registration:

```
$ vendor/bin/contao-console debug:router "App\Controller\ExampleController"
+--------------+---------------------------------------------------------+
| Property     | Value                                                   |
+--------------+---------------------------------------------------------+
| Route Name   | App\Controller\ExampleController                        |
| Path         | /example                                                |
| Path Regex   | #^/example$#sD                                          |
| Host         | ANY                                                     |
| Host Regex   |                                                         |
| Scheme       | ANY                                                     |
| Method       | ANY                                                     |
| Requirements | NO CUSTOM                                               |
| Class        | Symfony\Component\Routing\Route                         |
| Defaults     | _controller: App\Controller\ExampleController           |
| Options      | compiler_class: Symfony\Component\Routing\RouteCompiler |
+--------------+---------------------------------------------------------+
```

Accessing `https://example.com/example` in the front end should show the following:

```none
Hello World!
```


[SymfonyRouting]: https://symfony.com/doc/current/routing.html
[InvokableController]: https://symfony.com/doc/current/controller/service.html#invokable-controllers
[FQCN]: https://www.php-fig.org/psr/psr-4/
