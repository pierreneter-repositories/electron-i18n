# 应用部署

为了使用 Electron 部署你的应用程序，你需要下载 Electron 的 [prebuilt binaries](https://github.com/electron/electron/releases)。 接下来，你存放应用程序的文件夹需要叫做 `app` 并且需要放在 Electron 的 资源文件夹下，如下面的示例所示。 请注意，在下面的示例中，Electron的预制二进制文件的位置用`electron /`表示。

在 macOS 中:

```text
electron/Electron.app/Contents/Resources/app/
├── package.json
├── main.js
└── index.html
```

在 Windows 和 Linux 中:

```text
electron/resources/app
├── package.json
├── main.js
└── index.html
```

然后运行 `Electron.app` (或者 Linux 中的 `electron`，Windows 中的 `electron.exe`), 接着 Electron 就会以你的应用程序的方式启动。`electron` 文件夹将被部署并可以分发给最终的使用者。

## 将你的应用程序打包成一个文件

除了通过拷贝所有的资源文件来分发你的应用程序之外，你可以通过打包你的应用程序为一个 [asar](https://github.com/electron/asar) 库文件以避免暴露你的源代码。

为了使用一个 `asar` 库文件代替 `app` 文件夹，你需要修改这个库文件的名字为 `app.asar` ， 然后将其放到 Electron 的资源文件夹下，然后 Electron 就会试图读取这个库文件并从中启动。 如下所示：

在 macOS 中:

```text
electron/Electron.app/Contents/Resources/
└── app.asar
```

在 Windows 和 Linux 中:

```text
electron/resources/
└── app.asar
```

更多的细节参阅 [Application packaging](application-packaging.md).

## 使用下载的二进制文件重塑

将您的应用程序捆绑到Electron后，您将需要将Electron重新分发给用户。

### Windows

你可以将 `electron.exe` 改成任意你喜欢的名字，然后可以使用像 [rcedit](https://github.com/atom/rcedit) 编辑它的 icon 和其他信息。

### macOS

你可以将 `Electron.app` 改成任意你喜欢的名字，然后你也需要修改这些文件中的 `CFBundleDisplayName`， `CFBundleIdentifier` 以及 `CFBundleName` 字段。 这些文件如下：

* `Electron.app/Contents/Info.plist`
* `Electron.app/Contents/Frameworks/Electron Helper.app/Contents/Info.plist`

你也可以重命名帮助应用程序以避免在应用程序监视器中显示 `Electron Helper`， 但是请确保你已经修改了帮助应用的可执行文件的名字。

一个改过名字的应用程序的构造可能是这样的：

    MyApp.app/Contents
    ├── Info.plist
    ├── MacOS/
    │   └── MyApp
    └── Frameworks/
        ├── MyApp Helper EH.app
        |   ├── Info.plist
        |   └── MacOS/
        |       └── MyApp Helper EH
        ├── MyApp Helper NP.app
        |   ├── Info.plist
        |   └── MacOS/
        |       └── MyApp Helper NP
        └── MyApp Helper.app
            ├── Info.plist
            └── MacOS/
                └── MyApp Helper
    

### Linux

你可以将 `electron` 改成任意你喜欢的名字。

## 打包工具

除了手动打包应用程序，你还可以选择使用第三方打包工具来做这项工作

* [electron-forge](https://github.com/electron-userland/electron-forge)
* [electron-builder](https://github.com/electron-userland/electron-builder)
* [electron-packager](https://github.com/electron-userland/electron-packager)

## 通过重编译源代码来重塑

通过修改产品名称并重编译源代码来更换 Electron 的名称也是可行的。 你需要修改 `atom.gyp` 文件并彻底重编译一次。

### 创建一个自定义 Electron 分支

如果只是为了构建你的 app，你不需要创建一个自定义的 Electron 分支， 即使是“生产级”的应用程序。 可以使用工具，如 `electron-packager` 或 `electron-builder` 来 “重塑” 你的 Electron app。

当你想直接进入 Electron 来修改 C++ 代码，你需要 fork Electron ，不能使用 upstreamed 或者 rejected 官方版本。 作为 Electron 的维护者，我们非常希望 你的方案能够工作，并尽最大的努力让你的修改 进入 Electron 的官方版本，创建自定义分支对你来说更简单，最后， 我们感谢你的帮助。

#### 通过 surf-build 创建一个自定义版本

1. 通过 npm 安装 [Surf](https://github.com/surf-build/surf): `npm install -g surf-build@latest`

2. 创建一个新的 S3 bucket 并按照以下结构创建文件夹：
    
        - atom-shell/
          - symbols/
          - dist/
        

3. 设置以下环境变量：

* `ELECTRON_GITHUB_TOKEN` - 一个在 GitHub 创建版本的 token
* `ELECTRON_S3_ACCESS_KEY`, `ELECTRON_S3_BUCKET`, `ELECTRON_S3_SECRET_KEY` - 你将上传 Node.js 的 headers 以及 symbol 的位置
* `ELECTRON_RELEASE` - 设置为 `true`，上传部分将运行，不设置 和 `surf-build` 只是做 CI-type 的检查，适合每一个拉取请求运行。
* `CI` - 设置为 `true` ，否则无效
* `GITHUB_TOKEN` - 设置为与 `ELECTRON_GITHUB_TOKEN` 相同
* `SURF_TEMP` - 在 Windows 下设置为 `C:\Temp` 来防止路径太长的问题
* `TARGET_ARCH` - 设置为 `ia32` 或 `x64` 

1. 在 `script/upload.py`，你 *必须* 为你的分支(`MYORG/electron`)设置 `ELECTRON_REPO`， 尤其如果你本身是一个 Electron 贡献者。

2. `surf-build -r https://github.com/MYORG/electron -s YOUR_COMMIT -n 'surf-PLATFORM-ARCH'`

3. 需要很长的时间来等待构建完成