### Controller Methods
Let’s Open the `app/Http/Controllers/PermissionController.php`

```php
class PermissionController extends Controller implements HasMiddleware
{
    public static function middleware(): array
    {
        return [
            new Middleware('permission:view permission', only: ['index']),
            new Middleware('permission:create permission', only: ['create']),
            new Middleware('permission:edit permission', only: ['edit']),
            new Middleware('permission:delete permission', only: ['destroy']),
        ];
    }

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