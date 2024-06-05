```php
public function destroy($id)
{
    $deleteItems = Student::find($id);
    $deleteItems->delete();
    return redirect('/home')->with('message','delete Successfully!!');
}
```