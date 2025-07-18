
# 🌐 Intermediate HTML

## Table of Contents
- [HTML5 Semantic Elements](#html5-semantic-elements)
- [Advanced Forms](#advanced-forms)
- [Multimedia and Embedding](#multimedia-and-embedding)
- [HTML5 APIs](#html5-apis)
- [Data Attributes](#data-attributes)
- [Accessibility Features](#accessibility-features)
- [SEO Optimization](#seo-optimization)
- [Responsive HTML](#responsive-html)
- [Performance Optimization](#performance-optimization)

---

## 🏗️ HTML5 Semantic Elements

### Complete Semantic Layout

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Semantic HTML5 Layout</title>
</head>
<body>
    <header>
        <h1>Website Title</h1>
        <nav>
            <ul>
                <li><a href="#home">Home</a></li>
                <li><a href="#about">About</a></li>
                <li><a href="#services">Services</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section id="hero">
            <h2>Welcome to Our Website</h2>
            <p>Discover amazing content and services.</p>
        </section>

        <section id="features">
            <h2>Our Features</h2>
            <article>
                <h3>Feature 1</h3>
                <p>Description of feature 1...</p>
            </article>
            <article>
                <h3>Feature 2</h3>
                <p>Description of feature 2...</p>
            </article>
        </section>

        <aside>
            <h3>Related Links</h3>
            <ul>
                <li><a href="#blog">Blog</a></li>
                <li><a href="#resources">Resources</a></li>
                <li><a href="#newsletter">Newsletter</a></li>
            </ul>
        </aside>
    </main>

    <footer>
        <section>
            <h4>Contact Information</h4>
            <address>
                123 Main Street<br>
                Anytown, USA 12345<br>
                <a href="mailto:info@example.com">info@example.com</a>
            </address>
        </section>
        <section>
            <h4>Social Media</h4>
            <nav>
                <a href="https://facebook.com/example">Facebook</a>
                <a href="https://twitter.com/example">Twitter</a>
                <a href="https://linkedin.com/company/example">LinkedIn</a>
            </nav>
        </section>
        <p>&copy; 2024 Example Company. All rights reserved.</p>
    </footer>
</body>
</html>
```

### Advanced Semantic Elements

```html
<!-- Article with time and author -->
<article>
    <header>
        <h2>Understanding HTML5 Semantic Elements</h2>
        <p>Published on <time datetime="2024-01-15">January 15, 2024</time> by 
           <span>John Developer</span></p>
    </header>
    
    <section>
        <h3>Introduction</h3>
        <p>Semantic HTML elements provide meaning to web content...</p>
    </section>
    
    <section>
        <h3>Benefits</h3>
        <ul>
            <li>Better accessibility</li>
            <li>Improved SEO</li>
            <li>Cleaner code structure</li>
        </ul>
    </section>
    
    <footer>
        <p>Tags: <span>HTML5</span>, <span>Semantics</span>, <span>Web Development</span></p>
    </footer>
</article>

<!-- Figure with detailed caption -->
<figure>
    <img src="semantic-html-diagram.png" alt="HTML5 semantic elements layout diagram">
    <figcaption>
        <strong>Figure 1:</strong> HTML5 semantic elements provide structure and meaning
        to web content. This diagram shows the typical layout of semantic elements
        in a modern web page.
    </figcaption>
</figure>

<!-- Details and summary for expandable content -->
<details open>
    <summary>Browser Compatibility</summary>
    <p>HTML5 semantic elements are supported in:</p>
    <ul>
        <li>Chrome 5+</li>
        <li>Firefox 4+</li>
        <li>Safari 4.1+</li>
        <li>Internet Explorer 9+</li>
        <li>Edge (all versions)</li>
    </ul>
</details>
```

---

## 📋 Advanced Forms

### Complex Form Structure

```html
<form action="/submit-application" method="POST" enctype="multipart/form-data">
    <fieldset>
        <legend>Personal Information</legend>
        
        <div class="form-group">
            <label for="first-name">First Name *</label>
            <input type="text" id="first-name" name="firstName" required 
                   pattern="[A-Za-z]{2,}" title="First name must be at least 2 characters">
        </div>
        
        <div class="form-group">
            <label for="last-name">Last Name *</label>
            <input type="text" id="last-name" name="lastName" required 
                   pattern="[A-Za-z]{2,}" title="Last name must be at least 2 characters">
        </div>
        
        <div class="form-group">
            <label for="email">Email Address *</label>
            <input type="email" id="email" name="email" required
                   placeholder="john.doe@example.com">
        </div>
        
        <div class="form-group">
            <label for="phone">Phone Number</label>
            <input type="tel" id="phone" name="phone" 
                   pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
                   placeholder="123-456-7890"
                   title="Phone format: XXX-XXX-XXXX">
        </div>
        
        <div class="form-group">
            <label for="birth-date">Date of Birth</label>
            <input type="date" id="birth-date" name="birthDate" 
                   min="1900-01-01" max="2006-12-31">
        </div>
    </fieldset>

    <fieldset>
        <legend>Address Information</legend>
        
        <div class="form-group">
            <label for="street">Street Address</label>
            <input type="text" id="street" name="street" 
                   placeholder="123 Main Street">
        </div>
        
        <div class="form-row">
            <div class="form-group">
                <label for="city">City</label>
                <input type="text" id="city" name="city">
            </div>
            
            <div class="form-group">
                <label for="state">State</label>
                <select id="state" name="state">
                    <option value="">Select a state</option>
                    <option value="AL">Alabama</option>
                    <option value="AK">Alaska</option>
                    <option value="AZ">Arizona</option>
                    <!-- More states... -->
                </select>
            </div>
            
            <div class="form-group">
                <label for="zip">ZIP Code</label>
                <input type="text" id="zip" name="zip" 
                       pattern="[0-9]{5}(-[0-9]{4})?"
                       placeholder="12345 or 12345-6789">
            </div>
        </div>
    </fieldset>

    <fieldset>
        <legend>Preferences</legend>
        
        <div class="form-group">
            <span class="form-label">Newsletter Subscription</span>
            <div class="radio-group">
                <input type="radio" id="newsletter-yes" name="newsletter" value="yes">
                <label for="newsletter-yes">Yes, subscribe me</label>
                
                <input type="radio" id="newsletter-no" name="newsletter" value="no" checked>
                <label for="newsletter-no">No, thanks</label>
            </div>
        </div>
        
        <div class="form-group">
            <span class="form-label">Interests (select all that apply)</span>
            <div class="checkbox-group">
                <input type="checkbox" id="tech" name="interests" value="technology">
                <label for="tech">Technology</label>
                
                <input type="checkbox" id="science" name="interests" value="science">
                <label for="science">Science</label>
                
                <input type="checkbox" id="arts" name="interests" value="arts">
                <label for="arts">Arts</label>
                
                <input type="checkbox" id="sports" name="interests" value="sports">
                <label for="sports">Sports</label>
            </div>
        </div>
        
        <div class="form-group">
            <label for="experience">Experience Level</label>
            <select id="experience" name="experience" required>
                <option value="">Please select...</option>
                <option value="beginner">Beginner (0-1 years)</option>
                <option value="intermediate">Intermediate (2-5 years)</option>
                <option value="advanced">Advanced (5+ years)</option>
                <option value="expert">Expert (10+ years)</option>
            </select>
        </div>
    </fieldset>

    <fieldset>
        <legend>File Uploads</legend>
        
        <div class="form-group">
            <label for="resume">Resume (PDF, DOC, DOCX)</label>
            <input type="file" id="resume" name="resume" 
                   accept=".pdf,.doc,.docx" required>
        </div>
        
        <div class="form-group">
            <label for="portfolio">Portfolio Images (Optional)</label>
            <input type="file" id="portfolio" name="portfolio" 
                   accept="image/*" multiple>
        </div>
    </fieldset>

    <fieldset>
        <legend>Additional Information</legend>
        
        <div class="form-group">
            <label for="bio">Bio/Cover Letter</label>
            <textarea id="bio" name="bio" rows="6" cols="50" 
                      maxlength="1000" placeholder="Tell us about yourself..."></textarea>
            <small>Maximum 1000 characters</small>
        </div>
        
        <div class="form-group">
            <label for="salary">Expected Salary Range</label>
            <input type="range" id="salary" name="salary" 
                   min="30000" max="150000" step="5000" value="60000"
                   oninput="document.getElementById('salary-output').value = this.value">
            <output id="salary-output" for="salary">60000</output>
        </div>
        
        <div class="form-group">
            <label for="start-date">Available Start Date</label>
            <input type="date" id="start-date" name="startDate" 
                   min="2024-01-01">
        </div>
    </fieldset>

    <div class="form-actions">
        <button type="button" onclick="resetForm()">Clear Form</button>
        <button type="submit">Submit Application</button>
    </div>
</form>

<script>
function resetForm() {
    if (confirm('Are you sure you want to clear all form data?')) {
        document.querySelector('form').reset();
    }
}

// Real-time salary display
document.getElementById('salary').addEventListener('input', function() {
    document.getElementById('salary-output').textContent = 
        '$' + parseInt(this.value).toLocaleString();
});
</script>
```

### Form Validation and Enhancement

```html
<form novalidate>
    <!-- Custom validation messages -->
    <div class="form-group">
        <label for="username">Username</label>
        <input type="text" id="username" name="username" 
               pattern="[a-zA-Z0-9_]{3,16}" required
               data-error="Username must be 3-16 characters and contain only letters, numbers, and underscores">
        <div class="error-message"></div>
    </div>
    
    <!-- Password strength indicator -->
    <div class="form-group">
        <label for="password">Password</label>
        <input type="password" id="password" name="password" 
               minlength="8" required>
        <div class="password-strength">
            <div class="strength-bar"></div>
            <span class="strength-text">Password strength</span>
        </div>
    </div>
    
    <!-- Confirm password -->
    <div class="form-group">
        <label for="confirm-password">Confirm Password</label>
        <input type="password" id="confirm-password" name="confirmPassword" 
               required>
        <div class="error-message"></div>
    </div>
    
    <!-- Terms and conditions -->
    <div class="form-group">
        <input type="checkbox" id="terms" name="terms" required>
        <label for="terms">
            I agree to the <a href="/terms" target="_blank">Terms and Conditions</a> 
            and <a href="/privacy" target="_blank">Privacy Policy</a>
        </label>
    </div>
    
    <button type="submit">Create Account</button>
</form>

<script>
// Custom form validation
document.querySelector('form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    const username = document.getElementById('username');
    const password = document.getElementById('password');
    const confirmPassword = document.getElementById('confirm-password');
    
    // Clear previous errors
    document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
    
    let isValid = true;
    
    // Username validation
    if (!username.validity.valid) {
        showError(username, username.dataset.error);
        isValid = false;
    }
    
    // Password confirmation
    if (password.value !== confirmPassword.value) {
        showError(confirmPassword, 'Passwords do not match');
        isValid = false;
    }
    
    if (isValid) {
        alert('Form submitted successfully!');
        // Here you would normally submit the form
    }
});

function showError(input, message) {
    const errorDiv = input.parentNode.querySelector('.error-message');
    errorDiv.textContent = message;
    input.classList.add('error');
}

// Password strength checker
document.getElementById('password').addEventListener('input', function() {
    const password = this.value;
    const strengthBar = document.querySelector('.strength-bar');
    const strengthText = document.querySelector('.strength-text');
    
    let strength = 0;
    let feedback = '';
    
    if (password.length >= 8) strength++;
    if (/[a-z]/.test(password)) strength++;
    if (/[A-Z]/.test(password)) strength++;
    if (/[0-9]/.test(password)) strength++;
    if (/[^A-Za-z0-9]/.test(password)) strength++;
    
    switch (strength) {
        case 0:
        case 1:
            strengthBar.style.width = '20%';
            strengthBar.style.backgroundColor = '#ff4444';
            feedback = 'Very weak';
            break;
        case 2:
            strengthBar.style.width = '40%';
            strengthBar.style.backgroundColor = '#ff8800';
            feedback = 'Weak';
            break;
        case 3:
            strengthBar.style.width = '60%';
            strengthBar.style.backgroundColor = '#ffbb00';
            feedback = 'Fair';
            break;
        case 4:
            strengthBar.style.width = '80%';
            strengthBar.style.backgroundColor = '#88cc00';
            feedback = 'Good';
            break;
        case 5:
            strengthBar.style.width = '100%';
            strengthBar.style.backgroundColor = '#00aa00';
            feedback = 'Strong';
            break;
    }
    
    strengthText.textContent = feedback;
});
</script>
```

---

## 🎵 Multimedia and Embedding

### Advanced Video Implementation

```html
<!-- Responsive video with multiple sources -->
<figure>
    <video controls poster="video-thumbnail.jpg" preload="metadata">
        <source src="video.mp4" type="video/mp4">
        <source src="video.webm" type="video/webm">
        <source src="video.ogv" type="video/ogg">
        
        <!-- Subtitles and captions -->
        <track kind="subtitles" src="subtitles-en.vtt" srclang="en" label="English" default>
        <track kind="subtitles" src="subtitles-es.vtt" srclang="es" label="Español">
        <track kind="captions" src="captions-en.vtt" srclang="en" label="English Captions">
        
        <!-- Fallback content -->
        <p>Your browser doesn't support HTML5 video. 
           <a href="video.mp4">Download the video</a> instead.</p>
    </video>
    <figcaption>
        Product demonstration video showing key features and functionality.
    </figcaption>
</figure>

<!-- Custom video player controls -->
<div class="video-container">
    <video id="custom-video" poster="thumbnail.jpg">
        <source src="presentation.mp4" type="video/mp4">
    </video>
    
    <div class="video-controls">
        <button id="play-pause">▶️</button>
        <input type="range" id="progress" min="0" max="100" value="0">
        <span id="time-display">0:00 / 0:00</span>
        <button id="mute">🔊</button>
        <input type="range" id="volume" min="0" max="100" value="100">
        <button id="fullscreen">⛶</button>
    </div>
</div>

<script>
const video = document.getElementById('custom-video');
const playPause = document.getElementById('play-pause');
const progress = document.getElementById('progress');
const timeDisplay = document.getElementById('time-display');

playPause.addEventListener('click', () => {
    if (video.paused) {
        video.play();
        playPause.textContent = '⏸️';
    } else {
        video.pause();
        playPause.textContent = '▶️';
    }
});

video.addEventListener('timeupdate', () => {
    const progressPercent = (video.currentTime / video.duration) * 100;
    progress.value = progressPercent;
    
    const currentMinutes = Math.floor(video.currentTime / 60);
    const currentSeconds = Math.floor(video.currentTime % 60);
    const durationMinutes = Math.floor(video.duration / 60);
    const durationSeconds = Math.floor(video.duration % 60);
    
    timeDisplay.textContent = 
        `${currentMinutes}:${currentSeconds.toString().padStart(2, '0')} / ` +
        `${durationMinutes}:${durationSeconds.toString().padStart(2, '0')}`;
});

progress.addEventListener('input', () => {
    const time = (progress.value / 100) * video.duration;
    video.currentTime = time;
});
</script>
```

### Audio Player with Playlist

```html
<div class="audio-player">
    <audio id="audio-player" controls preload="metadata">
        <source src="song1.mp3" type="audio/mpeg">
        <source src="song1.ogg" type="audio/ogg">
        Your browser does not support the audio element.
    </audio>
    
    <div class="playlist">
        <h3>Playlist</h3>
        <ul id="playlist">
            <li data-src="song1.mp3" class="active">Song 1 - Artist 1</li>
            <li data-src="song2.mp3">Song 2 - Artist 2</li>
            <li data-src="song3.mp3">Song 3 - Artist 3</li>
        </ul>
    </div>
    
    <div class="player-info">
        <h4 id="current-song">Song 1 - Artist 1</h4>
        <div class="controls">
            <button id="prev">⏮️</button>
            <button id="play-pause-audio">⏸️</button>
            <button id="next">⏭️</button>
            <button id="shuffle">🔀</button>
            <button id="repeat">🔁</button>
        </div>
    </div>
</div>

<script>
class AudioPlayer {
    constructor() {
        this.audio = document.getElementById('audio-player');
        this.playlist = document.getElementById('playlist');
        this.currentSong = document.getElementById('current-song');
        this.playPauseBtn = document.getElementById('play-pause-audio');
        
        this.currentIndex = 0;
        this.songs = Array.from(this.playlist.querySelectorAll('li'));
        this.isShuffled = false;
        this.isRepeating = false;
        
        this.init();
    }
    
    init() {
        // Playlist click events
        this.songs.forEach((song, index) => {
            song.addEventListener('click', () => this.playSong(index));
        });
        
        // Control button events
        document.getElementById('prev').addEventListener('click', () => this.previousSong());
        document.getElementById('next').addEventListener('click', () => this.nextSong());
        this.playPauseBtn.addEventListener('click', () => this.togglePlayPause());
        
        // Audio events
        this.audio.addEventListener('ended', () => this.nextSong());
        this.audio.addEventListener('play', () => this.playPauseBtn.textContent = '⏸️');
        this.audio.addEventListener('pause', () => this.playPauseBtn.textContent = '▶️');
    }
    
    playSong(index) {
        this.currentIndex = index;
        const song = this.songs[index];
        
        // Update audio source
        this.audio.src = song.dataset.src;
        this.audio.load();
        this.audio.play();
        
        // Update UI
        this.songs.forEach(s => s.classList.remove('active'));
        song.classList.add('active');
        this.currentSong.textContent = song.textContent;
    }
    
    nextSong() {
        if (this.isRepeating) {
            this.audio.currentTime = 0;
            this.audio.play();
        } else {
            this.currentIndex = (this.currentIndex + 1) % this.songs.length;
            this.playSong(this.currentIndex);
        }
    }
    
    previousSong() {
        this.currentIndex = this.currentIndex === 0 ? this.songs.length - 1 : this.currentIndex - 1;
        this.playSong(this.currentIndex);
    }
    
    togglePlayPause() {
        if (this.audio.paused) {
            this.audio.play();
        } else {
            this.audio.pause();
        }
    }
}

new AudioPlayer();
</script>
```

### Responsive Embedded Content

```html
<!-- Responsive YouTube embed -->
<div class="video-wrapper">
    <iframe src="https://www.youtube.com/embed/dQw4w9WgXcQ" 
            frameborder="0" 
            allowfullscreen
            title="YouTube video">
    </iframe>
</div>

<!-- Responsive map embed -->
<div class="map-wrapper">
    <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3024.1!2d-74.0059!3d40.7128!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x0!2zNDAlNDInNDYuMSJOIDc0wrogMCczMy4zIlc!5e0!3m2!1sen!2sus!4v1234567890"
            allowfullscreen
            loading="lazy"
            title="Location map">
    </iframe>
</div>

<!-- Responsive social media embed -->
<div class="social-embed">
    <blockquote class="twitter-tweet">
        <p lang="en" dir="ltr">Just learned about HTML5 semantic elements! 🚀</p>
        &mdash; Web Developer (@webdev) <a href="https://twitter.com/webdev/status/123456789">January 1, 2024</a>
    </blockquote>
    <script async src="https://platform.twitter.com/widgets.js"></script>
</div>

<style>
.video-wrapper, .map-wrapper {
    position: relative;
    padding-bottom: 56.25%; /* 16:9 aspect ratio */
    height: 0;
    overflow: hidden;
    max-width: 100%;
}

.video-wrapper iframe, .map-wrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.social-embed {
    max-width: 550px;
    margin: 20px auto;
}
</style>
```

---

## 🔧 HTML5 APIs

### Geolocation API

```html
<div class="location-demo">
    <button id="get-location">Get My Location</button>
    <div id="location-info"></div>
    <div id="map-container"></div>
</div>

<script>
document.getElementById('get-location').addEventListener('click', function() {
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition, showError, {
            enableHighAccuracy: true,
            timeout: 5000,
            maximumAge: 0
        });
    } else {
        document.getElementById('location-info').innerHTML = 
            "Geolocation is not supported by this browser.";
    }
});

function showPosition(position) {
    const lat = position.coords.latitude;
    const lon = position.coords.longitude;
    const accuracy = position.coords.accuracy;
    
    document.getElementById('location-info').innerHTML = `
        <h3>Your Location:</h3>
        <p><strong>Latitude:</strong> ${lat.toFixed(6)}</p>
        <p><strong>Longitude:</strong> ${lon.toFixed(6)}</p>
        <p><strong>Accuracy:</strong> ${accuracy} meters</p>
        <p><strong>Timestamp:</strong> ${new Date(position.timestamp).toLocaleString()}</p>
    `;
    
    // Create a simple map link
    const mapUrl = `https://www.google.com/maps?q=${lat},${lon}`;
    document.getElementById('map-container').innerHTML = 
        `<a href="${mapUrl}" target="_blank">View on Google Maps</a>`;
}

function showError(error) {
    let errorMessage = '';
    switch(error.code) {
        case error.PERMISSION_DENIED:
            errorMessage = "User denied the request for Geolocation.";
            break;
        case error.POSITION_UNAVAILABLE:
            errorMessage = "Location information is unavailable.";
            break;
        case error.TIMEOUT:
            errorMessage = "The request to get user location timed out.";
            break;
        default:
            errorMessage = "An unknown error occurred.";
            break;
    }
    document.getElementById('location-info').innerHTML = errorMessage;
}
</script>
```

### Local Storage Demo

```html
<div class="storage-demo">
    <h3>Local Storage Demo</h3>
    
    <div class="form-group">
        <label for="storage-key">Key:</label>
        <input type="text" id="storage-key" placeholder="Enter key">
    </div>
    
    <div class="form-group">
        <label for="storage-value">Value:</label>
        <input type="text" id="storage-value" placeholder="Enter value">
    </div>
    
    <div class="buttons">
        <button onclick="setItem()">Save</button>
        <button onclick="getItem()">Load</button>
        <button onclick="removeItem()">Remove</button>
        <button onclick="clearAll()">Clear All</button>
        <button onclick="showAll()">Show All</button>
    </div>
    
    <div id="storage-output"></div>
    <div id="storage-list"></div>
</div>

<script>
function setItem() {
    const key = document.getElementById('storage-key').value;
    const value = document.getElementById('storage-value').value;
    
    if (key && value) {
        localStorage.setItem(key, value);
        document.getElementById('storage-output').innerHTML = 
            `✅ Saved: ${key} = ${value}`;
        updateStorageList();
    } else {
        alert('Please enter both key and value');
    }
}

function getItem() {
    const key = document.getElementById('storage-key').value;
    
    if (key) {
        const value = localStorage.getItem(key);
        if (value !== null) {
            document.getElementById('storage-value').value = value;
            document.getElementById('storage-output').innerHTML = 
                `📖 Loaded: ${key} = ${value}`;
        } else {
            document.getElementById('storage-output').innerHTML = 
                `❌ Key '${key}' not found`;
        }
    } else {
        alert('Please enter a key');
    }
}

function removeItem() {
    const key = document.getElementById('storage-key').value;
    
    if (key) {
        localStorage.removeItem(key);
        document.getElementById('storage-output').innerHTML = 
            `🗑️ Removed: ${key}`;
        updateStorageList();
    } else {
        alert('Please enter a key');
    }
}

function clearAll() {
    if (confirm('Are you sure you want to clear all stored data?')) {
        localStorage.clear();
        document.getElementById('storage-output').innerHTML = 
            '🧹 All data cleared';
        updateStorageList();
    }
}

function showAll() {
    updateStorageList();
}

function updateStorageList() {
    const listDiv = document.getElementById('storage-list');
    
    if (localStorage.length === 0) {
        listDiv.innerHTML = '<p>No items in local storage</p>';
        return;
    }
    
    let html = '<h4>Stored Items:</h4><ul>';
    for (let i = 0; i < localStorage.length; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);
        html += `<li><strong>${key}:</strong> ${value}</li>`;
    }
    html += '</ul>';
    listDiv.innerHTML = html;
}

// Initialize the list on page load
document.addEventListener('DOMContentLoaded', updateStorageList);
</script>
```

### Drag and Drop API

```html
<div class="drag-drop-demo">
    <h3>Drag and Drop Demo</h3>
    
    <div class="drag-items">
        <div class="drag-item" draggable="true" data-value="item1">Item 1</div>
        <div class="drag-item" draggable="true" data-value="item2">Item 2</div>
        <div class="drag-item" draggable="true" data-value="item3">Item 3</div>
        <div class="drag-item" draggable="true" data-value="item4">Item 4</div>
    </div>
    
    <div class="drop-zones">
        <div class="drop-zone" data-category="category-a">
            <h4>Category A</h4>
            <p>Drop items here</p>
        </div>
        <div class="drop-zone" data-category="category-b">
            <h4>Category B</h4>
            <p>Drop items here</p>
        </div>
    </div>
    
    <button onclick="resetDemo()">Reset Demo</button>
</div>

<style>
.drag-drop-demo {
    max-width: 800px;
    margin: 20px auto;
}

.drag-items {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
}

.drag-item {
    padding: 10px 20px;
    background: #3498db;
    color: white;
    border-radius: 5px;
    cursor: move;
    user-select: none;
}

.drag-item:hover {
    background: #2980b9;
}

.drag-item.dragging {
    opacity: 0.5;
}

.drop-zones {
    display: flex;
    gap: 20px;
}

.drop-zone {
    flex: 1;
    min-height: 200px;
    border: 2px dashed #bdc3c7;
    border-radius: 5px;
    padding: 20px;
    text-align: center;
    background: #ecf0f1;
}

.drop-zone.drag-over {
    border-color: #3498db;
    background: #e8f4fd;
}

.drop-zone .dropped-item {
    display: inline-block;
    margin: 5px;
    padding: 5px 10px;
    background: #27ae60;
    color: white;
    border-radius: 3px;
    font-size: 0.9em;
}
</style>

<script>
// Drag start event
document.querySelectorAll('.drag-item').forEach(item => {
    item.addEventListener('dragstart', function(e) {
        e.dataTransfer.setData('text/plain', this.dataset.value);
        e.dataTransfer.setData('text/html', this.outerHTML);
        this.classList.add('dragging');
    });
    
    item.addEventListener('dragend', function() {
        this.classList.remove('dragging');
    });
});

// Drop zone events
document.querySelectorAll('.drop-zone').forEach(zone => {
    zone.addEventListener('dragover', function(e) {
        e.preventDefault();
        this.classList.add('drag-over');
    });
    
    zone.addEventListener('dragleave', function() {
        this.classList.remove('drag-over');
    });
    
    zone.addEventListener('drop', function(e) {
        e.preventDefault();
        this.classList.remove('drag-over');
        
        const draggedValue = e.dataTransfer.getData('text/plain');
        const draggedHTML = e.dataTransfer.getData('text/html');
        
        // Create dropped item display
        const droppedItem = document.createElement('div');
        droppedItem.className = 'dropped-item';
        droppedItem.textContent = draggedValue;
        
        // Remove placeholder text if this is the first item
        if (!this.querySelector('.dropped-item')) {
            this.querySelector('p').style.display = 'none';
        }
        
        this.appendChild(droppedItem);
        
        // Remove original item (optional)
        // document.querySelector(`[data-value="${draggedValue}"]`).remove();
    });
});

function resetDemo() {
    document.querySelectorAll('.drop-zone').forEach(zone => {
        zone.querySelectorAll('.dropped-item').forEach(item => item.remove());
        zone.querySelector('p').style.display = 'block';
    });
}
</script>
```

---

## 📊 Data Attributes

### Using Data Attributes for Dynamic Content

```html
<div class="product-catalog">
    <h3>Product Catalog</h3>
    
    <div class="filters">
        <button data-filter="all" class="filter-btn active">All</button>
        <button data-filter="electronics" class="filter-btn">Electronics</button>
        <button data-filter="clothing" class="filter-btn">Clothing</button>
        <button data-filter="books" class="filter-btn">Books</button>
    </div>
    
    <div class="sort-options">
        <label for="sort-select">Sort by:</label>
        <select id="sort-select">
            <option value="name">Name</option>
            <option value="price">Price</option>
            <option value="rating">Rating</option>
        </select>
    </div>
    
    <div class="products-grid">
        <div class="product-card" 
             data-category="electronics" 
             data-price="999" 
             data-rating="4.5"
             data-name="Laptop">
            <h4>Laptop</h4>
            <p>Category: Electronics</p>
            <p>Price: $999</p>
            <p>Rating: ⭐⭐⭐⭐⭐</p>
        </div>
        
        <div class="product-card" 
             data-category="clothing" 
             data-price="49" 
             data-rating="4.2"
             data-name="T-Shirt">
            <h4>T-Shirt</h4>
            <p>Category: Clothing</p>
            <p>Price: $49</p>
            <p>Rating: ⭐⭐⭐⭐</p>
        </div>
        
        <div class="product-card" 
             data-category="books" 
             data-price="29" 
             data-rating="4.8"
             data-name="Web Development Guide">
            <h4>Web Development Guide</h4>
            <p>Category: Books</p>
            <p>Price: $29</p>
            <p>Rating: ⭐⭐⭐⭐⭐</p>
        </div>
        
        <div class="product-card" 
             data-category="electronics" 
             data-price="599" 
             data-rating="4.3"
             data-name="Smartphone">
            <h4>Smartphone</h4>
            <p>Category: Electronics</p>
            <p>Price: $599</p>
            <p>Rating: ⭐⭐⭐⭐</p>
        </div>
    </div>
</div>

<script>
class ProductCatalog {
    constructor() {
        this.products = document.querySelectorAll('.product-card');
        this.filterBtns = document.querySelectorAll('.filter-btn');
        this.sortSelect = document.getElementById('sort-select');
        
        this.init();
    }
    
    init() {
        // Filter button events
        this.filterBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                this.filterProducts(e.target.dataset.filter);
                this.updateActiveFilter(e.target);
            });
        });
        
        // Sort select event
        this.sortSelect.addEventListener('change', (e) => {
            this.sortProducts(e.target.value);
        });
    }
    
    filterProducts(category) {
        this.products.forEach(product => {
            if (category === 'all' || product.dataset.category === category) {
                product.style.display = 'block';
            } else {
                product.style.display = 'none';
            }
        });
    }
    
    updateActiveFilter(activeBtn) {
        this.filterBtns.forEach(btn => btn.classList.remove('active'));
        activeBtn.classList.add('active');
    }
    
    sortProducts(criteria) {
        const productsArray = Array.from(this.products);
        const container = document.querySelector('.products-grid');
        
        productsArray.sort((a, b) => {
            let aValue, bValue;
            
            switch (criteria) {
                case 'name':
                    aValue = a.dataset.name.toLowerCase();
                    bValue = b.dataset.name.toLowerCase();
                    return aValue.localeCompare(bValue);
                case 'price':
                    aValue = parseFloat(a.dataset.price);
                    bValue = parseFloat(b.dataset.price);
                    return aValue - bValue;
                case 'rating':
                    aValue = parseFloat(a.dataset.rating);
                    bValue = parseFloat(b.dataset.rating);
                    return bValue - aValue; // Descending for rating
                default:
                    return 0;
            }
        });
        
        // Re-append sorted elements
        productsArray.forEach(product => container.appendChild(product));
    }
}

new ProductCatalog();
</script>

<style>
.product-catalog {
    max-width: 1000px;
    margin: 20px auto;
}

.filters {
    margin: 20px 0;
}

.filter-btn {
    margin-right: 10px;
    padding: 8px 16px;
    border: 1px solid #ddd;
    background: white;
    cursor: pointer;
    border-radius: 4px;
}

.filter-btn.active {
    background: #3498db;
    color: white;
}

.sort-options {
    margin: 20px 0;
}

.products-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}

.product-card {
    border: 1px solid #ddd;
    padding: 20px;
    border-radius: 8px;
    background: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
</style>
```

---

## ♿ Accessibility Features

### ARIA Attributes and Screen Reader Support

```html
<!-- Skip navigation -->
<a href="#main-content" class="skip-link">Skip to main content</a>

<nav role="navigation" aria-label="Main navigation">
    <ul>
        <li><a href="#home" aria-current="page">Home</a></li>
        <li><a href="#about">About</a></li>
        <li><a href="#services" aria-expanded="false" aria-haspopup="true">Services</a>
            <ul aria-hidden="true">
                <li><a href="#web-design">Web Design</a></li>
                <li><a href="#development">Development</a></li>
            </ul>
        </li>
        <li><a href="#contact">Contact</a></li>
    </ul>
</nav>

<main id="main-content" role="main">
    <!-- Accessible form -->
    <form role="form" aria-labelledby="contact-heading">
        <h2 id="contact-heading">Contact Us</h2>
        
        <fieldset>
            <legend>Personal Information</legend>
            
            <div class="form-group">
                <label for="full-name">
                    Full Name <span aria-label="required">*</span>
                </label>
                <input type="text" 
                       id="full-name" 
                       name="fullName" 
                       required 
                       aria-required="true"
                       aria-describedby="name-help">
                <div id="name-help" class="help-text">
                    Please enter your first and last name
                </div>
            </div>
            
            <div class="form-group">
                <label for="email">
                    Email Address <span aria-label="required">*</span>
                </label>
                <input type="email" 
                       id="email" 
                       name="email" 
                       required 
                       aria-required="true"
                       aria-describedby="email-error"
                       aria-invalid="false">
                <div id="email-error" 
                     class="error-message" 
                     role="alert" 
                     aria-live="polite"
                     style="display: none;">
                </div>
            </div>
            
            <div class="form-group">
                <span class="form-label" id="gender-label">Gender</span>
                <div role="radiogroup" aria-labelledby="gender-label">
                    <input type="radio" id="male" name="gender" value="male">
                    <label for="male">Male</label>
                    
                    <input type="radio" id="female" name="gender" value="female">
                    <label for="female">Female</label>
                    
                    <input type="radio" id="prefer-not-to-say" name="gender" value="other">
                    <label for="prefer-not-to-say">Prefer not to say</label>
                </div>
            </div>
        </fieldset>
        
        <fieldset>
            <legend>Message</legend>
            
            <div class="form-group">
                <label for="subject">Subject</label>
                <select id="subject" name="subject" aria-describedby="subject-help">
                    <option value="">Please select a subject</option>
                    <option value="general">General Inquiry</option>
                    <option value="support">Technical Support</option>
                    <option value="billing">Billing Question</option>
                </select>
                <div id="subject-help" class="help-text">
                    Choose the category that best describes your message
                </div>
            </div>
            
            <div class="form-group">
                <label for="message">Message</label>
                <textarea id="message" 
                          name="message" 
                          rows="5" 
                          aria-describedby="message-counter"
                          maxlength="500"></textarea>
                <div id="message-counter" aria-live="polite">
                    0 / 500 characters
                </div>
            </div>
        </fieldset>
        
        <button type="submit" aria-describedby="submit-help">
            Send Message
        </button>
        <div id="submit-help" class="help-text">
            Click to send your message. We typically respond within 24 hours.
        </div>
    </form>
    
    <!-- Accessible data table -->
    <table role="table" aria-labelledby="sales-caption">
        <caption id="sales-caption">
            Quarterly Sales Report for 2024
        </caption>
        <thead>
            <tr>
                <th scope="col">Quarter</th>
                <th scope="col">Revenue</th>
                <th scope="col">Units Sold</th>
                <th scope="col">Growth %</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th scope="row">Q1 2024</th>
                <td>$125,000</td>
                <td>1,250</td>
                <td>+15%</td>
            </tr>
            <tr>
                <th scope="row">Q2 2024</th>
                <td>$142,000</td>
                <td>1,420</td>
                <td>+13.6%</td>
            </tr>
        </tbody>
    </table>
    
    <!-- Accessible modal dialog -->
    <button onclick="openModal()" aria-haspopup="dialog">
        Open Settings
    </button>
    
    <div id="settings-modal" 
         class="modal" 
         role="dialog" 
         aria-labelledby="modal-title"
         aria-describedby="modal-description"
         aria-hidden="true">
        <div class="modal-content">
            <h3 id="modal-title">Settings</h3>
            <p id="modal-description">Configure your preferences</p>
            
            <div class="modal-body">
                <label>
                    <input type="checkbox" name="notifications"> 
                    Enable notifications
                </label>
                <label>
                    <input type="checkbox" name="auto-save"> 
                    Auto-save changes
                </label>
            </div>
            
            <div class="modal-actions">
                <button onclick="closeModal()">Cancel</button>
                <button onclick="saveSettings()">Save</button>
            </div>
            
            <button class="modal-close" 
                    onclick="closeModal()" 
                    aria-label="Close settings modal">
                ×
            </button>
        </div>
    </div>
</main>

<script>
// Character counter for textarea
document.getElementById('message').addEventListener('input', function() {
    const counter = document.getElementById('message-counter');
    const current = this.value.length;
    const max = this.maxLength;
    counter.textContent = `${current} / ${max} characters`;
    
    if (current > max * 0.8) {
        counter.style.color = '#e74c3c';
    } else {
        counter.style.color = '';
    }
});

// Modal functions
function openModal() {
    const modal = document.getElementById('settings-modal');
    modal.style.display = 'block';
    modal.setAttribute('aria-hidden', 'false');
    
    // Focus management
    const firstFocusable = modal.querySelector('input, button');
    if (firstFocusable) firstFocusable.focus();
    
    // Trap focus within modal
    document.addEventListener('keydown', trapFocus);
}

function closeModal() {
    const modal = document.getElementById('settings-modal');
    modal.style.display = 'none';
    modal.setAttribute('aria-hidden', 'true');
    
    document.removeEventListener('keydown', trapFocus);
    
    // Return focus to trigger button
    document.querySelector('[aria-haspopup="dialog"]').focus();
}

function trapFocus(e) {
    if (e.key === 'Escape') {
        closeModal();
        return;
    }
    
    if (e.key === 'Tab') {
        const modal = document.getElementById('settings-modal');
        const focusableElements = modal.querySelectorAll(
            'input, button, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        const firstFocusable = focusableElements[0];
        const lastFocusable = focusableElements[focusableElements.length - 1];
        
        if (e.shiftKey) {
            if (document.activeElement === firstFocusable) {
                lastFocusable.focus();
                e.preventDefault();
            }
        } else {
            if (document.activeElement === lastFocusable) {
                firstFocusable.focus();
                e.preventDefault();
            }
        }
    }
}

function saveSettings() {
    alert('Settings saved!');
    closeModal();
}
</script>

<style>
.skip-link {
    position: absolute;
    top: -40px;
    left: 6px;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
    border-radius: 0 0 4px 4px;
}

.skip-link:focus {
    top: 0;
}

.help-text {
    font-size: 0.9em;
    color: #666;
    margin-top: 4px;
}

.error-message {
    color: #e74c3c;
    font-size: 0.9em;
    margin-top: 4px;
}

.modal {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0,0,0,0.5);
    z-index: 1000;
}

.modal-content {
    position: relative;
    background: white;
    margin: 50px auto;
    padding: 20px;
    width: 90%;
    max-width: 500px;
    border-radius: 8px;
}

.modal-close {
    position: absolute;
    top: 10px;
    right: 15px;
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
}
</style>
```

---

## 🚀 Performance Optimization

### Optimized Loading Strategies

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Performance Optimized Page</title>
    
    <!-- Critical CSS inline -->
    <style>
        body { font-family: Arial, sans-serif; margin: 0; }
        .hero { background: #3498db; color: white; padding: 60px 20px; text-align: center; }
        .container { max-width: 1200px; margin: 0 auto; }
    </style>
    
    <!-- Preload critical resources -->
    <link rel="preload" href="fonts/main-font.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="hero-image.jpg" as="image">
    
    <!-- DNS prefetch for external domains -->
    <link rel="dns-prefetch" href="//fonts.googleapis.com">
    <link rel="dns-prefetch" href="//cdn.example.com">
    
    <!-- Non-critical CSS with media queries -->
    <link rel="stylesheet" href="styles.css" media="print" onload="this.media='all'">
</head>
<body>
    <!-- Above-the-fold content -->
    <section class="hero">
        <div class="container">
            <h1>Welcome to Our Website</h1>
            <p>Fast loading and optimized for performance</p>
        </div>
    </section>
    
    <!-- Lazy loaded images -->
    <section class="gallery">
        <h2>Image Gallery</h2>
        <div class="image-grid">
            <!-- First image loads immediately -->
            <img src="image1.jpg" alt="Gallery image 1" width="300" height="200">
            
            <!-- Subsequent images lazy load -->
            <img src="placeholder.jpg" 
                 data-src="image2.jpg" 
                 alt="Gallery image 2" 
                 width="300" 
                 height="200" 
                 loading="lazy" 
                 class="lazy">
            
            <img src="placeholder.jpg" 
                 data-src="image3.jpg" 
                 alt="Gallery image 3" 
                 width="300" 
                 height="200" 
                 loading="lazy" 
                 class="lazy">
            
            <!-- Using picture element for responsive images -->
            <picture>
                <source media="(min-width: 800px)" 
                        srcset="large-image.jpg" 
                        width="300" 
                        height="200">
                <source media="(min-width: 400px)" 
                        srcset="medium-image.jpg" 
                        width="300" 
                        height="200">
                <img src="small-image.jpg" 
                     alt="Responsive image" 
                     width="300" 
                     height="200" 
                     loading="lazy">
            </picture>
        </div>
    </section>
    
    <!-- Deferred content loading -->
    <section id="content-placeholder">
        <div class="loader">Loading additional content...</div>
    </section>
    
    <!-- JavaScript at the end for non-blocking -->
    <script>
        // Intersection Observer for lazy loading
        if ('IntersectionObserver' in window) {
            const imageObserver = new IntersectionObserver((entries, observer) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        const img = entry.target;
                        img.src = img.dataset.src;
                        img.classList.remove('lazy');
                        observer.unobserve(img);
                    }
                });
            });
            
            document.querySelectorAll('.lazy').forEach(img => {
                imageObserver.observe(img);
            });
        }
        
        // Defer non-critical content
        function loadDeferredContent() {
            fetch('/api/additional-content')
                .then(response => response.text())
                .then(html => {
                    document.getElementById('content-placeholder').innerHTML = html;
                })
                .catch(error => {
                    console.error('Error loading content:', error);
                    document.getElementById('content-placeholder').innerHTML = 
                        '<p>Content could not be loaded.</p>';
                });
        }
        
        // Load deferred content after page load
        window.addEventListener('load', () => {
            setTimeout(loadDeferredContent, 1000);
        });
        
        // Service Worker registration for caching
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/sw.js')
                    .then(registration => {
                        console.log('SW registered: ', registration);
                    })
                    .catch(registrationError => {
                        console.log('SW registration failed: ', registrationError);
                    });
            });
        }
    </script>
    
    <!-- Load non-critical CSS asynchronously -->
    <noscript>
        <link rel="stylesheet" href="styles.css">
    </noscript>
</body>
</html>
```

### Resource Optimization

```html
<!-- Optimized meta tags for social sharing -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Learn intermediate HTML concepts including semantic elements, forms, and performance optimization.">
    <meta name="keywords" content="HTML5, web development, semantic HTML, forms, performance">
    <meta name="author" content="Web Development Team">
    
    <!-- Open Graph tags -->
    <meta property="og:title" content="Intermediate HTML Guide">
    <meta property="og:description" content="Comprehensive guide to intermediate HTML concepts">
    <meta property="og:image" content="https://example.com/og-image.jpg">
    <meta property="og:url" content="https://example.com/intermediate-html">
    <meta property="og:type" content="article">
    
    <!-- Twitter Card tags -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:site" content="@yourhandle">
    <meta name="twitter:title" content="Intermediate HTML Guide">
    <meta name="twitter:description" content="Comprehensive guide to intermediate HTML concepts">
    <meta name="twitter:image" content="https://example.com/twitter-image.jpg">
    
    <!-- Favicon and app icons -->
    <link rel="icon" type="image/x-icon" href="/favicon.ico">
    <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png">
    <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png">
    <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png">
    <link rel="manifest" href="/site.webmanifest">
    
    <title>Intermediate HTML - Complete Guide</title>
</head>
```

## 📚 Quick Reference

### HTML5 Semantic Elements

| Element | Purpose |
|---------|---------|
| `<header>` | Page or section header |
| `<nav>` | Navigation links |
| `<main>` | Main content area |
| `<article>` | Self-contained content |
| `<section>` | Thematic grouping |
| `<aside>` | Sidebar content |
| `<footer>` | Page or section footer |
| `<figure>` | Self-contained media |
| `<figcaption>` | Figure caption |
| `<time>` | Date/time information |

### Advanced Form Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `pattern` | Input validation regex | `pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"` |
| `autocomplete` | Browser autocomplete | `autocomplete="email"` |
| `aria-*` | Accessibility labels | `aria-label="Close dialog"` |
| `data-*` | Custom data storage | `data-category="electronics"` |
| `loading` | Lazy loading | `loading="lazy"` |

### Next Steps:
1. Master CSS for advanced styling
2. Learn JavaScript for interactivity
3. Explore Progressive Web Apps
4. Study web performance optimization
5. Practice accessibility testing
6. Learn about web security
