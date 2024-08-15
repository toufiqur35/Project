#### **Step 1: Install Laravel**

First of all we need to get fresh Laravel version application using bellow command, So open your terminal OR command prompt and run bellow command:

```
composer create-project laravel/laravel example-app
```

#### **Step 2: Create migration & controller file**

```
php artisan make:migration create_users_table
php artisan make:controller UserController
```

#### **Step 3: Create Users Table 

```mysql
Schema::create('users', function (Blueprint $table) {
     $table->id();
     $table->string('firstName',50);
     $table->string('lastName',50);
     $table->string('email',50)->unique();
     $table->string('mobile',30);
     $table->string('password',100);
     $table->string('otp',10);
     $table->timestamps();
});
```

#### **Step 4: Database Configuration**

In this step, we need to add database configuration in .env file. so let's add following details and then run migration command:

```mysql
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=root
DB_PASSWORD=
```

Next, run migration command to create users table.

```
php artisan migrate
```