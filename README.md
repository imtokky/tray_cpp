Tray
----
## Warning! We have not verified Mac and Linux compatibility.

Cross-platform, single header, super tiny C99 implementation of a system tray icon with a popup menu.

Works well on:

* Linux/Gtk (libappindicator)
* Windows XP or newer (shellapi.h)
* MacOS (Cocoa/AppKit)

There is also a stub implementation that returns errors on attempt to create a tray menu.

# Setup

Before you can compile `tray`, you'll need to add an environment definition before the line where you include `tray.h`. 

**For Windows:**
```c
#include <stdio.h>
#include <string.h>

#include "tray.h"
...
```

**For Linux:**
```c
#include <stdio.h>
#include <string.h>

#define TRAY_APPINDICATOR 1

#include "tray.h"
...
```

**For Mac:**
```c
#include <stdio.h>
#include <string.h>

#define TRAY_APPKIT 1

#include "tray.h"
...
```

# Demo

The included example `.c` files can be compiled based on your environment.

For example, to compile and run the program on Windows: 

```shell
$> gcc example_windows.c [Enter]
``` 

This will compile and build `a.out`. To run it: 

```
$> a [Enter]
```

# Example

```c++
void quit_cb(struct tray_menu* item);

static struct tray_menu menu_items[] = {
    {"Quit", 0, 0, quit_cb, nullptr},
    {nullptr, 0, 0, nullptr, nullptr}
};

static struct tray tray = {
    "C:\\Users\\path\\...\\icon.ico",
    menu_items
};

static void quit_cb(struct tray_menu* item) {
    (void)item;
    should_exit = true;
    if (g_assistant) {
        cout << "\nEnding processing..." << endl;
        g_assistant->stop();
    }
    tray_exit();
}

...

int main() {
    if (tray_init(&tray) < 0) {
    	cerr << "Failed to initialize tray icon" << endl;
        return -1;
    }

    while (!should_exit && tray_loop(1) == 0) {}
    tray_exit();
    return 0;
}

```

# API

Tray structure defines an icon and a menu.
Menu is a NULL-terminated array of items.
Menu item defines menu text, menu checked and disabled (grayed) flags and a
callback with some optional context pointer.

```c
struct tray {
  char *icon;
  struct tray_menu *menu;
};

struct tray_menu {
  char *text;
  int disabled;
  int checked;

  void (*cb)(struct tray_menu *);
  void *context;

  struct tray_menu *submenu;
};
```

* `int tray_init(struct tray *)` - creates tray icon. Returns -1 if tray icon/menu can't be created.
* `void tray_update(struct tray *)` - updates tray icon and menu.
* `int tray_loop(int blocking)` - runs one iteration of the UI loop. Returns -1 if `tray_exit()` has been called.
* `void tray_exit()` - terminates UI loop.

All functions are meant to be called from the UI thread only.

Menu arrays must be terminated with a NULL item, e.g. the last item in the
array must have text field set to NULL.

## License

This software is distributed under [MIT license](http://www.opensource.org/licenses/mit-license.php),
 so feel free to integrate it in your commercial products.

