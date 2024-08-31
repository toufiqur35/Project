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