# Chapter 4-1  Command-Line Tools to Find Domain Names
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
go mod init domainify
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

## Domainify
Some of the words that output from Sprinkle contain spaces and perhaps other characters that are not allowed in domains. So we are going to write a program called Domainify; it converts a line of text into an acceptable domain segment and adds an appropriate `Top-level Domain (TLD)` to the end. Alongside the `sprinkle` folder, create a new one called `domainify` and add the `main.go` file with the following code:

```go
package main

var tlds = []string{"com", "net"}

const allowedChars = "abcdefghijklmnopqrstuvwxyz0123456789_-"

func main() {
	rand.Seed(time.Now().UTC().UnixNano())
	s := bufio.NewScanner(os.Stdin)
	for s.Scan() {
		text := strings.ToLower(s.Text())
		var newText []rune
		for _, r := range text {
			if unicode.IsSpace(r) {
				r = '-'
			}
			if !strings.ContainsRune(allowedChars, r) {
				continue
			}
			newText = append(newText, r)
		}
		fmt.Println(string(newText) + "." +
			tlds[rand.Intn(len(tlds))])
	}
}
```
Type in some of these options to see how domainify reacts:
* Monkey
* Hello Domainify
* "What's up?"
* One (two) three!
You can see that, for example, `One (two) three!` might yield `one-two-three.com` .