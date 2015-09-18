使用不同的响应类型
=====================================

Web 和移动应用程序现在不仅仅只是用来呈现 HTML。
现在开发一个移动客户端，利用服务器 api 驱动前端，所有的用户交互都在客户端哪里。JSON 和 XML 格式通常用于序列化和传输结构化数据通过网络，所以能够创建这样的响应是任何一个现代服务器框架的必备。

响应格式
----------------

正如你可能知道的，在 Yii2 中需要从你的 action `return` 结果，而不是直接回应:

```php
// returning HTML result
return $this->render('index', [
    'items' => $items,
]);
```

好事是现在你可以从你的 action 直接返回不同类型的数据，即:

- 数组
- 一个实现 `Arrayable` 接口的对象
- 一个字符串
- 一个实现 `__toString()` 方法的对象。

只是别忘了告诉 Yii 做什么你想要的结果的格式，在 `return` 之前设置 `\Yii::$app->response->format`。例如：

```php
\Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
```

有效的格式:

- FORMAT_RAW
- FORMAT_HTML
- FORMAT_JSON
- FORMAT_JSONP
- FORMAT_XML

默认是 `FORMAT_HTML`.

JSON 响应
-------------

让我们返回一个数组：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

瞧!——我们的 JSON 响应框：

**结果**

```
{
    "0": "some",
    "1": "array",
    "2": "of",
    "data": ["associative", "array"]
}
```

Note: 你会得到一个异常，如果没有设置响应格式。

我们已经知道，我们也可以返回对象。

```php
public function actionView($id)
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $user = \app\models\User::find($id);
    return $user;
}
```

现在 $user 是  实现 Arrayable 接口的类 ActiveRecord 的实例，所以它可以很容易地转换为 JSON：

**结果**

```
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

我们甚至可以返回一个对象数组：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_JSON;
    $users = \app\models\User::find()->all();
    return $users;
}
```

现在 `$users `是 ActiveRecord 对象数组，但是在下面 Yii 使用 `\yii\helpers\Json::encode()` 遍历数据传递和转换，照顾类型本身:

**结果**

```
[
    {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com"
    },
    {
        "id": 2,
        "name": "Jane Foo",
        "email": "jane@example.com"
    },
    ...
]
```

XML 响应
------------

响应格式改为 FORMAT_XML 这样。现在你有了 XML：

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_XML;
    $items = ['some', 'array', 'of', 'data' => ['associative', 'array']];
    return $items;
}
```

**结果**

```xml
<response>
    <item>some</item>
    <item>array</item>
    <item>of</item>
    <data>
        <item>associative</item>
        <item>array</item>
    </data>
</response>
```

是的，我们可以跟我们之前做的一样转换对象和数组的对象。

```php
public function actionIndex()
{
    \Yii::$app->response->format = \yii\web\Response::FORMAT_XML;
    $users = \app\models\User::find()->all();
    return $users;
}
```

**结果**

```xml
<response>
    <User>
        <id>1</id>
        <name>John Doe</name>
        <email>john@example.com</email>
    </User>
    <User>
        <id>2</id>
        <name>Jane Foo</name>
        <email>jane@example.com</email>
    </User>
</response>
```

自定义响应格式
----------------------

让我们创建一个定制的响应格式。例子做点有趣和疯狂的事我回应 PHP 数组。
首先，我们需要格式化程序本身。创建 `components/PhpArrayFormatter.php`：

```php
<?php
namespace app\components;

use yii\helpers\VarDumper;
use yii\web\ResponseFormatterInterface;

class PhpArrayFormatter implements ResponseFormatterInterface
{
    public function format($response)
    {
        $response->getHeaders()->set('Content-Type', 'text/php; charset=UTF-8');
        if ($response->data !== null) {
            $response->content = "<?php\nreturn " . VarDumper::export($response->data) . ";\n";
        }
    }
}
```

现在我们需要在注册应用程序配置 (通常是 `config/web.php`):

```php
return [
    // ...
    'components' => [
        // ...
        'response' => [
            'formatters' => [
                'php' => 'app\components\PhpArrayFormatter',
            ],
        ],
    ],
];
```

现在是准备使用。在 `controllers/SiteController` 创建一个新的方法 `actionTest`:

```php
public function actionTest()
{
    Yii::$app->response->format = 'php';
    return [
        'hello' => 'world!',
    ];
}
```

就是这样。执行后，Yii 将回应以下:

```php
<?php
return [
    'hello' => 'world!',
];
```
