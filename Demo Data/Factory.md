### 1. Install the Laravel 10 App
* Open the command prompt/ terminal and go to the directory where you want to install Laravel 10 application and then run the below command to install Laravel.

```php
composer create-project --prefer-dist laravel/laravel:^10 laravel-factory
```

### 2. Configure Database Credentials
* Simply go to the root of our application and locate **.env** file and update it with our own database credentials details which are database name, database user and database password.

```php
DB_CONNECTION=mysql
DB_HOST=127.0.0.1 
DB_PORT=3306 
DB_DATABASE=laravel_factory 
DB_USERNAME=root
DB_PASSWORD=
```

### 3. Create a Product Model, Migration and Factory Class

```php
php artisan make:migration create_products_table
php artisan make:model Product
php artisan make:factory ProductFactory
```

### 4. Update Product Migration
* Now, we need to update our product migration file, just go to the **database\migrations** directory and our product migration file is available there with the following name.
* ``YYYY_MM_DD_TIMESTAMP_create_products_table.php``
* Just open this product migration file and copy paste the following code in it.

```php
use Illuminate\Database\Migrations\Migration; 
use Illuminate\Database\Schema\Blueprint; 
use Illuminate\Support\Facades\Schema; 

return new class extends Migration {
/** * Run the migrations. */ 
public function up(): void 
{ 
Schema::create('products', function (Blueprint $table) 
			   { 
			   $table->id(); 
			   $table->string('code')->unique();
			   $table->string('name'); 
			   $table->integer('quantity'); 
			   $table->decimal('price', 8, 2); $table->text('description'); $table->timestamps(); }); } /** * Reverse the migrations. */ public function down(): void { Schema::dropIfExists('products'); } };
```






### 4. Update Product Model, Migration and Factory Class