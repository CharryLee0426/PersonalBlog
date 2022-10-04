# Install Golang from Source Code

## 0. Overview
This article gives a step-by-step guide to install go from source code. My device is **M1 Mac**. So this article is very helpful for those who use Mac on Apple Silicon.

In this article, I assume that we haven't installed go from installer or package manager before. Although most users firstly install golang from package manager.

## 1. Get the source code
Golang, or Go, is an open source programming language which is created by Google team. We can get the source code very easily from Github or Golang official website.

If we use linux, macOS or windows running on amd64 (x86-64), we can choose any version to get the source code. If we use mac on Apple Silicon, whose architecture is arm64, we have to get the source code version 1.16 or later. From Go version 1.16, Golang supports darwin/arm64. If we want to build an earlier version, we have to install two Go versions. One is what you want, the other is the version which has arm64 support.

## 2. Build the Go Bootstrap
Before Go 1.5, the toolchain was written in C. After Go 1.4, the Go team decided to use go to rewrite the toolchain. Therefore, we must build the bootstrap before building the Go programming language. Thankfully, Go supports cross-compiling. We can use our devices to build bootstrap which is fit for other systems and architectures.

We can go to the src folder to find the `bootstrap.bash` (`bootstrap.bat` for Windows) and execute it. If we want to build the bootstrap that is fit to our OS and architecture, we can run this command:

```go
GOOS=linux GOARCH=amd64 ./bootstrap.bash
```

If we want to build an earlier version Go but our devices are Mac on Apple Silicon, we must use Go 1.16 or later to build a darwin/amd64 bootstrap. We can run this command:

```go
GOOS=darwin GOARCH=amd64 ./bootstrap.bash
```

Then we can get `go-darwin-amd64-bootstrap` and `go-darwin-amd64-bootstrap.tbz` at `../..` (if we are in the `src` folder).

## 3. Compiling the Go
We can compile the Go easily just by executing `make.bash` (`make.bat` for Windows) in the `src` subdirectory. We maybe need some configurations in the `make.bash`.

If we have Go installed from installer or package manager, we don't need to config `make.bash`. The `make.bash` can detect where our Go is and use it to compile the Go. In fact, step 2 is optional in that situation.

If we don't have go installed previously, we need to use our bootstrap to compile the go. We need to set `$GOROOT_BOOTSTRAP` variable to the path of Go bootstrap. For example, we need to do that config below in Go 1.19 (latest version when I wrote this article):

```bash
# ...
# line 158
goroot_bootstrap_set=${GOROOT_BOOTSTRAP+"true"}
if [ -z "$GOROOT_BOOTSTRAP" ]; then
	GOROOT_BOOTSTRAP="$HOME/Documents/go-darwin-arm64-bootstrap"
	for d in sdk/go1.17 go1.17; do
		if [ -d "$HOME/$d" ]; then
			GOROOT_BOOTSTRAP="$HOME/$d"
		fi
	done
fi
export GOROOT_BOOTSTRAP
# line 168
```

Maybe there are some slight differences between `make.bash` in different versions. But we can always find `$GOROOT_BOOTSTRAP` variable. After we finish the compile process, we can see these information in our terminal.

```
Building Go cmd/dist using /Users/lll/Documents/go-darwin-amd64-bootstrap. (go1.19 darwin/amd64)
Building Go toolchain1 using /Users/lll/Documents/go-darwin-amd64-bootstrap.
Building Go bootstrap cmd/go (go_bootstrap) using Go toolchain1.
Building Go toolchain2 using go_bootstrap and Go toolchain1.
Building Go toolchain3 using go_bootstrap and Go toolchain2.
Building packages and commands for darwin/amd64.
---
Installed Go for darwin/amd64 in /Users/lll/Documents/go1.14
Installed commands in /Users/lll/Documents/go1.14/bin
```

We can use `go` command at `$HOME/Documents/go1.14/bin/go`.