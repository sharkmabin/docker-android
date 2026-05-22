# Docker-Android 中文文档

Docker-Android 是一个面向 Android 开发、自动化测试和持续集成场景的 Docker 镜像项目。它可以在容器中运行 Android 模拟器，并通过 Web VNC、VNC 客户端、ADB、Appium 等方式进行访问和控制。

[English README](./README.md)

## 项目特性

- 支持多种 Android 版本镜像，包括 Android 9 到 Android 14。
- 内置多种设备配置和皮肤，例如 Samsung Galaxy S10、Samsung Galaxy S9、Nexus 5、Nexus 7 等。
- 支持 Web VNC，可以直接在浏览器中查看和操作模拟器。
- 支持从宿主机通过 ADB 连接和控制模拟器。
- 支持日志 Web UI，便于查看容器内日志。
- 可用于构建 Android 项目、运行单元测试和 UI 自动化测试。
- 可与 Appium、Espresso、Selenium Grid、Jenkins 以及 Genymotion Cloud 等工具或平台配合使用。

## 可用镜像

| Android 版本 | API | 最新版本镜像 | 指定发布版本镜像 |
| --- | --- | --- | --- |
| 9.0 | 28 | `budtmo/docker-android:emulator_9.0` | `budtmo/docker-android:emulator_9.0_<release_version>` |
| 10.0 | 29 | `budtmo/docker-android:emulator_10.0` | `budtmo/docker-android:emulator_10.0_<release_version>` |
| 11.0 | 30 | `budtmo/docker-android:emulator_11.0` | `budtmo/docker-android:emulator_11.0_<release_version>` |
| 12.0 | 32 | `budtmo/docker-android:emulator_12.0` | `budtmo/docker-android:emulator_12.0_<release_version>` |
| 13.0 | 33 | `budtmo/docker-android:emulator_13.0` | `budtmo/docker-android:emulator_13.0_<release_version>` |
| 14.0 | 34 | `budtmo/docker-android:emulator_14.0` | `budtmo/docker-android:emulator_14.0_<release_version>` |
| - | - | `budtmo/docker-android:genymotion` | `budtmo/docker-android:genymotion_<release_version>` |

## 支持的设备

| 类型 | 设备名称 |
| --- | --- |
| Phone | Samsung Galaxy S10 |
| Phone | Samsung Galaxy S9 |
| Phone | Samsung Galaxy S8 |
| Phone | Samsung Galaxy S7 Edge |
| Phone | Samsung Galaxy S7 |
| Phone | Samsung Galaxy S6 |
| Phone | Nexus 4 |
| Phone | Nexus 5 |
| Phone | Nexus One |
| Phone | Nexus S |
| Tablet | Nexus 7 |
| Tablet | Pixel C |

## 运行要求

1. 宿主机已安装 Docker。
2. 宿主机支持硬件虚拟化。
3. 镜像主要面向 Ubuntu 环境运行。macOS 和 Windows 用户通常需要使用支持嵌套虚拟化的 Ubuntu 虚拟机，或者在 Windows 11 WSL2 中正确启用 KVM。

在 Ubuntu 中可以使用以下命令检查 KVM 状态：

```shell
sudo apt install cpu-checker
kvm-ok
```

## 快速开始

启动一个 Android 11 模拟器，并启用 Web VNC：

```shell
docker run -d \
  -p 6080:6080 \
  -e EMULATOR_DEVICE="Samsung Galaxy S10" \
  -e WEB_VNC=true \
  --device /dev/kvm \
  --name android-container \
  budtmo/docker-android:emulator_11.0
```

启动后，在浏览器中打开：

```text
http://localhost:6080
```

查看模拟器状态：

```shell
docker exec -it android-container cat device_status
```

## 持久化数据

默认情况下，容器重启后模拟器数据会被销毁。如果需要保留数据，请将卷挂载到 `/home/androidusr`：

```shell
docker run -v data:/home/androidusr budtmo/docker-android:emulator_11.0
```

## Windows 11 WSL2 硬件加速

将当前用户加入 `kvm` 用户组：

```shell
sudo usermod -a -G kvm ${USER}
```

在 `/etc/wsl.conf` 中加入：

```ini
[boot]
command = /bin/bash -c 'chown -v root:kvm /dev/kvm && chmod 660 /dev/kvm'
```

在 PowerShell 中打开 `%USERPROFILE%\.wslconfig`，加入：

```ini
[wsl2]
nestedVirtualization=true
```

然后重启 WSL2：

```shell
wsl --shutdown
```

如果当前 WSL 版本不支持 `.wslconfig` 中的配置，可以参考英文 README 中的说明，将配置合并到 `/etc/wsl.conf`。

## 常用配置

### Web VNC

Web VNC 默认使用 `6080` 端口：

```shell
docker run -d \
  -p 6080:6080 \
  -e EMULATOR_DEVICE="Samsung Galaxy S10" \
  -e WEB_VNC=true \
  --device /dev/kvm \
  --name android-container \
  budtmo/docker-android:emulator_11.0
```

常用环境变量：

| 环境变量 | 说明 | 示例 |
| --- | --- | --- |
| `WEB_VNC` | 启用浏览器访问容器内 VNC | `-e WEB_VNC=true` |
| `WEB_VNC_PORT` | 设置 Web VNC 端口，默认 `6080` | `-e WEB_VNC_PORT=6081 -p 6081:6081` |
| `VNC_PASSWORD` | 设置 VNC 连接密码 | `-e VNC_PASSWORD=thisissecret` |

可用访问参数：

| 参数 | 说明 | 示例 |
| --- | --- | --- |
| `autoconnect` | 自动连接 Web VNC | `http://localhost:6080/?autoconnect=true` |
| `view_only` | 只读模式 | `http://localhost:6080/?autoconnect=true&view_only=true` |
| `password` | 使用密码连接 | `http://localhost:6080/?autoconnect=true&password=thisissecret` |

### 日志 Web UI

| 环境变量 | 说明 | 示例 |
| --- | --- | --- |
| `WEB_LOG` | 启用日志 Web UI | `-e WEB_LOG=true` |
| `WEB_LOG_PORT` | 设置日志 Web UI 端口，默认 `9000` | `-e WEB_LOG=true -e WEB_LOG_PORT=9001` |

### 模拟器配置

| 环境变量 | 说明 | 示例 |
| --- | --- | --- |
| `EMULATOR_DEVICE` | 指定模拟器设备 | `-e EMULATOR_DEVICE="Samsung Galaxy S10"` |
| `EMULATOR_NAME` | 指定模拟器名称 | `-e EMULATOR_NAME=my_emu` |
| `EMULATOR_DATA_PARTITION` | 设置数据分区大小，默认 `550m` | `-e EMULATOR_DATA_PARTITION=900m` |
| `EMULATOR_NO_SKIN` | 不加载设备皮肤 | `-e EMULATOR_NO_SKIN=true` |
| `EMULATOR_ADDITIONAL_ARGS` | 传递额外 Android Emulator 参数 | `-e EMULATOR_ADDITIONAL_ARGS="-no-snapshot"` |

如果需要覆盖模拟器配置文件，可以设置 `EMULATOR_CONFIG_PATH`，并将对应文件通过 Docker volume 挂载到容器内。更多配置请参考 [Custom Configurations](./documentations/CUSTOM_CONFIGURATIONS.md)。

## Appium 自动化测试

启用 Appium Server 需要开放 `4723` 端口，并设置 `APPIUM=true`：

```shell
docker run -d \
  -p 6080:6080 \
  -p 4723:4723 \
  -e EMULATOR_DEVICE="Samsung Galaxy S10" \
  -e WEB_VNC=true \
  -e APPIUM=true \
  --device /dev/kvm \
  --name android-container \
  budtmo/docker-android:emulator_11.0
```

可以通过 `APPIUM_ADDITIONAL_ARGS` 向 Appium Server 传递额外参数。更多说明请参考 [UI-Test with Appium](./documentations/USE_CASE_APPIUM.md)。

## 常见使用场景

- [构建 Android 项目](./documentations/USE_CASE_BUILD_ANDROID_PROJECT.md)
- [使用 Appium 进行 UI 测试](./documentations/USE_CASE_APPIUM.md)
- [在宿主机控制 Android 模拟器](./documentations/USE_CASE_CONTROL_EMULATOR.md)
- [短信模拟](./documentations/USE_CASE_SMS.md)
- [Jenkins 集成](./documentations/USE_CASE_JENKINS.md)
- [部署到云平台 Azure、AWS、GCP](./documentations/USE_CASE_CLOUD.md)

## Genymotion

如果没有足够资源维护本地模拟器，或者需要更多设备配置，可以尝试使用 [Genymotion SaaS](https://cloud.geny.io/)。Docker-Android 已与 Genymotion 在 Genymotion SaaS、AWS、GCP、Alibaba Cloud 等场景中集成。更多信息请参考 [Third Party Genymotion](./documentations/THIRD_PARTY_GENYMOTION.md)。

## Pro 版本

项目维护者提供了 sponsor-based 的 Docker-Android Pro 版本。Pro 版本包含普通版本没有的能力，例如代理配置、语言配置、新 Android 版本、root 权限、headless 模式、Selenium 4.x 集成等。详情请参考 [Docker-Android Pro](./documentations/DOCKER-ANDROID-PRO.md)。

## 许可证

请查看 [LICENSE.md](./LICENSE.md)。
