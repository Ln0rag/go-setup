# go-setup

A simple Bash command to install and configure the latest stable Go version on Linux.

## What it does
* Detects the system architecture.
* Downloads the latest stable Go release from the official Go website.
* Removes conflicting `golang-go` packages installed through apt.
* Installs Go under `/usr/local/go`.
* Configures `GOPATH`.
* Adds Go and Go binaries to `PATH`.
* Detects Bash, Zsh, or fallback profile configuration.
* Verifies the installed Go version and environment.

## Supported Architectures
* `amd64`
* `arm64`
* `armv6l`
* `386`

## Requirements
* Linux
* `curl`
* `tar`
* `sudo`
* `apt-get` is optional

## Installation
Run:
```bash
GO_VER="$(curl -fsSL 'https://go.dev/VERSION?m=text' | head -n1)" && \
case "$(uname -m)" in
    x86_64) ARCH=amd64;;
    aarch64|arm64) ARCH=arm64;;
    armv6l|armv7l) ARCH=armv6l;;
    i386|i686) ARCH=386;;
    *) echo "Unsupported arch: $(uname -m)"; false;;
esac && \
curl -fsSL "https://go.dev/dl/${GO_VER}.linux-${ARCH}.tar.gz" -o /tmp/go.tar.gz && \
tar -tzf /tmp/go.tar.gz >/dev/null && \
{
    if command -v apt-get >/dev/null 2>&1; then
        sudo apt-get remove -y golang-go 'golang-*-go' >/dev/null 2>&1 || true
    fi
} && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf /tmp/go.tar.gz && \
rm -f /tmp/go.tar.gz && \
case "$(basename "$SHELL")" in
    bash) CONFIG_FILE="$HOME/.bashrc";;
    zsh) CONFIG_FILE="$HOME/.zshrc";;
    *) CONFIG_FILE="$HOME/.profile";;
esac && \
touch "$CONFIG_FILE" && \
{
    grep -qxF 'export GOPATH=$HOME/go' "$CONFIG_FILE" ||
    echo 'export GOPATH=$HOME/go' >> "$CONFIG_FILE"
} && \
{
    grep -qxF 'export PATH="/usr/local/go/bin:$GOPATH/bin:$PATH"' "$CONFIG_FILE" ||
    echo 'export PATH="/usr/local/go/bin:$GOPATH/bin:$PATH"' >> "$CONFIG_FILE"
} && \
export GOPATH="$HOME/go" && \
export PATH="/usr/local/go/bin:$GOPATH/bin:$PATH" && \
hash -r && \
go version && \
which go && \
go env GOROOT GOPATH GOBIN
```
