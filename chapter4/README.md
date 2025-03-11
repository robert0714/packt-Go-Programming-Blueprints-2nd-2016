# Chapter 4.  Command-Line Tools to Find Domain Names
## Five simple programs
In this chapter, we will build five small programs that we will combine at the end. The key features of the programs are as follows:

* Sprinkle: This program will add some web-friendly sprinkle words to increase the chances of finding the available domain names.
* Domainify: This program will ensure words are acceptable for a domain name by removing unacceptable characters. Once this is done, it will replace spaces with hyphens and add an appropriate top-level domain (such as .com and .net) to the end.
* Coolify: This program will change a boring old normal word to Web 2.0 by fiddling around with vowels.
* Synonyms: This pro will use a third-party API to find synonyms.
* Available: This gram will use a third-party API to find synonyms. Available: This program will check to see whether the domain is available or not using an appropriate WHOIS server.

Five programs might seem like a lot for one chapter, but don't forget how small entire programs can be in Go.
### Sprinkle
Our first program augments the incoming words with some sugar terms in order to improve the odds of finding names that are available. Many companies use this approach to keep the core messaging consistent while being able to afford the `.com` domain. For example, if we pass in the word `chat`, it might pass out `chatapp`; alternatively, if we pass in `talk`, we may get back `talk time`.

Go's `math/rand` package allows us to break away from the predictability of computers. It gives our program the appearance of intelligence by introducing elements of chance into its decision making.

To make our Sprinkle program work, we will:
* Define an array of transformations, using a special constant to indicate where the original word will appear
* Use the `bufio` package to scan the input from `stdin` and `fmt.Println` in order to write the output to `stdout`
* Use the `math/rand` package to randomly select a transformation to apply