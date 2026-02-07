# Web Animations API: Complete Guide to `animate()`

## 🎯 What is the Web Animations API?

The Web Animations API is a **native browser API** that lets you create and control animations with JavaScript. It combines the power of CSS animations with JavaScript control, giving you the best of both worlds.

Think of it as: **CSS Animations + JavaScript Control = Web Animations API**

---

## 📝 Basic Syntax

```javascript
element.animate(keyframes, options)
```

**Returns:** An `Animation` object that you can control

---

## 🔑 The Two Parts

### 1. **Keyframes** (What to animate)
Defines the animation states from start to finish.

### 2. **Options** (How to animate)
Defines duration, timing, iterations, etc.

---

## 📚 Keyframes Explained

### Array Format (Most Common)

```javascript
element.animate([
  { opacity: 0, transform: 'translateY(-20px)' },  // Start (0%)
  { opacity: 1, transform: 'translateY(0)' }       // End (100%)
], {
  duration: 1000  // 1 second
})
```

### Object Format (With Explicit Offsets)

```javascript
element.animate([
  { opacity: 0, offset: 0 },      // 0% of animation
  { opacity: 0.5, offset: 0.3 },  // 30% of animation
  { opacity: 1, offset: 1 }       // 100% of animation
], {
  duration: 1000
})
```

### Multiple Properties

```javascript
element.animate([
  {
    opacity: 0,
    transform: 'scale(0.5) rotate(0deg)',
    backgroundColor: 'red'
  },
  {
    opacity: 1,
    transform: 'scale(1) rotate(360deg)',
    backgroundColor: 'blue'
  }
], {
  duration: 2000
})
```

---

## ⚙️ Options Explained

### Essential Options

```javascript
{
  duration: 1000,           // Animation length in milliseconds
  iterations: 1,            // How many times to repeat (Infinity for infinite)
  direction: 'normal',      // 'normal', 'reverse', 'alternate', 'alternate-reverse'
  easing: 'ease-in-out',    // Timing function
  delay: 0,                 // Delay before starting (ms)
  endDelay: 0,              // Delay after ending (ms)
  fill: 'auto'              // 'none', 'forwards', 'backwards', 'both', 'auto'
}
```

### Fill Mode Explained

```javascript
// fill: 'none' (default)
// Element returns to original state after animation

// fill: 'forwards'
// Element stays at final keyframe after animation
element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 1000,
  fill: 'forwards'  // Stays at opacity: 1
})

// fill: 'backwards'
// Element jumps to first keyframe immediately (even during delay)

// fill: 'both'
// Combines 'forwards' and 'backwards'
```

### Easing Options

```javascript
{
  easing: 'linear'              // Constant speed
  easing: 'ease'                // Slow start, fast middle, slow end (default)
  easing: 'ease-in'             // Slow start
  easing: 'ease-out'            // Slow end
  easing: 'ease-in-out'         // Slow start and end
  easing: 'cubic-bezier(0.4, 0, 0.2, 1)'  // Custom bezier
  easing: 'steps(4, end)'       // Stepped animation
}
```

---

## 🎮 Controlling Animations

### The Animation Object

```javascript
const animation = element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 1000
})

// Control methods
animation.play()       // Start/resume
animation.pause()      // Pause
animation.reverse()    // Play backwards
animation.cancel()     // Stop and reset
animation.finish()     // Jump to end

// Properties
animation.playState    // 'idle', 'running', 'paused', 'finished'
animation.playbackRate // Speed (1 = normal, 2 = 2x speed, -1 = reverse)
animation.currentTime  // Current position in ms
```

### Practical Control Example

```javascript
const box = document.querySelector('.box')

const animation = box.animate([
  { transform: 'translateX(0px)' },
  { transform: 'translateX(300px)' }
], {
  duration: 2000,
  fill: 'forwards'
})

// Pause on click
box.addEventListener('click', () => {
  if (animation.playState === 'running') {
    animation.pause()
  } else {
    animation.play()
  }
})

// Speed control
document.querySelector('#speed-up').addEventListener('click', () => {
  animation.playbackRate = 2  // 2x speed
})
```

---

## 🎭 Promises & Events

### Using Promises

```javascript
const animation = element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 1000
})

// Wait for animation to finish
animation.finished.then(() => {
  console.log('Animation completed!')
  // Do something after animation
})

// Using async/await
async function animateSequence() {
  const anim1 = element1.animate([...], {...})
  await anim1.finished
  
  const anim2 = element2.animate([...], {...})
  await anim2.finished
  
  console.log('Both animations done!')
}
```

### Animation Events

```javascript
animation.onfinish = () => {
  console.log('Animation finished')
}

animation.oncancel = () => {
  console.log('Animation cancelled')
}

animation.onremove = () => {
  console.log('Animation removed')
}
```

---

## 💡 Common Use Cases

### 1. Fade In

```javascript
element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 500,
  fill: 'forwards'
})
```

### 2. Slide In From Left

```javascript
element.animate([
  { transform: 'translateX(-100%)' },
  { transform: 'translateX(0)' }
], {
  duration: 600,
  easing: 'ease-out',
  fill: 'forwards'
})
```

### 3. Pulse Effect

```javascript
element.animate([
  { transform: 'scale(1)' },
  { transform: 'scale(1.2)' },
  { transform: 'scale(1)' }
], {
  duration: 600,
  iterations: Infinity
})
```

### 4. Shake Animation

```javascript
element.animate([
  { transform: 'translateX(0)' },
  { transform: 'translateX(-10px)' },
  { transform: 'translateX(10px)' },
  { transform: 'translateX(-10px)' },
  { transform: 'translateX(10px)' },
  { transform: 'translateX(0)' }
], {
  duration: 400
})
```

### 5. Loading Spinner

```javascript
spinner.animate([
  { transform: 'rotate(0deg)' },
  { transform: 'rotate(360deg)' }
], {
  duration: 1000,
  iterations: Infinity,
  easing: 'linear'
})
```

### 6. Sequential Animations

```javascript
async function sequentialAnimate() {
  // Fade in
  const fadeIn = element.animate([
    { opacity: 0 },
    { opacity: 1 }
  ], { duration: 500, fill: 'forwards' })
  
  await fadeIn.finished
  
  // Then slide
  const slide = element.animate([
    { transform: 'translateY(0)' },
    { transform: 'translateY(50px)' }
  ], { duration: 500, fill: 'forwards' })
  
  await slide.finished
  
  // Then rotate
  const rotate = element.animate([
    { transform: 'translateY(50px) rotate(0deg)' },
    { transform: 'translateY(50px) rotate(360deg)' }
  ], { duration: 800, fill: 'forwards' })
}
```

---

## 🆚 Comparison: Web Animations API vs CSS vs Other Methods

### CSS Animations
```css
.box {
  animation: fadeIn 1s ease-in-out forwards;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

### Web Animations API
```javascript
box.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 1000,
  easing: 'ease-in-out',
  fill: 'forwards'
})
```

### Comparison

| Feature | CSS | Web Animations API | setTimeout/setInterval |
|---------|-----|-------------------|----------------------|
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| **Control** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Promises** | ❌ | ✅ | ✅ (manual) |
| **Dynamic Values** | ❌ | ✅ | ✅ |
| **Browser Support** | ✅ Excellent | ✅ Good (95%+) | ✅ Excellent |

---

## ✅ Advantages of Web Animations API

1. **JavaScript Control with CSS Performance**
   - Hardware accelerated (GPU)
   - Smooth 60fps animations

2. **Promises Support**
   - Easy to chain animations
   - Async/await friendly

3. **Dynamic Animations**
   - Create animations based on user input
   - Calculate values at runtime

4. **Full Control**
   - Play, pause, reverse, cancel
   - Change speed on the fly
   - Jump to any point in timeline

5. **No CSS Required**
   - Everything in JavaScript
   - Better for dynamic SPAs

---

## 🎯 Best Practices

### 1. Use `transform` and `opacity` for Performance

```javascript
// ✅ Good (GPU accelerated)
element.animate([
  { opacity: 0, transform: 'translateX(0)' },
  { opacity: 1, transform: 'translateX(100px)' }
], { duration: 1000 })

// ❌ Bad (causes reflow)
element.animate([
  { width: '100px', height: '100px' },
  { width: '200px', height: '200px' }
], { duration: 1000 })
```

### 2. Always Use `fill: 'forwards'` If You Want Final State

```javascript
element.animate([
  { opacity: 0 },
  { opacity: 1 }
], {
  duration: 1000,
  fill: 'forwards'  // ← Important!
})
```

### 3. Clean Up Animations

```javascript
let currentAnimation = null

function animateElement() {
  // Cancel previous animation
  if (currentAnimation) {
    currentAnimation.cancel()
  }
  
  currentAnimation = element.animate([...], {...})
}
```

### 4. Use Promises for Sequences

```javascript
// ✅ Clean and readable
async function animateSequence() {
  await element.animate([...], {...}).finished
  await element.animate([...], {...}).finished
  await element.animate([...], {...}).finished
}

// ❌ Callback hell
element.animate([...], {...}).finished.then(() => {
  element.animate([...], {...}).finished.then(() => {
    element.animate([...], {...})
  })
})
```

---

## 🌐 Browser Support

**Supported:** 96%+ of browsers (as of 2025)
- ✅ Chrome 36+
- ✅ Firefox 48+
- ✅ Safari 13.1+
- ✅ Edge 79+

**Polyfill Available:** `web-animations-js` for older browsers

---

## 📖 Summary

The Web Animations API's `animate()` method is:
- **Native browser API** (no library needed)
- **Performant** (hardware accelerated)
- **Powerful** (full JavaScript control)
- **Promise-based** (easy async handling)
- **Modern** (cleaner than setTimeout/setInterval)

**Use it when:**
- You need JavaScript control over animations
- You want to create dynamic animations
- You need to chain multiple animations
- You want better performance than manipulating styles directly

**Stick with CSS when:**
- Simple hover effects
- Static, predefined animations
- You want maximum browser support