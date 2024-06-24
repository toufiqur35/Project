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
<form class="space-y-4 md:space-y-6" action="#" enctype="multipart/form-data" method="POST">

        @csrf
```