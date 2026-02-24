# HttpWaterfall - Project Notes for Future LLMs

## IMPORTANT: Keep This File Updated
When making changes or adding new features to the codebase, always update this file to document:
- New functions or components
- Changes to existing functionality
- New edge cases or patterns discovered
- Any breaking changes or important implementation details
- **IMPORTANT**: Also update this file when you discover important things about how the codebase is designed while working on tasks - even small but significant design decisions, patterns, or architectural choices should be documented here so future agents understand the reasoning behind implementations

This ensures future LLMs have accurate context about the project.

## Overview
HttpWaterfall is an HTTP request timeline visualizer in a single HTML file (`index.html`). It loads request data from JSON files or HAR files and displays them in a waterfall timeline.

## Core Features

### 1. Timeline Visualization
- Waterfall timeline showing request start/end times
- Thread badges with 15-char truncation and hover tooltips
- Row selection: click, Ctrl+click (toggle), Shift+click (range), Ctrl+A (all)
- Zoom to selection works with both timeline clicks and row selections
- Base URI filter that strips common prefixes for display only

### 2. HAR File Support
- `convertHarToRequests()` converts HAR format to internal format
- Extracts headers from `entry.request.headers[]` and `entry.response.headers[]`
- Handles body content from:
  - `req.postData.text` for request body
  - `res.content.text` for response body
- **Critical**: Detects base64 encoding via `res.content.encoding === 'base64'`
- Stores `responseMimeType`, `responseEncoding`, and `responseOriginalBody` for body viewing
- Falls back: if HAR has no explicit encoding but mimeType starts with `image/`, assumes base64

### 3. Body Viewer (Popup Dialog)
- Slides from right, 700px wide
- Format buttons: RAW, JSON, XML, HTML, Image, HEX
- HEX view shows hex on left, ASCII on right (16 bytes/row, padded)
- Size indicator calculated from actual content bytes
- Copy button with "Copied!" feedback
- Wrap toggle checkbox
- **Important**: Uses `<textarea>` for text formats (allows cursor navigation), separate `<div>` for images

### 4. Request Details Panel (Inline)
- Embedded in detail panel with collapsible sections
- Section titles are larger (14px, bold, gray)
- Body section has:
  - Collapsible (collapsed by default)
  - Format buttons inline
  - Wrap checkbox
  - Copy button in header
  - 250px fixed height with scroll
- When no body chunks but bodyPath exists: shows collapsible with "Open" link

## Key Technical Edge Cases

### 1. Timeline Header and Ticks
- Timeline header width is `max(timelineWidth, visibleWidth)` to ensure header extends beyond last request when zoomed out
- Ticks are dynamically calculated based on effectiveWidth with ~120px minimum spacing
- Tick format: `DDTHH:mm:ss.SSS` (includes milliseconds for precision)
- `updateTimelineHeader()` function updates header width and ticks without full re-render
- `setupResizeHandler()` sets up window resize listener with 100ms debounce for dynamic tick updates
- `initResizer()` also calls `updateTimelineHeader()` during drag (50ms throttle) and on mouseup for detail panel resizing
- Tick count: `max(5, floor(effectiveWidth / 120))` - more ticks when zoomed in
- Mouse clicks and cursor tracking work beyond the last request (up to effectiveWidth)

### 2. Image Rendering
- HAR stores binary images as base64 in `content.text` with `encoding: "base64"`
- Image viewer uses `data:image/[type];base64,[data]` format
- Three sources for base64 (in order):
  1. `currentBodyOriginal` - stored separately during HAR conversion
  2. `currentBodyContent` - if `encoding === 'base64'`
  3. Try to detect if mimeType is image and content looks like base64 (regex: `/^[A-Za-z0-9+/=]+$/`)
- Fallback: try `btoa()` encoding (rarely works with binary)

### 3. Type Column Detection
- Added "Type" column that detects content type from `Content-Type` response header
- `getTypeFromContentType()` function maps MIME types to short labels:
  - `application/json` → JSON
  - `application/xml`, `text/xml` → XML
  - `text/html` → HTML
  - `text/*` → TEXT
  - `image/*` → IMG
  - `application/javascript` → JS
  - `application/css` → CSS
  - `font/*` → FONT
  - `application/pdf` → PDF
  - `zip/gzip/tar/rar` → ARCH
  - `application/octet-stream`, `binary` → BIN
  - Otherwise: extracts subtype (first 8 chars)
- Falls back to extracting from `responseMimeType` if no Content-Type header
- Type stored in `req.type` field

### 4. HEX View Alignment
- Each row: 16 bytes × 3 chars = 48 chars for hex
- 2 spaces gap
- 16 chars for ASCII
- Last row padded with spaces to align ASCII column

### 5. Textarea vs div for body content
- Textarea for RAW/JSON/XML/HEX (allows cursor, selection, copy)
- Hidden div with innerHTML for images
- Must reset styles (display, justify-content, align-items) when switching from image to text
- Wrap toggle must use `whiteSpace: pre-wrap` and `word-break: break-all`

### 6. Collapsible Sections
- Use `collapsed` class on parent element
- CSS: `.detail-section.collapsed .collapsible-content { display: none; }`
- Toggle with `classList.toggle('collapsed')`
- Add `event.stopPropagation()` when click is on button inside collapsible title
- Make sure to have proper unique IDs for toggle (e.g., `detail_${uniqueId}`)

### 7. Escape HTML
- All user content (URIs, headers, body content) must be escaped with `escapeHtml()`
- Special care for hidden inputs storing base64 - must escape `&`, `<`, `>`, `"`, `'`

### 8. HAR Conversion
- Body chunks stored as arrays (`requestBodyChunks[]`, `responseBodyChunks[]`)
- For base64 content: keep original in `responseOriginalBody`, don't decode
- For text display: try to decode, if fails keep original base64

### 9. Body Popup Scroll Position
- When opening body modal, use `setSelectionRange(0, 0)` to reset cursor to top
- Use `requestAnimationFrame` to reset scroll after render if needed

### 10. Detail Panel Sticky Header
- Header uses `position: sticky; top: 0; z-index: 10;` to stay on top
- Must have `background` set on header for it to cover content when scrolling

## JSON Format Fields
- **Required**: `id`, `uri`, `method`, `statusCode`, `startRequestTimestamp`, `beginResponseTimestamp`, `endResponseTimestamp`, `threadId`
- **Additional**: `statusMessage`, `requestHeaders`, `responseHeaders`, `requestBodyPath`, `responseBodyPath`, `requestBodyChunks[]`, `responseBodyChunks[]`
- **Path vs Chunks**: `requestBodyPath`/`responseBodyPath` and `requestBodyChunks[]`/`responseBodyChunks[]` are interchangeable. Chunks contain the actual content inline; paths are links to external files (to save memory). When paths are used, `requests_data_path` in the object format provides the base directory.

## File Structure
```
./
  index.html   - Single file containing all HTML, CSS, JS
  AGENTS.md    - This file
```

## Testing
- HAR test file: `./samples/www.softwareishard.com.har` (contains image responses)
- Image mimeTypes tested: image/jpeg, image/png
- Verify: RAW shows base64 string, Image renders correctly, JSON/XML parse properly

## Code Patterns

### Event Delegation
Use event delegation instead of inline onclick handlers where possible. For click interactions on dynamically created elements, use proper delegation or unique IDs.

### CSS Classes Used
- `.detail-panel` - main panel container
- `.detail-panel-header` - sticky header with z-index
- `.detail-section` - sections within panel
- `.detail-section.collapsed` - collapsed state
- `.collapsible-content` - content hidden when collapsed
- `.format-btn` - format selection buttons
- `.format-btn.active` - active format button (cyan background)
- `.body-no-wrap` / `.body-wrap` - text wrapping styles

### Key Functions
- `convertHarToRequests(har)` - HAR to internal format
- `showBodyModal(type, content, id, mimeType, encoding, originalBody)` - opens body popup
- `setBodyFormat(format)` - handles format switching in popup
- `getDetailBodySection(req, bodyType)` - generates inline body HTML
- `setDetailBodyFormat(btn, uniqueId)` - handles format switching in detail panel
- `toggleDetailBodyWrap(uniqueId)` - wrap toggle in detail panel
- `escapeHtml(str)` - escape user content
- `updateTimelineHeader()` - updates header width and ticks without full re-render
- `setupResizeHandler()` - sets up window resize listener for dynamic tick updates
- `zoomToFit()` - uses binary search to find optimal zoom level that fits timeline to visible width (works zooming in or out)
- `sliderToWidth(value)` / `widthToSlider(width)` - logarithmic conversion between slider value (0-100) and timeline width (500-2M px)
