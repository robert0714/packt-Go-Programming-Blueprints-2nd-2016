# Chapter 2.  Adding User Accounts - Chat Application with Web Sockets - chat part
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
Build and run the chat application and try to hit 
* http://localhost:8080/chat
* http://localhost:8080/login
```bash
go build
go build -o chat
./chat -host=":8080"
```
## Clean
```bash
 go clean -i -n
```
