#### Number of Rating

```php
<div class="">

   @if ($testimonial->star)
   @for ($i = 1; $i <= $testimonial->star; $i++)
   <i class="fa-solid fa-star text-[#fec624]"></i>
   @endfor  

   @for ($j = $testimonial->star+1; $j <= 5; $j++)
   <i class="fa-solid fa-star"></i>
   @endfor

   @endif
</div>
```

#### Percentage of Rating

```html
//html
<div class="">

	@php
		$retingPer = $testimonial->star/5*100
	@endphp

	<div class="star-ratings">
		<div class="fill-ratings text-[#fec624]" style="width: {{$retingPer}}%;">
			<span>★★★★★</span>
		</div>
		<div class="empty-ratings block">
			<span>★★★★★</span>
		</div>
	</div>
</div>
```

```css
.star-ratings {
  unicode-bidi: bidi-override;
  color: #111;
  font-size: 34px;
  position: relative;
  margin: 0;
  padding: 0;
 }

.empty-ratings {
    padding: 0;
    display: block;
    z-index: 0;
  }

.fill-ratings {
  color: #e7711b;
  padding: 0;
  position: absolute;
  z-index: 1;
  display: block;
  top: 0;
  left: 0;
  overflow: hidden;
}

span {
  display: inline-block;
}
```

```js
$(document).ready(function() {
  var star_rating_width = $('.fill-ratings span').width();
  $('.star-ratings').width(star_rating_width);
});
```