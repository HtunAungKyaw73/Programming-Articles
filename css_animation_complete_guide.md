# CSS Animations & Transitions: Complete Guide
*Based on MDN Web Docs and CSS-Tricks*

---

# Part 1: CSS Transitions

## 🎨 What are CSS Transitions?

CSS transitions provide a way to control animation speed when changing CSS properties. Instead of having property changes take effect immediately, you can cause the changes in a property to take place over a period of time

Animations that involve transitioning between two states are often called implicit transitions as the states in between the start and final states are implicitly defined by the browser

### Why Use CSS Transitions?

- Simple state changes (hover, focus, active)
- No keyframes needed
- Automatic reverse animation
- Lightweight and performant
- Perfect for interactive UI elements

---

## 📝 Basic Syntax

```css
/* Apply transition to element */
.button {
  background-color: blue;
  transition: background-color 0.3s ease;
}

/* Change property on state change */
.button:hover {
  background-color: red;
}
```

**How it works:**
1. Element has default state (`blue` background)
2. User triggers state change (hover)
3. Browser smoothly animates from blue → red over 0.3s
4. User removes trigger (mouse leave)
5. Browser automatically reverses animation red → blue

---

## ⚙️ Transition Properties

CSS Transitions are controlled using the shorthand transition property. This is the best way to configure transitions, as it makes it easier to avoid out of sync parameters

### 1. `transition-property`

The transition-property CSS property sets the CSS properties to which a transition effect should be applied

```css
.element {
  /* Specific property */
  transition-property: background-color;
  
  /* Multiple properties */
  transition-property: width, height, background-color;
  
  /* All animatable properties */
  transition-property: all;
  
  /* No transition */
  transition-property: none;
}
```

### 2. `transition-duration`

Specifies the duration over which transitions should occur. You can specify a single duration that applies to all properties during the transition, or multiple values to allow each property to transition over a different period of time

```css
.element {
  transition-duration: 0.5s;      /* 500 milliseconds */
  transition-duration: 300ms;     /* 300 milliseconds */
  
  /* Different durations for different properties */
  transition-property: width, opacity;
  transition-duration: 2s, 1s;    /* width: 2s, opacity: 1s */
}
```

### 3. `transition-timing-function`

Specifies a function to define how intermediate values for properties are computed. Easing functions determine how intermediate values of the transition are calculated

```css
.element {
  /* Preset functions */
  transition-timing-function: linear;       /* Constant speed */
  transition-timing-function: ease;         /* Slow-fast-slow (default) */
  transition-timing-function: ease-in;      /* Slow start */
  transition-timing-function: ease-out;     /* Slow end */
  transition-timing-function: ease-in-out;  /* Slow start & end */
  
  /* Custom bezier */
  transition-timing-function: cubic-bezier(0.42, 0, 0.58, 1);
  
  /* Steps */
  transition-timing-function: steps(4, end);
}
```

**Timing Function Guide:**
- `ease` (default) - Best for most UI interactions
- `ease-out` - Best for elements entering the screen
- `ease-in` - Best for elements exiting the screen
- `ease-in-out` - Best for elements moving within screen
- `linear` - Best for color changes, loading spinners

### 4. `transition-delay`

Defines how long to wait between the time a property is changed and the transition actually begins

```css
.element {
  transition-delay: 0.5s;     /* Wait 500ms before starting */
  transition-delay: -0.2s;    /* Start 200ms into the transition */
}
```

### 5. `transition-behavior` (New!)

The transition-behavior CSS property specifies whether transitions will be started for properties whose animation behavior is discrete

```css
.element {
  transition-property: opacity, display;
  transition-duration: 0.3s;
  transition-behavior: allow-discrete;  /* Allow discrete properties */
}

.element.hidden {
  opacity: 0;
  display: none;  /* Can now be transitioned! */
}
```

---

## 🚀 The Shorthand Property

One or more single-property transitions, separated by commas. Each single-property transition describes the transition that should be applied to a single property or all properties

```css
.element {
  transition: property duration timing-function delay;
}
```

**Examples:**

```css
/* Simple */
.element {
  transition: background-color 0.3s;
}

/* Full specification */
.element {
  transition: opacity 0.5s ease-in-out 0.1s;
}

/* Multiple properties */
.element {
  transition: 
    width 0.3s ease-out,
    height 0.3s ease-out,
    background-color 0.5s ease-in;
}

/* All properties */
.element {
  transition: all 0.3s ease;
}
```

⚠️ **Best Practice:** It's best practice to specify each property individually. Finer control will lead to better performance and more predictable outcomes

```css
/* ❌ Avoid */
.element {
  transition: all 0.5s;  /* Can cause unintended transitions */
}

/* ✅ Better */
.element {
  transition: transform 0.3s, opacity 0.3s;
}
```

---

## 💡 Common Transition Patterns

### 1. Button Hover

```css
.button {
  background-color: #3b82f6;
  color: white;
  transform: scale(1);
  transition: background-color 0.2s, transform 0.2s;
}

.button:hover {
  background-color: #2563eb;
  transform: scale(1.05);
}
```

### 2. Fade In/Out

```css
.modal {
  opacity: 0;
  transition: opacity 0.3s ease-out;
}

.modal.visible {
  opacity: 1;
}
```

### 3. Slide Menu

```css
.menu {
  transform: translateX(-100%);
  transition: transform 0.3s ease-out;
}

.menu.open {
  transform: translateX(0);
}
```

### 4. Card Lift

```css
.card {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transform: translateY(0);
  transition: box-shadow 0.3s, transform 0.3s;
}

.card:hover {
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  transform: translateY(-4px);
}
```

### 5. Link Underline

```css
.link {
  position: relative;
  text-decoration: none;
}

.link::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background-color: currentColor;
  transition: width 0.3s ease-out;
}

.link:hover::after {
  width: 100%;
}
```

---

## 🎭 Multi-Step Transitions (CSS-Tricks)

We can make things more interesting by chaining our transitions together using commas, then playing with the duration and delay to create multi-step movement effects

```css
.box {
  width: 150px;
  height: 150px;
  background-color: red;
  box-shadow: 5px 5px 25px #000;
  transition: 
    /* Step 1: Width changes immediately, takes 1s */
    width 1s,
    /* Step 2: Color changes after width (1s delay), takes 1s */
    background-color 1s 1s,
    /* Step 3: Shadow changes after color (2s delay), takes 1s */
    box-shadow 1s 2s;
}

.box:hover {
  width: 300px;
  background-color: orange;
  box-shadow: -5px 5px 25px #000;
}
```

---

## 🔄 Transition Direction Control

By specifying the transition on the element itself, you define the transition to occur in both directions. That is, when styles are changed (on hover on), the properties will transition, and when styles change back (on hover off) they will transition

```css
/* Same transition in both directions */
.button {
  background: blue;
  transition: background 0.3s;
}

.button:hover {
  background: red;
}

/* Different transitions for each direction */
.button {
  background: blue;
  transition: background 0.3s;  /* Hover off: 0.3s */
}

.button:hover {
  background: red;
  transition: background 0.8s;  /* Hover on: 0.8s */
}
```

---

## ⚠️ The "Doom Flicker" Problem (CSS-Tricks)

When an element is moved up or down on hover, we need to be very careful we don't accidentally introduce a "doom flicker"

**The Problem:**
```css
/* ❌ BAD - Creates flicker */
.button {
  transition: transform 0.3s;
}

.button:hover {
  transform: translateY(-10px);
  /* Element moves away from cursor, loses hover, 
     falls back down, regains hover... infinite loop! */
}
```

**The Solution:**
The trick is to separate the trigger from the effect

```css
/* ✅ GOOD - Trigger on parent */
.button-wrapper:hover .button {
  transform: translateY(-10px);
}

/* Or use padding/margin to keep hover area */
.button {
  padding: 20px;  /* Hover area stays when button moves */
  transition: transform 0.3s;
}

.button:hover {
  transform: translateY(-10px);
}
```

---

## 🆕 Transitioning `display` (Modern CSS)

Previously impossible, now possible with modern CSS!

When transitioning display, @starting-style is needed to provide a set of starting values for properties set on an element that you want to transition from when the element receives its first style update

```css
.dialog {
  opacity: 0;
  transform: translateY(-20px);
  display: none;
  
  transition: 
    opacity 0.3s,
    transform 0.3s,
    display 0.3s allow-discrete;  /* New! */
}

.dialog.open {
  opacity: 1;
  transform: translateY(0);
  display: block;
}

/* Starting styles */
@starting-style {
  .dialog.open {
    opacity: 0;
    transform: translateY(-20px);
  }
}
```

---

## 🎮 Transitions with JavaScript

### Detecting Transition Events

```javascript
const element = document.querySelector('.box')

// When transition ends
element.addEventListener('transitionend', (e) => {
  console.log(`${e.propertyName} transition ended`)
})

// When transition starts
element.addEventListener('transitionstart', (e) => {
  console.log(`${e.propertyName} transition started`)
})

// When transition is cancelled
element.addEventListener('transitioncancel', (e) => {
  console.log(`${e.propertyName} transition cancelled`)
})
```

### Pausing Transitions (CSS-Tricks)

To pause an element's transition, use getComputedStyle and getPropertyValue at the point in the transition you want to pause it. Then set those CSS properties of that element equal to those values you just got

```javascript
const element = document.querySelector('.box')

// Pause transition
function pauseTransition() {
  const computedStyle = window.getComputedStyle(element)
  const currentTransform = computedStyle.getPropertyValue('transform')
  
  // Remove transition
  element.style.transition = 'none'
  
  // Set current computed value
  element.style.transform = currentTransform
}

// Resume transition
function resumeTransition() {
  element.style.transition = 'transform 0.3s'
  element.style.transform = 'translateX(200px)'
}
```

---

## ⚡ Performance Best Practices

When working with CSS transitions, you may encounter performance issues if you add transitions for certain CSS properties. When possible, we recommend using properties like transform and opacity instead

### Properties Safe to Transition (60fps):
✅ `transform` (translate, scale, rotate)
✅ `opacity`
✅ `filter` (use sparingly)

### Properties to Avoid:
❌ `width`, `height` (causes layout reflow)
❌ `top`, `left`, `right`, `bottom`
❌ `margin`, `padding`
❌ `border-width`

```css
/* ❌ BAD - Causes layout reflow */
.box {
  width: 100px;
  transition: width 0.3s;
}

.box:hover {
  width: 200px;
}

/* ✅ GOOD - Uses transform (GPU accelerated) */
.box {
  transform: scaleX(1);
  transition: transform 0.3s;
}

.box:hover {
  transform: scaleX(2);
}
```

---

## 🎯 Transition vs Animation: When to Use Which?

### Use **Transitions** when:
- Simple A → B state changes
- Hover/focus/active states
- Click interactions
- Two states only
- Automatic reverse needed

### Use **Animations** when:
- Complex multi-step sequences
- Looping animations
- Need precise keyframe control
- More than 2 states
- Auto-play on load

```css
/* Transition: Simple hover */
.button {
  background: blue;
  transition: background 0.3s;
}
.button:hover {
  background: red;
}

/* Animation: Complex sequence */
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  25% { transform: scale(1.1); }
  50% { transform: scale(0.9); }
  75% { transform: scale(1.05); }
}
.element {
  animation: pulse 2s infinite;
}
```

---

# Part 2: CSS Animations

## 🎬 What are CSS Animations?

CSS animations make it possible to animate transitions from one CSS style configuration to another. Animations consist of two components: a style describing the CSS animation and a set of keyframes that indicate the start and end states of the animation's style, as well as possible intermediate waypoints

### Why Use CSS Animations?

There are three key advantages to CSS animations over traditional script-driven animation techniques:
1. You can create basic animations with a few lines of CSS; no JavaScript required
2. The animations run well, even under moderate system load
3. Letting the browser control the animation sequence lets the browser optimize performance and efficiency by reducing the update frequency of animations running in tabs that aren't currently visible

---

## 🔑 Two Core Components

### 1. The `animation` Property
Controls WHEN and HOW the animation runs (timing, duration, iterations, etc.)

### 2. The `@keyframes` Rule  
Defines WHAT changes during the animation (the actual appearance)

---

## 📝 Basic Syntax

Each animation needs to be defined with the @keyframes at-rule which is then called with the animation property

```css
/* Step 1: Define the animation */
@keyframes pulse {
  0% {
    background-color: #001F3F;
  }
  100% {
    background-color: #FF4136;
  }
}

/* Step 2: Apply it to an element */
.element {
  animation: pulse 5s infinite;
}
```

---

## 🎨 The @keyframes Rule

Each keyframe describes how the animated element should render at a given time during the animation sequence. Keyframes use a percentage to indicate the time during the animation sequence at which they take place. 0% indicates the first moment of the animation sequence, while 100% indicates the final state

### Using Percentages

```css
@keyframes slide {
  0% {
    transform: translateX(0);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translateX(300px);
    opacity: 0;
  }
}
```

### Using `from` and `to` Keywords

Because 0% and 100% are so important, they have special aliases: from and to

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

### Multiple Properties

```css
@keyframes complexAnimation {
  0% {
    transform: scale(1) rotate(0deg);
    background-color: red;
    opacity: 1;
  }
  50% {
    transform: scale(1.5) rotate(180deg);
    background-color: blue;
    opacity: 0.5;
  }
  100% {
    transform: scale(1) rotate(360deg);
    background-color: green;
    opacity: 1;
  }
}
```

---

## ⚙️ Animation Properties

The animation shorthand property or its eight sub-properties can control how keyframes should be manipulated

### 1. `animation-name`

Declares the name of the @keyframes at-rule to manipulate

```css
.element {
  animation-name: fadeIn;
}
```

### 2. `animation-duration`

The length of time it takes for an animation to complete one cycle

```css
.element {
  animation-duration: 2s;      /* 2 seconds */
  animation-duration: 500ms;   /* 500 milliseconds */
}
```

⚠️ **Important**: Without `animation-duration`, no animation will occur (default is `0s`).

### 3. `animation-timing-function`

Establishes preset acceleration curves such as ease or linear

```css
.element {
  /* Preset values */
  animation-timing-function: linear;       /* Constant speed */
  animation-timing-function: ease;         /* Slow start, fast, slow end (default) */
  animation-timing-function: ease-in;      /* Slow start */
  animation-timing-function: ease-out;     /* Slow end */
  animation-timing-function: ease-in-out;  /* Slow start and end */
  
  /* Custom bezier curves */
  animation-timing-function: cubic-bezier(0.42, 0, 0.58, 1);
  
  /* Step function */
  animation-timing-function: steps(4, end);
}
```

### 4. `animation-delay`

The time between the element being loaded and the start of the animation sequence

```css
.element {
  animation-delay: 1s;      /* Wait 1 second before starting */
  animation-delay: -0.5s;   /* Start halfway through (negative delay) */
}
```

**Negative Delays** (CSS-Tricks trick):

A negative animation delay starts the animation immediately, as if that amount of time has already gone by. In other words, start the animation at a state further into the animation cycle

```css
/* Each circle starts at a different point in the animation */
.circle:nth-child(1) { animation-delay: 0s; }
.circle:nth-child(2) { animation-delay: -0.5s; }
.circle:nth-child(3) { animation-delay: -1s; }
```

### 5. `animation-iteration-count`

The number of times the animation should be performed

```css
.element {
  animation-iteration-count: 1;        /* Once (default) */
  animation-iteration-count: 3;        /* Three times */
  animation-iteration-count: infinite; /* Forever */
}
```

### 6. `animation-direction`

Sets the direction of the animation after the cycle. Its default resets on each cycle

```css
.element {
  animation-direction: normal;            /* Forward (default) */
  animation-direction: reverse;           /* Backward */
  animation-direction: alternate;         /* Forward, then backward */
  animation-direction: alternate-reverse; /* Backward, then forward */
}
```

**Example:**
```css
@keyframes slide {
  from { transform: translateX(0); }
  to { transform: translateX(100px); }
}

.normal {
  animation: slide 2s infinite normal;
  /* 0→100, 0→100, 0→100 */
}

.alternate {
  animation: slide 2s infinite alternate;
  /* 0→100, 100→0, 0→100 */
}
```

### 7. `animation-fill-mode`

Sets which values are applied before/after the animation. For example, you can set the last state of the animation to remain on screen, or you can set it to switch back to before when the animation began

```css
.element {
  animation-fill-mode: none;      /* Default - no styles applied */
  animation-fill-mode: forwards;  /* Keep final state */
  animation-fill-mode: backwards; /* Apply first keyframe during delay */
  animation-fill-mode: both;      /* Both forwards and backwards */
}
```

**Visual Example:**
```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.without-forwards {
  animation: fadeIn 1s;
  /* After animation: jumps back to original opacity */
}

.with-forwards {
  animation: fadeIn 1s forwards;
  /* After animation: stays at opacity: 1 ✓ */
}
```

### 8. `animation-play-state`

Specifies whether to pause or play an animation sequence

```css
.element {
  animation-play-state: running; /* Default */
  animation-play-state: paused;
}

/* Pause on hover */
.element:hover {
  animation-play-state: paused;
}
```

---

## 🚀 The Shorthand Property

```css
.element {
  animation: name duration timing-function delay iteration-count direction fill-mode play-state;
}
```

**Example:**
```css
.element {
  animation: fadeIn 2s ease-in-out 0.5s infinite alternate both running;
}

/* Equivalent to: */
.element {
  animation-name: fadeIn;
  animation-duration: 2s;
  animation-timing-function: ease-in-out;
  animation-delay: 0.5s;
  animation-iteration-count: infinite;
  animation-direction: alternate;
  animation-fill-mode: both;
  animation-play-state: running;
}
```

**Shorter version (most common):**
```css
.element {
  animation: fadeIn 2s ease-in-out;
  /* name duration timing-function */
}
```

---

## 🎯 Multiple Animations

The CSS animation longhand properties can accept multiple values, separated by commas. This feature can be used when you want to apply multiple animations in a single rule

```css
.element {
  animation: 
    fadeIn 2s ease-out,
    slide 3s linear,
    rotate 1s infinite;
}

/* Or with longhand: */
.element {
  animation-name: fadeIn, slide, rotate;
  animation-duration: 2s, 3s, 1s;
  animation-iteration-count: 1, 1, infinite;
}
```

---

## 💡 Best Practices & Performance

### ✅ DO: Animate Transform and Opacity

Animating CSS Box Model properties is discouraged. Animating any box model property is inherently CPU intensive; consider animating the transform property instead

```css
/* ✅ GOOD - Hardware accelerated */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-100px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* ❌ BAD - Causes reflow/repaint */
@keyframes slideInBad {
  from {
    left: -100px;
  }
  to {
    left: 0;
  }
}
```

### Properties Safe to Animate (60fps):
- `opacity`
- `transform` (translate, scale, rotate, skew)
- `filter` (with caution)

### Properties to Avoid:
- `width`, `height`
- `top`, `left`, `right`, `bottom`
- `margin`, `padding`
- `border-width`

---

## 🎨 Common Animation Patterns

### 1. Fade In

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-in {
  animation: fadeIn 0.5s ease-out forwards;
}
```

### 2. Slide In

```css
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.slide-in {
  animation: slideIn 0.6s ease-out forwards;
}
```

### 3. Bounce

```css
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
}

.bounce {
  animation: bounce 2s;
}
```

### 4. Rotate

```css
@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.rotate {
  animation: rotate 2s linear infinite;
}
```

### 5. Pulse

```css
@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

.pulse {
  animation: pulse 1s ease-in-out infinite;
}
```

### 6. Shake

```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-10px); }
  20%, 40%, 60%, 80% { transform: translateX(10px); }
}

.shake {
  animation: shake 0.5s;
}
```

---

## 🎭 Advanced Techniques

### 1. State Jumping (CSS-Tricks)

CSS animation makes it easy to transition properties to a new value over time. They also have the ability to jump properties to a new value virtually instantly. The trick is to use two keyframes with a very small difference, around .001% works well

```css
@keyframes toggleOpacity {
  50% {
    opacity: 1;
  }
  /* Jump instantly */
  50.001% {
    opacity: 0.4;
  }
  /* Keep off state for a short period */
  52.999% {
    opacity: 0.4;
  }
  /* Jump back */
  53% {
    opacity: 1;
  }
}
```

### 2. Animating transform-origin

Not only can transformation-origin be changed mid-animation, it is also animatable! This allows us to create one animation using rotations on different axes instead of using four separate animations

```css
@keyframes multiAxisRotate {
  0% {
    transform: rotate(0deg);
    transform-origin: center;
  }
  25% {
    transform: rotate(90deg);
    transform-origin: top left;
  }
  50% {
    transform: rotate(180deg);
    transform-origin: top right;
  }
  75% {
    transform: rotate(270deg);
    transform-origin: bottom right;
  }
  100% {
    transform: rotate(360deg);
    transform-origin: center;
  }
}
```

### 3. Staggered Animations (CSS-Tricks)

The core idea involves adding a simple CSS @keyframes animation that's applied to anything we want to animate on page load with animation-fill-mode: backwards to make sure that our initial animation state is active on page load

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.stagger-item {
  animation: fadeInUp 0.6s ease-out backwards;
}

.stagger-item:nth-child(1) { animation-delay: 0.1s; }
.stagger-item:nth-child(2) { animation-delay: 0.2s; }
.stagger-item:nth-child(3) { animation-delay: 0.3s; }
.stagger-item:nth-child(4) { animation-delay: 0.4s; }
```

### 4. Responsive Animations (CSS-Tricks)

Responsiveness for CSS animations is possible using percentages and other relative units

```css
@keyframes responsiveSlide {
  from {
    transform: translateX(-50vw); /* viewport-relative */
  }
  to {
    transform: translateX(0);
  }
}
```

---

## 🎬 Animation on Page Load

This is framework-agnostic because the element's animation will trigger once it's inserted into the DOM or its display property goes from none to visible

```css
.animate-on-load {
  opacity: 0;
  animation: fadeIn 1s ease-out 0.5s forwards;
}

@keyframes fadeIn {
  to {
    opacity: 1;
  }
}
```

---

## 🔄 Animation with JavaScript

### Detecting Animation Events

```javascript
const element = document.querySelector('.animated')

// When animation starts
element.addEventListener('animationstart', (e) => {
  console.log('Animation started:', e.animationName)
})

// When animation ends
element.addEventListener('animationend', (e) => {
  console.log('Animation ended:', e.animationName)
  // Do something after animation
})

// When animation repeats
element.addEventListener('animationiteration', (e) => {
  console.log('Animation iteration:', e.animationName)
})
```

### Adding/Removing Animations Dynamically

```javascript
const element = document.querySelector('.my-element')

// Add animation
element.classList.add('animate__animated', 'animate__bounceIn')

// Remove after animation ends
element.addEventListener('animationend', () => {
  element.classList.remove('animate__animated', 'animate__bounceIn')
}, { once: true })
```

---

## 🎨 Animating Non-Animatable Properties

### Using @property (Modern CSS)

The @property directive allows you to register custom properties for animation. When you use @property, you must provide it with three values: syntax, inherits, and initial-value

```css
/* Register custom properties */
@property --color-1 {
  syntax: '<color>';
  inherits: true;
  initial-value: red;
}

@property --color-2 {
  syntax: '<color>';
  inherits: true;
  initial-value: blue;
}

/* Now animate gradient colors */
button {
  background: linear-gradient(90deg, var(--color-1), var(--color-2));
  transition: --color-1 1s, --color-2 1s;
}

button:hover {
  --color-1: green;
  --color-2: yellow;
}
```

---

## ♿ Accessibility: Respecting User Preferences

```css
/* Disable animations for users who prefer reduced motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Or selectively: */
@media (prefers-reduced-motion: reduce) {
  .fancy-animation {
    animation: none;
  }
}
```

---

## 🐛 Common Pitfalls

### 1. Forgetting `animation-duration`
```css
/* ❌ Won't work - no duration */
.element {
  animation-name: fadeIn;
}

/* ✅ Works */
.element {
  animation: fadeIn 1s;
}
```

### 2. Not Using `forwards` Fill Mode
```css
/* ❌ Jumps back to original state */
@keyframes fadeIn {
  to { opacity: 1; }
}
.element {
  opacity: 0;
  animation: fadeIn 1s;
}

/* ✅ Stays at final state */
.element {
  opacity: 0;
  animation: fadeIn 1s forwards;
}
```

### 3. Animating Layout Properties
```css
/* ❌ Causes layout thrashing */
@keyframes badSlide {
  to { margin-left: 100px; }
}

/* ✅ Uses compositing */
@keyframes goodSlide {
  to { transform: translateX(100px); }
}
```

---

## 🎯 Performance Checklist

✅ **Only animate:**
- `transform`
- `opacity`
- `filter` (use sparingly)

✅ **Use `will-change` for complex animations:**
```css
.heavy-animation {
  will-change: transform, opacity;
}
```

✅ **Remove `will-change` after animation:**
```javascript
element.addEventListener('animationend', () => {
  element.style.willChange = 'auto'
})
```

✅ **Use hardware acceleration:**
```css
.element {
  transform: translateZ(0); /* Force GPU */
}
```

❌ **Avoid:**
- Animating many elements simultaneously
- Long animation chains
- Animations on scroll (use Intersection Observer instead)

---

## 📚 Quick Reference

### Shorthand Order
```css
animation: name | duration | timing-function | delay | iteration-count | direction | fill-mode | play-state;
```

### Common Durations
- **0.15s** - Very fast (micro-interactions)
- **0.3s** - Fast (button hover)
- **0.5s** - Medium (modal open)
- **1s** - Slow (page transitions)
- **2s+** - Very slow (decorative)

### Common Timing Functions
- `ease` - Default, natural feeling
- `linear` - Constant speed (loading spinners)
- `ease-in` - Accelerating
- `ease-out` - Decelerating (most UI animations)
- `ease-in-out` - Smooth start and end
- `cubic-bezier(0.68, -0.55, 0.265, 1.55)` - Bouncy effect

---

## 🌐 Browser Support

**Excellent support (98%+)**
- All modern browsers fully support CSS animations
- Use vendor prefixes for older browsers:
  ```css
  @-webkit-keyframes fadeIn { /* ... */ }
  @keyframes fadeIn { /* ... */ }
  
  .element {
    -webkit-animation: fadeIn 1s;
    animation: fadeIn 1s;
  }
  ```

---

## 🔗 Resources

- [MDN: Using CSS Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)
- [CSS-Tricks: Animation Property](https://css-tricks.com/almanac/properties/a/animation/)
- [MDN: Animatable Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties)
- [Animate.css](https://animate.style/) - Ready-to-use animations
- [Easing Functions Cheat Sheet](https://easings.net/)

---

## 💡 Pro Tips

1. **Start simple** - Master fade and slide before complex animations
2. **Less is more** - Subtle animations are more professional
3. **Consistent timing** - Use the same durations across your site
4. **Test performance** - Use Chrome DevTools Performance tab
5. **Think mobile** - Animations should work on slow devices
6. **Respect preferences** - Always implement `prefers-reduced-motion`
7. **Use shortcuts** - Libraries like Animate.css for rapid prototyping
8. **Combine with transitions** - Use animations for complex sequences, transitions for simple state changes