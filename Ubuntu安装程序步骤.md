# Ubuntu（VMware 虚拟机）安装 Clash.Verge `.deb` 全流程（含拖拽传输 & 依赖修复）

> 环境示例：Ubuntu 22.04 (jammy) / amd64（x86_64）
> 场景：`.deb` 已在 Windows 下载，通过 VMware 拖拽进入 Ubuntu 后完成安装与启动

------

## 0. 前置检查：系统版本与架构

```bash
uname -m
. /etc/os-release && echo "$VERSION_ID  $VERSION_CODENAME"
```

- `x86_64` → 选择 `amd64` 的 `.deb`
- `aarch64` → 选择 `arm64` 的 `.deb`

------

## 1. 确认 `.deb` 已进入 Ubuntu（常见：拖拽缓存目录）

通过查找文件确认拖拽是否成功：

```bash
sudo find /home/min -name "Clash.Verge*.deb" 2>/dev/null
```

示例输出：

```
/home/min/.cache/vmware/drag_and_drop/xxxxxx/Clash.Verge_2.4.5_amd64.deb
```

> 注意：拖拽缓存目录可能被清理，建议复制到固定目录再安装。

------

## 2. 创建安装目录并复制 `.deb`

```bash
mkdir -p ~/Downloads/clashverge
cp "/home/min/.cache/vmware/drag_and_drop/xxxxxx/Clash.Verge_2.4.5_amd64.deb" ~/Downloads/clashverge/
cd ~/Downloads/clashverge
ls -lh
```

------

## 3. 安装工具与依赖（关键：libwebkit2gtk-4.1-0）

### 3.1 使用 gdebi（推荐）

```bash
sudo apt update
sudo apt install -y gdebi-core
```

### 3.2 若提示依赖缺失 `libwebkit2gtk-4.1-0`（无法定位软件包）

表现如下：

```
Dependency is not satisfiable: libwebkit2gtk-4.1-0
E: 无法定位软件包 libwebkit2gtk-4.1-0
```

解决：修复/补全 Ubuntu 软件源（以 jammy 为例），并刷新索引。

> 先备份 sources.list（可选但推荐）：

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%F_%H%M%S) 2>/dev/null || true
```

写入标准 jammy 源（含 universe/multiverse）：

```bash
sudo tee /etc/apt/sources.list >/dev/null <<'EOF'
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu jammy-security main restricted universe multiverse
EOF
```

清理并更新索引：

```bash
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update
```

确认能找到依赖包：

```bash
apt-cache search libwebkit2gtk-4.1-0
```

安装依赖：

```bash
sudo apt install -y libwebkit2gtk-4.1-0
```

------

## 4. 安装 Clash.Verge `.deb`

```bash
cd ~/Downloads/clashverge
sudo gdebi -n ./Clash.Verge_2.4.5_amd64.deb
```

> 备用方案（不推荐优先）：dpkg + 修复依赖

```bash
sudo dpkg -i ./Clash.Verge_2.4.5_amd64.deb
sudo apt -f install -y
```

------

## 5. 验证安装是否成功

```bash
dpkg -l | grep -i verge
```

成功示例：

```
ii  clash-verge  2.4.5  amd64  Clash Verge Rev
```

------

## 6. 启动程序（GUI / 命令行）

### 6.1 查找桌面入口

```bash
ls /usr/share/applications | grep -i verge
```

示例：

```
Clash Verge.desktop
```

### 6.2 使用 gtk-launch 启动（注意空格）

```bash
gtk-launch "Clash Verge"
```

> 若写成 `gtk-launch clash-verge` 会失败，因为桌面 ID 不匹配且文件名包含空格。

### 6.3 若仍无法启动：查看 Exec 并按 Exec 直接运行

```bash
grep -nE '^(Name|Exec|TryExec)=' "/usr/share/applications/Clash Verge.desktop"
```

------

## 7. 关闭程序 / 关闭代理（可选）

### 7.1 关闭 Clash Verge 程序（强制退出）

```bash
pkill -f clash-verge
```

### 7.2 关闭系统代理（恢复直连）

```bash
gsettings set org.gnome.system.proxy mode 'none'
```

------

## 8. 常见问题排查速查

### 8.1 依赖找不到（无法定位软件包）

- 多数是软件源不全 / 缺 universe / sources.list 被改坏
  → 参考第 3.2 步恢复 jammy 源并 `sudo apt update`

### 8.2 gtk-launch 启动失败

- `.desktop` 文件名含空格或大小写不同
  → 用 `gtk-launch "Clash Verge"`
  → 或查看 `.desktop` 内的 `Exec=` 并直接运行

------









# Ubuntu 安装程序的通用步骤（.deb / apt / AppImage / tar.gz / Snap/Flatpak）

> 目标：给出一套“拿到安装包/安装方式后，从准备环境到安装、启动、卸载、排障”的通用流程。

------

## 0. 先确认系统信息（决定选包与架构）

```bash
uname -m
. /etc/os-release && echo "$PRETTY_NAME"
```

- `x86_64` → 选 `amd64/x64`
- `aarch64/arm64` → 选 `arm64`
- `VERSION_CODENAME`（如 `jammy/noble/focal`）会影响依赖是否存在

------

## 1. 建议先做的通用准备

### 1.1 更新软件索引

```bash
sudo apt update
```

### 1.2 建一个“安装包目录”（便于管理）

```bash
mkdir -p ~/Downloads/packages
cd ~/Downloads/packages
```

### 1.3 如果安装包来自 Windows/外部（VM/拷贝/拖拽）

- 将文件复制到自己的目录（不要长期依赖缓存目录）

```bash
cp "/path/to/file" ~/Downloads/packages/
ls -lh
```

------

## 2. 安装方式一：APT 仓库安装（推荐，最稳定）

适用：软件在 Ubuntu 官方仓库或你已添加第三方仓库

```bash
sudo apt update
sudo apt install -y <package-name>
```

验证：

```bash
dpkg -l | grep -i <keyword>
```

卸载：

```bash
sudo apt remove -y <package-name>
sudo apt purge -y <package-name>
sudo apt autoremove -y
```

------

## 3. 安装方式二：`.deb` 安装（最常见）

适用：你下载到一个 `*.deb` 安装包

### 3.1 推荐：gdebi（自动处理依赖）

```bash
sudo apt install -y gdebi-core
sudo gdebi -n ./your-package.deb
```

### 3.2 备用：dpkg + 修复依赖

```bash
sudo dpkg -i ./your-package.deb
sudo apt -f install -y
```

### 3.3 安装前查看依赖（有助于排错）

```bash
dpkg-deb -I ./your-package.deb | sed -n '/Depends:/p'
```

------

## 4. 安装方式三：AppImage（免安装/依赖少）

适用：单文件可执行（`.AppImage`）

```bash
chmod +x ./YourApp.AppImage
./YourApp.AppImage
```

可选：放到统一目录

```bash
mkdir -p ~/Apps
mv ./YourApp.AppImage ~/Apps/
```

------

## 5. 安装方式四：tar.gz / zip 压缩包（绿色软件）

适用：官网下载的压缩包，解压后直接运行

```bash
mkdir -p ~/Apps/yourapp
tar -xvf yourapp.tar.gz -C ~/Apps/yourapp
# 或 unzip yourapp.zip -d ~/Apps/yourapp
```

运行（示例）：

```bash
cd ~/Apps/yourapp
./yourapp
```

------

## 6. 安装方式五：Snap / Flatpak

### 6.1 Snap

```bash
sudo snap install <name>
sudo snap remove <name>
```

### 6.2 Flatpak（需要先装 flatpak 并添加仓库）

```bash
sudo apt install -y flatpak
# 常用仓库（例如 flathub）按需添加
flatpak install <remote> <app-id>
flatpak run <app-id>
flatpak uninstall <app-id>
```

------

## 7. 启动程序的通用方法

### 7.1 GUI 启动

- “显示应用/Applications”里搜索软件名启动（最推荐）

### 7.2 命令行启动（找可执行文件）

```bash
command -v <binary-name> || true
ls /usr/share/applications | head
```

### 7.3 使用 `.desktop` 启动（注意大小写/空格）

```bash
# 先找 desktop 文件
ls /usr/share/applications | grep -i <keyword>

# 用 gtk-launch 启动：去掉 .desktop，并保持名字一致（有空格要加引号）
gtk-launch "App Name"
```

------

## 8. 常见问题排障模板

### 8.1 “无法定位软件包”

原因：源没配好 / 没启用 universe / 索引没更新 / 网络问题
处理：

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository -y universe
sudo apt update
```

### 8.2 `.deb` 提示依赖不可满足

处理（优先顺序）：

1. 用 `gdebi` 安装
2. `sudo apt -f install -y` 修复依赖
3. 检查系统版本是否过老（包要求更高版本库）
4. 必要时升级系统或换适配版本的软件包

### 8.3 GUI 程序打不开/闪退

常见原因：缺少图形依赖、Wayland/Xorg 兼容、权限或库版本问题
通用排查：

```bash
# 从终端直接运行，看报错
<binary-name>

# 查看 desktop 里的 Exec
grep -nE '^(Name|Exec|TryExec)=' "/usr/share/applications/<file>.desktop"
```

------

## 9. 安装后检查清单（推荐做一次）

```bash
# 1) 包是否安装
dpkg -l | grep -i <keyword> || true

# 2) 可执行文件是否存在
command -v <binary-name> || true

# 3) 桌面入口是否存在
ls /usr/share/applications | grep -i <keyword> || true
```

------

## 10. 最实用的“通用模板”（拿到安装包就套用）

### APT 安装模板

```bash
sudo apt update
sudo apt install -y <package>
```

### `.deb` 安装模板（推荐）

```bash
sudo apt update
sudo apt install -y gdebi-core
sudo gdebi -n ./package.deb
```

### AppImage 模板

```bash
chmod +x ./app.AppImage
./app.AppImage
```