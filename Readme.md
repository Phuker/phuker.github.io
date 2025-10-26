# 从公开仓库私密发布 GitHub Pages 网站

[简体中文](./Readme.md) [English](./Readme.en.md)

使用 GitHub Actions 从公开仓库私密发布 GitHub Pages 网站，完全隐藏网站文件列表和历史记录，无需付费

## 背景和原因

GitHub 免费账户[限制](https://github.com/pricing)只能从公开仓库发布 GitHub Pages 网站，这会带来一些隐私问题。任何人都可以：

- 查看文件修改历史记录，历史隐私数据难以删除
- 查看文件列表，轻松找到未发布的草稿
- 随意 fork 仓库，所有数据将保持公开，难以删除
- 轻松打包下载所有静态网站文件

## 原理和效果

不在 GitHub 仓库中存储任何静态网站文件，而是使用 GitHub Actions 远程下载静态网站打包文件并直接发布到 GitHub Pages。

效果：

- GitHub 免费账户也可以使用，不需要仅为此功能付费
- 文件修改历史记录不公开，无法随意查看历史隐私数据
- 文件列表不公开，无法轻松找到存储到秘密路径的页面文件
- Fork、clone、打包下载不会涉及网站数据

## 使用方法

### 需求

- 可以上传和下载文件的 web 服务，例如：
    - Web 服务器（推荐每次都上传到相同路径，下载链接固定，不需要每次运行 workflow 都指定参数）
    - 可以直链下载的文件临时共享服务，例如 [Litterbox](https://litterbox.catbox.moe/)（这类服务的下载链接一般都不固定，每次运行 workflow 都需要指定参数）
- 已生成的静态网站文件

### 初始化仓库

- Fork 本仓库，`repository name` 改为想要的名字，一般是`<用户名小写>.github.io`（[官方文档](https://docs.github.com/cn/pages/getting-started-with-github-pages/about-github-pages)）
- 修改仓库设置：
    - `Settings` - `Actions` - `General` - `Artifact and log retention` 设置为最小值 `1` day
    - `Settings` - `Pages`，`Source` 改为 `GitHub Actions`
- 进入 `Actions`，首次进入会出现警告 `Workflows aren’t being run on this forked repository`，点击 `I understand my workflows, go ahead and enable them` 按钮确认该警告。

### 设置参数

共有 3 个参数需要设置：

- `REMOTE_FILE_URL`：必须设置，静态网站打包文件的 URL。
- `REMOTE_FILE_TYPE`：必须设置，静态网站打包文件的格式，可选选项：`7z`，`tar`。
- `REMOTE_FILE_PASSWORD`：可选，静态网站打包文件的加密解密口令（密码）。如果未加密，则不需要设置此参数。

可以在 2 个位置设置参数：

- 固定参数：`Settings` - `Secrets` - `Actions`，点击 `New repository secret`，添加为 secrets。仅需在此处设置 1 次，在运行 workflow 时留空，无需设置。
- 非固定参数：在每次运行 workflow 时设置。如果 secrets 中存在相同的参数，会被此处填写的值覆盖。

建议尽量使用固定的参数并设置为 secrets，而不要在每次运行 workflow 时指定参数。因为 secrets 参数会在 workflow run 的日志中隐藏，而运行 workflow 时指定的参数会直接输出到日志中，可公开查看，无法隐藏。

### 打包静态网站文件

共支持 4 种打包文件，请按需选择打包文件类型。各种类型及示例文件如下：

- `demo/test.7z`：使用 7-Zip 打包压缩，未加密
- `demo/test.enc.7z`：使用 7-Zip 打包压缩并加密，加密文件名，密码为 `123456`
- `demo/test.tar.gz`：使用 tar 打包压缩，未加密
- `demo/test.tar.gz.enc`：使用 tar 打包压缩，然后使用 openssl 加密，密码为 `123456`

假设静态网站文件位于 `/path/to/static/dir` 目录，密码为 `YOUR_PASSWORD_123456`。以下是打包的命令示例，运行环境为 Ubuntu 20.04。

使用 7z 打包压缩为 `/path/to/files.7z`，未加密：

```bash
cd /path/to/static/dir && 7z a /path/to/files.7z .
```

使用 7z 打包压缩并加密为 `/path/to/files.7z`，加密文件名，加密解密口令硬编码到命令参数：

```bash
cd /path/to/static/dir && 7z a -mhe=on -pYOUR_PASSWORD_123456 /path/to/files.7z .
```

也可以使用 Windows 图形化界面程序将静态网站文件打包为 7z 格式。

使用 tar 打包压缩为 `./files.tar.gz`，未加密：

```bash
tar --owner 0 --group 0 --numeric-owner -czvf files.tar.gz -C /path/to/static/dir .
```

使用 tar 和 openssl 打包压缩并加密为 `./files.tar.gz.enc`，加密解密口令硬编码到命令参数：

```bash
tar --owner 0 --group 0 --numeric-owner -czvf - -C /path/to/static/dir . | openssl enc -aes-256-cbc -pbkdf2 -pass pass:YOUR_PASSWORD_123456 -in - -out files.tar.gz.enc
```

### 部署

把打包文件上传到你的服务器或者文件共享服务。使用命令行将打包文件 `/path/to/files.7z` 上传到 [Litterbox](https://litterbox.catbox.moe/) 示例：

```bash
curl -vv -F 'reqtype=fileupload' -F 'time=1h' -F 'fileNameLength=16' -F 'fileToUpload=@/path/to/files.7z' https://litterbox.catbox.moe/resources/internals/api.php
```

`Actions` - `Deploy to GitHub Pages` - `Run workflow`，填写非固定参数，点击 `Run workflow`，等待运行完毕。运行完毕后：

- 如果运行成功，workflow run 会自动删除，无需进一步操作。
- 如果运行失败，workflow run 不会自动删除，其中的日志、artifacts 可能包含隐私数据，并且可以公开查看。请在排除错误后手动删除所有 workflow run。

最后删除服务器上的打包文件，取消文件共享。

建议将以上打包和部署步骤固定为自定义脚本。
