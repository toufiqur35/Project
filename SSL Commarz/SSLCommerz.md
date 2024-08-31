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