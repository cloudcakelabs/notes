# System

## Linux

### Files and directories

#### Find large files

```sh
find [path] -type f -exec du -h --time {} + | sort -hr | head -20
```

```sh
find [path] -type f -printf "%s\t%p\n" | sort -rh | head -20
```

```sh
find [path] -type f -exec ls -s {} + | sort -n -r | head -20
```

```sh
# Do not cross filesystems
find [path] -xdev -type f -exec ls -s {} + | sort -n -r | head -20
```

#### Display files older than 60 days

```sh
find [path] -type f -mtime +60 -exec ls -s {} \;
```

#### Delete files older than 60 days

```sh
find [path] -type f -mtime +60 -exec rm -f {} \;
```

#### Find and exec a command

```sh
find [path] [args] -exec [command] {} \;
# example
find [path] -type f -exec cp {} {}.bak \;
```

`--relative` flag creates subdirectories in the target directory

```sh
find [path]/ -type f -name [name] -exec rsync -av --relative {} [path]/ \;
```

#### Sort by size

```sh
ls -lha --sort=size
```

#### Get absolute path of a file

```sh
realpath [filename]
```

#### Sync directories

```sh
rsync -av --delete --progress [src]/ [dst]/
```

#### Redirect stderr and stdout

```sh
# stdout
command 1> /dev/null
# stderr
command 2> /dev/null
# stderr to stdout to a file
command > /dev/null 2>&1
# stderr to stdout to a file
command &> /dev/null
```

#### Get some informations about a program

```sh
file [program]
```

#### Print shared object dependencies

```sh
ldd [program]
```

#### Display permissions and user/group with `tree`

```sh
tree -pugfiaD [directory]
```

#### Prevent file/dir modification and deletion

- View file extended attributes

```sh
lsattr [file]
```

- Set the immuable flag

```sh
chattr +i [file]
```

### Memory, CPU and process management

#### Sort processes by memory usage

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head
```

#### Sort processes by cpu usage

```sh
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

#### Sort processes by start time

```sh
ps -ef --sort=start_time
```

- Reverse the order

```sh
ps -ef --sort=-start_time
```

#### With start time

```sh
ps -eo pid,lstart,cmd
```

#### `ps` using grep and displaying headers

```sh
ps -eo user,pid,ppid,lstart,%mem,%cpu,cmd --sort=start_time | { head -1; grep '\.py[[:blank:]]'; } | grep -v grep
```

```sh
ps -eo user,pid,ppid,lstart,%mem,%cpu,cmd --sort=start_time | sed -n '1p; /[.]py[[:blank:]]/p'
```

```sh
ps -eo user,pid,ppid,lstart,%mem,%cpu,cmd --sort=start_time | awk 'NR == 1 || /[.]py[[:blank:]]/'
```

#### List file descriptors

```sh
lsof -a -p [PID]
```

#### List processes based on name/pattern

```sh
pgrep -fa '[pattern]'
# example
pgrep -fa 'rsync'
```

#### Kill processes based on name/pattern

```sh
pkill -f '[pattern]'
```

#### Show CPU details

```sh
lscpu
```

`top` then, press `1` to display usage per CPU.

```sh
top
```

#### Display a tree of processes

```sh
pstree -laps [pid]
```

#### Keep processes running after exiting the shell

```sh
nohup [command] &
```

- Redirect to a file and to standard error and output

```sh
nohup [command] > output.log 2>&1 &
```

- Different files for standard output and error

```sh
nohup [command] 1> output.log 2> error.log &
```

#### Clear cache/buffer

```sh
sysctl vm.drop_caches=3
```

or

```sh
sync; echo 3 > /proc/sys/vm/drop_caches
```

#### Clear swap

```sh
swapoff -a
swapon -a
```

### Disks

Input/Output(I/O) performance

```sh
dnf install sysstat
```

```sh
iostat -d
```

```sh
iostat -d /dev/sda
```

```sh
iostat -p /dev/sda1
```

#### LVM

- Display file system types

```sh
lsblk -o PATH,FSTYPE,MOUNTPOINT [partition_name]
```

Example

```sh
lsblk -o PATH,FSTYPE,MOUNTPOINT /dev/sda
```

- Extend a filesystem(with `-r|--resizefs`)

```sh
lvextend -L +[size] [filesystem] -r
```

Example

```sh
lvextend -L +2G /dev/mapper/vg_root-lv_home -r
```

- Increase size of an XFS filesystem

```sh
xfs_growfs -d [filesystem]
```

Example

```sh
xfs_growfs -d /dev/mapper/vg_root-lv_home
```

### DNS queries

```sh
dig +noall +answer +multiline [fqdn]
```

### Managing Users

Add user

```sh
useradd -d [home_dir] -s [shell] [username] -G [group1] [group2]
```

Change password

```sh
passwd [username]
```

### SSH

#### Generate SSH key pair

```sh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C "[email or comment]"
```

```sh
ssh-keygen -t rsa -b 4096 -C "[email or comment]"
```

#### Start ssh-agent in the background

```sh
eval "$(ssh-agent -s)"
```

#### Add the SSH key to the ssh-agent

```sh
ssh-add ~/.ssh/id_ed25519
```

#### Disable Strict Host Key Checking

```sh
ssh -o StrictHostKeyChecking=no username@remotehost
```

#### Get SSH version

```sh
echo ~ | nc localhost 22
```

#### Permissions

```sh
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
chmod 644 ~/.ssh/config
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

#### User config file

```sh
Host *
    LogLevel error
Host [name1]
    HostName [ip or fqdn]
    User [username]
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
Host [name2]
    HostName [ip or fqdn]
    User [username]
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
    ProxyCommand ssh -W %h:%p [name1]
```

#### Logs

```sh
tail -f -n 100 /var/log/secure | grep sshd
```

```sh
grep sshd /var/log/secure
```

### SSHFS

#### Install(client side)

```sh
dnf install fuse-sshfs
```

#### Mount a remote FS

```sh
sshfs [user]@[host]:[dir] [mountpoint] [options]
```

Example

```sh
sshfs fedora@192.168.0.10:/data /mnt/data
```

#### Automatically mount the remote FS

```sh
# file: /etc/fstab
fedora@192.168.0.10:/data /mnt/data sshfs
```

### SSL

#### Check a certificate

```sh
openssl x509 -in [srv.crt] -text -noout
```

#### Extract the private key from the PFX file

```sh
openssl pkcs12 -in [file.pfx] -nocerts -out [private.key]
```

#### Extract the certificate from the PFX file

```sh
openssl pkcs12 -in [file.pfx] -clcerts -nokeys -out [certificate.crt]
```

#### Extract the decrypted private key

```sh
openssl rsa -in [private.key] -out [decrypted.key]
```

#### Extract the CA chain from the PFX file

```sh
 openssl pkcs12 -in [file.pfx] -cacerts -nokeys -chain -out [ca.pem]
```

#### Get a website certificate

```sh
echo | openssl s_client -showcerts -servername [example.com] -connect [example.com]:443 2>/dev/null | openssl x509 -inform pem -noout -text
```

### Git

#### Setup git configuration

```sh
git config --global user.name "[name]"
```

```sh
git config --global user.email "[email]"
```

```sh
git config --global http.proxy http://[ip_or_fqdn]:[port]
```

- [Customizing Git](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)

#### Work with Git

#### Stage all changes

```sh
git add -A
```

#### Commit staged changes

```sh
git commit -m [commit_message]
```

- Edit previous commit message

```sh
git commit --amend
```

#### Revert to a previous commit

```sh
git reset --hard HEAD~
```

```sh
git reset --soft HEAD~
```

```sh
git reset --hard [commit_id]
```

#### Discard local changes

```sh
git fetch origin
```

```sh
git reset --hard origin/[branch_name]
```

#### Diff between commits

```sh
git diff [commit_id]..[commit_id]
```

#### Diff between last commit and the working tree

```sh
git diff
```

#### Show informations recorded in the reflogs

```sh
git reflog show
```

#### `~/.gitconfig`

```sh
[credential]
  helper = store --file /path/to/.gitcred
[user]
  name = <name>
  email = <email>
    # GPG config
  signingkey = <signingkey>
[http]
[http "https://gitlab.example.com"]
  proxy = http://<ip or fqdn>:<port>
[core]
    # vscode as a default editor
  editor = code --wait
[commit]
    # GPG config
  gpgsign = true
[alias]
  graph = log --oneline --graph --decorate
  llog = log --graph --name-status --pretty=format:\"%C(red)%h %C(reset)(%cd) %C(green)%an %Creset%s %C(yellow)%d%Creset\" --date=relative
  acp = "!f() { git add -A && git commit -m \"$@\" && git push; }; f"
```

Multiple configs

```sh
[includeIf "gitdir:~/personal/"]
  path = ~/.gitconfig-personal
[includeIf "gitdir:~/work/"]
  path = ~/.gitconfig-work
```

### Vim

#### Find and replace

```sh
# Syntax
:[range]s/{pattern}/{string}/[flags]
# Replace in all lines
:%s/hello/hi/g
# Confirm before replacing
:%s/hello/hi/gc
# Case-insensitive search
:%s/hello/hi/gi
# Replace between lines 1 to 5
:1,5s/hello/hi/g
```

#### Comment

```sh
# Syntax
:[start line],[end line]s/^/#
# Comment line 1 to 5
:1,5s/^/#
```

#### Uncomment

```sh
# Uncomment line 1 to 5
:1,5s/^#//
```

#### Set number

```sh
:set number
```

#### Unset number

```sh
:set number!
```

#### Undo and redo

Undo = Press `u`
Redo = Press `CTRL+R`

#### List the available undo options

```sh
:undolist
```

#### Delete blank lines

```sh
:g/^$/d
```

### systemd

List all running systemd services

```sh
systemctl list-units --type=service --state=running
```

List all enabled systemd services

```sh
systemctl list-unit-files --type=service --state=enabled
```

List all active sockets

```sh
systemctl list-sockets
```

Show the most recent system errors

```sh
journalctl -xe
```

Monitor logs in real-time

```sh
journalctl -f
```

Monitor logs in real-time for a specific service

```sh
journalctl -u [unit] -f
```

 Show log entries since a specific date and time.

```sh
journalctl --since "YYYY-MM-DD HH:MM:SS"
```

Display the paths and directories used by the systemd system

```sh
systemd-path
```

Show the current locale and keyboard settings

```sh
localectl
```

### Grub

Modify `/etc/default/grub`

```sh
vim /etc/default/grub
```

Update `grub.cfg` file

```sh
grub2-mkconfig -o /boot/grub2/grub.cfg
```

### VirtualBox

Error: Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_SVM_IN_USE).

Fix for VirtualBox `7.1.4` and Kernel `6.12`

Update `/etc/default/grub` file: Add `kvm.enable_virt_at_load=0` parameter to `GRUB_CMDLINE_LINUX_DEFAULT`

### Networking

#### NetworkManager

Check if NetworkManager is running

```sh
nmcli general
```

List all connection profiles

```sh
nmcli con show 
```

Check device status

```sh
nmcli device status
```

Get details about connection

```sh
nmcli con show [con_name]
```

Configure network

```sh
nmcli con mod [con_name] connection.autoconnect yes
```

```sh
nmcli con mod [con_name] ipv4.addresses [ip_addr/mask] ipv4.method manual ipv4.gateway [gw] ipv4.dns "[dns1] [dns2]" ipv4.dns-search [domain]
```

```sh
nmcli con up [con_name]
```

Restart NetworkManager

```sh
systemctl restart NetworkManager
```

#### Network configuration with network scripts[(doc)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/deployment_guide/ch-network_interfaces#ch-Network_Interfaces)

```sh
vim /etc/sysconfig/network-scripts/ifcfg-[if_name]
```

Example

```sh
vim /etc/sysconfig/network-scripts/ifcfg-eth0
```

Routing and host information for all network interface: `/etc/sysconfig/network`

```sh
vim /etc/sysconfig/network
```

#### Network connections and statistics

Monitoring all listening TCP connections

```sh
ss -ltnp
```

Monitoring all listening UDP connections

```sh
ss -lunp
```

Monitoring all listening port(tcp/udp) and established connections

```sh
ss -plantus
```

#### Display CLOSE-WAIT socket connections

```sh
ss --tcp state CLOSE-WAIT
```

#### Kill CLOSE-WAIT socket connections

```sh
ss --tcp state CLOSE-WAIT --kill
```

#### Test TCP connections

```sh
nc -zv [ip_dst] [port]
```

```sh
nc -zv -s [ip_src] [ip_dst] [port]
```

#### Test UDP connections

```sh
nc -zv -u [ip_dst] [port]
```

### Debug

#### tcpdump (with root)

```sh
tcpdump -i [interface name] -w [filename].pcap
```

```sh
tcpdump -i [interface name] dst [ip addr or fqdn] -w [filename].pcap
```

```sh
tcpdump -i [interface name] port [port] -w [filename].pcap
```

#### Checksum: md5sum or sha256sum

```sh
find -type f -exec md5sum '{}' \;
```

```sh
find -type f -exec sha256sum '{}' \; > hashes.txt
sha256sum --check hashes.txt
```

```sh
md5sum --check hashes.txt
```

```sh
sha256sum --check hashes.txt
```

```sh
echo "[hash]  [filename]" | sha256sum --check
```

```sh
echo -n 'hi' | sha256sum
```

```sh
sha256sum [filename]
```

#### Hardware

```sh
# Show USB ports details
lsusb
```

#### Trace system calls and signals

Basic usage

```sh
strace [command]
```

Attach to an running process

```sh
strace -p [pid]
```

### Auditd

#### Failed login attempts

```sh
ausearch --message USER_LOGIN --success no --interpret
```

#### Failed system call

```sh
ausearch --start yesterday --end now -m SYSCALL -sv no -i
```

#### AUID system call

```sh
ausearch --start yesterday --end now -ua [auid] -i
```

#### List of login events

```sh
aureport --login -i
```

### $HOME

#### `~/.bashrc`

```sh
shopt -s histappend
PROMPT_COMMAND="history -a;$PROMPT_COMMAND"
HISTCONTROL=ignoreboth
HISTFILESIZE=50000
HISTSIZE=${HISTFILESIZE}
HISTTIMEFORMAT="$(date +"%Y-%m-%dT%H:%M:%S%z") "

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# Setup the ssh-agent
if [ ! -S ~/.ssh/ssh_auth_sock ]; then
  eval `ssh-agent`
  ln -sf "$SSH_AUTH_SOCK" ~/.ssh/ssh_auth_sock
fi
export SSH_AUTH_SOCK=~/.ssh/ssh_auth_sock
ssh-add -l > /dev/null || ssh-add

GPG_TTY=$(tty)
export GPG_TTY

export DATE_ISO8601=$(date +"%Y-%m-%dT%H-%M-%S%z")

[ -f ~/.fzf.bash ] && source ~/.fzf.bash

source <(kubectl completion bash)

# Custom Prompt
# Display exit code if not equal 0
__sh_exitcode() { ret=$?; if [[ $ret != 0 ]]; then echo "$ret "; fi }

if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        PS1='\[\033[1;31m\]$(__sh_exitcode)\[\033[1;32m\](label) \[\033[1;37m\]\u\[\033[0;39m\]@\[\033[1;37m\]\h\[\033[0;39m\]:\[\033[1;34m\]\w\[\033[0;39m\]\$\[\033[0;39m\] '
else
    PS1='$(__sh_exitcode)\u@\h:\w\$ '
fi
```

#### `~/.inputrc`

```sh
set show-all-if-ambiguous off
set colored-completion-prefix on
set colored-stats on

"\e[A": history-search-backward
"\e[B": history-search-forward
"\e[C": forward-char
"\e[D": backward-char
# pgup
"\e[6~": menu-complete-backward
# pgdn
"\e[5~": menu-complete
```

#### `~/.tmux.conf`

```sh
#unbind C-b
#set -g prefix C-a
#bind -n C-a send-prefix

unbind C-c
bind -n C-k killp
bind -n C-Left split-window -h
bind -n C-Right split-window -h
bind -n C-Down split-window -v
bind -n C-Up split-window -v

bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

bind -n C-x setw synchronize-panes

set -g visual-activity off
set -g visual-bell off
set -g visual-silence off
set -g monitor-activity off
set -g bell-action none
set -g mouse on

bind z source-file ~/.tmux.conf
```

#### `~/.bash_aliases`

```sh
# https://opensource.com/article/19/7/bash-aliases
# Sort by modification time
alias ll="ls -lthrA --color=auto"
# Sort by file size
alias lt='ls --human-readable --size -1 -S --classify'
# View only mounted drives
alias mnt="mount | awk -F' ' '{ printf \"%s\t%s\n\",\$1,\$3; }' | column -t | egrep ^/dev/ | sort"
# Count files
alias count='find . -type f | wc -l'
# Add a copy progress bar
alias cpv='rsync -ah --info=progress2' 
# Find a command in your grep history
# Example: gh <search_something>
alias gh='history | grep'
```

### Date and time

#### Show current time settings

```sh
timedatectl status
```

#### Set timezone

```sh
timedatectl set-timezone UTC
```

### RabbitMQ

- Check RabbitMQ cluster status

```sh
rabbitmqctl cluster_status
```

```sh
rabbitmq-diagnostics check_running
```

- List queues

```sh
rabbitmqctl list_queues
```

- List consumers

```sh
rabbitmqctl list_consumers
```

### Tools

- Extract full paths of json keys

```sh
jq -r '[paths | join(".")]'  [json_file]
```

  * Example

```sh
k get deploy [deployment_name] -o json | jq -r '[paths | join(".")]'
```

- Base64 Encoding/Decoding

Encoding

```sh
echo -n 'string' | base64
```

```sh
base64 [filename]
```

Decoding

```sh
echo -n 'string' | base64 --decode
```

```sh
base64 --decode [filename]
```

- Convert Yaml file to Json file

```sh
yq -o=json [yaml_file] > [json_file]
```

- Diff between two directories

```sh
diff -r [dir_a] [dir_b]
```

- Remove comments and blank lines

```sh
grep -Ev '^$|#' [filename]
```

### Shell scripting

- Read a file line by line

```sh
FILE_PATH=<path_file>

while IFS= read -r line; do
    echo "${line}"
done < "${FILE_PATH}"
```

- List all executables

```sh
IFS=:;
set -f;
find -L $PATH -maxdepth 1 -type f -perm -100 -print;
```

## RedHat-based Linux

### Admin

#### List installed packages

```sh
dnf list installed
```

```sh
dnf list installed | grep [package_name]
```

```sh
rpm -qa | grep [package_name]
```

#### List all available versions of a package

```sh
dnf list [package_name] --showduplicates
```

#### Update a package

```sh
dnf update [package_name]
```

#### Download RPM package file

```sh
dnf download [package_name]
```

- Resolve and download needed dependencies

```sh
dnf download --resolve [package_name]
```

#### Downgrade a package

```sh
dnf downgrade [package_name]-[version]
```

#### `versionlock`: Protect packages from being updated

```sh
dnf install python3-dnf-plugin-versionlock -y
```

```sh
dnf versionlock [package_name]-[version]
```

```sh
dnf versionlock list
```

#### Finds the packages providing the given file

```sh
dnf provides [filename]
```

#### Exclude package from getting updated

```sh
dnf update --exclude=PACKAGENAME 
```

Example

```sh
dnf update --exclude=kernel*
```

#### Clean all cached files(packages included)

```sh
rm -rf /var/cache/yum/*
```

or

```sh
rm -rf /var/cache/dnf/*
```

then

```sh
dnf clean all
```

#### Remove old kernels

```sh
dnf remove --oldinstallonly --setopt installonly_limit=2 kernel
```

#### Disable/Enable Repo

Show all repos

```sh
dnf repolist --all
```

Disable a repo

```sh
dnf config-manager --disable [repo_name]
```

Show disabled repos

```sh
dnf repolist --disabled
```

Enable a repo

```sh
dnf config-manager --enable [repo_name]
```

Show enabled repos

```sh
dnf repolist --enabled
```

or

```sh
dnf repolist
```

#### View transaction history

```sh
dnf history
```

### Security and bugfixes updates

#### Check security and/or bugfixes updates

```sh
dnf check-update --security
```

```sh
dnf check-update --bugfix
```

```sh
dnf check-update --security --bugfix
```

[check-update command doc](https://dnf.readthedocs.io/en/latest/command_ref.html#check-update-command)

#### Display information about update advisories

```sh
dnf updateinfo list --security
```

```sh
dnf updateinfo list --bugfix
```

```sh
dnf updateinfo list --security --bugfix
```

[updateinfo command doc](https://dnf.readthedocs.io/en/latest/command_ref.html#updateinfo-command)

#### Install security and/or bugfixes updates

```sh
dnf update --security
```

```sh
dnf update --bugfix
```

```sh
dnf update --security --bugfix
```

[upgrade command info](https://dnf.readthedocs.io/en/latest/command_ref.html#upgrade-command)

#### Display information about CVE

```sh
dnf updateinfo list --cve=[cve_id]
```

#### Display information about RHSA ID

```sh
dnf updateinfo info [advisory_id]
```

#### Install specific update

```sh
dnf update --advisory=[advisory_id]
```

Example

```sh
dnf update --advisory=RHSA-2023:4102
```

## Windows

### Powershell

- Test a connection

```powershell
Test-NetConnection [ip_addr or fqdn] -port [port]
```

- Network stats

```powershell
netstat
```

- Get the basic network adapter properties

```powershell
Get-NetAdapter
```

- Get routing table

```powershell
Get-NetRoute
```

## Shortcuts

|          |         |
| -------- | ------- |
| `ctrl + alt + F2`  | x11: Get a virtual terminal |
| `ctrl + alt + F9`  | x11: Back to X |
