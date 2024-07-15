The `Spatie` Laravel Permission package is a robust tool for managing roles and permissions in a Laravel application.
### Step 1: Installing Laravel Application

Run this command in your terminal to create a new `laravel` project.

```
composer create-project --prefer-dist laravel/laravel permission_tutorial
```

### Step 2: Create Authentication Using Laravel Breeze

Now, we are going to use **Laravel Breeze** to generate a nice auth scaffolding for our application. Let’s quickly do this. Run the command

```
composer require laravel/breeze --dev
php artisan breeze:install
npm install
npm run dev
```

### Step 3: Install `spatie/laravel-permission` Packages

Here we install the `spatie` permissions package with the command below.

```
composer require spatie/laravel-permission
```

### Step 4: Publish the Migration

Next, publish the migration files to create the necessary database tables:

```
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

### Step 5: Clear your config cache. 
This package requires access to the `permission` config settings in order to run migrations. If you've been caching configurations locally, clear your config cache with either of these commands:

```
 php artisan optimize:clear
 # or
 php artisan config:clear
```

### Step 6: Run the migrations: 
After the config and migration have been published and configured, you can create the tables for this package by running:

```
 php artisan migrate
```

```
```