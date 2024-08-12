1. Open `JWTToken.php` file and create a function as Create Token.
2. Just reduce the `exp` time to 5 min.

```php
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
```

#### Create Verify OTP function
* Open `UserController.php` file:

```php
function verifyOTP(Request $request){
        $email = $request->input('email');
        $otp = $request->input('otp');
        $user = User::where('email','=',$email)
                    ->where('otp', '=', $otp)->count();
        if($user == 1){
            // update otp code in database
            $user = User::where('email','=',$email)
                            ->update(['otp' => 0]);
            // Pass reset token to user
            $token = JWTToken::CreateTokenForSetPassword($request->input('email'));
            return response()->json([
                'status' => 'success',
                'message' => 'OTP verified successfully',
                'token' => $token
            ],200);
        }
        else{
            return response()->json([
                'status' => 'failed',
                'message' => 'OTP verification failed'
            ],500);
        }
    }
```