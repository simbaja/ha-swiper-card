# Swiper Card

> ⚠️ **Notice**: This is a custom, unmaintained fork tailored specifically for internal use. It contains highly specific physics modifications—particularly for complex 3D CSS rendering like the Cube effect—but it is not actively maintained or supported for the general public.

A Lovelace card that uses [swiper](http://idangero.us/swiper/) to create a touch slider that lets you flick through multiple cards.
You can use (almost?) all options of swiper, these can be found [here](http://idangero.us/swiper/api/).

## Installation (HACS)

Since this is a custom fork, it is not available in the default HACS store. You must add it as a custom repository:
1. Go to **HACS** -> **Frontend**.
2. Click the three dots in the top right and select **Custom repositories**.
3. Paste the URL of this repository and select **Dashboard** as the category.
4. Click **Add**, then install the card.


## Configuration:

And add a card with type `custom:swiper-card`:

```yaml
- type: custom:swiper-card
  cards: []
```

## Parameters

| Name              | Type           | Default | Supported options                                                                                                                                                                                                                                                                                                                  | Description                                                                                                         |
|-------------------|----------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|
| `card_width`      | string         |         | Any css option that fits in the `width` css value                                                                                                                                                                                                                                                                                  | Will force the width of the swiper container                                                                        |
| `start_card`      | number         | 0       | Any number                                                                                                                                                                                                                                                                                                                         | The card being displayed at the beginning - indexing starts at 0, negative values index from the end starting at -1 |
| `reverse`         | boolean        | false   | true/false                                                                                                                                                                                                                                                                                                                         | Will reverse the slider direction rtl <-> ltr                                                                       |
| `background_html` | string         |         | Any HTML in string form, currently no template support                                                                                                                                                                                                                                                                             | Will render the given HTML as the first child of the slider element, for use as a slider background or similar      |
| `style`           | string         |         | Any css - it will be injected in the same node as the swiper DOM elements                                                                                                                                                                                                                                                          |                                                                                                                     |
| `script`          | string         |         | Any JS code to be prepended to the card before the swiper container                                                                                                                                                                                                                                                                |                                                                                                                     |
| `parameters`      | object         |         | Any parameter from [here](https://swiperjs.com/swiper-api#parameters)                                                                                                                                                                                                                                                              | Configuration of the swiper                                                                                         |
| `reset_after`     | number         |         | Any number                                                                                                                                                                                                                                                                                                                         | Will reset the swiper to the `start_card` if defined or the first card after `reset_after` seconds                  |
| `aspect_ratio`    | string         |         | e.g. '1/1', '16/9', '4/3'                                                                                                                                                                                                                                                                                                          | Enforces a strict CSS aspect ratio on the Swiper container, fixing vertical cutoffs inside custom Lovelace grids    |
| `cards`           | Array / string |         | Any Lovelace Card config OR type: 'swiper-image' with 'src' and optional html attribute to simply render an image, or type: swiper-html with html attribute to directly render HTML content<br/>   If given an Array, this will render immediately, if a string it will be rendered as a template and the result rendered as cards |

# Swiper Modules

The following Swiper Modules are currently supported:
 - Autoplay
 - Scrollbar
 - Navigation
 - Pagination
 - Zoom
 - EffectFade
 - EffectCards
 - EffectCube
 - EffectCoverflow
 - EffectFlip
 - Parallax

## Under the Hood: 3D Cube Hit-Testing Fixes

Building a 3D Cube slider in Swiper that seamlessly supports encapsulating custom Lovelace Web Components (like `button-card`) requires isolating the 3D physics engine and resolving HTML shadow-DOM layout collisions. This fork employs a few specific CSS techniques to achieve this:

1. **Structural Decoupling Wrapper**: 
   When Swiper applies 3D inline `transform` CSS natively onto Web Components, it can disrupt their internal canvas hit-testing. To bypass this, generated Lovelace cards are decoupled by wrapping them in a standard native `<div class="swiper-slide">` container. Swiper's 3D rotation matrix acts entirely upon the wrapper div, preserving the untouched Web Component safely positioned inside.
2. **Aspect Ratio Padding Constraint Resolution**:
   Custom elements frequently use internal `<div id="aspect-ratio">` nodes with padding to force a 1:1 square. To prevent this height logic from expanding past Swiper's dynamically calculated exact subpixel boundaries, `width: 100% !important` and `height: 100% !important` are explicitly asserted on the child Web Component. The `swiper-slide` wrapper uses standard `display: block` to lock the element perfectly within Swiper's exact container boundaries without causing Flexbox expansion.
3. **3D Collision & Intersection Masking**:
   In the `cube` effect, adjacent 3D-rotated faces overlap perfectly at the `Z=0` corners. Browser hit-testing (even for disabled hidden elements) naturally defaults to DOM order, which can cause the "next" inactive side-face to overlap and intercept mouse clicks from the front card. To resolve this intersection mathematically:
   *   The **active** inner card is explicitly translated one pixel forward through the collision plane (`transform: translateZ(1px)`).
   *   Because `translateZ` forces an element to visually expand outward under GPU perspective projection (creating a 1px artifact bleed across the 3D seam), the active element strictly asserts `overflow: hidden`. By placing `overflow: hidden` directly on the child (which occurs before Swiper computes the container's 3D projection), it cleanly clips any intersecting geometry outside its own mathematical boundaries, perfectly sealing the cube.
