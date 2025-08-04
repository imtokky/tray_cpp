# Tray

## ⚠️ Warning! We have not verified Mac and Linux compatibility.

Cross-platform, single header, super tiny C99 implementation of a system tray icon with a popup menu.

**Works well on:**
* Windows XP or newer (shellapi.h) ✅ **Verified**
* Linux/Gtk (libappindicator) ⚠️ **Not verified**
* MacOS (Cocoa/AppKit) ⚠️ **Not verified**

There is also a stub implementation that returns errors on attempt to create a tray menu.

## Setup

The library now includes **automatic platform detection**! You no longer need to manually define platform macros in most cases.

### Automatic Detection (Recommended)
Simply include the header - the library will automatically detect your platform:

```cpp
#include <iostream>
#include "tray.hpp"

// That's it! The library automatically detects:
// - Windows -> TRAY_WINAPI
// - macOS -> TRAY_APPKIT  
// - Linux -> TRAY_APPINDICATOR
```

### Manual Definition (If Needed)
If automatic detection doesn't work, you can manually define the platform:

**For Windows:**
```cpp
#define TRAY_WINAPI
#include "tray.hpp"
```

**For Linux:**
```cpp
#define TRAY_APPINDICATOR
#include "tray.hpp"
```

**For Mac:**
```cpp
#define TRAY_APPKIT
#include "tray.hpp"
```

## Example

```cpp
#include <iostream>
#include <atomic>
#include "tray.hpp"

using namespace std;

// Global exit flag
static atomic<bool> should_exit{false};

// Forward declaration
void quit_cb(struct tray_menu* item);

// Menu items (must be static or global to persist)
static struct tray_menu menu_items[] = {
    {"Toggle me", 0, 0, toggle_cb, nullptr, nullptr},
    {"-", 0, 0, nullptr, nullptr, nullptr},        // Separator
    {"Quit", 0, 0, quit_cb, nullptr, nullptr},
    {nullptr, 0, 0, nullptr, nullptr, nullptr}     // Terminator
};

// Tray structure
static struct tray tray = {
    "icon.ico",     // Windows: .ico file recommended
    menu_items
};

void toggle_cb(struct tray_menu* item) {
    item->checked = !item->checked;
    tray_update(&tray);
    cout << "Toggle: " << (item->checked ? "ON" : "OFF") << endl;
}

void quit_cb(struct tray_menu* item) {
    (void)item;  // Suppress unused parameter warning
    should_exit = true;
    cout << "Ending processing..." << endl;
    tray_exit();
}

int main() {
    // Initialize tray icon
    if (tray_init(&tray) < 0) {
        cerr << "Failed to initialize tray icon" << endl;
        return -1;
    }
    
    cout << "Tray icon created. Right-click to access menu." << endl;
    
    // Main event loop
    while (!should_exit && tray_loop(1) == 0) {
        // Continue running
    }
    
    cout << "Application terminated." << endl;
    return 0;
}
```

## API Reference

### Structures

Tray structure defines an icon and a menu. Menu is a NULL-terminated array of items.

```cpp
struct tray {
    const char* icon;           // Path to icon file or system icon
    struct tray_menu* menu;     // Pointer to menu items array
};

struct tray_menu {
    const char* text;           // Menu item text (NULL for terminator)
    int disabled;               // 1 = grayed out, 0 = enabled
    int checked;                // 1 = checkmark, 0 = no checkmark
    void (*cb)(struct tray_menu*); // Callback function (can be NULL)
    void* context;              // Optional user data
    struct tray_menu* submenu;  // Submenu items (can be NULL)
};
```

### Functions

**All functions must be called from the UI thread only.**

* **`int tray_init(struct tray* tray)`** - Creates tray icon. Returns -1 if tray icon/menu can't be created.
* **`void tray_update(struct tray* tray)`** - Updates tray icon and menu.
* **`int tray_loop(int blocking)`** - Runs one iteration of the UI loop. Returns -1 if `tray_exit()` has been called.
* **`void tray_exit()`** - Terminates UI loop and cleans up resources.

### Important Notes

- Menu arrays must be terminated with a NULL item (text field set to NULL)
- Menu arrays must remain valid throughout the tray's lifetime (use static/global)
- Call `tray_exit()` to properly clean up before program termination

## Icon Guidelines

### Windows ✅
- **Best**: `.ico` files (16x16, 32x32 pixels)
- **Also works**: `.png`, `.bmp`
- **System icons**: `"shell32.dll"` for default system icon
- **Example**: `"C:\\path\\to\\icon.ico"` or `"./icon.ico"`

### Linux ⚠️ (Not verified)
- **Recommended**: `.png` files
- **System icons**: Standard icon theme names
- **Example**: `"/usr/share/pixmaps/myapp.png"`

### macOS ⚠️ (Not verified)
- **Recommended**: `.icns` files
- **System icons**: NSImage names like `"NSImageNameInfo"`
- **Example**: `"icon.icns"` or system icon names

## Troubleshooting

### "Failed to initialize tray icon"
1. **Check icon file exists** and is accessible
2. **Try system icon**: Use `"shell32.dll"` on Windows
3. **Run as administrator** (Windows only, if needed)
4. **Verify system tray is enabled** in your OS

### Compilation Errors
1. **Missing libraries**: Install platform-specific development packages
2. **Wrong platform detection**: Manually define `TRAY_WINAPI`, `TRAY_APPINDICATOR`, or `TRAY_APPKIT`

## License

This software is distributed under [MIT license](http://www.opensource.org/licenses/mit-license.php), so feel free to integrate it in your commercial products.
