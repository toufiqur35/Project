* update data into database

```php
public function update(Request $request,$id)
{
	$data = Student::find($id);
        $validate = $request->validate([
            'title' => 'required',
            'sort_description' => 'required',
            'image' => 'required|mimes:jpeg,jpg,png,gif',
        ]);
        
        if(isset($request->image))
        {
            $imageName = time().'.'.$request->image->extension();
	        $path = 'uploads/carousel';
	        $request->image->move($path, $imageName);
            $data->image = $imageName;
        }
        
        $data->title=$request->title,
        $data->sort_description = $request->sort_description,
        $data->save();
return redirect('/home')->with('message', 'Student update Successfully');
}
```