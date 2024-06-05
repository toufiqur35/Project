```php
//common part start
//validate
'image' => 'required|mimes:jpeg,jpg,png,gif'
//create
'image' => $imageName
//common part end

//example 1
$imageName = time().'.'.$request->image->extension();
$path = 'uploads/carousel';
$request->image->move($path, $imageName);

//example 2
if($request->has('image'))
{
	$file = $request->file('image');
	$extension = $file->getClientOriginalExtension();
	$filename = time().'.'.$extension;
	$path = 'uploads/schedule/';
	$file->move($path, $filename);
}

//multi image
//validate
'image*' => 'required|mimes:jpeg,jpg,png,gif'
```