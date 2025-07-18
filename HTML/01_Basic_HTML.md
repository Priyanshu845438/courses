# 🌐 Basic HTML

## Table of Contents
- [What is HTML?](#what-is-html)
- [HTML Document Structure](#html-document-structure)
- [Basic HTML Tags](#basic-html-tags)
- [Text Formatting](#text-formatting)
- [Links and Navigation](#links-and-navigation)
- [Images and Media](#images-and-media)
- [Lists](#lists)
- [Tables](#tables)
- [Forms](#forms)
- [Best Practices](#best-practices)

---

## 🔍 What is HTML?

**HTML (HyperText Markup Language)** is the standard markup language for creating web pages and web applications.

### Key Features:
- **Structure**: Defines the structure and content of web pages
- **Markup**: Uses tags to mark up content
- **Platform Independent**: Works on all browsers and devices
- **Easy to Learn**: Simple syntax and rules
- **Foundation**: Base for all web development

### HTML vs Other Languages

| Language | Purpose | Type | Example |
|----------|---------|------|---------|
| **HTML** | Structure | Markup | `<h1>Title</h1>` |
| **CSS** | Styling | Stylesheet | `color: blue;` |
| **JavaScript** | Behavior | Programming | `alert('Hello');` |
| **PHP** | Server-side | Programming | `<?php echo "Hi"; ?>` |

---

## 📄 HTML Document Structure

### Basic HTML Document Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Web Page</title>
</head>
<body>
    <h1>Welcome to My Website!</h1>
    <p>This is my first HTML page.</p>
</body>
</html>
```

### Document Structure Breakdown

```
<!DOCTYPE html>                    ← Document type declaration
<html>                            ← Root element
  ├── <head>                      ← Document metadata
  │   ├── <meta>                  ← Character encoding
  │   ├── <title>                 ← Page title
  │   └── <link>                  ← External resources
  └── <body>                      ← Visible content
      ├── <header>                ← Page header
      ├── <main>                  ← Main content
      └── <footer>                ← Page footer
```

### Essential Meta Tags

```html
<head>
    <!-- Character encoding -->
    <meta charset="UTF-8">

    <!-- Responsive design -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Page description -->
    <meta name="description" content="Learn HTML basics with examples">

    <!-- Keywords for SEO -->
    <meta name="keywords" content="HTML, web development, tutorial">

    <!-- Author information -->
    <meta name="author" content="Your Name">

    <!-- Page title -->
    <title>Basic HTML Tutorial</title>
</head>
```

---

## 🏷️ Basic HTML Tags

### Heading Tags

```html
<h1>Main Heading (Largest)</h1>
<h2>Section Heading</h2>
<h3>Subsection Heading</h3>
<h4>Sub-subsection Heading</h4>
<h5>Minor Heading</h5>
<h6>Smallest Heading</h6>
```

### Paragraph and Text Tags

```html
<!-- Paragraphs -->
<p>This is a paragraph of text. It will wrap automatically when it reaches the edge of its container.</p>

<p>This is another paragraph. Notice the spacing between paragraphs.</p>

<!-- Line break -->
<p>First line<br>Second line</p>

<!-- Horizontal rule -->
<hr>

<!-- Preformatted text -->
<pre>
    This text preserves
    spaces and    line breaks
    exactly as typed.
</pre>

<!-- Blockquote -->
<blockquote>
    "The best way to learn HTML is to practice coding every day."
    <cite>- Web Developer</cite>
</blockquote>
```

### Semantic HTML5 Elements

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Semantic HTML Example</title>
</head>
<body>
    <header>
        <h1>Website Header</h1>
        <nav>
            <ul>
                <li><a href="#home">Home</a></li>
                <li><a href="#about">About</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <h2>Article Title</h2>
            <p>Article content goes here...</p>
        </article>

        <aside>
            <h3>Related Links</h3>
            <p>Sidebar content...</p>
        </aside>
    </main>

    <footer>
        <p>&copy; 2024 My Website. All rights reserved.</p>
    </footer>
</body>
</html>
```

---

## ✏️ Text Formatting

### Basic Text Formatting

```html
<!-- Bold text -->
<b>Bold text</b> or <strong>Strong importance</strong>

<!-- Italic text -->
<i>Italic text</i> or <em>Emphasized text</em>

<!-- Underlined text -->
<u>Underlined text</u>

<!-- Strikethrough -->
<s>Strikethrough text</s> or <del>Deleted text</del>

<!-- Superscript and Subscript -->
<p>E = mc<sup>2</sup></p>
<p>H<sub>2</sub>O</p>

<!-- Highlighted text -->
<mark>Highlighted text</mark>

<!-- Small text -->
<small>Small print text</small>

<!-- Code -->
<code>console.log("Hello World");</code>

<!-- Keyboard input -->
<kbd>Ctrl + C</kbd>

<!-- Sample output -->
<samp>Error 404: File not found</samp>
```

### Advanced Text Elements

```html
<!-- Abbreviation -->
<p>The <abbr title="World Wide Web">WWW</abbr> was invented by Tim Berners-Lee.</p>

<!-- Address -->
<address>
    John Doe<br>
    123 Main Street<br>
    Anytown, USA 12345<br>
    <a href="mailto:john@example.com">john@example.com</a>
</address>

<!-- Definition -->
<p><dfn>HTML</dfn> stands for HyperText Markup Language.</p>

<!-- Time -->
<p>The meeting is scheduled for <time datetime="2024-01-15T14:00">January 15, 2024 at 2:00 PM</time>.</p>

<!-- Progress bar -->
<p>Download progress:</p>
<progress value="70" max="100">70%</progress>

<!-- Details and Summary -->
<details>
    <summary>Click to expand</summary>
    <p>This content is hidden by default and shown when clicked.</p>
</details>
```

---

## 🔗 Links and Navigation

### Basic Links

```html
<!-- External link -->
<a href="https://www.google.com">Visit Google</a>

<!-- Internal link (same page) -->
<a href="#section1">Go to Section 1</a>

<!-- Link to another page -->
<a href="about.html">About Us</a>

<!-- Email link -->
<a href="mailto:contact@example.com">Send Email</a>

<!-- Phone link -->
<a href="tel:+1234567890">Call Us</a>

<!-- Download link -->
<a href="document.pdf" download>Download PDF</a>

<!-- Link with target -->
<a href="https://www.example.com" target="_blank">Open in New Tab</a>
```

### Navigation Menu Example

```html
<nav>
    <ul>
        <li><a href="index.html">Home</a></li>
        <li><a href="products.html">Products</a>
            <ul>
                <li><a href="laptops.html">Laptops</a></li>
                <li><a href="phones.html">Phones</a></li>
                <li><a href="tablets.html">Tablets</a></li>
            </ul>
        </li>
        <li><a href="services.html">Services</a></li>
        <li><a href="contact.html">Contact</a></li>
    </ul>
</nav>
```

---

## 🖼️ Images and Media

### Images

```html
<!-- Basic image -->
<img src="photo.jpg" alt="A beautiful sunset">

<!-- Image with dimensions -->
<img src="logo.png" alt="Company Logo" width="200" height="100">

<!-- Responsive image -->
<img src="large-image.jpg" alt="Description" style="max-width: 100%; height: auto;">

<!-- Image with figure caption -->
<figure>
    <img src="chart.png" alt="Sales Chart">
    <figcaption>Annual Sales Report for 2024</figcaption>
</figure>
```

### Audio and Video

```html
<!-- Audio element -->
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    Your browser does not support the audio element.
</audio>

<!-- Video element -->
<video controls width="640" height="480">
    <source src="movie.mp4" type="video/mp4">
    <source src="movie.webm" type="video/webm">
    Your browser does not support the video tag.
</video>

<!-- Embedded content -->
<iframe width="560" height="315" 
        src="https://www.youtube.com/embed/VIDEO_ID" 
        frameborder="0" 
        allowfullscreen>
</iframe>
```

---

## 📝 Lists

### Unordered Lists

```html
<!-- Basic unordered list -->
<ul>
    <li>Apple</li>
    <li>Banana</li>
    <li>Orange</li>
</ul>

<!-- Nested unordered list -->
<ul>
    <li>Fruits
        <ul>
            <li>Apple</li>
            <li>Banana</li>
        </ul>
    </li>
    <li>Vegetables
        <ul>
            <li>Carrot</li>
            <li>Broccoli</li>
        </ul>
    </li>
</ul>
```

### Ordered Lists

```html
<!-- Basic ordered list -->
<ol>
    <li>First step</li>
    <li>Second step</li>
    <li>Third step</li>
</ol>

<!-- Different numbering types -->
<ol type="A">
    <li>Item A</li>
    <li>Item B</li>
</ol>

<ol type="i">
    <li>Roman numeral i</li>
    <li>Roman numeral ii</li>
</ol>
```

### Description Lists

```html
<dl>
    <dt>HTML</dt>
    <dd>HyperText Markup Language - used for structuring web content</dd>

    <dt>CSS</dt>
    <dd>Cascading Style---

## 📊 Tables

### Basic Table

```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Age</th>
            <th>City</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John Doe</td>
            <td>30</td>
            <td>New York</td>
        </tr>
        <tr>
            <td>Jane Smith</td>
            <td>25</td>
            <td>Los Angeles</td>
        </tr>
    </tbody>
</table>
```

### Advanced Table Features

```html
<table border="1" style="border-collapse: collapse; width: 100%;">
    <caption>Employee Information</caption>
    <thead>
        <tr>
            <th rowspan="2">Name</th>
            <th colspan="2">Contact</th>
            <th rowspan="2">Department</th>
        </tr>
        <tr>
            <th>Email</th>
            <th>Phone</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John Doe</td>
            <td>john@company.com</td>
            <td>555-0123</td>
            <td>IT</td>
        </tr>
        <tr>
            <td>Jane Smith</td>
            <td>jane@company.com</td>
            <td>555-0456</td>
            <td>Marketing</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="4">Total Employees: 2</td>
        </tr>
    </tfoot>
</table>
```

---

## 📋 Forms

### Basic Form Elements

```html
<form action="submit.php" method="POST">
    <!-- Text input -->
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>

    <!-- Email input -->
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <!-- Password input -->
    <label for="password">Password:</label>
    <input type="password" id="password" name="password" required>

    <!-- Number input -->
    <label for="age">Age:</label>
    <input type="number" id="age" name="age" min="18" max="100">

    <!-- Date input -->
    <label for="birthday">Birthday:</label>
    <input type="date" id="birthday" name="birthday">

    <!-- Submit button -->
    <input type="submit" value="Submit">
</form>
```

### Advanced Form Elements

```html
<form>
    <!-- Radio buttons -->
    <fieldset>
        <legend>Gender:</legend>
        <input type="radio" id="male" name="gender" value="male">
        <label for="male">Male</label>

        <input type="radio" id="female" name="gender" value="female">
        <label for="female">Female</label>

        <input type="radio" id="other" name="gender" value="other">
        <label for="other">Other</label>
    </fieldset>

    <!-- Checkboxes -->
    <fieldset>
        <legend>Interests:</legend>
        <input type="checkbox" id="sports" name="interests" value="sports">
        <label for="sports">Sports</label>

        <input type="checkbox" id="music" name="interests" value="music">
        <label for="music">Music</label>

        <input type="checkbox" id="travel" name="interests" value="travel">
        <label for="travel">Travel</label>
    </fieldset>

    <!-- Select dropdown -->
    <label for="country">Country:</label>
    <select id="country" name="country">
        <option value="">Select a country</option>
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="uk">United Kingdom</option>
        <option value="au">Australia</option>
    </select>

    <!-- Textarea -->
    <label for="comments">Comments:</label>
    <textarea id="comments" name="comments" rows="4" cols="50" 
              placeholder="Enter your comments here..."></textarea>

    <!-- File upload -->
    <label for="resume">Upload Resume:</label>
    <input type="file" id="resume" name="resume" accept=".pdf,.doc,.docx">

    <!-- Buttons -->
    <button type="submit">Submit Form</button>
    <button type="reset">Reset Form</button>
    <button type="button" onclick="alert('Button clicked!')">Custom Action</button>
</form>
```

---

## 💡 Best Practices

### Accessibility

```html
<!-- Use semantic HTML -->
<main>
    <article>
        <h1>Article Title</h1>
        <p>Article content...</p>
    </article>
</main>

<!-- Provide alt text for images -->
<img src="chart.png" alt="Sales increased 25% from 2023 to 2024">

<!-- Use proper heading hierarchy -->
<h1>Main Title</h1>
    <h2>Section 1</h2>
        <h3>Subsection 1.1</h3>
        <h3>Subsection 1.2</h3>
    <h2>Section 2</h2>

<!-- Label form elements -->
<label for="email">Email Address:</label>
<input type="email" id="email" name="email" required>
```

### SEO Optimization

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Title tag (important for SEO) -->
    <title>HTML Tutorial - Learn Web Development | YourSite</title>

    <!-- Meta description -->
    <meta name="description" content="Learn HTML basics with comprehensive examples and tutorials. Perfect for beginners starting web development.">

    <!-- Open Graph tags for social media -->
    <meta property="og:title" content="HTML Tutorial - Learn Web Development">
    <meta property="og:description" content="Learn HTML basics with comprehensive examples">
    <meta property="og:image" content="https://yoursite.com/html-tutorial-image.jpg">
    <meta property="og:url" content="https://yoursite.com/html-tutorial">

    <!-- Canonical URL -->
    <link rel="canonical" href="https://yoursite.com/html-tutorial">
</head>
<body>
    <!-- Use heading tags properly -->
    <h1>HTML Tutorial</h1>
    <h2>Introduction to HTML</h2>
    <h3>What is HTML?</h3>
</body>
</html>
```

### Complete Example Project

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Personal Portfolio - John Doe</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; }
        header { background: #333; color: white; padding: 1rem; }
        nav ul { list-style: none; padding: 0; }
        nav li { display: inline; margin-right: 20px; }
        nav a { color: white; text-decoration: none; }
        .container { max-width: 800px; margin: 0 auto; }
        .project { border: 1px solid #ddd; padding: 20px; margin: 20px 0; }
        footer { text-align: center; padding: 20px; background: #f5f5f5; }
    </style>
</head>
<body>
    <header>
        <div class="container">
            <h1>John Doe</h1>
            <p>Web Developer & Designer</p>
            <nav>
                <ul>
                    <li><a href="#about">About</a></li>
                    <li><a href="#projects">Projects</a></li>
                    <li><a href="#contact">Contact</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <main class="container">
        <section id="about">
            <h2>About Me</h2>
            <img src="profile.jpg" alt="John Doe Profile Picture" width="200" height="200">
            <p>I'm a passionate web developer with 5 years of experience creating beautiful and functional websites.</p>

            <h3>Skills</h3>
            <ul>
                <li>HTML5 & CSS3</li>
                <li>JavaScript</li>
                <li>React.js</li>
                <li>Node.js</li>
            </ul>
        </section>

        <section id="projects">
            <h2>My Projects</h2>

            <article class="project">
                <h3>E-commerce Website</h3>
                <img src="ecommerce-preview.jpg" alt="E-commerce website preview" width="300" height="200">
                <p>A fully responsive e-commerce platform built with React and Node.js.</p>
                <p><strong>Technologies:</strong> React, Node.js, MongoDB</p>
                <a href="https://github.com/johndoe/ecommerce" target="_blank">View on GitHub</a>
                <a href="https://ecommerce-demo.com" target="_blank">Live Demo</a>
            </article>

            <article class="project">
                <h3>Portfolio Website</h3>
                <img src="portfolio-preview.jpg" alt="Portfolio website preview" width="300" height="200">
                <p>A personal portfolio website showcasing my work and skills.</p>
                <p><strong>Technologies:</strong> HTML, CSS, JavaScript</p>
                <a href="https://github.com/johndoe/portfolio" target="_blank">View on GitHub</a>
            </article>
        </section>

        <section id="contact">
            <h2>Contact Me</h2>
            <form action="contact.php" method="POST">
                <div>
                    <label for="name">Name:</label>
                    <input type="text" id="name" name="name" required>
                </div>

                <div>
                    <label for="email">Email:</label>
                    <input type="email" id="email" name="email" required>
                </div>

                <div>
                    <label for="subject">Subject:</label>
                    <select id="subject" name="subject">
                        <option value="general">General Inquiry</option>
                        <option value="project">Project Discussion</option>
                        <option value="job">Job Opportunity</option>
                    </select>
                </div>

                <div>
                    <label for="message">Message:</label>
                    <textarea id="message" name="message" rows="5" required></textarea>
                </div>

                <button type="submit">Send Message</button>
            </form>

            <h3>Other Ways to Reach Me</h3>
            <address>
                Email: <a href="mailto:john@example.com">john@example.com</a><br>
                Phone: <a href="tel:+1234567890">+1 (234) 567-890</a><br>
                LinkedIn: <a href="https://linkedin.com/in/johndoe" target="_blank">linkedin.com/in/johndoe</a>
            </address>
        </section>
    </main>

    <footer>
        <div class="container">
            <p>&copy; 2024 John Doe. All rights reserved.</p>
            <p>
                <a href="https://github.com/johndoe" target="_blank">GitHub</a> |
                <a href="https://twitter.com/johndoe" target="_blank">Twitter</a> |
                <a href="https://linkedin.com/in/johndoe" target="_blank">LinkedIn</a>
            </p>
        </div>
    </footer>
</body>
</html>
```

## 📚 Quick Reference

### Common HTML Tags

| Category | Tags | Purpose |
|----------|------|---------|
| **Structure** | `<html>`, `<head>`, `<body>` | Document structure |
| **Headings** | `<h1>` to `<h6>` | Content hierarchy |
| **Text** | `<p>`, `<span>`, `<div>` | Text content |
| **Formatting** | `<strong>`, `<em>`, `<mark>` | Text emphasis |
| **Links** | `<a>` | Hyperlinks |
| **Images** | `<img>`, `<figure>` | Visual content |
| **Lists** | `<ul>`, `<ol>`, `<li>` | Organized content |
| **Tables** | `<table>`, `<tr>`, `<td>` | Tabular data |
| **Forms** | `<form>`, `<input>`, `<button>` | User input |
| **Semantic** | `<header>`, `<nav>`, `<main>`, `<footer>` | Page structure |

### HTML Attributes

```html
<!-- Global attributes (can be used on any element) -->
<div id="unique-id" class="css-class" title="Tooltip text" data-custom="value">

<!-- Common attributes -->
<a href="url" target="_blank" title="Link description">
<img src="image.jpg" alt="Description" width="100" height="100">
<input type="text" name="field-name" value="default-value" placeholder="Hint text">
<form action="submit.php" method="POST" enctype="multipart/form-data">