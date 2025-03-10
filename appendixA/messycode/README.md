# Steps
* step 1
```go
package main

import ( 
	"encoding/json"
	"log"
	"os"
)

func main() {
	data := struct {
		Message string `json:"message"`
	}{Message: "Hello world"}

	err := json.NewEncoder(os.Stdout).Encode(data)
	if err != nil {
		log.Fatalln(err)
	}
}

```
* step 2
```bash
go mod init example.com/messycode
go mod tidy
```
* step3
```bash
go run .
```
