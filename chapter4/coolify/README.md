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

## Coolify
Often, domain names for common words, such as `chat`, are already taken, and a common solution is to play around with the vowels in the words. For example, we might remove `a` and make it `cht` (which is actually less likely to be available) or add a to produce `chaat`. While this clearly has no actual effect on coolness, it has become a popular, albeit slightly dated, way to secure domain names that still sound like the original word.

Our third program, Coolify, will allow us to play with the vowels of words that come in via the input and write modified versions to the output.

Create a new folder called `coolify` alongside `sprinkle` and `domainify`, and create the `main.go` code file with the following code:
```go
package main

const (
	duplicateVowel bool = true
	removeVowel    bool = false
)

func randBool() bool {
	return rand.Intn(2) == 0
}

func main() {
	rand.Seed(time.Now().UTC().UnixNano())
	s := bufio.NewScanner(os.Stdin)
	for s.Scan() {
		word := []byte(s.Text())
		if randBool() {
			var vI = -1
			for i, char := range word {
				switch char {
				case 'a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U':
					if randBool() {
						vI = i
					}
				}
			}
			if vI >= 0 {
				switch randBool() {
				case duplicateVowel:
					word = append(word[:vI+1], word[vI:]...)
				case removeVowel:
					word = append(word[:vI], word[vI+1:]...)
				}
			}
		}
		fmt.Println(string(word))
	}
}
```
While the preceding Coolify code looks very similar to the code of Sprinkle and Domainify, it is slightly more complicated. At the very top of the code, we declare two constants, `duplicateVowel` and `removeVowel`, that help make the Coolify code more readable. The `switch` statement decides whether we duplicate or remove a vowel. Also, using these constants, we are able to express our intent very clearly, rather than use just `true` or `false`.

We then define the `randBool` helper function that just randomly returns either `true` or `false`. This is done by asking the `rand` package to generate a random number and confirming whether that number comes out as zero. It will be either `0` or `1`, so there's a fifty-fifty chance of it being `true`.