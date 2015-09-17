[日志：问题和解决方案](https://github.com/iiYii/yii2-cookbook/edit/gh-pages/book/logging-problems-and-solutions.md)
===============================


Yii 中的日志是非常灵活的。基础知识很简单但有时需要时间配置得到你想要的一切。下面有收集了一些准备使用解决方案。希望你会找到你所要找的。

写404文件和发送通过电子邮件
---------------------------------------------

是说404发生了太多次，导致已经不能通过邮件通知你了，只好写入文件。让我们实现它。


```php
'components' => [
    'log' => [
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'categories' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
            ],
            'email' => [
                'class' => 'yii\log\EmailTarget',
                'except' => ['yii\web\HttpException:404'],
                'levels' => ['error', 'warning'],
                'message' => ['from' => 'robot@example.com', 'to' => 'admin@example.com'],
            ],
        ],
    ],
],
```

When there's unhandled exception in the application Yii logs it additionally to displaying it
to end user or showing customized error page. Exception message is what actually to be written and
the fully qualified exception class name is the category we can use to filter messages when
configuring targets. 404 can be triggered by throwing `yii\web\NotFoundHttpException` or automatically.
In both cases exception class is the same and is inherited from `\yii\web\HttpException` which is a bit
special in regards to logging. The speciality is the fact that HTTP status code prepended by `:` is
appended to the end of the log message category. In the above we're using `categories` to include
and `except` to exclude 404 log messages.

Immediate logging
-----------------

By default Yii accumulates logs till the script is finished or till the number of logs accumulated is
enough which is 1000 messages by default for both logger itself and log target. It could be that you
want to log messages immediately. For example, when running an import job and checking logs to see
the progress. In this case you need to change settings via application config file:

```php
'components' => [
    'log' => [
        'flushInterval' => 1, // <-- here
        'targets' => [
            'file' => [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
                'exportInterval' => 1, // <-- and here
            ],
        ],
    ]
]
```

Write different logs to different files
-----------------

Usually a program has a lot of functions. Sometimes it is necessary to control these functions by logging. If everything is logged in one file this file becomes too big and too difficult to maintain. Good solution is to write different functions logs to different files.

For example you have two functions: catalog and basket. Let's write logs to catalog.log and basket.log respectively. In this case you need to establish categories for your log messages. Make a connection between them and log targets by changing application config file:

```php
'components' => [
    'log' => [
        'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'categories' => ['catalog'],
                    'logFile' => '@app/runtime/logs/catalog.log',
                ],
                [
                    'class' => 'yii\log\FileTarget',
                    'categories' => ['basket'],
                    'logFile' => '@app/runtime/logs/basket.log',
                ],
        ],
    ]
]
```

After this you are able to write logs to separate files adding category name to log function as second parameter. Examples: 

```php
\Yii::info('catalog info', 'catalog');
\Yii::error('basket error', 'basket');
\Yii::beginProfile('add to basket', 'basket');
\Yii::endProfile('add to basket', 'basket');
```
