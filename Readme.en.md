# Publish GitHub Pages sites privately from public repositories

[简体中文](./Readme.md) [English](./Readme.en.md)

Use GitHub Actions to privately publish GitHub Pages websites from public repositories, completely hiding the file list and history of the websites, no payment required

## Background

Free GitHub accounts [can only](https://github.com/pricing) publish GitHub Pages sites from public repositories, which raises some privacy issues. Anyone can:

- View file modification history, historical private data is difficult to delete
- View the list of files and easily find unpublished drafts
- Fork repositories at will, and all data will remain public and hard to delete
- Easily download all static website files

## Solution and results

Instead of storing any static website files in GitHub repositories, use GitHub Actions to remotely download static website package file and publish them directly to GitHub Pages.

Results:

- This solution is available for free GitHub accounts, you don't need to pay only for this feature
- File modification history is not public, nobody can view historical private data at will
- The list of files is not public, nobody can easily find secret pages and files
- Nobody can fork, clone, and downloads static website files

## Usage

### Requirements

- A web service that can upload and download files, examples include:
    - A web server (it is recommended to upload to the same path every time, so that the download link will be fixed and you don't need to specify the URL every time you run the workflow)
    - A temporary file-sharing service that provides direct download links, such as [Litterbox](https://litterbox.catbox.moe/) (download links from these services are usually not fixed, so you need to specify the remote file URL parameter each time the workflow runs)
- Static website files that have been generated

### Initialize the code repository

- Fork this repository, change `repository name`, usually it should be `<your lowercase username>.github.io` ([official documents](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages))
- Modify the repository settings:
    - `Settings` - `Actions` - `General` - `Artifact and log retention` set to a minimum value of `1` day
    - `Settings` - `Pages`, change `Source` to `GitHub Actions`
- Enter `Actions`, you will get a warning `Workflows aren’t being run on this forked repository` for the first time, click the `I understand my workflows, go ahead and enable them` button to confirm the warning.

### Set parameters

There are 3 parameters to set:

- `REMOTE_FILE_URL`: required, the URL of the static website package file.
- `REMOTE_FILE_TYPE`: required, the format of the static website package file, options: `7z`, `tar`.
- `REMOTE_FILE_PASSWORD`: optional, the encryption and decryption passphrase of the static website package file. If the file is not encrypted, this parameter does not need to be set.

Parameters can be set in 2 places:

- For fixed parameters: `Settings` - `Secrets` - `Actions`, click `New repository secret`, add them as secrets. You only need to set it here once, and leave it blank when running the workflow.
- For non-fixed parameters: set each time the workflow is run. If the same parameter exists in secrets, it will be overwritten by the value set here.

It is recommended that you use fixed parameters and set them to secrets if possible, rather than specifying them each time you run the workflow. This is because secrets are hidden in the workflow run log, whereas the parameters specified when running the workflow are output directly to the log, which is publicly viewable and cannot be hidden.

### Package the static website files

There are 4 types of packaged files supported, please select the packaged file type according to your preference. Each type and corresponding example file is as follows:

- `demo/test.7z`: compressed with 7-Zip, not encrypted
- `demo/test.enc.7z`: compressed and encrypted using 7-Zip, filenames are also encrypted, the password is `123456`
- `demo/test.tar.gz`: compressed with tar, not encrypted
- `demo/test.tar.gz.enc`: compressed with tar and then encrypted with openssl, the password is `123456`

Assume that the static website files are located in the `/path/to/static/dir` directory, and the password is `YOUR_PASSWORD_123456`. The following are example commands for packaging, running on Ubuntu 20.04.

Compressed to `/path/to/files.7z` using 7z, not encrypted:

```bash
cd /path/to/static/dir && 7z a /path/to/files.7z .
```

Compress and encrypt to `/path/to/files.7z` using 7z, filenames are also encrypted, the encryption and decryption passphrase is hard-coded into the command parameters:

```bash
cd /path/to/static/dir && 7z a -mhe=on -pYOUR_PASSWORD_123456 /path/to/files.7z .
```

You can also use the Windows GUI program to package static website files into 7z format.

Use tar to compress as `./files.tar.gz`, not encrypted:

```bash
tar --owner 0 --group 0 --numeric-owner -czvf files.tar.gz -C /path/to/static/dir .
```

Use tar and openssl to compress and encrypt the files as `./files.tar.gz.enc`, the encryption and decryption passphrase is hard-coded into the command parameters:

```bash
tar --owner 0 --group 0 --numeric-owner -czvf - -C /path/to/static/dir . | openssl enc -aes-256-cbc -pbkdf2 -pass pass:YOUR_PASSWORD_123456 -in - -out files.tar.gz.enc
```

### Deployment

Upload the static website package file to your server or file sharing services. Example command for uploading `/path/to/files.7z` to [Litterbox](https://litterbox.catbox.moe/) using the CLI:

```bash
curl -vv -F 'reqtype=fileupload' -F 'time=1h' -F 'fileNameLength=16' -F 'fileToUpload=@/path/to/files.7z' https://litterbox.catbox.moe/resources/internals/api.php
```

`Actions` - `Deploy to GitHub Pages` - `Run workflow`, fill in the non-fixed parameters, click `Run workflow`, and wait for it to finish. After it is finished:

- If it runs successfully, the workflow run is automatically deleted, no further operation required.
- If the run fails, the workflow run will not be automatically deleted. The logs and artifacts may contain private data and are publicly viewable. Please delete all workflow runs manually after troubleshooting.

Finally, delete the package file from your server, or cancel the file sharing.

It is recommended that the above steps be fixed as a custom script.
