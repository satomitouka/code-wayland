# `code-wayland`

## 说明
本fork增加了中文输入法所需的参数

## 概述

`code-wayland` 包为 Visual Studio Code 桌面入口 (`code.desktop`) 提供了一个补丁，以便在基于 Wayland 的系统上实现更好的兼容性。请参阅 [VS Code 应该在可能的情况下默认使用 Wayland](https://github.com/microsoft/vscode/issues/207033) 和 [Wayland 下模糊的文本](https://wiki.archlinux.org/title/Visual_Studio_Code#Blurry_text_under_Wayland)。

该包会在安装或更新 `code` 包时自动修改 `code.desktop` 文件，以便 Visual Studio Code 使用适当的 `--enable-ozone` 和 `--ozone-platform=wayland` 选项运行 Wayland 支持。

## 功能

- **自动补丁应用**: 自动检测 `code.desktop` 文件的更改并重新应用 Wayland 补丁。
- **基于触发器**: 使用 Debian 的触发器系统，以确保在修改 `code` 包时应用补丁。
- **手动重新补丁选项**: 提供一个脚本，可以手动运行以重新应用补丁。

## 安装

1. 确保系统上安装了 `code` 包：

   ```bash
   sudo apt-get install code
   ```

2. 下载并安装 `code-wayland` 包：

   ```bash
   sudo dpkg -i code-wayland_<版本>.deb
   ```

   将 `<版本>` 替换为 `code-wayland` 包的适当版本。

3. 安装后，该包会自动修补 `code.desktop` 文件，以便使用 `--enable-ozone` 标志启用 Wayland 支持。

## 工作原理

`code-wayland` 包使用 Debian 的 `interest-noawait` 触发器来监视 `/usr/share/applications/code.desktop` 文件的更改。当 `code` 包安装或更新时，该触发器激活 `code-wayland` 补丁脚本，以修改桌面文件中的 `Exec` 条目。

### 触发器机制

- **触发器文件**: `debian/triggers`

  ```text
  interest-noawait /usr/share/applications/code.desktop
  ```

- **安装后脚本**: `debian/postinst`

  该脚本在修改 `code` 包时执行。它运行 `patch-code-desktop` 脚本以应用桌面文件所需的更改。

- **补丁脚本**: `/usr/bin/patch-code-desktop`

  补丁脚本检查 `/usr/share/applications/code.desktop` 文件中的 `Exec` 行是否已修改为包含 Wayland 特定的标志 (`--enable-ozone --ozone-platform=wayland`)。如果没有，则更新该行：

  ```bash
  # 原始 Exec 行：
  Exec=/usr/share/code/code

  # 补丁后的 Exec 行：
  Exec=/usr/share/code/code --enable-ozone --ozone-platform=wayland
  ```

## 手动重新安装 `code`

如果需要手动测试或重新应用补丁，可以触发 `code` 包的重新安装。这将导致 `code-wayland` 包自动重新应用补丁：

```bash
sudo apt-get install --reinstall code
```

该命令将触发 `code-wayland` 补丁脚本以重新应用补丁。

## 验证补丁

要确认补丁已正确应用，请打开 `code.desktop` 文件并查找修改后的 `Exec` 行：

```bash
cat /usr/share/applications/code.desktop | grep Exec
```

您应该看到以下行（或类似的行）：

```text
Exec=/usr/share/code/code --enable-ozone --ozone-platform=wayland
```

## 卸载

要从系统中删除 `code-wayland`，请使用以下命令：

```bash
sudo apt-get remove --purge code-wayland
```

卸载包后，如果修改了原始 `code.desktop` 文件，则可能需要手动恢复原始文件：

```bash
sudo apt-get install --reinstall code
```

## 参考

- [`DpkgTriggers`](https://wiki.debian.org/DpkgTriggers)（Debian Wiki 中）。
- [`deb-triggers(5)`](https://manpages.debian.org/bookworm/dpkg-dev/deb-triggers.5.en.html) 手册页。

## 许可

该项目以 GPL-3+ 许可证发布。请参阅 [`LICENSE`] 文件以获取更多详细信息。
