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
go mod init synonyms
go mod tidy
```
## Run 
```bash
go run .
```
## Build
```bash
go build
go build -o synonyms
```
## Clean
```bash
 go clean -i -n
```

## Synonyms
So far, our programs have only modified words, but to really bring our solution to life, we need to be able to integrate a third-party API that provides word synonyms. This allows us to suggest different domain names while retaining the original meaning. Unlike Sprinkle and Domainify, Synonyms will write out more than one response for each word given to it. Our architecture of piping programs together means this won't be much of a problem; in fact, we do not even have to worry about it since each of the three programs is capable of reading multiple lines from the input source.
### Using environment variables for configuration
Your API key is a sensitive piece of configuration information that you don't want to share with others. We could store it as `const` in our code. However, this would mean we will not be able to share our code without sharing our key (not good, especially if you love open source projects). Additionally, perhaps more importantly, you will have to recompile your entire project if the key expires or if you want to use a different one (you don't want to get into such a situation).

A better solution is using an environment variable to store the key, as this will allow you to easily change it if you need to. You could also have different keys for different deployments; perhaps you could have one key for development or testing and another for production. This way, you can set a specific key for a particular execution of code so you can easily switch between keys without having to change your system-level settings. Also, different operating systems deal with environment variables in similar ways, so they are a perfect choice if you are writing cross-platform code.

Create a new environment variable called `BHT_APIKEY` and set your API key as its value.

> ### Note
> For machines running a bash shell, you can modify your ~/.bashrc file or similar to include export commands, such as the following:
> ```bash
> export BHT_APIKEY=abc123def456ghi789jkl
> ```
> On Windows machines, you can navigate to the properties of your computer and look for `Environment Variables` in the `Advanced` section.
### Consuming a web API
Making a request for in a web browser shows us what the structure of JSON response data looks like when finding synonyms for the word love:
```json
{ 
  "noun":{ 
    "syn":[ 
      "passion", 
      "beloved", 
      "dear" 
    ] 
  }, 
  "verb":{ 
    "syn":[ 
      "love", 
      "roll in the hay", 
      "make out" 
    ], 
    "ant":[ 
      "hate" 
    ] 
  } 
} 
```
A real API will return a lot more actual words than what is printed here, but the structure is the important thing. It represents an object, where the keys describe the types of word (verbs, nouns, and so on). Also, values are objects that contain arrays of strings keyed on `syn` or `ant` (for the synonym and antonym, respectively); it is the synonyms we are interested in.

To turn this JSON string data into something we can use in our code, we must decode it into structures of our own using the capabilities found in the `encoding/json` package. Because we're writing something that could be useful outside the scope of our project, we will consume the API in a reusable package rather than directly in our program code. Create a new folder called `thesaurus` alongside your other program folders (in `$GOPATH/src`) and insert the following code into a new `bighuge.go` file:
```go
package thesaurus 
import ( 
  "encoding/json" 
  "errors" 
  "net/http" 
) 
type BigHuge struct { 
  APIKey string 
} 
type synonyms struct { 
  Noun *words `json:"noun"` 
  Verb *words `json:"verb"` 
} 
type words struct { 
  Syn []string `json:"syn"` 
} 
func (b *BigHuge) Synonyms(term string) ([]string, error) { 
  var syns []string 
  response, err := http.Get("http://words.bighugelabs.com/api/2/"  + 
   b.APIKey + "/" + term + "/json") 
  if err != nil { 
    return syns, errors.New("bighuge: Failed when looking for  synonyms    
     for "" + term + """ + err.Error()) 
  } 
  var data synonyms 
  defer response.Body.Close() 
  if err := json.NewDecoder(response.Body).Decode(&data); err !=  nil { 
    return syns, err 
  } 
  if data.Noun != nil { 
    syns = append(syns, data.Noun.Syn...) 
  } 
  if data.Verb != nil { 
    syns = append(syns, data.Verb.Syn...) 
  } 
  return syns, nil 
} 
```
In the preceding code, the `BigHuge` type we define houses the necessary API key and provides the `Synonyms` method that will be responsible for doing the work of accessing the endpoint, parsing the response, and returning the results. The most interesting parts of this code are the `synonyms` and `words` structures. They describe the JSON response format in Go terms, namely an object containing noun and verb objects, which in turn contain a slice of strings in a variable called `Syn`. The tags (strings in backticks following each field definition) tell the `encoding/json` package which fields to map to which variables; this is required since we have given them different names.

> ### Tip
> Typically in JSON, keys have lowercase names, but we have to use capitalized names in our structures so that the `encoding/json` package would also know that the fields exist. If we don't, the package would simply ignore the fields. However, the types themselves (`synonyms` and `words`) do not need to be exported.

The `Synonyms` method takes a `term` argument and uses `http.Get` to make a web request to the API endpoint in which the URL contains not only the API key value, but also the `term` value itself. If the web request fails for some reason, we will make a call to `log.Fatalln`, which will write the error to the standard error stream and exit the program with a non-zero exit code (actually an exit code of `1`). This indicates that an error has occurred.

If the web request is successful, we pass the response body (another `io.Reader`) to the `json.NewDecoder` method and ask it to decode the bytes into the data variable that is of our `synonyms` type. We defer the closing of the response body in order to keep the memory clean before using Go's built-in `append` function to concatenate both `noun` and `verb` synonyms to the `syns` slice that we then return.

Although we have implemented the `BigHuge` thesaurus, it isn't the only option out there, and we can express this by adding a `Thesaurus` interface to our package. In the `thesaurus` folder, create a new file called `thesaurus.go` and add the following interface definition to the file:
```go
package thesaurus 
type Thesaurus interface { 
  Synonyms(term string) ([]string, error) 
} 
```
This simple interface just describes a method that takes a `term` string and returns either a slice of strings containing the synonyms or an error (if something goes wrong). Our `BigHuge` structure already implements this interface, but now, other users could add interchangeable implementations for other services, such as http://www.dictionary.com/ or the Merriam-Webster online service.

Next, we are going to use this new package in a program. Change the directory in the terminal back up a level to `$GOPATH/src`, create a new folder called `synonyms`, and insert the following code into a new `main.go` file you will place in this folder:
```go
func main() { 
  apiKey := os.Getenv("BHT_APIKEY") 
  thesaurus := &thesaurus.BigHuge{APIKey: apiKey} 
  s := bufio.NewScanner(os.Stdin) 
  for s.Scan() { 
    word := s.Text() 
    syns, err := thesaurus.Synonyms(word) 
    if err != nil { 
      log.Fatalln("Failed when looking for synonyms for  "+word+", err) 
    } 
    if len(syns) == 0 { 
      log.Fatalln("Couldn't find any synonyms for " + word +  ") 
    } 
    for _, syn := range syns { 
      fmt.Println(syn) 
    } 
  } 
} 
```
Now when you manage your imports again, you will have written a complete program that is capable of looking up synonyms of words by integrating the Big Huge Thesaurus API.

In the preceding code, the first thing our `main` function does is that it gets the `BHT_APIKEY` environment variable value via the `os.Getenv` call. To protect your code, you might consider double-checking it to ensure the value is properly set; if not, report the error. For now, we will assume that everything is configured properly.

Next, the preceding code starts to look a little familiar since it scans each line of input again from `os.Stdin` and calls the `Synonyms` method to get a list of the replacement words.

Let's build a program and see what kind of synonyms the API comes back with when we input the word `chat`:
```bash
go build -o synonyms
./synonyms
chat
confab
confabulation
schmooze
New World chat
Old World chat
conversation
thrush
wood warbler
chew the fat
shoot the breeze
chitchat
chatter
```
The results you get will most likely differ from what we have listed here since we're hitting a live API. However, the important thing is that when we provide a word or term as an input to the program, it returns a list of synonyms as the output, one per line.

### Getting domain suggestions
By composing the four programs we have built so far in this chapter, we already have a useful tool for suggesting domain names. All we have to do now is to run the programs while piping the output to the input in an appropriate way. In a terminal, navigate to the parent folder and run the following single line:
```bash
./synonyms/synonyms | ./sprinkle/sprinkle | ./coolify/coolify |  ./domainify/domainify
```
Because the `synonyms` program is first in our list, it will receive the input from the terminal (whatever the user decides to type in). Similarly, because `domainify` is last in the chain, it will print its output to the terminal for the user to see. Along the way, the lines of words will be piped through other programs, giving each of them a chance to do their magic.

Type in a few words to see some domain suggestions; for example, when you type `chat` and hit return, you may see the following:
```bash
getcnfab.com
confabulationtim.com
getschmoozee.net
schmosee.com
neew-world-chatsite.net
oold-world-chatsite.com
conversatin.net
new-world-warblersit.com
gothrush.net
lets-wood-wrbler.com
chw-the-fat.com
```
The number of suggestions you get will actually depend on the number of synonyms. This is because it is the only program that generates more lines of output than what we input.

We still haven't solved our biggest problem: the fact that we have no idea whether the suggested domain names are actually available or not. So we still have to sit and type each one of them into a website. In the next section, we will address this issue.