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

Let’s Create & Open the `resourch/views/roles/roles-create.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
               {{ __('Create Roles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('roles') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('roles.store') }}" method="POST">
                        @csrf
                        <div>
                           <label for="" class="text-lg font-medium">Name</label>
                            <div class="mb-3">
                                <input type="text" name="name" value="{{ old('name') }}" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Name">
                            </div>
                            @error('name')
                               <div class=" text-red-600 mb-3">{{ $message }}</div>
                            @enderror
                        </div>
                        <div class="grid grid-cols-4 mb-3">
                                @if ($permissions->isNotEmpty())
                                    @foreach ($permissions as $permission)
                                       <div class="mt-3">
                                            <input type="checkbox" name="permission[]" id="{{ $permission->id }}" class="rounded" value="{{ $permission->name }}">
                                            <label for="{{ $permission->id }}">{{ $permission->name }}</label>
                                        </div>
                                    @endforeach
                                @endif
                        </div>
                        <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/roles/roles-edit.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Edit Roles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('roles') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('roles.update', $roles->id)}}" method="POST">
                        @csrf
                        @method('PUT')
                        <div>
                            <label for="" class="text-lg font-medium">Name</label>
                            <div class="mb-3">
                                <input type="text" name="name" value="{{ old('name',$roles->name) }}" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Name">
                            </div>
                            @error('name')
                               <div class=" text-red-600 mb-3">{{ $message }}</div>
                            @enderror
                        </div>
                        <div class="grid grid-cols-4 mb-3">
                                @if ($permissions->isNotEmpty())
                                    @foreach ($permissions as $permission)
                                        <div class="mt-3">
                                            <input type="checkbox" name="permission[]" id="{{ $permission->id }}" class="rounded" value="{{ $permission->name }}" {{($hasPermission->contains($permission->name) ? 'checked' : '')}}>
                                            <label for="{{ $permission->id }}">{{ $permission->name }}</label>
                                        </div>
                                    @endforeach
                                @endif
                        </div>
                        <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```