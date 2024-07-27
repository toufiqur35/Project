### Controller Methods
Let’s Open the `app/Http/Controllers/UserController.php`

```php
class UserController extends Controller implements HasMiddleware
{
    public static function middleware(): array
    {
        return [
            new Middleware('permission:view users', only: ['index']),
            new Middleware('permission:edit users', only: ['edit']),
            new Middleware('permission:delete users', only: ['destroy']),
        ];
    }

    public function index()
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }

  
    function edit($id){
        $users = User::find($id);
        $roles = Role::orderBy('name','ASC')->get();
        $hasRoles = $users->roles->pluck('id');
        return view('users.edit',compact('users','roles','hasRoles'));
    }

    function update(Request $request,$id){
        $user = User::find($id);
        $user->syncRoles($request->role);
        return redirect('/users')->with('success', 'User updated successfully');
    }

    function destroy($id)
    {
        $users = User::find($id);
        $users->delete();
        return redirect('/users')->with('success', 'Users deleted successfully');
    }
}
```

### Create Blade File User:
Let’s Create & Open the `resourch/views/users/users.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                {{ __('Users') }}
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
                                    <th scope="col" class="px-6 py-3 w-[20%]">
                                        Name
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[25%]">
                                        Email
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[30%]">
                                        Role Permissions
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[25%]">
                                        Action
                                    </th>
                                </tr>
                            </thead>
                            <tbody>
                                @foreach ($users as $user)
                                <tr class="bg-white border-b border-gray-200">
                                    <td class="px-6 py-4">
                                        {{ $user->name }}
                                    </td>
                                    <td class="px-6 py-4">
                                        {{ $user->email }}
                                    </td>
                                    <td class="px-6 py-4">
                                        @if (!empty($user->getRoleNames()))
                                            @foreach ($user->getRoleNames() as $roleName)                              
                                            <label class="bg-green-500 text-white px-3 py-2 rounded-md"> {{ $roleName }}</label>
                                            @endforeach
                                        @endif
                                    </td>
                                    <td class="px-6 py-4">
                                        @can('edit users')
                                        <a class="bg-slate-700 text-white px-3 py-2 rounded-md" href="{{ route('users.edit', $user->id) }}">Edit</a>
                                        @endcan

                                        @can('delete users')
                                        <a class="bg-red-700 text-white px-3 py-2 rounded-md" href="{{ route('users.destroy', $user->id) }}">Delete</a>
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

Let’s Create & Open the `resourch/views/users/users_create.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Create Articles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('articles') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('articles.store') }}" method="POST">
                        @csrf
                        <div>
                            <div>
                                <label for="" class="text-lg font-medium">Title</label>
                                <div class="mb-3">
                                    <input type="text" name="title" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter title" value="{{ old('title') }}">
                                </div>
                                @error('title')
                                <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>

                            <div>
                                <label for="" class="text-lg font-medium">Author</label>
                                <div class="mb-3">
                                    <input type="text" name="author" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter author name" value="{{ old('author') }}">
                                </div>
                                @error('author')
                                <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>

                            <div>
                                <label for="" class="text-lg font-medium">Article</label>
                                <div class="mb-3">
                                    <textarea name="article" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter article">{{old('article')}}</textarea></textarea>
                                </div>
                                @error('article')
                                <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>
                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/users/users_edit.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Edit Users') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('users') }}">Back</a>
            </h2>
        </div>
    </x-slot>


    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('users.update',$users->id)}}" method="POST">
                        @csrf
                        <div>
                            <div>
                                <label for="" class="text-lg font-medium">Name</label>
                                <div class="mb-3">
                                    <input type="text" name="name" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Name" value="{{ old('name',$users->name) }}">
                                </div>
                                @error('name')
                                   <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>
                            <div>
                                <label for="" class="text-lg font-medium">Email</label>
                                <div class="mb-3">
                                    <input type="email" name="email" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter Email" value="{{ old('email',$users->email) }}">
                                </div>
                                @error('email')
                                   <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>
                            <div class="grid grid-cols-4 mb-3">
                                @if ($roles->isNotEmpty())
                                    @foreach ($roles as $role)
                                        <div class="mt-3">
                                            <input type="checkbox" name="role[]"
                                            {{($hasRoles->contains($role->id) ? 'checked' : '')}}
                                            id="{{ $role->id }}" class="rounded"
                                            value="{{ $role->name }}" >
                                            <label for="{{ $role->id }}">{{ $role->name }}</label>
                                        </div>
                                    @endforeach
                                @endif
                            </div>
                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```