# `code-wayland`

## Overview

The `code-wayland` package provides a patch for the Visual Studio Code desktop entry (`code.desktop`) to enable better compatibility with Wayland-based systems. See [VS Code should default to Wayland when possible](https://github.com/microsoft/vscode/issues/207033) and [Blurry text under Wayland](https://wiki.archlinux.org/title/Visual_Studio_Code#Blurry_text_under_Wayland).

This package automatically modifies the `code.desktop` file whenever the `code` package is installed or updated, ensuring that Visual Studio Code runs with Wayland support using the appropriate `--enable-ozone` and `--ozone-platform=wayland` options.

## Features

- **Automatic Patch Application**: Automatically detects changes to the `code.desktop` file and re-applies the Wayland patch.
- **Trigger-Based**: Uses Debian's trigger system to ensure that the patch is applied whenever the `code` package is modified.
- **Manual Re-Patching Option**: Provides a script that can be run manually if needed to re-apply the patch.

## Installation

1. Make sure the `code` package is installed on your system:

   ```bash
   sudo apt-get install code
   ```

2. Download and install the `code-wayland` package:

   ```bash
   sudo dpkg -i code-wayland_<version>.deb
   ```

   Replace `<version>` with the appropriate version of the `code-wayland` package.

3. After installation, the package will automatically patch the `code.desktop` file to enable Wayland support using the `--enable-ozone` flag.

## How It Works

The `code-wayland` package uses a Debian path `interest-noawait` trigger to watch for changes to the `/usr/share/applications/code.desktop` file. When the `code` package is installed or updated, this trigger activates the `code-wayland` patch script to modify the `Exec` entry in the desktop file.

### Trigger Mechanism

- **Trigger File**: `debian/triggers`

  ```text
  interest-noawait /usr/share/applications/code.desktop
  ```

- **Post-Installation Script**: `debian/postinst`

  This script is executed whenever the `code` package is modified. It runs the `patch-code-desktop` script to apply the necessary changes to the desktop file.

- **Patch Script**: `/usr/bin/patch-code-desktop`

  The patching script checks if the `Exec` line in `/usr/share/applications/code.desktop` has been modified to include the Wayland-specific flags (`--enable-ozone --ozone-platform=wayland`). If not, it updates the line accordingly:

  ```bash
  # Original Exec line:
  Exec=/usr/share/code/code

  # Patched Exec line:
  Exec=/usr/share/code/code --enable-ozone --ozone-platform=wayland
  ```

## Manual Reinstallation of `code`

If you need to manually test or reapply the patch, you can trigger the reinstallation of the `code` package. This will cause the `code-wayland` package to reapply the patch automatically:

```bash
sudo apt-get install --reinstall code
```

This command will trigger the `code-wayland` patch script to reapply the patch.

## Verifying the Patch

To confirm that the patch has been applied correctly, open the `code.desktop` file and look for the modified `Exec` line:

```bash
cat /usr/share/applications/code.desktop | grep Exec
```

You should see the following line (or a similar one):

```text
Exec=/usr/share/code/code --enable-ozone --ozone-platform=wayland
```

## Uninstallation

To remove `code-wayland` from your system, use the following command:

```bash
sudo apt-get remove --purge code-wayland
```

After removing the package, you may need to manually restore the original `code.desktop` file if it was modified:

```bash
sudo apt-get install --reinstall code
```

## References

- [`DpkgTriggers`](https://wiki.debian.org/DpkgTriggers) (in the Debian Wiki).
- [`deb-triggers(5)`](https://manpages.debian.org/bookworm/dpkg-dev/deb-triggers.5.en.html) manpage.

## License

This project is licensed under the GPL-3+ License. See the [`LICENSE`] file for more details.
