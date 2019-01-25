
## [Laravel SendCloud 发送邮件][1]


  [1]: https://github.com/NauxLiu/Laravel-SendCloud

 1. composer 安装
 
 > composer require naux/sendcloud
 
 2. 修改 `config/app.php`，添加服务提供者
 
 ```php
'providers' => [
   // 添加这行
    Naux\Mail\SendCloudServiceProvider::class,
];
```

 3. 在 `.env` 中配置你的密钥， 并修改邮件驱动为 `sendcloud`
 
 ```
 ## 选择发送
 MAIL_DRIVER=sendcloud

 ## SEND_CLOUD 配置
 SEND_CLOUD_USER=hu243373737_test_Uf9sAJ
 SEND_CLOUD_KEY=0Zv2HwgZ4FedBHJ1
 ```
 
### 实例
 
 > 应用场景：注册用户需要验证邮箱。注册的时候会发送邮件到邮箱，然后去邮箱中点击验证链接，修改激活状态，更改 `token`
 
 1. 注册 auth
 
 > php artisan make:auth

 2. 修改 `app/Http/Controllers/Auth/RegisterController.php`
 
```php


namespace App\Http\Controllers\Auth;
use Mail;
use Naux\Mail\SendCloudTemplate;

protected function create(array $data)
{
	$user =  User::create([
	'name' => $data['name'],
	'email' => $data['email'],
	'avatar' => 'images/avatars/default.gif',
	'confirmation_token' => str_random(40),
	'password' => bcrypt($data['password']),
	]);

	$this->sendVerifyEmailTo($user);
	return $user;
}


public function sendVerifyEmailTo($user)
{
	// 模板变量
	$bind_data = [
	'url' => route("send_email", ['token' => $user->confirmation_token]),
	'name' => $user->name
	];
	$template = new SendCloudTemplate('test_template_active', $bind_data);

	Mail::raw($template, function ($message) use ($user) {
	$message->from('9265959@qq.com', 'mason');

	$message->to($user->email);
	});
}
```

 3. 增加创建链接逻辑

```php
public function send(Request $request)
{
	$user = User::where('confirmation_token', $request->token)->first();

	if (is_null($user)) {
	return redirect('/');
	}

	$user->is_active = 1;
	$user->confirmation_token = str_random(40);
	$user->save();

	return redirect('/home');
}
```

## [laracasts/flash插件提示信息](https://github.com/laracasts/flash "laracasts/flash插件提示信息")

1. composer 安装

> composer require laracasts/flash

2. 修改 `config/app.php`，添加服务提供者

 
 ```php
'providers' => [
   // 添加这行
   Laracasts\Flash\FlashServiceProvider::class,
];
```

### 实例
 
 > 应用场景：操作成功提示用户信息

1. 引入 `bootstrap.css` `jquery.js` `bootstrap.js`

``` html
<!doctype html>
<html lang="{{ app()->getLocale() }}">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">

        <title>Hello Feng</title>

        <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    </head>
    <body>
        <div class="container">
            @include("flash::message")
        </div>
    </body>
    <script src="//code.jquery.com/jquery.js"></script>
    <script src="//maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <script>
        $('#flash-overlay-modal').modal();
    </script>
</html>
```

2. 程序调用

```php
public function send(Request $request)
{
    ...
    
    flash("邮箱验证成功！", 'success');

    ...

    return redirect('/home');
}
```

