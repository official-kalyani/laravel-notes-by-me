>>How to login from different tables

```
$credentials = $request->only('email', 'password');
if (Auth::guard('web')->attempt($credentials)) {
// User login successful
return redirect('/dashboard');
}
if (Auth::guard('customer')->attempt($credentials)) {
// Admin login successful
return redirect('/homepage');
}
```
