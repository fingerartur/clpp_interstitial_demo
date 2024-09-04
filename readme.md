# PRESTOplay Interstitial Player Guide

The PRESTOplay interstitial player API is part of the [@castlabs/prestoplay SDK](https://www.npmjs.com/package/@castlabs/prestoplay) and it is an experimental API aimed at
playback of HLS content containing interstitials. This guide is here to
help you experiment with the API until it gets finalized.

## Demo App

[index.html](./index.html) contains a demo app demonstrating the use of this API.
Start the demo app:

```sh
npm run demo
```

## API Docs

The [typings.d.ts](./typings.d.ts) file contains TS typings for the PRESTOplay
SDK. In this file look for [namespace interstitial] which contains the whole
API of the interstitial player.

The main class is [clpp.interstitial.Player].

## Limitations

- Live streams are not supported.
- Platforms that do not support multiple video elements are not supported (e.g.
  older smart TVs).
- Certain parts of the spec of HLS interstitials are not supported,
  see comments of [clpp.interstitial.Player].

## Usage Manual

The interstitial player is based on multi-controller architecture where two
video elements and two instances of [clpp.Player] are used to perform
transitions between primary and interstitial content.
This is reflected in the API.

Start by instantiating the interstitial player and attaching it to a div element.
Video elements will be created automatically as descendants of this anchor element.

```html
<div id="anchor"></div>
```
```js
const anchorEl = document.getElementById('anchor');
const player = new clpp.interstitial.Player({
  config: {
    license: '...', // castlabs license key
    viewerId: 'user-1',
  },
  anchorEl,
});
```

Play the HLS asset.

```js
await player.load({
  source: 'https://.../asset.m3u8'
});
```

Or load the HLS asset paused and then unpause it later.

```js
await player.loadPaused({
  source: 'https://.../asset.m3u8'
});
await player.unpause();
```

Stop playback by unloading the currently playing asset.

```js
await player.unload();
```

When the player is no longer needed destroy it.

```js
await player.destroy();
```

Get timeline cues representing the interstitial date ranges found in
the HLS playlist.

```js
const cues = player.getCues();
cues.forEach(cue => {
  console.log(`Cue ${cue.cueId} at ${cue.startTime} s`);
});
```

Get the underlying [clpp.Player] instance in order to draw a timeline.

```js
const p = player.getCurrentPlayer();
if (!p) return;
const position = p.getPosition();
const duration = p.getDuration();
// Draw the timeline
```

Seek using the underlying [clpp.Player] instance.

```js
player.getCurrentPlayer()?.seek(100);
```

Pause and un-pause playback using the underlying [clpp.Player] instance.

```js
await player.getCurrentPlayer()?.pause();
await player.getCurrentPlayer()?.play();
```

### Events

For a full list of interstitial player events see [clpp.interstitial.Event].

The `item-changed` event is emitted when playback starts or whenever playback
transitions from primary to interstitial content or vice versa or between
different interstitial items.

```js
player.on('item-changed', (event) => {
  // Active item has changed
});
```

This is a good place to redraw the timeline.

```js
player.on('item-changed', (event) => {
  const p = player.getCurrentPlayer();
  const position = p.getPosition();
  const duration = p.getDuration();
  // Redraw the timeline
  // ...
});
```

It is also a good place to detach and attach any listeners to the underlying
[clpp.Player] instance.

```js
player.on('item-changed', (event) => {
  // Detach the existing listeners
  // ...
  const p = player.getCurrentPlayer();
  // Attach listeners to the clpp.Player instance
  p.addEventListener('timeupdate', () => { ... });
  p.addEventListener(clpp.events.STATE_CHANGED, () => { ... });
  p.addEventListener(clpp.events.BUFFERING_STARTED, () => { ... });
});
```

It is also a good place to set attributes of the underlying video element
if needed. E.g. to display default video controls:

```js
player.on('item-changed', (event) => {
  const p = player.getCurrentPlayer();
  const videoEl = p.getSurface().getMedia();
  videoEl.controls = true;
});
```

## Plugins

### Yospace

Import Yospace Ad Management SDK version 3.6.11.

```html
<script src="admanagement-sdk-3.6.11.min.js"></script>
```

Enable the Yospace plugin.

```js
const player = new clpp.interstitial.Player({
  config: {
    license: '...', // castlabs license key
    viewerId: 'user-1',
  },
  anchorEl,
  yospace: { enabled: true },
});
```

Pass a Yospace CSM URI to the player which will automatically start a Yospace
playback session and play the underlying asset.

```js
await player.load({
  source: 'https://...' // Yospace CSM URI
});
```


[namespace interstitial]: ./typings.d.ts#L6442
[clpp.interstitial.Player]: ./typings.d.ts#L6657
[clpp.interstitial.Event]: ./typings.d.ts#L6566
[clpp.Player]: https://demo.castlabs.com/#/docs?q=clpp.Player
