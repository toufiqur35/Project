Laravel includes the ability to seed your database with data using seed classes. All seed classes are stored in the `database/seeders` directory. By default, a `DatabaseSeeder` class is defined for you. From this class, you may use the `call` method to run other seed classes, allowing you to control the seeding order.

#### Create Seeder Class:
* Laravel gives command to create seeder in laravel. so you can run following command to make seeder in laravel application.

```
php artisan make:seeder UserSeeder
```

after running the above command, it will create one file UserSeeder.php on the seeders folder. All seed classes are stored in the database/seeders directory.

Then you can write code of create  user using model in laravel.

```php
namespace Database\Seeders;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use App\Models\User;
class AdminUserSeeder extends Seeder

{
    public function run(): void
    {
        User::create([
            'name' => 'Hardik',
            'email' => 'admin@gmail.com',
        ]);
    }
}
```

### Calling Additional Seeders
Within the `DatabaseSeeder` class, you may use the `call` method to execute additional seed classes. Using the `call` method allows you to break up your database seeding into multiple files so that no single seeder class becomes too large. The `call` method accepts an array of seeder classes that should be executed:

* Seeders/DatabaseSeeders.php
```php
public function run(): void
{
	$this->call([
		UserSeeder::class,
		PostSeeder::class,
		CommentSeeder::class,
	]);
}
```

```
php artisan db:seed
```