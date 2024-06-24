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
	Mail::to($toEmail)→send(new welcomeemail($message,$subject));
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

public function envelope():Envelope
{
	return new Envelope(
	subject: $this->subject,
	);
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

Step-6: Create blade file
resources/view/mail/`wellcome.blade.php`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
	<body>
		// j sokol data phatata cai sob dita hoba
	</body>
</html>
```

Step-7: Create Route

```php
Route::get(‘send-email’, [EmailController::class,’sendEmail’]);
```

### Send Email With Attachment file
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
public function sendEmail(Request $request)
{
	$toEmail = “yourcliend@gmail.com”;
	$message = “hello ,wellcome”;
	
	$file = $request->file('attachment');
	$file->move('uploads',$file->getClintOriginalName());
	Mail::to($toEmail)->send(new welcomeemail($message,$file));
	//Mail::to($toEmail)->send(new welcomeemail($request->all(),$file));
	// jodi from thaka data nita cai
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
// public $request;               jodi from thaka data nita cai
public $file;

public function _construct($message,$file)){
	$this->message = $message;
	$this->file = $file;
}
//jodi from thaka data nita cai
//public function _construct($request,$file)){
//	$this->request = $request;
//	$this->file = $file;
//}
public function envelope():Envelope
{
	return new Envelope(
		subject: 'any text',
	);
}

public function contend(){
	return new Content(
	view: ’ mail.wellcome’, /// resources/view/mail/wellcome.blade.php
	);
}

public function attachments():array
{
	$attachments = [];

	if($this->file){
		$attachments = [
			Attachment::fromPath(public_path('/uploads/'.$this->file))
		];
	}

	return $attachments;
}

}
```

Step-6: Create blade file
resources/view/mail/`wellcome.blade.php`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
	<body>
		// j sokol data phatata cai sob dita hoba
		//jodi from thaka data nita cai
		<p>name : {{$request[name]</p>
	</body>
</html>
```


Step-7: Create Route

```php
Route::get(‘send-email’, [EmailController::class,’sendEmail’]);
```


### Another
1. blade file

```html
<a href="{{ url('send_mail',$data->id) }}">send mail</a>
```

2. Route file

```php
Route::get('/send_mail/{id}',[yourcontroller::class,'sendMail'] );
```

3. Create the Controller

```php
php artisan make:controller MailController
```

4. Open the mail controller

```php
// App\http\controller\MailController

public function sendMail($id)
{
	$data = Contact::find($id);            // use any model name
	return view('mail.semd_mail',compact('data'));
}
```

5. Create the blade file name (`send_mail`)
```html
Resource\view\mail\send_mail.blade.php

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
	<body>
		<h1>{{ $data->name }}</h1>
		<h1>{{ $data->email }}</h1>
		<h1>{{ $data->phone }}</h1>
		<h1>{{ $data->address }}</h1>
		//more data
	</body>
</html>
```