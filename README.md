Demo of an Aerobatic app using SoundCloud client JavaScript authentication.

(https://developers.soundcloud.com/docs#authentication)

The sample code uses `window.opener` to invoke functions on the parent window. But the Aerobatic simulator will serve the `callback.html` popup off localhost resulting in a cross origin browser security error. The workaround (and arguably a better approach anyway) is to use [window.postMessage] which is capable of sending messages across domains.

## Popup JavaScript

Here's the modified contents of the `callback.html`.
~~~html
<script>
  function callbackToParent() {
    var eventData = {
      event: 'SCConnect',
      location: window.location.href
    };

    window.opener.postMessage(JSON.stringify(eventData), '*');
    setTimeout(this.close, 100);
  }
</script>
<body onload="callbackToParent()">
~~~

## Main Window JavaScript

~~~js
// initialize client with app credentials
SC.initialize({
  // Use a different clientId for debug and release builds. You could potentially get
  // around doing this by deploying once and always specify the release callback,
  // i.e. http://appname.aerobaticapp.com/callback.html
  client_id: __config__.buildType === 'debug' ? '{{debug url client_key}}' : '{{release url client_key}}',
  redirect_uri: window.location.protocol + '//' + location.hostname + '/callback.html'
});

function onSoundCloudConnect() {
  SC.get('/me', function(me) {
    alert('Hello, ' + me.username);
  });
}

// initiate auth popup
SC.connect({});

// Listen for message event from the popup
window.addEventListener("message", function(event) {
  var eventData = JSON.parse(event.data);

  if (eventData.event === 'SCConnect') {
    var loc = new SC.URI(eventData.location, {decodeQuery: true, decodeFragment: true});

    SC.accessToken(loc.fragment.access_token);
    onSoundCloudConnect();
  }
}, false);
~~~

### Live Demo

http://soundcloud-test.aerobaticapp.com/
