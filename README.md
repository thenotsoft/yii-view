<p align="center">
    <a href="https://github.com/yiisoft" target="_blank">
        <img src="https://yiisoft.github.io/docs/images/yii_logo.svg" height="100px">
    </a>
    <h1 align="center">Yii View Extension</h1>
    <br>
</p>

[![Latest Stable Version](https://poser.pugx.org/yiisoft/yii-view/v/stable.png)](https://packagist.org/packages/yiisoft/yii-view)
[![Total Downloads](https://poser.pugx.org/yiisoft/yii-view/downloads.png)](https://packagist.org/packages/yiisoft/yii-view)
[![Build status](https://github.com/yiisoft/yii-view/workflows/build/badge.svg)](https://github.com/yiisoft/yii-view/actions?query=workflow%3Abuild)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/yiisoft/yii-view/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/yiisoft/yii-view/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/yiisoft/yii-view/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/yiisoft/yii-view/?branch=master)
[![Mutation testing badge](https://img.shields.io/endpoint?style=flat&url=https%3A%2F%2Fbadge-api.stryker-mutator.io%2Fgithub.com%2Fyiisoft%2Fyii-view%2Fmaster)](https://dashboard.stryker-mutator.io/reports/github.com/yiisoft/yii-view/master)
[![static analysis](https://github.com/yiisoft/yii-view/workflows/static%20analysis/badge.svg)](https://github.com/yiisoft/yii-view/actions?query=workflow%3A%22static+analysis%22)
[![type-coverage](https://shepherd.dev/github/yiisoft/yii-view/coverage.svg)](https://shepherd.dev/github/yiisoft/yii-view)

The package is an extension of the [Yii View Rendering Library](https://github.com/yiisoft/view/). It adds
WEB-specific functionality and compatibility with [PSR-7](https://www.php-fig.org/psr/psr-7/) interfaces.

## Requirements

- PHP 7.4 or higher.

## Installation

The package could be installed with composer:

```shell
composer require yiisoft/yii-view --prefer-dist
```

## General usage

There are two ways to render a view:

- Return an instance of the `Yiisoft\DataResponse\DataResponse` class with deferred rendering.
- Render immediately and return the rendered result as a string.

### Rendering result as a PSR-7 response

The `Yiisoft\DataResponse\DataResponse` class is an implementation of the `Psr\Http\Message\ResponseInterface`. For
more information about this class, see the [yiisoft/data-response](https://github.com/yiisoft/data-response) package.
You can get an instance of a response with deferred rendering as follows:

```php
/**
 * @var \Yiisoft\Aliases\Aliases $aliases
 * @var \Yiisoft\DataResponse\DataResponseFactoryInterface $dataResponseFactory
 * @var \Yiisoft\View\WebView $webView
 */

$viewRenderer = new \Yiisoft\Yii\View\ViewRenderer(
    $dataResponseFactory,
    $aliases,
    $webView,
    '/path/to/views', // Full path to the directory of view templates or its alias.
    'layouts/main', // Default is null, which means not to use a layout.
);

// Rendering a view with a layout.
$response = $viewRenderer->render('site/page', [
    'parameter-name' => 'parameter-value',
]);
```

The rendering will be performed directly when calling `getBody()` or `getData()` methods of the
`Yiisoft\DataResponse\DataResponse`. If a layout is set, but you need to render a view
without the layout, you can use an immutable setter `withLayout()`:

```php
$viewRenderer = $viewRenderer->withLayout(null);

// Rendering a view without a layout.
$response = $viewRenderer->render('site/page', [
    'parameter-name' => 'parameter-value',
]);
```

Or use `renderPartial()` method, which will call `withLayout(null)`:

```php
// Rendering a view without a layout.
$response = $viewRenderer->renderPartial('site/page', [
    'parameter-name' => 'parameter-value',
]);
```

### Rendering result as a string

To render immediately and return the rendering result as a string,
use `renderAsString()` and `renderPartialAsString()` methods:

```php
// Rendering a view with a layout.
$result = $viewRenderer->renderAsString('site/page', [
    'parameter-name' => 'parameter-value',
]);

// Rendering a view without a layout.
$result = $viewRenderer->renderPartialAsString('site/page', [
    'parameter-name' => 'parameter-value',
]);
```

### Change view templates path 

You can change view templates path in runtime as follows:

```php
$viewRenderer = $viewRenderer->withViewPath('/new/path/to/views');
```

You can specify full path to the views directory or its alias. For more information about path aliases,
see description of the [yiisoft/aliases](https://github.com/yiisoft/aliases) package.

### Use in the controller

If the view renderer is used in a controller, you can either specify controller name explicitly using
`withControllerName()` or determine name automatically by passing a controller instance to `withController()`.
In this case the name is determined as follows:

```
App\Controller\FooBar\BazController -> foo-bar/baz
App\Controllers\FooBar\BazController -> foo-bar/baz
Path\To\File\BlogController -> blog
```

With this approach, you do not need to specify the directory name each time when rendering a view template:

```php
use Psr\Http\Message\ResponseInterface;
use Yiisoft\Yii\View\ViewRenderer;

class SiteController
{
    private ViewRenderer $viewRenderer;

    public function __construct(ViewRenderer $viewRenderer)
    {
        // Specify the name of the controller:
        $this->viewRenderer = $viewRenderer->withControllerName('site');
        // or specify an instance of the controller:
        //$this->viewRenderer = $viewRenderer->withController($this);
    }

    public function index(): ResponseInterface
    {
        return $this->viewRenderer->render('index');
    }
    
    public function contact(): ResponseInterface
    {
        // Some actions.
        return $this->viewRenderer->render('contact', [
            'parameter-name' => 'parameter-value',
        ]);
    }
}
```

This is very convenient if there are many methods (actions) in the controller.

### Injection of additional data to the views

In addition to parameters passed directly when rendering the view template, you can set extra parameters that will be
available in all views. In order to do it you need a class implementing at least one of the injection interfaces:

```php
use Yiisoft\Yii\View\CommonParametersInjectionInterface;
use Yiisoft\Yii\View\LayoutParametersInjectionInterface;

final class MyParametersInjection implements
    CommonParametersInjectionInterface,
    LayoutParametersInjectionInterface
{
    // Pass both to view template and to layout
    public function getCommonParameters(): array
    {
        return [
            'common-parameter-name' => 'common-parameter-value',
        ];
    }
    
    // Pass only to layout
    public function getLayoutParameters(): array
    {
        return [
            'layout-parameter-name' => 'layout-parameter-value',
        ];
    }
}
```

Link tags and meta tags should be organized in the same way.

```php
use Yiisoft\Html\Html;
use Yiisoft\View\WebView;
use Yiisoft\Yii\View\LinkTagsInjectionInterface;
use Yiisoft\Yii\View\MetaTagsInjectionInterface;

final class MyTagsInjection implements
    LinkTagsInjectionInterface,
    MetaTagsInjectionInterface
{
    public function getLinkTags(): array
    {
        return [
            Html::link()->toCssFile('/main.css'),
            'favicon' => Html::link('/myicon.png', [
                'rel' => 'icon',
                'type' => 'image/png',
            ]),
            'themeCss' => [
                '__position' => WebView::POSITION_END,
                Html::link()->toCssFile('/theme.css'),
            ],
            'userCss' => [
                '__position' => WebView::POSITION_BEGIN,
                'rel' => 'stylesheet',
                'href' => '/user.css',
            ],
        ];
    }
    
    public function getMetaTags(): array
    {
        return [
            Html::meta()->name('http-equiv')->content('public'),
            'noindex' => Html::meta()->name('robots')->content('noindex'),
            [
                'name' => 'description',
                'content' => 'This website is about funny raccoons.',
            ],
            'keywords' => [
                'name' => 'keywords',
                'content' => 'yii,framework',
            ],
        ];
    }
}
```

You can pass instances of these classes as the sixth optional parameter to the constructor when
creating a view renderer, or use the `withInjections()` and `withAddedInjections` methods.

```php
$parameters = new MyParametersInjection();
$tags = new MyTagsInjection();

$viewRenderer = $viewRenderer->withInjections($parameters, $tags);
// Or append it:
$viewRenderer = $viewRenderer->withAddedInjections($parameters, $tags);
```

The parameters passed to `render()` method have more priority
and will overwrite the injected content parameters if their names match.

## Testing

### Unit testing

The package is tested with [PHPUnit](https://phpunit.de/). To run tests:

```shell
./vendor/bin/phpunit
```

### Mutation testing

The package tests are checked with [Infection](https://infection.github.io/) mutation framework with
[Infection Static Analysis Plugin](https://github.com/Roave/infection-static-analysis-plugin). To run it:

```shell
./vendor/bin/roave-infection-static-analysis-plugin
```

### Static analysis

The code is statically analyzed with [Psalm](https://psalm.dev/). To run static analysis:

```shell
./vendor/bin/psalm
```

## License

The Yii View Extension is free software. It is released under the terms of the BSD License.
Please see [`LICENSE`](./LICENSE.md) for more information.

Maintained by [Yii Software](https://www.yiiframework.com/).

## Support the project

[![Open Collective](https://img.shields.io/badge/Open%20Collective-sponsor-7eadf1?logo=open%20collective&logoColor=7eadf1&labelColor=555555)](https://opencollective.com/yiisoft)

## Follow updates

[![Official website](https://img.shields.io/badge/Powered_by-Yii_Framework-green.svg?style=flat)](https://www.yiiframework.com/)
[![Twitter](https://img.shields.io/badge/twitter-follow-1DA1F2?logo=twitter&logoColor=1DA1F2&labelColor=555555?style=flat)](https://twitter.com/yiiframework)
[![Telegram](https://img.shields.io/badge/telegram-join-1DA1F2?style=flat&logo=telegram)](https://t.me/yii3en)
[![Facebook](https://img.shields.io/badge/facebook-join-1DA1F2?style=flat&logo=facebook&logoColor=ffffff)](https://www.facebook.com/groups/yiitalk)
[![Slack](https://img.shields.io/badge/slack-join-1DA1F2?style=flat&logo=slack)](https://yiiframework.com/go/slack)
