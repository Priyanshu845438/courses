
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
