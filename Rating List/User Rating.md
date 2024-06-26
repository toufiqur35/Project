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