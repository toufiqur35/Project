### Controller Methods

Create controller:

```
php artisan make:controller RoleController
```

Let’s Open the `app/Http/Controllers/RoleController.php`

```php
class RoleController extends Controller implements HasMiddleware
{
    public static function middleware(): array
    {
        return [
            new Middleware('permission:view roles', only: ['index']),
            new Middleware('permission:create roles', only: ['create']),
            new Middleware('permission:edit roles', only: ['edit']),
            new Middleware('permission:delete roles', only: ['destroy']),
        ];
    }

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

### Create Blade File Roles:
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
                                    <th scope="col" class="px-6 py-3 w-[10%]">
                                        #
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[10%]">
                                        Name
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[20%]">
                                        Permission
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[20%]">
                                        Created
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[40%]">
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
                                        @can('edit roles')                             
                                        <a class="bg-slate-700 text-white px-3 py-2 rounded-md" href="{{ route('roles.edit', $role->id) }}">Add/Edit Role Permission</a>
                                        @endcan

                                        @can('delete roles')
                                        <a class="bg-red-700 text-white px-3 py-2 rounded-md" href="{{ route('roles.destroy', $role->id) }}">Delete</a>
                                        @endcan
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