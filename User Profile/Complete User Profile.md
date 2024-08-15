#### **Step 1: Install Laravel**

First of all we need to get fresh Laravel version application using bellow command, So open your terminal OR command prompt and run bellow command:

```
composer create-project laravel/laravel example-app
```

#### **Step 2: Create migration & controller file**

```
php artisan make:migration create_users_table
php artisan make:controller UserController
```

#### **Step 3: Create Users Table 

```mysql
Schema::create('users', function (Blueprint $table) {
     $table->id();
     $table->string('firstName',50);
     $table->string('lastName',50);
     $table->string('email',50)->unique();
     $table->string('mobile',30);
     $table->string('password',100);
     $table->string('otp',10);
     $table->timestamps();
});
```

#### **Step 4: Database Configuration**

In this step, we need to add database configuration in .env file. so let's add following details and then run migration command:

```mysql
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=database_name
DB_USERNAME=root
DB_PASSWORD=
```

Next, run migration command to create users table.

```
php artisan migrate
```

#### **Step 5: Configuration User Model**
app/Model/`User.php`

```php
 protected $fillable = ['firstName','lastName','email','mobile','password','otp',];
 protected $attributes = ['otp' => '0'];
```

#### **Step 6: Route File**
routes/`web.php`

```php
// API Routes
Route::post('/user-registration',[UserController::class,'userRegistration']);
Route::post('/user-login',[UserController::class,'userLogin']);
Route::post('/user-otp',[UserController::class,'sendOTPCode']);
Route::post('/verify-otp',[UserController::class,'verifyOTP']);
Route::post('/reset-password',[UserController::class,'ResetPassword'])->middleware(TokenVerificationMiddleware::class);
// User Logout
Route::get('/logout',[UserController::class,'UserLogout'])->name('user.logout');

// Page Routes
Route::get('/user-registration',[UserController::class,'RegistrationPage'])->name('user.register');
Route::get('/user-login',[UserController::class,'LoginPage'])->name('user.login');
Route::get('/user-otp',[UserController::class,'sendOTPPage'])->name('user.otp');
Route::get('/verify-otp',[UserController::class,'verifyOTPPage'])->name('user.otp.verify');
Route::get('/reset-password',[UserController::class,'ResetPasswordPage'])->name('user.password.reset');
Route::get('/dashboard',[DashboardController::class,'dashboard'])->name('dashboard');
```
#### **Step 7: User Controller**

In this step, we need to create User Controller and add following code on that file:
app/Http/Controllers/`UserController.php`

```php
amespace App\Http\Controllers;
use Exception;
use App\Models\User;
use App\Mail\OTPMail;
use App\Helper\JWTToken;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Validator;

class UserController extends Controller
{
    // backend api view page
    function RegistrationPage()
    {
        return view('backend.auth.registration-page');
    }
    
    function LoginPage(){
        return view('backend.auth.login-page');
    }

    function ResetPasswordPage(){
        return view('backend.auth.reset-password-page');
    }
    
    function sendOTPPage(){
        return view('backend.auth.send-otp-page');
    }

    function verifyOTPPage(){
        return view('backend.auth.verify-otp-page');
    }


    // login API backend
    // user Registration
    function userRegistration(Request $request)
    {
        $validate = Validator::make($request->all(),[
            'firstName' => 'required',
            'lastName' => 'required',
            'mobile' => 'required',
            'email' => 'required|email|unique:users',
            'password' => 'required',
        ]);
        
        if($validate->fails()){
            return response()->json([
                'status' => 'failed',
                'message' => "Validation failed",
            ],401);
        }else{
            try{
                User::create([
                    'firstName' => $request->input('firstName'),
                    'lastName' => $request->input('lastName'),
                    'email' => $request->input('email'),
                    'mobile' => $request->input('mobile'),
                    'password' => $request->input('password'),
                ]);

                return response()->json([
                    'status' => 'success',
                    'message' => 'User registration successfully'
                ],200);
            }

            catch(Exception $e){
                return response()->json([
                    'status' => 'failed',
                    'message' => 'User registration failed'
                ],401);
            }
        }
    }

	// user Login
    function userLogin(Request $request)
    {
        $count=User::where('email','=',$request->input('email'))
            ->where('password','=',$request->input('password'))
            ->select('id')->first();

       if($count!==null){
           // User Login-> JWT Token Issue
           $token=JWTToken::CreateToken($request->input('email'),$count->id);
           return response()->json([
               'status' => 'success',
               'message' => 'User Login Successful',
           ],200)->cookie('token',$token,time()+60*24*30);
       }
       else{
           return response()->json([
               'status' => 'failed',
               'message' => 'unauthorized'
           ],200);
       }  
    }  

	// send OTP Code
    function sendOTPCode(Request $request)
    {
        $email = $request->input('email');
        $otp = rand(1000, 9999);
        $user = User::where('email','=',$email)->count();
        if($user == 1){
            // send mail
            Mail::to($email)->send(new OTPMail($otp));
            // update otp code in database
            $user = User::where('email','=',$email)->update(['otp' => $otp]);

            return response()->json([
                'status' => 'success',
                'message' => 'OTP code sent successfully'
            ],200);
        }
        else{
            return response()->json([
                'status' => 'failed',
                'message' => 'user not found'
            ],500);
        }
    }

	// verify OTP
    function verifyOTP(Request $request){
        $email = $request->input('email');
        $otp = $request->input('otp');
        $user = User::where('email','=',$email)->where('otp', '=', $otp)->count();

        if($user == 1){
            // update otp code in database
            $user = User::where('email','=',$email)->update(['otp' => 0]);
            // Pass reset token to user
            $token = JWTToken::CreateTokenForSetPassword($request->input('email'));
            return response()->json([
                'status' => 'success',
                'message' => 'OTP verified successfully',
            ],200)->cookie('token',$token,60*24*30);
        }
        else{
            return response()->json([
                'status' => 'failed',
                'message' => 'OTP verification failed'
            ],500);
        }
    }

	// reset password
    function ResetPassword(Request $request){
        try{
            $email = $request->header('email');
            $password = $request->input('password');
            User::where('email','=',$email)->update(['password' => $password]);
            return response()->json([
                'status' => 'success',
                'message' => 'Password Reset successfully'
            ],200);
        }

        catch(Exception $e){
            return response()->json([
                'status' => 'failed',
                'message' => 'Password Reset failed'
            ],401);
        }
    }

	// log out
    function UserLogout(){
        return redirect('/user-login')->cookie('token','',-1);
    }
}
```

#### **Step 8: JWT Token**

```
composer require tymon/jwt-auth
```

In this step, we need to create Helper file and add following code on that file:

app/Helper/`JWTToken.php`

```php
namespace App\Helper;
use Firebase\JWT\JWT;
use Firebase\JWT\Key;

class JWTToken
{
	// Create Token
    public static function CreateToken($userEmail,$userID):string{
        $key = env('JWT_KEY');
        $payload=[
            'iss' => 'laravel-token',
            'iat' => time(),
            'exp' => time()+60*60,
            'userEmail' => $userEmail,
            'userID' => $userID
        ];
        return JWT::encode($payload, $key, 'HS256');
    }

	// Create Token For Set Password
    public static function CreateTokenForSetPassword($userEmail):string{
        $key = env('JWT_KEY');
        $payload=[
            'iss' => 'laravel-token',
            'iat' => time(),
            'exp' => time()+60*5,
            'userEmail' => $userEmail
        ];
        return JWT::encode($payload, $key, 'HS256');
    }

  // Verify Token
    public static function VerifyToken($token):string
    {
        try{
            $key = env('JWT_KEY');
            $decoded = JWT::decode($token, new Key($key, 'HS256'));
            return $decoded->userEmail;
        }
        catch(\Exception $e){
            return 'Unauthorized';
        }
    }
}
```

#### **Step 9: Create Middleware**

In this step, we need to create `TokenVerificationMiddleware` file and add following code on that file:
app/Http/Middleware/`TokenVerificationMiddleware.php`

```php
public function handle(Request $request, Closure $next): Response
    {
        $token = $request->cookie('token');
        $result = JWTToken::VerifyToken($token);
        if($result == "unauthorized"){
            return response()->json([
                'status' => 'failed',
                'message' => 'unauthorized'
            ],401);
        }

        else{
            $request->headers->set('email', $result);
            return $next($request);
        }
    }
```

#### **Step 10: OTP Mail**

Step-10.1: Open `.env` file

```php
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your@gmail.com
MAIL_PASSWORD=yourpassword
MAIL_ENCRIPTION=tls
MAIL_FROM_ADDRESS=”your@gmail.com”
MAIL_FROM_NAME=”${APP_NAME}”
```

Step-10.2 : Create Mail Folder

```
php artisan make:mail OTPMail
```

Step-10.3 : Open `App/Mail/OTPMail.php` file

```php
namespace App\Mail;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Mail\Mailables\Envelope;
use Illuminate\Queue\SerializesModels;

class OTPMail extends Mailable
{
    use Queueable, SerializesModels;

    public $otp;
    public function __construct($otp)
    {
        $this->otp = $otp;
    }
    
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'O T P Mail',
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'email.OTPMail',
        );
    }

    public function attachments(): array
    {
        return [];
    }
}
```

#### **Step 11: Make views.**

1. Mail: Create & Open `resource/view/email/OTPMail.blade.php` file

* This template sent to user email.

```html
<div style="font-family: Helvetica,Arial,sans-serif;min-width:1000px;overflow:auto;line-height:2">
    <div style="margin:50px auto;width:70%;padding:20px 0">
      <div style="border-bottom:1px solid #eee">
        <a href="" style="font-size:1.4em;color: #00466a;text-decoration:none;font-weight:600">Your Brand</a>
      </div>
      <p style="font-size:1.1em">Hi,</p>
      <p>Thank you for choosing Your Brand. Use the following OTP to complete your Sign Up procedures. OTP is valid for 5 minutes</p>
      <h2 style="background: #00466a;margin: 0 auto;width: max-content;padding: 0 10px;color: #fff;border-radius: 4px;">{{ $otp }}</h2>
      <p style="font-size:0.9em;">Regards,<br />Your Brand</p>
      <hr style="border:none;border-top:1px solid #eee" />
      <div style="float:right;padding:8px 0;color:#aaa;font-size:0.8em;line-height:1;font-weight:300">
        <p>Your Brand Inc</p>
        <p>1600 Amphitheatre Parkway</p>
        <p>California</p>
      </div>
    </div>
  </div>
```

2. Layout: Create & Open `resource/view/backend/layout/app.blade.php` file

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>X-Bakery</title>
    <link rel="stylesheet" href="{{ asset('css/progress.css') }}">
    <link rel="stylesheet" href="{{asset('css/toastify.min.css')}}">
    <link rel="stylesheet" href="{{asset('css/fontawesome.css')}}">
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
    <div id="loader" class="LoadingOverlay hidden">
        <div class="Line-Progress">
            <div class="indeterminate"></div>
        </div>
    </div>
    <div>
        @yield('content')
    </div>

    <script src="{{asset('js/toastify-js.js')}}"></script>
    <script src="{{asset('js/axios.min.js')}}"></script>
    <script src="{{asset('js/script.js')}}"></script>
</body>
</html>
```

3. Login: Create & Open `resource/view/backend/auth/login.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
    <section class="bg-gray-50">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
      <div class="w-[37%] bg-white rounded-lg shadow-xl md:mt-0 sm:max-w-md xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl sob font-bold leading-tight tracking-tight text-gray-900 md:text-2xl">Sign In</h1>
                <div class="space-y-4 md:space-y-4">
                    <div>
                        <label for="email" class="block mb-1 text-sm font-medium text-gray-900">Your email</label>
                        <input type="email" name="email" id="email" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User Email" required="">
                    </div>
                    <div>
                        <label for="password" class="block mb-1 text-sm font-medium text-gray-900">Password</label>
                        <input type="password" name="password" id="password" placeholder="User Password" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" required="">
                    </div>
                    <div class="flex items-center justify-between float-right">
                        <a href="{{route('user.otp')}}" class="text-sm font-medium text-primary-600 hover:underline">Forgot password?</a>
                    </div>
                    <button onclick="SubmitLogin()" type="submit" class="w-full text-white bg-gradient-to-r from-[#8523be] to-[#e5068a] hover:bg-gradient-to-l duration-500 font-medium rounded-lg text-sm px-5 py-2.5 text-center">Sign in</button>
                    <p class="text-sm font-light text-gray-500">
                        Don’t have an account yet? <a href="{{ route('user.register') }}" class="font-medium text-primary-600 hover:underline">Sign up</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
  </section>

<script>

    async function SubmitLogin() {
              let email=document.getElementById('email').value;
              let password=document.getElementById('password').value;
              if(email.length===0){
                  errorToast("Email is required");
              }
              else if(password.length===0){
                  errorToast("Password is required");
              }
              else{
                  showLoader();
                  let res=await axios.post("/user-login",{email:email, password:password});
                  hideLoader();
                  if(res.status===200 && res.data['status']==='success'){
                      window.location.href="/dashboard";
                      successToast('login successfully');
                  }
                  else{
                      errorToast(res.data['message']);
                  }
              }
      }
  </script>
  
@endsection
```

4. Registration: Create & Open `resource/view/backend/auth/registration.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
<section class="bg-gray-50">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
      <div class="w-[37%] bg-white rounded-lg shadow-xl md:mt-0 sm:max-w-md xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl sob font-bold leading-tight tracking-tight text-gray-900 md:text-2xl">Registration Form</h1>
                <div class="space-y-4 md:space-y-4">
                        <input type="text" name="firstName" id="firstName" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User First Name" required="">
                        <input type="text" name="lastName" id="lastName" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User Last Name" required="">
                        <input type="text" name="mobile" id="mobile" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User Mobile" required="">
                        <input type="email" name="email" id="email" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User Email" required="">
                        <input type="password" name="password" id="password" placeholder="User Password" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" required="">
                    <button onclick="onRegistration()" type="submit" class="w-full text-white bg-gradient-to-r from-[#8523be] to-[#e5068a] hover:bg-gradient-to-l duration-500 font-medium rounded-lg text-sm px-5 py-2.5 text-center">Sign in</button>
                    <p class="text-sm font-light text-gray-500">
                        alrady have an account yet? <a href="{{ route('user.login') }}" class="font-medium text-primary-600 hover:underline">Sign In</a>
                    </p>
                </div>
            </div>
        </div>
    </div>
  </section>
  

  <script>
  
        async function onRegistration() {
        let email = document.getElementById('email').value;
        let firstName = document.getElementById('firstName').value;
        let lastName = document.getElementById('lastName').value;
        let mobile = document.getElementById('mobile').value;
        let password = document.getElementById('password').value;

        if(firstName.length===0){
            errorToast('First Name is required')
        }
        else if(lastName.length===0){
            errorToast('Last Name is required')
        }
        else if(mobile.length===0){
            errorToast('Mobile is required')
        }else if(email.length===0){
            errorToast('Email is required')
        }
        else if(password.length===0){
            errorToast('Password is required')
        }
        else{
            showLoader();
            let res=await axios.post("/user-registration",{
                email:email,
                firstName:firstName,
                lastName:lastName,
                mobile:mobile,
                password:password
            })
            hideLoader();
            if(res.status===200 && res.data['status']==='success'){
                setTimeout(function (){
                    window.location.href='/user-login'
                },1000)
                successToast(res.data['message']);
            }
            else{
                errorToast(res.data['message'])
            }
        }
     }
  </script>
  
@endsection
```

5. Send Otp: Create & Open `resource/view/backend/auth/send_otp.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
<section class="bg-gray-50">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
      <div class="w-[37%] bg-white rounded-lg shadow-xl md:mt-0 sm:max-w-md xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl sob font-bold leading-tight tracking-tight text-gray-900 md:text-2xl">Send Otp</h1>
                <div class="space-y-4 md:space-y-4">
                    <input type="email" name="email" id="email" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User Email" required="">
                    <button onclick="VerifyEmail()" type="submit" class="w-full text-white bg-gradient-to-r from-[#8523be] to-[#e5068a] hover:bg-gradient-to-l duration-500 font-medium rounded-lg text-sm px-5 py-2.5 text-center">Send Otp</button>
                </div>
            </div>
        </div>
    </div>
  </section>

  <script>
    async function VerifyEmail() {
              let email=document.getElementById('email').value;
              if(email.length===0){
                  errorToast("Email is required");
              }
              else{
                  showLoader();
                  let res=await axios.post("/user-otp",{email:email});
                  hideLoader();
                  if(res.status===200 && res.data['status']==='success'){
                     sessionStorage.setItem('email', email);
                     window.location.href="/verify-otp";
                     successToast('login successfully');
                  }
                  else{
                      errorToast(res.data['message']);
                  }
              }
      }

  </script>
  @endsection
  
```

6. Verify Otp: Create & Open `resource/view/backend/auth/verify_otp.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
<section class="bg-gray-50">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
      <div class="w-[37%] bg-white rounded-lg shadow-xl md:mt-0 sm:max-w-md xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl sob font-bold leading-tight tracking-tight text-gray-900 md:text-2xl">verify Otp</h1>
                <div class="space-y-4 md:space-y-4">
                    <input type="text" name="otp" id="otp" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="User verify Otp" required="">
                    <button onclick="VerifyOtp()" type="submit" class="w-full text-white bg-gradient-to-r from-[#8523be] to-[#e5068a] hover:bg-gradient-to-l duration-500 font-medium rounded-lg text-sm px-5 py-2.5 text-center">Next</button>
                </div>
            </div>
        </div>
    </div>
  </section>

  <script>
  
async function VerifyOtp() {
        let otp = document.getElementById('otp').value;
        if(otp.length !==4){
           errorToast('Invalid OTP')
        }
        else{
            showLoader();
            let res=await axios.post('/verify-otp', {
                otp: otp,
                email:sessionStorage.getItem('email')
            })
            hideLoader();
            if(res.status===200 && res.data['status']==='success'){
                setTimeout(() => {
                    window.location.href='/reset-password'
                }, 1000);
                successToast(res.data['message'])
                sessionStorage.clear();
            }
            else{
                errorToast(res.data['message'])
            }
        }
    }
  </script>
@endsection
  
```

7. Reset Password: Create & Open `resource/view/backend/auth/reset_password.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
<section class="bg-gray-50">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
      <div class="w-[37%] bg-white rounded-lg shadow-xl md:mt-0 sm:max-w-md xl:p-0">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl sob font-bold leading-tight tracking-tight text-gray-900 md:text-2xl">Reset Password</h1>
                <div class="space-y-4 md:space-y-4">
                    <input id="password" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="New Password" class="form-control" type="password"/>
                    <input id="cpassword" class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2" placeholder="Confirm Password" class="form-control" type="password"/>
                    <button onclick="ResetPass()" class="w-full text-white bg-gradient-to-r from-[#8523be] to-[#e5068a] hover:bg-gradient-to-l duration-500 font-medium rounded-lg text-sm px-5 py-2.5 text-center">Next</button>
                </div>
            </div>
        </div>
    </div>
  </section>

<script>
  async function ResetPass() {
        let password = document.getElementById('password').value;
        let cpassword = document.getElementById('cpassword').value;
        if(password.length===0){
            errorToast('Password is required')
        }
        else if(cpassword.length===0){
            errorToast('Confirm Password is required')
        }
        else if(password!==cpassword){
            errorToast('Password and Confirm Password must be same')
        }
        else{
          showLoader()
          let res=await axios.post("/reset-password",{password:password});
          hideLoader();
          if(res.status===200 && res.data['status']==='success'){
            window.location.href="/user-login";
          }
          else{
            errorToast(res.data['message'])
          }
        }
    }

</script>
@endsection
  
```

8. Dashboard: Create & Open `resource/view/backend/dashboard.blade.php` file

```html
@extends('backend.layout.app')
@section('content')
@extends('backend.layout.app')

<h1>dashboard page</h1>
<a href="{{route('user.logout')}}">Logout</a>

@endsection
```

8. Custom js: Create & Open `public/js/script.js` file

```js
function showLoader() {
  document.getElementById('loader').classList.remove('hidden')
}

function hideLoader() {
  document.getElementById('loader').classList.add('hidden')
}


function sucessToast(message) {
  Toastify({
    text: message,
    gravity: "bottom", // `top` or `bottom`
    position: "center", // `left`, `center` or `right`
    className: "mb-5",
    style: {
      background: "green",
    }
  }).showToast();
}

function errorToast(message) {
  Toastify({
    text: message,
    gravity: "bottom", // `top` or `bottom`
    position: "center", // `left`, `center` or `right`
    className: "mb-5",
    style: {
      background: "red",
    }
  }).showToast();
}
```