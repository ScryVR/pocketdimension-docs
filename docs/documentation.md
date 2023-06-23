# Pocket Dimension Documentation

**NOTE:** At the time of writing, Pocket Dimension is still a work in progress. Some sections of this documentation are aspirational, and indicate an intended final state rather than the current implementation.

## Getting started with Pocket Dimension

Pocket Dimension works best on a mobile device or VR headset. It's a web app, so simply navigate to [pocketdimension.io](https://pocketdimension.io) to get started. Once Pocket Dimension loads, you'll be prompted for location permissions. Once Pocket Dimension reads your location, you'll be able to view terrain, check what resources are available, look for connected users, and enter the dimension.

Once the user enters a dimension, they can use the trackpad to walk around and explore the landscape. A number of glowing purple orbs randomly appear and disappear across the terrain. These orbs are resource generators. Each orb produces some default number of resources at regular intervals. Interacting with orb increases a counter. When the orbs generate resources, the number of resources they generate increases based on this counter. The counter resets whenever resources are generated. This allows users to either actively or passively collect resources. The number of resources generated is one of the dimension properties, just like the resource types and shape of the landscape. If you're very lucky, you'll find a dimension that generates a large base number of rare resources.

Resources can be used to build things. Pocket Dimension has a simple 3D modeling system that's optimized for either touch screens or VR controllers. To use the build system, scroll to the "build" option in the action selector near the bottom of the screen. This will cause a list of available objects to appear. Touching anywhere on the screen will create an object. Touching this object with the "interact" action selected allows you to change the object's size, position, and rotation. Dragging your finger horizontally or vertically changes the size of the object. If you have the "rotate" mode enabled, a one-finger drag will rotate the object instead. A two-finger drag moves the object.

Group selection and blueprints are important parts of Pocket Dimension's build system. Tapping on additional objects while in "interact" mode allows you to select things. This allows you to transform multiple things at once. Tapping the "group" button while multiple objects are selected makes these groups permanent. Groups can be easily copied selecting "build copy" in the action menu. Groups can be turned into blueprints, which can be reused even more easily and even shared with other users.

## Technical information

### Geolocation sequence

The following diagram shows how users' GPS coordinates are used to deterministically generate terrain.

```mermaid
sequenceDiagram
  participant user as User
  participant client as App client
  participant gps as Browser Geolocation API
  participant did_api as Dimension API
  participant terr as Terrainosaurus terrain generator
  participant db as Dimension database
  
  user->>client: Open app
  client->>gps: Get user location
    gps->>client: 
  client->>did_api: Get dimension attributes
    did_api->>client: 
  client->>terr: Generate terrain
    terr->>client: 
  client->>db: Load previously-built structures from database
    db->>client: 
```

### Multiplayer initialization sequence

The following diagram shows how Pocket Dimension creates peer-to-peer networks.
The exact process for creating WebRTC connections has been somewhat simplified for readability.

```mermaid
sequenceDiagram
  participant user as User
  participant client as App client
  participant gps as Browser Geolocation API
  participant did_api as Dimension API
  participant ws as Serverless Websocket API
  participant others as Other connected users
  
  user->>client: Open app
  client->>gps: Get user location
    gps->>client: 
  client->>did_api: Get dimension attributes
    did_api->>client: 
  client->>ws: Connect to websocket channel using dimension ID
  ws->>others: Notify other connected users of new connection
  others->>ws: Send WebRTC SDP offer
    ws->>client: 
  client->>ws: Send WebRTC SDP answer
    ws->>others: 
  others->>client: Create p2p network.<br>Broadcast audio, video, and user actions across network.
  client->>others: 
  
```