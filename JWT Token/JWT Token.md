1. Install the required packages:

```
composer require firebase/php-jwt
```

2. Open .`env` file & add this code

```
JWT_KEY=123XYSPOHB6794WLkp
```

3. Create a Helper folder inside App folder, then inside Helper folder create new file `JWTToken.php `
open file: APP/Helper/`JWTToken.php`

```php
namespace App\Helper;
use Firebase\JWT\JWT;
use Firebase\JWT\Key;

class JWTToken
{
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

4. Create login functionality inside `UserController.php` file:

```php
function userLogin(Request $request)
    {
        $user = User::where('email','=',$request->input('email'))
                    ->where('password', '=', $request->input('password'))
                    ->count();
                    
        if($user == 1){
            $token = JWTToken::CreateToken($request->input('email'));
            return response()->json([
                'status' => 'success',
                'message' => 'user login successfully',
                'token' => $token
            ],200);
        }
        else{
            return response()->json([
                'status' => 'failed',
                'message' => 'unauthorized'
            ],500);
        }
    }
```

5. Create Route:

```
Route::post('/user-login',[UserController::class,'userLogin']);
```



