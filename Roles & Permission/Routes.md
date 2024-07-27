### Create Routes :
Open your `route/web.php`

```php
Route::get('/', function () {
    return view('home');
})->name('dashboard');

  
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

    // roles
    Route::get('/roles', [RoleController::class, 'index'])->name('roles');
    Route::get('/roles/create', [RoleController::class, 'create'])->name('roles.create');
    Route::post('/roles/store', [RoleController::class, 'store'])->name('roles.store');
    Route::get('/roles/edit/{id}', [RoleController::class, 'edit'])->name('roles.edit');
    Route::put('/roles/update/{id}', [RoleController::class, 'update'])->name('roles.update');
    Route::get('/roles/delete/{id}', [RoleController::class, 'destroy'])->name('roles.destroy');

     // articles
     Route::get('/articles', [ArticleController::class, 'index'])->name('articles');
     Route::get('/articles/create', [ArticleController::class, 'create'])->name('articles.create');
     Route::post('/articles/store', [ArticleController::class, 'store'])->name('articles.store');
     Route::get('/articles/edit/{id}', [ArticleController::class, 'edit'])->name('articles.edit');
     Route::put('/articles/update/{id}', [ArticleController::class, 'update'])->name('articles.update');
     Route::get('/articles/delete/{id}', [ArticleController::class, 'destroy'])->name('articles.destroy');

     // users
     Route::get('/users', [UserController::class, 'index'])->name('users');
     Route::get('/users/edit/{id}', [UserController::class, 'edit'])->name('users.edit');
     Route::post('/users/update/{id}', [UserController::class, 'update'])->name('users.update');
     Route::get('/users/delete/{id}', [UserController::class, 'destroy'])->name('users.destroy');

});
```