---
sidebar_position: 2
---

# External API

This page is intended to be an technical reference on asbplayer's external interface and the [pre-packaged WebSocket server](https://github.com/asbplayer/asbplayer/tree/main/scripts/web-socket-server) that implements this interface. See the [guide](../guides/web-socket-server) for how to setup the WebSocket server.

## WebSocket commands

asbplayer, as a WebSocket client, responds to the following commands from a WebSocket server.

### `mine-subtitle`

#### Request

    ```javascript
    {
        "command": "mine-subtitle",
        // Message ID to correlate with asbplayer's response
        "messageId": "10281760-d787-4356-8572-f698d8ff3884",
        "body": {
            // 0 = "None", 1 = "Show anki dialog", 2 = "Update last card", 3 = "Export card"
            "postMineAction": 1,
            // Key-value pairs corresponding to an Anki note type
            "fields": {
                "key1": "value1",
                "key2": "value2"
            }
        }
    }
    ```

#### Response

    ```javascript
    {
        "command": "response",
        // Same message ID received in request
        "messageId": "10281760-d787-4356-8572-f698d8ff3884",
        "body": {
            // Whether the command was successfully published to an asbplayer client
            "published": true
        }
    }
    ```

### `load-subtitles`

#### Request

```javascript
{
    "command": "load-subtitles",
    // Message ID to correlate with asbplayer's response
    "messageId": "3565510c-342f-4cec-ad2e-dee81af88d75",
    "body": {
        "files": [{
            // Name of the file, including its extension
            "name": "some-file.srt",
            // Base64-encoded file contents
            "base64": "Zm9vYmFyY..."
        }]
    }
}
```

#### Response

```javascript
{
    "command": "response",
    // Same message ID received in request
    "messageId": "3565510c-342f-4cec-ad2e-dee81af88d75",
    "body": {}
}
```

### `seek-timestamp`

#### Request

```javascript
{
    "command": "seek-timestamp",
    // Message ID to correlate with asbplayer's response
    "messageId": "6e4b2d8f-3a1c-4d9e-8f7b-2c0a9d5e1f3b",
    "body": {
        //The timestamp to seek in seconds
        "timestamp": 30.5,
    }
}
```

#### Response

```javascript
{
    "command": "response",
    // Same message ID received in request
    "messageId": "6e4b2d8f-3a1c-4d9e-8f7b-2c0a9d5e1f3b",
    "body": {}
}
```

### `get-bound-media`

Returns every `<video>` element asbplayer is currently tracking, keyed by tab and `src`.

#### Request

```javascript
{
    "command": "get-bound-media",
    // Message ID to correlate with asbplayer's response
    "messageId": "9f1c2b3a-4d5e-6f70-8190-a1b2c3d4e5f6",
    "body": {}
}
```

#### Response

```javascript
{
    "command": "response",
    // Same message ID received in request
    "messageId": "9f1c2b3a-4d5e-6f70-8190-a1b2c3d4e5f6",
    "body": {
        "media": [{
            // ID of the tab the video element lives in
            "tabId": 123,
            // The video element's src; serves as the media's identifier
            "src": "https://example.com/video.mp4",
            // Title of the tab
            "title": "Example video",
            // Favicon of the tab
            "faviconUrl": "https://example.com/favicon.ico",
            // Whether the video element is subscribed to extension messages
            "subscribed": true,
            // Whether subtitles have been synced to the video element
            "synced": true,
            // Whether a non-empty subtitle track is loaded
            "loadedSubtitles": true,
            // When syncing occurred (epoch milliseconds), if synced
            "syncedTimestamp": 1717000000000,
            // Whether the tab is the active tab of its window
            "active": true,
            // Whether the tab is the active tab of the currently focused window
            "focused": true,
            // Whether the tab has been discarded (unloaded from memory) - acting on it may fail
            "discarded": false
        }]
    }
}
```

## HTTP-based API

The WebSocket server also implements an HTTP-based API which can trigger the commands above.

- `POST asbplayer/load-subtitles` ([script](https://github.com/asbplayer/asbplayer/blob/main/scripts/web-socket-server/cli/load-subtitles))
- `POST asbplayer/seek` ([script](https://github.com/asbplayer/asbplayer/blob/main/scripts/web-socket-server/cli/seek))
- `GET asbplayer/bound-media` ([script](https://github.com/asbplayer/asbplayer/blob/main/scripts/web-socket-server/cli/bound-media))

## AnkiConnect proxy

It also functions as an AnkiConnect proxy that allows `addNote` requests to be enriched with asbplayer-provided context, such as audio and screenshots.

The proxy passes through all AnkiConnect requests as-is except for `addNote`. The proxy's specific behavior in the case of `addNote` depends on the value of `POST_MINE_ACTION` in the configuration documented below.

## Server configuration

The server is configured with an `.env` file placd next to it in the same directory. Below is an example file with explanation.

```
# Port that the proxy will listen on
PORT=8766

# AnkiConnect URL
ANKI_CONNECT_URL=http://127.0.0.1:8765

# Action for asbplayer to take when the proxy receives an addNote request
# 0 = 'None'
# 1 = 'Open Anki dialog'
# 2 = 'Update last card' (updates last card with asbplayer media AFTER passing through original addNote request)
# 3 = 'Export card'
POST_MINE_ACTION=2
```
