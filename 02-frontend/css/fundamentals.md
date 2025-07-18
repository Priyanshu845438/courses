
# 🎨 Basic CSS

## Table of Contents
- [What is CSS?](#what-is-css)
- [CSS Syntax](#css-syntax)
- [Adding CSS to HTML](#adding-css-to-html)
- [CSS Selectors](#css-selectors)
- [Basic Properties](#basic-properties)
- [Colors](#colors)
- [Text Styling](#text-styling)
- [Box Model](#box-model)
- [Layout Basics](#layout-basics)
- [Best Practices](#best-practices)

---

## 🎯 What is CSS?

**CSS (Cascading Style Sheets)** is a stylesheet language used to describe the presentation of HTML documents.

### Key Features:
- **Separation of Concerns**: Structure (HTML) vs Presentation (CSS)
- **Reusability**: One stylesheet for multiple pages
- **Maintainability**: Easy to update site-wide styles
- **Flexibility**: Multiple ways to style elements
- **Responsive Design**: Adapt to different screen sizes

### CSS vs HTML

| HTML | CSS |
|------|-----|
| Structure and content | Styling and layout |
| `<h1>Title</h1>` | `h1 { color: blue; }` |
| Semantic meaning | Visual presentation |
| What content is | How content looks |

---

## 📝 CSS Syntax

### Basic CSS Rule

```css
selector {
    property: value;
    property: value;
}
```

### Example:
```css
h1 {
    color: blue;
    font-size: 24px;
    text-align: center;
}
```

### CSS Comments
```css
/* This is a single-line comment */

/*
This is a
multi-line comment
*/

h1 {
    color: red; /* Inline comment */
}
```

---

## 🔗 Adding CSS to HTML

### 1. Inline CSS
```html
<h1 style="color: blue; font-size: 24px;">Inline Styled Heading</h1>
<p style="background-color: yellow;">Inline styled paragraph</p>
```

### 2. Internal CSS
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        h1 {
            color: green;
            text-align: center;
        }
        p {
            color: red;
            font-family: Arial;
        }
    </style>
</head>
<body>
    <h1>Internal CSS Example</h1>
    <p>This paragraph uses internal CSS.</p>
</body>
</html>
```

### 3. External CSS
```html
<!-- HTML file -->
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>External CSS Example</h1>
    <p>This uses external CSS.</p>
</body>
</html>
```

```css
/* styles.css */
h1 {
    color: purple;
    font-size: 28px;
}

p {
    color: darkblue;
    line-height: 1.6;
}
```

---

## 🎯 CSS Selectors

### Basic Selectors

| Selector | Syntax | Example | Selects |
|----------|--------|---------|---------|
| **Element** | `element` | `p` | All `<p>` elements |
| **Class** | `.class` | `.highlight` | Elements with `class="highlight"` |
| **ID** | `#id` | `#header` | Element with `id="header"` |
| **Universal** | `*` | `*` | All elements |

### Example Usage:
```html
<div id="header">Header</div>
<p class="highlight">Highlighted paragraph</p>
<p>Regular paragraph</p>
<span class="highlight">Highlighted span</span>
```

```css
/* Element selector */
p {
    color: black;
}

/* Class selector */
.highlight {
    background-color: yellow;
    font-weight: bold;
}

/* ID selector */
#header {
    background-color: blue;
    color: white;
    padding: 20px;
}

/* Universal selector */
* {
    margin: 0;
    padding: 0;
}
```

### Combining Selectors

```css
/* Multiple selectors */
h1, h2, h3 {
    color: darkblue;
}

/* Descendant selector */
div p {
    font-size: 16px;
}

/* Child selector */
div > p {
    margin-bottom: 10px;
}

/* Class and element */
p.highlight {
    background-color: lightyellow;
}
```

---

## 🎨 Colors

### Color Formats

```css
/* Named colors */
h1 {
    color: red;
    background-color: lightblue;
}

/* Hexadecimal */
h2 {
    color: #ff0000;        /* Red */
    background-color: #00ff00; /* Green */
}

/* RGB */
h3 {
    color: rgb(255, 0, 0);           /* Red */
    background-color: rgb(0, 255, 0); /* Green */
}

/* RGBA (with transparency) */
h4 {
    color: rgba(255, 0, 0, 0.5);     /* Semi-transparent red */
    background-color: rgba(0, 0, 255, 0.3); /* Semi-transparent blue */
}

/* HSL */
h5 {
    color: hsl(0, 100%, 50%);        /* Red */
    background-color: hsl(120, 100%, 50%); /* Green */
}
```

### Color Examples:
```html
<div class="color-examples">
    <div class="red">Red Background</div>
    <div class="green">Green Background</div>
    <div class="blue">Blue Background</div>
    <div class="gradient">Gradient Background</div>
</div>
```

```css
.color-examples div {
    padding: 20px;
    margin: 10px;
    color: white;
    text-align: center;
}

.red {
    background-color: #e74c3c;
}

.green {
    background-color: #27ae60;
}

.blue {
    background-color: #3498db;
}

.gradient {
    background: linear-gradient(45deg, #e74c3c, #3498db);
}
```

---

## ✏️ Text Styling

### Font Properties

```css
/* Font family */
h1 {
    font-family: Arial, sans-serif;
}

h2 {
    font-family: "Times New Roman", serif;
}

h3 {
    font-family: "Courier New", monospace;
}

/* Font size */
.small {
    font-size: 12px;
}

.medium {
    font-size: 16px;
}

.large {
    font-size: 24px;
}

.responsive {
    font-size: 2em;    /* Relative to parent */
    font-size: 1.5rem; /* Relative to root */
}

/* Font weight */
.thin {
    font-weight: 100;
}

.normal {
    font-weight: normal; /* 400 */
}

.bold {
    font-weight: bold;   /* 700 */
}

.extra-bold {
    font-weight: 900;
}

/* Font style */
.italic {
    font-style: italic;
}

.oblique {
    font-style: oblique;
}
```

### Text Properties

```css
/* Text alignment */
.left {
    text-align: left;
}

.center {
    text-align: center;
}

.right {
    text-align: right;
}

.justify {
    text-align: justify;
}

/* Text decoration */
.underline {
    text-decoration: underline;
}

.line-through {
    text-decoration: line-through;
}

.no-decoration {
    text-decoration: none;
}

/* Text transform */
.uppercase {
    text-transform: uppercase;
}

.lowercase {
    text-transform: lowercase;
}

.capitalize {
    text-transform: capitalize;
}

/* Line height */
.tight {
    line-height: 1.2;
}

.normal {
    line-height: 1.5;
}

.loose {
    line-height: 2;
}

/* Letter spacing */
.spaced {
    letter-spacing: 2px;
}

.tight-letters {
    letter-spacing: -1px;
}
```

---

## 📦 Box Model

### Understanding the Box Model

```
┌─────────────────────────────────────┐
│              Margin                 │
│  ┌─────────────────────────────────┐ │
│  │           Border                │ │
│  │  ┌─────────────────────────────┐ │ │
│  │  │          Padding            │ │ │
│  │  │  ┌─────────────────────────┐ │ │ │
│  │  │  │        Content          │ │ │ │
│  │  │  └─────────────────────────┘ │ │ │
│  │  └─────────────────────────────┘ │ │
│  └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

### Box Model Properties

```css
.box {
    /* Content dimensions */
    width: 200px;
    height: 100px;
    
    /* Padding (space inside border) */
    padding: 20px;
    padding-top: 10px;
    padding-right: 15px;
    padding-bottom: 10px;
    padding-left: 15px;
    
    /* Border */
    border: 2px solid black;
    border-width: 2px;
    border-style: solid;
    border-color: black;
    
    /* Margin (space outside border) */
    margin: 10px;
    margin-top: 5px;
    margin-right: 10px;
    margin-bottom: 5px;
    margin-left: 10px;
}

/* Shorthand properties */
.shorthand {
    /* All sides */
    padding: 20px;
    margin: 10px;
    
    /* Top/bottom and left/right */
    padding: 20px 15px;
    margin: 10px 5px;
    
    /* Top, right, bottom, left */
    padding: 10px 15px 20px 25px;
    margin: 5px 10px 15px 20px;
}
```

### Box Sizing

```css
/* Default box-sizing */
.content-box {
    box-sizing: content-box;
    width: 200px;
    padding: 20px;
    border: 2px solid black;
    /* Total width = 200px + 40px + 4px = 244px */
}

/* Border-box sizing */
.border-box {
    box-sizing: border-box;
    width: 200px;
    padding: 20px;
    border: 2px solid black;
    /* Total width = 200px (includes padding and border) */
}

/* Apply to all elements */
* {
    box-sizing: border-box;
}
```

---

## 📐 Layout Basics

### Display Property

```css
/* Block elements */
.block {
    display: block;
    width: 100%;
    background-color: lightblue;
}

/* Inline elements */
.inline {
    display: inline;
    background-color: lightgreen;
}

/* Inline-block elements */
.inline-block {
    display: inline-block;
    width: 100px;
    height: 50px;
    background-color: lightcoral;
}

/* Hide elements */
.hidden {
    display: none;
}
```

### Position Property

```css
/* Static positioning (default) */
.static {
    position: static;
}

/* Relative positioning */
.relative {
    position: relative;
    top: 20px;
    left: 30px;
}

/* Absolute positioning */
.absolute {
    position: absolute;
    top: 50px;
    right: 20px;
}

/* Fixed positioning */
.fixed {
    position: fixed;
    bottom: 0;
    right: 0;
}

/* Sticky positioning */
.sticky {
    position: sticky;
    top: 0;
}
```

### Float Property

```css
/* Float elements */
.float-left {
    float: left;
    width: 200px;
    margin-right: 20px;
}

.float-right {
    float: right;
    width: 200px;
    margin-left: 20px;
}

/* Clear floats */
.clear {
    clear: both;
}

/* Clearfix */
.clearfix::after {
    content: "";
    display: table;
    clear: both;
}
```

---

## 🎯 Basic Properties

### Width and Height

```css
/* Fixed dimensions */
.fixed-size {
    width: 300px;
    height: 200px;
}

/* Percentage dimensions */
.percentage {
    width: 50%;
    height: 80%;
}

/* Max and min dimensions */
.responsive {
    width: 100%;
    max-width: 800px;
    min-width: 300px;
    height: auto;
    min-height: 200px;
}
```

### Background Properties

```css
/* Background color */
.bg-color {
    background-color: #f0f0f0;
}

/* Background image */
.bg-image {
    background-image: url('image.jpg');
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
}

/* Background shorthand */
.bg-shorthand {
    background: #f0f0f0 url('image.jpg') no-repeat center/cover;
}

/* Multiple backgrounds */
.multiple-bg {
    background: 
        url('overlay.png') no-repeat center,
        url('background.jpg') no-repeat center/cover;
}
```

### Border Properties

```css
/* Border styles */
.solid-border {
    border: 2px solid black;
}

.dashed-border {
    border: 3px dashed red;
}

.dotted-border {
    border: 1px dotted blue;
}

/* Individual borders */
.custom-borders {
    border-top: 2px solid red;
    border-right: 3px dashed blue;
    border-bottom: 1px dotted green;
    border-left: 4px solid purple;
}

/* Border radius */
.rounded {
    border-radius: 10px;
}

.circle {
    border-radius: 50%;
}

.custom-radius {
    border-radius: 10px 20px 30px 40px;
}
```

---

## 💡 Best Practices

### CSS Organization

```css
/* 1. Reset/Normalize */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* 2. Typography */
body {
    font-family: Arial, sans-serif;
    font-size: 16px;
    line-height: 1.6;
    color: #333;
}

h1, h2, h3, h4, h5, h6 {
    font-weight: bold;
    margin-bottom: 1rem;
}

/* 3. Layout */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* 4. Components */
.button {
    display: inline-block;
    padding: 10px 20px;
    background-color: #007bff;
    color: white;
    text-decoration: none;
    border-radius: 5px;
    transition: background-color 0.3s;
}

.button:hover {
    background-color: #0056b3;
}

/* 5. Utilities */
.text-center {
    text-align: center;
}

.margin-bottom {
    margin-bottom: 1rem;
}

.hidden {
    display: none;
}
```

### Naming Conventions

```css
/* Use meaningful class names */
.navigation-menu { }
.article-title { }
.contact-form { }

/* Use hyphens for multi-word names */
.main-header { }
.sidebar-widget { }
.footer-links { }

/* Use consistent prefixes */
.btn-primary { }
.btn-secondary { }
.btn-danger { }

.card-header { }
.card-body { }
.card-footer { }
```

### Complete Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Basic Example</title>
    <style>
        /* Reset and base styles */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            color: #333;
            background-color: #f4f4f4;
        }

        /* Container */
        .container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        /* Header */
        .header {
            text-align: center;
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 2px solid #007bff;
        }

        .header h1 {
            color: #007bff;
            font-size: 2.5em;
            margin-bottom: 10px;
        }

        .header p {
            color: #666;
            font-size: 1.1em;
        }

        /* Navigation */
        .nav {
            text-align: center;
            margin-bottom: 30px;
        }

        .nav a {
            display: inline-block;
            margin: 0 15px;
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            text-decoration: none;
            border-radius: 5px;
            transition: background-color 0.3s;
        }

        .nav a:hover {
            background-color: #0056b3;
        }

        /* Content sections */
        .section {
            margin-bottom: 30px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 5px;
        }

        .section h2 {
            color: #007bff;
            margin-bottom: 15px;
        }

        .section p {
            margin-bottom: 10px;
        }

        /* Highlighted text */
        .highlight {
            background-color: yellow;
            padding: 2px 4px;
            border-radius: 3px;
        }

        /* Lists */
        .feature-list {
            list-style: none;
            padding-left: 0;
        }

        .feature-list li {
            margin-bottom: 10px;
            padding-left: 20px;
            position: relative;
        }

        .feature-list li:before {
            content: "✓";
            position: absolute;
            left: 0;
            color: #28a745;
            font-weight: bold;
        }

        /* Footer */
        .footer {
            text-align: center;
            padding-top: 20px;
            border-top: 1px solid #dee2e6;
            color: #6c757d;
        }
    </style>
</head>
<body>
    <div class="container">
        <header class="header">
            <h1>Welcome to CSS Basics</h1>
            <p>Learn styling with practical examples</p>
        </header>

        <nav class="nav">
            <a href="#basics">Basics</a>
            <a href="#styling">Styling</a>
            <a href="#layout">Layout</a>
            <a href="#examples">Examples</a>
        </nav>

        <main>
            <section class="section" id="basics">
                <h2>CSS Basics</h2>
                <p>CSS is used to style HTML elements. It controls the <span class="highlight">visual presentation</span> of web pages.</p>
                <p>Key concepts include selectors, properties, and values.</p>
            </section>

            <section class="section" id="styling">
                <h2>Text and Color Styling</h2>
                <p>You can style text in many ways:</p>
                <ul class="feature-list">
                    <li>Change font family and size</li>
                    <li>Apply colors and backgrounds</li>
                    <li>Add decorations and effects</li>
                    <li>Control spacing and alignment</li>
                </ul>
            </section>

            <section class="section" id="layout">
                <h2>Layout and Positioning</h2>
                <p>CSS provides several methods for layout:</p>
                <ul class="feature-list">
                    <li>Box model for spacing</li>
                    <li>Display properties</li>
                    <li>Positioning elements</li>
                    <li>Float and clear</li>
                </ul>
            </section>
        </main>

        <footer class="footer">
            <p>&copy; 2024 CSS Tutorial. Keep practicing to master CSS!</p>
        </footer>
    </div>
</body>
</html>
```

## 📚 Quick Reference

### Essential Properties

| Property | Description | Example |
|----------|-------------|---------|
| `color` | Text color | `color: red;` |
| `background-color` | Background color | `background-color: blue;` |
| `font-size` | Text size | `font-size: 16px;` |
| `font-family` | Font type | `font-family: Arial;` |
| `width` | Element width | `width: 200px;` |
| `height` | Element height | `height: 100px;` |
| `margin` | Outside spacing | `margin: 10px;` |
| `padding` | Inside spacing | `padding: 15px;` |
| `border` | Element border | `border: 1px solid black;` |
| `text-align` | Text alignment | `text-align: center;` |

### Common Values

- **Colors**: `red`, `#ff0000`, `rgb(255,0,0)`
- **Sizes**: `px`, `em`, `rem`, `%`, `vh`, `vw`
- **Positions**: `static`, `relative`, `absolute`, `fixed`
- **Display**: `block`, `inline`, `inline-block`, `none`

### Next Steps:
1. Learn CSS Grid and Flexbox
2. Study responsive design
3. Explore CSS animations
4. Practice with real projects
5. Learn CSS preprocessors (Sass/Less)
# 🎨 CSS Fundamentals

## Table of Contents
- [CSS Syntax and Selectors](#css-syntax-and-selectors)
- [Box Model](#box-model)
- [Layout Systems](#layout-systems)
- [Typography and Colors](#typography-and-colors)
- [Responsive Design](#responsive-design)
- [CSS Units](#css-units)
- [Positioning](#positioning)
- [Flexbox](#flexbox)
- [CSS Grid](#css-grid)
- [Transitions and Animations](#transitions-and-animations)

---

## 🎯 CSS Syntax and Selectors

### Basic Syntax

```css
/* CSS Rule Structure */
selector {
    property: value;
    property: value;
}

/* Example */
h1 {
    color: blue;
    font-size: 24px;
    margin: 16px 0;
}
```

### CSS Selectors

```css
/* Element Selector */
p {
    color: black;
}

/* Class Selector */
.highlight {
    background-color: yellow;
}

/* ID Selector */
#header {
    background-color: navy;
}

/* Attribute Selector */
input[type="text"] {
    border: 1px solid gray;
}

/* Descendant Selector */
nav ul li {
    list-style: none;
}

/* Child Selector */
nav > ul {
    margin: 0;
}

/* Pseudo-classes */
a:hover {
    color: red;
}

button:focus {
    outline: 2px solid blue;
}

li:nth-child(odd) {
    background-color: #f0f0f0;
}

/* Pseudo-elements */
p::first-line {
    font-weight: bold;
}

h2::before {
    content: "★ ";
    color: gold;
}
```

---

## 📦 Box Model

### Understanding the Box Model

```css
.box {
    /* Content area */
    width: 200px;
    height: 100px;
    
    /* Padding (inside the border) */
    padding: 20px;
    
    /* Border */
    border: 2px solid black;
    
    /* Margin (outside the border) */
    margin: 10px;
    
    /* Box-sizing property */
    box-sizing: border-box; /* Includes padding and border in width/height */
}

/* Visual representation:
┌─────────────────── margin ──────────────────┐
│ ┌─────────────── border ─────────────────┐ │
│ │ ┌─────────── padding ─────────────┐ │ │
│ │ │ ┌─── content (width/height) ─┐ │ │ │
│ │ │ │                             │ │ │ │
│ │ │ └─────────────────────────────┘ │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
*/
```

### Box Model Properties

```css
.element {
    /* Margin shortcuts */
    margin: 10px;                    /* All sides */
    margin: 10px 20px;              /* Top/bottom, left/right */
    margin: 10px 20px 15px;         /* Top, left/right, bottom */
    margin: 10px 20px 15px 25px;    /* Top, right, bottom, left */
    
    /* Individual margins */
    margin-top: 10px;
    margin-right: 20px;
    margin-bottom: 15px;
    margin-left: 25px;
    
    /* Padding (same pattern as margin) */
    padding: 15px;
    
    /* Border */
    border: 1px solid black;
    border-width: 2px;
    border-style: solid;
    border-color: red;
    border-radius: 5px;
}
```

---

## 🏗️ Layout Systems

### Normal Flow

```css
/* Block elements */
div, p, h1, section {
    display: block;        /* Takes full width, stacks vertically */
}

/* Inline elements */
span, a, strong, em {
    display: inline;       /* Takes only needed width, flows horizontally */
}

/* Inline-block elements */
.inline-block {
    display: inline-block; /* Inline flow but can have width/height */
    width: 100px;
    height: 50px;
}
```

### Display Property

```css
.hidden {
    display: none;         /* Removes element from layout */
}

.invisible {
    visibility: hidden;    /* Hides element but keeps space */
}

.flex-container {
    display: flex;         /* Flexbox layout */
}

.grid-container {
    display: grid;         /* Grid layout */
}
```

---

## 🎨 Typography and Colors

### Font Properties

```css
body {
    font-family: 'Helvetica Neue', Arial, sans-serif;
    font-size: 16px;
    font-weight: 400;      /* normal, bold, 100-900 */
    font-style: normal;    /* normal, italic, oblique */
    line-height: 1.5;      /* Unitless preferred */
    letter-spacing: 0.02em;
    text-transform: none;   /* uppercase, lowercase, capitalize */
}

h1 {
    font-size: 2.5rem;     /* Relative to root font size */
    font-weight: 700;
    line-height: 1.2;
}

.small-text {
    font-size: 0.875rem;  /* 14px if root is 16px */
}
```

### Color Properties

```css
.colors {
    /* Named colors */
    color: red;
    
    /* Hex colors */
    background-color: #ff0000;      /* Red */
    border-color: #f00;             /* Short hex */
    
    /* RGB/RGBA */
    color: rgb(255, 0, 0);          /* Red */
    background-color: rgba(255, 0, 0, 0.5);  /* Semi-transparent red */
    
    /* HSL/HSLA */
    color: hsl(0, 100%, 50%);       /* Red */
    background-color: hsla(0, 100%, 50%, 0.5);
    
    /* CSS Variables */
    color: var(--primary-color, blue);  /* With fallback */
}

:root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
}
```

---

## 📱 Responsive Design

### Media Queries

```css
/* Mobile First Approach */
.container {
    width: 100%;
    padding: 16px;
}

/* Tablet */
@media (min-width: 768px) {
    .container {
        max-width: 750px;
        margin: 0 auto;
        padding: 24px;
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .container {
        max-width: 1200px;
        padding: 32px;
    }
}

/* Common Breakpoints */
@media (max-width: 767px) { /* Mobile */ }
@media (min-width: 768px) and (max-width: 1023px) { /* Tablet */ }
@media (min-width: 1024px) { /* Desktop */ }
```

### Flexible Images

```css
img {
    max-width: 100%;
    height: auto;
}

.responsive-video {
    position: relative;
    padding-bottom: 56.25%; /* 16:9 aspect ratio */
    height: 0;
    overflow: hidden;
}

.responsive-video iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
```

---

## 📏 CSS Units

### Absolute Units

```css
.absolute-units {
    width: 300px;          /* Pixels */
    height: 2in;           /* Inches */
    margin: 1cm;           /* Centimeters */
    padding: 10mm;         /* Millimeters */
    border-width: 1pt;     /* Points */
}
```

### Relative Units

```css
.relative-units {
    /* Relative to font size */
    width: 20em;           /* Relative to element's font size */
    height: 15rem;         /* Relative to root font size */
    
    /* Relative to viewport */
    width: 50vw;           /* 50% of viewport width */
    height: 100vh;         /* 100% of viewport height */
    font-size: 4vmin;      /* 4% of viewport's smaller dimension */
    
    /* Relative to parent */
    width: 80%;            /* 80% of parent's width */
}
```

---

## 🎯 Positioning

### Position Property

```css
.static {
    position: static;      /* Default - normal flow */
}

.relative {
    position: relative;    /* Relative to normal position */
    top: 10px;
    left: 20px;
}

.absolute {
    position: absolute;    /* Relative to nearest positioned ancestor */
    top: 0;
    right: 0;
}

.fixed {
    position: fixed;       /* Relative to viewport */
    bottom: 20px;
    right: 20px;
}

.sticky {
    position: sticky;      /* Switches between relative and fixed */
    top: 0;               /* Sticks when this distance from top */
}
```

### Z-index

```css
.layer-1 {
    position: relative;
    z-index: 1;
}

.layer-2 {
    position: absolute;
    z-index: 10;          /* Higher values appear on top */
}

.modal {
    position: fixed;
    z-index: 1000;        /* High value for modals */
}
```

---

## 🔧 Flexbox

### Flex Container

```css
.flex-container {
    display: flex;
    
    /* Direction */
    flex-direction: row;        /* row, row-reverse, column, column-reverse */
    
    /* Wrapping */
    flex-wrap: nowrap;          /* nowrap, wrap, wrap-reverse */
    
    /* Shorthand */
    flex-flow: row wrap;        /* flex-direction + flex-wrap */
    
    /* Alignment */
    justify-content: flex-start; /* flex-start, flex-end, center, space-between, space-around, space-evenly */
    align-items: stretch;        /* stretch, flex-start, flex-end, center, baseline */
    align-content: flex-start;   /* For wrapped lines */
}
```

### Flex Items

```css
.flex-item {
    /* Flexibility */
    flex-grow: 1;          /* How much to grow */
    flex-shrink: 1;        /* How much to shrink */
    flex-basis: auto;      /* Initial size */
    
    /* Shorthand */
    flex: 1 1 auto;        /* grow shrink basis */
    flex: 1;               /* Common: grow=1, shrink=1, basis=0 */
    
    /* Individual alignment */
    align-self: center;    /* Overrides align-items for this item */
}
```

### Common Flexbox Patterns

```css
/* Center content */
.center {
    display: flex;
    justify-content: center;
    align-items: center;
}

/* Equal width columns */
.equal-columns {
    display: flex;
}

.equal-columns > * {
    flex: 1;
}

/* Sidebar layout */
.sidebar-layout {
    display: flex;
}

.sidebar {
    flex: 0 0 250px;       /* Fixed width sidebar */
}

.main-content {
    flex: 1;               /* Takes remaining space */
}
```

---

## 🔲 CSS Grid

### Grid Container

```css
.grid-container {
    display: grid;
    
    /* Columns and rows */
    grid-template-columns: 200px 1fr 100px;    /* Fixed, flexible, fixed */
    grid-template-rows: auto 1fr auto;          /* Header, content, footer */
    
    /* Alternative syntax */
    grid-template-columns: repeat(3, 1fr);      /* Three equal columns */
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Responsive */
    
    /* Gaps */
    grid-gap: 20px;                   /* Row and column gap */
    grid-row-gap: 20px;              /* Row gap only */
    grid-column-gap: 15px;           /* Column gap only */
    
    /* Areas */
    grid-template-areas: 
        "header header header"
        "sidebar main main"
        "footer footer footer";
}
```

### Grid Items

```css
.grid-item {
    /* Position by line numbers */
    grid-column: 1 / 3;        /* From line 1 to line 3 */
    grid-row: 2 / 4;
    
    /* Position by area */
    grid-area: header;
    
    /* Span multiple cells */
    grid-column: span 2;       /* Span 2 columns */
    grid-row: span 3;          /* Span 3 rows */
}

/* Named areas */
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### Responsive Grid

```css
.responsive-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    grid-gap: 20px;
}

/* Card layout */
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    grid-auto-rows: 200px;
    grid-gap: 16px;
}
```

---

## ✨ Transitions and Animations

### CSS Transitions

```css
.button {
    background-color: blue;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    
    /* Transition */
    transition: background-color 0.3s ease;
}

.button:hover {
    background-color: darkblue;
}

/* Multiple properties */
.card {
    transform: scale(1);
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    
    transition: 
        transform 0.3s ease,
        box-shadow 0.3s ease;
}

.card:hover {
    transform: scale(1.05);
    box-shadow: 0 8px 16px rgba(0,0,0,0.2);
}
```

### CSS Animations

```css
/* Define keyframes */
@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}

/* Apply animation */
.fade-in {
    animation: fadeIn 0.5s ease-out;
}

.loading-spinner {
    animation: spin 2s linear infinite;
}

/* Animation properties */
.animated-element {
    animation-name: fadeIn;
    animation-duration: 1s;
    animation-timing-function: ease-in-out;
    animation-delay: 0.5s;
    animation-iteration-count: 1;        /* or infinite */
    animation-direction: normal;         /* normal, reverse, alternate */
    animation-fill-mode: forwards;       /* none, forwards, backwards, both */
    
    /* Shorthand */
    animation: fadeIn 1s ease-in-out 0.5s forwards;
}
```

---

## 🎯 Key Concepts Summary

### CSS Fundamentals Checklist:
- ✅ Understand CSS syntax and selectors
- ✅ Master the box model
- ✅ Learn display and positioning
- ✅ Practice flexbox for 1D layouts
- ✅ Practice CSS Grid for 2D layouts
- ✅ Implement responsive design
- ✅ Work with typography and colors
- ✅ Add transitions and animations

### Best Practices:
1. **Use semantic class names** (`.button-primary` not `.blue-button`)
2. **Follow mobile-first approach** for responsive design
3. **Use CSS custom properties** for consistent theming
4. **Optimize for performance** (avoid expensive properties)
5. **Keep specificity low** for maintainable CSS
6. **Use flexbox for components**, Grid for layouts
7. **Test across different browsers** and devices

### Next Steps:
1. Practice building layouts with Flexbox and Grid
2. Learn CSS preprocessors (Sass, Less)
3. Explore CSS frameworks (Bootstrap, Tailwind)
4. Study CSS-in-JS solutions
5. Learn about CSS architecture patterns

This comprehensive guide covers the fundamental concepts of CSS that every web developer should master.
