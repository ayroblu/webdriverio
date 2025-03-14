---
id: browserobject
title: The Browser Object
---

The browser object is the session instance you use to control the browser or mobile device with. If you use the WDIO test runner, you can access the WebDriver instance through the global `browser` or `driver` object.

The session is initialized by the test runner. The same goes for ending the session. This is also done by the test runner process.

Besides all commands from the [API](API.md), the `browser` object provides some more information you might be interested in during your test run:

## Get Desired Capabilities

```js
console.log(browser.sessionId) // outputs: "57b15c6ea81d0edb9e5b372da3d9ce28"
console.log(browser.capabilities)
/**
 * outputs capabilities returned by the browser driver, e.g.:
   { acceptInsecureCerts: false,
     acceptSslCerts: false,
     applicationCacheEnabled: false,
     browserConnectionEnabled: false,
     browserName: 'chrome',
     chrome:
      { chromedriverVersion: '2.40.565386 (45a059dc425e08165f9a10324bd1380cc13ca363)',
        userDataDir: '/var/folders/ns/8mj2mh0x27b_gsdddy1knnsm0000gn/T/.org.chromium.Chromium.mpJ0yc' },
     cssSelectorsEnabled: true,
     databaseEnabled: false,
     handlesAlerts: true,
     hasTouchScreen: false,
     javascriptEnabled: true,
     locationContextEnabled: true,
     mobileEmulationEnabled: false,
     nativeEvents: true,
     networkConnectionEnabled: false,
     pageLoadStrategy: 'normal',
     platform: 'Mac OS X',
     rotatable: false,
     setWindowRect: true,
     takesHeapSnapshot: true,
     takesScreenshot: true,
     unexpectedAlertBehaviour: '',
     version: '68.0.3440.106',
     webStorageEnabled: true }
 */
console.log(browser.requestedCapabilities)
/**
 * outputs original capabilities set by the user, e.g.:
 * {
 *   browserName: 'chrome'
 * }
 */
console.log(browser.commandList)
/**
 * outputs all commands assigned to this instance (browser or element), e.g.:
 * [
 *   'newSession',
 *   'deleteSession',
 *   'status',
 *   'getTimeouts',
 *   'setTimeouts',
 *   'getUrl',
 *   'navigateTo',
 *   'back',
 *   'forward',
 *   'refresh',
 *   'getTitle',
 *   ... 115 more items
 * ]
 */
```

## Custom Configurations

If using the WDIO testrunner you can always define custom configurations within your WDIO config:

```js
// wdio.conf.js
exports.config = {
    // ...
    fakeUser: 'maxmustermann',
    fakePassword: 'foobar',
    baseUrl: 'example.com',
    // ...
}
```

And access it in your tests:

```js
console.log(browser.options)
/**
 * outputs:
 * {
        port: 4444,
        protocol: 'http',
        waitforTimeout: 10000,
        waitforInterval: 250,
        logLevel: 'debug',
        connectionRetryTimeout: 120000,
        connectionRetryCount: 3,
        specs: [ 'err.js' ],
        fakeUser: 'maxmustermann', // <-- custom configuration
        fakePassword: 'foobar', // <-- custom configuration
        baseUrl: 'example.com', // <-- custom configuration
        // ...
 */

console.log(browser.options.fakeUser) // outputs: "maxmustermann"
```


```

## Mobile Flags

If you need to modify your test based on whether or not your session runs on a mobile device, you can access the mobile flags to check.

For example, given this config:

```js
// wdio.conf.js
exports.config = {
    // ...
    capabilities: {
        platformName: 'iOS',
        app: 'net.company.SafariLauncher',
        udid: '123123123123abc',
        deviceName: 'iPhone',
        // ...
    }
    // ...
}
```

You can access these flags in your test like so:

```js
// Note: `driver` is the equivalent to the `browser` object but semantically more correct
// you can choose which global variable you want to use
console.log(driver.isMobile) // outputs: true
console.log(driver.isIOS) // outputs: true
console.log(driver.isAndroid) // outputs: false
```

This can be useful if, for example, you want to define selectors in your [page objects](PageObjects.md) based on the device type, like this:

```js
// mypageobject.page.js
import Page from './page'

class LoginPage extends Page {
    // ...
    get username() {
        const selectorAndroid = 'new UiSelector().text("Cancel").className("android.widget.Button")'
        const selectorIOS = 'UIATarget.localTarget().frontMostApp().mainWindow().buttons()[0]'
        const selectorType = driver.isAndroid ? 'android' : 'ios'
        const selector = driver.isAndroid ? selectorAndroid : selectorIOS
        return $(`${selectorType}=${selector}`)
    }
    // ...
}
```

You can also use these flags to run only certain tests for certain device types:

```js
// mytest.e2e.js
describe('my test', () => {
    // ...
    // only run test with Android devices
    if (driver.isAndroid) {
        it('tests something only for Android', () => {
            // ...
        })
    }
    // ...
})
```

## Events
The browser object is an EventEmitter and a couple of events are emitted for your use cases.

Here is a list of events. Keep in mind that this is not the full list of available events yet.
Feel free to contribute to update the document by adding descriptions of more events here.

### `request.performance`
This is an event to measure WebDriver level operations. Whenever WebDriverIO requests to your WebDriver endoints,
this event will be emitted with some useful information:

- `durationMillisecond`: Time duration of the request in millisecond.
- `error`: Error object if the request failed.
- `request`: Request object. You can find url, method, headers, etc.
- `retryCount`: If it's `0`, the request was the first attempt. It will increase when WebDriverIO retries under the hood.
- `success`: Boolean to represent the request was succeeded or not. If it's `false`, `error` property will be provided as well.

An example event:
```js
Object {
  "durationMillisecond": 0.01770925521850586,
  "error": [Error: Timeout],
  "request": Object { ... },
  "retryCount": 0,
  "success": false,
},
```
