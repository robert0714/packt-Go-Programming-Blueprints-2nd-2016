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
go mod init sprinkle
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

## Sprinkle
Our first program augments the incoming words with some sugar terms in order to improve the odds of finding names that are available. Many companies use this approach to keep the core messaging consistent while being able to afford the `.com` domain. For example, if we pass in the word `chat`, it might pass out `chatapp`; alternatively, if we pass in `talk`, we may get back `talk time`.

Go's `math/rand` package allows us to break away from the predictability of computers. It gives our program the appearance of intelligence by introducing elements of chance into its decision making.

To make our Sprinkle program work, we will:
* Define an array of transformations, using a special constant to indicate where the original word will appear
* Use the `bufio` package to scan the input from `stdin` and `fmt.Println` in order to write the output to `stdout`
* Use the `math/rand` package to randomly select a transformation to apply