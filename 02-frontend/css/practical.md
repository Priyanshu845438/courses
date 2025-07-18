
# 🎨 Intermediate CSS

## Table of Contents
- [Advanced Selectors](#advanced-selectors)
- [Flexbox Layout](#flexbox-layout)
- [CSS Grid](#css-grid)
- [Responsive Design](#responsive-design)
- [Transitions and Animations](#transitions-and-animations)
- [Pseudo-classes and Pseudo-elements](#pseudo-classes-and-pseudo-elements)
- [CSS Variables](#css-variables)
- [Advanced Positioning](#advanced-positioning)
- [Performance Optimization](#performance-optimization)

---

## 🎯 Advanced Selectors

### Attribute Selectors

```css
/* Exact attribute value */
input[type="text"] {
    border: 2px solid blue;
}

/* Attribute exists */
a[href] {
    color: green;
}

/* Attribute starts with */
a[href^="https"] {
    color: secure-green;
}

/* Attribute ends with */
a[href$=".pdf"] {
    color: red;
}

/* Attribute contains */
a[href*="github"] {
    color: black;
}

/* Attribute contains word */
div[class~="highlight"] {
    background-color: yellow;
}

/* Attribute starts with (language codes) */
div[lang|="en"] {
    font-family: "English Font";
}
```

### Combinators

```css
/* Descendant combinator */
article p {
    line-height: 1.6;
}

/* Child combinator */
nav > ul {
    list-style: none;
}

/* Adjacent sibling combinator */
h1 + p {
    font-size: 1.2em;
    margin-top: 0;
}

/* General sibling combinator */
h1 ~ p {
    color: #666;
}
```

### Advanced Pseudo-selectors

```css
/* Structural pseudo-classes */
:nth-child(odd) {
    background-color: #f5f5f5;
}

:nth-child(even) {
    background-color: white;
}

:nth-child(3n) {
    border-left: 3px solid blue;
}

:nth-of-type(2) {
    font-weight: bold;
}

:first-child {
    margin-top: 0;
}

:last-child {
    margin-bottom: 0;
}

:only-child {
    text-align: center;
}

/* Negation pseudo-class */
:not(.special) {
    color: black;
}

:not(p):not(div) {
    font-style: italic;
}
```

---

## 🔧 Flexbox Layout

### Flex Container Properties

```css
.container {
    display: flex;
    
    /* Direction */
    flex-direction: row;        /* row, row-reverse, column, column-reverse */
    
    /* Wrap */
    flex-wrap: nowrap;          /* nowrap, wrap, wrap-reverse */
    
    /* Shorthand */
    flex-flow: row wrap;        /* flex-direction flex-wrap */
    
    /* Justify content (main axis) */
    justify-content: flex-start; /* flex-start, flex-end, center, space-between, space-around, space-evenly */
    
    /* Align items (cross axis) */
    align-items: stretch;       /* stretch, flex-start, flex-end, center, baseline */
    
    /* Align content (multiple lines) */
    align-content: flex-start;  /* flex-start, flex-end, center, space-between, space-around, stretch */
    
    /* Gap between items */
    gap: 20px;                  /* row-gap column-gap */
}
```

### Flex Item Properties

```css
.item {
    /* Flex grow */
    flex-grow: 1;               /* How much space to take */
    
    /* Flex shrink */
    flex-shrink: 1;             /* How much to shrink */
    
    /* Flex basis */
    flex-basis: 200px;          /* Initial size */
    
    /* Shorthand */
    flex: 1 1 200px;            /* flex-grow flex-shrink flex-basis */
    
    /* Align self */
    align-self: center;         /* Override align-items */
    
    /* Order */
    order: 2;                   /* Change visual order */
}
```

### Flexbox Examples

```html
<div class="flex-container">
    <div class="flex-item">1</div>
    <div class="flex-item">2</div>
    <div class="flex-item">3</div>
</div>
```

```css
/* Basic flex layout */
.flex-container {
    display: flex;
    gap: 20px;
    padding: 20px;
    background-color: #f5f5f5;
}

.flex-item {
    flex: 1;
    padding: 20px;
    background-color: #007bff;
    color: white;
    text-align: center;
    border-radius: 5px;
}

/* Navigation bar */
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    background-color: #333;
}

.navbar .logo {
    font-size: 1.5rem;
    font-weight: bold;
    color: white;
}

.navbar .nav-links {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.navbar .nav-links a {
    color: white;
    text-decoration: none;
}

/* Card layout */
.card-container {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    padding: 20px;
}

.card {
    flex: 1 1 300px;
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    padding: 20px;
}

/* Centering content */
.center-content {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
}
```

---

## 🎯 CSS Grid

### Grid Container Properties

```css
.grid-container {
    display: grid;
    
    /* Define columns */
    grid-template-columns: 200px 1fr 200px;
    grid-template-columns: repeat(3, 1fr);
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    
    /* Define rows */
    grid-template-rows: auto 1fr auto;
    grid-template-rows: repeat(3, 200px);
    
    /* Grid gaps */
    gap: 20px;
    grid-gap: 20px;              /* Legacy */
    grid-row-gap: 20px;          /* Row gap only */
    grid-column-gap: 30px;       /* Column gap only */
    
    /* Grid template areas */
    grid-template-areas: 
        "header header header"
        "sidebar main main"
        "footer footer footer";
    
    /* Justify items */
    justify-items: stretch;      /* stretch, start, end, center */
    
    /* Align items */
    align-items: stretch;        /* stretch, start, end, center */
    
    /* Justify content */
    justify-content: stretch;    /* stretch, start, end, center, space-between, space-around, space-evenly */
    
    /* Align content */
    align-content: stretch;      /* stretch, start, end, center, space-between, space-around, space-evenly */
}
```

### Grid Item Properties

```css
.grid-item {
    /* Grid positioning */
    grid-column: 1 / 3;          /* Start at column 1, end at column 3 */
    grid-row: 2 / 4;             /* Start at row 2, end at row 4 */
    
    /* Shorthand */
    grid-area: 2 / 1 / 4 / 3;    /* row-start / column-start / row-end / column-end */
    
    /* Named grid areas */
    grid-area: header;
    
    /* Spanning */
    grid-column: span 2;         /* Span 2 columns */
    grid-row: span 3;            /* Span 3 rows */
    
    /* Self alignment */
    justify-self: center;        /* Override justify-items */
    align-self: end;             /* Override align-items */
}
```

### Grid Examples

```html
<div class="grid-layout">
    <header class="header">Header</header>
    <nav class="sidebar">Sidebar</nav>
    <main class="main">Main Content</main>
    <footer class="footer">Footer</footer>
</div>
```

```css
/* Basic grid layout */
.grid-layout {
    display: grid;
    grid-template-columns: 250px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas: 
        "header header"
        "sidebar main"
        "footer footer";
    gap: 20px;
    min-height: 100vh;
}

.header {
    grid-area: header;
    background-color: #333;
    color: white;
    padding: 1rem;
}

.sidebar {
    grid-area: sidebar;
    background-color: #f5f5f5;
    padding: 1rem;
}

.main {
    grid-area: main;
    background-color: white;
    padding: 1rem;
}

.footer {
    grid-area: footer;
    background-color: #333;
    color: white;
    padding: 1rem;
    text-align: center;
}

/* Responsive card grid */
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    padding: 20px;
}

.card {
    background-color: white;
    border-radius: 10px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    padding: 20px;
    transition: transform 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
}

/* Photo gallery */
.photo-gallery {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    grid-auto-rows: 200px;
    gap: 10px;
}

.photo {
    background-size: cover;
    background-position: center;
    border-radius: 5px;
}

.photo.large {
    grid-column: span 2;
    grid-row: span 2;
}
```

---

## 📱 Responsive Design

### Media Queries

```css
/* Basic media query */
@media (max-width: 768px) {
    .container {
        padding: 10px;
    }
    
    .grid-layout {
        grid-template-columns: 1fr;
        grid-template-areas: 
            "header"
            "main"
            "sidebar"
            "footer";
    }
}

/* Multiple conditions */
@media (min-width: 768px) and (max-width: 1024px) {
    .container {
        max-width: 750px;
    }
}

/* Orientation */
@media (orientation: landscape) {
    .hero {
        height: 60vh;
    }
}

/* High DPI screens */
@media (min-resolution: 192dpi) {
    .logo {
        background-image: url('logo@2x.png');
    }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
    body {
        background-color: #1a1a1a;
        color: #ffffff;
    }
}
```

### Responsive Breakpoints

```css
/* Mobile First Approach */
.container {
    width: 100%;
    padding: 0 15px;
}

/* Small devices (landscape phones, 576px and up) */
@media (min-width: 576px) {
    .container {
        max-width: 540px;
    }
}

/* Medium devices (tablets, 768px and up) */
@media (min-width: 768px) {
    .container {
        max-width: 720px;
    }
}

/* Large devices (desktops, 992px and up) */
@media (min-width: 992px) {
    .container {
        max-width: 960px;
    }
}

/* Extra large devices (large desktops, 1200px and up) */
@media (min-width: 1200px) {
    .container {
        max-width: 1140px;
    }
}
```

### Responsive Units

```css
/* Viewport units */
.hero {
    height: 100vh;        /* Full viewport height */
    width: 100vw;         /* Full viewport width */
    font-size: 4vw;       /* 4% of viewport width */
}

/* Relative units */
.text {
    font-size: 1.2rem;    /* Relative to root font size */
    line-height: 1.5em;   /* Relative to element font size */
    margin: 2ch;          /* Relative to character width */
}

/* Flexible sizing */
.flexible {
    width: clamp(300px, 50%, 800px);  /* Min, preferred, max */
    font-size: clamp(1rem, 2.5vw, 2rem);
}
```

---

## ✨ Transitions and Animations

### CSS Transitions

```css
/* Basic transition */
.button {
    background-color: #007bff;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    transition: background-color 0.3s ease;
}

.button:hover {
    background-color: #0056b3;
}

/* Multiple properties */
.card {
    transform: translateY(0);
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

/* All properties */
.element {
    transition: all 0.3s ease;
}

/* Different timing functions */
.ease-in {
    transition: transform 0.3s ease-in;
}

.ease-out {
    transition: transform 0.3s ease-out;
}

.ease-in-out {
    transition: transform 0.3s ease-in-out;
}

.cubic-bezier {
    transition: transform 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
```

### CSS Animations

```css
/* Define keyframes */
@keyframes slideIn {
    from {
        transform: translateX(-100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

@keyframes bounce {
    0%, 20%, 53%, 80%, 100% {
        transform: translateY(0);
    }
    40%, 43% {
        transform: translateY(-20px);
    }
    70% {
        transform: translateY(-10px);
    }
    90% {
        transform: translateY(-4px);
    }
}

@keyframes rotate {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

/* Apply animations */
.slide-in {
    animation: slideIn 0.5s ease-out;
}

.bounce-animation {
    animation: bounce 1s infinite;
}

.rotate-animation {
    animation: rotate 2s linear infinite;
}

/* Complex animation */
.complex-animation {
    animation: 
        slideIn 0.5s ease-out,
        bounce 1s ease-in-out 0.5s infinite;
}

/* Animation properties */
.animated-element {
    animation-name: slideIn;
    animation-duration: 0.5s;
    animation-timing-function: ease-out;
    animation-delay: 0.2s;
    animation-iteration-count: 1;
    animation-direction: normal;
    animation-fill-mode: both;
    animation-play-state: running;
}
```

### Transform Properties

```css
/* 2D Transforms */
.transform-2d {
    transform: translate(50px, 100px);
    transform: translateX(50px);
    transform: translateY(100px);
    transform: scale(1.5);
    transform: scaleX(2);
    transform: scaleY(0.5);
    transform: rotate(45deg);
    transform: skew(20deg, 10deg);
    transform: skewX(20deg);
    transform: skewY(10deg);
}

/* 3D Transforms */
.transform-3d {
    transform: perspective(800px) rotateX(45deg);
    transform: perspective(800px) rotateY(45deg);
    transform: perspective(800px) rotateZ(45deg);
    transform: perspective(800px) translateZ(100px);
    transform: perspective(800px) scale3d(1.2, 1.2, 1.2);
}

/* Transform origin */
.transform-origin {
    transform-origin: top left;
    transform-origin: 50% 50%;
    transform-origin: 20px 30px;
}

/* Multiple transforms */
.multiple-transforms {
    transform: translateX(50px) rotate(45deg) scale(1.2);
}
```

---

## 🎨 Pseudo-classes and Pseudo-elements

### Interactive Pseudo-classes

```css
/* Link states */
a:link {
    color: blue;
}

a:visited {
    color: purple;
}

a:hover {
    color: red;
    text-decoration: underline;
}

a:active {
    color: darkred;
}

/* Form states */
input:focus {
    border-color: #007bff;
    box-shadow: 0 0 5px rgba(0,123,255,0.5);
}

input:valid {
    border-color: green;
}

input:invalid {
    border-color: red;
}

input:disabled {
    background-color: #f5f5f5;
    cursor: not-allowed;
}

/* UI states */
.checkbox:checked + label {
    font-weight: bold;
}

.element:target {
    background-color: yellow;
}

.item:nth-child(even) {
    background-color: #f5f5f5;
}
```

### Pseudo-elements

```css
/* ::before and ::after */
.quote::before {
    content: """;
    font-size: 2em;
    color: #007bff;
}

.quote::after {
    content: """;
    font-size: 2em;
    color: #007bff;
}

/* Clear floats */
.clearfix::after {
    content: "";
    display: table;
    clear: both;
}

/* Decorative elements */
.section-title::before {
    content: "";
    display: inline-block;
    width: 30px;
    height: 3px;
    background-color: #007bff;
    margin-right: 10px;
    vertical-align: middle;
}

/* Tooltips */
.tooltip {
    position: relative;
    cursor: pointer;
}

.tooltip::after {
    content: attr(data-tooltip);
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    background-color: #333;
    color: white;
    padding: 5px 10px;
    border-radius: 4px;
    font-size: 12px;
    white-space: nowrap;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.3s;
}

.tooltip:hover::after {
    opacity: 1;
}

/* First line and letter */
.article::first-line {
    font-weight: bold;
    color: #007bff;
}

.article::first-letter {
    font-size: 3em;
    float: left;
    line-height: 1;
    margin-right: 5px;
}

/* Selection */
::selection {
    background-color: #007bff;
    color: white;
}

/* Placeholder */
input::placeholder {
    color: #999;
    font-style: italic;
}
```

---

## 🎛️ CSS Variables

### Defining Variables

```css
:root {
    /* Colors */
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #17a2b8;
    --light-color: #f8f9fa;
    --dark-color: #343a40;
    
    /* Fonts */
    --font-family-base: 'Arial', sans-serif;
    --font-family-heading: 'Georgia', serif;
    --font-size-base: 16px;
    --font-size-large: 1.25rem;
    --font-size-small: 0.875rem;
    
    /* Spacing */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 3rem;
    
    /* Breakpoints */
    --breakpoint-sm: 576px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 992px;
    --breakpoint-xl: 1200px;
    
    /* Shadows */
    --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    --shadow-md: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
    --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
    
    /* Transitions */
    --transition-fast: 0.15s ease-in-out;
    --transition-base: 0.3s ease-in-out;
    --transition-slow: 0.6s ease-in-out;
}
```

### Using Variables

```css
/* Basic usage */
.button {
    background-color: var(--primary-color);
    color: white;
    padding: var(--spacing-sm) var(--spacing-md);
    border: none;
    border-radius: 4px;
    font-family: var(--font-family-base);
    font-size: var(--font-size-base);
    transition: all var(--transition-base);
}

.button:hover {
    background-color: var(--primary-color);
    filter: brightness(0.9);
}

/* Fallback values */
.element {
    color: var(--text-color, #333);
    background-color: var(--bg-color, white);
}

/* Calculations with variables */
.container {
    width: calc(100% - var(--spacing-xl) * 2);
    margin: 0 auto;
    padding: var(--spacing-lg);
}

/* Theme switching */
.theme-light {
    --bg-color: white;
    --text-color: #333;
    --border-color: #e0e0e0;
}

.theme-dark {
    --bg-color: #1a1a1a;
    --text-color: #ffffff;
    --border-color: #333;
}

.themed-element {
    background-color: var(--bg-color);
    color: var(--text-color);
    border: 1px solid var(--border-color);
}
```

### Dynamic Variables with JavaScript

```html
<div class="color-picker">
    <label for="primary-color">Primary Color:</label>
    <input type="color" id="primary-color" value="#007bff">
    
    <label for="font-size">Font Size:</label>
    <input type="range" id="font-size" min="12" max="24" value="16">
</div>
```

```css
.dynamic-element {
    background-color: var(--dynamic-primary, #007bff);
    font-size: var(--dynamic-font-size, 16px);
    padding: var(--spacing-md);
    border-radius: 4px;
    color: white;
}
```

```javascript
// JavaScript to update CSS variables
document.getElementById('primary-color').addEventListener('input', function(e) {
    document.documentElement.style.setProperty('--dynamic-primary', e.target.value);
});

document.getElementById('font-size').addEventListener('input', function(e) {
    document.documentElement.style.setProperty('--dynamic-font-size', e.target.value + 'px');
});
```

---

## 🎯 Advanced Positioning

### Sticky Positioning

```css
.sticky-header {
    position: sticky;
    top: 0;
    background-color: white;
    z-index: 1000;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.sticky-sidebar {
    position: sticky;
    top: 20px;
    height: fit-content;
}

/* Sticky navigation */
.sticky-nav {
    position: sticky;
    top: 0;
    background-color: #333;
    padding: 1rem 0;
}

.sticky-nav ul {
    display: flex;
    justify-content: center;
    list-style: none;
    margin: 0;
    padding: 0;
}

.sticky-nav a {
    color: white;
    text-decoration: none;
    padding: 0.5rem 1rem;
    margin: 0 0.5rem;
    border-radius: 4px;
    transition: background-color 0.3s;
}

.sticky-nav a:hover {
    background-color: rgba(255,255,255,0.1);
}
```

### Z-Index Management

```css
/* Z-index layering system */
.z-index-base {
    z-index: 1;
}

.z-index-dropdown {
    z-index: 1000;
}

.z-index-sticky {
    z-index: 1020;
}

.z-index-fixed {
    z-index: 1030;
}

.z-index-modal-backdrop {
    z-index: 1040;
}

.z-index-modal {
    z-index: 1050;
}

.z-index-popover {
    z-index: 1060;
}

.z-index-tooltip {
    z-index: 1070;
}
```

---

## ⚡ Performance Optimization

### Efficient Selectors

```css
/* Efficient - ID selector */
#header {
    background-color: blue;
}

/* Efficient - Class selector */
.nav-item {
    color: white;
}

/* Less efficient - Universal selector */
* {
    box-sizing: border-box;
}

/* Avoid overly specific selectors */
/* Bad */
div.container div.content div.article p.text {
    color: red;
}

/* Good */
.article-text {
    color: red;
}
```

### CSS Optimization

```css
/* Minimize repaints and reflows */
.optimized-animation {
    /* Use transform instead of changing position */
    transform: translateX(100px);
    
    /* Use opacity instead of visibility */
    opacity: 0;
    
    /* Use will-change for performance */
    will-change: transform, opacity;
}

/* Efficient transitions */
.efficient-transition {
    transition: transform 0.3s ease-out;
}

/* Avoid animating expensive properties */
.avoid {
    /* Expensive - causes reflow */
    transition: width 0.3s ease;
    
    /* Expensive - causes repaint */
    transition: background-color 0.3s ease;
}

.prefer {
    /* Cheap - only compositing */
    transition: transform 0.3s ease;
    transition: opacity 0.3s ease;
}
```

### Critical CSS

```css
/* Critical CSS - above the fold */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
}

.header {
    background-color: #333;
    color: white;
    padding: 1rem;
}

.hero {
    height: 60vh;
    background-color: #f5f5f5;
    display: flex;
    align-items: center;
    justify-content: center;
}

/* Non-critical CSS - below the fold */
.footer {
    background-color: #333;
    color: white;
    padding: 2rem;
    text-align: center;
}

.contact-form {
    max-width: 500px;
    margin: 2rem auto;
}
```

## 📚 Complete Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Intermediate CSS Example</title>
    <style>
        :root {
            --primary-color: #007bff;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --spacing-sm: 0.5rem;
            --spacing-md: 1rem;
            --spacing-lg: 1.5rem;
            --border-radius: 8px;
            --transition: 0.3s ease;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            line-height: 1.6;
            color: #333;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 var(--spacing-md);
        }

        /* Sticky header */
        .header {
            position: sticky;
            top: 0;
            background-color: white;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
        }

        .navbar {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: var(--spacing-md) 0;
        }

        .logo {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--primary-color);
        }

        .nav-links {
            display: flex;
            list-style: none;
            gap: var(--spacing-lg);
        }

        .nav-links a {
            text-decoration: none;
            color: #333;
            font-weight: 500;
            transition: color var(--transition);
        }

        .nav-links a:hover {
            color: var(--primary-color);
        }

        /* Hero section */
        .hero {
            height: 60vh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
            color: white;
            text-align: center;
        }

        .hero h1 {
            font-size: 3rem;
            margin-bottom: var(--spacing-md);
            animation: fadeInUp 1s ease-out;
        }

        .hero p {
            font-size: 1.2rem;
            margin-bottom: var(--spacing-lg);
            animation: fadeInUp 1s ease-out 0.2s both;
        }

        .cta-button {
            display: inline-block;
            padding: var(--spacing-md) var(--spacing-lg);
            background-color: white;
            color: var(--primary-color);
            text-decoration: none;
            border-radius: var(--border-radius);
            font-weight: bold;
            transition: all var(--transition);
            animation: fadeInUp 1s ease-out 0.4s both;
        }

        .cta-button:hover {
            background-color: var(--primary-color);
            color: white;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        /* Grid section */
        .features {
            padding: 4rem 0;
            background-color: #f8f9fa;
        }

        .features h2 {
            text-align: center;
            margin-bottom: 3rem;
            font-size: 2.5rem;
            color: #333;
        }

        .features-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 2rem;
        }

        .feature-card {
            background-color: white;
            padding: 2rem;
            border-radius: var(--border-radius);
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            text-align: center;
            transition: transform var(--transition);
        }

        .feature-card:hover {
            transform: translateY(-5px);
        }

        .feature-icon {
            width: 60px;
            height: 60px;
            background-color: var(--primary-color);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 auto 1rem;
            font-size: 1.5rem;
            color: white;
        }

        .feature-card h3 {
            margin-bottom: 1rem;
            color: #333;
        }

        .feature-card p {
            color: #666;
        }

        /* Responsive design */
        @media (max-width: 768px) {
            .navbar {
                flex-direction: column;
                gap: var(--spacing-md);
            }

            .nav-links {
                flex-direction: column;
                text-align: center;
            }

            .hero h1 {
                font-size: 2rem;
            }

            .hero p {
                font-size: 1rem;
            }

            .features-grid {
                grid-template-columns: 1fr;
            }
        }

        /* Animations */
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

        /* Utility classes */
        .text-center {
            text-align: center;
        }

        .mb-2 {
            margin-bottom: var(--spacing-md);
        }

        .mt-2 {
            margin-top: var(--spacing-md);
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="container">
            <nav class="navbar">
                <div class="logo">MyBrand</div>
                <ul class="nav-links">
                    <li><a href="#home">Home</a></li>
                    <li><a href="#about">About</a></li>
                    <li><a href="#features">Features</a></li>
                    <li><a href="#contact">Contact</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <section class="hero">
        <div class="container">
            <h1>Welcome to the Future</h1>
            <p>Discover amazing features with our innovative platform</p>
            <a href="#features" class="cta-button">Get Started</a>
        </div>
    </section>

    <section class="features" id="features">
        <div class="container">
            <h2>Our Features</h2>
            <div class="features-grid">
                <div class="feature-card">
                    <div class="feature-icon">⚡</div>
                    <h3>Lightning Fast</h3>
                    <p>Experience blazing fast performance with our optimized platform.</p>
                </div>
                <div class="feature-card">
                    <div class="feature-icon">🔒</div>
                    <h3>Secure</h3>
                    <p>Your data is protected with enterprise-grade security measures.</p>
                </div>
                <div class="feature-card">
                    <div class="feature-icon">📱</div>
                    <h3>Responsive</h3>
                    <p>Works perfectly on all devices, from mobile to desktop.</p>
                </div>
                <div class="feature-card">
                    <div class="feature-icon">🎨</div>
                    <h3>Customizable</h3>
                    <p>Tailor the experience to match your brand and preferences.</p>
                </div>
                <div class="feature-card">
                    <div class="feature-icon">📊</div>
                    <h3>Analytics</h3>
                    <p>Get detailed insights with comprehensive analytics dashboard.</p>
                </div>
                <div class="feature-card">
                    <div class="feature-icon">🌟</div>
                    <h3>Premium Support</h3>
                    <p>24/7 premium support from our expert team.</p>
                </div>
            </div>
        </div>
    </section>
</body>
</html>
```

This comprehensive intermediate CSS guide covers the essential topics needed to create modern, responsive, and performant web layouts. Practice these concepts to build more sophisticated web interfaces!
# 🎨 CSS Practical Exercises

## 🎯 Learning Objectives
By completing these exercises, you will:
- Apply CSS selectors and properties
- Build responsive layouts with Flexbox and Grid
- Create interactive components with transitions
- Implement modern design patterns
- Practice CSS debugging and optimization

---

## 🛠️ Exercise 1: CSS Selectors and Styling

### Task: Style a Blog Post Layout

Create a blog post layout with proper typography and spacing.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Selectors Exercise</title>
    <link rel="stylesheet" href="blog-style.css">
</head>
<body>
    <article class="blog-post">
        <header class="post-header">
            <h1 class="post-title">Understanding CSS Selectors</h1>
            <div class="post-meta">
                <span class="author">By Jane Doe</span>
                <time class="date" datetime="2024-01-15">January 15, 2024</time>
                <div class="tags">
                    <span class="tag">CSS</span>
                    <span class="tag">Web Development</span>
                    <span class="tag featured">Tutorial</span>
                </div>
            </div>
        </header>
        
        <div class="post-content">
            <p class="lead">CSS selectors are the foundation of styling web pages. They allow you to target specific elements and apply styles.</p>
            
            <h2>Types of Selectors</h2>
            <ul class="selector-list">
                <li>Element selectors</li>
                <li>Class selectors</li>
                <li>ID selectors</li>
                <li>Attribute selectors</li>
            </ul>
            
            <blockquote class="quote">
                "CSS is like magic, but you're the magician." - Unknown
            </blockquote>
            
            <p>Remember to always use <code>semantic HTML</code> elements and meaningful class names.</p>
        </div>
        
        <footer class="post-footer">
            <button class="btn btn-primary">Like</button>
            <button class="btn btn-secondary">Share</button>
        </footer>
    </article>
</body>
</html>
```

**Your CSS Task (`blog-style.css`):**

```css
/* Reset and base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Georgia', serif;
    line-height: 1.6;
    color: #333;
    background-color: #fafafa;
    padding: 20px;
}

/* Blog post container */
.blog-post {
    max-width: 800px;
    margin: 0 auto;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    overflow: hidden;
}

/* Header styles */
.post-header {
    padding: 40px 40px 20px;
    border-bottom: 1px solid #eee;
}

.post-title {
    font-size: 2.5rem;
    font-weight: 700;
    margin-bottom: 20px;
    color: #2c3e50;
}

.post-meta {
    display: flex;
    flex-wrap: wrap;
    gap: 20px;
    align-items: center;
    font-size: 0.9rem;
    color: #666;
}

.author {
    font-weight: 600;
}

.date {
    color: #888;
}

.tags {
    display: flex;
    gap: 8px;
}

.tag {
    background: #e3f2fd;
    color: #1976d2;
    padding: 4px 12px;
    border-radius: 16px;
    font-size: 0.8rem;
    font-weight: 500;
}

.tag.featured {
    background: #f3e5f5;
    color: #7b1fa2;
}

/* Content styles */
.post-content {
    padding: 40px;
}

.lead {
    font-size: 1.2rem;
    font-weight: 300;
    color: #555;
    margin-bottom: 30px;
    line-height: 1.8;
}

h2 {
    font-size: 1.8rem;
    margin: 30px 0 20px;
    color: #2c3e50;
}

.selector-list {
    margin: 20px 0;
    padding-left: 30px;
}

.selector-list li {
    margin-bottom: 8px;
    position: relative;
}

.selector-list li::marker {
    color: #1976d2;
}

.quote {
    background: #f8f9fa;
    border-left: 4px solid #1976d2;
    padding: 20px;
    margin: 30px 0;
    font-style: italic;
    font-size: 1.1rem;
    color: #555;
}

code {
    background: #f1f3f4;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'Monaco', monospace;
    color: #d63384;
}

/* Footer styles */
.post-footer {
    padding: 20px 40px;
    background: #f8f9fa;
    display: flex;
    gap: 15px;
}

.btn {
    padding: 10px 20px;
    border: none;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
}

.btn-primary {
    background: #1976d2;
    color: white;
}

.btn-primary:hover {
    background: #1565c0;
    transform: translateY(-2px);
}

.btn-secondary {
    background: transparent;
    color: #1976d2;
    border: 2px solid #1976d2;
}

.btn-secondary:hover {
    background: #1976d2;
    color: white;
}

/* Responsive design */
@media (max-width: 768px) {
    .blog-post {
        margin: 0;
        border-radius: 0;
    }
    
    .post-header,
    .post-content,
    .post-footer {
        padding: 20px;
    }
    
    .post-title {
        font-size: 2rem;
    }
    
    .post-meta {
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
    }
}
```

---

## 🔧 Exercise 2: Flexbox Layout

### Task: Create a Responsive Navigation Bar

Build a responsive navigation that collapses on mobile devices.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox Navigation</title>
    <link rel="stylesheet" href="nav-style.css">
</head>
<body>
    <nav class="navbar">
        <div class="nav-container">
            <div class="nav-brand">
                <img src="https://via.placeholder.com/40x40/1976d2/ffffff?text=L" alt="Logo" class="logo">
                <span class="brand-text">WebDev</span>
            </div>
            
            <button class="nav-toggle" aria-label="Toggle navigation">
                <span></span>
                <span></span>
                <span></span>
            </button>
            
            <ul class="nav-menu">
                <li class="nav-item">
                    <a href="#" class="nav-link active">Home</a>
                </li>
                <li class="nav-item">
                    <a href="#" class="nav-link">About</a>
                </li>
                <li class="nav-item">
                    <a href="#" class="nav-link">Services</a>
                </li>
                <li class="nav-item">
                    <a href="#" class="nav-link">Portfolio</a>
                </li>
                <li class="nav-item">
                    <a href="#" class="nav-link">Contact</a>
                </li>
            </ul>
            
            <div class="nav-actions">
                <button class="btn btn-outline">Login</button>
                <button class="btn btn-primary">Sign Up</button>
            </div>
        </div>
    </nav>
    
    <main class="content">
        <h1>Flexbox Navigation Demo</h1>
        <p>Resize the window to see the responsive behavior.</p>
    </main>
</body>
</html>
```

**Your CSS Task (`nav-style.css`):**

```css
/* Reset and base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    line-height: 1.6;
}

/* Navigation styles */
.navbar {
    background: #fff;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    position: sticky;
    top: 0;
    z-index: 1000;
}

.nav-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    min-height: 70px;
}

/* Brand section */
.nav-brand {
    display: flex;
    align-items: center;
    gap: 12px;
}

.logo {
    width: 40px;
    height: 40px;
    border-radius: 8px;
}

.brand-text {
    font-size: 1.5rem;
    font-weight: 700;
    color: #1976d2;
}

/* Navigation menu */
.nav-menu {
    display: flex;
    list-style: none;
    gap: 0;
}

.nav-item {
    position: relative;
}

.nav-link {
    display: block;
    padding: 25px 20px;
    text-decoration: none;
    color: #555;
    font-weight: 500;
    transition: color 0.3s ease;
}

.nav-link:hover,
.nav-link.active {
    color: #1976d2;
}

.nav-link.active::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 60%;
    height: 3px;
    background: #1976d2;
}

/* Navigation actions */
.nav-actions {
    display: flex;
    gap: 12px;
}

.btn {
    padding: 8px 16px;
    border-radius: 6px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    border: 2px solid transparent;
}

.btn-outline {
    background: transparent;
    color: #1976d2;
    border-color: #1976d2;
}

.btn-outline:hover {
    background: #1976d2;
    color: white;
}

.btn-primary {
    background: #1976d2;
    color: white;
    border-color: #1976d2;
}

.btn-primary:hover {
    background: #1565c0;
    border-color: #1565c0;
}

/* Mobile toggle button */
.nav-toggle {
    display: none;
    flex-direction: column;
    gap: 4px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 8px;
}

.nav-toggle span {
    width: 25px;
    height: 3px;
    background: #333;
    transition: all 0.3s ease;
}

/* Content area */
.content {
    padding: 60px 20px;
    max-width: 1200px;
    margin: 0 auto;
    text-align: center;
}

.content h1 {
    font-size: 2.5rem;
    margin-bottom: 20px;
    color: #333;
}

.content p {
    font-size: 1.1rem;
    color: #666;
}

/* Mobile responsive */
@media (max-width: 768px) {
    .nav-container {
        flex-wrap: wrap;
    }
    
    .nav-toggle {
        display: flex;
        order: 2;
    }
    
    .nav-brand {
        order: 1;
    }
    
    .nav-menu {
        order: 3;
        width: 100%;
        flex-direction: column;
        background: #f8f9fa;
        max-height: 0;
        overflow: hidden;
        transition: max-height 0.3s ease;
    }
    
    .nav-menu.active {
        max-height: 300px;
    }
    
    .nav-link {
        padding: 15px 20px;
        border-bottom: 1px solid #eee;
    }
    
    .nav-link.active::after {
        display: none;
    }
    
    .nav-actions {
        order: 4;
        width: 100%;
        justify-content: center;
        padding: 20px 0;
        background: #f8f9fa;
        display: none;
    }
    
    .nav-actions.active {
        display: flex;
    }
    
    /* Hamburger animation */
    .nav-toggle.active span:nth-child(1) {
        transform: rotate(45deg) translate(5px, 5px);
    }
    
    .nav-toggle.active span:nth-child(2) {
        opacity: 0;
    }
    
    .nav-toggle.active span:nth-child(3) {
        transform: rotate(-45deg) translate(7px, -6px);
    }
}
```

**Add JavaScript for mobile toggle:**

```javascript
// Add this script tag before closing body tag
const navToggle = document.querySelector('.nav-toggle');
const navMenu = document.querySelector('.nav-menu');
const navActions = document.querySelector('.nav-actions');

navToggle.addEventListener('click', () => {
    navToggle.classList.toggle('active');
    navMenu.classList.toggle('active');
    navActions.classList.toggle('active');
});
```

---

## 🎮 Exercise 3: CSS Grid Layout

### Task: Create a Responsive Dashboard

Build a dashboard layout using CSS Grid that adapts to different screen sizes.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Grid Dashboard</title>
    <link rel="stylesheet" href="dashboard-style.css">
</head>
<body>
    <div class="dashboard">
        <header class="header">
            <h1>Analytics Dashboard</h1>
            <div class="user-info">
                <img src="https://via.placeholder.com/40x40/666/fff?text=U" alt="User" class="avatar">
                <span>John Doe</span>
            </div>
        </header>
        
        <nav class="sidebar">
            <ul class="nav-list">
                <li><a href="#" class="nav-item active">Dashboard</a></li>
                <li><a href="#" class="nav-item">Analytics</a></li>
                <li><a href="#" class="nav-item">Reports</a></li>
                <li><a href="#" class="nav-item">Settings</a></li>
            </ul>
        </nav>
        
        <main class="main-content">
            <div class="stats-grid">
                <div class="stat-card">
                    <h3>Total Users</h3>
                    <p class="stat-number">12,345</p>
                    <span class="stat-change positive">+5.2%</span>
                </div>
                
                <div class="stat-card">
                    <h3>Revenue</h3>
                    <p class="stat-number">$45,678</p>
                    <span class="stat-change positive">+12.1%</span>
                </div>
                
                <div class="stat-card">
                    <h3>Conversion Rate</h3>
                    <p class="stat-number">3.24%</p>
                    <span class="stat-change negative">-2.1%</span>
                </div>
                
                <div class="stat-card">
                    <h3>Bounce Rate</h3>
                    <p class="stat-number">42.8%</p>
                    <span class="stat-change positive">-1.3%</span>
                </div>
            </div>
            
            <div class="content-grid">
                <section class="chart-section">
                    <h2>Website Traffic</h2>
                    <div class="chart-placeholder">
                        <p>Chart would go here</p>
                    </div>
                </section>
                
                <section class="recent-activity">
                    <h2>Recent Activity</h2>
                    <ul class="activity-list">
                        <li>User signed up - 2 mins ago</li>
                        <li>New order received - 5 mins ago</li>
                        <li>Payment processed - 8 mins ago</li>
                        <li>Report generated - 15 mins ago</li>
                    </ul>
                </section>
                
                <section class="top-products">
                    <h2>Top Products</h2>
                    <div class="product-list">
                        <div class="product-item">
                            <span>Web Development Course</span>
                            <span>$99</span>
                        </div>
                        <div class="product-item">
                            <span>UI/UX Design Book</span>
                            <span>$29</span>
                        </div>
                        <div class="product-item">
                            <span>Premium Template</span>
                            <span>$49</span>
                        </div>
                    </div>
                </section>
            </div>
        </main>
        
        <footer class="footer">
            <p>&copy; 2024 Dashboard Inc. All rights reserved.</p>
        </footer>
    </div>
</body>
</html>
```

**Your CSS Task (`dashboard-style.css`):**

```css
/* Reset and base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: #f5f7fa;
    color: #333;
}

/* Dashboard Grid Layout */
.dashboard {
    display: grid;
    grid-template-areas:
        "header header"
        "sidebar main"
        "footer footer";
    grid-template-columns: 250px 1fr;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

/* Header */
.header {
    grid-area: header;
    background: white;
    padding: 20px 30px;
    border-bottom: 1px solid #e1e5e9;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.header h1 {
    color: #2c3e50;
    font-size: 1.8rem;
}

.user-info {
    display: flex;
    align-items: center;
    gap: 12px;
}

.avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
}

/* Sidebar */
.sidebar {
    grid-area: sidebar;
    background: #2c3e50;
    padding: 20px 0;
}

.nav-list {
    list-style: none;
}

.nav-item {
    display: block;
    padding: 15px 30px;
    color: #bdc3c7;
    text-decoration: none;
    transition: all 0.3s ease;
}

.nav-item:hover,
.nav-item.active {
    background: #34495e;
    color: white;
    border-right: 3px solid #3498db;
}

/* Main Content */
.main-content {
    grid-area: main;
    padding: 30px;
    overflow-y: auto;
}

/* Stats Grid */
.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin-bottom: 30px;
}

.stat-card {
    background: white;
    padding: 25px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    text-align: center;
}

.stat-card h3 {
    color: #7f8c8d;
    font-size: 0.9rem;
    margin-bottom: 10px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.stat-number {
    font-size: 2.5rem;
    font-weight: 700;
    color: #2c3e50;
    margin-bottom: 10px;
}

.stat-change {
    padding: 4px 8px;
    border-radius: 4px;
    font-size: 0.8rem;
    font-weight: 600;
}

.stat-change.positive {
    background: #d5f4e6;
    color: #27ae60;
}

.stat-change.negative {
    background: #fadbd8;
    color: #e74c3c;
}

/* Content Grid */
.content-grid {
    display: grid;
    grid-template-columns: 2fr 1fr;
    grid-template-rows: auto auto;
    gap: 20px;
}

.chart-section {
    grid-column: 1 / 2;
    grid-row: 1 / 3;
}

.recent-activity {
    grid-column: 2 / 3;
    grid-row: 1 / 2;
}

.top-products {
    grid-column: 2 / 3;
    grid-row: 2 / 3;
}

/* Content Sections */
.chart-section,
.recent-activity,
.top-products {
    background: white;
    padding: 25px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.chart-section h2,
.recent-activity h2,
.top-products h2 {
    color: #2c3e50;
    margin-bottom: 20px;
    font-size: 1.3rem;
}

.chart-placeholder {
    height: 300px;
    background: #ecf0f1;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #7f8c8d;
    font-size: 1.1rem;
}

.activity-list {
    list-style: none;
}

.activity-list li {
    padding: 12px 0;
    border-bottom: 1px solid #ecf0f1;
    color: #5d6d7e;
}

.activity-list li:last-child {
    border-bottom: none;
}

.product-list {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.product-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 0;
    border-bottom: 1px solid #ecf0f1;
}

.product-item:last-child {
    border-bottom: none;
}

.product-item span:last-child {
    font-weight: 600;
    color: #27ae60;
}

/* Footer */
.footer {
    grid-area: footer;
    background: white;
    padding: 20px 30px;
    border-top: 1px solid #e1e5e9;
    text-align: center;
    color: #7f8c8d;
}

/* Mobile Responsive */
@media (max-width: 1024px) {
    .content-grid {
        grid-template-columns: 1fr;
        grid-template-rows: auto auto auto;
    }
    
    .chart-section,
    .recent-activity,
    .top-products {
        grid-column: 1 / 2;
    }
    
    .chart-section {
        grid-row: 1 / 2;
    }
    
    .recent-activity {
        grid-row: 2 / 3;
    }
    
    .top-products {
        grid-row: 3 / 4;
    }
}

@media (max-width: 768px) {
    .dashboard {
        grid-template-areas:
            "header"
            "main"
            "footer";
        grid-template-columns: 1fr;
    }
    
    .sidebar {
        display: none; /* In a real app, you'd make this a collapsible menu */
    }
    
    .main-content {
        padding: 20px 15px;
    }
    
    .stats-grid {
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
    }
    
    .header {
        padding: 15px 20px;
    }
    
    .header h1 {
        font-size: 1.5rem;
    }
}

@media (max-width: 480px) {
    .stats-grid {
        grid-template-columns: 1fr;
    }
    
    .user-info span {
        display: none;
    }
}
```

---

## 🎨 Exercise 4: Animations and Transitions

### Task: Create an Interactive Card Component

Build an animated card component with hover effects and micro-interactions.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animated Cards</title>
    <link rel="stylesheet" href="cards-style.css">
</head>
<body>
    <div class="cards-container">
        <h1>Interactive Card Components</h1>
        
        <div class="cards-grid">
            <div class="card">
                <div class="card-image">
                    <img src="https://via.placeholder.com/300x200/3498db/ffffff?text=Web+Dev" alt="Web Development">
                    <div class="card-overlay">
                        <button class="btn-favorite">♡</button>
                    </div>
                </div>
                <div class="card-content">
                    <span class="card-category">Development</span>
                    <h3 class="card-title">Web Development Masterclass</h3>
                    <p class="card-description">Learn modern web development with HTML, CSS, JavaScript, and popular frameworks.</p>
                    <div class="card-footer">
                        <span class="card-price">$99</span>
                        <button class="btn-primary">Enroll Now</button>
                    </div>
                </div>
            </div>
            
            <div class="card">
                <div class="card-image">
                    <img src="https://via.placeholder.com/300x200/e74c3c/ffffff?text=UI%2FUX" alt="UI/UX Design">
                    <div class="card-overlay">
                        <button class="btn-favorite">♡</button>
                    </div>
                </div>
                <div class="card-content">
                    <span class="card-category">Design</span>
                    <h3 class="card-title">UI/UX Design Fundamentals</h3>
                    <p class="card-description">Master the principles of user interface and user experience design.</p>
                    <div class="card-footer">
                        <span class="card-price">$79</span>
                        <button class="btn-primary">Enroll Now</button>
                    </div>
                </div>
            </div>
            
            <div class="card">
                <div class="card-image">
                    <img src="https://via.placeholder.com/300x200/27ae60/ffffff?text=Mobile" alt="Mobile Development">
                    <div class="card-overlay">
                        <button class="btn-favorite">♡</button>
                    </div>
                </div>
                <div class="card-content">
                    <span class="card-category">Mobile</span>
                    <h3 class="card-title">React Native Development</h3>
                    <p class="card-description">Build cross-platform mobile applications with React Native framework.</p>
                    <div class="card-footer">
                        <span class="card-price">$129</span>
                        <button class="btn-primary">Enroll Now</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

**Your CSS Task (`cards-style.css`):**

```css
/* Reset and base styles */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    padding: 40px 20px;
}

.cards-container {
    max-width: 1200px;
    margin: 0 auto;
}

.cards-container h1 {
    text-align: center;
    color: white;
    font-size: 2.5rem;
    margin-bottom: 50px;
    text-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

/* Cards Grid */
.cards-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
    gap: 30px;
    padding: 0 20px;
}

/* Card Component */
.card {
    background: white;
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
    position: relative;
    cursor: pointer;
}

.card:hover {
    transform: translateY(-10px) scale(1.02);
    box-shadow: 0 20px 40px rgba(0,0,0,0.3);
}

/* Card Image */
.card-image {
    position: relative;
    overflow: hidden;
    height: 200px;
}

.card-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.3s ease;
}

.card:hover .card-image img {
    transform: scale(1.1);
}

.card-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,0.4);
    display: flex;
    align-items: flex-start;
    justify-content: flex-end;
    padding: 15px;
    opacity: 0;
    transition: opacity 0.3s ease;
}

.card:hover .card-overlay {
    opacity: 1;
}

.btn-favorite {
    background: rgba(255,255,255,0.9);
    border: none;
    border-radius: 50%;
    width: 45px;
    height: 45px;
    font-size: 1.5rem;
    cursor: pointer;
    transition: all 0.3s ease;
    color: #e74c3c;
}

.btn-favorite:hover {
    background: white;
    transform: scale(1.1);
    color: #c0392b;
}

/* Card Content */
.card-content {
    padding: 25px;
}

.card-category {
    display: inline-block;
    background: #ecf0f1;
    color: #7f8c8d;
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 0.8rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-bottom: 15px;
}

.card-title {
    font-size: 1.4rem;
    color: #2c3e50;
    margin-bottom: 12px;
    line-height: 1.3;
    transition: color 0.3s ease;
}

.card:hover .card-title {
    color: #3498db;
}

.card-description {
    color: #7f8c8d;
    line-height: 1.6;
    margin-bottom: 25px;
    font-size: 0.95rem;
}

/* Card Footer */
.card-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.card-price {
    font-size: 1.5rem;
    font-weight: 700;
    color: #27ae60;
}

.btn-primary {
    background: linear-gradient(45deg, #3498db, #2980b9);
    color: white;
    border: none;
    padding: 12px 24px;
    border-radius: 25px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
    overflow: hidden;
}

.btn-primary::before {
    content: '';
    position: absolute;
    top: 0;
    left: -100%;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, transparent, rgba(255,255,255,0.2), transparent);
    transition: left 0.5s ease;
}

.btn-primary:hover::before {
    left: 100%;
}

.btn-primary:hover {
    background: linear-gradient(45deg, #2980b9, #1f6397);
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(52, 152, 219, 0.4);
}

.btn-primary:active {
    transform: translateY(0);
}

/* Loading Animation */
@keyframes pulse {
    0% {
        box-shadow: 0 0 0 0 rgba(52, 152, 219, 0.7);
    }
    70% {
        box-shadow: 0 0 0 10px rgba(52, 152, 219, 0);
    }
    100% {
        box-shadow: 0 0 0 0 rgba(52, 152, 219, 0);
    }
}

.card:nth-child(1) {
    animation: slideInUp 0.6s ease-out 0.1s both;
}

.card:nth-child(2) {
    animation: slideInUp 0.6s ease-out 0.2s both;
}

.card:nth-child(3) {
    animation: slideInUp 0.6s ease-out 0.3s both;
}

@keyframes slideInUp {
    from {
        opacity: 0;
        transform: translateY(60px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Shimmer effect for loading */
@keyframes shimmer {
    0% {
        background-position: -200px 0;
    }
    100% {
        background-position: calc(200px + 100%) 0;
    }
}

.card.loading {
    background: #f6f7f8;
    background-image: linear-gradient(
        90deg,
        #f6f7f8 0px,
        #edeef1 40px,
        #f6f7f8 80px
    );
    background-size: 200px;
    animation: shimmer 1.5s infinite linear;
}

/* Responsive Design */
@media (max-width: 768px) {
    .cards-grid {
        grid-template-columns: 1fr;
        gap: 20px;
        padding: 0 10px;
    }
    
    .cards-container h1 {
        font-size: 2rem;
        margin-bottom: 30px;
    }
    
    .card-content {
        padding: 20px;
    }
    
    .card-footer {
        flex-direction: column;
        gap: 15px;
        align-items: stretch;
    }
    
    .btn-primary {
        width: 100%;
        justify-content: center;
    }
}

/* Focus states for accessibility */
.btn-favorite:focus,
.btn-primary:focus {
    outline: 2px solid #3498db;
    outline-offset: 2px;
}

/* Reduced motion for accessibility */
@media (prefers-reduced-motion: reduce) {
    .card,
    .card-image img,
    .btn-primary,
    .btn-favorite {
        transition: none;
    }
    
    .card:nth-child(n) {
        animation: none;
    }
}
```

---

## 🎯 Practice Assignments

### Assignment 1: Responsive Portfolio Layout
Create a responsive portfolio website with:
- Header with navigation
- Hero section with CSS animations
- Project grid using CSS Grid
- Contact form with custom styling
- Footer with social links

### Assignment 2: E-commerce Product Page
Build a product page featuring:
- Image gallery with hover effects
- Product information layout
- Price and purchase section
- Related products grid
- Responsive design for all devices

### Assignment 3: CSS Component Library
Create a component library with:
- Buttons (primary, secondary, outline, disabled states)
- Form elements (inputs, selects, checkboxes)
- Cards with different variants
- Navigation components
- Loading states and animations

## 🔍 Self-Assessment Questions

1. How does the CSS box model affect element sizing?
2. When should you use Flexbox vs CSS Grid?
3. What are the differences between relative, absolute, and fixed positioning?
4. How do CSS transitions differ from animations?
5. What makes a design responsive?
6. How do you optimize CSS for performance?
7. What are CSS custom properties and how do you use them?

## 📚 Additional Resources

- [CSS Grid Garden](https://cssgridgarden.com/) - Interactive Grid learning
- [Flexbox Froggy](https://flexboxfroggy.com/) - Interactive Flexbox learning
- [CSS Tricks](https://css-tricks.com/) - Comprehensive CSS guides
- [Can I Use](https://caniuse.com/) - Browser compatibility checker
- [CSS Animation Rocks](https://cssanimation.rocks/) - Animation tutorials

---

**Next:** Proceed to `advanced.md` for CSS architecture patterns and advanced techniques.
