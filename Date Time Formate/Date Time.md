
```php
<td> {{ Carbon\Carbon::parse($appoinment->date)->format('d-M-Y' ) }}</td>
<td> {{ Carbon\Carbon::parse($appoinment->time)->format('g:i:s A' ) }}</td>
<td> {{ Carbon\Carbon::parse($item->created_at)->format('d-M-Y g:i:s A' ) }}</td>

link:https://gist.github.com/mdsami/ae7b3b3226e6f39df0c95698a03d743f
```
