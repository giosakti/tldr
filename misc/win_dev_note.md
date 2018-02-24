# Windows Developer Note

## WSL

- WSL root directory:

```
C:\Users\giosakti\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs
```

## Docker

- Docker on windows, add this to `.bashrc`

```
PATH="$HOME/bin:$HOME/.local/bin:$PATH"
PATH="$PATH:/mnt/c/Program\ Files/Docker/Docker/resources/bin"
```

## Samba

- Be aware that Samba has default permission when we are saving files from non-UNIX OS, see [this](https://serverfault.com/questions/562875/samba-default-file-creation-mask-calculation) article
- If we want to set chmod recursively on directory that has chaotic permission, see [this](https://stackoverflow.com/questions/18817744/change-all-files-and-folders-permissions-of-a-directory-to-644-755) article

- Or, we can also follow these instructions:
  - To recursively give directories read&execute privileges:

```
find /path/to/base/dir -type d -exec chmod 755 {} +
```

  - To recursively give files read privileges:

```
find /path/to/base/dir -type f -exec chmod 644 {} +
```
