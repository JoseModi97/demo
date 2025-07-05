```markdown
# Recreating Scroll-Controlled Lottie Animation

This document explains how to recreate a Lottie animation that is controlled by user scroll (mouse wheel), touch gestures (swipe up/down), and keyboard arrow keys (Up/Down). The animation frames are smoothly transitioned based on user input, and a visual scroll indicator provides feedback on the animation's progress.

## Prerequisites

1.  **Lottie Player Library:** You need the `lottie.js` (or `lottie.min.js`) library. This can be included from a CDN or hosted locally.
2.  **Lottie Animation JSON Data:** The animation itself is defined in a JSON file/object.
3.  **(Optional) Image Assets:** If your Lottie animation uses raster images (like the example provided), ensure these images are accessible in a path relative to your HTML (e.g., an `images/` folder).

## Steps to Recreate

### 1. HTML Structure

Set up the basic HTML. You'll need:
*   A container `div` for the Lottie animation.
*   A `div` for the visual scroll indicator.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scroll-Controlled Lottie</title>
    <link rel="stylesheet" href="style.css"> <!-- Link to your CSS file -->
</head>
<body>
    <div id="lottie"></div>
    <div id="scroll-indicator"></div>

    <!-- Lottie Player (replace with your preferred method of inclusion) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.12.2/lottie.min.js"></script>
    <script src="script.js"></script> <!-- Link to your JavaScript file -->
</body>
</html>
```

### 2. CSS Styling (`style.css`)

Add basic styles for the body, Lottie container, and scroll indicator.

```css
body {
    background-color: #ffffff; /* Or your desired background */
    margin: 0;
    height: 100vh; /* Use viewport height */
    overflow: hidden; /* Prevent native page scrollbars */
}

#lottie {
    background-color: #ffffff; /* Match body or desired container background */
    width: 100%;
    height: 100%;
    display: block;
    overflow: hidden;
    transform: translate3d(0,0,0); /* Potential performance boost */
    text-align: center; /* If needed for content within Lottie (less common) */
    opacity: 1;
}

#scroll-indicator {
    position: fixed;
    right: 20px;
    top: 10vh; /* Initial position, will be updated by JS */
    width: 8px;
    height: 8px;
    background-color: rgba(0, 0, 0, 0.5);
    border-radius: 50%;
    z-index: 1000;
    pointer-events: none; /* So it doesn't interfere with other interactions */
}
```

### 3. Include Lottie Player

As shown in the HTML structure, include the Lottie player library. You can download it and host it locally or use a CDN like `cdnjs`.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/bodymovin/5.12.2/lottie.min.js"></script>
```
*(The example provided embedded the entire library, which is generally not recommended for production due to page load impact. Linking to a CDN or a local file is better.)*

### 4. JavaScript Implementation (`script.js`)

This is where the core logic resides.

```javascript
window.addEventListener('DOMContentLoaded', function() {
    // 1. Define your Lottie Animation Data
    // This can be loaded from a .json file or embedded directly as in the example
    var animationData = {
        "v": "5.12.1", // Lottie version
        "fr": 30,      // Frame rate
        "ip": 0,       // In-point (start frame)
        "op": 520,     // Out-point (end frame)
        "w": 1920,     // Width
        "h": 1080,     // Height
        "nm": "YourAnimationName",
        "ddd": 0,
        "assets": [
            // Asset definitions (e.g., image sequences)
            // For image sequences, 'u' is the directory and 'p' is the image name pattern.
            // Example from provided code:
            // { "id": "58623572_[00000-00519].jpg_0", "w": 1920, "h": 1080, "t": "seq", "u": "images/", "p": "58623572__00000-00519_.jpg", "e": 0 },
            // ... more assets
        ],
        "layers": [
            // Layer definitions
            // Example from provided code (image sequence layer):
            // { "ddd": 0, "ind": 1, "ty": 0, "nm": "Image_Sequence_Layer", "cl": "jpg", "refId": "sequence_0", ... }
        ],
        "markers": [],
        "props": {}
    };
    // IMPORTANT: Replace the above with YOUR actual Lottie JSON data.
    // If your animation has an 'images/' subfolder for assets, make sure it exists
    // and the paths in `animationData.assets[n].u` and `animationData.assets[n].p` are correct.

    // 2. Set up Lottie Parameters
    var params = {
        container: document.getElementById('lottie'), // Required: a DOM element
        renderer: 'svg', // 'svg', 'canvas', or 'html'
        loop: false,     // Animation will not loop
        autoplay: false, // Animation will not play on load
        animationData: animationData, // Required: the animation data
        rendererSettings: {
            preserveAspectRatio: 'xMidYMid slice' // Adjust as needed
            // progressiveLoad: true, // Can be useful for large animations
        }
    };

    var anim; // To store the animation instance

    // 3. Load the Animation
    anim = lottie.loadAnimation(params);

    // 4. Scroll/Touch/Keyboard-based Animation Control
    anim.addEventListener('DOMLoaded', function () {
        let currentFrame = 0;
        let targetFrame = 0;
        // Adjust for sensitivity: smaller means less sensitive (more scroll/swipe needed)
        const scrollSensitivityFactor = 0.2;
        // Adjust for smoothness: 0.1 is quite smooth, closer to 1 is faster/jumpier
        const animationSmoothness = 0.1;

        let isAnimating = false; // Flag to check if animation loop is active
        let rafId = null;        // To store requestAnimationFrame ID

        const scrollIndicator = document.getElementById('scroll-indicator');
        const indicatorMinTopVh = 10; // Start position in vh
        const indicatorMaxTopVh = 90; // End position in vh
        const indicatorTravelRangeVh = indicatorMaxTopVh - indicatorMinTopVh;

        // Initialize animation at the first frame and update indicator
        anim.goToAndStop(currentFrame, true);
        updateScrollIndicator();

        function updateScrollIndicator() {
            if (scrollIndicator && anim.totalFrames > 1) {
                const progress = currentFrame / (anim.totalFrames - 1);
                const indicatorTop = indicatorMinTopVh + (progress * indicatorTravelRangeVh);
                scrollIndicator.style.top = indicatorTop + 'vh';
            }
        }

        function smoothAnimateToTarget() {
            // If close enough to the target, snap and stop
            if (Math.abs(targetFrame - currentFrame) < 0.1) {
                currentFrame = targetFrame;
                anim.goToAndStop(Math.round(currentFrame), true);
                updateScrollIndicator();
                isAnimating = false;
                rafId = null;
                return;
            }

            // Interpolate currentFrame towards targetFrame
            currentFrame += (targetFrame - currentFrame) * animationSmoothness;
            anim.goToAndStop(Math.round(currentFrame), true);
            updateScrollIndicator();

            // Continue animation loop
            rafId = requestAnimationFrame(smoothAnimateToTarget);
        }

        function handleScroll(delta) {
            targetFrame += delta * scrollSensitivityFactor;

            // Clamp targetFrame to animation bounds
            if (targetFrame < 0) {
                targetFrame = 0;
            } else if (targetFrame >= anim.totalFrames) {
                targetFrame = anim.totalFrames - 1;
            }

            // Start animation loop if not already running
            if (!isAnimating) {
                isAnimating = true;
                rafId = requestAnimationFrame(smoothAnimateToTarget);
            }
        }

        // Mouse Wheel Event
        window.addEventListener('wheel', function (event) {
            event.preventDefault(); // Prevent default page scrolling
            handleScroll(event.deltaY);
        }, { passive: false });

        // Touch Event Handling
        let lastTouchY = 0;
        let isTouching = false;

        window.addEventListener('touchstart', function(event) {
            if (event.touches.length === 1) {
                lastTouchY = event.touches[0].clientY;
                isTouching = true;
            }
        }, { passive: true });

        window.addEventListener('touchmove', function(event) {
            if (!isTouching || event.touches.length !== 1) {
                return;
            }
            const touchY = event.touches[0].clientY;
            const deltaY = lastTouchY - touchY; // Inverted for natural swipe direction
            lastTouchY = touchY;
            handleScroll(deltaY);
        }, { passive: true });

        window.addEventListener('touchend', function() {
            isTouching = false;
        });

        window.addEventListener('touchcancel', function() {
            isTouching = false;
        });

        // Keyboard Event Handling (Arrow Keys)
        const keyboardScrollStep = 30; // Adjust step for keyboard, similar to a small wheel delta

        window.addEventListener('keydown', function(event) {
            let delta = 0;
            if (event.key === 'ArrowDown') {
                delta = keyboardScrollStep;
                event.preventDefault(); // Prevent default page scroll
            } else if (event.key === 'ArrowUp') {
                delta = -keyboardScrollStep;
                event.preventDefault(); // Prevent default page scroll
            }

            if (delta !== 0) {
                handleScroll(delta);
            }
        });
    });
});
```

## Key Concepts/Variables

*   **`lottie.loadAnimation(params)`**: The core Lottie function to load and initialize an animation.
    *   `container`: The DOM element where the animation will render.
    *   `animationData`: Your animation's JSON data.
    *   `renderer`: `'svg'`, `'canvas'`, or `'html'`. SVG is often preferred for scalability and sharpness.
    *   `loop: false`: Prevents the animation from looping automatically.
    *   `autoplay: false`: Prevents the animation from playing as soon as it's loaded.
*   **`anim.goToAndStop(frame, isFrame)`**:
    *   `frame`: The frame number to go to.
    *   `isFrame: true`: Indicates that the first argument is a frame number (not time).
*   **`anim.totalFrames`**: Property of the Lottie animation instance that gives the total number of frames.
*   **`requestAnimationFrame(callback)`**: Used to create smooth animations by synchronizing with the browser's repaint cycle.
*   **Event Listeners**:
    *   `DOMContentLoaded`: Ensures the DOM is ready before executing JavaScript.
    *   `anim.addEventListener('DOMLoaded', callback)`: Ensures Lottie has loaded its internal structure before attempting to control it.
    *   `wheel`: For mouse scroll.
    *   `touchstart`, `touchmove`, `touchend`, `touchcancel`: For touch-based scrolling on mobile devices.
    *   `keydown`: For keyboard (ArrowUp/Down) navigation.
*   **`scrollSensitivityFactor`**: A multiplier to control how much a single scroll/swipe/keypress action moves the animation.
    *   _Smaller value_ = less sensitive (more scrolling/swiping needed for the same animation progress).
    *   _Larger value_ = more sensitive.
*   **`animationSmoothness`**: A value between 0 and 1 that controls the easing of the animation towards the `targetFrame`.
    *   _Closer to 0_ (e.g., 0.05) = very smooth, slower to reach the target.
    *   _Closer to 1_ (e.g., 0.5 or 1) = faster, more direct/jumpy transition.

## Customization

*   **Animation Data:** Replace the placeholder `animationData` with your actual Lottie JSON. If using external image assets, ensure the `u` (path) and `p` (filename) in the `assets` array of your JSON are correct and point to an accessible `images/` folder (or the folder you've configured).
*   **Sensitivity and Smoothness:** Adjust `scrollSensitivityFactor` and `animationSmoothness` in the JavaScript to fine-tune the scroll/swipe responsiveness and animation feel.
*   **Scroll Indicator:** Modify the CSS for `#scroll-indicator` or its JavaScript update logic (`updateScrollIndicator` function) to change its appearance or behavior.
*   **Renderer:** You can change `renderer: 'svg'` to `'canvas'` or `'html'` depending on your needs and the animation's complexity.
*   **Keyboard Step:** Modify `keyboardScrollStep` to change how much each arrow key press affects the animation.

This guide should provide a solid foundation for recreating the scroll-controlled Lottie functionality.
```
