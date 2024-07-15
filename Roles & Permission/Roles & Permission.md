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
Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
    
    // permission
    Route::get('/permissions', [PermissionController::class, 'index'])->name('permission');
    Route::get('/permissions/create', [PermissionController::class, 'create'])->name('permission.create');
    Route::post('/permissions/store', [PermissionController::class, 'store'])->name('permission.store');
    Route::get('/permissions/edit/{id}', [PermissionController::class, 'edit'])->name('permission.edit');
    Route::put('/permissions/update/{id}', [PermissionController::class, 'update'])->name('permission.update');
    Route::get('/permissions/delete/{id}', [PermissionController::class, 'destroy'])->name('permission.destroy');

    // permission
    Route::get('/roles', [RoleController::class, 'index'])->name('roles');
    Route::get('/roles/create', [RoleController::class, 'create'])->name('roles.create');
    Route::post('/roles/store', [RoleController::class, 'store'])->name('roles.store');
    Route::get('/roles/edit/{id}', [RoleController::class, 'edit'])->name('roles.edit');
    Route::put('/roles/update/{id}', [RoleController::class, 'update'])->name('roles.update');
    Route::get('/roles/delete/{id}', [RoleController::class, 'destroy'])->name('roles.destroy');
});
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

Let’s Open the `app/Http/Controllers/RoleController.php`

```php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use Spatie\Permission\Models\Permission;
use Illuminate\Support\Facades\Validator;
use Spatie\Permission\Models\Role;

class RoleController extends Controller
{
    public function index()
    {
        $roles = Role::all();
        return view('roles.roles', compact('roles'));
    }

    public function create()
    {
        $permissions = Permission::all();
        return view('roles.create', compact('permissions'));
    }

    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|unique:permissions|min:3',
        ]);

        if($validator->passes()){
            $role = Role::create(['name' => $request->name]);
            if(!empty($request->permission)){
                $role->givePermissionTo($request->permission);
            }
            return redirect('/roles')->with('success', 'Roles created successfully');
        }else{
            return redirect('/roles/create')->withInput()->withErrors($validator);
        }
    }

    public function edit($id)
    {
        $roles = Role::find($id);
        $permissions = Permission::all();
        $hasPermission = $roles->permissions->pluck('name');
        return view('roles.edit', compact('permissions', 'roles', 'hasPermission'));
    }

  

    public function update(Request $request, $id)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|unique:roles,name,'.$id.',id',
        ]);
        $roles = Role::find($id);
        if($validator->passes()){
            $roles->update(['name' => $request->name]);
            if(!empty($request->permission)){
                $roles->syncPermissions($request->permission);
            }else{
                $roles->syncPermissions([]);
            }
            return redirect('/roles')->with('success', 'Roles update successfully');
        }else{
            return redirect('/roles/create')->withInput()->withErrors($validator);
        }
    }

    public function destroy($id)
    {
        $roles = Role::find($id);
        $roles->delete();
        return redirect('/roles')->with('success', 'Roles deleted successfully');
    }
}
```

### Step 10: Create Blade File Permission

Let’s Create & Open the `resourch/views/permission/permission.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                {{ __('Permission') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('permission.create') }}">Create</a>
            </h2>
        </div>
</x-slot>

@if (Session::has('success'))
<div class="bg-green-200 border-green-600 p-4 mt-3 mx-16 rounded-sm shadow-sm">{{ Session::get('success') }}</div>
@endif

<div class="py-12">
   <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
       <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
           <div class="p-6 text-gray-900">
               <div class="relative overflow-x-auto">
                   <table class="w-full text-sm text-left rtl:text-right text-gray-500 ">
                       <thead class="text-sm font-bold text-gray-800 border-b border-gray-200 ">
                           <tr>
                               <th scope="col" class="px-6 py-3 w-[15%]">
                                  #
                               </th>
                               <th scope="col" class="px-6 py-3 w-[40%]">
                                   Name
                               </th>
                              <th scope="col" class="px-6 py-3 w-[25%]">
                                  Created
                               </th>
                               <th scope="col" class="px-6 py-3 w-[25%]">
                                  Action
                               </th>
                           </tr>
                       </thead>
                        <tbody>
                            @foreach ($permissions as $permission)
                               <tr class="bg-white border-b border-gray-200">
                                   <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap ">
                                        {{ $permission->id }}
                                    </th>
                                    <td class="px-6 py-4">
                                        {{ $permission->name }}
                                    </td>
                                    <td class="px-6 py-4">
                                        {{ Carbon\Carbon::parse($permission->created_at)->format('d-M-Y') }}
                                    </td>
                                    <td class="px-6 py-4">
                                       <a class="bg-slate-700 text-white px-3 py-2 rounded-md" href="{{ route('permission.edit', $permission->id) }}">Edit</a>
                                        <a class="bg-red-700 text-white px-3 py-2 rounded-md" href="{{ route('permission.destroy', $permission->id) }}">Delete</a>
                                    </td>
                                </tr>
                                @endforeach
                            </tbody>
                       </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/permission/permission-create.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Create Permission') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('permission') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('permission.store') }}" method="POST">
                        @csrf
                        <div>
                            <label for="" class="text-lg font-medium">Name</label>
                            <div class="mb-3">
                                <input type="text" name="name" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Name">
                            </div>
                            @error('name')
                               <div class=" text-red-600 mb-3">{{ $message }}</div>
                            @enderror
                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/permission/permission-edit.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Edit Permission') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('permission') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('permission.update', $permission->id)}}" method="POST">
                        @csrf
                        @method('PUT')
                        <div>
                            <label for="" class="text-lg font-medium">Name</label>
                            <div class="mb-3">
                                <input type="text" name="name" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Name" value="{{ old('name',$permission->name) }}">
                            </div>
                            @error('name')
                               <div class=" text-red-600 mb-3">{{ $message }}</div>
                            @enderror
                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                        </div>
                    </form>
               </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

### Step 11: Create Blade File Roles

Let’s Create & Open the `resourch/views/roles/roles.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                {{ __('Roles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('roles.create') }}">Create</a>
            </h2>
        </div>
    </x-slot>
    @if (Session::has('success'))
        <div class="bg-green-200 border-green-600 p-4 mt-3 mx-16 rounded-sm shadow-sm">           {{ Session::get('success') }}
        </div>
    @endif

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <div class="relative overflow-x-auto">
                        <table class="w-full text-sm text-left rtl:text-right text-gray-500 ">
                           <thead class="text-sm font-bold text-gray-800 border-b border-gray-200 ">
                                <tr>
                                    <th scope="col" class="px-6 py-3 w-[10%]">
                                        #
                                    </th>

                                    <th scope="col" class="px-6 py-3 w-[20%]">

                                        Name

                                    </th>

                                    <th scope="col" class="px-6 py-3 w-[30%]">

                                        Permission

                                    </th>

                                    <th scope="col" class="px-6 py-3 w-[20%]">

                                        Created

                                    </th>

                                    <th scope="col" class="px-6 py-3 w-[30%]">

                                        Action

                                    </th>

                                </tr>

                            </thead>

                            <tbody>

                                @foreach ($roles as $role)

                                <tr class="bg-white border-b border-gray-200">

                                    <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap ">

                                        {{ $role->id }}

                                    </th>

                                    <td class="px-6 py-4">

                                        {{ $role->name }}

                                    </td>

                                    <td class="px-6 py-4">

                                        {{ $role->permissions->pluck('name')->implode(', ') }}

                                    </td>

                                    <td class="px-6 py-4">

                                        {{ Carbon\Carbon::parse($role->created_at)->format('d-M-Y') }}

                                    </td>

                                    <td class="px-6 py-4">

                                        <a class="bg-slate-700 text-white px-3 py-2 rounded-md" href="{{ route('roles.edit', $role->id) }}">Edit</a>

                                        <a class="bg-red-700 text-white px-3 py-2 rounded-md" href="{{ route('roles.destroy', $role->id) }}">Delete</a>

                                    </td>

                                </tr>

                                @endforeach

                            </tbody>

                        </table>

                    </div>

                </div>

            </div>

        </div>

    </div>

</x-app-layout>
```