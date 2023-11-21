# Use nerdctl to manage dockerd on your kubernetes master node

## Download Desired nerdctl from github
```
wget https://github.com/containerd/nerdctl/releases/download/v1.7.0/nerdctl-1.7.0-linux-amd64.tar.gz
```
```
wget https://github.com/containerd/nerdctl/releases/download/v1.7.0/nerdctl-full-1.7.0-linux-amd64.tar.gz
```
## Minimal Installation

### Extract the archive to a path like /usr/local/bin or ~/bin .
```
tar Cxzvvf /usr/local/bin nerdctl-1.7.0-linux-amd64.tar.gz
```
## Full Installation

### Extract the archive to a path like /usr/local or ~/.local .
```
tar Cxzvvf /usr/local nerdctl-full-1.7.0-linux-amd64.tar.gz
```
## alias the ./nerdctl to be more easy to use
```
nano .bashrc
```
```
alias nerdctl='/usr/local/nerdctl'
```
## apply bashrc
```
source ~/.bashrc
```