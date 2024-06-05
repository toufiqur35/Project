* store data into database

```php
public function carouselCreateProcess(Request $request)
{
        $validate = $request->validate([
            'title' => 'required',
            'sort_description' => 'required',
            'image' => 'required|mimes:jpeg,jpg,png,gif',
        ]);
        
        $imageName = time().'.'.$request->image->extension();
        $path = 'uploads/carousel';
        $request->image->move($path, $imageName);
        
        Carusel::create([
            'title'=>$request->title,
            'sort_description' => $request->sort_description,
            'image' => $imageName,
        ]);
        
return redirect('/admin/carousel')->with('message', 'carousel Created Successfully');
}
```