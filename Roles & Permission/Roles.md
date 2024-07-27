### Controller Methods
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

```