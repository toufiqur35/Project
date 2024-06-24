Step-1: Open `.env` file

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

Step-2:  Create the Controller

```php
php artisan make:controller EmailController
```

Step-3: open Email Controller file:
* App \\ Http\\ Controller\\`EmailController`

```php
public function sendEmail()
{
	$toEmail = “yourcliend@gmail.com”;
	$message = “hello ,wellcome”;
	$subject = “wellcome”;
	Mail::to($toEmail)→send(new welcomeemail($message,$subject);
}
```

Step-4: Create the Mail

```php
php artisan make:mail welcomeemail
```

Step-5: open (app/mail/`welcomeemail.php`)

```php
class welcomeemail extends mailable{
public $message;
public $subject;

public function _construct($message,$subject)){
	$this->message = $message;
	$this->subject = $subject;
}

public function envelope(){
	subject: $this->subject,
}

public function contend(){
	return new Content(
	view: ’ mail.wellcome’, /// resources/view/mail/wellcome.blade.php
	);
}

public function attachments(){

}

}
```

Step-6: Create Route

```php
Route::get(‘send-email’, [EmailController::class,’sendEmail’]);
```