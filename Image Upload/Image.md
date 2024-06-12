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
//input field
<input type="file" multiple name="image[]">
//validate
'image.*' => 'required|image|mimes:jpeg,jpg,png,gif,svg|max:5120'
foreach($request->image as $key => $value)
{
	$imageName = time().'.'.$value->extension();
	$path = 'uploads/schedule/';
	$value->move($path, $imageName);
	$imageName[] = $imageName;
}


// delete file
$image_path = public_path('about/'.$destroy->image);
if(File::exists($image_path))
{
  File::delete($image_path);
}
```