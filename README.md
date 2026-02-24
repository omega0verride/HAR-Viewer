<p align="center">
  <img src="icon.svg" width="120" height="120" alt="HTTP-Waterfall">
</p>
<h1 align="center">HTTP-Waterfall</h1>
<p align="center">
  A single-file HTTP request timeline visualizer. Drop in a HAR or JSON file and get an interactive waterfall chart of your HTTP traffic — no server, no dependencies, no install.
</p>

<!-- TODO: uncomment when deployed
## Try It Online

**[Launch HTTP-Waterfall](https://your-deployed-url.example.com)** — no install required, runs entirely in your browser.
-->

## Quick Start

1. Download or clone this repository
2. Open `index.html` in any modern browser
3. Drop a HAR or JSON file onto the page (or click to browse)

That's it. Everything runs client-side in a single HTML file.

## Features

- **Waterfall timeline** — visualize request timing with start, response, and end markers
- **HAR support** — load `.har` files exported from Chrome DevTools, Firefox, or any HAR-compliant tool
- **JSON support** — use a simple JSON array/object format for custom instrumentation
- **Request details** — click any request to inspect headers, body content, and timing
- **Body viewer** — view request/response bodies as RAW, JSON, XML, HTML, Image, or HEX
- **Thread grouping** — see which requests ran on which threads
- **Multi-select** — click, Ctrl+click, Shift+click, or Ctrl+A to select rows
- **Zoom** — zoom into selections, scroll to zoom, or use the slider; zoom-to-fit button
- **Content type detection** — automatic type column (JSON, XML, HTML, IMG, JS, CSS, etc.)
- **Drag & drop** — drop files directly onto the page
- **Zero dependencies** — single HTML file, no build step, no server required
- **Dark theme** — easy on the eyes

## Keyboard & Mouse Shortcuts

| Action | Shortcut |
|---|---|
| Select row | Click |
| Toggle selection | Ctrl+Click |
| Range select | Shift+Click |
| Select all | Ctrl+A |
| Horizontal scroll | Shift+Scroll |
| Zoom in/out | `+` / `-` |
| Open request details | Click request ID |

## Supported Formats

### 1. HAR (HTTP Archive)

Standard `.har` files as exported from browser DevTools. See the [HAR 1.2 spec](http://www.softwareishard.com/blog/har-12-spec/).

### 2. JSON Array

A JSON array of request objects:

```json
[
  {
    "id": 1,
    "uri": "http://localhost:3000/api/users",
    "method": "GET",
    "statusCode": 200,
    "statusMessage": "OK",
    "startRequestTimestamp": 1704067200000,
    "beginResponseTimestamp": 1704067200150,
    "endResponseTimestamp": 1704067200200,
    "threadId": "main",
    "requestHeaders": "Content-Type: application/json\nAuthorization: Bearer ...",
    "responseHeaders": "Content-Type: application/json\nContent-Length: 512",
    "requestBodyChunks": ["{ \"name\": \"Alice\" }"],
    "responseBodyChunks": ["{ \"id\": 1, \"name\": \"Alice\" }"],
    "requestBodyPath": "req_body_1.json",
    "responseBodyPath": "res_body_1.json"
  }
]
```

### 3. JSON Object

Wraps the array in an object with an optional base path for body files:

```json
{
  "requests_data_path": "C:/data/connections/",
  "requests": [
    { "id": 1, "uri": "http://localhost:3000/api/users", "..." : "..." }
  ]
}
```

### Field Reference

| Field | Description |
|---|---|
| `id` | Unique request identifier |
| `uri` | Request URL |
| `method` | HTTP method (GET, POST, etc.) |
| `statusCode` | HTTP response status code |
| `statusMessage` | HTTP response status text (e.g. "OK") |
| `startRequestTimestamp` | Request start time (epoch ms) |
| `beginResponseTimestamp` | First response byte time (epoch ms) |
| `endResponseTimestamp` | Response complete time (epoch ms) |
| `threadId` | Thread or connection identifier |
| `requestHeaders` | Request headers as a newline-separated string |
| `responseHeaders` | Response headers as a newline-separated string |
| `requestBodyChunks` | Array of strings containing the request body |
| `responseBodyChunks` | Array of strings containing the response body |
| `requestBodyPath` | File path to the request body content |
| `responseBodyPath` | File path to the response body content |

> **Note:** `requestBodyPath`/`responseBodyPath` and `requestBodyChunks[]`/`responseBodyChunks[]` are interchangeable. Chunks contain the actual body content inline. Paths are links to external files, useful for saving memory when bodies are large. When using paths, set `requests_data_path` in the object format to provide the base directory.

## License

[AGPL-3.0](LICENSE)
