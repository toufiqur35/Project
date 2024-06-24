```php
public function search(Request $request){
	$search = $request->search;
	$posts = Post::where(function($query) use ($search){
		$query->where('title','like',"%$search%)
		->orWhere('description','like',"%$search%");
	})
	->orWhereHas('category',function($query) use ($search){
		$query->where('name','like',"%$search%);
	})
	->orWhereHas('user',function($query) use ($search){
		$query->where('name','like',"%$search%);
	})->get();
	return view('index',compact('posts','search'));
}
```

```html
<form action="/search" method="get">
	<div>
		<input type="text" name="search" placeholder="search......." value="{{ isset($search) ? $search : '' }}">
		<button type="submit">Search</button>
	</div>
</form>
```

```php
Route::get('/',[DemoController::class,'index']);
Route::get('/search',[DemoController::class,'search']);
```