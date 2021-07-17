---
title: 'Easy Service Workers with Workbox.js'
tldr: "Service Workers don't have to be hard. Use Workbox.js" 
date: 'September 18 2018'
preamble: "Service Workers are possibly the most revolutionary Browser API we have ever received. They unlock a whole realm of possiblities: offline apps, push notifications, background sync, proxying requests, and so much more. But where there's power, there's complexity."
---

### Service workers don't have to be hard.

Service Workers are possibly the most revolutionary Browser API we have ever received. They unlock a whole realm of possiblities: offline apps, push notifications, background sync, proxying requests, and so much more. **But where there's power, there's complexity.**

Service Workers come with a confusing lifecycle, a large API surface, and a lot of decisions around caching left up to the developer. Do you want an offline first cache? What about stale while revalidate? How do you even get your app up to update when the new Service Worker installs?

There's a lot of difficulty around Service Workers. That's because they're designed to cover a large set of use-cases. Browser APIs are made with this philosophy: create low level APIs and allow developers to build on top of them. **We need tools to make things easier, that's what [Workbox](https://developers.google.com/web/tools/workbox/) does**.

### Workbox is a set of tools for creating Service Workers

[Workbox](https://developers.google.com/web/tools/workbox/) is an official library from the Google Web team. Workbox provides a modular set of tools for building offline apps. You can let Workbox create your entire Service Worker or just use the parts you need. Workbox provides:

*   Precaching
*   Runtime caching
*   Caching strategies
*   Background sync

Don't know what these terms mean? Let's break a few of them down.

### Precaching sets your app up to work offline

Precaching works like an app install. When the Browser installs the Service Worker it begins to cache assets needed for the app to work offline. After the assets are downloaded the user could put their phone in airplane mode and the browser will serve these assets instead of the dino game.

Worbox provides a tool to generate a Service Worker to cache a list of assets. The way Workbox handles cache changes is fantastic. Workbox generates a hash for each file, when the file's contents changes the hash changes. This hash change allows Workbox to invalidate each asset individually. Thank the lord it does, because doing this by hand is difficult.

**What kind of assets should you precache?** This is a tricky question. You should precache vital assets for an offline experience. Asset like `index.html`, `styles.css`, and important smaller images. But you shouldn't precache your entire blog or newspaper.

### Runtime caching offlines assets as they are requested

Runtime caching saves assets on the fly. Preaching is for vital assets. Runtime caching is for dynamic assets. This works well for apps that follow an "App Shell" model. The core assets are precached but the dynamic content is cached as it appears. When the user goes offline, the core app and everything they have seen is available offline.

**What kind of assets should you runtime cache?** Images, json requests, or whatever appears dynamically in your app.

### Caching strategies determines network behavior

Caching strategies affects what the user sees on the screen, and how assets are updated. A network first cache gives the user the most fresh content, but could lead to redundant requests. An offline first cache reduces requests but could lead to stale content. Workbox provides an entire set of strategies so you can use what works best for your app.

This covers some of the terms. Let's see how we can put them to work.

### Workbox can create a Service Worker for you

Workbox comes with a CLI to generate a Service Worker based on your configuration. You specify what file types to precache in what folders.

Install it locally or globally.

```bash npm i -g workbox-cli ``` ```bash workbox wizard ```

Workbox will take you through a set of questions to help set up your Service Worker. It asks where to look for files and what kind of files to precache. After the questions are done Workbox creates the configuration, not the Service Worker. From here you run a command that generates the worker.

```bash workbox generateSW workbox-config.js ```

That command generates a `sw.js` file. That's it. You have a worker. Well, you need to register it, so you're not done yet. Register your worker in your `index.html`.

```html  
<script>
// Check that service workers are registered
if ('serviceWorker' in navigator) {
  // Use the window load event to keep the page load performant
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js');
  });
}
</script>
```

Now you have a working Service Worker and you didn't have to write any crazy worker code. Workbox will even handle the cache invaldation each time a file changes. **This is just precaching, you can also configure runtime caching.**

### Workbox can configure runtime caching

Workbox has a configuration section for runtime caching. Let's say you want to cache images as the appear in a dynamic feed.

```js 
// Define runtime caching rules.
runtimeCaching: [{
  // Match any request ends with .png, .jpg, .jpeg or .svg.
  urlPattern: /\.(?:png|jpg|jpeg|svg)$/,
  
  // Apply a cache-first strategy.
  handler: 'cacheFirst',
  
  options: {
    // Use a custom cache name.
    cacheName: 'images',

    // Only cache 50 images.
    expiration: {
      maxEntries: 50,
    },
  },
```

The configuration above creates a single runtime caching rule. It has a `urlPattern` that matches common image extensions, and uses a `handler` or a strategy of `'cacheFirst'`. This means when the browser requests images the Service Worker intercepts the request and first looks in the cache. If the image isn't in the cache it goes to the network.

The last part of the configuration allows you to set custom `options`. In this case I have supplied a custom cache name and supplied a `maxEntries` of 50 to make sure the cache doesn't get too large.

This is just one small example of runtime caching. You can use it to cache requests that come over different domains as well. Let's say you're authenticating users with Google and displaying their profile picture. That picture is not located on your domain and it appears dynamically. You can specify a `urlPattern` to match Google's domain so profile pictures load when offline.

### Workbox has a Webpack plugin

The CLI module is easy to use and works across multiple environments. But Workbox has just the plugin for you if you're in the Webpack ecoystem.

This plugin makes it easy to add Workbox support to your webpack app.

```bash 
npm i workbox-webpack-plugin --save-dev 
```

Once it's installed you can set up the plugin in your `webpack.config.js`.

```js  
// Inside of webpack.config.js:
const WorkboxPlugin = require('workbox-webpack-plugin');

module.exports = {
  // Other webpack config...

  plugins: [
    // Other plugins...

    new WorkboxPlugin.GenerateSW()
  ]
};
```

From there you can supply the same config format used in the `workbox-config.js`.

```js
new WorkboxPlugin.GenerateSW({
  // Exclude images from the precache
  exclude: [/\.(?:png|jpg|jpeg|svg)$/],

  // Define runtime caching rules.
  runtimeCaching: [{
    // Match any request ends with .png, .jpg, .jpeg or .svg.
    urlPattern: /\.(?:png|jpg|jpeg|svg)$/,

    // Apply a cache-first strategy.
    handler: 'cacheFirst',

    options: {
      // Use a custom cache name.
      cacheName: 'images',

      // Only cache 10 images.
      expiration: {
        maxEntries: 10,
      },
    },
  }],
})
```

When you build your app with Webpack, workbox will take care of setting up the precaching and runtime caching rules.

### Workbox is for offline apps

While Workbox is a fantastic tool, make sure that it fits your use-case. Workbox is for offline apps. If you don't need complex offline behavior then don't burden yourself with Service Worker logic. However, if you do need offline, it's hard to beat the simplicity of Workbox.