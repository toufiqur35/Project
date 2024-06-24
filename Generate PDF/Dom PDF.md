**Step 1: Install Laravel 10**
* This step is not required; however, if you have not created the `laravel` app, then you may go ahead and execute the below command:

```php
composer create-project laravel/laravel example-app
```

**Step 2: Install Dom PDF Package**
* next, we will install `DomPDF` package using following composer command, let's run bellow command:

```php
composer require barryvdh/laravel-dompdf
```

Step 3:  Configuration
* The defaults configuration settings are set in `config/dompdf.php`. Copy this file to your own config directory to modify the values. You can publish the config using this command:

```php
    php artisan vendor:publish --provider="Barryvdh\DomPDF\ServiceProvider"
```

**Step 4: Create Controller**
* In this step, we will create `PDFController` with `generatePDF`() where we write code of generate pdf. so let's create controller using bellow command.

```php
php artisan make:controller PDFController
```

Now, update the code on the controller file.
app/Http/Controllers/`PDFController.php`

```php

```

**Step 5: Add Route**
* routes/`web.php`

```php
Route::get('generate-pdf', [PDFController::class, 'generatePDF']);
```