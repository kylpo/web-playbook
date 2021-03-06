# Image Considerations for the Web

Images on the web are hard. Straight up. You need to make many decisions, it is so much more work than "just throw an image in".

[High Performance Images - By O'Reilly & Akamai](https://www.youtube.com/watch?v=UlrHvF_J-oI) is a good 1 minute video intro to some challenges.

Here are the major decisions you'll need to make
- What type is it?
- What should load?
- When should it load?
- How should it load?
- How should it fill its space?

## What type is it?
In the wonderful DX world of React Native, we need only reach for one tool: `<Image>`. But on the web, we have more to choose from: `<img>`, `<picture>`, and `background-image` of a `<div>`.

So, which one should we use? Well, let me answer this question with a question: is the image **content**?

If it is something that you'd like indexed by SEO, and visible if a user printed the page, then it is content. Often these images are stand-alone, like logos, images of a product on an e-commerce site, and images in a blog post. This is the world of `<img>`s and `<picture>`s. Exactly which of these two you'll use is decided in a future question.

If it is there to **enhance the display** of other content, like the background image of a header, then it is likely in the world of `<div>`s with `background-image` styles.

Note: this writeup will not cover `svg` and icons. I think it is different enough to break out into its own.

TODO: example picture with the two

I actually can't think of a case where I've put content on top of content images, so maybe it'd be easier to think of the separation as background vs foreground/content images.

TODO: image of decision tree


---


## What should load?
When a user loads your page, what should they see? Well, your image, right? Yes, but its not that easy.

### Responsive
If you serve a desktop sized retina image to a mobile user, you negatively affect TTI ([Time To Interactive](https://developers.google.com/web/tools/lighthouse/audits/time-to-interactive)) and page weight, which can seriously affect performance of animations, scrolling, interactivity, etc. On the other hand, if you serve your small mobile image to a 4k desktop user, s/he will surely insta-close that tab. So, we want to serve the smallest possible image that still looks good on the user's display. This is what is known as a **Responsive** Image.

To accomplish this, we'll need to do two things:
1. Create all permutations of image resolutions that you'll potentially serve
2. Tell the browser which permutation to load, and when

The first is either a lot of manual work in exporting, a decent amount of work in setting up a build process to auto-generate these, or a small amount of work and a little bit of money to have a 3rd part service do this work for you. More on this in the Tooling section

The second step depends on the type of image:
- Content -> use `srcset` of an `<img>` tag
  - [Harry Roberts on Twitter: "The simplest way I’ve found (so far) to distill/explain `srcset` and `sizes`"](https://twitter.com/csswizardry/status/836960832789565440)
  - ![](https://pbs.twimg.com/media/C517RYNWgAEqwy5.jpg)
  - [Responsive Images: If you're just changing resolutions, use srcset. | CSS-Tricks](https://css-tricks.com/responsive-images-youre-just-changing-resolutions-use-srcset/)
  - [The anatomy of responsive images - JakeArchibald.com](https://jakearchibald.com/2015/anatomy-of-responsive-images/)
- Style -> either use `@media` queries, or `image-set` if you aren't worried about the browsers lacking the feature
  - See [Responsive Images in CSS | CSS-Tricks](https://css-tricks.com/responsive-images-css/)

### Art Direction
What about going further than just resizing the same image depending on the viewport size? Maybe in a small viewport, you'd like to crop the image to zoom in and focus a specific part of it. Or perhaps you'd like a totally different image. This is **Art Direction**, and to accomplish it, we'll reach for the `<picture>` element for content, and `@media` queries for style.

```html
<picture>
   <source media="(min-width: 36em)"
      srcset="large.jpg  1024w,
         medium.jpg 640w,
         small.jpg  320w"
      sizes="33.3vw" />
   <source srcset="cropped-large.jpg 2x,
         cropped-small.jpg 1x" />
   <img src="small.jpg" alt="A rad wolf" />
</picture>
```
from [Responsive Images Done Right: A Guide To And srcset](https://www.smashingmagazine.com/2014/05/responsive-images-done-right-guide-picture-srcset/)

TODO: image of decision tree


---


## When should it load?
When a user visits your page, every single image is requested from a server (or local cache), then rendered on the page. This can be wasteful, and slows down TTI.

![](https://varvy.com/pagespeed/images/pageload.png)

Fortunately, there is a solve for this: load *above the fold* images first, then use a strategy below to handle the rest.

![](https://varvy.com/pagespeed/images/pageload-defer.png)

*images from [Defer images without jQuery or lazy loading](https://varvy.com/pagespeed/defer-images.html)*

### Strategy: Lazy Loading
"Lazy loading can significantly speed up loading on long pages that include many images *below the fold* by loading them either as needed or when the primary content has finished loading and rendering. In addition to performance improvements, using lazy loading can create infinite scrolling experiences."
- [Images | Web | Google Developers](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images)

Lazy Load monitors scroll position to load in images based on their proximity to the viewport (usually called a `threshold`)

### Strategy: Deferred Loading
Deferred will load every non-main image (usually below the fold) after page load.

#### How is this different from lazy loading?
Instead of monitoring the user's position to decide what should be loaded it, this strategy just loads every image (after the initial load), no matter the user's scroll position.

#### When is this the preferred solution?
Necessary for those long, anchor-linked, single page sites.

If it were Lazy Loaded, clicking an anchor link with an animated scroll would look janky (since the images would dynamically be load in). [Defer images without jQuery or lazy loading](https://varvy.com/pagespeed/defer-images.html) for more.

### Implementations
`lazySizes` plugins show how complicated this feature can be: https://github.com/aFarkas/lazysizes#available-plugins-in-this-repo

- [Building a Media Player #9: Lazy-Loading Images - YouTube](https://www.youtube.com/watch?v=ncYQkOrKTaI&feature=youtu.be)
  - uses `IntersectionObserver` instead of listening to `scroll`
- [Simple Image Lazy Load and Fade](https://davidwalsh.name/lazyload-image-fade)
- [lazysizes: High performance and SEO friendly lazy loader for images (responsive and normal), iframes and more, that detects any visibility changes triggered through user interaction, CSS or JavaScript without configuration.](https://github.com/aFarkas/lazysizes)
- [Building a high performance lazy load module – Front-end architecture by Robert Smith](http://rbrtsmith.com/2015/02/building-a-high-performance-lazy-load-module)
  - "Initially this sounds very simple, on page-load we can just grab the offsetTop of the images and store the values in the array, but upon further investigation this presents a problem. When we scroll to an image and it loads in, it pushes the content below further down the page, rendering the remaning offset values invalid. ... One benefit I forgot to mention of the placeholder is that once the image loads the page will not have to be repainted unlike before because the placeholder already takes up that space so another +1 for the performance"

### Considerations/Hurdles
#### Relies on Javascript
"What would happen if JavaScript failed or didn't load for whatever reason? (connectivity problems, bad 3G, JavaScript-blockers, third-party scripts fail... you name it)"

"In that case you need to use the noscript tag. Your final markup should look like:"

```jsx
<img class="lazy" data-original="image-src.jpg" width="1200" height="600">
<noscript>
    <img src="image-src.jpg" width="1200" height="600">
</noscript>
```
from comments section of https://www.sitepoint.com/lazy-loading-images-not-really-annoy-users/

Posts like http://dinbror.dk/blog/lazy-load-images-seo-problem/ argue that noscript is no longer needed, but bing was not able to index the images, so we're not totally there yet

#### SEO
For images that are not visible on page load, how would SEO index them?

Doesn't seem like it affects Google, but will affect any other search engines that do not run javascript (Bing)
- [Lazy Loading Images SEO Experiment -](http://www.theseotailor.com.au/blog/lazy-loading-images-seo-experiment/)
- [The lazy loading SEO problem, SOLVED! | dinbror](http://dinbror.dk/blog/lazy-load-images-seo-problem/)

To test your site, google `site:your-site.com`, then switch to the Images tab, like https://goo.gl/42KNZQ


#### Content jumping as images load in
padding-bottom https://www.smashingmagazine.com/2013/09/responsive-images-performance-problem-case-study/
The lazyload_images filter defers loading of images until they become visible in the client's viewport or the page's onload event fires. This avoids blocking the download of other critical resources necessary for rendering the above the fold section of the page.

- https://developers.google.com/speed/pagespeed/module/filter-lazyload-images


---


## How should it load?
After an image request has resolved, how should it appear on the screen? Based on its format, it may flash from nothing to visible, it may render top-down line by line (**progressive** JPEGs), or it may go from blurry to clear (**interlaced** GIF/PNG).

![](https://blog.codinghorror.com/content/images/uploads/2005/12/6a0120a85dcdae970b0128776fcab6970c-pi.gif) ![](https://blog.codinghorror.com/content/images/uploads/2005/12/6a0120a85dcdae970b0128776fcadb970c-pi.gif)

*images from [Progressive Image Rendering](https://blog.codinghorror.com/progressive-image-rendering/)*

Sure would be nice if we could have nice image renders based on the format alone, but saving them as *progressive* or *interlaced* adds to the file size, which is likely not worth it since we can show similar and better effects with `css` and `js`.

### Strategy: LQIP
load blurry LQIP (Low Quality Image Placeholder), then full
- [How Medium does progressive image loading - JMPerez Blog](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)
- [The technology behind preview photos | Engineering Blog | Facebook Code](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos)
  - ![](https://scontent.fsnc1-1.fna.fbcdn.net/v/t39.2365-6/11405147_157657891232533_1930722323_n.jpg?oh=c5ea1f9620d9aa33a369e6cd9d01c2b5&oe=59B7A9DB)
  - ![](https://scontent.fsnc1-1.fna.fbcdn.net/v/t39.2365-6/11405195_892704657466957_1372817644_n.jpg?oh=ee0fc03de3fc3461796c249270c6570a&oe=59863D96)
- [zouhir/lqip-loader: Low Quality Image Placeholders (LQIP) for Webpack](https://github.com/zouhir/lqip-loader)
  - [Zouhir ⚡ on Twitter: "🎉 just published lqip-loader for webpack ⚡️ fast blurry version of your image asset 🖼 load full image when you ready https://t.co/vrHsCZo52M https://t.co/0tEZ4EhTml"](https://twitter.com/_zouhir/status/867733179003580418)
- [technopagan/sqip: "SQIP" \(pronounced \\skwɪb\\ like the non\-magical folk of magical descent\) is a SVG\-based LQIP technique\.](https://github.com/technopagan/sqip?utm_source=frontendfocus&utm_medium=email)

### Strategy: Colored Background Placeholder
![](https://cdn-images-1.medium.com/max/800/0*QSSsKDNMDUCojsR6.gif)

[Pinterest’s Colored Background Placeholders – Embedly Notes](https://blog.embed.ly/pinterests-colored-background-placeholders-4b4c9fb8bb77)

### Strategy: Fade In
Just use good ol' css to animate `opacity` from `0` to `1` after the image has loaded.



---


## How should it fill its space?
Should it maintain aspect ratio? Should it be bound by a certain height/width? Can it stretch and still look good?

### Content
- [css - How to maintain aspect ratio using HTML IMG tag - Stack Overflow](http://stackoverflow.com/questions/12912048/how-to-maintain-aspect-ratio-using-html-img-tag)
  - "Don't set height AND width. Use one or the other and the correct aspect ratio will be maintained."
- [object-fit - CSS | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit?v=example)

### Style
- [Aspect Ratios in CSS are a Hack | Bram.us](https://www.bram.us/2017/06/16/aspect-ratios-in-css-are-a-hack/)
  - I particularly like the CSS Variables solution
- [background-size - CSS | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/background-size?v=example)

How to fill the entire page with an image, no white space, scales as needed, retains aspect ratio, and centered?

```
html {
  background: url(images/bg.jpg) no-repeat center center fixed;
  background-size: cover;
}
```

or use `background-image: url(_)`


---


## Performance considerations
### Tell the browser its starting size
- See Image section of [Twitter Lite and High Performance React Progressive Web Apps at Scale](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)
- [React Native - Images](https://facebook.github.io/react-native/docs/images.html#why-not-automatically-size-everything)
  - In the browser if you don't give a size to an image, the browser is going to render a 0x0 element, download the image, and then render the image based with the correct size. The big issue with this behavior is that your UI is going to jump all around as images load, this makes for a very bad user experience.
  - Image decoding can take more than a frame-worth of time. This is one of the major sources of frame drops on the web because decoding is done in the main thread. In React Native, image decoding is done in a different thread. In practice, you already need to handle the case when the image is not downloaded yet, so displaying the placeholder for a few more frames while it is decoding does not require any code change.
- `<img>`'s `sizes` is like the upfront, static form of react router. Pre-parser needs it to optimize performance, but it's a bummer. You'd rather just say that the responsive image is here, not find the right size to load (like RR v4).

### Render the exact size if possible
Even full loaded images can cause jank. When we had retina desktop images loaded on mobile, anchor scrolling and carousel scrolling were janky. This was because the browser does work to convert an images size if it is not the correct starting size. Reducing image size was the fix.

### Format and Compression
#### Which format to use?
- [Transparent JPG (With SVG) | CSS-Tricks](https://css-tricks.com/transparent-jpg-svg/)
- webp, transparent png, Progressive JPEG
- [Saving Bandwidth by Using Images the Smart Way — SitePoint](https://www.sitepoint.com/saving-bandwidth-by-using-images-the-smart-way/)
- [Performance Calendar » Squeezing PNG Images](https://calendar.perfplanet.com/2016/squeezing-png-images/)
- [Image Optimization | Web | Google Developers](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization#image-optimization-checklist)
- videos are way smaller than animated GIFs

#### How to deliver it?
```html
<picture>
  <source type="image/webp" srcset="snow.webp"/>
  <img alt="Hut in the snow" src="snow.jpg"/>
</picture>
```
from [The anatomy of responsive images - JakeArchibald.com](https://jakearchibald.com/2015/anatomy-of-responsive-images/) will deliver the `webp` if it is supported by the browser. Else it'll fall back to the `<img>`'s `jpg`.

### IMG Sprites
instead of loading multiple small images, consider spriting (single image with coordinates)

### Server considerations
- [Performance Calendar » Even Faster Images using HTTP2 and Progressive JPEGs](https://calendar.perfplanet.com/2016/even-faster-images-using-http2-and-progressive-jpegs/)


---


## Automation
- Build time solutions (great for a limited amount of images, but does not scale)
  - generate responsive permutations: [responsive-loader: A webpack loader for responsive images](https://github.com/herrstucki/responsive-loader)
  - compression: [image-webpack-loader: Image loader module for webpack](https://github.com/tcoopman/image-webpack-loader)
  - image sprite generation: [webpack-spritesmith: Webpack plugin that converts set of images into a spritesheet and SASS/LESS/Stylus mixins](https://github.com/mixtur/webpack-spritesmith)
- Separate process/task from the usual build
  - responsive and compression: [Performance Calendar » Image Optimization](https://calendar.perfplanet.com/2016/image-optimization/) - see the "Automating the asset creation process" section.
  - image sprite generation: [gulp.spritesmith: Convert a set of images into a spritesheet and CSS variables via gulp](https://github.com/twolfson/gulp.spritesmith)
- 3rd party services
  - [Cloudinary - Cloud image service, upload, storage & CDN](http://cloudinary.com/)
    - [10 Excellent Image Tricks and Enhancements with Cloudinary](https://davidwalsh.name/10-excellent-image-tricks-enhancements-cloudinary)
  - [Let The Content Delivery Network Optimize Your Images – Smashing Magazine](https://www.smashingmagazine.com/2017/04/content-delivery-network-optimize-images/)


---


## Concepts/Terms
- Responsive -> send the smallest asset for the screen that still looks good
- Art Direction -> more than just resizing, it is cropping (or replacing) for specific viewports
- Lazy Loading -> defer loading until last possible moment
- Deferred Loading -> asynchronously load in all images below the fold after page load
- Progressive JPEG -> image that renders top-down, line by line
- Interlaced GIF/PNG -> image that renders blurry, and progressively renders full image


---


## Other Links
- [Essential Image Optimization](https://images.guide/)
- [Images | Web | Google Developers](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images) good overview of many of these topics
- [An Event Apart News: Responsive Images Are Here. Now What? by Jason Grigsby—An Event Apart video](https://aneventapart.com/news/post/responsive-images-jason-grigsby-an-event-apart-video)
- [Saving Bandwidth with Images](https://medium.com/dev-channel/saving-bandwidth-with-images-8c28f52b5ef7)
- What type is it?
  - [html - When to use IMG vs. CSS background-image? - Stack Overflow](http://stackoverflow.com/questions/492809/when-to-use-img-vs-css-background-image?rq=1)
  - [React Native - Image](https://facebook.github.io/react-native/docs/image.html)
- Lazy Loading
  - [Lazyload Images](https://modpagespeed.com/doc/filter-lazyload-images)
  - [Five Techniques to Lazy Load Images for Website Performance — SitePoint](https://www.sitepoint.com/five-techniques-lazy-load-images-website-performance/)
