# PRESTOplay interstitial player guide

The PRESTOplay interstitial player API is an experimental API aimed at
playback of HLS content with interstitials.

This guide is here to help you experiment with the API until it gets finalized
and until it's officially released with proper documentation.

## Demo app

`index.html` contains a demo app demonstrating the use of this API. Start
the demo app:

```sh
npm run demo
```

## API docs

The [typings.d.ts](./typings.d.ts) file contains TS typings for the PRESTOplay
SDK. In this file look for `namespace interstitial` on line 6442, this namespace
contains the whole API of the interstitial player.

The main class is `clpp.interstitial.Player`.


