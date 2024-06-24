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
 public function generatePDF()
    {
        $users = User::get();
        $data = [
            'title' => 'Welcome to my website',
            'date' => date('m/d/Y'),
            'users' => $users
        ]; 
        $pdf = PDF::loadView('myPDF', $data);            //myPDF is a view file
        return $pdf->download('website_name.pdf');
    }
```

**Step 5: Add Route**
* routes/`web.php`

```php
Route::get('generate-pdf', [PDFController::class, 'generatePDF']);
```

**Step 6: Create View File**
* In Last step, let's create `myPDF.blade.php (resources/views/myPDF.blade.php)` for layout of pdf file and put following code:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{ $title }}</title>
</head>
<body>
    <h1>{{ $title }}</h1>
    <p>{{ $date }}</p>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
    tempor incididunt ut labore et dolore magna aliqua.</p>
    <table class="table table-bordered">
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
        </tr>
        @foreach($users as $user)
        <tr>
            <td>{{ $user->id }}</td>
            <td>{{ $user->name }}</td>
            <td>{{ $user->email }}</td>
        </tr>
        @endforeach
    </table>
</body>
</html>
```