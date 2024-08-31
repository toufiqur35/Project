#### 1. Create Invoice Table

```
php artisan make:migration create_invoice_table
```

```php
Schema::create('invoices', function (Blueprint $table) {
            $table->id();
            $table->string('total', 50);
            $table->string('vat', 50);
            $table->string('payable', 50);
            $table->string('cus_details', 500);
            $table->string('ship_details', 500);
            $table->string('tran_id', 100);
            $table->string('val_id', 100)->default(0);
            $table->enum('delivery_status', ['pending', 'processing', 'completed']);
            $table->string('payment_status');
            $table->unsignedBigInteger('user_id');
            $table->foreign('user_id')->references('id')->on('users')->restrictOnDelete()->cascadeOnUpdate();
            $table->timestamps();
        });
```

#### 2. Create Invoice Products Table

```
php artisan make:migration create_invoices_products_table
```

```php
Schema::create('invoices_products', function (Blueprint $table) {
            $table->id();
            $table->string('qty', 50);
            $table->string('sale_price', 50);
            $table->unsignedBigInteger('invoice_id');
            $table->unsignedBigInteger('product_id');
            $table->unsignedBigInteger('user_id');
            $table->foreign('invoice_id')->references('id')->on('invoices')->cascadeOnUpdate()->restrictOnDelete();
            $table->foreign('product_id')->references('id')->on('products')->cascadeOnUpdate()->restrictOnDelete();
            $table->foreign('user_id')->references('id')->on('users')->cascadeOnUpdate()->restrictOnDelete();
            $table->timestamps();
        });
```

#### 3. Create `Sslcommerz` Accounts Table

```
php artisan make:migration create_sslcommerz_accounts_table
```

```php
Schema::create('sslcommerz_accounts', function (Blueprint $table) {
            $table->id();
            $table->string('store_id');
            $table->string('store_passwd');
            $table->string('currency');
            $table->string('success_url');
            $table->string('fail_url');
            $table->string('cancel_url');
            $table->string('ipn_url');
            $table->string('init_url');
            $table->timestamps();
        });
```

#### 4. Insert Data Into `Sslcommerz` Accounts Table

* Create `sslcommerz` send box account and insert this data.
* https://developer.sslcommerz.com/registration

```sql
INSERT INTO `sslcommerz_accounts` (`id`, `store_id`, `store_passwd`, `currency`, `success_url`, `fail_url`, `cancel_url`, `ipn_url`, `init_url`, `created_at`, `updated_at`) VALUES

(1, 'teamr64c9e84055219', 'teamr64c9e84055219@ssl', 'BDT', 'http://127.0.0.1:8000/PaymentSuccess', 'http://127.0.0.1:8000/PaymentFail', 'http://127.0.0.1:8000/PaymentCancel', 'http://127.0.0.1:8000/api/PaymentIPN', 'https://sandbox.sslcommerz.com/gwprocess/v4/api.php', '2023-08-25 21:35:23', '2023-08-25 21:35:23');
```

#### 5. Create User Model

```
php artisan make:model User
```

* app/Models/`User.php`

```php
 protected $fillable = [
        'email',
        'otp',
    ];

    public function profile()
    {
        return $this->hasOne(CustomerProfile::class);
    }
```

#### 6. Create Customer Profile Model

```
php artisan make:model CustomerProfile
```

* app/Models/`CustomerProfile.php`

```php
protected $fillable = [
        'cus_name',
        'cus_add',
        'cus_city',
        'cus_state',
        'cus_postcode',
        'cus_country',
        'cus_phone',
        'cus_fax',
        'ship_name',
        'ship_add',
        'ship_city',
        'ship_state',
        'ship_postcode',
        'ship_country',
        'ship_phone',
        'user_id',
    ];

    public function user():BelongsTo
    {
        return $this->BelongsTo(User::class);
    }
```
#### 7. Create Invoice Model

```
php artisan make:model Invoice
```

* app/Models/`Invoice.php`

```php
protected $fillable = [
        'total',
        'discount',
        'vat',
        'payable',
        'cus_details',
        'ship_details',
        'shipping_method',
        'tran_id',
        'delivery_status',
        'payment_status',
        'user_id'
    ];
```

#### 8. Create Invoice Product Model

```
php artisan make:model InvoiceProduct
```

* app/Models/`InvoiceProduct.php`

```php
protected $fillable = ['invoice_id', 'product_id', 'qty', 'sale_price','user_id'];

    public function product(): BelongsTo
    {
        return $this->belongsTo(Product::class);
    }
```

#### 9. Create `Sslcommerz` Account Model

```
php artisan make:model SslcommerzAccount
```

* app/Models/`SslcommerzAccount.php`

```php
class SslcommerzAccount extends Model
{
    use HasFactory;
}
```

#### 10. Create Invoice Controller

```
php artisan make:controller InvoiceController
```

* app/Http/controller/`InvoiceController.php`

```php
namespace App\Http\Controllers;
use Exception;
use App\Models\Invoice;
use App\Helper\SSLCommerz;
use App\Models\ProductCart;
use Illuminate\Http\Request;
use App\Helper\ResponseHelper;
use App\Models\InvoiceProduct;
use App\Models\CustomerProfile;
use Illuminate\Support\Facades\DB;

class InvoiceController extends Controller
{
    function InvoiceCreate(Request $request)
    {  
        DB::beginTransaction();
        try {
            $user_id=$request->header('id');
            $user_email=$request->header('email');
            $tran_id=uniqid();
            $delivery_status='Pending';
            $payment_status='Pending';
            $Profile=CustomerProfile::where('user_id','=',$user_id)->first();
            $cus_details="Name:$Profile->cus_name,Address:$Profile->cus_add,City:$Profile->cus_city,Phone: $Profile->cus_phone";
            $ship_details="Name:$Profile->ship_name,Address:$Profile->ship_add ,City:$Profile->ship_city ,Phone: $Profile->cus_phone";

            // Payable Calculation
            $total=0;
            $cartList=ProductCart::where('user_id','=',$user_id)->get();
            foreach ($cartList as $cartItem) {
                $total=$total+$cartItem->price;
            }
            
            $vat=($total*3)/100;
            $payable=$total+$vat;

            $invoice= Invoice::create([
                'total'=>$total,
                'vat'=>$vat,
                'payable'=>$payable,
                'cus_details'=>$cus_details,
                'ship_details'=>$ship_details,
                'tran_id'=>$tran_id,
                'delivery_status'=>$delivery_status,
                'payment_status'=>$payment_status,
                'user_id'=>$user_id
            ]);

            $invoiceID=$invoice->id;
            foreach ($cartList as $EachProduct) {
                InvoiceProduct::create([
                    'invoice_id' => $invoiceID,
                    'product_id' => $EachProduct['product_id'],
                    'user_id'=>$user_id,
                    'qty' =>  $EachProduct['qty'],
                    'sale_price'=>  $EachProduct['price'],
                ]);
            }
				           $paymentMethod=SSLCommerz::InitiatePayment($Profile,$payable,$tran_id,$user_email);
			 DB::commit();
			return ResponseHelper::Out('success',array(['paymentMethod'=>$paymentMethod,
			'payable'=>$payable,'vat'=>$vat,'total'=>$total]),200);
        }
        catch (Exception $e) {
            DB::rollBack();
            return ResponseHelper::Out('fail',$e,200);
        }
    }

    function InvoiceList(Request $request){
        $user_id=$request->header('id');
        return Invoice::where('user_id',$user_id)->get();
    }

    function InvoiceProductList(Request $request){
        $user_id=$request->header('id');
        $invoice_id=$request->invoice_id;
        return InvoiceProduct::where(['user_id'=>$user_id,'invoice_id'=>$invoice_id])->with('product')->get();
    }
  
    function PaymentSuccess(Request $request){
        SSLCommerz::InitiateSuccess($request->query('tran_id'));
        // return redirect('/profile');
    }

    function PaymentCancel(Request $request){
        SSLCommerz::InitiateCancel($request->query('tran_id'));
        // return redirect('/profile');
    }

    function PaymentFail(Request $request){
        return SSLCommerz::InitiateFail($request->query('tran_id'));
        // return redirect('/profile');
    }

    function PaymentIPN(Request $request){
        return SSLCommerz::InitiateIPN($request->input('tran_id'),$request->input('status'),$request->input('val_id'));
    }
}
```

#### 11. Create `SSLCommerz` Helper File 

* app/Helper/`SSLCommerz.php`

```php
namespace App\Helper;
use App\Models\Invoice;
use App\Models\SslcommerzAccount;
use Exception;
use Illuminate\Support\Facades\Http;

class SSLCommerz
{
   static function  InitiatePayment($Profile,$payable,$tran_id,$user_email):array
   {
      try{
          $ssl= SslcommerzAccount::first();
          $response = Http::asForm()->post($ssl->init_url,[
              "store_id"=>$ssl->store_id,
              "store_passwd"=>$ssl->store_passwd,
              "total_amount"=>$payable,
              "currency"=>$ssl->currency,
              "tran_id"=>$tran_id,
              "success_url"=>"$ssl->success_url?tran_id=$tran_id",
              "fail_url"=>"$ssl->fail_url?tran_id=$tran_id",
              "cancel_url"=>"$ssl->cancel_url?tran_id=$tran_id",
              "ipn_url"=>$ssl->ipn_url,
              "cus_name"=>$Profile->cus_name,
              "cus_email"=>$user_email,
              "cus_add1"=>$Profile->cus_add,
              "cus_add2"=>$Profile->cus_add,
              "cus_city"=>$Profile->cus_city,
              "cus_state"=>$Profile->cus_city,
              "cus_postcode"=>"1200",
              "cus_country"=>$Profile->cus_country,
              "cus_phone"=>$Profile->cus_phone,
              "cus_fax"=>$Profile->cus_phone,
              "shipping_method"=>"YES",
              "ship_name"=>$Profile->ship_name,
              "ship_add1"=>$Profile->ship_add,
              "ship_add2"=>$Profile->ship_add,
              "ship_city"=>$Profile->ship_city,
              "ship_state"=>$Profile->ship_city,
              "ship_country"=>$Profile->ship_country ,
              "ship_postcode"=>"12000",
              "product_name"=>"Apple Shop Product",
              "product_category"=>"Apple Shop Category",
              "product_profile"=>"Apple Shop Profile",
              "product_amount"=>$payable,
          ]);
          return $response->json('desc');
      }
      catch (Exception $e){
          return $ssl;
      }

    }


    static function InitiateSuccess($tran_id):int{
        Invoice::where(['tran_id'=>$tran_id,'val_id'=>0])->update(['payment_status'=>'Success']);
        return 1;
    }

    static function InitiateFail($tran_id):int{
       Invoice::where(['tran_id'=>$tran_id,'val_id'=>0])->update(['payment_status'=>'Fail']);
       return 1;
    }
    

    static function InitiateCancel($tran_id):int{
        Invoice::where(['tran_id'=>$tran_id,'val_id'=>0])->update(['payment_status'=>'Cancel']);
        return 1;
    }


    static function InitiateIPN($tran_id,$status,$val_id):int{
        Invoice::where(['tran_id'=>$tran_id,'val_id'=>0])->update(['payment_status'=>$status,'val_id'=>$val_id]);
        return 1;
    }
}
```

#### 12. Create Response Helper File (Optional)

* app/Helper/`ResponseHelper.php`

```php
namespace App\Helper;
use Illuminate\Http\JsonResponse;

class ResponseHelper
{
    public static function Out($msg,$data,$code): JsonResponse{
        return response()->json(['msg'=>$msg,'data'=>$data], $code);
    }
}
```


#### 13. Create Route

* routes/`werb.php`

```php
// User login Auth
Route::get('/login/{email}',[UserController::class,'UserLogin']);
Route::get('/veryfyLogin/{email}/{otp}',[UserController::class,'VerifyLogin']);
Route::get('/logout',[UserController::class,'UserLogout']);

// user profile
Route::post('/createProfile',[ProfileController::class,'createProfile'])->middleware(TokenAuthenticate::class);
Route::get('/readProfile',[ProfileController::class,'readProfile'])->middleware(TokenAuthenticate::class);

// Invoice and payment
Route::get("/InvoiceCreate",[InvoiceController::class,'InvoiceCreate'])->middleware([TokenAuthenticate::class]);
Route::get("/InvoiceList",[InvoiceController::class,'InvoiceList'])->middleware([TokenAuthenticate::class]);
Route::get("/InvoiceProductList/{invoice_id}",[InvoiceController::class,'InvoiceProductList'])->middleware([TokenAuthenticate::class]);

//payment success cancel and fail url

Route::post("/PaymentSuccess",[InvoiceController::class,'PaymentSuccess']);
Route::post("/PaymentCancel",[InvoiceController::class,'PaymentCancel']);
Route::post("/PaymentFail",[InvoiceController::class,'PaymentFail']);
```

#### 13. Create `Api` Route

* routes/`api.php`

```php
Route::post("/PaymentIPN",[InvoiceController::class,'PaymentIPN']);
```