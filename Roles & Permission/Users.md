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