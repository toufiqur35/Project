```php
public function filter(Request $request, User $user)  
{  
// Search for a user based on their name.  
if ($request->has('name')) {  
return $user->where('name', $request->input('name'))->get();  
}  
  
// Search for a user based on their company.  
if ($request->has('company')) {  
return $user->where('company', $request->input('company'))  
->get();  
}  
  
// Search for a user based on their city.  
if ($request->has('city')) {  
return $user->where('city', $request->input('city'))->get();  
}  
    
return User::_all_();  
}
```

```php
// Search for a user based on their name.  
if ($request->has('name')) {    // Has a 'city' search parameter also been provided?  
    if ($request->has('city')) 
    {        
    // Search for a user based on their name and their city.        
		    return $user->where('name', $request->input('name'))  
            ->where('city', $request->input('city'))  
            ->get();  
    }    
    return $user->where('name', $request->input('name'))->get();}
```