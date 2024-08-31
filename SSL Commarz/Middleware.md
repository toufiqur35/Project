#### Token Authenticate

```php
namespace App\Http\Middleware;
use Closure;
use App\Helper\JWTToken;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;
  
class TokenAuthenticate
{
    public function handle(Request $request, Closure $next): Response

    {
        $token = $request->cookie('token');
        $result = JWTToken::ReadToken($token);
        if($result == 'unauthorized'){
            return redirect('/userLoginPage');
        }
        else{
            $request->headers->set('email',$result->userEmail);
            $request->headers->set('id',$result->userID);
            return $next($request);
        }
    }
}
```