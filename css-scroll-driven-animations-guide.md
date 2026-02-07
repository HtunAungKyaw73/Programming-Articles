# CSS Scroll-Driven Animations: A Comprehensive Guide to scroll() and view()

## Introduction

Scroll-driven animations represent a transformative evolution in web animation capabilities. After a decade in development, these declarative CSS features finally enable developers to create smooth, performant animations that respond to scroll position without requiring JavaScript or external libraries. Perhaps most remarkably, these animations run off the main thread, delivering GPU-accelerated performance that remains silky smooth even when the main thread is busy.

The journey to scroll-driven animations began with proposals dating back over ten years. Today, with support landing in Chrome 115 and expanding browser adoption, these capabilities are ready for production use with appropriate progressive enhancement strategies.

## Theoretical Foundations

### The Animation Timeline Concept

To understand scroll-driven animations, you must first grasp the concept of an animation timeline. Traditionally, CSS animations run on the default document timeline, where progress is measured by the passage of time. An animation starts when the page loads or when triggered, and its progress advances from 0% to 100% based on the animation-duration property.

Scroll-driven animations introduce two new types of timelines that replace time with spatial progress:

**Scroll Progress Timeline**: Progress is linked directly to the scroll position of a container along a particular axis. When you scroll from the top of a container to the bottom, the timeline progresses from 0% to 100%, regardless of how long the scrolling takes.

**View Progress Timeline**: Progress is tied to an element's visibility within its scrollable container. The timeline begins when the element starts entering the viewport and reaches completion when it exits, creating animations that respond to an element's journey through the scrollport.

These timelines integrate seamlessly with existing CSS Animations and the Web Animations API (WAAPI), meaning they inherit all the benefits these APIs provide, including hardware acceleration and the ability to run independently of the main thread.

### How Scroll Timelines Work

A scroll progress timeline converts a scroll position into a percentage value. Imagine a document with 1000 pixels of scrollable content. When the scroll position is at pixel 0, the timeline is at 0%. When scrolled to pixel 500, the timeline sits at 50%. At pixel 1000, it reaches 100%. This percentage then drives the animation's keyframe progression.

The crucial insight here is that scroll-driven animations are not time-based. If you scroll quickly, the animation progresses quickly. If you scroll slowly, the animation moves slowly. If you stop scrolling, the animation pauses. If you scroll backwards, the animation reverses. This creates a direct, synchronous relationship between user interaction and visual feedback.

### How View Timelines Work

View timelines operate on a different principle. Rather than tracking scroll position itself, they track when an element intersects with a scrollable area. Think of this as similar to the Intersection Observer API, but baked directly into CSS animations.

By default, a view timeline begins at 0% when the element's edge first touches one side of the scrollport and reaches 100% when the element's opposite edge exits the scrollport. This entire journey represents the animation's progress range. You can fine-tune when the animation starts and ends using insets, which adjust the boundaries of what's considered "in view."

The beauty of view timelines lies in their element-specific nature. Each animated element has its own timeline based on its position, allowing multiple elements to animate independently as they scroll through the viewport, each at different stages of their respective animations.

## Core Syntax and Properties

### The animation-timeline Property

The `animation-timeline` property is your gateway to scroll-driven animations. It specifies which timeline controls an animation's progress. This property accepts several types of values:

```css
/* Use the default time-based timeline */
animation-timeline: auto;

/* Remove the timeline (no animation) */
animation-timeline: none;

/* Reference a named timeline */
animation-timeline: --custom-timeline;

/* Anonymous scroll progress timeline */
animation-timeline: scroll();

/* Anonymous view progress timeline */
animation-timeline: view();
```

A critical detail when working with the animation shorthand is that `animation-timeline` is a reset-only component. This means that using the `animation` shorthand will reset any previously declared `animation-timeline` to `auto`. Therefore, you must always declare `animation-timeline` after any shorthand declarations:

```css
/* Correct order */
.element {
  animation: slide 1s linear;
  animation-timeline: scroll(); /* Declared after shorthand */
}

/* Wrong - timeline will be reset to auto */
.element {
  animation-timeline: scroll();
  animation: slide 1s linear; /* This resets timeline to auto */
}
```

### The scroll() Function

The `scroll()` function creates an anonymous scroll progress timeline. It accepts two optional parameters that define which scroller to track and which axis to monitor:

```css
animation-timeline: scroll(<scroller> <axis>);
```

The scroller parameter accepts these values:

- `nearest` (default): Uses the nearest ancestor element with scrollbars
- `root`: Uses the document's root element (the viewport)
- `self`: Uses the current element itself as the scroller

The axis parameter specifies the scroll direction:

- `block` (default): The block axis (typically vertical in horizontal writing modes)
- `inline`: The inline axis (typically horizontal in horizontal writing modes)
- `y`: The vertical axis (physical direction)
- `x`: The horizontal axis (physical direction)

Here's a practical example of a reading progress indicator:

```css
@keyframes progress {
  from {
    transform: scaleX(0);
  }
  to {
    transform: scaleX(1);
  }
}

#progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: linear-gradient(to right, #4f46e5, #06b6d4);
  transform-origin: 0% 50%;
  
  animation: progress linear;
  animation-timeline: scroll(root block);
}
```

This creates a progress bar that fills from left to right as you scroll down the page. Because we're using `scroll()` with default parameters, we could actually simplify it to just `scroll()`, which will automatically use the root scroller and block axis.

### The view() Function

The `view()` function creates an anonymous view progress timeline based on an element's visibility. Its syntax provides control over both the tracking axis and the inset boundaries:

```css
animation-timeline: view(<axis> <inset>);
```

The axis parameter works the same as in `scroll()`: block, inline, x, or y.

The inset parameter is where view timelines become particularly powerful. Insets adjust when an element is considered "in view," accepting one or two length-percentage values:

```css
/* Single value applies to both start and end */
animation-timeline: view(block 20%);

/* Two values: start inset, end inset */
animation-timeline: view(block 100px 200px);

/* Mix percentages and lengths */
animation-timeline: view(block 10% 50px);

/* Use auto to defer to scroll-padding */
animation-timeline: view(block auto 20%);
```

Let's examine a fade-in effect that triggers as elements enter the viewport:

```css
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.reveal-element {
  animation: fade-in linear;
  animation-timeline: view(block);
  
  /* Start when element is 20% into viewport,
     finish when it's 80% through */
  animation-range: entry 20% entry 80%;
}
```

The animation begins when the element is 20% visible from the bottom of the viewport and completes when it reaches 80% visibility, creating a smooth reveal effect.

## Named Timelines

While anonymous timelines using `scroll()` and `view()` are convenient, named timelines offer more flexibility, especially when you need to animate elements that aren't direct descendants of the scroller.

### Creating Named Scroll Timelines

You create named scroll timelines using the `scroll-timeline` property (or its longhand properties `scroll-timeline-name` and `scroll-timeline-axis`) on the scrolling container:

```css
.scroll-container {
  overflow-y: scroll;
  height: 500px;
  
  /* Shorthand version */
  scroll-timeline: --container-timeline block;
}

/* Or using longhands */
.scroll-container {
  overflow-y: scroll;
  height: 500px;
  
  scroll-timeline-name: --container-timeline;
  scroll-timeline-axis: block;
}
```

Notice that timeline names must begin with two dashes (--), making them custom identifiers similar to CSS custom properties. This naming convention helps avoid conflicts with CSS keywords.

Once defined, reference the named timeline in your animation:

```css
.animated-child {
  animation: rotate 1s linear;
  animation-timeline: --container-timeline;
}

@keyframes rotate {
  from {
    rotate: 0deg;
  }
  to {
    rotate: 360deg;
  }
}
```

### Creating Named View Timelines

Named view timelines work similarly, using the `view-timeline` property (or its longhands) on the subject element:

```css
.subject {
  /* Shorthand: name, axis, and insets */
  view-timeline: --reveal-timeline block 10% 20%;
}

/* Longhand version */
.subject {
  view-timeline-name: --reveal-timeline;
  view-timeline-axis: block;
  view-timeline-inset: 10% 20%;
}

/* Now animate this element or another element */
.subject {
  animation: scale-up linear;
  animation-timeline: --reveal-timeline;
}
```

The powerful feature of named view timelines is that the subject (the element being tracked) doesn't have to be the same as the target (the element being animated). You can track one element's visibility to drive animations on completely different elements.

### The timeline-scope Property

Named timelines face a limitation: they're only visible to descendant elements in the DOM tree. The `timeline-scope` property solves this by declaring timeline names at a higher level in the DOM, making them available to siblings and other branches:

```css
.parent {
  /* Declare timeline names that children can use */
  timeline-scope: --image-timeline, --text-timeline;
}

.image-section {
  scroll-timeline: --image-timeline;
}

.text-section {
  scroll-timeline: --text-timeline;
}

/* Sibling sections can now reference each other's timelines */
.image-content {
  animation-timeline: --image-timeline;
}

.text-content {
  animation-timeline: --text-timeline;
}
```

This becomes invaluable in complex layouts where you need elements in different parts of your DOM structure to coordinate their animations based on shared scroll timelines.

## Animation Ranges

Animation ranges allow you to control which portion of a timeline an animation should occupy. By default, animations span the entire timeline (0% to 100%), but ranges let you be more specific.

### Understanding Range Keywords

For view timelines, several named ranges help you target specific portions of an element's journey through the scrollport:

- `cover`: The full range from when the element starts entering until it completely exits (0% to 100%)
- `contain`: When the element is fully contained within the scrollport
- `entry`: When the element is entering the scrollport
- `exit`: When the element is exiting the scrollport
- `entry-crossing`: When the element's leading edge crosses the scrollport boundary
- `exit-crossing`: When the element's trailing edge crosses the scrollport boundary

### Using animation-range

The `animation-range` property (shorthand for `animation-range-start` and `animation-range-end`) lets you specify which part of the timeline to use:

```css
.element {
  animation: appear linear;
  animation-timeline: view();
  
  /* Animate only during the entry phase */
  animation-range: entry 0% entry 100%;
  
  /* Simplified syntax for full named range */
  animation-range: entry;
}
```

You can create sophisticated effects by combining ranges with percentages:

```css
.stagger-effect {
  animation: slide linear;
  animation-timeline: view(block);
  
  /* Start when 20% visible, end at 80% visible */
  animation-range: entry 20% entry 80%;
}

.early-reveal {
  animation: fade linear;
  animation-timeline: view();
  
  /* Finish animation before element is fully in view */
  animation-range: entry 0% entry 60%;
}

.late-reveal {
  animation: scale linear;
  animation-timeline: view();
  
  /* Don't start until element is well into view */
  animation-range: entry 40% entry 100%;
}
```

## Real-World Use Cases

### 1. Reading Progress Indicator

One of the most straightforward applications is a progress bar showing how far through an article the reader has scrolled:

```html
<div id="progress"></div>
<article>
  <!-- Your content -->
</article>
```

```css
#progress {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transform-origin: 0% 50%;
  z-index: 1000;
  
  animation: grow-progress linear;
  animation-timeline: scroll(root);
}

@keyframes grow-progress {
  from {
    transform: scaleX(0);
  }
  to {
    transform: scaleX(1);
  }
}
```

This implementation is remarkably simple compared to JavaScript alternatives that require scroll event listeners and manual calculations. The browser handles all the heavy lifting, and the animation runs smoothly off the main thread.

### 2. Parallax Effects

Parallax scrolling creates depth by moving background and foreground elements at different speeds:

```html
<section class="hero">
  <div class="background-layer"></div>
  <div class="content-layer">
    <h1>Welcome</h1>
  </div>
</section>
```

```css
.hero {
  position: relative;
  height: 100vh;
  overflow: hidden;
}

.background-layer {
  position: absolute;
  inset: 0;
  background: url('mountains.jpg') center/cover;
  
  animation: parallax-bg linear;
  animation-timeline: scroll(root);
}

.content-layer {
  position: relative;
  z-index: 1;
  
  animation: parallax-content linear;
  animation-timeline: scroll(root);
}

@keyframes parallax-bg {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-40%);
  }
}

@keyframes parallax-content {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-20%);
  }
}
```

The background moves faster than the content, creating a sense of depth as users scroll.

### 3. Shrinking Header

A common UX pattern involves a header that becomes more compact as users scroll down:

```html
<header class="site-header">
  <div class="logo">Brand</div>
  <nav><!-- Navigation items --></nav>
</header>
```

```css
.site-header {
  position: sticky;
  top: 0;
  background: white;
  padding: 2rem;
  transition: padding 0.3s ease;
  
  animation: shrink-header linear;
  animation-timeline: scroll(root);
  animation-range: 0px 500px; /* Shrink within first 500px of scroll */
}

@keyframes shrink-header {
  to {
    padding: 0.5rem;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  }
}

.logo {
  font-size: 2rem;
  
  animation: shrink-logo linear;
  animation-timeline: scroll(root);
  animation-range: 0px 500px;
}

@keyframes shrink-logo {
  to {
    font-size: 1.5rem;
  }
}
```

### 4. Image Reveal on Scroll

Create engaging reveal effects as images enter the viewport:

```html
<div class="image-container">
  <img src="photo.jpg" alt="Description" class="reveal-image">
</div>
```

```css
.reveal-image {
  opacity: 0;
  transform: scale(0.8) translateY(50px);
  
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 25% entry 75%;
}

@keyframes reveal {
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

/* Alternative: Clip-path reveal */
.clip-reveal {
  clip-path: inset(0 100% 0 0);
  
  animation: slide-reveal linear both;
  animation-timeline: view();
  animation-range: entry 20% entry 80%;
}

@keyframes slide-reveal {
  to {
    clip-path: inset(0 0 0 0);
  }
}
```

### 5. Contact List with Entry and Exit Animations

Create polished list animations where items fade in when entering and fade out when exiting:

```html
<ul class="contact-list">
  <li class="contact-item">
    <img src="avatar1.jpg" alt="Contact 1">
    <span>John Doe</span>
  </li>
  <!-- More contacts -->
</ul>
```

```css
.contact-item {
  animation: 
    fade-in linear both,
    fade-out linear both;
  animation-timeline: view();
  animation-range: 
    entry 0% entry 100%,
    exit 0% exit 100%;
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fade-out {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0;
    transform: translateY(-20px);
  }
}
```

This example demonstrates how you can apply multiple animations to the same element, each targeting different ranges of the view timeline.

### 6. Horizontal Gallery Scroller

For horizontal scrolling galleries where items animate based on horizontal scroll position:

```html
<div class="gallery-container">
  <div class="gallery">
    <div class="gallery-item">Item 1</div>
    <div class="gallery-item">Item 2</div>
    <div class="gallery-item">Item 3</div>
  </div>
</div>
```

```css
.gallery-container {
  overflow-x: scroll;
  scroll-timeline: --gallery-timeline inline;
}

.gallery {
  display: flex;
  gap: 2rem;
  width: max-content;
}

.gallery-item {
  width: 300px;
  height: 400px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  
  animation: item-scale linear;
  animation-timeline: --gallery-timeline;
}

@keyframes item-scale {
  0%, 100% {
    transform: scale(0.9);
    opacity: 0.6;
  }
  50% {
    transform: scale(1);
    opacity: 1;
  }
}
```

### 7. Sticky Section Transitions

Create smooth transitions between sticky sections:

```html
<main>
  <section class="sticky-section" style="--section-index: 1">
    <h2>Section 1</h2>
  </section>
  <section class="sticky-section" style="--section-index: 2">
    <h2>Section 2</h2>
  </section>
  <section class="sticky-section" style="--section-index: 3">
    <h2>Section 3</h2>
  </section>
</main>
```

```css
main {
  scroll-timeline: --page-scroll block;
}

.sticky-section {
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  
  animation: section-fade linear both;
  animation-timeline: --page-scroll;
  animation-range: 
    calc((var(--section-index) - 1) * 100vh) 
    calc(var(--section-index) * 100vh);
}

@keyframes section-fade {
  0% {
    opacity: 0;
    filter: blur(10px);
  }
  10%, 90% {
    opacity: 1;
    filter: blur(0);
  }
  100% {
    opacity: 0;
    filter: blur(10px);
  }
}
```

## Best Practices

### Performance Optimization

While scroll-driven animations run off the main thread, following these practices ensures optimal performance:

**Animate compositor-friendly properties**: Stick to properties that can be animated on the compositor thread: `transform`, `opacity`, `filter`, and `clip-path`. Avoid animating layout properties like `width`, `height`, `margin`, or `padding`, as these trigger expensive layout recalculations.

```css
/* Good - compositor-friendly */
@keyframes good {
  from {
    transform: translateY(50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

/* Avoid - triggers layout */
@keyframes avoid {
  from {
    margin-top: 50px;
    height: 100px;
  }
  to {
    margin-top: 0;
    height: 200px;
  }
}
```

**Use will-change judiciously**: While `will-change` can hint to browsers about upcoming animations, overuse creates memory overhead. Apply it only to elements actively being animated:

```css
.animated-element {
  /* Only when actually animating */
  will-change: transform, opacity;
}

/* Better: Let the browser manage it */
.animated-element {
  animation: slide linear;
  animation-timeline: view();
  /* Browser automatically optimizes */
}
```

**Minimize the number of simultaneously animated elements**: Although performant, animating hundreds of elements simultaneously can still impact performance. Consider using intersection-based animations that only activate when elements are visible.

### Accessibility Considerations

Respect user preferences for reduced motion. Always provide alternatives for users who have enabled this setting:

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-timeline: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Better approach: Disable scroll-driven animations specifically */
@media (prefers-reduced-motion: reduce) {
  .scroll-animated {
    animation-timeline: auto;
    animation: none;
    /* Provide the final state immediately */
    opacity: 1;
    transform: none;
  }
}
```

Ensure your content remains accessible without animations. The content should be readable and usable even if animations fail to load or are disabled:

```css
/* Ensure base styles work without animation */
.reveal-element {
  /* Default visible state for no-JS or animation failure */
  opacity: 1;
  transform: none;
}

/* Animation as progressive enhancement */
@supports (animation-timeline: view()) {
  .reveal-element {
    opacity: 0;
    transform: translateY(30px);
    animation: reveal linear;
    animation-timeline: view();
  }
}
```

### Progressive Enhancement

Always implement scroll-driven animations as progressive enhancement. Test for support and provide fallbacks:

```css
@supports (animation-timeline: scroll()) {
  .progress-bar {
    animation: progress linear;
    animation-timeline: scroll(root);
  }
}

@supports not (animation-timeline: scroll()) {
  /* Fallback: simple static element or JavaScript solution */
  .progress-bar {
    display: none;
  }
}
```

For JavaScript detection:

```javascript
if (CSS.supports('animation-timeline', 'scroll()')) {
  // Browser supports scroll-driven animations
  document.body.classList.add('supports-scroll-timeline');
} else {
  // Fall back to JavaScript implementation or simpler solution
  console.log('Scroll-driven animations not supported');
}
```

### Debugging and Development

Use the Scroll-Driven Animations Debugger extension for Chrome DevTools, which visualizes:

- The scroller, animated element, and subject
- Timeline progress in real-time
- Animation ranges and their boundaries
- Both CSS-based and WAAPI-based animations

When developing, add visual indicators to help understand your timelines:

```css
/* Debug helper: Show when element is in view */
.debug .animated-element {
  outline: 2px dashed red;
  animation: debug-pulse linear;
  animation-timeline: view();
}

@keyframes debug-pulse {
  0%, 100% {
    outline-color: red;
  }
  50% {
    outline-color: blue;
  }
}
```

### Common Pitfalls to Avoid

**Forgetting scroll containers need overflow**: A scroll timeline can't exist without actual scrolling. Ensure your container has overflow and enough content:

```css
/* Won't work - no overflow */
.container {
  height: 500px;
  scroll-timeline: --timeline;
}

/* Works - has overflow */
.container {
  height: 500px;
  overflow-y: scroll;
  scroll-timeline: --timeline;
}

.container-content {
  height: 1500px; /* Exceeds container height */
}
```

**Using animation shorthand incorrectly**: Remember that the `animation` shorthand resets `animation-timeline`. Always declare the timeline last:

```css
/* Wrong */
.element {
  animation-timeline: scroll();
  animation: fade 1s linear; /* Resets timeline to auto */
}

/* Correct */
.element {
  animation: fade 1s linear;
  animation-timeline: scroll(); /* Declared after shorthand */
}
```

**Not considering initial visibility**: Elements using view timelines start at their initial animation state. If your animation begins with `opacity: 0`, the element will be invisible until it enters the viewport:

```css
/* Element invisible until scrolled into view */
.element {
  animation: fade-in linear;
  animation-timeline: view();
}

@keyframes fade-in {
  from {
    opacity: 0; /* Invisible by default */
  }
  to {
    opacity: 1;
  }
}

/* Solution: Use animation-fill-mode or set default state */
.element {
  opacity: 1; /* Default visible */
  animation: fade-in linear backwards;
  animation-timeline: view();
}
```

### Browser Support and Polyfills

As of early 2025, scroll-driven animations are supported in:

- Chrome/Edge 115+
- Firefox (behind flag, native support coming)
- Safari (in development)

For broader compatibility, use the official polyfill:

```html
<script src="https://flackr.github.io/scroll-timeline/dist/scroll-timeline.js"></script>
```

The polyfill provides JavaScript-based fallback support for browsers without native implementation, though it won't achieve the same off-main-thread performance as native support.

## Advanced Techniques

### Combining Multiple Timelines

You can apply multiple animations with different timelines to the same element:

```css
.complex-element {
  animation: 
    rotate linear,
    scale linear,
    fade linear;
  animation-timeline: 
    scroll(root block),
    scroll(nearest inline),
    view();
}

@keyframes rotate {
  to { rotate: 360deg; }
}

@keyframes scale {
  to { transform: scale(1.5); }
}

@keyframes fade {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

Each animation progresses according to its own timeline, creating complex, layered effects.

### Dynamic Timeline Ranges with calc()

Use CSS calc() to create dynamic animation ranges based on viewport units or custom properties:

```css
.dynamic-animation {
  animation: slide linear both;
  animation-timeline: scroll(root);
  animation-range: 
    calc(var(--start-offset) * 1vh) 
    calc(var(--end-offset) * 1vh);
}

/* Adjust via custom properties */
.early-start {
  --start-offset: 10;
  --end-offset: 40;
}

.late-start {
  --start-offset: 60;
  --end-offset: 90;
}
```

### View Timeline Insets for Precision Control

Insets give you surgical precision over when view animations begin and end:

```css
.precise-reveal {
  animation: appear linear both;
  animation-timeline: view(block);
  view-timeline-inset: 100px 200px;
}

/* Negative insets extend the range beyond the scrollport */
.early-trigger {
  animation: pre-load linear both;
  animation-timeline: view();
  view-timeline-inset: -50% 0%;
  /* Starts animating when element is still 50% of its height 
     below the viewport */
}
```

## Integration with JavaScript

While scroll-driven animations work purely in CSS, you can enhance them with JavaScript when needed:

```javascript
// Create a ScrollTimeline programmatically
const progressBar = document.querySelector('#progress');

progressBar.animate(
  {
    transform: ['scaleX(0)', 'scaleX(1)']
  },
  {
    fill: 'forwards',
    timeline: new ScrollTimeline({
      source: document.documentElement,
      axis: 'block'
    })
  }
);

// Create a ViewTimeline
const revealElement = document.querySelector('.reveal');

revealElement.animate(
  {
    opacity: [0, 1],
    transform: ['translateY(50px)', 'translateY(0)']
  },
  {
    fill: 'both',
    timeline: new ViewTimeline({
      subject: revealElement,
      axis: 'block',
      inset: [CSS.percent(20), CSS.percent(80)]
    })
  }
);
```

This JavaScript approach offers more dynamic control, allowing you to create timelines based on runtime conditions or user interactions.

## Conclusion

CSS scroll-driven animations represent a paradigm shift in how we create interactive web experiences. By moving scroll-linked animations from JavaScript event handlers to declarative CSS (or WAAPI), we achieve:

**Better performance**: Animations run off the main thread, eliminating jank and staying smooth even under heavy load.

**Simpler code**: What once required dozens of lines of JavaScript and complex calculations now takes just a few lines of CSS.

**Perfect synchronization**: Animations are directly tied to scroll position, creating an instant, responsive feel that's impossible with asynchronous event-based approaches.

**Broader accessibility**: Built-in support for respecting user preferences like `prefers-reduced-motion`.

As browser support continues to expand and developers embrace these new capabilities, we can expect scroll-driven animations to become a fundamental tool in the modern web developer's toolkit. The combination of power, simplicity, and performance makes them ideal for creating engaging, polished web experiences that delight users while maintaining excellent performance characteristics.

Whether you're building a simple reading progress indicator or orchestrating complex parallax effects across an entire page, scroll-driven animations provide the tools you need with elegance and efficiency. Start with progressive enhancement, respect user preferences, and focus on compositor-friendly properties, and you'll create scroll experiences that feel smooth, responsive, and thoroughly modern.

## Additional Resources

- [MDN Web Docs: CSS Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)
- [Chrome for Developers: Scroll-driven Animations](https://developer.chrome.com/docs/css-ui/scroll-driven-animations)
- [Scroll-Driven Animations Demo Site](https://scroll-driven-animations.style/)
- [Smashing Magazine: Introduction to CSS Scroll-Driven Animations](https://www.smashingmagazine.com/2024/12/introduction-css-scroll-driven-animations/)
- [Official Scroll-Driven Animations Video Course](https://developer.chrome.com/blog/scroll-driven-animations-video-course)
- [Scroll-Driven Animations Debugger Extension](https://chromewebstore.google.com/detail/scroll-driven-animations/ojihehfngalmpghicjgbfdmloiifhoce)
