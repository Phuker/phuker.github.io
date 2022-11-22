# Publish GitHub Pages sites privately from public repositories

[简体中文](./Readme.md) [English](./Readme.en.md)

Publish GitHub Pages sites privately from public repositories, the effect is similar to using a private repository

## Background

Free GitHub accounts [can only](https://github.com/pricing) publish GitHub Pages sites from public repositories, which raises some privacy issues. Anyone can:

- View file modification history, historical private data is difficult to delete
- View the list of files and easily find unpublished drafts
- Fork repositories at will, and all data will remain public and hard to delete
- Easily download all static website files

## Solution and results

Instead of storing any static website files in GitHub repositories, use GitHub Actions to remotely download static website package file and publish them directly to GitHub Pages.

Results:

- This solution is also available for free GitHub accounts
- File modification history is not public, nobody can view historical private data at will
- The list of files is not public, nobody can easily find secret pages and files
- Nobody can fork, clone, and downloads static website files

## Usage

### Requirements

- A web service that can upload and download files, examples include:
    - A web server (it is recommended to upload to the same path every time, so that the download link will be fixed and you don't need to specify the URL every time you run the workflow)
    - A file sharing service that provides direct download links, such as [file.io](https://www.file.io/) (file sharing services generally provide dynamic download links, you need to specify the URL every time you run the workflow)
- Static website files that have been generated

### Initialization

- Fork this repository, change `repository name`, usually it should be `<your lowercase username>.github.io` ([official documents](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages))
- Modify the repository settings:
    - `Settings` - `Actions` - `General` - `Artifact and log retention` set to a minimum value of `1` day
    - `Settings` - `Secrets` - `Actions`, click `New repository secret`, and add 2 parameters:
        - `REMOTE_FILE_URL`: if the URL of the static website package file is fixed, set this parameter to the URL; if the URL is dynamic, you do not need to set this parameter, but specify the URL when you run the workflow.
        - `REMOTE_FILE_PASSWORD`: it is recommended to set a fixed password for file encryption and decryption. If you prefer to use a dynamic password, you do not need to set this parameter, but specify a password each time you run the workflow. If not specified in both places, the decryption step will be skipped after downloading the static website package file.
        - If any of the above parameters are fixed, set them as secrets here, and don't specify them when running the workflow. The reason is that the parameters specified when running workflow are output directly to the workflow run log, which is publicly viewable and cannot be hidden. If you specify these parameters both in secrets and when running the workflow, the latter will overwrite the former.
    - `Settings` - `Pages`, change `Source` to `GitHub Actions`
- The first time you enter `Actions`, you will get a warning `Workflows aren’t being run on this forked repository`, click the `I understand my workflows, go ahead and enable them` button to confirm the warning.

### Deployment

Assuming the static website files are located in `/path/to/static/dir` directory, package and compress them as `./files.tar.gz` and then encrypt them as `./files.tar.gz.enc`. Command example:

```bash
tar --owner 0 --group 0 --numeric-owner -czvf files.tar.gz -C /path/to/static/dir .

# Enter your password after this command is started
openssl enc -aes-256-cbc -pbkdf2 -pass stdin -in files.tar.gz -out files.tar.gz.enc

# ------ Or ------

# Combine the two commands above, the password is hard-coded into the command parameters
tar --owner 0 --group 0 --numeric-owner -czvf - -C /path/to/static/dir . | openssl enc -aes-256-cbc -pbkdf2 -pass pass:YOUR_PASSWORD_123456 -in - -out files.tar.gz.enc
```

- Upload `./files.tar.gz.enc` to your server
- `Actions` - `Deploy to GitHub Pages` - `Run workflow`, click `Run workflow`, and wait for it to finish. In the workflow run, logs and artifacts will still contain private data and can be viewed publicly. After it is finished:
    - If it runs successfully, the workflow run is automatically deleted, no further operation required
    - If the run fails, you need to manually delete all workflow runs after troubleshooting
- Delete the `files.tar.gz.enc file` from your server

It is recommended that the above steps be fixed as a custom script.
