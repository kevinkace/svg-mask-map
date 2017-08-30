# SVG Transparency Mask and Image Map

This is my first post of this nature. It describes combining a few different capabilities of SVG, namely transparency masks (smaller than PNGs), CSS transitions, positioning elements within an SVG using CSS `transform`, and SVG `path`s for an image map.

I hope you enjoy it, and if you see any improvements (technical or otherwise) please comment.

## Feature Description

*Guild Wars 2* has a rich, blended, organic art style which can be difficult to incorporate into a website. There were a few tricky sections when developing the *Path of Fire* website. One specifically was the section for newly launched **mounts**!* It has a graphic of 4 mounts centered on a background, hovering on a mount shifts the "lighting", highlighting the hovered mount, and fading out the other mounts and background. Clicking on a mount opens detail of the mount but not descibed here.

Here's what the [final product](https://www.guildwars2.com/en/path-of-fire/#mounts) looks like:

<video autoplay loop style="max-width: 100%">
<source src="https://fat.gfycat.com/MealyNippyCreature.webm" type="video/mp4">
</video>

It's functionally a juiced up version of the [professions page](https://www.guildwars2.com/en/the-game/professions/).

* The specializations section also used this feature, but no one wants to write or read "specializations" a hundred times in this article. TL;DR: it got double usage.

## Unique problems

Other than the mounts graphic, the section has intro text above (localized into 4 languages), and navigation below, all on top of a fullbleed background image. To keep text length flexible, the mounts image had to be on a transparent background. Using transparent PNGs for the mounts images would be the conventional approach, and would look/work great, with one major detraction: file size. The mounts image was over 1200x600px, and there were 3 of them (base, highlighted, faded out).

Defining the hoverable region around a mount image is also a little tricky, as the mounts are irregularly shaped so a rectangular `div` wouldn't cut it. The professions section on GW2.com uses an [image `map`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/map) with mouse events added to `area`s, one for each profession. This actually works great, but I didn't reuse it here.

## Coming to a solution

The one part that had a clear solution get-go was fading the background. Transitioning opacity on a `div` with a background color filling the entire section would be simple and effective.

For the other problems, well... I've been using SVG more and more recently, and perhaps in a case of "everything is a nail when you have a hammer", I looked to SVG to solve the other problems, file size and hover regions.

### SVG transparancy mask

This is a simple technique that we've been using at ArenaNet after seeing this article [Using SVG to Shrink Your PNGs](http://peterhrynkow.com/how-to-compress-a-png-like-a-jpeg/) in Sept 2014. The idea is to split a single transparent image (PNG) into 2 opaque images, extracting the transparancy data to a separate image - the transparancy mask. The mask works the same as a layer mask in Photoshop - a black and white image where black is transparent. These 2 images are opque meaning they can be much smaller JPGs.

Creating these 2 images todo

### SVG paths

These are irregular shapes which are well suiting for defining the hoverable regions for each mount. I created these paths in Adobe Illustrator.

I'm going to fast forward through the trial and error. I first looked to SVG primarily for defining the hoverable regions with polyshapes, but SVG can also solve the size issue of large PNGs: using a transparency mask. This combines a grayscale image defining the transparency (or alpha), and a second image with the actual image data. Neither image has transparency so formats with better compression can be used (usually JPG).

Modifying opacity is a common go-to solution for highlighting on hover, but it wasn't an option here as additional art was created for the highlighted and faded states, effectively 3x 1000x500 images. The visual elements stacked and aligned from back to front:

- 1 full bleed background image
- 1 element of a solid color fill to overlay background (`rgba(255, 255, 255, 0.3)`)
- 1 image of all mounts faded out
- 1 image of all mounts default state
- 1 image of **each** mount highlighted

<video autoplay loop style="max-width: 100%">
<source src="https://giant.gfycat.com/DependentSnoopyIrishwolfhound.webm" type="video/mp4">
</video>

Initially the solid overlay, and all highlighted images have their `opacity` set to `0`, when a mount is hovered the overlay and one highlighted mount transition to `opacity: 1`, while the default mounts image fades out.

One last component is to define regions for the mouseover event to trigger the highlight/fade effect. This region would need to be irregularly shaped, (a div/span isn't well suited).

Let's put aside the complexity of the hover transitions, just one of the mounts images posed an issue: it's large, around 1000x500, and requires transparency. PNGs are the usual go-to for something like this, but even with optimization a PNG was over 2mb.

## The Plan

I'm going to fast forward through the trial and error. I first looked to SVG primarily for defining the hoverable regions with polyshapes, but SVG can also solve the size issue of large PNGs: using a transparency mask. This combines a grayscale image defining the transparency (or alpha), and a second image with the actual image data. Neither image has transparency so formats with better compression can be used (usually JPG).

<video autoplay loop style="max-width: 100%">
<source src="https://giant.gfycat.com/RadiantAshamedAfghanhound.webm" type="video/mp4">
</video>

Here's the DOM and SVG outline:

```html
<!-- 00. background -->
<section>

 <!-- 01. overlay -->
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
    <path d="path data for mount 0"></path>
    <path d="path data for mount 1"></path>
    <path d="path data for mount 2"></path>
    <path d="path data for mount 3"></path>
  </g>
</svg>
```

### 00. Background

Just a standard JPG, compressed at medium quality in Photoshop, 1920x1080. This is set as the sections background with `background-size` set to `cover`.

### 01. Overlay

A single `div`, positioned `absolute` to fill the section. Setting a solid background color on the `div` and transitioning the `opacity` from `0` to `0.3` was enough to achieve the background dimming.

### 02. Mask

Note: *You can skip this paragraph if you're familiar with SVG masks (or layer masks in Photoshop). The process describing how to make the actual mask is later.*

The basic idea is to use a separate grey-scale image which defines the transparency; black being fully transparent, white being fully opaque and greys in between for translucency. This image mask is then applied over another image to create transparency. The `def` element, for "definition", means it's children aren't to be shown directly, the `image` within the `mask` will be referenced later in the SVG by ID.

### 03. Images

Here we have the actual image content, but with no transparency. The images will be stacked on top of each other, one for the default state, one for the faded out state. Animating the opacity of the default image will reveal the faded out image behind. and because they are grouped in a `g`, the mask is applied to the combination of the 2 images.

### 04. Highlight Images

Four individual images, one for each mount when hovered, (grouped in a `g` for clarity). I left these as individual PNGs, but they could be separate SVGs using their own mask/image. Animating the opacity will reveal them on top the faded out image behind. Positioning each highlight `image` is a little tricky, the only option is translating with `transform`. IE11 will only apply a transformation to SVG elements if it is in matrix form added as a `transform` attribute. Fine.

### 05. Mouse Regions

And finally 4 `path`s, one for each mount, when hovered will set off the opacity animations for the hover effect. SVG paths can be complex shapes which makes them well suited here.

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



