---
title: "JavaScript WebView and iOS"
author: "Vivek Kalyan"
date: "20/03/2018"
---

I am in the process of building a webGL based AR application. An iOS application opens the webGL-powered website using `WKWebView`. The iOS application sends the website information about the device - its rotation, position etc. The browser also sends the device instructions, eg. to look for a plane using ARkit at the touch position. To do this, the javascript and iOS applications need to communicate with each other.

### Javascript to iOS 
To send data from javascript to iOS, we need to make use of WebKit's message handlers. In javascript, this can be invoked pretty easily.

```javascript
// javascript
data = {x: 1, y: 2}
window.webkit.messageHandlers.eventHandler.postMessage(data);
```


On the iOS side, we set up a handler which will be activated if it receives a message from the javascript side

```swift
// iOS
let webConfiguration = WKWebViewConfiguration()
let contentController = WKUserContentController()

contentController.add(self, name: "eventHandler")
webConfiguration.userContentController = contentController;

webView = WKWebView(configuration: webConfiguration) // plus any other settings

func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
    if (message.name == "eventHandler"){
        let data = message.body as! NSDictionary
        // use data
    }
}
```

### iOS to Javascript

One possible use case for sending data from iOS to webview is to incorporate sensor data with the scene. For example, in my case, I was passing the light intensity data from the camera to control the lighting of my AR scene. This resulted in a more realistic experience.

To do this, we need to prepare the webview with a javascript function that allows us to update the values in the scene.

```javascript
// javascript
ambientLightUpdate(ambient) {
    // make use of ambient value which is sent to alter scene
}

window.ambientLightUpdate = ambientLightUpdate;
```

After doing that, its a matter of calling the function on the iOS side with the appropriate value.

```swift
// iOS
let ambient:Float = Float() // get value from camera sensor

self.webView.evaluateJavaScript("ambientLightUpdate('\(ambient)');", completionHandler: { (result, error) in
    // handle errors accordingly
});
```


### Debugging

One really painful part of developing was not being able to have access to the javascript console when the webview was opened in iOS. Therefore I made use of message handlers to redirect all calls to `console.log` to iOS.

```javascript
// javascript
if (window.webkit) {
  console.log = function(msg) {
    if (typeof msg == 'object') {
      msg = objToString(msg);
    }
    window.webkit.messageHandlers.logHandler.postMessage(msg);
  };
}

function objToString(obj) {
  var str = '';
  for (var p in obj) {
    if (obj.hasOwnProperty(p)) {
      str += p + '::' + obj[p] + '\n';
    }
  }
  return str;
}
```

Objects are converted to strings, because there is an obscure bug where trying to `print(obj)` would result in the javascript script failing silently. The iOS setup is similar to before.

```swift
// iOS
let webConfiguration = WKWebViewConfiguration()
let contentController = WKUserContentController()

contentController.add(self, name: "logHandler")
webConfiguration.userContentController = contentController;

webView = WKWebView(configuration: webConfiguration) // plus any other settings

func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
    if (message.name == "logHandler"){
        print(message.body)
    }
}
```

### Keong Saik Road

On March 17-18, we held an event in collaboration with Urban Design Festival. We transformed an unused stretch of road into a futuristic Space Age themed street. The public got a chance to use our AR experience which incorporated interactivity in the form of shooting drones. Here is a post event summary video:

<div class="embed-responsive embed-responsive-16by9">
  <iframe class="embed-responsive-item" width="560" height="315"
    src="https://www.youtube.com/embed/736C2mXavNo" frameborder="0"
    allowfullscreen=""></iframe>
</div>

To find out more, check out our social media!

[Website](https://www.outpost.social/)

[Facebook](https://www.facebook.com/outpost.social/)

[Instagram](https://www.instagram.com/outpost.social/)
