* Html section
```html
<div class="hero_slider">
	<div class="slideshow-container">
		<div class="mySlides fade">
			<img class="w-[100%] lg:h-[50vh] h-[20vh]" src="./img/1.jpg" alt="" />
		</div>
		
		<div class="mySlides fade">
			<img class="w-[100%] lg:h-[50vh] h-[20vh]" src="./img/2.jpg" alt="" />
		</div>
		
		<div class="mySlides fade">
			<img class="w-[100%] lg:h-[50vh] h-[20vh]" src="./img/3.png" alt="" />
		</div>
	</div>

	<div style="text-align: center" class="hidden">
		<span class="dot"></span>
		<span class="dot"></span>
		<span class="dot"></span>
	</div>
	<div class="overlay2 w-[100%] lg:h-[50vh] h-[20vh] text-white text-center lg:pt-32 pt-16">sobuj</div>
</div>
```

* CSS Section
```css
.overlay2 {
	position: absolute;
	top: 3.5rem;
	background-color: rgba(0, 0, 0, 0.5);
}
.slideshow-container {
	position: relative;
	margin: auto;
}
/* The dots/bullets/indicators */
.dot {
	height: 15px;
	width: 15px;
	margin: 0 2px;
	background-color: #bbb;
	border-radius: 50%;
	display: inline-block;
	transition: background-color 0.6s ease;
}
.active {
	background-color: #717171;
}
/* Fading animation */
.fade {
	animation-name: fade;
	animation-duration: 1.5s;
}

@keyframes fade {
from {
	opacity: 0.4;
}
to {
	opacity: 1;
}
}

/* On smaller screens, decrease text size */
@media only screen and (max-width: 300px) {
.text {
	font-size: 11px;
}
}
```

* Java Script Section
```js
let slideIndex = 0;
showSlides();
function showSlides() {
	let i;
	let slides = document.getElementsByClassName("mySlides");
	let dots = document.getElementsByClassName("dot");
	for (i = 0; i < slides.length; i++) {
	slides[i].style.display = "none";
	}
	slideIndex++;
	if (slideIndex > slides.length) {
	slideIndex = 1;
	}
	for (i = 0; i < dots.length; i++) {
	dots[i].className = dots[i].className.replace(" active", "");
	}
	slides[slideIndex - 1].style.display = "block";
	dots[slideIndex - 1].className += " active";
	setTimeout(showSlides, 5000); // Change image every 2 seconds
}
```
