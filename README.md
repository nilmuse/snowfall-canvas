# snowfall-canvas

Render snowfall on an existing canvas, with control over snowflake density, size, color, and motion. It resizes with its container and reduces rendering load when frames slow down.

[![npm](https://img.shields.io/npm/v/snowfall-canvas.svg?colorB=brightgreen)](https://www.npmjs.com/package/snowfall-canvas)
[![NPM Downloads](https://img.shields.io/npm/dm/snowfall-canvas.svg?style=flat)](https://www.npmjs.com/package/snowfall-canvas)

[Demo](https://codepen.io/ux-ui/pen/zxBGXgm)

## Installation

```bash
npm install snowfall-canvas
```

## Quick start

In a browser app with a bundler, add a container and canvas:

```html
<div id="snow-wrap">
  <canvas id="snow-canvas" aria-hidden="true"></canvas>
</div>
```

Give the container a visible size and make the canvas cover it. The dark background makes the snow visible:

```css
#snow-wrap {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  background: #101820;
}

#snow-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
}
```

Run this TypeScript module after the markup exists:

```ts
import { SnowfallCanvas } from 'snowfall-canvas';

const snow = new SnowfallCanvas({
  container: 'snow-wrap',
  canvas: 'snow-canvas',
});

snow.init();
snow.start();
```

The canvas fills with falling snow. Call `snow.destroy()` before removing the container or canvas.

## API

The package exports the `SnowfallCanvas` class and `defaultConfig`:

```ts
import { SnowfallCanvas, defaultConfig } from 'snowfall-canvas';
```

`defaultConfig` contains the `SnowConfig` defaults listed below. The package also exports the types `SnowConfig`, `SnowConfigInput`, `SnowfallCanvasOptions`, and `Range`.

### Options

Pass these options to `new SnowfallCanvas(options)`. String values are DOM ids, without a leading `#`:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `container` | `HTMLElement \| string` | Required | Container element or its `id`. Its size determines the canvas drawing-buffer size. |
| `canvas` | `HTMLCanvasElement \| string` | Required | Canvas element or its `id`. |
| `config` | `Partial<SnowConfig>` | `undefined` | Override any of the configuration values below. Omitted values use `defaultConfig`. |

#### Snow configuration

| Option         | Type               | Default              | Description                                                              |
|:---------------|:-------------------|:---------------------|:-------------------------------------------------------------------------|
| `amount`       | `number`           | `1500`               | Base particle density for a 1920×1080 area (scales with container size). |
| `maxParticles` | `number`           | `2500`               | Hard cap for total particles.                                            |
| `size`         | `[number, number]` | `[0.8, 1.5]`         | Particle size range in pixels.                                           |
| `swingSpeed`   | `[number, number]` | `[0.2, 0.8]`         | Horizontal sinusoid speed factor.                                        |
| `fallSpeed`    | `[number, number]` | `[40, 80]`           | Fall speed range in px/s.                                                |
| `amplitude`    | `[number, number]` | `[20, 45]`           | Horizontal swing amplitude range in pixels.                              |
| `color`        | `string`           | `"rgb(225,225,225)"` | Particle color (`fillStyle`).                                            |
| `dprCap`       | `number`           | `2`                  | Upper bound for `devicePixelRatio`.                                      |

As an alternative to the instance in Quick start, use the same elements with configuration overrides:

```ts
const snow = new SnowfallCanvas({
  container: 'snow-wrap',
  canvas: 'snow-canvas',
  config: {
    amount: 2000,
    maxParticles: 3000,
    size: [0.6, 1.5],
    color: 'rgba(200,200,200,1)',
  },
});

snow.init();
snow.start();
```

### Methods

| Method            | Description                                                                                         |
|:------------------|:----------------------------------------------------------------------------------------------------|
| `init()`          | Read the container size, resize the canvas drawing buffer, and subscribe to resize and visibility events. |
| `start()`         | Starts the render loop.                                                                             |
| `stop()`          | Stops the render loop.                                                                              |
| `destroy()`       | Stop the animation and remove listeners and observers. |
| `requestResize()` | Schedules a resize on the next animation frame. Useful after manual layout updates.                 |
| `setAmount(amount)` | Updates base density and re-seeds particles.                                                      |
| `setMaxParticles(max)` | Adjusts the upper limit and re-seeds particles.                                                |

## Vue 3

This component creates the animation after mount and destroys it before unmount:

```vue
<template>
  <div ref="wrap" class="wrap">
    <canvas ref="cnv" class="cnv" aria-hidden="true"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { SnowfallCanvas } from 'snowfall-canvas';

const wrap = ref<HTMLElement | null>(null);
const cnv = ref<HTMLCanvasElement | null>(null);

let snow: SnowfallCanvas | null = null;

onMounted(() => {
  if (!wrap.value || !cnv.value) return;

  snow = new SnowfallCanvas({
    container: wrap.value,
    canvas: cnv.value,
    config: {
      amount: 2000,
      maxParticles: 3000,
      size: [0.6, 1.5],
      color: 'rgba(200,200,200,1)',
    },
  });

  snow.init();
  snow.start();
});

onBeforeUnmount(() => {
  snow?.destroy();
  snow = null;
});
</script>

<style scoped>
.wrap {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  background: #101820;
}

.cnv {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
}
</style>
```

## Rendering and sizing

- The canvas drawing buffer follows the container size and current device pixel ratio, capped by `dprCap`.
- Particle density is calculated from the container area, using 1920×1080 as the base, and adjusted for DPR. The library may reduce DPR and particle count when frames take too long.
- Pauses when the tab is hidden and resumes on return.
- Uses your existing DOM and CSS; it does not add elements or styles.

## Browser support

Create the instance in the browser after its container and canvas exist. The container needs a nonzero computed width and height, and your CSS must make the canvas cover it.

The animation needs Canvas 2D and `requestAnimationFrame`. Resizing uses `ResizeObserver` when available and falls back to window resize events otherwise.

## License

MIT
