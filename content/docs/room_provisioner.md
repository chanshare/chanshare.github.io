---
title: "Room Provisioner"
weight: 1
---

# Room Provisioner

The room provisioner takes a create room request,
tells the content service to gather the media, 
and tells the room manager to start a room.

The request for provisioning a room looks like this 
```json
ENDPOINT: /create_room
{
"user": "your perfered nickname",
"thread_id": "1234123",
"board_sn": "wsg"
}
```

On receiving that request the first thing to happen is the playlist is assembled by making a request to the content service.
Chances are it will not be fast enough to just download all the media and have the user wait.
(Well it might be with a nice spinner :) ) 
So having the CDN links be generated in a predictable format is pretty essential.

While the playlist is being generated we generate a name for the room.
This will be done though mnemonic generation. 
Probably 3 words to keep it short.

Once we have a name and a playlist we can make a request to the Room Manager to start the room 
```json
{
"user": "the users username",
"playlist": [
"media item 1",
"media item 2"
],
"room_name": "the-room-name"
}
```

{{< mermaid >}}
sequenceDiagram
    API Gateway->Room Provisioner: /create_room
    Room Provisioner->>Content Service: generate playlist
    Content Service-XRoom Provisioner: Recive playlist
    Content Service-XContent Service: Download media
    Room Provisioner->>Room Provisioner: Generate Name
    Room Provisioner->>Room Manager: Start Room
{{< /mermaid >}}
