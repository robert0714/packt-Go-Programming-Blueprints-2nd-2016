# Chapter 1.  Chat Application with Web Sockets - trace part
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
go mod init trace
go mod tidy
```
## Run 
```bash
go test
go test -cover
```

## Clean
```bash
 go clean -i -n
```
