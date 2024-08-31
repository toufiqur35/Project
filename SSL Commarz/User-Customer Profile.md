#### 1. Create `users` Table

```
php artisan make:migration create_users_table
```

```php
Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('email',50)->unique();
            $table->string('otp',10);
            $table->timestamps();
        });
```

#### 2. Create `customer_profiles` Table

```
php artisan make:migration create_customer_profiles_table
```

```php
Schema::create('customer_profiles', function (Blueprint $table) {
            $table->id();
            $table->string('cus_name',100);
            $table->string('cus_add',500);
            $table->string('cus_city',50);
            $table->string('cus_state',50);
            $table->string('cus_postcode',50);
            $table->string('cus_country',50);
            $table->string('cus_phone',50);
            $table->string('cus_fax',50);

            $table->string('ship_name',100);
            $table->string('ship_add',100);
            $table->string('ship_city',100);
            $table->string('ship_state',100);
            $table->string('ship_postcode',100);
            $table->string('ship_country',100);
            $table->string('ship_phone',50);

            $table->unsignedBigInteger('user_id')->unique();
            $table->foreign('user_id')->references('id')
                    ->on('users')
                    ->restrictOnDelete()
                    ->cascadeOnUpdate();
            $table->timestamps();
        });
```


```php
namespace App\Http\Controllers;
use App\Models\User;
use App\Mail\OTPMail;
use App\Helper\JWTToken;
use Illuminate\Http\Request;
use App\Helper\ResponseHelper;
use Illuminate\Support\Facades\Mail;

class UserController extends Controller
{
    public function UserLogin(Request $request)
    {
        try{
            $userEmail = $request->email;
            $otp = rand(100000, 999999);
            $details = ['otp' => $otp];
            Mail::to($userEmail)->send(new OTPMail($details));
            User::updateOrCreate(['email'=> $userEmail],['email'=> $userEmail,'otp' => $otp]);
            return ResponseHelper::Out('success', 'A 6 digit OTP has been sent to your email', 200);
        }
        catch(\Exception $e){
            return ResponseHelper::Out('fail', $e, 500);
        }
    }

    public function VerifyLogin(Request $request)
    {
        try{
            $userEmail = $request->email;
            $otp = $request->otp;
            $varification = User::where('email', $userEmail)->where('otp', $otp)->first();
            if($varification){
                User::where('email', $userEmail)->where('otp', $otp)->update(['otp' => 0]);
                $token = JWTToken::CreateToken($userEmail, $varification->id);
                return ResponseHelper::Out('success', '', 200)->cookie('token', $token, 60*24*30);
            }
            else{
                return ResponseHelper::Out('fail', 'Invalid OTP', 401);
            }
        }
        catch(\Exception $e){
            return ResponseHelper::Out('fail', $e, 401);
        }
    }

    public function UserLogout()
    {
        return redirect('/userLoginPage')->cookie('token', '', -1);
    }
}
```