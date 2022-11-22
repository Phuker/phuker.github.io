# 从公开仓库私密发布 GitHub Pages 网站

[简体中文](./Readme.md) [English](./Readme.en.md)

从公开仓库私密发布 GitHub Pages 网站，类似使用私有仓库

## 背景和原因

GitHub 免费账户[限制](https://github.com/pricing)只能从公开仓库发布 GitHub Pages 网站，这会带来一些隐私问题。任何人都可以：

- 查看文件修改历史记录，历史隐私数据难以删除
- 查看文件列表，轻松找到未发布的草稿
- 随意 fork 仓库，所有数据将保持公开，难以删除
- 轻松打包下载所有静态网站文件

## 原理和效果

不在 GitHub 仓库中存储任何静态网站文件，而是使用 GitHub Actions 远程下载静态网站打包文件并直接发布到 GitHub Pages。

效果：

- GitHub 免费账户也可以使用
- 文件修改历史记录不公开，无法随意查看历史隐私数据
- 文件列表不公开，无法轻松找到存储到秘密路径的页面文件
- Fork、clone、打包下载不会涉及网站数据

## 使用方法

### 需求

- 可以上传和下载文件的 web 服务，例如：
    - Web 服务器（推荐每次都上传到相同路径，下载链接固定，不需要每次运行 workflow 都指定参数）
    - 可以直链下载的文件共享服务，例如 [file.io](https://www.file.io/)（这类服务的下载链接一般都不固定，每次运行 workflow 都需要指定参数）
- 已生成的静态网站文件

### 首次初始化设置

- Fork 本仓库，`repository name` 改为想要的名字，一般是`<用户名小写>.github.io`（[官方文档](https://docs.github.com/cn/pages/getting-started-with-github-pages/about-github-pages)）
- 修改仓库设置：
    - `Settings` - `Actions` - `General` - `Artifact and log retention` 设置为最小值 `1` day
    - `Settings` - `Secrets` - `Actions`，点击 `New repository secret`，添加 2 个参数：
        - `REMOTE_FILE_URL`：如果静态网站打包文件的 URL 是固定的，将此参数设置为该 URL；如果 URL 是动态的，则无需设置此参数，而是在运行 workflow 时指定。
        - `REMOTE_FILE_PASSWORD`：建议设置为固定的文件加解密口令。你也可以在每次运行 workflow 时填写一次性的加解密口令。如果两处均未指定，则下载静态网站打包文件后，不会运行解密步骤。
        - 如果以上参数是固定的，请务必在此处设置为 secrets，而不要在运行 workflow 是指定参数。因为运行 workflow 时指定的参数会直接输出到 workflow run 的日志中，可公开查看，无法隐藏。如果同时在 secrets 和运行 workflow 时指定以上参数，后者将会覆盖前者。
    - `Settings` - `Pages`，`Source` 改为 `GitHub Actions`
- 首次进入 `Actions` 会出现警告 `Workflows aren’t being run on this forked repository`，点击 `I understand my workflows, go ahead and enable them` 按钮确认该警告。

### 部署

- 假设静态网站文件位于 `/path/to/static/dir` 目录，先打包压缩为 `./files.tar.gz`，然后加密为 `./files.tar.gz.enc`。命令示例：

```bash
tar --owner 0 --group 0 --numeric-owner -czvf files.tar.gz -C /path/to/static/dir .

# 启动此命令后，输入加密解密口令
openssl enc -aes-256-cbc -pbkdf2 -pass stdin -in files.tar.gz -out files.tar.gz.enc

# ------ 或者 ------

# 合并以上两条命令，加密解密口令硬编码到命令参数
tar --owner 0 --group 0 --numeric-owner -czvf - -C /path/to/static/dir . | openssl enc -aes-256-cbc -pbkdf2 -pass pass:YOUR_PASSWORD_123456 -in - -out files.tar.gz.enc
```

- 把 `./files.tar.gz.enc` 文件上传到你的服务器
- `Actions` - `Deploy to GitHub Pages` - `Run workflow`，点击 `Run workflow`，等待运行完毕。workflow run 中，日志、artifacts 仍然包含隐私数据，可公开查看。运行完毕后：
    - 如果运行成功，会自动删除 workflow run，无需进一步操作
    - 如果运行失败，需要在排除错误后手动删除所有 workflow run
- 删除你的服务器上的 `files.tar.gz.enc` 文件

建议将以上步骤固定为自定义脚本。
