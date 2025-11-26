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
```

### Raffle winner

[json](https://pkg.go.dev/encoding/json)
[Marshal](https://pkg.go.dev/encoding/json#example-Marshal)

```go
// importData reads the raffle entries from file and creates the entries slice.
func importData() []raffleEntry {
	file, err := os.ReadFile(path)
	if err != nil {
		log.Fatalf("failed reading data from file: %s", err)
	}

	var data []raffleEntry
	err = json.Unmarshal(file, &data)
	if err != nil {
		log.Fatalf("failed unmarshaling data: %s", err)
	}

	return data
}
```

### Calculate change

```go
// calculateChange returns the coins required to calculate the
func calculateChange(amount float64) map[coin]int {
	change := make(map[coin]int)
	for _, c := range coins {
		if amount >= c.value {
			count := math.Floor(amount / c.value)
			amount -= count * c.value
			change[c] = int(count)
		}
	}
	return change
}
```

### The big sale

[sort](https://pkg.go.dev/sort)

```go
// matchSales adds the sales procentage of the item
// and sorts the array accordingly.
func matchSales(budget float64, items []SaleItem) []SaleItem {
	var mi []SaleItem
	for _, item := range items {
		if item.ReducedPrice <= budget {
			item.SalePercentage = ((item.OriginalPrice - item.ReducedPrice) / item.OriginalPrice) * 100
		}
		mi = append(mi, item)
	}

	sort.Slice(mi, func(i, j int) bool {
		return mi[i].SalePercentage > mi[j].SalePercentage
	})

	return mi
}
```

### Biggest market

```go
func getBiggestMarket(users []User) (string, int) {
	cMap := make(map[string]int)

	for _, u := range users {
		cMap[u.Country]++
	}

	var country string
	var max int

	for k, v := range cMap {
		if v > max {
			country = k
			max = v
		}
	}

	return country, max
}
```

### Balanced brackets

```go
package main

import (
	"flag"
	"log"
)

// operatorType represents the type of operator in an expression
type operatorType int

const (
	openBracket operatorType = iota
	closedBracket
	otherOperator
)

// bracketPairs is the map legal bracket pairs
var bracketPairs = map[rune]rune{
	'(': ')',
	'[': ']',
	'{': '}',
}

// getOperatorType return the operator type of the give operator.
func getOperatorType(op rune) operatorType {
	for ob, cb := range bracketPairs {
		switch op {
		case ob:
			return openBracket
		case cb:
			return closedBracket
		}
	}

	return otherOperator
}

// stack is a simple LIFO stack implementation using a slice.
type stack struct {
	elems []rune
}

// push adds a new element to the stack.
func (s *stack) push(e rune) {
	s.elems = append(s.elems, e)
}

// pop removes the last element from the stack.
func (s *stack) pop() *rune {
	if len(s.elems) == 0 {
		return nil
	}
	n := len(s.elems) - 1
	last := s.elems[n]
	s.elems = s.elems[:n]
	return &last
}

// isBalanced returns whether the given expression
// has balanced brackets.
func isBalanced(expr string) bool {
	s := stack{}
	for _, e := range expr {
		switch getOperatorType(e) {
		case openBracket:
			s.push(e)
		case closedBracket:
			last := s.pop()
			if last == nil || bracketPairs[*last] != e {
				return false
			}
		}
	}

	return len(s.elems) == 0
}

// printResult prints whether the expression is balanced.
func printResult(expr string, balanced bool) {
	if balanced {
		log.Printf("%s is balanced.\n", expr)
		return
	}
	log.Printf("%s is not balanced.\n", expr)
}

func main() {
	expr := flag.String("expr", "", "The expression to validate brackets on.")
	flag.Parse()
	printResult(*expr, isBalanced(*expr))
}
```

```sh
go run main.go -expr="(a + b) * [c - d]"
```
