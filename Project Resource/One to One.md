```mysql
$table->foreignId('carusel_id')
	->references('id')
	->on('carusels')
	->onDelete('cascade');
```