@page guides/recipes/tinder-carousel Tinder Carousel
@parent guides/recipes/intermediate

@description This intermediate guide walks you through building a [Tinder](https://tinder.com/)-like
carousel. Learn how to build apps that use dragging user interactions.

@body


In this guide, you will learn how to create a custom [Tinder](https://tinder.com/)-like
carousel. The custom widget will have:

- Touch and drag functionality that works on mobile and desktop.
- Custom `<button>`’s for liking and disliking


The final widget looks like:

<p class="codepen" data-height="536" data-theme-id="0" data-default-tab="js,result" data-user="bitovi" data-slug-hash="QWLmPXo" style="height: 536px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="CanJS 6 Tinder-Like Carousel">
  <span>See the Pen <a href="https://codepen.io/bitovi/pen/QWLmPXo/">
  CanJS 6 Tinder-Like Carousel</a> by Bitovi (<a href="https://codepen.io/bitovi">@bitovi</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

The following sections are broken down the following parts:

- __The problem__ - A description of what the section is trying to accomplish.
- __What you need to know__ - Information about CanJS that is useful for solving the problem.
- __The solution__ - The solution to the problem.

## Setup ##

__START THIS TUTORIAL BY CLICKING THE “EDIT ON CODEPEN” BUTTON IN THE TOP RIGHT CORNER OF THE FOLLOWING EMBED__:

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="html,result" data-user="bitovi" data-slug-hash="MWgGJgJ" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="CanJS 6 Tinder-Like Carousel">
  <span>See the Pen <a href="https://codepen.io/bitovi/pen/MWgGJgJ/">
  CanJS 6 Tinder-Like Carousel</a> by Bitovi (<a href="https://codepen.io/bitovi">@bitovi</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

This CodePen loads CanJS (`import { StacheElement } from "//unpkg.com/can@6/core.mjs"`).

### The problem

When someone adds `<evil-tinder></evil-tinder>` to their HTML, we want the following HTML
to show up:

```html
<div class="header"></div>

<div class="images">
  <div class="current">
    <img src="https://user-images.githubusercontent.com/78602/40454720-7c3d984c-5eaf-11e8-9fa7-f68ddd33e3f0.jpg"/>
  </div>
  <div class="next">
    <img src="https://user-images.githubusercontent.com/78602/40454716-76bef438-5eaf-11e8-9d29-5002260e96e1.jpg"/>
  </div>
</div>

<div class="footer">
  <button class="dissBtn">Dislike</button>
  <button class="likeBtn">Like</button>
</div>
```

### What you need to know

To set up a basic CanJS application, you define a custom element in JavaScript and
use the custom element in your page’s `HTML`.

To define a custom element, extend [can-stache-element] and [register it](https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry/define)
with the tag name you want to use in the HTML.

For example, we will use `<evil-tinder>` as our custom tag:

```js
class EvilTinder extends StacheElement {}
customElements.define("evil-tinder", EvilTinder);
```

But this doesn’t do anything.  Components add their own HTML through their [can-stache-element/static.view]
property like this:

```js
class EvilTinder extends StacheElement {
  static view = `
    <h2>Evil-Tinder</h2>
  `,
  static props = {};
}
customElements.define("evil-tinder", EvilTinder);
```

> **NOTE:** We’ll make use of the `props` object later.


### The solution

Update the __JavaScript__ tab to:

@sourceref ./0-setup.js
@highlight 3-25,only

Update the `<body>` element in the __HTML__ tab to:

```html
<evil-tinder></evil-tinder>
```

## Show the current and next profile images

### The problem

Instead of hard-coding the current and next image URLs, we want to show the first two items
in the following list of profiles:

```js
[
  { name: "gru", img: "https://user-images.githubusercontent.com/78602/40454685-5cab196e-5eaf-11e8-87ac-4af6792994ed.jpg" },
  { name: "hannibal", img: "https://user-images.githubusercontent.com/78602/40454705-6bf4d3d8-5eaf-11e8-9562-2bd178485527.jpg" },
  { name: "joker", img: "https://user-images.githubusercontent.com/78602/40454830-e71178dc-5eaf-11e8-80ee-efd64911e35f.png" },
  { name: "darth", img: "https://user-images.githubusercontent.com/78602/40454681-59cffdb8-5eaf-11e8-94ac-4849ab08d90c.jpg" },
  { name: "norman", img: "https://user-images.githubusercontent.com/78602/40454709-6fecc536-5eaf-11e8-9eb5-3da39730adc4.jpg" },
  { name: "stapuft", img: "https://user-images.githubusercontent.com/78602/40454711-72b19d78-5eaf-11e8-9732-80155ff8bb52.jpg" },
  { name: "dalek", img: "https://user-images.githubusercontent.com/78602/40454672-566b4984-5eaf-11e8-808d-cb5afd445e89.jpg" },
  { name: "wickedwitch", img: "https://user-images.githubusercontent.com/78602/40454720-7c3d984c-5eaf-11e8-9fa7-f68ddd33e3f0.jpg" },
  { name: "zod", img: "https://user-images.githubusercontent.com/78602/40454722-802ef694-5eaf-11e8-8964-ca648368720d.jpg" },
  { name: "venom", img: "https://user-images.githubusercontent.com/78602/40454716-76bef438-5eaf-11e8-9d29-5002260e96e1.jpg" }
]
```

If we were to remove items on the `evil-tinder` component as follows, the images will update:

```js
document.querySelector("evil-tinder").profiles.shift()
```

### What you need to know

- A component's `view` is rendered with its [can-stache-element/static.props]. For example, we can create
  a list of profiles and write out an `<img>` for each one like:

  ```js
  class EvilTinder extends StacheElement {
    static view = `
      {{# for(profile of this.profiles) }}
        <img src="{{ profile.img }}">
      {{/ for }}
    `;

    static props = {
      get default() {
        return new ObservableArray([
          { img: "https://user-images.githubusercontent.com/78602/40454672-566b4984-5eaf-11e8-808d-cb5afd445e89.jpg" },
          { img: "https://user-images.githubusercontent.com/78602/40454720-7c3d984c-5eaf-11e8-9fa7-f68ddd33e3f0.jpg" }
        ]);
      }
    };
  }
  customElements.define("evil-tinder", EvilTinder);
  ```

  The [can-observable-object/define/get-default] behavior specifies the default value of the `profiles`
  property; [can-observable-array] is used to make sure the `view` is updated when `profiles` changes.

  The `view` uses [can-stache.tags.escaped] to write out values from the component `props` into the DOM.

- Use a [can-observable-object/define/get getter] to derive a value from another 
  value on the component `props`, this will allow us to get the next profile image:

  ```js
  get currentProfile() {
    return this.profiles[0];
  },
  ```

### How to verify it works

Run the following in the **Console** tab.  The background image should move to the foreground.

```js
document.querySelector("evil-tinder").profiles.shift()
```

### The solution

Update the __JavaScript__ tab to:

@sourceref ./1-profiles.js
@highlight 9,12,23-37,only


## Add a like button

### The problem

- When someone clicks the like button, [console.log](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) `LIKED`, remove the first profile image, and show the next one in the list.


### What you need to know

- Use [can-stache-bindings.event] to call a function on the component when a DOM event happens:

  ```html
  <button on:click="doSomething()"></button>
  ```

- Those functions (example: `doSomething`) are added as methods on the component like:

  ```js
  class SomeElement extends StacheElement {
    static view = `<button on:click="doSomething('dance')"></button>`;
    static props = { ... };
    doSomething(cmd) {
      alert("doing " + cmd);
    }
  }
  customElements.define("some-element", SomeElement);
  ```

- Use [.shift](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/shift) to remove an item from the start of an array:
  ```js
  this.profiles.shift();
  ```





### The solution

Update the __JavaScript__ tab to:

@sourceref ./2-like-btn.js
@highlight 18-19,50-53,only




## Add a nope button

### The problem

- When someone clicks the nope button, [console.log](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) `NOPED` and remove the first profile.

### What you need to know

- You know everything you need to know

### The solution

Update the __JavaScript__ tab to:

@sourceref ./3-dislike-btn.js
@highlight 17-18,56-59,only



## Drag and move the profile to the left and right


### The problem

In this section we will:

- Move the current profile to the left or right as user drags the image to the
  left or right.
- Implement drag functionality so it works on a mobile or desktop device.
- Move the `<div class="current">` element


### What you need to know

We need to listen to when a user drags and update the `<div class="current">` element’s
horizontal position to match how far the user has dragged.

- To update an element’s horizontal position with [can-stache] you can set the `element.style.left`
  property like:
  ```HTML
  <div class="current" style="left: {{ howFarWeHaveMoved }}px">
  ```

The remaining problem is how to get a `howFarWeHaveMoved` property to update
as the user creates a drag motion.

- Define a number property on the component `props` with:

  ```js
  static props = {
    // ...
    howFarWeHaveMoved: Number
  };
  ```

- A drag motion needs to be captured just not on the element itself, but on the 
entire `document`, we will setup the event binding in the [can-stache-element/lifecycle-hooks.connected] hook of the component as follows:

  ```js
    class SomeElement extends StacheElement {
      ...
      connected() {
        let current = el.querySelector(".current");
      }
    }
  ```

- Desktop browsers dispatch mouse events. Mobile browsers dispatch touch events.
  _Most_ desktop and dispatch [Pointer events](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events).

  You can listen to pointer events with [can-event-queue/map/map.listenTo] inside `connected` like:

  ```js
  this.listenTo(current, "pointerdown", (event) => { /* ... */ })
  ```

  Drag motions on images in desktop browsers will attempt to drag the image unless
  this behavior is turned off.  It can be turned off with `draggable="false"` like:

  ```html
  <img draggable="false">
  ```

- Pointer events dispatch with an `event` object that contains the position of
  the mouse or finger:

  ```js
  this.listenTo(current, "pointerdown", (event) => {
    event.clientX //-> 200
  });
  ```

  On a pointerdown, this will be where the drag motion starts. Listen to
  `pointermove` to be notified as the user moves their mouse or finger.

- Listen to `pointermove` on the `document` instead of the dragged item to
  better tollerate drag motions that extend outside the dragged item.

  ```js
  this.listenTo(document, "pointermove", (event) => {

  });
  ```
  The difference between `pointermove`’s position and `pointerdown`’s
  position is how far the current profile `<div>` should be moved.

### The solution

Update the __JavaScript__ tab to:

@sourceref ./4-move-current-profile.js
@highlight 8-13,45,66-77,only


## Show liking animation when you drag to the right

### The problem

In this section, we will:

- Show a like "stamp" when the user has dragged the current profile to the right 100 pixels.
- The like stamp will appear when an element like `<div class="result">` has `liking`
  added to its class list.

### What you need to know

- Use [can-stache.helpers.if] to test if a value is truthy and add a value to an element’s class list like:
  ```html
  <div class="result {{# if(liking) }}liking{{/ if}}">
  ```
- Use a [can-observable-object/define/get getter] to derive a value from another value:
  ```js
  get liking() {
    return this.howFarWeHaveMoved >= 100;
  },
  ```

### The solution

Update the __JavaScript__ tab to:

@sourceref ./5-show-liking.js
@highlight 6,55-57,only




## Show noping animation when you drag to the left

### The problem

- Show a nope "stamp" when the user has dragged the current profile to the left 100 pixels.
- The nope stamp will appear when an element like `<div class="result">` has `noping`
  added to its class list.

### What you need to know

You know everything you need to know!

### The solution

Update the __JavaScript__ tab to:

@sourceref ./6-show-nope.js
@highlight 6-7,60-62,only




## On release, like or nope

### The problem

In this section, we will perform one of the following when the user completes their
drag motion:

- [console.log](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) `like` and move to the next profile if the drag motion has moved at least 100 pixels to the right
- [console.log](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) `nope` and move to the next profile if the drag motion has moved at least 100 pixels to the left
- do nothing if the drag motion did not move 100 pixels horizontally

And, we will perform the following no matter what state the drag motion ends:

- Reset the state of the application so it can accept further drag motions and the
  new profile image is centered horizontally.

### What you need to know

- Listen to `pointerup` to know when the user completes their drag motion:
  ```js
  this.listenTo(document, "pointerup", (event) => { });
  ```

- To stopListening to the `pointermove` and `pointerup` events on the document with:

  ```js
  this.stopListening(document);
  ```


### The solution

Update the __JavaScript__ tab to:

@sourceref ./7-release.js
@highlight 86-97,only

## Add an empty profile

### The problem

In this section, we will:

-  Show the following stop sign URL when the user runs out of profiles:
  `https://stickwix.com/wp-content/uploads/2016/12/Stop-Sign-NH.jpg`.


### What you need to know

- Use [can-observable-object/define/get-default] to create a default property value:

  ```js
  emptyProfile: {
    get default() {
      return { img: "https://stickwix.com/wp-content/uploads/2016/12/Stop-Sign-NH.jpg" };
    }
  },
  ```

### The solution

Update the __JavaScript__ tab to:

@sourceref ./8-empty-profile.js
@highlight 48-54,57,61,only

## Result

When finished, you should see something like the following CodePen:

<p class="codepen" data-height="536" data-theme-id="0" data-default-tab="js,result" data-user="bitovi" data-slug-hash="QWLmPXo" style="height: 536px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid black; margin: 1em 0; padding: 1em;" data-pen-title="CanJS 6 Tinder-Like Carousel">
  <span>See the Pen <a href="https://codepen.io/bitovi/pen/QWLmPXo/">
  CanJS 6 Tinder-Like Carousel</a> by Bitovi (<a href="https://codepen.io/bitovi">@bitovi</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

<script async src="https://static.codepen.io/assets/embed/ei.js"></script>
