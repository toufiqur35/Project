You may assign aliases to middleware in your application's `bootstrap/app.php` file. Middleware aliases allows you to define a short alias for a given middleware class, which can be especially useful for middleware with long class names:

```php
->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role' => \Spatie\Permission\Middleware\RoleMiddleware::class,
            'permission' => \Spatie\Permission\Middleware\PermissionMiddleware::class,
            'role_or_permission' => \Spatie\Permission\Middleware\RoleOrPermissionMiddleware::class,
        ]);
    })
```

In Laravel 11, if your controller implements the `HasMiddleware` interface, you can register [controller middleware](https://laravel.com/docs/11.x/controllers#controller-middleware) using the `middleware()` method:

```php
public static function middleware(): array

    {

        return [

            new Middleware('permission:view articles', only: ['index']),

            new Middleware('permission:create articles', only: ['create']),

            new Middleware('permission:edit articles', only: ['edit']),

            new Middleware('permission:delete articles', only: ['destroy']),

        ];

    }
```