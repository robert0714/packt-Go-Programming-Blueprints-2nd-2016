# Chapter 1.  Chat Application with Web Sockets - chat part
## Inspect Your environment
* In windows:
```bash
go env GOPATH
go env GOROOT
```
* Adjsut the errors:
```bash
setx GOPATH "E:\go_workspaces"
```
## Initialization 
```bash
go mod init chat
go mod tidy
```
## Run 
```bash
go run .
```
## Build
```bash
go build
go build -o chat
```
## Clean
```bash
 go clean -i -n
```
