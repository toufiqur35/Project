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

#### **Step 9: Make views.**

Create an registration page. **views/auth/registration.blade.php**