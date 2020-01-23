[-] Title: Using the Intersection Observer API

[-] Summary: Learn to use the Intersection Observer API to efficiently detect when elements come into view and put that knowledge into action by using the Intersection Observer API to lazy-load images.

[-] Published: true

[-] Read Time: 30

===

**Ever needed to know if a particular element is visible to the user?** If you've tried to solve this problem before you probably used something like `getBoundingClientRect()` to get the elements location and dimensions and then cross-referenced these with window's width and height to hopefully figure out if the element is within the users visible viewport. 

Not only is this a pain to do it's really inefficient as a call `getBoundingClientRect()` or `clientWith` causes the browser to go through its layout process!

In this article I'll show you how **simple** it can be to do this using the **Intersection Observer API** a new Web API that makes it possible to efficiently detect when an element is inside (or outside) a particular area on a page.

I'll also provide a practical example of using the Intersection Observer API by showing you how you can **improve page load times** by lazy-loading images.

So let's jump right in and see how to use the Intersection Observer API.

## What is an Intersection?

**Before we go any further it'll help if we understand what is meant by an "intersection"?** It's pretty simple really. When we say "intersection" we mean that a target elements visibility has changed within a containing element based on the configuration we've provided.

~IMG:An intersection is when a target element changes its visibility within a container|https://firebasestorage.googleapis.com/v0/b/usetheplatform-tv.appspot.com/o/article_images%2FintersectionExample.png?alt=media&token=5834635d-16bd-462f-bf12-bf48fb246c00~

In the most basic case an intersection occurs when an element goes from being completely invisible within a containing element (or the browsers viewport) to being all or partically visible. We can then provide different options to change when we get told about the different intersections which we'll learn all about next 🤓.

## Intersection Observer API Basics

The API is really easy to use and it all starts by creating an observer instance.

```javascript
const observer = new IntersectionObserver((entries) => {
    // Callback
}, {
    root: null,
    threshold: 0.0,
    rootMargin: "0px 0px 0px 0px"
});
```

The first argument is the function to be executed when an intersection change occurs and the second argument is a set of optional options. Let's take a look at each of these options in detail.

### The 'root' option

~IMG:Use the root option to change the intersection container|https://firebasestorage.googleapis.com/v0/b/usetheplatform-tv.appspot.com/o/article_images%2FrootRender.png?alt=media&token=1a95bdee-c812-49a2-b8ea-c448be8dea56~

This is a reference to the element you want to consider the container. So if you had a `#box` element within a `#frame` element and wanted to observe whenever the box element passed the boundary of the `#frame` element then you'd set the `root` option to be a reference to the `#frame` element like so:

```js
{
    root: document.getElemenyById("frame")
}
```

If the `root` option is not specified or set to null the container will be the browsers viewport, handy if you want to detect if an element is visible anywhere on the page. 

### The 'threshold' option

~IMG:Use the threshold option to dictate when the intersection happens|https://firebasestorage.googleapis.com/v0/b/usetheplatform-tv.appspot.com/o/article_images%2FthresholdExample.png?alt=media&token=d98388ed-ca55-4113-9103-ad4128db50cf~

This option allows you to control when we're told about an intersection. By default the threshold is 0.0 which means an intersection will occur when the target elements visibility changes to or from 0%. By this I mean, if the element starts out on the page as 100% visible, the callback won't fire to inform us of an intersection change until 0% of the target element is visible. If the value is 1.0 then the callback won't get fired until the target elements visibility has changed to or from 100% within the configured container (In this example that's the document viewport). If we set this value to 0.5 then it will fire when the target elements visibility changes to or from 50%, setting a value of 0.25 would do the same for 25% visibility and so on. You can also specify an array of values if you want the callback to fire when say 25%, 75% and 100% of the element is visible like so: `threshold: [0.25, 0.75, 1.0]`.  

*Read more about [thresholds](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#Thresholds).*

### The 'rootMargin' option

~IMG:Use the rootMargin to change where the intersection happens|https://firebasestorage.googleapis.com/v0/b/usetheplatform-tv.appspot.com/o/article_images%2FrootMarginRender.png?alt=media&token=a99bb036-6db9-4994-a7e0-f97650631a58~

For added control there is also a `rootMargin` option which allows you to change when you're informed of an intersection based on the root element. Let's say you wanted to be informed of the intersection `10px` **before** the `#box` element actually passes the `#frame` boundary you'd use the `rootMargin` option like so: 

```js
{
    rootMargin: "-10px"
}
```

Or, if you wanted to to be informed of the intersection `10px` **after** the `#box` element passed the `#frame` element boundary you'd set the `rootMargin` to `10px`.

You can also specify the margins just like you would in CSS so something like `10px 5px 12px 20px` would also work!

*Read more about the root and rootMargin options [here](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#The_intersection_root_and_root_margin).*

### Observing an element

Okay, so how do we actually use this observer on an element then? Simple, we use the `observe()` method and specify a reference to an element.

```javascript
observer.observe(document.getElementById('cat'));
```

*It's as simple as that!* Now our callback will fire whenever the `#cat` element's intersection changes, lets look at how we can actually interface with this information.

```javascript
const observer = new IntersectionObserver(entries => {
    const entry = entries[0];
    if (entry.isIntersecting) {
        entry.target.style.filter = "invert(.8)";
    } else {
        entry.target.style.filter = "none";
    }
}, {
    root: null,
    threshold: 1.0
});
observer.observe(document.getElementById("cat"));
```

Because you can observe multiple elements with the same observer the entries argument to the callback is an array with all of the [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) instances for all of the observed elements. As we're only observing a single element in this example we can just assume there is a single entry.

An [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) has many useful properties but for this simple example we just want to know when the element is visible to the user. To do that we need to check if the `isIntersecting` property is `true` as the callback will get fired anytime the intersection changes, not just when it's within the specified root. Because we set our threshold to be `1.0`, `isIntersecting` will only be `true` when 100% of the element (in this case a picture of a cat) is within the browser viewport.

When this happens we're applying a CSS filter so we can visibly see this happening.

~GLITCH:https://glitch.com/embed/#!/embed/ioapi-basic-example?path=index.html&previewSize=100~

Try it above, as you scroll the page, when the cat image is 100% within the users viewport the CSS filter gets applied. When the cat image is either not visibile at all or is less than 100% visible the CSS filter is removed.

With this very basic understanding you can now easily implement some pretty neat features in your web apps. 

**Continue reading to find out how.**

## How to lazy-load images using the Intersection Observer API

A brilliant use case for the Intersection Observer API and a great way to improve page load times is lazy-loading images.

Lazy-loading an image is simply the act of deferring the loading of an image until the user is at a place on your page or app where they can actually see it. Why waste precious bandwidth loading images the user may never look at?

In this final section of this article I'll show you how simple this is with the **Intersection Observer API**.

~APV:https://firebasestorage.googleapis.com/v0/b/usetheplatform-tv.appspot.com/o/article_images%2Flazy_load_image_example.mp4?alt=media&token=ab35ea7c-69c0-4302-94e9-eada43e21ddc~

Imagine you've got a simple scrollable page with five large images. We only want to load the images which are initially visible within the users viewport and then we want to load the images which come into the users viewport as they scroll down the page. 

> This is a really simplistic example of the lazy-loading technique but it's enough to demonstrate how the Intersection Observer API can be used to achieve this functionality.

We'll start with the basic mark-up for the web page.

```html
<body>
    <div class="container">
      <img class="lazy" data-src="forest1.jpg" alt="Forest 1" width="960" height="720" />
      <img class="lazy" data-src="forest2.jpg" alt="Forest 2" width="960" height="720" />
      <img class="lazy" data-src="forest3.jpg" alt="Forest 3" width="960" height="720" />
      <img class="lazy" data-src="forest4.jpg" alt="Forest 4" width="960" height="720" />
      <img class="lazy" data-src="forest5.jpg" alt="Forest 5" width="960" height="720" />
    </div>
  </body>
```

We don't specify a `src` attribute on the `img` elements because we don't want them to load right away instead we use the `data-` prefix so we can pick up the image path later. We must also specify the image height and width for each image. This is so the elements intersect according to the source image dimensions, otherwise all empty image elements would be considered visible on the page all the time. 

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.onload = () => {
        entry.target.classList.add("loaded");  
        observer.unobserve(entry.target);
      };
      entry.target.setAttribute("src", entry.target.dataset.src);
    }
  });
}, {
    root: null,
    threshold: 0
});
```

The observer we create is pretty simple. As we **are** observing multiple elements this time we need to loop through each entry. When we come across an intersection (i.e. `entry.isIntersecting` is `true`) we add a `src` attribute to the target element (which will obviously be one of the `img` elements) setting its value to the image path specific via the`data-src` attribute, this will then load the image and display it. For an extra nice effect we add the `loaded` class to the element when the image has finished downloading, which we know thanks to the `onload` callback. Adding this class changes the opacity of the element and using CSS transitions fades it in.

```css
.lazy {
  margin: 1rem auto;
  display: block;
  opacity: 0;
  transition: opacity ease-in-out 300ms;
}
.lazy.loaded {
  opacity: 1;
}
```

We also have no need to continue observing the element once its loaded, we can't reload it so we  call the `unobserve()` method available on the observer instance to stop listening to intersection changes for elements we lazy-loaded.

```js
observer.unobserve(entry.target);
```

Finally we set the threshold to `0` as we want to be notified when any part of the `img` element is visible. 

**And that's it.** We now have a really simple and efficient way to defer the loading of images until the user can actually see them. 

~GLITCH:https://glitch.com/embed/#!/embed/ioapi-lazy-load?path=style.css&previewSize=100~

For a complete, drop-in solution check out my [Img-2 web component](https://github.com/RevillWeb/img-2), simply replace your existing `img` elements with `img-2` and all this gets taken care for you. You can even display a blurred preview of the image while the full size version is downloading! Checkout the [demo](https://revillweb.github.io/img-2/).

## Platform Support

~CIU:intersectionobserver~

**For unsupported platforms checkout this [polyfill](https://github.com/w3c/IntersectionObserver/tree/master/polyfill)!**

## Conclusion

The **Intersection Observer API** provides a simple and efficient way to detect when an element or elements are currently visible or invisible to the user. This capability makes it easier to implement features lazy-loading images which I've demonstrated in this article but also other features like infinite scroll, triggering animations, ad revenue calculation and many more.

I hope you learn't something from this article, please share your thoughts and experiments within the comments and if you liked it, please share - it really helps!
