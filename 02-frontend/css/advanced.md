
# 🚀 Advanced CSS

## Table of Contents
- [Modern Layout Techniques](#modern-layout-techniques)
- [CSS Architecture](#css-architecture)
- [Advanced Animations](#advanced-animations)
- [CSS Preprocessing](#css-preprocessing)
- [Performance Optimization](#performance-optimization)
- [CSS-in-JS](#css-in-js)
- [Advanced Selectors](#advanced-selectors)
- [CSS Houdini](#css-houdini)
- [Modern CSS Features](#modern-css-features)

---

## 🎯 Modern Layout Techniques

### CSS Subgrid

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
}

.nested-grid {
    display: grid;
    grid-template-columns: subgrid;
    grid-template-rows: subgrid;
    grid-column: 1 / -1;
    gap: inherit;
}

/* Card layout with subgrid */
.card-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
}

.card {
    display: grid;
    grid-template-rows: subgrid;
    grid-row: span 3;
}

.card-header,
.card-body,
.card-footer {
    padding: 1rem;
}
```

### Container Queries

```css
/* Container query */
.card-container {
    container-type: inline-size;
}

@container (min-width: 400px) {
    .card {
        display: flex;
        flex-direction: row;
    }
    
    .card-image {
        width: 150px;
        height: 150px;
    }
}

@container (max-width: 300px) {
    .card {
        font-size: 0.9rem;
    }
    
    .card-image {
        width: 100%;
        height: 120px;
    }
}

/* Named containers */
.sidebar {
    container-name: sidebar;
    container-type: inline-size;
}

@container sidebar (min-width: 250px) {
    .widget {
        padding: 1rem;
    }
}
```

### CSS Intrinsic Web Design

```css
/* Intrinsic sizing */
.responsive-layout {
    display: grid;
    grid-template-columns: 
        minmax(150px, 25%) 
        minmax(300px, 1fr) 
        minmax(100px, 20%);
    gap: 20px;
}

/* Aspect ratio */
.video-container {
    aspect-ratio: 16 / 9;
    width: 100%;
    background-color: #000;
}

.square-image {
    aspect-ratio: 1;
    object-fit: cover;
}

/* Logical properties */
.content {
    margin-inline: auto;
    padding-inline: 1rem;
    max-inline-size: 60ch;
    border-inline-start: 3px solid blue;
}

/* Flow layout */
.flow > * + * {
    margin-block-start: 1rem;
}
```

---

## 🏗️ CSS Architecture

### BEM (Block Element Modifier)

```css
/* Block */
.button {
    display: inline-block;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    transition: all 0.3s ease;
}

/* Element */
.button__icon {
    margin-right: 0.5rem;
}

.button__text {
    font-weight: 500;
}

/* Modifier */
.button--primary {
    background-color: #007bff;
    color: white;
}

.button--secondary {
    background-color: #6c757d;
    color: white;
}

.button--large {
    padding: 0.75rem 1.5rem;
    font-size: 1.1rem;
}

.button--small {
    padding: 0.25rem 0.5rem;
    font-size: 0.9rem;
}

/* Complex example */
.card {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    overflow: hidden;
}

.card__header {
    padding: 1rem;
    background-color: #f8f9fa;
    border-bottom: 1px solid #dee2e6;
}

.card__title {
    margin: 0;
    font-size: 1.25rem;
    font-weight: 600;
}

.card__body {
    padding: 1rem;
}

.card__footer {
    padding: 1rem;
    background-color: #f8f9fa;
    border-top: 1px solid #dee2e6;
}

.card--featured {
    border: 2px solid #007bff;
}

.card--compact .card__header,
.card--compact .card__body,
.card--compact .card__footer {
    padding: 0.5rem;
}
```

### ITCSS (Inverted Triangle CSS)

```css
/* Settings - Variables and config */
:root {
    --color-primary: #007bff;
    --color-secondary: #6c757d;
    --font-size-base: 16px;
    --line-height-base: 1.5;
    --spacing-unit: 1rem;
}

/* Tools - Mixins and functions */
@mixin button-variant($bg-color, $text-color) {
    background-color: $bg-color;
    color: $text-color;
    border: 1px solid $bg-color;
    
    &:hover {
        background-color: darken($bg-color, 10%);
        border-color: darken($bg-color, 10%);
    }
}

/* Generic - Normalize, reset, box-sizing */
*,
*::before,
*::after {
    box-sizing: border-box;
}

html {
    font-size: var(--font-size-base);
    line-height: var(--line-height-base);
}

body {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* Elements - Bare HTML elements */
h1, h2, h3, h4, h5, h6 {
    margin-top: 0;
    margin-bottom: var(--spacing-unit);
    font-weight: 600;
}

p {
    margin-top: 0;
    margin-bottom: var(--spacing-unit);
}

a {
    color: var(--color-primary);
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}

/* Objects - Design patterns, layout */
.o-container {
    max-width: 1200px;
    margin-left: auto;
    margin-right: auto;
    padding-left: var(--spacing-unit);
    padding-right: var(--spacing-unit);
}

.o-grid {
    display: grid;
    gap: var(--spacing-unit);
}

.o-flex {
    display: flex;
}

.o-flex--center {
    justify-content: center;
    align-items: center;
}

/* Components - UI components */
.c-button {
    display: inline-block;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    text-align: center;
    transition: all 0.3s ease;
}

.c-button--primary {
    background-color: var(--color-primary);
    color: white;
}

.c-button--secondary {
    background-color: var(--color-secondary);
    color: white;
}

.c-card {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    overflow: hidden;
}

/* Utilities - Overrides, helpers */
.u-text-center {
    text-align: center !important;
}

.u-margin-bottom-small {
    margin-bottom: calc(var(--spacing-unit) * 0.5) !important;
}

.u-margin-bottom-large {
    margin-bottom: calc(var(--spacing-unit) * 2) !important;
}

.u-hidden {
    display: none !important;
}

.u-visually-hidden {
    position: absolute !important;
    width: 1px !important;
    height: 1px !important;
    padding: 0 !important;
    margin: -1px !important;
    overflow: hidden !important;
    clip: rect(0, 0, 0, 0) !important;
    border: 0 !important;
}
```

### Atomic CSS

```css
/* Layout */
.d-flex { display: flex; }
.d-block { display: block; }
.d-inline { display: inline; }
.d-inline-block { display: inline-block; }
.d-grid { display: grid; }
.d-none { display: none; }

/* Flexbox */
.flex-row { flex-direction: row; }
.flex-column { flex-direction: column; }
.justify-center { justify-content: center; }
.justify-between { justify-content: space-between; }
.align-center { align-items: center; }
.align-start { align-items: flex-start; }
.flex-wrap { flex-wrap: wrap; }
.flex-1 { flex: 1; }

/* Spacing */
.m-0 { margin: 0; }
.m-1 { margin: 0.25rem; }
.m-2 { margin: 0.5rem; }
.m-3 { margin: 1rem; }
.m-4 { margin: 1.5rem; }
.m-5 { margin: 3rem; }

.mt-0 { margin-top: 0; }
.mt-1 { margin-top: 0.25rem; }
.mt-2 { margin-top: 0.5rem; }
.mt-3 { margin-top: 1rem; }

.p-0 { padding: 0; }
.p-1 { padding: 0.25rem; }
.p-2 { padding: 0.5rem; }
.p-3 { padding: 1rem; }
.p-4 { padding: 1.5rem; }
.p-5 { padding: 3rem; }

/* Typography */
.text-xs { font-size: 0.75rem; }
.text-sm { font-size: 0.875rem; }
.text-base { font-size: 1rem; }
.text-lg { font-size: 1.125rem; }
.text-xl { font-size: 1.25rem; }
.text-2xl { font-size: 1.5rem; }
.text-3xl { font-size: 1.875rem; }

.text-left { text-align: left; }
.text-center { text-align: center; }
.text-right { text-align: right; }

.font-thin { font-weight: 100; }
.font-light { font-weight: 300; }
.font-normal { font-weight: 400; }
.font-medium { font-weight: 500; }
.font-semibold { font-weight: 600; }
.font-bold { font-weight: 700; }

/* Colors */
.text-white { color: white; }
.text-black { color: black; }
.text-gray-100 { color: #f7fafc; }
.text-gray-500 { color: #a0aec0; }
.text-gray-900 { color: #1a202c; }

.bg-white { background-color: white; }
.bg-black { background-color: black; }
.bg-gray-100 { background-color: #f7fafc; }
.bg-gray-500 { background-color: #a0aec0; }
.bg-blue-500 { background-color: #4299e1; }
.bg-red-500 { background-color: #f56565; }

/* Sizing */
.w-0 { width: 0; }
.w-1 { width: 0.25rem; }
.w-2 { width: 0.5rem; }
.w-3 { width: 1rem; }
.w-4 { width: 1.5rem; }
.w-5 { width: 3rem; }
.w-full { width: 100%; }
.w-screen { width: 100vw; }

.h-0 { height: 0; }
.h-1 { height: 0.25rem; }
.h-2 { height: 0.5rem; }
.h-3 { height: 1rem; }
.h-4 { height: 1.5rem; }
.h-5 { height: 3rem; }
.h-full { height: 100%; }
.h-screen { height: 100vh; }

/* Borders */
.border { border: 1px solid #e2e8f0; }
.border-0 { border: 0; }
.border-2 { border: 2px solid #e2e8f0; }
.border-t { border-top: 1px solid #e2e8f0; }
.border-r { border-right: 1px solid #e2e8f0; }
.border-b { border-bottom: 1px solid #e2e8f0; }
.border-l { border-left: 1px solid #e2e8f0; }

.rounded { border-radius: 0.25rem; }
.rounded-md { border-radius: 0.375rem; }
.rounded-lg { border-radius: 0.5rem; }
.rounded-full { border-radius: 9999px; }

/* Shadows */
.shadow-sm { box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05); }
.shadow { box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1); }
.shadow-md { box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1); }
.shadow-lg { box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
.shadow-xl { box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1); }
```

---

## 🎬 Advanced Animations

### Complex Keyframe Animations

```css
/* Multi-stage loading animation */
@keyframes complexLoader {
    0% {
        transform: rotate(0deg) scale(1);
        border-radius: 50%;
        background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    }
    25% {
        transform: rotate(90deg) scale(1.2);
        border-radius: 25%;
        background: linear-gradient(135deg, #4ecdc4, #45b7d1);
    }
    50% {
        transform: rotate(180deg) scale(1);
        border-radius: 0%;
        background: linear-gradient(225deg, #45b7d1, #96ceb4);
    }
    75% {
        transform: rotate(270deg) scale(0.8);
        border-radius: 25%;
        background: linear-gradient(315deg, #96ceb4, #feca57);
    }
    100% {
        transform: rotate(360deg) scale(1);
        border-radius: 50%;
        background: linear-gradient(45deg, #feca57, #ff6b6b);
    }
}

.complex-loader {
    width: 60px;
    height: 60px;
    animation: complexLoader 3s infinite ease-in-out;
}

/* Text animation effects */
@keyframes typewriter {
    from { width: 0; }
    to { width: 100%; }
}

@keyframes blinkCursor {
    from, to { border-color: transparent; }
    50% { border-color: #333; }
}

.typewriter-text {
    overflow: hidden;
    border-right: 2px solid #333;
    white-space: nowrap;
    margin: 0 auto;
    animation: 
        typewriter 4s steps(40, end),
        blinkCursor 0.75s step-end infinite;
}

/* Morphing shapes */
@keyframes morphShape {
    0% {
        border-radius: 50%;
        transform: rotate(0deg);
    }
    25% {
        border-radius: 25% 75%;
        transform: rotate(90deg);
    }
    50% {
        border-radius: 75% 25% 25% 75%;
        transform: rotate(180deg);
    }
    75% {
        border-radius: 25%;
        transform: rotate(270deg);
    }
    100% {
        border-radius: 50%;
        transform: rotate(360deg);
    }
}

.morphing-shape {
    width: 100px;
    height: 100px;
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    animation: morphShape 4s infinite ease-in-out;
}
```

### CSS Physics Animations

```css
/* Bouncing ball with realistic physics */
@keyframes bouncingBall {
    0% {
        transform: translateY(0);
        animation-timing-function: ease-out;
    }
    50% {
        transform: translateY(200px);
        animation-timing-function: ease-in;
    }
    55% {
        transform: translateY(150px);
        animation-timing-function: ease-out;
    }
    65% {
        transform: translateY(200px);
        animation-timing-function: ease-in;
    }
    70% {
        transform: translateY(175px);
        animation-timing-function: ease-out;
    }
    80% {
        transform: translateY(200px);
        animation-timing-function: ease-in;
    }
    85% {
        transform: translateY(190px);
        animation-timing-function: ease-out;
    }
    95% {
        transform: translateY(200px);
        animation-timing-function: ease-in;
    }
    100% {
        transform: translateY(200px);
    }
}

.bouncing-ball {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background: radial-gradient(circle at 30% 30%, #ff6b6b, #d63031);
    animation: bouncingBall 2s infinite;
}

/* Pendulum swing */
@keyframes pendulumSwing {
    0%, 100% {
        transform: rotate(30deg);
        animation-timing-function: ease-in-out;
    }
    50% {
        transform: rotate(-30deg);
        animation-timing-function: ease-in-out;
    }
}

.pendulum {
    width: 2px;
    height: 150px;
    background: #333;
    transform-origin: top center;
    animation: pendulumSwing 2s infinite;
}

.pendulum::after {
    content: '';
    position: absolute;
    bottom: -15px;
    left: -13px;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    background: #feca57;
}
```

---

## 🎨 Advanced Visual Effects

### CSS Filters and Blend Modes

```css
/* Image filter effects */
.filter-gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 20px;
    margin: 20px 0;
}

.filter-item {
    position: relative;
    overflow: hidden;
    border-radius: 8px;
    transition: all 0.3s ease;
}

.filter-item img {
    width: 100%;
    height: 200px;
    object-fit: cover;
    transition: all 0.3s ease;
}

/* Different filter effects */
.filter-vintage img {
    filter: sepia(0.8) contrast(1.2) brightness(1.1) saturate(0.8);
}

.filter-dramatic img {
    filter: contrast(1.5) brightness(0.9) saturate(1.3) hue-rotate(10deg);
}

.filter-cyberpunk img {
    filter: hue-rotate(270deg) saturate(2) contrast(1.5) brightness(1.2);
}

.filter-noir img {
    filter: grayscale(1) contrast(1.3) brightness(0.9);
}

.filter-dreamy img {
    filter: blur(1px) brightness(1.2) saturate(1.3) hue-rotate(15deg);
}

/* Hover effects with filters */
.filter-item:hover img {
    filter: none;
    transform: scale(1.05);
}

/* Blend mode effects */
.blend-overlay {
    position: relative;
    overflow: hidden;
}

.blend-overlay::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    mix-blend-mode: overlay;
    opacity: 0;
    transition: opacity 0.3s ease;
}

.blend-overlay:hover::before {
    opacity: 0.8;
}
```

### CSS Shapes and Clipping

```css
/* Advanced CSS shapes */
.shape-gallery {
    display: flex;
    flex-wrap: wrap;
    gap: 30px;
    justify-content: center;
    margin: 40px 0;
}

.shape-item {
    width: 120px;
    height: 120px;
    background: linear-gradient(135deg, #667eea, #764ba2);
    position: relative;
}

/* Heart shape */
.heart-shape {
    transform: rotate(-45deg);
    border-radius: 50% 0;
}

.heart-shape::before,
.heart-shape::after {
    content: '';
    position: absolute;
    width: 120px;
    height: 120px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea, #764ba2);
}

.heart-shape::before {
    top: -60px;
    left: 0;
}

.heart-shape::after {
    top: 0;
    left: 60px;
}

/* Star shape */
.star-shape {
    clip-path: polygon(
        50% 0%, 
        61% 35%, 
        98% 35%, 
        68% 57%, 
        79% 91%, 
        50% 70%, 
        21% 91%, 
        32% 57%, 
        2% 35%, 
        39% 35%
    );
}

/* Hexagon shape */
.hexagon-shape {
    clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%);
}

/* Arrow shape */
.arrow-shape {
    clip-path: polygon(40% 0%, 40% 20%, 100% 20%, 100% 80%, 40% 80%, 40% 100%, 0% 50%);
}

/* Complex polygon */
.complex-shape {
    clip-path: polygon(
        20% 0%, 
        80% 0%, 
        100% 20%, 
        100% 80%, 
        80% 100%, 
        20% 100%, 
        0% 80%, 
        0% 20%
    );
}
```

---

## 🌊 CSS Scroll Effects

### Scroll-triggered Animations

```css
/* Scroll-based parallax */
.parallax-container {
    height: 100vh;
    overflow-x: hidden;
    overflow-y: auto;
    perspective: 1px;
}

.parallax-layer {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}

.parallax-back {
    transform: translateZ(-1px) scale(2);
}

.parallax-mid {
    transform: translateZ(-0.5px) scale(1.5);
}

.parallax-front {
    transform: translateZ(0);
}

/* Scroll snap */
.scroll-snap-container {
    height: 100vh;
    overflow-y: scroll;
    scroll-snap-type: y mandatory;
}

.scroll-snap-section {
    height: 100vh;
    scroll-snap-align: start;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2rem;
}

.scroll-snap-section:nth-child(odd) {
    background: linear-gradient(135deg, #667eea, #764ba2);
    color: white;
}

.scroll-snap-section:nth-child(even) {
    background: linear-gradient(135deg, #f093fb, #f5576c);
    color: white;
}

/* Sticky navigation with backdrop filter */
.sticky-nav {
    position: sticky;
    top: 0;
    z-index: 100;
    backdrop-filter: blur(10px) saturate(180%);
    background: rgba(255, 255, 255, 0.8);
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
}

.sticky-nav.scrolled {
    backdrop-filter: blur(20px) saturate(200%);
    background: rgba(255, 255, 255, 0.95);
    box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
}
```

---

## 📱 Responsive Design Mastery

### Container Queries

```css
/* Container query support */
.card-container {
    container-type: inline-size;
    container-name: card;
}

.card {
    padding: 1rem;
    border-radius: 8px;
    background: white;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

/* Adjust layout based on container size */
@container card (min-width: 300px) {
    .card {
        display: grid;
        grid-template-columns: 1fr 2fr;
        gap: 1rem;
    }
}

@container card (min-width: 500px) {
    .card {
        grid-template-columns: 1fr 3fr;
        padding: 2rem;
    }
    
    .card h2 {
        font-size: 1.5rem;
    }
}

/* Intrinsic web design patterns */
.intrinsic-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
    gap: 1rem;
}

.flexible-layout {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.flexible-layout > * {
    flex: 1 1 min(300px, 100%);
}
```

---

## 🎯 Performance Optimization

### CSS Optimization Techniques

```css
/* Efficient selectors */
/* ❌ Avoid universal selectors */
* {
    box-sizing: border-box; /* Only use when necessary */
}

/* ✅ Use specific selectors */
.component {
    box-sizing: border-box;
}

/* ❌ Avoid deep nesting */
.nav ul li a span {
    color: blue;
}

/* ✅ Use flatter hierarchy */
.nav-link-text {
    color: blue;
}

/* Critical CSS loading */
/* Inline critical styles in <head> */
.above-fold {
    display: block;
    font-family: Arial, sans-serif;
    line-height: 1.4;
}

/* Load non-critical styles asynchronously */
/*
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
*/

/* GPU acceleration for animations */
.gpu-accelerated {
    transform: translateZ(0); /* Force GPU layer */
    will-change: transform; /* Hint browser about changes */
}

/* Contain layout changes */
.contained-element {
    contain: layout style paint;
}

/* Optimize repaints */
.smooth-animation {
    transform: translateX(0); /* Use transform instead of left/top */
    opacity: 1; /* Use opacity instead of visibility */
    transition: transform 0.3s ease, opacity 0.3s ease;
}
```

---

## 📊 Modern CSS Features

### CSS Custom Properties (Advanced)

```css
/* Dynamic color schemes */
:root {
    --hue: 220;
    --saturation: 70%;
    --lightness: 50%;
    
    --primary: hsl(var(--hue), var(--saturation), var(--lightness));
    --primary-light: hsl(var(--hue), var(--saturation), calc(var(--lightness) + 20%));
    --primary-dark: hsl(var(--hue), var(--saturation), calc(var(--lightness) - 20%));
    
    --spacing-unit: 8px;
    --spacing-xs: calc(var(--spacing-unit) * 0.5);
    --spacing-sm: calc(var(--spacing-unit) * 1);
    --spacing-md: calc(var(--spacing-unit) * 2);
    --spacing-lg: calc(var(--spacing-unit) * 3);
    --spacing-xl: calc(var(--spacing-unit) * 4);
}

/* Theme switching */
[data-theme="dark"] {
    --bg-color: #1a1a1a;
    --text-color: #ffffff;
    --border-color: #333333;
}

[data-theme="light"] {
    --bg-color: #ffffff;
    --text-color: #000000;
    --border-color: #e0e0e0;
}

/* Responsive variables */
@media (max-width: 768px) {
    :root {
        --spacing-unit: 4px;
    }
}

/* Math functions */
.responsive-text {
    font-size: clamp(1rem, 4vw, 2rem);
    line-height: calc(1em + 0.5rem);
    margin-bottom: max(1rem, 3vh);
}

.golden-ratio {
    aspect-ratio: 1.618 / 1; /* Golden ratio */
}

.responsive-grid {
    grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
    gap: clamp(1rem, 5vw, 3rem);
}
```

### CSS Layers and Cascade Control

```css
/* CSS Layers for better cascade control */
@layer reset, base, components, utilities;

@layer reset {
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
}

@layer base {
    body {
        font-family: system-ui, sans-serif;
        line-height: 1.6;
        color: var(--text-color);
    }
}

@layer components {
    .button {
        padding: 0.5rem 1rem;
        border: none;
        border-radius: 4px;
        background: var(--primary);
        color: white;
        cursor: pointer;
    }
}

@layer utilities {
    .sr-only {
        position: absolute;
        width: 1px;
        height: 1px;
        padding: 0;
        margin: -1px;
        overflow: hidden;
        clip: rect(0, 0, 0, 0);
        white-space: nowrap;
        border: 0;
    }
}
```

### Logical Properties

```css
/* Use logical properties for better internationalization */
.card {
    margin-block: 1rem; /* margin-top and margin-bottom */
    margin-inline: 2rem; /* margin-left and margin-right */
    padding-block-start: 1rem; /* padding-top */
    padding-block-end: 1rem; /* padding-bottom */
    padding-inline-start: 2rem; /* padding-left in LTR, padding-right in RTL */
    padding-inline-end: 2rem; /* padding-right in LTR, padding-left in RTL */
    
    border-inline-start: 2px solid var(--primary); /* left border in LTR */
    border-start-start-radius: 8px; /* top-left corner in LTR */
    
    inline-size: 100%; /* width */
    block-size: auto; /* height */
}

/* Text alignment for RTL support */
.text-content {
    text-align: start; /* left in LTR, right in RTL */
}
```

### CSS Math Functions

```css
/* Advanced math functions */
.responsive-container {
    width: min(90vw, 1200px); /* Responsive width with max */
    padding: max(1rem, 3vw); /* Responsive padding with min */
    margin: clamp(1rem, 5vw, 4rem) auto; /* Fluid margin */
}

.aspect-ratio-box {
    aspect-ratio: 16 / 9; /* Modern aspect ratio */
    background: var(--primary);
}

.circular-progress {
    background: conic-gradient(
        var(--primary) calc(var(--progress) * 1%), 
        #e0e0e0 0
    );
    border-radius: 50%;
    mask: radial-gradient(circle at center, transparent 40%, black 40%);
}

/* Advanced calculations */
.grid-with-gap {
    --columns: 3;
    --gap: 1rem;
    display: grid;
    grid-template-columns: repeat(
        var(--columns), 
        calc((100% - (var(--columns) - 1) * var(--gap)) / var(--columns))
    );
    gap: var(--gap);
}

/* Trigonometric functions (upcoming) */
.rotating-element {
    transform: rotate(calc(sin(var(--angle)) * 1turn));
}

/* Round function for pixel-perfect layouts */
.pixel-perfect {
    width: round(down, 33.333%, 1px); /* Rounds down to nearest 5px */
}
```

### CSS Nesting

```css
.component {
    background-color: white;
    border-radius: 8px;
    
    .header {
        padding: 1rem;
        
        h2 {
            margin: 0;
            color: #333;
        }
    }
    
    .content {
        padding: 1rem;
        
        p {
            margin-bottom: 1rem;
            
            &:last-child {
                margin-bottom: 0;
            }
        }
        
        a {
            color: #007bff;
            
            &:hover {
                text-decoration: underline;
            }
        }
    }
    
    &:hover {
        box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    }
    
    &.featured {
        border: 2px solid #007bff;
    }
}
```

This comprehensive advanced CSS guide covers cutting-edge techniques and modern features that will help you create sophisticated, performant, and maintainable stylesheets. Keep practicing these concepts to master advanced CSS development!ced Animations

### Complex Keyframe Animations

```css
@keyframes morphing {
    0% {
        transform: scale(1) rotate(0deg);
        border-radius: 50%;
        background-color: #e74c3c;
    }
    25% {
        transform: scale(1.2) rotate(90deg);
        border-radius: 25%;
        background-color: #f39c12;
    }
    50% {
        transform: scale(0.8) rotate(180deg);
        border-radius: 0%;
        background-color: #27ae60;
    }
    75% {
        transform: scale(1.1) rotate(270deg);
        border-radius: 75%;
        background-color: #3498db;
    }
    100% {
        transform: scale(1) rotate(360deg);
        border-radius: 50%;
        background-color: #9b59b6;
    }
}

.morphing-element {
    width: 100px;
    height: 100px;
    margin: 50px auto;
    animation: morphing 3s ease-in-out infinite;
}

/* Staggered animations */
.stagger-container {
    display: flex;
    gap: 10px;
    justify-content: center;
}

.stagger-item {
    width: 20px;
    height: 20px;
    background-color: #3498db;
    border-radius: 50%;
    animation: bounce 1s ease-in-out infinite;
}

.stagger-item:nth-child(1) { animation-delay: 0s; }
.stagger-item:nth-child(2) { animation-delay: 0.1s; }
.stagger-item:nth-child(3) { animation-delay: 0.2s; }
.stagger-item:nth-child(4) { animation-delay: 0.3s; }
.stagger-item:nth-child(5) { animation-delay: 0.4s; }

@keyframes bounce {
    0%, 100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-20px);
    }
}

/* Text animations */
.typewriter {
    overflow: hidden;
    border-right: 0.15em solid orange;
    white-space: nowrap;
    margin: 0 auto;
    letter-spacing: 0.15em;
    animation: 
        typing 3.5s steps(40, end),
        blink-caret 0.75s step-end infinite;
}

@keyframes typing {
    from { width: 0; }
    to { width: 100%; }
}

@keyframes blink-caret {
    from, to { border-color: transparent; }
    50% { border-color: orange; }
}

/* Parallax scrolling effect */
.parallax-container {
    height: 100vh;
    overflow-x: hidden;
    overflow-y: auto;
    perspective: 1px;
}

.parallax-layer {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}

.parallax-back {
    transform: translateZ(-1px) scale(2);
}

.parallax-base {
    transform: translateZ(0);
}
```

### CSS Animation Performance

```css
/* Optimized animations */
.optimized-animation {
    /* Use transform and opacity for better performance */
    transform: translateX(0);
    opacity: 1;
    will-change: transform, opacity;
    transition: transform 0.3s ease, opacity 0.3s ease;
}

.optimized-animation:hover {
    transform: translateX(10px);
    opacity: 0.8;
}

/* Avoid animating these properties */
.avoid-animation {
    /* These cause layout recalculations */
    /* width: 100px; */
    /* height: 100px; */
    /* padding: 10px; */
    /* margin: 10px; */
    
    /* These cause paint operations */
    /* background-color: red; */
    /* color: blue; */
    /* border: 1px solid black; */
}

/* Preferred animations */
.preferred-animation {
    /* These only affect the composite layer */
    transform: translateX(0) translateY(0) scale(1);
    opacity: 1;
    filter: blur(0px);
}

/* Animation with proper timing */
.smooth-animation {
    animation: smoothMove 2s cubic-bezier(0.25, 0.46, 0.45, 0.94) infinite;
}

@keyframes smoothMove {
    0% { transform: translateX(0); }
    50% { transform: translateX(200px); }
    100% { transform: translateX(0); }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
    .animation-element {
        animation: none;
        transition: none;
    }
}
```

### Advanced Animation Techniques

```css
/* Path animations */
.path-animation {
    offset-path: path('M 0,20 Q 50,0 100,20 T 200,20');
    animation: moveAlongPath 3s ease-in-out infinite;
}

@keyframes moveAlongPath {
    0% { offset-distance: 0%; }
    100% { offset-distance: 100%; }
}

/* Mask animations */
.mask-animation {
    mask: linear-gradient(90deg, transparent, black, transparent);
    mask-size: 200% 100%;
    animation: shine 2s ease-in-out infinite;
}

@keyframes shine {
    0% { mask-position: -100% 0; }
    100% { mask-position: 100% 0; }
}

/* Clip-path animations */
.clip-path-animation {
    clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
    animation: reveal 2s ease-in-out infinite alternate;
}

@keyframes reveal {
    from { clip-path: polygon(0 0, 0 0, 0 100%, 0 100%); }
    to { clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%); }
}

/* 3D animations */
.cube-3d {
    width: 100px;
    height: 100px;
    position: relative;
    transform-style: preserve-3d;
    animation: rotateCube 4s ease-in-out infinite;
}

.cube-face {
    position: absolute;
    width: 100px;
    height: 100px;
    border: 1px solid #ccc;
    opacity: 0.8;
}

.cube-front { background: #ff0000; transform: translateZ(50px); }
.cube-back { background: #00ff00; transform: translateZ(-50px) rotateY(180deg); }
.cube-right { background: #0000ff; transform: translateX(50px) rotateY(90deg); }
.cube-left { background: #ffff00; transform: translateX(-50px) rotateY(-90deg); }
.cube-top { background: #ff00ff; transform: translateY(-50px) rotateX(90deg); }
.cube-bottom { background: #00ffff; transform: translateY(50px) rotateX(-90deg); }

@keyframes rotateCube {
    0% { transform: rotateX(0deg) rotateY(0deg); }
    25% { transform: rotateX(90deg) rotateY(0deg); }
    50% { transform: rotateX(90deg) rotateY(90deg); }
    75% { transform: rotateX(0deg) rotateY(90deg); }
    100% { transform: rotateX(0deg) rotateY(0deg); }
}
```

---

## 🎨 CSS Preprocessing

### Sass/SCSS Advanced Features

```scss
// Variables and maps
$primary-color: #007bff;
$secondary-color: #6c757d;

$font-sizes: (
    small: 0.8rem,
    medium: 1rem,
    large: 1.2rem,
    xl: 1.5rem
);

$breakpoints: (
    small: 576px,
    medium: 768px,
    large: 992px,
    xl: 1200px
);

// Mixins
@mixin flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}

@mixin button-variant($bg-color, $text-color: white) {
    background-color: $bg-color;
    color: $text-color;
    border: 1px solid $bg-color;
    
    &:hover {
        background-color: darken($bg-color, 10%);
        border-color: darken($bg-color, 10%);
    }
    
    &:active {
        background-color: darken($bg-color, 15%);
        transform: translateY(1px);
    }
}

@mixin respond-to($breakpoint) {
    @if map-has-key($breakpoints, $breakpoint) {
        @media (min-width: map-get($breakpoints, $breakpoint)) {
            @content;
        }
    }
}

// Functions
@function rem($pixels) {
    @return ($pixels / 16) * 1rem;
}

@function color-contrast($color) {
    @if (lightness($color) > 50%) {
        @return black;
    } @else {
        @return white;
    }
}

// Placeholders
%card-base {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    padding: 1rem;
}

%button-base {
    display: inline-block;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    text-align: center;
    text-decoration: none;
    transition: all 0.3s ease;
    
    &:focus {
        outline: none;
        box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
    }
}

// Usage
.card {
    @extend %card-base;
    
    &__header {
        margin-bottom: 1rem;
        padding-bottom: 0.5rem;
        border-bottom: 1px solid #e9ecef;
    }
    
    &__title {
        margin: 0;
        font-size: map-get($font-sizes, large);
        color: $primary-color;
    }
    
    &__content {
        margin-bottom: 1rem;
        line-height: 1.6;
    }
    
    &__actions {
        @include flex-center;
        gap: 0.5rem;
        
        @include respond-to(medium) {
            justify-content: flex-end;
        }
    }
}

.button {
    @extend %button-base;
    
    &--primary {
        @include button-variant($primary-color);
    }
    
    &--secondary {
        @include button-variant($secondary-color);
    }
    
    &--success {
        @include button-variant(#28a745);
    }
    
    &--danger {
        @include button-variant(#dc3545);
    }
    
    &--large {
        padding: 0.75rem 1.5rem;
        font-size: map-get($font-sizes, large);
    }
    
    &--small {
        padding: 0.25rem 0.5rem;
        font-size: map-get($font-sizes, small);
    }
}

// Advanced nesting with parent selector
.navigation {
    background-color: white;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    
    &__list {
        display: flex;
        list-style: none;
        margin: 0;
        padding: 0;
        
        @include respond-to(medium) {
            flex-direction: row;
        }
    }
    
    &__item {
        position: relative;
        
        &:hover &__submenu {
            opacity: 1;
            visibility: visible;
            transform: translateY(0);
        }
    }
    
    &__link {
        display: block;
        padding: 1rem;
        color: #333;
        text-decoration: none;
        transition: all 0.3s ease;
        
        &:hover {
            background-color: #f8f9fa;
            color: $primary-color;
        }
        
        &--active {
            color: $primary-color;
            font-weight: 600;
        }
    }
    
    &__submenu {
        position: absolute;
        top: 100%;
        left: 0;
        min-width: 200px;
        background-color: white;
        box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        opacity: 0;
        visibility: hidden;
        transform: translateY(-10px);
        transition: all 0.3s ease;
        
        .navigation__link {
            padding: 0.75rem 1rem;
            border-bottom: 1px solid #e9ecef;
            
            &:last-child {
                border-bottom: none;
            }
        }
    }
}
```

### PostCSS Advanced Features

```css
/* Future CSS features with PostCSS */
:root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --font-size-base: 1rem;
}

/* Nesting */
.component {
    background-color: white;
    border-radius: 8px;
    
    & .header {
        padding: 1rem;
        border-bottom: 1px solid #e9ecef;
        
        & h2 {
            margin: 0;
            color: var(--primary-color);
        }
    }
    
    & .content {
        padding: 1rem;
        
        & p {
            margin-bottom: 1rem;
            line-height: 1.6;
            
            &:last-child {
                margin-bottom: 0;
            }
        }
    }
}

/* Custom media queries */
@custom-media --small-viewport (max-width: 30em);
@custom-media --medium-viewport (min-width: 30em) and (max-width: 50em);
@custom-media --large-viewport (min-width: 50em);

.responsive-element {
    font-size: 1rem;
    
    @media (--small-viewport) {
        font-size: 0.9rem;
    }
    
    @media (--medium-viewport) {
        font-size: 1.1rem;
    }
    
    @media (--large-viewport) {
        font-size: 1.2rem;
    }
}

/* Custom selectors */
@custom-selector :--heading h1, h2, h3, h4, h5, h6;
@custom-selector :--enter :hover, :focus;

:--heading {
    font-weight: 600;
    margin-bottom: 1rem;
}

.button:--enter {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

/* Color function */
.color-functions {
    background-color: color(var(--primary-color) alpha(0.1));
    border-color: color(var(--primary-color) shade(10%));
    color: color(var(--primary-color) contrast());
}

/* Custom properties sets */
--button-theme: {
    background-color: var(--primary-color);
    color: white;
    border: 1px solid var(--primary-color);
    border-radius: 4px;
    padding: 0.5rem 1rem;
    cursor: pointer;
    transition: all 0.3s ease;
}

.button {
    @apply --button-theme;
}
```

---

## 🚀 Performance Optimization

### Critical CSS Optimization

```css
/* Critical CSS - Above the fold */
html {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    line-height: 1.6;
}

body {
    margin: 0;
    padding: 0;
    color: #333;
    background-color: #ffffff;
}

.header {
    background-color: #ffffff;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    position: sticky;
    top: 0;
    z-index: 1000;
}

.nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    max-width: 1200px;
    margin: 0 auto;
}

.hero {
    height: 60vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    color: white;
}

.hero h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
    font-weight: 700;
}

.hero p {
    font-size: 1.2rem;
    margin-bottom: 2rem;
}

.cta-button {
    display: inline-block;
    padding: 1rem 2rem;
    background-color: #ffffff;
    color: #667eea;
    text-decoration: none;
    border-radius: 6px;
    font-weight: 600;
    transition: transform 0.3s ease;
}

.cta-button:hover {
    transform: translateY(-2px);
}

/* Non-critical CSS - Below the fold */
.features {
    padding: 4rem 2rem;
    max-width: 1200px;
    margin: 0 auto;
}

.features-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 2rem;
    margin-top: 2rem;
}

.feature-card {
    background-color: #ffffff;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
    text-align: center;
}

.footer {
    background-color: #333;
    color: white;
    padding: 2rem;
    text-align: center;
}
```

### CSS Loading Optimization

```html
<!-- Preload critical CSS -->
<link rel="preload" href="critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">

<!-- Async load non-critical CSS -->
<link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="non-critical.css"></noscript>

<!-- Font loading optimization -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
```

### CSS Minification and Compression

```css
/* Before minification */
.button {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background-color: #007bff;
    color: white;
    text-decoration: none;
    border-radius: 6px;
    font-weight: 500;
    transition: all 0.3s ease;
}

.button:hover {
    background-color: #0056b3;
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0, 123, 255, 0.4);
}

/* After minification */
.button{display:inline-block;padding:.75rem 1.5rem;background-color:#007bff;color:#fff;text-decoration:none;border-radius:6px;font-weight:500;transition:all .3s ease}.button:hover{background-color:#0056b3;transform:translateY(-2px);box-shadow:0 5px 15px rgba(0,123,255,.4)}

/* CSS purging - remove unused styles */
/* Only include styles that are actually used in the HTML */
.used-class {
    color: red;
}

/* .unused-class { color: blue; } - This would be removed */
```

---

## 🔧 Modern CSS Features

### CSS Logical Properties

```css
.content {
    /* Instead of margin-left and margin-right */
    margin-inline: 1rem;
    
    /* Instead of margin-top and margin-bottom */
    margin-block: 2rem;
    
    /* Instead of padding-left and padding-right */
    padding-inline: 1rem;
    
    /* Instead of padding-top and padding-bottom */
    padding-block: 1rem;
    
    /* Instead of border-left */
    border-inline-start: 3px solid #007bff;
    
    /* Instead of text-align: left */
    text-align: start;
    
    /* Instead of width */
    inline-size: 100%;
    
    /* Instead of height */
    block-size: 200px;
}

/* Writing mode support */
.vertical-text {
    writing-mode: vertical-rl;
    text-orientation: mixed;
}
```

### CSS Scroll Snap

```css
.scroll-container {
    scroll-snap-type: x mandatory;
    overflow-x: scroll;
    display: flex;
    gap: 1rem;
    padding: 1rem;
}

.scroll-item {
    scroll-snap-align: start;
    flex: 0 0 300px;
    height: 200px;
    background-color: #f0f0f0;
    border-radius: 8px;
}

/* Vertical scroll snap */
.vertical-scroll {
    scroll-snap-type: y mandatory;
    overflow-y: scroll;
    height: 100vh;
}

.section {
    scroll-snap-align: start;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

### CSS Color Functions

```css
.color-functions {
    /* HSL with alpha */
    background-color: hsla(210, 100%, 50%, 0.8);
    
    /* RGB with alpha */
    border-color: rgba(255, 0, 0, 0.5);
    
    /* Color mix */
    color: color-mix(in srgb, blue 30%, red);
    
    /* Relative colors */
    --base-color: #007bff;
    background-color: color(from var(--base-color) srgb r g b / 0.5);
    
    /* Contrast function */
    color: color-contrast(var(--base-color) vs white, black);
}
```

### CSS Math Functions

```css
.math-functions {
    /* Clamp function */
    font-size: clamp(1rem, 2.5vw, 2rem);
    width: clamp(300px, 50%, 800px);
    
    /* Min and max */
    width: min(100%, 600px);
    height: max(200px, 30vh);
    
    /* Calculations */
    margin: calc(100vh - 200px);
    width: calc(100% - 2rem);
    
    /* Trigonometric functions */
    transform: rotate(calc(sin(45deg) * 90deg));
    
    /* Round function */
    margin: round(up, 13px, 5px); /* rounds up to nearest 5px */
}
```

### CSS Nesting

```css
.component {
    background-color: white;
    border-radius: 8px;
    
    .header {
        padding: 1rem;
        
        h2 {
            margin: 0;
            color: #333;
        }
    }
    
    .content {
        padding: 1rem;
        
        p {
            margin-bottom: 1rem;
            
            &:last-child {
                margin-bottom: 0;
            }
        }
        
        a {
            color: #007bff;
            
            &:hover {
                text-decoration: underline;
            }
        }
    }
    
    &:hover {
        box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    }
    
    &.featured {
        border: 2px solid #007bff;
    }
}
```

This comprehensive advanced CSS guide covers cutting-edge techniques and modern features that will help you create sophisticated, performant, and maintainable stylesheets. Keep practicing these concepts to master advanced CSS development!
