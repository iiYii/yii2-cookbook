[URL 变量参数的数量](https://github.com/iiYii/yii2-cookbook/edit/gh-pages/book/urls-variable-number-of-parameters.md)
=======================================

There are many cases when you need to get variable number of parameters via URL.
For example one may want URLs such as `http://example.com/products/cars/sport` to lead to `ProductController::actionCategory`
where it's expected to get an array containing `cars` and `sport`.

做好准备
---------

首先，我们需要启用漂亮的 URL。在应用程序配置文件中添加以下:

```php
$config = [
    // ...
    'components' => [
        // ...
        'urlManager' => [
            'showScriptName' => false,
            'enablePrettyUrl' => true,
            'rules' => require 'urls.php',
        ],
    ]
```

Note that we're including separate file instead of listing rules directly. It is helpful when application grows large.

Now in `config/urls.php` add the following content:

```php
<?php
return [
    'products/<categories:.*>' => 'product/category',
];
```

Create `ProductController`:

```php
namespace app\controllers;

use yii\web\Controller;

class ProductController extends Controller
{
    public function actionCategory($categories)
    {
        $params = explode('/', $categories);
        print_r($categories);
    }
}
```

That's it. Now you can try `http://example.com/products/cars/sport`. What you'll get is

```
Array ( [0] => cars [1] => sport)
```
