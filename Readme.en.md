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

Instead of storing any static website files in GitHub repositories, download static website files remotely through GitHub Actions and publish them directly to GitHub Pages.

Results:

- This solution is also available for free GitHub accounts
- File modification history is not public, nobody can view historical private data at will
- The list of files is not public, nobody can easily find secret pages and files
- Nobody can fork, clone, and downloads static website files

## Usage

### Requirements

- A web server, or a file sharing service with direct download links (preferably fixed download links)
- Static website files that have been generated

### Initialization

- Fork this repository, change `repository name`, usually it should be `<your lowercase username>.github.io` ([official documents](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages))
- Modify the repository settings:
    - `Settings` - `Actions` - `General` - `Artifact and log retention` set to a minimum value of `1` day
    - `Settings` - `Secrets` - `Actions`, click `New repository secret`, add:
        - `REMOTE_FILE_URL`: set to the file download URL, e.g. `https://example.com/path/to/files.tar.gz.enc`
        - `REMOTE_FILE_PASSWORD`: set to the file encryption and decryption password
    - `Settings` - `Pages`, change `Source` to `GitHub Actions`
- The first time you enter `Actions`, you will get a warning `I understand my workflows, go ahead and enable them`, confirm the warning

### Deployment

Assuming the static website files are located in `/path/to/static/dir` directory, package and compress them as `./files.tar.gz` and then encrypt them as `./files.tar.gz.enc`. Command example:

```bash
tar --owner 0 --group 0 --numeric-owner -czvf files.tar.gz -C /path/to/static/dir .

# Enter your password after this command is started
openssl enc -aes-256-cbc -pbkdf2 -pass stdin -in files.tar.gz -out files.tar.gz.enc
```

- Upload `./files.tar.gz.enc` to your server
- `Actions` - `Deploy to GitHub Pages` - `Run workflow`, click `Run workflow`, and wait for it to finish. In the workflow run, logs and artifacts will still contain private data and can be viewed publicly. After it is finished:
    - If it runs successfully, the workflow run is automatically deleted, no further operation required
    - If the run fails, you need to manually delete all workflow runs after troubleshooting
- Delete the `files.tar.gz.enc file` from your server
