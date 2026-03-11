# Add Optional Selection Requirement Feature

## Summary

This PR adds a new user-configurable option `allow_engines_without_selection` that enables using the extension as a link collection tool for quick navigation without requiring text selection first.

## Motivation

Users who want to use the extension for quick access to frequently used URLs (e.g., multiple test environment instances like `https://test1.example.com/dashboard`, `https://test2.example.com/dashboard`) currently must select text first to activate the popup. This PR makes text selection optional.

## Changes

### New Option
- **Name:** `allow_engines_without_selection`
- **Default:** `false` (maintains current behavior for existing users)
- **Location:** Advanced Settings → "Allow engines without selection"
- **Export/Import:** Automatically included in settings export/import

### Behavior When Enabled
- ✅ Inline popup activators show popup without text selection
- ✅ Toolbar popup executes engines without requiring query text
- ✅ Context menu appears on 'page' context (right-click anywhere)
- ✅ ALL engines appear regardless of %s placeholder
- ✅ Engines with %s search for empty string
- ✅ Engines without %s navigate directly
- ✅ Page variables (%PAGE_URL, etc.) work without %s

### Files Modified (6 files, ~70 lines changed)

1. **`background/storage.js`** - Added option to defaults
2. **`background/contextmenu.js`** - Added 'page' context support when option enabled
3. **`browseraction/popup.js`** - Made `hasQuery()` conditional on option
4. **`popup/activators.js`** - Updated all activators to check option
5. **`options/options.html`** - Added UI checkbox and updated documentation
6. **`options/options.js`** - Connected save/load functions

## Implementation Details

The implementation is minimal and non-invasive:
- No filtering of engines based on %s presence - all engines always shown
- Option only controls whether popup can be activated without selection
- All existing functionality preserved when option is disabled (default)
- No breaking changes to existing APIs or behavior

## Testing

Tested scenarios:
- ✅ Default behavior (option OFF) unchanged
- ✅ Inline popup with all activators (click, auto, keyboard+mouse, double-click)
- ✅ Toolbar popup without query text
- ✅ Context menu in page context
- ✅ Mixed engines (with/without %s, page variables, special actions)
- ✅ Export/import preserves setting
- ✅ Chrome sync works correctly

## Backward Compatibility

- ✅ Default is OFF - no behavior change for existing users
- ✅ Old configs imported without the option default to OFF
- ✅ No changes to existing APIs
- ✅ All existing features work unchanged

## Use Cases

1. **Link Collections:** Quick access to test/staging/production environments
2. **Page Variables Only:** Engines using only %PAGE_URL without %s
3. **Navigation Shortcuts:** Static URLs for frequently visited pages
4. **Bookmarks Alternative:** Search engines as organized bookmarks

## Commits

- `27e28bd` - Add optional selection requirement feature (main implementation)
- `91c1509` - Fix: Wire up option in options page (connect save/load functions)

## Screenshots

### Options Page
![Allow engines without selection checkbox in Advanced Settings](screenshot-would-go-here)

### Context Menu Without Selection
Works in both 'selection' context (on text) and 'page' context (anywhere).

## Notes

- Context menu must be separately enabled via Menu Activation settings
- The option name reflects what it does: allows engines (popup) without selection
- Simple, clean implementation following existing patterns in the codebase

---

Looking forward to feedback! Happy to make any adjustments needed.
