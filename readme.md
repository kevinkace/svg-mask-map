# SVG Transparency Mask and Image Map

This is my first post of this nature. It describes combining a few different capabilities of SVG, namely transparency masks, CSS transitions, positioning elements within an SVG using CSS `transform`, and using `path`s for an image map.

I hope you enjoy it, and if you see any improvements (technical or otherwise) please comment.

## Project description

*Guild Wars 2* has a rich, blended, organic art style which can be difficult to incorporate into a website. There were a few tricky sections when developing the *Path of Fire* website. One specifically was the section for newly launched **mounts**!* It has a graphic of 4 mounts centered on a background, hovering on a mount shifts the "lighting", highlighting the hovered mount, and fading out the other mounts and background. Clicking on a mount opens detail of the mount but not descibed here.

Here's what the [final product](https://www.guildwars2.com/en/path-of-fire/#mounts) looks like:

<video autoplay loop style="max-width: 100%">
  <source src="https://fat.gfycat.com/MealyNippyCreature.webm" type="video/mp4">
</video>

It's similar to the UI on the [professions page](https://www.guildwars2.com/en/the-game/professions/), but ended up with a very different solution.

* The specializations section also used this feature, but no one wants to write or read "specializations" a hundred times throughout this article. TL;DR: I got double usage out of this technique.

## Unique problems

Other than the mounts graphic, the section has intro text above (localized into 4 languages), and navigation below, all on top of a fullbleed background image. To keep text length flexible, the mounts image had to be on a transparent background. Using transparent PNGs for the mounts images would be the conventional approach, and would look/work great, with one major detraction: file size. The mounts image was over 1200x600px, and there were 3 of them (base, highlighted, faded out).

Defining the hoverable region around a mount image is also a little tricky, as the mounts are irregularly shaped so a rectangular `div` wouldn't cut it. The professions section on GW2.com uses an [image `map`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/map) with mouse events added to `area`s, one for each profession. This actually works great, but I didn't reuse it here.

Here are the visual components:

todo: video of visual components

## Working towards a solution

The one part that had a clear solution get-go was fading the background. Transitioning opacity on a `div` with a background color filling the entire section would be simple and effective.

For the other problems, well... I've been using SVG more and more recently, and perhaps in a case of "everything is a nail when you have a hammer", I looked to SVG to solve the other problems, file size and hover regions.

### SVG transparancy mask

This is a simple technique that we've been using at ArenaNet for a while. I first saw it described in an article [Using SVG to Shrink Your PNGs](http://peterhrynkow.com/how-to-compress-a-png-like-a-jpeg/) in Sept 2014. The idea is to split a transparent image (PNG) into 2 opaque images, separating the transparancy data from the color data. The transparancy data is saved as a greyscale image, and works the same as a layer mask in Photoshop - black is transparent, white is opaque. These 2 actual images are fully opque meaning they can be compressed more easily.

Creating these 2 images todo

### SVG paths

An SVG `path` can define an irregular shape with a variety of line commands, including `curveto` which uses Bezier curves - this is a great fit for the hoverable regions.

I created these paths in Adobe Illustrator:

1. open one of the base PNGs in Illustrator
1. use the pen-tool to draw shapes around each mount
1. File > Export > Export as...
1. select SVG
1. check "Use Artbox"
1. click ok

I opened the SVG in a text editor, and copied the path strings to my component.

## Putting it all together

Here are all the assets for this feature:

- full bleed background image
- background overlay element
- image of all mounts faded out
- image of all mounts default state
- image of transparency mask
- 4 images, one of each mount highlighted
- paths for hoverable region

<video autoplay loop style="max-width: 100%">
  <source src="https://giant.gfycat.com/DependentSnoopyIrishwolfhound.webm" type="video/mp4">
</video>

Here's the DOM and SVG outline:

```html
<!-- 01. background & overlay -->
<section>
  <div></div>

  <svg>
    <!-- > 02. mask definition -->
    <defs>
      <mask id="mask">
        <image href="/grey-scale-alpha.jpg"></image>
      </mask>
    </defs>

    <!-- 03. default and faded out collage of all mounts -->
    <g mask="url(#mask)">
      <image href="/faded-out-mounts.jpg"/>
      <image href="/default-mounts.jpg"/>
    </g>

    <!-- 04. individual images of each mount highlighted -->
    <g>
      <image href="/mount-0-hightlight.png"></image>
      <image href="/mount-1-hightlight.png"></image>
      <image href="/mount-2-hightlight.png"></image>
      <image href="/mount-3-hightlight.png"></image>
    </g>

    <!-- 05. mouseover regions -->
    <g>
      <path d="/* path data for mount 0 */"></path>
      <path d="/* path data for mount 1 */"></path>
      <path d="/* path data for mount 2 */"></path>
      <path d="/* path data for mount 3 */"></path>
    </g>
  </svg>
</section>
```

*I excluded some SVG boilerplate attrs, but the [full code is below](#fullcode)*

### 00. Background & overlay

The full bleed background image is applied to the `section`, and the background overlay `div` is stretched to fill. When a mount is hovered, the `opacity` of the overlay `div` is transitioned to `1`, obscuring the full bleed background image behind.

todo: gif of just BG interaction

```html
<section class="mountsSection">
    <div class="bgFade"></div>
    <!-- more code -->
</section>
```

```css
.mountsSection {
  position: relative;
  background: url(./img/bg.jpg);
  background-size: cover;
}
.bgFade {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: rgba(255, 255, 255, 0.3);

  opacity: 0;
  transition: opacity 0.3s;
}
.bgFade-in {
  opacity: 1;
}
```

### 02. SVG mask

We start here with a [`def` element](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/defs), it's children aren't to be shown directly but will be referenced by ID later. Within the `def` there's a `mask`, and an `image` with `href` referring to the transparancy mask made earlier.

```html
<defs>
  <mask id="mountsMask">
    <image width="1269" height="654" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="transparancy-mask.jpg"></image>
  </mask>
</defs>
```

### 03. Images

Here we have 2 `image`s nested in a group `g`, and the transparency mask is applied to the group. The 2 images (and anything within the `g`) are masked, removing the background. The images are stacked on top of each other, the default image on top. Transitioning the `opacity` of the default image will reveal the faded out image behind.

```html
<g mask="url(#mask)">
  <image width="1269" height="654" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="faded-image.jpg"></image>
  <image width="1269" height="654" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="default-image.jpg"></image>
</g>
```

### 04. Highlight Images

One individual image for each mount when hovered, grouped in a `g` for clarity. When hovered, the `opacity` transitions from `0` to `1`, revealing the highlight image on top the faded image.

Positioning each highlight `image` is a little tricky, the only option is translating with `transform`. Normally this would be simple enough, but IE11 will only apply a transformation to SVG elements if it is in matrix form added as a `transform` attribute. Fine.

```html
<g>
  <image width="558" height="386" transform="matrix(1, 0, 0, 1, 270, 75)" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="highlight-0.png">
  </image>
  <image width="820" height="250" transform="matrix(1, 0, 0, 1, 0, 320)" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="highlight-1.png">
  </image>
  <image width="387" height="600" transform="matrix(1, 0, 0, 1, 562, -1)" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="highlight-2.png">
  </image>
  <image width="322" height="468" transform="matrix(1, 0, 0, 1, 846, 100)" xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="highlight-3.png">
  </image>
  </g>
```

### 05. Mouse Regions

And finally 4 `path`s, one for each mount, when hovered will set off the `opacity` transitions.

```html
<g>
  <path d="M444.5,64.5c-73.65-10-167.08,24.57-182,81-8.89,33.6,8.64,80.35,35,89,20.33,6.67,29.18-14.83,55-10,26.32,4.92,28.3,29.35,76,78,4.73,4.82,10.92,10.91,14,21,5.83,19.09-5.64,28,1,36,5,6,35,6,51,4,49.77-6.22,88,7,95,1,11.84-10.15,10-18,24-21,14.24-3.05,20.57,13.31,29,10,9.54-3.75,15-19,17-33,1-6.87,4-43,4-63,0-37-107-68.18-141-102C480.85,114,506.07,72.85,444.5,64.5Z">
  </path>
  <path d="M438.5,319.5c-51-9-105.86,81.87-191,117-96,39.62-248,17.68-248,41,0,28,704,104,747,85,90.84-40.14,74-74,74-74-4.26-15.53-35-19-35-41s28-15,28-25-45-11-51-23c1-5,24.83-9.64,45-30,11.62-11.73,17-40.12,10-41-8-1-2,19-29,38-6.12,4.31-24.55,16.92-37,19-6,1-8-5-21-4-10,.77-32.79-.34-35-2-6.88-5.16,5.44-42.14,2-45-6-5-29,17-49,21-20.69,4.14-21-14-35-12-11.88,1.7-15.8,20.7-25,23-12,3-37.81-2.54-55-3-37-1-56,3-84,1C429.82,363.09,453.56,322.16,438.5,319.5Z">
  </path>
  <path d="M715.5,46.5C648,55,615.69,35.24,583.5,63.5c-17.56,15.41-30.85,41.38-23,55,10.22,17.73,29.91,59.61,57,79,30.94,22.15,47.4,19,46,73-.94,36.32-8.21,71.62,2,75,8.23,2.73,25.08-15.39,32-12,8.44,4.14-10,36,0,44,9.65,7.72,32-2,46,5,13.65,6.82,51-17,61-33,7.49-12,6.2-21.17,12-22,7-1,10,20-10,44-12.3,14.76-43,23-41,28,1.53,3.83,7.06,8,22,11,20,4,26,7,26,13,0,9-25,6.63-27,23-2,16,26.8,26.7,33,36,12,18-6.26,29.67,2,38,6.34,6.4,19.56,3.1,32,0,23.2-5.79,24.09-14.17,40-16,21.27-2.44,30.12,11.34,41,5,10.18-5.94,14.38-25,8-36-11-19-49.54-21-62-37-7-9,10-14.33,13-42,7-65-61.56-95.25-48-133,9-25.15,39.85-14.57,47-39,10-34.34-57.45-58.54-53-104,3.4-34.77,45.61-48.43,47-98,.26-9.32.32-14.05-3-18C866.49-17.69,808.62,34.82,715.5,46.5Z">
  </path>
  <path d="M1075.5,127.5c-33.75-5-38.18-31.66-66-35-25-3-113,42-117,133-1.45,33-39.7,44.12-20,94,8.94,22.62,17.39,27.8,22,52,8,42-18,52-10,65,10.52,17.09,44,14.35,57,32,11.78,16,1.12,29.14,7,60,8,42,23.62,45.9,36,44,26-4,7.08-25.82,20-61,6.71-18.25,34.49-101.05,49-101,17.75.07,11.21,90.21,40,99,22.26,6.8,42.67-4.59,49-14,39-58,47-371-3-385C1117.25,104.27,1114.69,133.33,1075.5,127.5Z" >
  </path>
  </g>
```

## Making the Web Assets

I started in Adobe Illustrator, adding the base mounts image. This set the artboard size (similar to the canvas size in Photoshop), and this will be the size of the SVG.

I then added paths using the pen tool, tracing loosely around each mount to define the hoverable regions.

For the implementation I was pursuing (Mithril rendered) I needed to extract the path data.

Hide the image layer
File > Export > SVG
Check “use artboard"
Save to file
Open SVG in editor
Copy path data to your app

## The Implementation

The webteam at ArenaNet has been using Mithril for a couple years, and this project was no different. Mithril uses hyperscript (`m("div", attrs, children)`) for rendering HTML. Events can be attached to elements via the `attrs` object. Let's get right to the code:

Note: *SVG has a lot of boilerplate attributes which I'm leaving out from the example code*

```js
m("svg",
  m("defs",
    m("mask", { id : "mask" },
      m("image", { href : "/faded-out-mounts.jpg" }),
      m("image", { href : "/faded-out-mounts.jpg" })
    )
  ),

  m("g")
)
```



