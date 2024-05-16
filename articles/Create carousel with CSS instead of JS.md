Usually, when a client wanted to have a carousel on their website, I would just look for a JS library that creates one. But recently I tried things differently. I looked for "CSS based carousel" in Google and I found this [article](https://css-tricks.com/css-only-carousel/). This article shows few more examples what you can do without JS, but I decided to take just a core of it and add a little bit of JS on top, which to me is a nice in between that gives you nice carousel with grate performance. Additionally, this seems to me much more stable since core functionality is on the browser side and all we do is just some declarative CSS that describes what behavior we want.

So let's say we have this structure

```html
<div class="cmp-gallery-carousel">
	<div class="cmp-gallery-carousel__container">
		<div class="cmp-gallery-carousel__item"></div>
		<div class="cmp-gallery-carousel__item"></div>
		<div class="cmp-gallery-carousel__item"></div>
		<div class="cmp-gallery-carousel__item"></div>
	</div>
</div>
```

If we want to have carousel like behavior, you can use this CSS.

```css
.cmp-gallery-carousel .cmp-gallery-carousel__container {
  display: flex;

  overflow-x: auto;
  scroll-snap-type: x mandatory;

  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

.cmp-gallery-carousel .cmp-gallery-carousel__item {
  scroll-snap-align: start;
  flex-shrink: 0;
  display: flex;
}
```

This will make items (`cmp-gallery-carousel__item`) always snap to the left side of the container. So if we move scroll just a little bit and the first visible item is overflowing the container, after we let it go the browser will decide to which item we should snap (scroll to its start).

<img src="https://cms.filipizydorczyk.pl/api/v1/media/carusel-overview.gif">

This behavior is really similar to what TikTok or You Tube Shorts are doing. If we scroll up or down we, the app will snap to one of the items. So we can achieve the same with this CSS by changing to `scroll-snap-type: y mandatory;` and choosing flex direction to be vertical instead of horizontal.

<img src="https://cms.filipizydorczyk.pl/api/v1/media/yt-shorts-preview.gif">

Then you can add more CSS to `cmp-gallery-carousel__item` to customize how the items are displayed (size, color, image etc.).

Later, I decided to add very little JavaScript to make it behave more as carousels we already know.

```tsx

const GALLERY_CAROUSEL_SCROLL_BY_OFFSET = 200;
const ref = useRef<HTMLDivElement>(null);

const handleNextSlide = () => {
	if (!ref) {
		console.warn("Gallery Carusel refrence not found");
		return;
    }

    ref.current?.scrollBy(GALLERY_CAROUSEL_SCROLL_BY_OFFSET, 0);
};

const handlePreviousSlide = () => {
	if (!ref) {
		console.warn("Gallery Carusel refrence not found");
		return;
    }

    ref.current?.scrollBy(-GALLERY_CAROUSEL_SCROLL_BY_OFFSET, 0);
};

useEffect(() => {
	setInterval(handleNextSlie, 3000);
}, []);
```
*example from react project*

We have two main functionalities. `handleNextSlide` and `handlePreviousSlide` are functions that allow you to add left and right buttons to switch between slides. The way it works is JavaScript will scroll the container (`cmp-gallery-carousel__container`) scroll bar in one direction just a bit and the browser will snap it for you. No complicated calculations, and it works just like a regular JavaScript based carousel.

I also added `setInterval(handleNextSlie, 3000);` which is changing to the next slide every 3 seconds. You could also add some code that will stop from changing to next item if the mouse is over the container.