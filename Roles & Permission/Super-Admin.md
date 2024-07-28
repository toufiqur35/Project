If you want a "Super Admin" role to respond `true` to all permissions, without needing to assign all those permissions to a role, you can use [Laravel's `Gate::before()` method](https://laravel.com/docs/master/authorization#intercepting-gate-checks). For example:

In Laravel 11 this would go in the `boot()` method of `AppServiceProvider`: In Laravel 10 and below it would go in the `boot()` method of `AuthServiceProvider.php`:

```php
use Illuminate\Support\Facades\Gate;
// ...
public function boot()
{
    Gate::before(function ($user, $ability) {
        return $user->hasRole('Super Admin') ? true : null;
    });
}
```