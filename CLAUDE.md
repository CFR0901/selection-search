# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Selection Search is a Chrome/Firefox browser extension (Manifest V3) that allows users to search for selected text using customizable search engines via popup menus, context menus, or keyboard shortcuts.

**Repository:** This is a fork of the original Selection Search extension by Pitmairen.
- Original repository: https://github.com/Pitmairen/selection-search
- This fork: https://github.com/CFR0901/selection-search

**Key capabilities:**
- Inline popup menus that appear when text is selected
- Context menu integration
- Toolbar popup with search suggestions
- Keyboard hotkeys for search engines
- Submenus and separators for organizing search engines
- Blacklist/whitelist for controlling where popups appear
- Custom styling and circular menu layouts
- Special features: copy to clipboard, direct URL opening, POST searches

## Architecture

### Manifest V3 Structure

The extension uses Manifest V3 with a service worker background script:

**Background (Service Worker):**
- Entry point: `background/service_worker.js` imports all background scripts via `importScripts()`
- `background/init.js` - Initializes the Background controller
- `background/storage.js` - Main storage abstraction using DataStore pattern
- `background/iconloader.js` - Loads and caches search engine icons
- `background/contextmenu.js` - Manages context menu integration
- `background/sync.js` - Handles Chrome sync storage
- `background/blacklist.js` - URL filtering for popup activation

**Content Scripts:**
- Entry point: `popup/init_script.js` - Main initialization, creates shadow DOM
- `popup/popup.js` - Core Popup class that renders the menu
- `popup/activators.js` - Different activation methods (click, auto, keyboard+mouse, combo)
- `popup/actions.js` - Action handlers (DefaultAction, CopyAction, DomainAction)
- `popup/positioning.js` - Menu positioning logic
- `popup/search_engine_hotkeys.js` - Keyboard shortcut handling

**Options Page:**
- `options/options.html` and `options/options.js`
- Uses jQuery for UI interactions
- `options/reorder.js` - Drag-and-drop reordering of search engines

**Toolbar Popup:**
- `browseraction/popup.html` and `browseraction/popup.js`
- Separate popup for the toolbar icon with search suggestions

### Key Data Flow

1. **Initialization:** Service worker loads settings from chrome.storage.local and optionally syncs with chrome.storage.sync
2. **Content Script Injection:** On page load, content scripts request configuration via `getContentScriptData` message
3. **Selection Detection:** Activators monitor page events (mouseup, keydown) to detect text selection
4. **Popup Display:** When activated, popup is rendered in a closed shadow DOM to prevent style conflicts
5. **Search Execution:** User clicks search engine → message to background → background opens new tab/window

### Storage System

The extension uses a layered storage approach:

- **DataStore** (`background/storage.js`) - Provides getters/setters with default values
- **MemoryKWStore** - In-memory key-value store
- **storageLocalSync** (`background/storage_local.js`) - Persists to chrome.storage.local
- **Sync** (`background/sync.js`) - Optionally syncs settings to chrome.storage.sync

Storage keys:
- `searchEngines` - Array of search engine objects
- `options` - User preferences
- `styleSheet` - Custom CSS for popup
- `styleSheetTB` - Custom CSS for toolbar
- `blacklist` - URL patterns to disable popup
- `click-count` - Usage statistics for sort-by-click feature

### Shadow DOM Usage

The popup menu is injected into a **closed shadow DOM** to:
- Prevent website CSS from affecting the popup
- Prevent the popup CSS from affecting the website
- Isolate event handling

Created in `popup/init_script.js`:
```javascript
var shadowElement = document.createElement("div");
var shadowDOM = BrowserSupport.createShadowDOM(shadowElement);
document.documentElement.appendChild(shadowElement);
```

### Search Engine Object Structure

```javascript
{
  name: 'Search Name',
  url: 'https://example.com/search?q=%s',  // %s is replaced with selection
  icon_url: 'https://example.com/favicon.ico',
  is_submenu: false,  // true for submenus
  is_separator: false,  // true for separators
  engines: [],  // array of engines for submenus
  hide_in_popup: false,  // hide in popup but show in context menu
  // ... other options
}
```

### Activator Pattern

Activators control how the popup is triggered:

- **ClickActivator** - Specific mouse button click
- **AutoActivator** - Automatically shows button on selection
- **KeyAndMouseActivator** - Keyboard key + mouse button combo (e.g., Ctrl+Click)
- **DoubleClickActivator** - Double-click to activate
- **ComboActivator** - Multiple activators enabled simultaneously

All extend the base `Activator` class with `setup()` and `popupShouldOpen()` methods.

### Action Pattern

Actions handle what happens when a search engine is clicked:

- **DefaultAction** - Standard search (opens URL with %s replaced)
- **CopyAction** - Special "COPY" engine copies selection to clipboard
- **DomainAction** - Special "%s" engine opens selected URL directly

Actions implement `onClick()`, `onAuxClick()`, and `onEnter()` methods.

## Common Development Tasks

### Testing the Extension

Since this is a browser extension, there's no traditional test runner. Testing is done manually:

1. Load unpacked extension in Chrome:
   - Navigate to `chrome://extensions`
   - Enable "Developer mode"
   - Click "Load unpacked" and select the repository root

2. Test pages are available in `tests/` directory:
   - `tests/selection.html` - Basic selection testing
   - `tests/forms.html` - Testing in form inputs
   - `tests/iframes.html` - Testing in iframes

3. After making changes, reload the extension:
   - Go to `chrome://extensions`
   - Click the reload icon on the Selection Search extension

### Adding a New Feature

1. **For popup menu features:**
   - Modify `popup/popup.js` for UI changes
   - Add new activator in `popup/activators.js` if needed
   - Update options in `background/storage.js` default options

2. **For options page features:**
   - Add UI elements in `options/options.html`
   - Add handlers in `options/options.js`
   - Update `background/storage.js` defaults

3. **For background features:**
   - Add message handler in `background/init.js` onMessage()
   - Implement functionality in appropriate background script

### URL Placeholder Variables

The extension supports various placeholders in search engine URLs:

- `%s` - Selected text (URL encoded)
- `{%s}` - Selected text (URL encoded, alternative syntax)
- `{%-s}` - Selected text with spaces not encoded
- `{%+s}` - Selected text with spaces replaced by +
- `{%RAW_s}` - Selected text without encoding
- `{%s|upper}` / `{%s|lower}` - Convert case
- `{%s|replace:pattern:replacement}` - Regex replacement
- `%PAGE_URL` - Current page URL
- `%PAGE_HOST` - Current page hostname
- `%PAGE_ORIGIN` - Current page origin
- `%PAGE_QUERY_STRING` - Current page query string
- `%PAGE_QS_VAR(name)` - Specific query string variable
- Special: `COPY` - Copy to clipboard instead of searching
- Special: `%s` alone - Open selected text as URL

### Modifying Styles

The extension has two main style systems:

1. **Popup styles** (`background/default_style.js`):
   - Contains default CSS for the popup menu
   - Users can add custom CSS in options page
   - Custom CSS is stored in `styleSheet` storage key

2. **Circular menu** (`popup/circular_popup.js`):
   - Alternative popup layout
   - Uses `PopupModifier` pattern to transform standard popup

### Icon Loading

Icons are loaded asynchronously in the background:

1. `background/iconloader.js` - Fetches icons from URLs
2. Icons cached in chrome.storage.session (limited capacity)
3. Fallback icon loading from multiple sources (Google S2, DuckDuckGo)
4. Special handling for `CURRENT_DOMAIN` variable in icon URLs

## User Scripts

The `userscripts/` directory contains Tampermonkey/Violentmonkey scripts for sites that require JavaScript to trigger searches (e.g., sites that don't use traditional query parameters). These are separate from the extension and must be installed by users manually.

## SAP-Specific Configuration

A pre-configured settings file for time management use cases is available at:
https://github.tools.sap/D068075/selection-search-settings/tree/main/settings

These settings can be imported via the extension's options page.

## Important Notes

- This is a **Manifest V3** extension - use service workers, not background pages
- Always use chrome.storage APIs, never localStorage (changed in v0.8.48)
- The shadow DOM is **closed** - website scripts cannot access it
- Icons have size limits due to chrome.storage.session quota
- POST searches are supported via special helper pages in `old/` directory
- The extension supports both Chrome and Firefox with browser compatibility layer in `common/browsersupport.js`
