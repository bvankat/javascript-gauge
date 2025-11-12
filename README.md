# Javascript Speedometer Gauge

A small, self-contained speedometer-style gauge built with custom SVG assets and plain Javascript. Created for [nebrasketball.info] as a replacement for the standard Google Charts gauge. Inspired by the jittery chaos of the New York Times election night projection needle.

Demo allows you to control speed and amplitude of the needle's bounce. Light and dark dial options available.


## Files

- `index.html` — working demo and control panel. Loads the SVG dial and needle from `src/svg/` and wires the interactive controls.
- `src/svg/` — contains the gauge SVG assets (light/dark folders). The demo expects a dial SVG and a needle SVG inside the chosen theme folder.

## How it works

1. The dial (background) SVG is static and placed behind the needle.
2. The needle SVG is embedded separately inside a container element centered over the dial.
3. JavaScript computes an angle for the needle based on a value in the range 0..100 and applies a CSS transform (`translate(-50%,-50%) rotate(angle)` ) to the needle container.
4. A CSS transition is used on the needle container so changes in the transform are smoothly animated. The demo applies and updates this transition from JavaScript so you can tune the smoothing at runtime.
5. An optional bounce/jitter feature periodically nudges the needle a few points around the target value for a subtle, alive effect (configurable amplitude and frequency).

## Controls / Settings (demo)

The demo (`index.html`) exposes the following controls:

- Main slider (Speed 0..100) — sets the gauge value. This is the primary target value.
- Buttons — quick set shortcuts: Reset (0), Half (50), Max (100) and Animate (sweeps the needle programmatically).
- Bounce Adjustment panel:
	- Smoothness (ms): transition duration for needle rotation. Larger values = slower, smoother movement.
	- Bounce Interval (ms): how often the demo applies a small random jitter around the target value.
	- Bounce Amplitude (±): the maximum jitter amount (in value units) applied to the target when bouncing.

All sliders show their current numeric values so you can tune and observe the visual effect immediately.

## JavaScript API (functions & variables)

The demo exposes, in `index.html`, the following functions and configurable variables that you can reuse or wire into your own app logic:

- updateNeedle(value, skipSlider = false)
	- value: number (0..100)
	- skipSlider: when true the function will not update the UI slider (used by the bounce so the slider doesn't jump)

- setSpeed(value)
	- Sets the demo's base target value (updates slider and needle)

- animateSpeed()
	- Runs a simple programmatic sweep of the needle (demo-only) and temporarily pauses the bounce while animating

- startBounce(), stopBounce() (demo helper)
	- start/restart the bounce timer. In the demo the bounce starts automatically.

- Configurable variables you can set from your code:
	- `bounceSmoothMs` (ms): duration of transform transition for needle smoothing.
	- `bounceTimingFn` (CSS timing function): easing curve for the transition.
	- `bounceIntervalMs` (ms): frequency of the jitter.
	- `bounceAmplitude` (number): max jitter around the target value.
	- `config.minAngle` and `config.maxAngle`: the angles in degrees that map to values 0 and 100 respectively (useful if you swap SVGs with different orientation).

If you embed the demo script into another page, call `updateNeedle()` with your values to control the needle.

## Customization

- SVG assets: replace the dial and needle SVGs in `src/svg/light/` or `src/svg/dark/` with your own images. Ensure the needle graphic is centered so the rotation anchor (the container center) aligns with the needle pivot.
- Angle mapping: adjust `config.minAngle` and `config.maxAngle` in the script to match the needle starting and ending rotation for your SVG artwork.
- Smoothness & easing: change `bounceSmoothMs` and `bounceTimingFn` to tweak how the needle interpolates between values. The demo uses a cubic-bezier curve tuned for subtle bounces, but `ease`, `ease-in-out` or custom curves are all supported.
- Bounce behavior: the demo's bounce is a debugging/visual effect — you can disable it by stopping the bounce timer or setting `bounceAmplitude = 0`.
- Remove controls: when you move the gauge into production, you can delete the adjustment panel and hardcode the values you prefer. Keep only the minimal functions you need (e.g., `updateNeedle()` and `setSpeed()`).

## Integration notes

- Include the needle container and dial container markup in your page similar to `index.html`. The needle container must be centered on the dial for correct rotation.
- Ensure your CSS preserves `transform-origin: center center` and applies the transform to the needle container (not necessarily the needle SVG itself).
- If you use multiple gauges on one page, keep independent state for each gauge (baseValue, bounce settings, interval id, etc.). Consider wrapping the gauge code into a small class or factory function that returns an API object per gauge instance.

## Example: basic usage

Call `updateNeedle(72)` to point the needle to value 72. For debugging you can tweak the smoothing and bounce settings from the demo panel.

## Troubleshooting

- Needle rotates incorrectly (pivot off-center): check the needle SVG artwork; the pivot should be located at the logical center where it will rotate.
- Needle moves but the slider doesn't update: the demo intentionally keeps the slider at the base value and uses `skipSlider=true` for bounce updates so the slider only moves when the user changes the value.
- Animation feels jumpy: increase `bounceSmoothMs` or lower `bounceIntervalMs` until the movement looks natural, or use a smoother timing function.

## Embedding example (integrate on another page)

This example shows the minimal markup and JS you need to reuse the gauge on another page. The demo keeps all logic inline; for production you may prefer to extract the JS into a separate file and expose a small API (for example, `createGauge(containerEl, options)`).

Minimal HTML (copy the objects and a value display into your page):

```html
<!-- gauge container -->
<div class="gauge-wrapper">
	<object class="gauge-background" id="gaugeDial" data="svg/light/gauge-dial.svg" type="image/svg+xml"></object>
	<div class="gauge-needle-container">
		<object id="gaugeNeedle" data="svg/light/needle.svg" type="image/svg+xml"></object>
	</div>
	<div id="gaugeValue" class="value-display light">50</div>
</div>
```

Tiny integration script (example only — copy/update the functions you need from the demo):

```html
<script>
	// angle mapping must match your SVG artwork
	const cfg = { minAngle: -215.2, maxAngle: 34.2, minValue: 0, maxValue: 100 };

	// helper to set the needle (similar to demo)
	function updateNeedleSimple(value) {
		value = Math.max(cfg.minValue, Math.min(cfg.maxValue, value));
		const range = cfg.maxAngle - cfg.minAngle;
		const fraction = (value - cfg.minValue) / (cfg.maxValue - cfg.minValue);
		const angle = cfg.minAngle + fraction * range;
		const container = document.querySelector('.gauge-needle-container');
		const valueEl = document.getElementById('gaugeValue');
		if (container) container.style.transform = `translate(-50%, -50%) rotate(${angle}deg)`;
		if (valueEl) valueEl.textContent = value;
	}

	// Example: update to 72
	updateNeedleSimple(72);
</script>
```

Notes:

- Copy any CSS that positions the `.gauge-wrapper`, keeps the needle container centered, and sets `transform-origin: center center` for the needle container.
- If you want the demo's bounce behavior, also copy `startBounce()` and the bounce configuration variables and wire them to your page.
- If you extract the JS into a module, return an API that includes `updateNeedle()` and `setSpeed()` for consumers to call.