# Level up Go

## Level up with Go

### Sleep until your birthday

Count how many night to a given date

[time](https://pkg.go.dev/time)

```go
package main

import (
	"flag"
	"log"
	"time"
)

var expectedFormat = "2006-01-02"

// parseTime validates and parses a given date string.
func parseTime(target string) time.Time {
	bday, err := time.Parse(expectedFormat, target)
	if err != nil || time.Now().After(bday) {
		log.Fatal("invalid date")
	}
	return bday
}

// calcSleeps returns the number of sleeps until the target.
func calcSleeps(target time.Time) float64 {
	return time.Until(target).Hours() / 24
}

func main() {
	bday := flag.String("bday", "", "Your next bday in YYYY-MM-DD format")
	flag.Parse()
	target := parseTime(*bday)
	log.Printf("You have %d sleeps until your birthday. Hurray!",
		int(calcSleeps(target)))
}
```

```sh
go run main.go -bday 2026-03-28
```

### Slow down

ex: Go time! -> Goo tiimmmeeee!!!!!

[strings](https://pkg.go.dev/strings)

```go
package main

import (
	"log"
	"strings"
	"time"
)

const delay = 700 * time.Millisecond

// print outputs a message and then sleeps for a pre-determined amount
func print(msg string) {
	log.Println(msg)
	time.Sleep(delay)
}

// slowDown takes the given string and repeats its characters
// according to their index in the string.
func slowDown(msg string) {

	words := strings.SplitSeq(msg, " ")
	for w := range words {

		var pw []string
		for idx, c := range w {
			rb := strings.Repeat(string(c), idx+1)
			pw = append(pw, rb)
		}
		print(strings.Join(pw, ""))
	}

}

func main() {
	msg := "Time to learn about Go strings!"
	slowDown(msg)
}
```
