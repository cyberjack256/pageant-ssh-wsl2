# pageant-ssh-wsl2
Set up WSL2 on new Windows host and create a ssh socket with pageant

# wsl2-ssh-pageant

## Motivation
I use a CAC/PIV to SSH to server. On Windows I wanted to exposes a Pageant style SSH agent and I wanted a way to use this key within WSL2.

## Installing WSL2

David Bombal is a professional and a champion. Big thanks for his simplistic yet sophisticated instructing style. Follow along with his video:

[WSL 2: Getting started Jun 2020](https://www.youtube.com/watch?v=_fntjriRe48)

Use my PowerShell below to achieve the same results. I did not use the `wsl --install` feature, as the below method works on older versions as well as 2022

```PowerShell
# Enable WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
# Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
# Visit the following link and update the kernel https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
```
Download and install Ubuntu 2204LTS for WSL
```
curl.exe -L -o ubuntu-2204.zip https://aka.ms/wslubuntu2204
Expand-Archive ubuntu-2204.zip ubuntu
$userenv = [System.Environment]::GetEnvironmentVariable("Path", "User")
[System.Environment]::SetEnvironmentVariable("PATH", $userenv + (get-location) + "\ubuntu", "User")
.\ubuntu\ubuntu2204.exe
```

```PowerShell
# Set the default WSL version
wsl --set-default-version 2
Restart-Computer
```


## How to use with WSL2

### Prerequisite
In order to use `wsl-ssh-pageant` you must have installed `socat` and `ss` on your machine.

For example, on Ubuntu2204 you can install these by running: `sudo apt install socat iproute2`

### Installation
1. Download latest version from [release page](https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/latest) and copy `wsl2-ssh-pageant.exe` to your windows home directory (or other location within the windows file system). Then simlink to your `$HOME/.ssh` directory for easy access

```bash
windows_destination="/mnt/c/Users/Public/Downloads/wsl2-ssh-pageant.exe"
linux_destination="$HOME/.ssh/wsl2-ssh-pageant.exe"
wget -O "$windows_destination" "https://github.com/BlackReloaded/wsl2-ssh-pageant/releases/latest/download/wsl2-ssh-pageant.exe"
# Set the executable bit.
chmod +x "$windows_destination"
# ensure .ssh directory exists if not create it
if [ -d ~/.ssh ] 
then
    echo "The directory exists"
else
   mkdir ~/.ssh
fi
# Symlink to linux for ease of use later
ln -s $windows_destination $linux_destination
```

2. Add the following to your shell configuration (for e.g. `.bashrc` or `.zshrc`). For advanced configurations consult the documentation of your shell.

#### Bash/Zsh

*SSH:*
```bash
export SSH_AUTH_SOCK="$HOME/.ssh/agent.sock"
if ! ss -a | grep -q "$SSH_AUTH_SOCK"; then
  rm -f "$SSH_AUTH_SOCK"
  wsl2_ssh_pageant_bin="$HOME/.ssh/wsl2-ssh-pageant.exe"
  if test -x "$wsl2_ssh_pageant_bin"; then
    (setsid nohup socat UNIX-LISTEN:"$SSH_AUTH_SOCK,fork" EXEC:"$wsl2_ssh_pageant_bin" >/dev/null 2>&1 &)
  else
    echo >&2 "WARNING: $wsl2_ssh_pageant_bin is not executable."
  fi
  unset wsl2_ssh_pageant_bin
fi
```

## Troubleshooting

### Smartcard is detected in Windows and WSL, but ssh-add -L returns error

## Credit

Some of the code is copied from BlackReloadeds [wsl2-ssh-pageant](https://github.com/BlackReloaded/wsl2-ssh-pageant). Awesome work!
