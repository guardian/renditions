# Renditions

The communication layer between Editions and editions-rendering.

## Overview

### The raw communication

React Native app communicates with its WebView in two different ways, based on the direction:

- _From React Native to the WebView:_ The native app has access to the global context of the WebView and you can inject JavaScript code either as a prop for code you want executed for example on _content loaded_ or with an [`injectJavaScript` method](https://github.com/react-native-webview/react-native-webview/blob/master/docs/Guide.md#communicating-between-js-and-native) which is what this library uses.
- _From WebView to React Native_ The web context can use `postMessage` to send data to React Native

You can learn more about this in the [official guide](https://github.com/react-native-webview/react-native-webview/blob/master/docs/Guide.md#communicating-between-js-and-native).

### The problems

The first problem is the fact these two ways of communicating are different in the ways the developer would use them, the methods are very different. One can execute any JavaScript string and the other can only send stringified data.
The second problem being we want our hydrated React components to be able to listen to messages from the native layer. We want these messages to be typed and consitent between the two layers.

### The solution

Something like [Bridget](https://github.com/guardian/bridget) would be very heavy in this instance with additional set of its own challenges and cost.
The solution in this repo is very simple in a way that it is a bunch of types and helper methods that could be used by both editions-rendering and Editions.
The functions `pingEditionsNative` and `pingEditionsRenderingJsString` both take the same type, a `Message` and the rest is abstracted away from the developer.

Under the hood, all this does is either use `postMessage` to the native app with the right message type, or trigger a Custom Event `editionsPing` that the web context, be it the hydrated React components or vanilla JavaScript can listen to.

## Usage

You can install the library inside your project by adding the appropriate git reference in your `package.json`

```jsonc
{
  "dependencies": {
  // ...,
  "@guardian/renditions": "git+https://github.com/guardian/renditions.git#0.1.0"
  }
}
```

To use inside editions-rendering, make use of the `pingEditionsNative` function:

```typescript
import {
	MessageKind,
	pingEditionsNative,
} from '@guardian/renditions';

pingEditionsNative({ kind: MessageKind.Share })
```

To use inside the Editions app, make use of the `pingEditionsRenderingJsString` function:

```typescript
import {
  MessageKind,
  Platform,
  pingEditionsRenderingJsString
} from '@guardian/renditions';

... // webView code
const platformMessage: PlatformMessage = {
  kind: MessageKind.Platform;
  value: Platform.IOS;
};

webView.injectJavaScript(pingEditionsRenderingJsString(platformMessage));

```
