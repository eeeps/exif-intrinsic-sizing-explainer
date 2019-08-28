# Content-DPR explainer

By default, browsers display images at their [“density-corrected intrinsic size”](https://html.spec.whatwg.org/multipage/images.html#density-corrected-intrinsic-width-and-height). [An 800×600, 1x image will display at 800×600 CSS `px`](https://codepen.io/eeeps/pen/mdbmbPq). [A 1600×1200, 2x image will *also* display at 800×600 CSS `px`](https://codepen.io/eeeps/pen/mdbmbEq). Even though the two images have different resource dimensions, as far as layout is concerned, these they are identically-sized.

The `Content-DPR` response header gives servers control over the density part of that equation, allowing them to serve arbitrarily-scaled responses without affecting (and breaking) layouts.

## Example

Let’s say a server sends `400x300.jpg` with the following header:

```
Content-DPR: 0.5
```

This instructs the user agent to give the image an intrinsic density of 0.5x. Thus, the image will have a density-corrected intrinsic size of 800×600 CSS `px`, no matter what markup or CSS delivers it.

## Why?

`Content-DPR` was invented in conjunction with [the `DPR` `Viewport-Width`, and `Width` client hints](https://whatpr.org/html/3774/e32a6f8...ddb0544/images.html#image-related-client-hints-request-headers), so that servers could respond to these hints without breaking layouts, performing [device-pixel-ratio-based selection](http://usecases.responsiveimages.org/#device-pixel-ratio-based-selection), [viewport-size-based selection](http://usecases.responsiveimages.org/#viewport-based-selection), and [resolution-based selection](http://usecases.responsiveimages.org/#resolution-based-selection) on the server-side.

Notably, the header solves other important use cases, all on its own.

But first—

## The Client Hints use case 

Let's say a page author has chosen to craft a variable-device-pixel-ratio responsive image, with `srcset`.

Their markup might look like this:

```html
<img
	src="small.jpg"
	srcset="medium.jpg 2x,
	        large.jpg  3x"
	alt="A DPR-responsive JPEG"
/>
```

No matter which resource the browser selects, the density-corrected intrinsic size of the `<img>` will always be the same.

Here’s the equivalent Client Hints markup:

```html
<img
	src="client-hints.jpg"
	alt="A DPR-responsive JPEG"
/>
```

Let's say this request goes out from [a 2.6x device](https://vizdevices.yesviz.com/devices/google-pixel2/), with the following hint:

```
DPR: 2.6
```

But the server only has 1x, 2x, and 3x versions readily available. Reasonably, it responds with the 2x version, along with the following header:

```
Content-DPR: 2
```

This ensures that, when the image arrives, the client knows its density, and it is given proper, consistent, density-corrected intrinsic dimensions.

## Other use cases

But even in browsers that don’t implement the `Width`, `DPR`, and `Viewport-Width` request hints, support for the `Content-DPR` response header would open up new opportunities for image servers (and CDNs) to perform image optimization via resizing, without risking breaking layouts that rely on images’ default (intrinsic) size.

For example:

### Capping useful resolution

If a server receives a request for a very-large image, from a user agent that it knows does not have enough device pixels to take advantage of all of that resolution, it might reasonably choose to send a smaller version of the image. Currently, there is no way to do that without affecting the image’s intrinsic dimensions, potentially breaking layouts. `Content-DPR` would solve this.

### Prioritizing speed over quality

If the server has been given some signal that the user is in a constrained-bandwidth context (perhaps [an explicit signal, like `Save-Data: on`](https://wicg.github.io/netinfo/#save-data-request-header-field), or, perhaps an inferred one, based on measurements of previous response deliveries), it might want to respond with a down-scaled, low-resolution image. Again, this is impossible without either risking breaking some layouts, or universal support for `Content-DPR`.

### Low-quality image placeholders

Authors [commonly](https://jmperezperez.com/more-progressive-image-loading/) serve up [low-quality versions of images as placeholders](https://www.guypo.com/introducing-lqip-low-quality-image-placeholders), so that *something* is visible immediately, even if the full image load has been intentionally deferred (via lazy-loading), or is simply slow.

Right now, these techniques require some work on the front end to ensure that the downscaled placeholder is stretched to match the final dimensions of the full image.

`Content-DPR` would allow servers to implement foolproof features that could respond to requests, for, say,

`https://give.me/the-img.jpg?placeholder=true`

with downscaled images that behaved just like their full-scale counterparts, as far as layout is concerned.




