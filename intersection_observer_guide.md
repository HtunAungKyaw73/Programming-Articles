# Intersection Observer API: Complete Guide
*Based on MDN Web Docs*

## 🎯 What is Intersection Observer API?

The Intersection Observer API provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or with a top-level document's viewport.

**In simple terms:** It lets you know when an element enters or exits the viewport (or another container) without constantly checking scroll position.

### Why Was It Created?

Historically, detecting visibility of an element has been a difficult task for which solutions have been unreliable and prone to causing the browser and the sites the user is accessing to become sluggish.

**The old way (❌ Bad):**
```javascript
// Don't do this!
window.addEventListener('scroll', () => {
  const rect = element.getBoundingClientRect()
  if (rect.top >= 0 && rect.bottom <= window.innerHeight) {
    // Element is visible
  }
})
// Problems: Runs constantly, causes performance issues
```

**The new way (✅ Good):**
```javascript
// Much better!
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Element is visible
    }
  })
})
observer.observe(element)
// Benefits: Efficient, asynchronous, optimized by browser
```

---

## 📚 Core Concepts (From MDN)

### 1. **Root Element**
The specified element is called the root element or root for the purposes of the Intersection Observer API.

- **`root: null`** → Uses the viewport (most common)
- **`root: document.querySelector('.container')`** → Uses a specific container

### 2. **Intersection Ratio**
The degree of intersection between the target element and its root is the intersection ratio.

- **0.0** = Not visible at all
- **0.5** = 50% visible
- **1.0** = 100% visible

### 3. **Thresholds**
A list of thresholds, sorted in increasing numeric order, where each threshold is a ratio of intersection area to bounding box area of an observed target.

```javascript
{
  threshold: 0.5  // Callback fires when 50% visible
}

{
  threshold: [0, 0.25, 0.5, 0.75, 1.0]  // Fires at each 25%
}
```

---

## 🔧 Complete Syntax

### Constructor

The IntersectionObserver() constructor creates and returns a new IntersectionObserver object.

```javascript
const observer = new IntersectionObserver(callback, options)
```

### Callback Function

```javascript
const callback = (entries, observer) => {
  entries.forEach(entry => {
    // entry properties:
    entry.isIntersecting      // Boolean: is element visible?
    entry.intersectionRatio   // Number: 0.0 to 1.0
    entry.target              // The observed element
    entry.boundingClientRect  // Element's position
    entry.intersectionRect    // Visible portion
    entry.rootBounds          // Root container bounds
    entry.time                // Timestamp
  })
}
```

### Options Object

An optional object which customizes the observer. You can provide any combination (or none) of the following options.

```javascript
const options = {
  root: null,              // Element or null (viewport)
  rootMargin: '0px',       // Margin around root
  threshold: 0.5,          // When to trigger (0.0 - 1.0)
  
  // Advanced options:
  scrollMargin: '0px',     // Margin for nested scrollers
  trackVisibility: true,   // Track actual visibility
  delay: 100              // Minimum delay between notifications (ms)
}
```

### Methods

The IntersectionObserver interface provides methods to observe and stop observing target elements.

```javascript
observer.observe(element)        // Start watching an element
observer.unobserve(element)      // Stop watching an element
observer.disconnect()            // Stop watching all elements
observer.takeRecords()           // Get pending entries
```

---

## 🎯 Real-World Use Cases (From MDN)

### 1. **Lazy Loading Images** ⭐ Most Common

Lazy-loading of images or other content as a page is scrolled.

```javascript
const imageObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target
      img.src = img.dataset.src  // Load the actual image
      img.classList.remove('lazy')
      observer.unobserve(img)    // Stop watching once loaded
    }
  })
}, {
  rootMargin: '50px'  // Start loading 50px before visible
})

// Observe all lazy images
document.querySelectorAll('img.lazy').forEach(img => {
  imageObserver.observe(img)
})
```

**HTML:**
```html
<img class="lazy" data-src="high-res.jpg" src="placeholder.jpg" alt="Description">
```

---

### 2. **Infinite Scroll**

Implementing "infinite scrolling" websites, where more and more content is loaded and rendered as you scroll, so that the user doesn't have to flip through pages.

```javascript
const sentinelObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadMoreContent()
    }
  })
}, {
  rootMargin: '100px'  // Trigger 100px before reaching bottom
})

// Observe a sentinel element at the bottom
const sentinel = document.querySelector('.scroll-sentinel')
sentinelObserver.observe(sentinel)

function loadMoreContent() {
  // Fetch and append new content
  fetch('/api/posts?page=' + currentPage)
    .then(res => res.json())
    .then(data => {
      appendPosts(data)
      currentPage++
    })
}
```

---

### 3. **Ad Visibility Tracking**

Reporting of visibility of advertisements in order to calculate ad revenues.

There's a good reason why the notion of tracking visibility of ads is being used in this example. It turns out that one of the most common uses of Flash or other script in advertising on the Web is to record how long each ad is visible, for the purpose of billing and payment of revenues.

```javascript
const adObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const ad = entry.target
    
    if (entry.isIntersecting) {
      // Ad became visible
      ad.dataset.visibleSince = Date.now()
    } else {
      // Ad is no longer visible
      if (ad.dataset.visibleSince) {
        const visibleTime = Date.now() - parseInt(ad.dataset.visibleSince)
        trackAdVisibility(ad.id, visibleTime)
        delete ad.dataset.visibleSince
      }
    }
  })
}, {
  threshold: 0.5  // Consider "visible" when 50% shown
})

document.querySelectorAll('.ad').forEach(ad => {
  adObserver.observe(ad)
})
```

---

### 4. **Scroll Animations**

```javascript
const animateObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('animate-in')
    }
  })
}, {
  threshold: 0.2  // Trigger when 20% visible
})

document.querySelectorAll('.animate-on-scroll').forEach(el => {
  animateObserver.observe(el)
})
```

**CSS:**
```css
.animate-on-scroll {
  opacity: 0;
  transform: translateY(50px);
  transition: opacity 0.6s, transform 0.6s;
}

.animate-on-scroll.animate-in {
  opacity: 1;
  transform: translateY(0);
}
```

---

### 5. **Active Navigation Highlighting**

```javascript
const sectionObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const id = entry.target.id
    const navLink = document.querySelector(`a[href="#${id}"]`)
    
    if (entry.isIntersecting) {
      navLink.classList.add('active')
    } else {
      navLink.classList.remove('active')
    }
  })
}, {
  threshold: 0.5,
  rootMargin: '-100px 0px -66%'  // Only top section is active
})

document.querySelectorAll('section[id]').forEach(section => {
  sectionObserver.observe(section)
})
```

---

### 6. **Video Auto-Play/Pause**

```javascript
const videoObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const video = entry.target
    
    if (entry.isIntersecting) {
      video.play()
    } else {
      video.pause()
    }
  })
}, {
  threshold: 0.5  // Play when 50% visible
})

document.querySelectorAll('video.auto-play').forEach(video => {
  videoObserver.observe(video)
})
```

---

### 7. **Analytics - Track What Users See**

```javascript
const viewTracker = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // User saw this content
      analytics.track('content_viewed', {
        element: entry.target.id,
        visibleRatio: entry.intersectionRatio
      })
      
      // Only track once
      viewTracker.unobserve(entry.target)
    }
  })
}, {
  threshold: 0.8  // 80% visible = "viewed"
})

document.querySelectorAll('.track-view').forEach(el => {
  viewTracker.observe(el)
})
```

---

## 💡 Best Practices

### 1. **Always Unobserve When Done**

```javascript
// ✅ Good - Stop observing after action
const observer = new IntersectionObserver((entries, obs) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadImage(entry.target)
      obs.unobserve(entry.target)  // ← Important!
    }
  })
})

// ❌ Bad - Keeps observing unnecessarily
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadImage(entry.target)
      // Still being observed = wasted resources
    }
  })
})
```

### 2. **Clean Up in React/Vue Components**

```javascript
// React example
useEffect(() => {
  const observer = new IntersectionObserver(callback, options)
  const element = ref.current
  
  if (element) {
    observer.observe(element)
  }
  
  // ✅ Cleanup function
  return () => {
    if (element) {
      observer.unobserve(element)
    }
  }
}, [])
```

### 3. **Use rootMargin for Preloading**

```javascript
// ✅ Good - Start loading before visible
const observer = new IntersectionObserver(callback, {
  rootMargin: '200px'  // Trigger 200px before entering viewport
})

// ❌ Less optimal - Wait until visible
const observer = new IntersectionObserver(callback, {
  rootMargin: '0px'  // User might see loading
})
```

### 4. **Handle Multiple Thresholds Efficiently**

Either a single number or an array of numbers between 0.0 and 1.0, specifying a ratio of intersection area to total bounding box area for the observed target.

```javascript
// ✅ Good - Multiple thresholds for precise control
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.intersectionRatio > 0.9) {
      element.classList.add('fully-visible')
    } else if (entry.intersectionRatio > 0.5) {
      element.classList.add('mostly-visible')
    } else if (entry.intersectionRatio > 0.1) {
      element.classList.add('partially-visible')
    }
  })
}, {
  threshold: [0, 0.1, 0.5, 0.9, 1.0]
})
```

### 5. **Consider Page Visibility API for Ads**

The Intersection Observer API doesn't take this into account when detecting intersection, since intersection isn't affected by page visibility. Therefore, we need to pause our timers while the page is tabbed out.

```javascript
// Handle tab switching for accurate ad tracking
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Pause all ad timers
    pauseAdTimers()
  } else {
    // Resume ad timers
    resumeAdTimers()
  }
})
```

### 6. **Use trackVisibility for Real Visibility**

When tracking visibility, the browser will check that the target does not have compromised visibility when calculating intersections; for example, that it hasn't been covered by another element, have reduced opacity, or be distorted by a filter, transform, or other modification.

```javascript
// For ads or critical content
const observer = new IntersectionObserver(callback, {
  trackVisibility: true,
  delay: 100  // Required when trackVisibility is true
})
```

### 7. **Reuse Observers**

```javascript
// ✅ Good - One observer for many elements
const imageObserver = new IntersectionObserver(callback, options)

document.querySelectorAll('img.lazy').forEach(img => {
  imageObserver.observe(img)
})

// ❌ Bad - New observer for each element
document.querySelectorAll('img.lazy').forEach(img => {
  const observer = new IntersectionObserver(callback, options)
  observer.observe(img)
})
```

### 8. **Check isIntersecting First**

The observer callback will always fire the first render cycle after observe() is called, even if the observed element has not yet moved with respect to the viewport.

```javascript
// ✅ Good - Check isIntersecting
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // Only act when entering
      doSomething()
    }
  })
})

// ⚠️ Be aware - fires immediately on observe()
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // This runs even before scrolling!
    doSomething()
  })
})
```

### 9. **Handle Errors Gracefully**

```javascript
// Check browser support
if ('IntersectionObserver' in window) {
  const observer = new IntersectionObserver(callback, options)
  observer.observe(element)
} else {
  // Fallback: load everything immediately
  loadAllImages()
}
```

### 10. **Optimize Threshold Arrays**

```javascript
// ❌ Bad - Too many thresholds
const observer = new IntersectionObserver(callback, {
  threshold: Array.from({ length: 100 }, (_, i) => i / 100)
})

// ✅ Good - Only what you need
const observer = new IntersectionObserver(callback, {
  threshold: [0, 0.25, 0.5, 0.75, 1.0]
})
```

---

## 🚀 Advanced Patterns

### Lazy Load with Blur Effect

```javascript
const lazyObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target
      const fullImg = new Image()
      
      fullImg.onload = () => {
        img.src = fullImg.src
        img.classList.add('loaded')
      }
      
      fullImg.src = img.dataset.src
      observer.unobserve(img)
    }
  })
}, { rootMargin: '50px' })
```

**CSS:**
```css
img.lazy {
  filter: blur(10px);
  transition: filter 0.3s;
}

img.lazy.loaded {
  filter: blur(0);
}
```

### Pause Expensive Animations

```javascript
const animationObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const animation = entry.target.animation
    
    if (entry.isIntersecting) {
      animation.play()
    } else {
      animation.pause()
    }
  })
}, { threshold: 0.5 })
```

### Progressive Image Loading

```javascript
const progressiveObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target
      
      // Load low quality first
      img.src = img.dataset.lowres
      
      // Then load high quality
      const highResImg = new Image()
      highResImg.onload = () => {
        img.src = highResImg.src
      }
      highResImg.src = img.dataset.highres
      
      observer.unobserve(img)
    }
  })
})
```

---

## 📊 Performance Comparison

### Old Scroll Listener vs Intersection Observer

```javascript
// ❌ OLD WAY - Runs constantly
let ticking = false

window.addEventListener('scroll', () => {
  if (!ticking) {
    window.requestAnimationFrame(() => {
      checkVisibility()  // Expensive!
      ticking = false
    })
    ticking = true
  }
})

// ✅ NEW WAY - Runs only when needed
const observer = new IntersectionObserver(callback)
observer.observe(element)

// Performance difference:
// Scroll listener: 60-100 calls/second
// Intersection Observer: 1-2 calls per visibility change
```

---

## 🌐 Browser Support

**Excellent support (97%+ as of 2025)**
- ✅ Chrome 51+
- ✅ Firefox 55+
- ✅ Safari 12.1+
- ✅ Edge 15+

**Polyfill:** `intersection-observer` for older browsers

```bash
npm install intersection-observer
```

---

## 📖 Summary

The Intersection Observer API is:
- **Performant**: Optimized by browser, doesn't block main thread
- **Asynchronous**: Runs in background
- **Versatile**: Works for lazy loading, infinite scroll, animations, analytics
- **Easy to use**: Simple API, powerful results
- **Well-supported**: Available in all modern browsers

**Use it for:**
- ✅ Lazy loading images/videos
- ✅ Infinite scroll
- ✅ Scroll animations
- ✅ Ad visibility tracking
- ✅ Analytics
- ✅ Auto-play/pause videos
- ✅ Active navigation highlighting

**Avoid using it for:**
- ❌ Precise pixel measurements (use getBoundingClientRect)
- ❌ Continuous scroll position tracking (use scroll events with throttling)
- ❌ Drag and drop (use Pointer Events)

---

# Advanced Concept: rootMargin, threshold & intersectionRatio

## 📐 Part 1: rootMargin and threshold Together

### Understanding Each Separately

#### **rootMargin** (When to check)
- Grows or shrinks the **root's bounding box** before calculating intersection
- Works exactly like CSS margin: `"top right bottom left"`
- **Positive values**: Triggers BEFORE element enters viewport
- **Negative values**: Triggers AFTER element enters viewport

#### **threshold** (How much to see)
- A number between `0.0` and `1.0` (or array of numbers)
- Specifies what percentage of the element must be visible to trigger
- `0` = any pixel visible
- `0.5` = 50% visible
- `1.0` = 100% visible

---

### Why Use Them Together?

**rootMargin** controls **WHEN** to start checking, **threshold** controls **HOW MUCH** needs to be visible.

---

### Example 1: Lazy Load Images Early

**Goal**: Start loading images 200px BEFORE they enter viewport, but only trigger when 50% visible.

```javascript
const imageObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        // This fires when:
        // 1. Element is within 200px of viewport (rootMargin)
        // 2. AND 50% of element is visible (threshold)
        loadImage(entry.target)
      }
    })
  },
  {
    rootMargin: '200px',  // Expand viewport by 200px
    threshold: 0.5        // Need 50% visible
  }
)
```

**Visual Explanation**:
```
Without rootMargin:
┌─────────────────┐
│   Viewport      │ ← Normal viewport boundary
│                 │
│   [Image]       │ ← Image enters here (0% visible)
│                 │
└─────────────────┘

With rootMargin: '200px':
    ┌──────────┐
    │          │ ← Extended boundary (200px above viewport)
    │          │
┌───┼──────────┼───┐
│   │ Viewport │   │ ← Actual viewport
│   │          │   │
│   │ [Image]  │   │ ← Image triggers here (200px before entering)
│   │          │   │
└───┼──────────┼───┘
    │          │
    └──────────┘ ← Extended boundary (200px below viewport)
```

---

### Example 2: Trigger After Element is Fully In View

**Goal**: Only trigger when element is FULLY inside viewport AND 100% visible.

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        // Element is completely inside viewport
        animateElement(entry.target)
      }
    })
  },
  {
    rootMargin: '-50px',  // Shrink viewport by 50px
    threshold: 1.0        // Need 100% visible
  }
)
```

**Visual Explanation**:
```
Without rootMargin (element at edge triggers):
┌─────────────────┐
│   Viewport      │
│   [Element]     │ ← Triggers here (partially visible)
│                 │
└─────────────────┘

With rootMargin: '-50px':
┌─────────────────┐
│                 │
│  ┌──────────┐   │
│  │ Shrunken │   │ ← Element must be fully inside this
│  │ Viewport │   │
│  │          │   │
│  │ [Element]│   │ ← Triggers here (fully inside)
│  │          │   │
│  └──────────┘   │
│                 │
└─────────────────┘
```

---

### Example 3: Different Margins for Different Sides

```javascript
const observer = new IntersectionObserver(
  callback,
  {
    // "top right bottom left" (like CSS)
    rootMargin: '100px 0px 200px 0px',  
    // 100px above viewport
    // 0px on right
    // 200px below viewport
    // 0px on left
    
    threshold: 0.5
  }
)
```

---

### Common Patterns

#### Pattern 1: Preload Content (Images, Ads, Videos)
```javascript
{
  rootMargin: '200px',  // Load before visible
  threshold: 0          // Start as soon as any part enters
}
```

#### Pattern 2: Ad Visibility Tracking (50% rule)
```javascript
{
  rootMargin: '0px',    // Use actual viewport
  threshold: 0.5        // Count as "viewed" at 50%
}
```

#### Pattern 3: Scroll Animations (Wait until well inside)
```javascript
{
  rootMargin: '-100px', // Wait until inside viewport
  threshold: 0.3        // Trigger at 30% visible
}
```

#### Pattern 4: Infinite Scroll (Load before bottom)
```javascript
{
  rootMargin: '0px 0px 400px 0px',  // 400px before bottom
  threshold: 0                       // Any visibility
}
```

---

### The Decision Tree

```
How to choose values?

1. When to start checking?
   └─ Before visible → rootMargin: positive ('200px')
   └─ When visible → rootMargin: '0px'
   └─ After visible → rootMargin: negative ('-50px')

2. How much needs to be visible?
   └─ Any pixel → threshold: 0
   └─ Half visible → threshold: 0.5
   └─ Fully visible → threshold: 1.0
```

---

## 📊 Part 2: threshold Array

### What is a threshold Array?

Instead of a single number, you can provide an **array of thresholds**. The callback fires **every time** the element crosses one of these percentages.

---

### Single threshold vs Array

**Single threshold**:
```javascript
{
  threshold: 0.5  // Fires once when crossing 50%
}
```

**Array of thresholds**:
```javascript
{
  threshold: [0, 0.25, 0.5, 0.75, 1.0]
  // Fires 5 times:
  // - When any pixel becomes visible (0)
  // - When 25% becomes visible
  // - When 50% becomes visible
  // - When 75% becomes visible
  // - When 100% becomes visible
}
```

---

### How It Works

As you scroll, the callback triggers **every time you cross a threshold**:

```
Element scrolling into view:
┌────────────┐
│  Viewport  │
│            │   [Element] ← 0% visible
│            │   ↓
│            │   [Elem...  ← 25% visible → TRIGGER! 🔔
│            │   ↓
│            │   [Elemen   ← 50% visible → TRIGGER! 🔔
│      [Element  ↓
│            │   [Element] ← 75% visible → TRIGGER! 🔔
│      [Element] ↓
│            │            ← 100% visible → TRIGGER! 🔔
└────────────┘
```

---

### Example 1: Fade In at Different Stages

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ratio = entry.intersectionRatio
      const element = entry.target
      
      if (ratio >= 0.75) {
        element.style.opacity = '1'
        element.classList.add('fully-visible')
      } else if (ratio >= 0.5) {
        element.style.opacity = '0.7'
        element.classList.add('mostly-visible')
      } else if (ratio >= 0.25) {
        element.style.opacity = '0.4'
        element.classList.add('partially-visible')
      } else if (ratio > 0) {
        element.style.opacity = '0.2'
        element.classList.add('barely-visible')
      }
    })
  },
  {
    threshold: [0, 0.25, 0.5, 0.75, 1.0]
  }
)
```

---

### Example 2: Progress Bar Animation

```javascript
const progressObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const percentage = Math.round(entry.intersectionRatio * 100)
      
      // Update progress bar
      progressBar.style.width = `${percentage}%`
      progressBar.textContent = `${percentage}% visible`
      
      // Log each threshold crossing
      console.log(`Element is ${percentage}% visible`)
    })
  },
  {
    // Create 101 thresholds (0, 0.01, 0.02, ... 0.99, 1.0)
    threshold: Array.from({ length: 101 }, (_, i) => i / 100)
  }
)
```

---

### Example 3: Reading Progress Tracker

```javascript
const articleObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.intersectionRatio >= 0.9) {
        // User read 90% of article
        trackEvent('article_almost_complete')
      } else if (entry.intersectionRatio >= 0.5) {
        // User read half
        trackEvent('article_half_read')
      } else if (entry.intersectionRatio >= 0.25) {
        // User started reading
        trackEvent('article_started')
      }
    })
  },
  {
    threshold: [0.25, 0.5, 0.9]
  }
)

observer.observe(articleElement)
```

---

### Creating threshold Arrays

#### Method 1: Manual Array
```javascript
{
  threshold: [0, 0.25, 0.5, 0.75, 1.0]
}
```

#### Method 2: Generate with Array.from()
```javascript
{
  // Every 10%: [0, 0.1, 0.2, ... 0.9, 1.0]
  threshold: Array.from({ length: 11 }, (_, i) => i / 10)
}
```

#### Method 3: Very Smooth (Every 1%)
```javascript
{
  // Every 1%: [0, 0.01, 0.02, ... 0.99, 1.0]
  threshold: Array.from({ length: 101 }, (_, i) => i / 100)
}
```

---

### When to Use threshold Arrays

✅ **Use threshold array when:**
- You need different behavior at different visibility levels
- Creating smooth animations based on scroll position
- Tracking reading progress
- Building scroll-based progress indicators
- Need fine-grained control over visibility states

❌ **Don't use threshold array when:**
- You only care about one specific visibility point
- Performance is critical (more thresholds = more callback fires)
- Simple show/hide is enough

---

### Performance Considerations

```javascript
// ❌ BAD - 1000 thresholds = 1000 possible callback fires
{
  threshold: Array.from({ length: 1001 }, (_, i) => i / 1000)
}

// ✅ GOOD - 11 thresholds is usually enough
{
  threshold: Array.from({ length: 11 }, (_, i) => i / 10)
}

// ✅ BEST - Only what you need
{
  threshold: [0, 0.5, 1.0]  // Just 3 states
}
```

---

## 📈 Part 3: intersectionRatio

### What is intersectionRatio?

`intersectionRatio` is a **property on the entry object** that tells you exactly how much of the element is currently visible.

- **Type**: Number between `0.0` and `1.0`
- **Read-only**: You can't set it, only read it
- **Real-time**: Updates as element scrolls

---

### Values Explained

```javascript
entry.intersectionRatio === 0.0   // Element is 0% visible (completely hidden)
entry.intersectionRatio === 0.25  // Element is 25% visible
entry.intersectionRatio === 0.5   // Element is 50% visible
entry.intersectionRatio === 0.75  // Element is 75% visible
entry.intersectionRatio === 1.0   // Element is 100% visible (fully shown)
```

---

### Relationship with threshold

**threshold** = what triggers the callback
**intersectionRatio** = what's actually visible

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      console.log('Threshold:', 0.5)              // What we set
      console.log('Actual ratio:', entry.intersectionRatio)  // What's visible
      
      // These might be different!
      // threshold = 0.5 means "fire callback when crossing 50%"
      // intersectionRatio could be 0.51, 0.6, 0.75, etc.
    })
  },
  { threshold: 0.5 }
)
```

---

### Example 1: Dynamic Opacity Based on Visibility

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const element = entry.target
      const ratio = entry.intersectionRatio
      
      // Element opacity matches how visible it is
      element.style.opacity = ratio
      
      // 0% visible → opacity: 0 (invisible)
      // 50% visible → opacity: 0.5 (half transparent)
      // 100% visible → opacity: 1 (fully opaque)
    })
  },
  {
    threshold: Array.from({ length: 101 }, (_, i) => i / 100)
  }
)
```

---

### Example 2: Scale Element Based on Visibility

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ratio = entry.intersectionRatio
      
      // Scale from 0.5 to 1.0 based on visibility
      const scale = 0.5 + (ratio * 0.5)
      entry.target.style.transform = `scale(${scale})`
      
      // 0% visible → scale(0.5)
      // 50% visible → scale(0.75)
      // 100% visible → scale(1.0)
    })
  },
  {
    threshold: Array.from({ length: 21 }, (_, i) => i / 20)
  }
)
```

---

### Example 3: Blur Effect Based on Visibility

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ratio = entry.intersectionRatio
      
      // Blur reduces as element becomes more visible
      const blur = 20 - (ratio * 20)
      entry.target.style.filter = `blur(${blur}px)`
      
      // 0% visible → blur(20px) - very blurry
      // 50% visible → blur(10px) - medium blur
      // 100% visible → blur(0px) - sharp
    })
  },
  {
    threshold: Array.from({ length: 21 }, (_, i) => i / 20)
  }
)
```

---

### Example 4: Progress Indicator

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ratio = entry.intersectionRatio
      const percentage = Math.round(ratio * 100)
      
      // Update progress bar
      progressBar.style.width = `${percentage}%`
      progressText.textContent = `Reading: ${percentage}%`
      
      // Update color based on progress
      if (percentage >= 75) {
        progressBar.style.backgroundColor = '#10b981' // green
      } else if (percentage >= 50) {
        progressBar.style.backgroundColor = '#f59e0b' // orange
      } else if (percentage >= 25) {
        progressBar.style.backgroundColor = '#3b82f6' // blue
      } else {
        progressBar.style.backgroundColor = '#64748b' // gray
      }
    })
  },
  {
    threshold: Array.from({ length: 101 }, (_, i) => i / 100)
  }
)
```

---

### Example 5: Video Playback Control

```javascript
const videoObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const video = entry.target
      const ratio = entry.intersectionRatio
      
      if (ratio >= 0.5) {
        // More than 50% visible → play
        video.play()
      } else {
        // Less than 50% visible → pause
        video.pause()
      }
      
      // Also adjust volume based on visibility
      video.volume = ratio
      // 100% visible → volume: 1.0 (full)
      // 50% visible → volume: 0.5 (half)
      // 0% visible → volume: 0.0 (muted)
    })
  },
  {
    threshold: [0, 0.5, 1.0]
  }
)
```

---

### Example 6: Ad Visibility Tracking (IAB Standard)

The Interactive Advertising Bureau (IAB) defines a viewable impression as:
- At least 50% of ad pixels visible
- For at least 1 continuous second

```javascript
const adTimers = new Map()

const adObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ad = entry.target
      const ratio = entry.intersectionRatio
      
      if (ratio >= 0.5) {
        // Ad is 50%+ visible
        if (!adTimers.has(ad)) {
          // Start timer
          const timerId = setTimeout(() => {
            // After 1 second, count as viewable impression
            trackViewableImpression(ad.id)
          }, 1000)
          
          adTimers.set(ad, timerId)
        }
      } else {
        // Ad dropped below 50%
        if (adTimers.has(ad)) {
          // Clear timer (didn't meet 1 second requirement)
          clearTimeout(adTimers.get(ad))
          adTimers.delete(ad)
        }
      }
    })
  },
  {
    threshold: [0.5]  // IAB standard
  }
)
```

---

### Converting intersectionRatio to Useful Values

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const ratio = entry.intersectionRatio
    
    // As percentage
    const percentage = ratio * 100  // 0.75 → 75
    
    // As pixels (approximate)
    const height = entry.boundingClientRect.height
    const visiblePixels = height * ratio
    
    // As degrees (for rotation)
    const degrees = ratio * 360  // 0.5 → 180deg
    
    // As color intensity
    const alpha = ratio  // For rgba(255, 0, 0, alpha)
    
    console.log({
      percentage: `${percentage.toFixed(0)}%`,
      pixels: `${visiblePixels.toFixed(0)}px`,
      degrees: `${degrees.toFixed(0)}deg`,
      alpha: alpha.toFixed(2)
    })
  })
})
```

---

### intersectionRatio vs isIntersecting

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // isIntersecting: Boolean (true/false)
    console.log('Is intersecting?', entry.isIntersecting)
    // true if ratio > 0 and passes threshold
    
    // intersectionRatio: Number (0.0 to 1.0)
    console.log('How much?', entry.intersectionRatio)
    // Exact amount visible
  })
})

// When to use which?

// Use isIntersecting when:
if (entry.isIntersecting) {
  // Simple yes/no check
  loadImage()
}

// Use intersectionRatio when:
const opacity = entry.intersectionRatio
element.style.opacity = opacity
// Need exact visibility amount
```

---

## 🎯 Putting It All Together

### Complete Example: Sophisticated Image Lazy Loader

```javascript
const imageObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const img = entry.target
      const ratio = entry.intersectionRatio
      
      // Load when any part enters (with early margin)
      if (entry.isIntersecting && !img.dataset.loaded) {
        // Start loading
        const fullImg = new Image()
        fullImg.onload = () => {
          img.src = fullImg.src
          img.dataset.loaded = 'true'
        }
        fullImg.src = img.dataset.src
      }
      
      // Fade in based on visibility
      img.style.opacity = ratio
      
      // Add blur effect
      img.style.filter = `blur(${(1 - ratio) * 10}px)`
      
      // Scale slightly
      img.style.transform = `scale(${0.95 + ratio * 0.05})`
    })
  },
  {
    rootMargin: '100px',  // Start loading 100px early
    threshold: Array.from({ length: 21 }, (_, i) => i / 20)  // Smooth animation
  }
)

document.querySelectorAll('img[data-src]').forEach(img => {
  imageObserver.observe(img)
})
```

---

## 📝 Quick Reference

### rootMargin
- **Purpose**: Adjust when to check (expand/shrink viewport)
- **Format**: `"10px 20px 30px 40px"` (CSS margin syntax)
- **Positive**: Triggers before entering viewport
- **Negative**: Triggers after entering viewport

### threshold
- **Purpose**: How much must be visible to trigger
- **Type**: Number (0-1) or array of numbers
- **Single**: `0.5` → fires once at 50%
- **Array**: `[0, 0.5, 1]` → fires at 0%, 50%, 100%

### intersectionRatio
- **Purpose**: Exact visibility amount
- **Type**: Number (0.0 to 1.0)
- **Read-only**: Can't be set, only read
- **Use for**: Dynamic effects, progress tracking

## 🔗 MDN Resources

- [Intersection Observer API Overview](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [IntersectionObserver Interface](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver)
- [Timing Element Visibility Example](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API/Timing_element_visibility)