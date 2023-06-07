>How to login from different tables

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
     }
}
```
 And in Home page 
 ```
 @if($customerName)
    <li>
        {{ $customerName }} <i class="fa fa-user" aria-hidden="true"></i>
    </li>
    <li>
     <a  href="{{url('/logout')}}"><i class="mdi mdi-logout font-size-16 align-middle me-1"></i> Logout</a>
     </li>
    @else
    <li><a href="#">Login / Signup</a></li>
@endif
 ```

 # How to send otp

 ### AuthOtpController file

 ```
 namespace App\Http\Controllers;

use App\Models\User;
use App\Models\VerificationCode;
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AuthOtpController extends Controller
{
    // Return View of OTP Login Page
    public function login()
    {
        return view('auth.otp-login');
    }

    // Generate OTP
    public function generate(Request $request)
    {
        # Validate Data
        $request->validate([
            'mobile' => 'required|exists:users,mobile'
        ]);

        # Generate An OTP
        $verificationCode = $this->generateOtp($request->mobile);
        
        // code for sending otp start
         $msg = "The Secret OTP for login to your Businessodisha Account is $verificationCode->otp. Valid only for 3 mins.Do not share this OTP with anyone. GRAPHEMETECH SERVICES";
         VerificationCode::sendSms($request->mobile,$msg);
         // code for sending otp end

        $message = "Your OTP To Login is - ".$verificationCode->otp;
        # Return With OTP 

        return redirect()->route('otp.verification', ['user_id' => $verificationCode->user_id])->with('success',  $message); 
    }

    public function generateOtp($mobile_no)
    {
        $user = User::where('mobile', $mobile_no)->first();

        # User Does not Have Any Existing OTP
        $verificationCode = VerificationCode::where('user_id', $user->id)->latest()->first();

        $now = Carbon::now();

        if($verificationCode && $now->isBefore($verificationCode->expire_at)){
            return $verificationCode;
        }
        $otp =  rand(123456, 999999);
        // Create a New OTP
        return VerificationCode::create([
            'user_id' => $user->id,
            'otp' => $otp,
            'expire_at' => Carbon::now()->addMinutes(3)
        ]);
        
    }

    public function verification($user_id)
    {
        return view('auth.otp-verification')->with([
            'user_id' => $user_id
        ]);
    }

    public function loginWithOtp(Request $request)
    {
       
        #Validation
        $request->validate([
            'user_id' => 'required|exists:users,id',
            'otp' => 'required'
        ]);

        #Validation Logic
        $verificationCode   = VerificationCode::where('user_id', $request->user_id)->where('otp', $request->otp)->first();

        $now = Carbon::now();
        if (!$verificationCode) {
            return redirect()->back()->with('error', 'Your OTP is not correct');
        }elseif($verificationCode && $now->isAfter($verificationCode->expire_at)){
            return redirect()->route('otp.login')->with('error', 'Your OTP has been expired');
        }

        $user = User::whereId($request->user_id)->first();

        if($user){
            // Expire The OTP
            $verificationCode->update([
                'expire_at' => Carbon::now()
            ]);

            Auth::login($user);

            return redirect('/');
        }

        return redirect()->route('otp.login')->with('error', 'Your Otp is not correct');
    }
}
```
### VerificationCode.php (model)
```
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class VerificationCode extends Model
{
    use HasFactory;
    protected $fillable = ['user_id', 'otp', 'expire_at'];

    public static function sendSms($mobile,$msg){
        $sender_id = 'GRAPHM';
        $template_id = '1707168414494793391';
        $phone = $mobile;
        $msg = $msg;
        $username = 'digitalrath_admin';
        $apikey = 'DD9AE-0CF92';
        $uri = 'https://www.alots.in/sms-panel/api/http/index.php';
        $data = array(
        'username'=> $username,
        'apikey'=> $apikey,
        'apirequest'=>'Text',
        'sender'=> $sender_id,
        'route'=>'TRANS',
        'format'=>'JSON',
        'message'=> $msg,
        'mobile'=> $phone,
        'TemplateID' => $template_id,
        );

        $ch = curl_init($uri);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_FAILONERROR, true);
        curl_setopt($ch, CURLOPT_TIMEOUT, 0);
        curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 0);
        curl_setopt($ch, CURLOPT_NOSIGNAL, 1);
        $resp = curl_exec($ch);
        $error = curl_error($ch);
        curl_close ($ch);
        echo json_encode(compact('resp', 'error'));

    }
}
```
### view pages
> /home/projectssecurecl/public_html/laravel-sms-demo/resources/views/auth/otp-login.blade.php
```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('OTP Login') }}</div>

                <div class="card-body">

                    @if (session('error'))
                    <div class="alert alert-danger" role="alert"> {{session('error')}} 
                    </div>
                    @endif

                    <form method="POST" action="{{ route('otp.generate') }}">
                        @csrf

                        <div class="row mb-3">
                            <label for="mobile" class="col-md-4 col-form-label text-md-end">{{ __('Mobile No') }}</label>

                            <div class="col-md-6">
                                <input id="mobile" type="text" class="form-control @error('mobile') is-invalid @enderror" name="mobile" value="{{ old('mobile') }}" required autocomplete="mobile" autofocus placeholder="Enter Your Registered Mobile Number">

                                @error('mobile')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{{ $message }}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>



                        <div class="row mb-0">
                            <div class="col-md-8 offset-md-4">
                                <button type="submit" class="btn btn-primary">
                                    {{ __('Generate OTP') }}
                                </button>

                                @if (Route::has('login'))
                                    <a class="btn btn-link" href="{{ route('login') }}">
                                        {{ __('Login With Username') }}
                                    </a>
                                @endif
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
>/home/projectssecurecl/public_html/laravel-sms-demo/resources/views/auth/otp-verification.blade.php
```
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">{{ __('OTP Login') }}</div>

                <div class="card-body">
                    @if (session('success'))
                    <div class="alert alert-success" role="alert"> {{session('success')}} 
                    </div>
                    @endif

                    @if (session('error'))
                    <div class="alert alert-danger" role="alert"> {{session('error')}} 
                    </div>
                    @endif

                    <form method="POST" action="{{ route('otp.getlogin') }}">
                        @csrf
                        <input type="hidden" name="user_id" value="{{$user_id}}" />
                        <div class="row mb-3">
                            <label for="mobile_no" class="col-md-4 col-form-label text-md-end">{{ __('OTP') }}</label>

                            <div class="col-md-6">
                                <input id="otp" type="text" class="form-control @error('otp') is-invalid @enderror" name="otp" value="{{ old('otp') }}" required autocomplete="otp" autofocus placeholder="Enter OTP">

                                @error('otp')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{{ $message }}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>



                        <div class="row mb-0">
                            <div class="col-md-8 offset-md-4">
                                <button type="submit" class="btn btn-primary">
                                    {{ __('Login') }}
                                </button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
### How to create column using migration command
```
php artisan migrate --path='./database/migrations/2023_06_07_062102_add_price_to_table.php
```


### How to restict numbers inline onkeypress
```
<input type="text" onkeypress="return event.charCode >= 48 && event.charCode <= 57" />

```
### if you want to allow dot  and numeric only then code will be
```
onkeypress="return event.charCode >= 48 && event.charCode <= 57 || event.charCode === 46"
```

### How to display dynamic accordion

```
@php
                                    if($service_info->question)
                                    {
                                    $qa_arr = explode(',',$service_info->question);
                                    $ans_arr = explode(',',$service_info->answer);
                                    }
                                @endphp
                                <div id="main">
                                    <div class="accordion" id="faq">
                                    @if (count($qa_arr) > 0)
                                @foreach ($qa_arr as $index => $qa)
                                <div class="card">
                                            <div class="card-header" id="faqhead{{ $index}}">
                                                <a href="#" class="btn btn-header-link" data-toggle="collapse"
                                                    data-target="#faq{{ $index}}" aria-expanded="true"
                                                    aria-controls="faq{{ $index}}">{{ $qa }}</a>
                                            </div>

                                            <div id="faq{{ $index}}" class="collapse @if($index == 0) show @endif" aria-labelledby="faqhead{{ $index}}"
                                                data-parent="#faq{{ $index}}">
                                                <div class="card-body">
                                                {{ $ans_arr[$index] }}
                                                </div>
                                            </div>
                                        </div>
                                
                                @endforeach
                                @else
                                <p>No questions available.</p>
                                @endif
                                        
```
### There are two tables product and category  product table is reference table for category table then what will be the relation between them in laravel 8

In Laravel 8, you can define a relationship between the Product and Category tables using Eloquent ORM. Based on your description, it seems like a "one-to-many" relationship, where a category can have multiple products, but a product belongs to only one category.

To establish this relationship, you can define the following methods in your respective Product and Category model classes:

Product Model (Product.php):
```
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // ...

    public function category()
    {
        return $this->belongsTo(Category::class);
    }
}

```
Category Model (Category.php):

```
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    // ...

    public function products()
    {
        return $this->hasMany(Product::class);
    }
}
```
In the Product model, the category() method defines a belongsTo relationship, indicating that a product belongs to a category.

In the Category model, the products() method defines a hasMany relationship, specifying that a category can have multiple products.

Once you have defined these relationships, you can access them in your code. For example, to retrieve the category of a product or the products belonging to a category, you can use the following:
```
// Retrieve the category of a product
$product = Product::find(1);
$category = $product->category;

// Retrieve the products belonging to a category
$category = Category::find(1);
$products = $category->products;
```
These examples assume that you have proper database migrations for both the products and categories tables with appropriate foreign key constraints.

By utilizing these relationships, you can easily access related data between the Product and Category models in your Laravel application.

how to dispaly the category name in blade view

To display the category name in a Blade view, assuming you have set up the relationship between the Product and Category models as mentioned earlier, you can use the category relationship to access the category name. Here's an example:
```
@foreach ($products as $product)
    <div>
        <h3>Product: {{ $product->name }}</h3>
        <p>Category: {{ $product->category->name }}</p>
    </div>
@endforeach
```
> In this example, assuming you have a variable $products containing a collection of products passed to the Blade view, the @foreach loop iterates over each product. Inside the loop, you can access the category name using the $product->category->name syntax, where category represents the relationship method defined in the Product model, and name represents the column name that holds the category name.

> By using $product->category->name, you can display the category name for each product in your Blade view. Make sure to adjust the variable names and property names according to your specific code structure.
