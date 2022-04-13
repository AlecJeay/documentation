## OBS WebSocket using Node.js Continued

Continuing on from the previous article [Getting started with the OBS WebSocket using NODE.js](https://alecjeay.hashnode.dev/getting-started-with-the-obs-websocket-using-nodejs). lets use the template we built last time to request some information about a source, and then push a change to a source setting.

### OBS Test environment

First lets create a new Scene in OBS and add in some test sources. Iv created a Scene called "test" and added in two sources; a Browser source named "Browser"




![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649812142796/pbs_82DBu.png)


and an Image source named "Background"


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649812166204/kW7nKDHbf.png)


Make sure the "Background" source is higher on the list of sources so it appears on top.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649812252717/8UX7IPPKG.png)

### WebSocket Requests in Node.js

The code you have so far should look something like this, if not go back to the [previous article](https://alecjeay.hashnode.dev/getting-started-with-the-obs-websocket-using-nodejs) and take a quick look, its just a 3 min read. 

```
const OBSWebSocket = require("obs-websocket-js");
const obs = new OBSWebSocket();


obs.connect({
    address: "127.0.0.1:4444",
  })
  .then(() => console.log(`Success! We're connected & authenticated.`))
  .then(() => obs.send("GetSourcesList"))
  .then((data) => {
    console.log(data);
  });
```

When its run it should output something like this to the console.

```
Success! We're connected & authenticated.
{
  'message-id': '1',
  sources: [
    { name: 'Background', type: 'input', typeId: 'image_source' },
    { name: 'Browser', type: 'input', typeId: 'browser_source' },
    ],
  status: 'ok',
  messageId: '1'
}
```

Great. Now lets make a new request to get some specific information about the "Background" source.

```
.then(() => obs.send("GetSceneItemProperties", { item: "Background" }))
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

If everything is working we should get some information about the "Background" source logged to console.

```
{
  bounds: { alignment: 0, type: 'OBS_BOUNDS_NONE', x: 0, y: 0 },
  crop: { bottom: 0, left: 0, right: 0, top: 0 },
  height: 1080,
  itemId: 3,
  locked: false,
  'message-id': '2',
  name: 'Background',
  position: { alignment: 5, x: -271, y: 0 },
  rotation: 0,
  scale: { x: 1, y: 1 },
  sourceHeight: 1080,
  sourceWidth: 2560,
  status: 'ok',
  visible: true,
  width: 2560,
  messageId: '2'
}
```

### Sending data to the OBS WebSocket
Great!

Ok now we will will do something basic like; Check if the background is set to visible, then make it invisible.

```
.then((data) => {
    if (data.visible) {
      obs.send("SetSceneItemProperties", {
        item: "Background",
        visible: false,
      });
    } else
      obs.send("SetSceneItemProperties", {
        item: "Background",
        visible: true,
      });
  })
```

Our full code should look like
```
const OBSWebSocket = require("obs-websocket-js");
const obs = new OBSWebSocket();

obs
  .connect({
    address: "127.0.0.1:4444",
  })
  .then(() => console.log(`Success! We're connected & authenticated.`))
  .then(() => obs.send("GetSourcesList"))
  .then((data) => {
    console.log(data);
  })

  .then(() => obs.send("GetSceneItemProperties", { item: "Background" }))
  .then((data) => {
    if (data.visible) {
      obs.send("SetSceneItemProperties", {
        item: "Background",
        visible: false,
      });
    } else
      obs.send("SetSceneItemProperties", {
        item: "Background",
        visible: true,
      });
  })
  .catch((err) => console.error(err));
```

If everything is working correctly, now when you run app.js it will toggle the background source between visible and invisible.

Run the app a few times while watching the OBS preview window to see the effects happen real time.

Hopefully with this information you can start to see what setting are available to be changed, and get some ideas about how you want to start controlling OBS with your App

### Reference material
For a full list of settings see [OBS WebSocket Protocol Refference](https://github.com/obsproject/obs-websocket/blob/4.x-current/docs/generated/protocol.md)



