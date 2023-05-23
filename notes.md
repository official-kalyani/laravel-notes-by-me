>>How to login from different tables

It will be in login method

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

In config\auth.php

```
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ], 
        <!-- for customer guard -->
        'customer' => [
            'driver' => 'session',
            'provider' => 'customers',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],

        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Models\Admin::class,
        ],
        <!-- for customer table -->
        'customers' => [
            'driver' => 'eloquent',
            'model' => App\Models\Customer::class,
        ],

        
    ],
```

In the customer controller
```
class CustomerController extends Controller
{
    public function home(){
        if (Auth::guard('customer')->check()) {
            $customerName = Auth::guard('customer')->user()->full_name;
        } else {
            // Customer is not logged in
            $customerName = null;
        }
        $services = Category::where('type', '=', 'service')->get();
        $products = Category::where('type', '=', 'product')->get();
        return view('customer.homepage',compact('services','products','customerName')); 
        // return view('layouts.frontend_layout.layouts.master'); 
     }
}
```
 And in Home page 
 ```
 @if($customerName)
                <li>
                {{ $customerName }} <i class="fa fa-user" aria-hidden="true"></i>
                </li>
                <li> <a  href="{{url('/logout')}}"><i class="mdi mdi-logout font-size-16 align-middle me-1"></i> Logout</a></li>
              @else
                <li><a href="#">Login / Signup</a></li>
              @endif
 ```
