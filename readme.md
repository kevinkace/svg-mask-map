# SVG Transparency Mask and Image Map

This is my first post of this nature. It describes combining a few different capabilities of SVG, namely transparency masks (smaller than PNGs), CSS transitions, positioning elements within an SVG using CSS `transform`, and SVG `path`s for an image map.

I hope you enjoy it, and if you see any improvements (technical or otherwise) please comment.

## Feature Description

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

Here are the visual components

todo: video of visual components

## Coming to a solution

The one part that had a clear solution get-go was fading the background. Transitioning opacity on a `div` with a background color filling the entire section would be simple and effective.

For the other problems, well... I've been using SVG more and more recently, and perhaps in a case of "everything is a nail when you have a hammer", I looked to SVG to solve the other problems, file size and hover regions.

### SVG transparancy mask

This is a simple technique that we've been using at ArenaNet after seeing this article [Using SVG to Shrink Your PNGs](http://peterhrynkow.com/how-to-compress-a-png-like-a-jpeg/) in Sept 2014. The idea is to split a single transparent image (PNG) into 2 opaque images, extracting the transparancy data to a separate image - the transparancy mask. The mask works the same as a layer mask in Photoshop - a black and white image where black is transparent. These 2 images are opque meaning they can be much smaller JPGs.

Creating these 2 images todo

### SVG paths

An SVG `path` can define an irregular shape with a variety of line commands, including `curveto` which uses Bezier curves - this is a great fit for the hoverable regions.

I created these paths in Adobe Illustrator:

# open one of the base PNGs in Illustrator
# use the pen-tool to draw shapes around each mount
# File > Export > Export as...
# select SVG
# check Use Artbox
# click ok

I opened the SVG in a text editor, and copied the path strings to my component.

## Putting it all together

Here are all the assets for this feature:

- full bleed background image
- background fade element
- image of all mounts faded out
- image of all mounts default state
- image of transparency mask
- 4 images of mounts highlighted
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

### 00. Background & overlay

Starting with the outer most elements; the full bleed background image and the background fade element. When a mount is hovered, the `bgFade-in` class will be added to `div.bgFade` obscuring the background image behind.

todo: gif of just BG interaction

```html
<section class="mountsSection">
    <div class="bgFade"></div>
    <!-- ...more code -->
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

We start here with a `def` element, for "definition", it's children aren't to be shown directly but will be referenced later in (or outside) the SVG by ID. Within the `def` there's a `mask`, and an `image`. This transparancy mask will be used by it's `id`.

```html
<defs>
  <mask id="mask">
    <image href="/grey-scale-alpha.jpg"></image>
  </mask>
</defs>
```

### 03. Images

Here we have 2 `image`s nested in a group `g`, and the transparency mask is applied to the group. The 2 images (and anything within the `g`) are masked, removing the background. content, but with no transparency. The images will be stacked on top of each other, one for the default state, one for the faded out state. Animating the opacity of the default image will reveal the faded out image behind. and because they are grouped in a `g`, the mask is applied to the combination of the 2 images.

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



