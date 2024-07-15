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

### Step 7: Add Controllers
Let’s get this done, we’ll create 3 controllers, and they would be resource controllers. Let’s quickly do this with command below.

```
php artisan make:controller PermissionController 
php artisan make:controller RoleController 
```

### Step 8: Create Routes
Open your `route/web.php`

```php
use App\Http\Controllers\PermissionController;

Route::resource('permissions',[PermissionController::class]);
```

### Step 9: Controller Methods
Let’s Open the `app/Http/Controllers/PermissionController.php`

```php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use Spatie\Permission\Models\Permission;

class PermissionController extends Controller
{
    public function index()
    {
        $permissions = Permission::all();
        return view('permissions.permission', compact('permissions'));
    }

    public function create()
    {
        return view('permissions.create');
    }

    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|unique:permissions|min:3',
        ]);
        
        if($validator->passes()){
            Permission::create(['name' => $request->name]);
            return redirect('/permissions')->with('success', 'Permission created successfully');
        }else{
            return redirect('/permissions/create')->withInput()->withErrors($validator);
        }
    }

    public function edit($id)
    {
        $permission = Permission::find($id);
        return view('permissions.edit', compact('permission'));
    }

    public function update(Request $request, $id)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|unique:permissions|min:3',
        ]);

        $permission = Permission::find($id);
        if($validator->passes()){
            $permission->update(['name' => $request->name]);
            return redirect('/permissions')->with('success', 'Permission updated successfully');
        }else{
            return redirect('/permissions/edit')->withInput()->withErrors($validator);
        }
    }

    public function destroy($id)
    {
        $permission = Permission::find($id);
        $permission->delete();
        return redirect('/permissions')->with('success', 'Permission deleted successfully');
    }
}
```

### Step 9: Create Blade File

