修改的东西

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a standalone single-file frontend application (`import.html`) for uploading and processing knowledge base files (PDF/Markdown). It provides a real-time progress tracking interface for a backend processing pipeline.

**Technology Stack:**
- Pure HTML5, CSS3, and vanilla JavaScript (ES6+)
- No build tools, bundlers, or frontend frameworks
- No external dependencies (all code is self-contained)

## Development

### Running the Application

Since this is a static HTML file, serve it with any HTTP server:

```bash
# Python 3
python -m http.server 8000

# Node.js (if npx available)
npx serve .

# Or simply open in browser (file:// protocol works but some features may be limited)
```

Then open `http://localhost:8000/import.html` in your browser.

### Backend Dependency

This frontend requires a backend API running on `http://127.0.0.1:8000` with the following endpoints:

- `POST /upload` - Accepts multipart/form-data with file, returns `{"task_id": "..."}`
- `GET /status/{task_id}` - Returns processing status:
  ```json
  {
    "status": "processing|completed|failed",
    "done_list": ["upload_file", "pdf_to_md", ...],
    "running_list": ["current_node"],
    "durations": {"node_name": seconds}
  }
  ```

## Architecture

### Core Components

The file is organized into three sections:

1. **CSS (lines 7-379)**: Modern glassmorphism UI with dark theme
   - Uses CSS custom properties implicitly through hardcoded values
   - Key classes: `.upload-area`, `.file-item`, `.progress-bar`, `.log-details`

2. **HTML (lines 381-394)**: Single container structure
   - Upload drop zone with drag-and-drop support
   - Dynamic file list container

3. **JavaScript (lines 396-658)**: Upload and progress management
   - `handleFiles()` - Entry point for file selection
   - `uploadFile()` - Uploads file and initiates polling
   - `pollStatus()` - Polls backend every 1.5 seconds for status updates
   - `renderLogsAndCalculateProgress()` - Renders timeline and calculates percentage

### Key Logic

**Dynamic Step Calculation:**
- PDF files: 8 processing steps (includes `pdf_to_md` node)
- Markdown files: 7 processing steps (skips `pdf_to_md`)

**Progress Algorithm:**
```javascript
percentage = (completed_steps / total_steps) * 100
if (has_running_step) percentage += (0.5 / total_steps) * 100
```

**State Machine:**
```
uploading → processing → completed|failed
```

## File Structure

```
import.html          # Complete standalone application (660 lines)
```

No package.json, no build configuration, no test suite. This is intentionally a zero-dependency single-file solution.

## Important Implementation Details

- **Polling Interval**: 1500ms (defined in `pollStatus()`)
- **Max File Types**: Accepts `.pdf` and `.md` files only
- **API Base URL**: Hardcoded to `http://127.0.0.1:8000` (line 402)
- **Progress Bar Animation**: Uses CSS shimmer effect and color transitions
- **Log Timeline**: Visual timeline with color-coded nodes (blue for running, green for done)

## Testing Changes

Since there's no automated test suite:

1. Manually test file upload with both PDF and MD files
2. Verify progress bar updates correctly (check percentage math in console)
3. Test error handling by stopping the backend mid-upload
4. Verify responsive design at different viewport sizes
5. Test drag-and-drop functionality across browsers

## Common Modifications

- **Change API endpoint**: Edit line 402 (`const API_BASE`)
- **Adjust polling frequency**: Edit line 656 (interval milliseconds)
- **Add new file types**: Update line 389 (`accept` attribute) and line 451 (isPDF logic)
- **Modify step count**: Update lines 453 (PDF steps) and adjust node counting logic
