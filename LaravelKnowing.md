## 核心概念

### 服务容器

Laravel 服务容器是一个用于管理类的依赖和执行依赖注入的强大工具。依赖注入这个花哨名词实质上是指：类的依赖通过构造函数，或者某些情况下通过 "setter" 方法 "注入" 到类中。

#### 简单绑定

在服务提供器中，你总是可以通过 $this->app 属性访问容器。我们可以通过容器的 bind 方法注册绑定，bind 方法的第一个参数为要绑定的类 / 接口名，第二个参数是一个返回类实例的 Closure：

```
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

#### 绑定一个单例

singleton 方法将类或接口绑定到只解析一次的容器中。一旦单例绑定被解析，相同的对象实例会在随后的调用中返回到容器中：

```
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```

#### 绑定实例

你也可以使用 instance 方法将现有对象实例绑定到容器中。给定的实例会始终在随后的调用中返回到容器中：

```
$api = new HelpSpot\API(new HttpClient);
$this->app->instance('HelpSpot\API', $api);
```

#### 绑定基本值

当你有一个类不仅需要接受一个注入类，还需要注入一个基本值（比如整数）。你可以使用上下文绑定来轻松注入你的类需要的任何值：

```
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```

#### 绑定接口到实现

服务容器有一个很强大的功能，就是支持绑定接口到给定的实现。例如，如果我们有个 EventPusher 接口 和一个 RedisEventPusher 实现。一旦我们写完了 EventPusher 接口的 RedisEventPusher 实现，我们就可以在服务容器中注册它，像这样：

```
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

#### 上下文绑定

有时你可能有两个类使用了相同的接口，但你希望各自注入不同的实现。例如，有两个控制器可能依赖了不同的 Illuminate\Contracts\Filesystem\Filesystem 契约 实现。Laravel 提供了一个简单的，优雅的接口来定义这个行为：

```
use Illuminate\Support\Facades\Storage;
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when(VideoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```
#### 解析实例

你可以使用 make 方法从容器中解析出类实例。make 方法接受一个你想要解析的类名或接口名：

```
$api = $this->app->make('HelpSpot\API');
```

#### 自动注入

另外，并且更重要的是，你可以简单地使用 "类型提示" 的方式在类的构造函数中注入那些需要容器解析的依赖项，包括 控制器 , 事件监听器 , 队列任务 , 中间件等。实际上，这才是大多数对象应该被容器解析的方式。


### 服务提供者

服务提供器是所有 Laravel 应用程序的引导中心。你的应用程序，以及 通过服务器引导的 Laravel 核心服务都是通过服务提供器引导。

但是，「引导」是什么意思呢？ 通常，我们的可以理解为注册，比如注册服务容器绑定，事件监听器，中间件，甚至是路由。服务提供器是配置应用程序的中心。

当你打开 Laravel 的 config/app.php 文件时，你会看到 providers 数组。数组中的内容是应用程序要加载的所有服务提供器的类。当然，其中有很多 「延迟」 提供器，他们并不会在每次请求的时候都加载，只有他们的服务实际被需要时才会加载

#### 编写服务提供器

所有的服务提供器都会继承 Illuminate\Support\ServiceProvider 类。 大多服务提供器都包含一个 register 和一个 boot 方法。在 register 方法中， 你只需要将事物绑定到服务容器。而不要尝试在 register 方法中注册任何监听器，路由，或者其他任何功能

使用 Artisan 命令界面，通过 make:provider 命令可以生成一个新的提供器

#### 注册方法

如上所述，在 register 方法中，你只需要将服务绑定到 服务容器中。而不要尝试在 register 方法中注册任何监听器，路由，或者其他任何功能。否则，你可能会意外地使用到尚未加载的服务提供器提供的服务

让我们来看一个基础的服务提供器。在任何服务提供器方法中，你总是通过 $app 属性来访问服务容器:

```
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

如果你的服务提供器注册了许多简单的绑定，你可能想用 bindings 和 singletons 属性替代手动注册每个容器绑定。当服务提供器被框架加载时，将自动检查这些属性并注册相应的绑定

#### Boot方法

如果我们要在服务提供者中注册一个 视图 composer 该怎么做？ 这就需要用到 boot 方法了。 该方法在所有服务提供者被注册以后才会被调用， 这就是说我们可以在其中访问框架已注册的所有其它服务：

```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * 启动所有的应用服务
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
}
```

#### 注册服务提供者

所有服务提供者都是通过配置文件 config/app.php 进行注册。该文件包含了一个列出所有服务提供者名字的 providers 数组，默认情况下，其中列出了所有核心服务提供者，这些服务提供者启动 Laravel 核心组件，比如邮件、队列、缓存等等。

要注册提供器，只需要将其添加到数组：

```
'providers' => [
    // 其他服务提供者
    App\Providers\ComposerServiceProvider::class,
],
```

#### 延迟服务提供者

如果你的服务提供者 仅 在 服务容器 中注册，可以选择延迟加载该绑定直到注册绑定的服务真的需要时再加载，延迟加载这样的一个提供者将会提升应用的性能，因为它不会在每次请求时都从文件系统加载。

Laravel 编译并保存延迟服务提供者提供的所有服务的列表，以及其服务提供者类的名称。因此，只有当你在尝试解析其中一项服务时，Laravel 才会加载服务提供者。

要延迟加载一个提供者，设置 defer 属性为 true 并设置一个 provides 方法。 provides 该方法返回该提供者注册的服务容器绑定：

```
<?php

namespace App\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    protected $defer = true;

    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 获取由提供者提供的服务
     *
     * @return array
     */
    public function provides()
    {
        return [Connection::class];
    }
}
```

### Facades

Facades 为应用的 服务容器 提供了一个「静态」 接口。Laravel 自带了很多 Facades，可以访问绝大部分功能。Laravel Facades 实际是服务容器中底层类的 「静态代理」 ，相对于传统静态方法，在使用时能够提供更加灵活、更加易于测试、更加优雅的语法

所有的 Laravel Facades 都定义在 Illuminate\Support\Facades 命名空间下。所以，我们可以轻松的使用 Facade :

```
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

#### 何时使用 Facades

Facades 有很多优点，它提供了简单，易记的语法，从而无需手动注入或配置长长的类名。此外，由于他们对 PHP 静态方法的独特调用，使得测试起来非常容易。

然而，在使用 Facades 时，有些地方需要特别注意。使用 Facades 时最主要的危险就是会引起类作用范围的膨胀。由于 Facades 使用起来非常简单并且不需要注入，就会使得我们不经意间在单个类中使用许多 Facades ，从而导致类变得越来越大。然而使用依赖注入的时候，使用的类越多，构造方法就会越长，在视觉上注意到这个类有些庞大了。因此在使用 Facades 的时候，要特别注意控制类的大小，让类的作用范围保持短小。

#### Facades 工作原理

在 Laravel 应用中，Facade 就是一个可以从容器访问对象的类。其中核心的部件就是 Facade 类。不管是 Laravel 自带的 Facades，还是自定义的 Facades，都继承自 Illuminate\Support\Facades\Facade 类。

Facade 基类使用了__callStatic() 魔术方法，直到对象从容器中被解析出来后，才会进行调用。在下面的例子中，调用了 Laravel 的缓存系统。通过浏览这段代码，可以假定在 Cache 类中调用了静态方法 get：

```
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 显示给定用户的信息。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

注意在上面这段代码中，我们『导入』了 Cache Facade。这个 Facade 作为访问 Illuminate\Contracts\Cache\Factory 接口底层实现的代理。我们使用 Facade 进行的任何调用都将传递给 Laravel 缓存服务的底层实例。

如果我们看一下 Illuminate\Support\Facades\Cache 这个类，你会发现类中根本没有 get 这个静态方法：

```
class Cache extends Facade
{
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

CacheFacade 继承了 Facade 类，并且定义了 getFacadeAccessor() 方法。这个方法的作用是返回服务容器绑定的名称。当用户调用 CacheFacade 中的任何静态方法时，Laravel 会从 服务容器 中解析 cache 绑定以及该对象运行所请求的方法（在这个例子中就是 get 方法）。

### Contracts

Laravel 的契约是指框架提供的一系列定义核心服务的接口。例如 Illuminate\Contracts\Queue\Queue 契约定义了队列任务需要实现的方法，而 Illuminate\Contracts\Mail\Mailer 契约定义了发送邮件所需要实现的方法。

每一个契约都有框架提供相应的实现。例如 Laravel 为队列提供了多个驱动的实现，邮件则由 SwiftMailer 驱动实现。

所有的契约都有其 对应的 GitHub 仓库。这为所有可用的契约提供了一个快速入门的指南，同时也可以单独作为低耦合的扩展包被开发者使用。

#### 契约 Vs. Facades

Laravel 的 facades 和辅助函数都提供了一种使用 Laravel 服务的简单方法，既不需类型提示，也不需要从服务容器中解析契约。大多情况下，每一个 Facades 都有一个等效的契约。

在 Facades 中，你不需要在构造函数中做类型提示，但是契约需要你在类中明确的定义依赖。一些开发者倾向于使用契约这种明确定义依赖的方式，而其他开发者则更喜欢 Facades 带来的便捷。

#### 何时使用契约

综上所述，使用契约还是 Facades 很大程度上取决于你个人或者团队的喜好。两者都可以使创建强大的、测试友好的 Laravel 应用程序。只要你保持类的职责单一，你会发现使用契约还是 Facades，其实并没有什么实质性的区别。

### mixin

简单来说，laravel的macro就是可以添加到laravel的class里面的静态方法。

## 核心代码

#### public/index.php

```
<?php
define('LARAVEL_START', microtime(true));

require __DIR__.'/../vendor/autoload.php';

$app = require_once __DIR__.'/../bootstrap/app.php';

$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);

$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);

$response->send();

$kernel->terminate($request, $response);
```

#### bootstrap/app.php

```
<?php
$app = new Illuminate\Foundation\Application(
    $_ENV['APP_BASE_PATH'] ?? dirname(__DIR__)
);

$app->singleton(
    Illuminate\Contracts\Http\Kernel::class,
    App\Http\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Console\Kernel::class,
    App\Console\Kernel::class
);

$app->singleton(
    Illuminate\Contracts\Debug\ExceptionHandler::class,
    App\Exceptions\Handler::class
);

return $app;
```

### Illuminate\Foundation\Application.php

Key是数组中的类在容器中的IdName，数组中一般是某个类以及它继承的接口，在容器中存储用的是同一个名字，生成的则是同一个对象。

```
<?php
class Application extends Container implements ApplicationContract, CachesConfiguration, CachesRoutes, HttpKernelInterface
{
    //load方法会调用$this->register，然后执行每个Providers->register();
    public function registerConfiguredProviders()
    {
        $providers = Collection::make($this->config['app.providers'])
                        ->partition(function ($provider) {
                            return strpos($provider, 'Illuminate\\') === 0;
                        });

        $providers->splice(1, 0, [$this->make(PackageManifest::class)->providers()]);

        (new ProviderRepository($this, new Filesystem, $this->getCachedServicesPath()))
                    ->load($providers->collapse()->toArray());
    }
    
    public function register($provider, $force = false)
    {
        if (($registered = $this->getProvider($provider)) && ! $force) {
            return $registered;
        }

        if (is_string($provider)) {
            $provider = $this->resolveProvider($provider);
        }

        $provider->register();

        if (property_exists($provider, 'bindings')) {
            foreach ($provider->bindings as $key => $value) {
                $this->bind($key, $value);
            }
        }

        if (property_exists($provider, 'singletons')) {
            foreach ($provider->singletons as $key => $value) {
                $this->singleton($key, $value);
            }
        }

        $this->markAsRegistered($provider);

        if ($this->isBooted()) {
            $this->bootProvider($provider);
        }

        return $provider;
    }
    
    public function bootstrapWith(array $bootstrappers)
    {
        $this->hasBeenBootstrapped = true;
    
        foreach ($bootstrappers as $bootstrapper) {
            $this['events']->dispatch('bootstrapping: '.$bootstrapper, [$this]);
    
            $this->make($bootstrapper)->bootstrap($this);
    
            $this['events']->dispatch('bootstrapped: '.$bootstrapper, [$this]);
        }
    }
        
    public function registerCoreContainerAliases()
    {
        foreach ([
            'app'                  => [self::class, \Illuminate\Contracts\Container\Container::class, \Illuminate\Contracts\Foundation\Application::class, \Psr\Container\ContainerInterface::class],
            'auth'                 => [\Illuminate\Auth\AuthManager::class, \Illuminate\Contracts\Auth\Factory::class],
            'auth.driver'          => [\Illuminate\Contracts\Auth\Guard::class],
            'blade.compiler'       => [\Illuminate\View\Compilers\BladeCompiler::class],
            'cache'                => [\Illuminate\Cache\CacheManager::class, \Illuminate\Contracts\Cache\Factory::class],
            'cache.store'          => [\Illuminate\Cache\Repository::class, \Illuminate\Contracts\Cache\Repository::class, \Psr\SimpleCache\CacheInterface::class],
            'cache.psr6'           => [\Symfony\Component\Cache\Adapter\Psr16Adapter::class, \Symfony\Component\Cache\Adapter\AdapterInterface::class, \Psr\Cache\CacheItemPoolInterface::class],
            'config'               => [\Illuminate\Config\Repository::class, \Illuminate\Contracts\Config\Repository::class],
            'cookie'               => [\Illuminate\Cookie\CookieJar::class, 
            'log'                  => [\Illuminate\Log\LogManager::class, \Psr\Log\LoggerInterface::class],
            'mail.manager'         => [\Illuminate\Mail\MailManager::class, \Illuminate\Contracts\Mail\Factory::class],
            'mailer'               => [\Illuminate\Mail\Mailer::class, \Illuminate\Contracts\Mail\Mailer::class, \Illuminate\Contracts\Mail\MailQueue::class],
            'auth.password'        => [\Illuminate\Auth\Passwords\PasswordBrokerManager::class, \Illuminate\Contracts\Auth\PasswordBrokerFactory::class],
            'auth.password.broker' => [\Illuminate\Auth\Passwords\PasswordBroker::class, \Illuminate\Contracts\Auth\PasswordBroker::class],
            'queue'                => [\Illuminate\Queue\QueueManager::class, \Illuminate\Contracts\Queue\Factory::class, \Illuminate\Contracts\Queue\Monitor::class],
            'queue.connection'     => [\Illuminate\Contracts\Queue\Queue::class],
            'queue.failer'         => [\Illuminate\Queue\Failed\FailedJobProviderInterface::class],
            'redirect'             => [\Illuminate\Routing\Redirector::class],
            'redis'                => [\Illuminate\Redis\RedisManager::class, 
            ......
        ] as $key => $aliases) {
            foreach ($aliases as $alias) {
                $this->alias($key, $alias);
            }
        }
    }
}
```

#### Illuminate\Foundation\Http\Kernel.php

```
<?php
class Kernel implements KernelContract
{
    protected $bootstrappers = [
        \Illuminate\Foundation\Bootstrap\LoadEnvironmentVariables::class, //加载环境变量
        \Illuminate\Foundation\Bootstrap\LoadConfiguration::class, //加载配置文件
        \Illuminate\Foundation\Bootstrap\HandleExceptions::class, //设置异常处理
        \Illuminate\Foundation\Bootstrap\RegisterFacades::class, //注册配置中的app.aliases门面类的简写，增加spl_autoload_register自动加载简写类
        \Illuminate\Foundation\Bootstrap\RegisterProviders::class, //调用$app->registerConfiguredProviders，遍历执行Providers->register()
        \Illuminate\Foundation\Bootstrap\BootProviders::class, //调用$app->boot()，遍历Providers如果存在boot()就执行
    ];
        
    public function handle($request)
    {
        try {
            $request->enableHttpMethodParameterOverride();
    
            $response = $this->sendRequestThroughRouter($request);
        } catch (Throwable $e) {
            $this->reportException($e);
    
            $response = $this->renderException($request, $e);
        }
    
        $this->app['events']->dispatch(
            new RequestHandled($request, $response)
        );
    
        return $response;
    }
        
    protected function sendRequestThroughRouter($request)
    {
        $this->app->instance('request', $request);
    
        Facade::clearResolvedInstance('request');
    
        $this->bootstrap();
    
        //通过array_reduce函数实现管道，先执行中间件的handle，然后再解析路由
        //执行Controller也类型，先寻找Controller的Middleware，然后管道执行中间件和Action
        return (new Pipeline($this->app))
                    ->send($request)
                    ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                    ->then($this->dispatchToRouter());
    }
    
    public function bootstrap()
    {
        if (! $this->app->hasBeenBootstrapped()) {
            $this->app->bootstrapWith($this->bootstrappers());
        }
    }
}
```

#### Illuminate\Foundation\Bootstrap\RegisterFacades.php

```
class RegisterFacades
{
    public function bootstrap(Application $app)
    {
        Facade::clearResolvedInstances();

        Facade::setFacadeApplication($app);

        AliasLoader::getInstance(array_merge(
            $app->make('config')->get('app.aliases', []),
            $app->make(PackageManifest::class)->aliases()
        ))->register();
    }
}
```

#### Illuminate\Cache\CacheServiceProvider.php

Kernel.bootstrappers.RegisterProviders执行$app->registerConfiguredProviders()获取config.app.providers遍历执行register()，这是其中的一个Provider，绑定了多个抽象的实现。

```
class CacheServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register()
    {
        $this->app->singleton('cache', function ($app) {
            return new CacheManager($app);
        });

        $this->app->singleton('cache.store', function ($app) {
            return $app['cache']->driver();
        });

        $this->app->singleton('cache.psr6', function ($app) {
            return new Psr16Adapter($app['cache.store']);
        });

        $this->app->singleton('memcached.connector', function () {
            return new MemcachedConnector;
        });
    }

    public function provides()
    {
        return [
            'cache', 'cache.store', 'cache.psr6', 'memcached.connector',
        ];
    }
}
```