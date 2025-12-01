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

### Spread gossip

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"os"
	"time"
)

const path = "friends.json"

// Friend represents a friend and their connections.
type Friend struct {
	ID      string   `json:"id"`
	Name    string   `json:"name"`
	Friends []string `json:"friends"`
}

// hearGossip indicates that the friend has heard the gossip.
func (f *Friend) hearGossip() {
	log.Printf("%s has heard the gossip!\n", f.Name)
}

// Friends represents the map of friends and connections
type Friends struct {
	fmap map[string]Friend
}

// getFriend fetches the friend given an id.
func (f *Friends) getFriend(id string) Friend {
	return f.fmap[id]
}

// getRandomFriend returns an random friend.
func (f *Friends) getRandomFriend() Friend {
	rand.Seed(time.Now().Unix())
	id := (rand.Intn(len(f.fmap)-1) + 1) * 100
	return f.getFriend(fmt.Sprint(id))
}

// spreadGossip ensures that all the friends in the map have heard the news
func spreadGossip(root Friend, friends Friends,
	visited map[string]struct{}) {
	for _, id := range root.Friends {
		if _, isVisited := visited[id]; !isVisited {
			f := friends.getFriend(id)
			f.hearGossip()
			visited[id] = struct{}{}
			spreadGossip(f, friends, visited)
		}
	}
}

func main() {
	friends := importData()
	root := friends.getRandomFriend()
	root.hearGossip()
	visited := make(map[string]struct{})
	visited[root.ID] = struct{}{}
	spreadGossip(root, friends, visited)
}

// importData reads the input data from file and creates the friends map.
func importData() Friends {
	file, err := os.ReadFile(path)
	if err != nil {
		log.Fatal(err)
	}

	var data []Friend
	err = json.Unmarshal(file, &data)
	if err != nil {
		log.Fatal(err)
	}

	fm := make(map[string]Friend, len(data))
	for _, d := range data {
		fm[d.ID] = d
	}

	return Friends{
		fmap: fm,
	}
}
```

### Playist

[container](https://pkg.go.dev/container)

```go
package main

import (
	"container/heap"
	"encoding/json"
	"fmt"
	"log"
	"os"
	"text/tabwriter"
)

const path = "songs.json"

// Song stores all the song related information
type Song struct {
	Name      string `json:"name"`
	Album     string `json:"album"`
	PlayCount int64  `json:"play_count"`

	AlbumCount, SongCount int
}

// An PlaylistHeap is a max-heap of PlaylistEntries.
type PlaylistHeap []Song

func (h PlaylistHeap) Len() int {
	return len(h)
}
func (h PlaylistHeap) Less(i, j int) bool {
	// We want Pop to return the highest play count.
	return h[i].PlayCount > h[j].PlayCount
}
func (h PlaylistHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h *PlaylistHeap) Push(x any) {
	*h = append(*h, x.(Song))
}

func (h *PlaylistHeap) Pop() any {
	original := *h
	n := len(original)
	x := original[n-1]
	*h = original[0 : n-1]
	return x
}

// makePlaylist makes the merged sorted list of songs
func makePlaylist(albums [][]Song) []Song {
	var playlist []Song
	pHeap := &PlaylistHeap{}
	if len(albums) == 0 {
		return playlist
	}

	// initialize the heap and add first of each album, since they are the max
	heap.Init(pHeap)
	for i, f := range albums {
		firstSong := f[0]
		firstSong.AlbumCount, firstSong.SongCount = i, 0
		heap.Push(pHeap, firstSong)
	}

	for pHeap.Len() != 0 {
		// take max elem from the list
		p := heap.Pop(pHeap)
		song := p.(Song)
		playlist = append(playlist, song)
		// the next song after the max is a good candidate to look at
		if song.SongCount < len(albums[song.AlbumCount])-1 {
			nextSong := albums[song.AlbumCount][song.SongCount+1]
			nextSong.AlbumCount, nextSong.SongCount =
				song.AlbumCount, song.SongCount+1
			heap.Push(pHeap, nextSong)
		}
	}

	return playlist
}

func main() {
	albums := importData()
	printTable(makePlaylist(albums))
}

// printTable prints merged playlist as a table
func printTable(songs []Song) {
	w := tabwriter.NewWriter(os.Stdout, 3, 3, 3, ' ', tabwriter.TabIndent)
	fmt.Fprintln(w, "####\tSong\tAlbum\tPlay count")
	for i, s := range songs {
		fmt.Fprintf(w, "[%d]:\t%s\t%s\t%d\n", i+1, s.Name, s.Album, s.PlayCount)
	}
	w.Flush()

}

// importData reads the input data from file and creates the friends map
func importData() [][]Song {
	file, err := os.ReadFile(path)
	if err != nil {
		log.Fatal(err)
	}

	var data [][]Song
	err = json.Unmarshal(file, &data)
	if err != nil {
		log.Fatal(err)
	}

	return data
}
```

### Calcultor edge cases

```go
package main

import (
	"flag"
	"fmt"
	"log"
	"strconv"
	"strings"
)

// operators is the map of legal operators and their functions
var operators = map[string]func(x, y float64) float64{
	"+": func(x, y float64) float64 { return x + y },
	"-": func(x, y float64) float64 { return x - y },
	"*": func(x, y float64) float64 { return x * y },
	"/": func(x, y float64) float64 { return x / y },
}

// parseOperand parses a string to a float64
func parseOperand(op string) (*float64, error) {
	parsedOp, err := strconv.ParseFloat(op, 64)
	if err != nil {
		return nil, fmt.Errorf("cannot parse:%v", err)
	}

	return &parsedOp, nil
}

// calculate returns the result of a 2 operand mathematical expression
func calculate(expr string) (*float64, error) {
	ops := strings.Fields(expr)
	nops := len(ops)
	if nops != 3 {
		return nil, fmt.Errorf("cannot calculate: need 3 ops, got %d", nops)
	}
	left, err := parseOperand(ops[0])
	if err != nil {
		return nil, err
	}
	right, err := parseOperand(ops[2])
	if err != nil {
		return nil, err
	}
	f, ok := operators[ops[1]]
	if !ok {
		return nil, fmt.Errorf("cannot calculate: %s is unknown", ops[1])
	}

	result := f(*left, *right)
	return &result, nil
}

func main() {
	expr := flag.String("expr", "",
		"The expression to calculate on, separated by spaces.")
	flag.Parse()
	result, err := calculate(*expr)
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("%s = %.2f\n", *expr, *result)
}
```

## Concurrency challenges

### Stop copying me

```go
package main

import (
	"flag"
	"log"
)

var messages = []string{
	"Hello!",
	"How are you?",
	"Are you just going to repeat what I say?",
	"So immature",
	"Stop copying me!",
}

// repeat concurrently prints out the given message n times
func repeat(n int, message string) {
	ch := make(chan struct{})
	for i := range n {
		go func(i int) {
			log.Printf("[G%d]:%s\n", i, message)
			ch <- struct{}{}
		}(i)
	}

	for range n {
		<-ch
	}
}

func main() {
	factor := flag.Int64("factor", 0, "The fan-out factor to repeat by")
	flag.Parse()

	for _, m := range messages {
		log.Printf("[Main]:%s\n", m)
		repeat(int(*factor), m)
	}
}
```

### Walk the dog

```go
package main

import (
	"log"
	"math/rand"
	"sync"
	"time"
)

const maxSeconds = 3

type Dog struct {
	name string
}

func (d Dog) fetchLeash() {
	log.Printf("%s goes to fetch leash.\n", d.name)
	randomSleep()
	log.Printf("%s has fetched leash. Woof woof!\n", d.name)
}

func (d Dog) findTreats() {
	log.Printf("%s goes to fetch treats.\n", d.name)
	randomSleep()
	log.Printf("%s has fetched the treats. Woof woof!\n", d.name)
}

func (d Dog) runOutside() {
	log.Printf("%s starts running outside.\n", d.name)
	randomSleep()
	log.Printf("%s is having fun outside. Woof woof!\n", d.name)
}

type Owner struct {
	name string
}

func (o Owner) putShoesOn() {
	log.Printf("%s starts putting shoes on.\n", o.name)
	randomSleep()
	log.Printf("%s finishes putting shoes on.\n", o.name)
}

func (o Owner) findKeys() {
	log.Printf("%s starts looking for keys.\n", o.name)
	randomSleep()
	log.Printf("%s has found keys.\n", o.name)
}

func (o Owner) lockDoor() {
	log.Printf("%s starts locking the door.\n", o.name)
	randomSleep()
	log.Printf("%s has locked the door.\n", o.name)
}

func randomSleep() {
	r := rand.Intn(maxSeconds)
	time.Sleep(time.Duration(r)*time.Second + 500*time.Millisecond)
}

func main() {
	owner := Owner{name: "Jimmy"}
	dog := Dog{name: "Lucky"}
	ownerActions := []func(){
		owner.putShoesOn,
		owner.findKeys,
		owner.lockDoor,
	}
	dogActions := []func(){
		dog.fetchLeash,
		dog.findTreats,
		dog.runOutside,
	}
	executeWalk(ownerActions, dogActions)
}

func executeWalk(ownerActions []func(), dogActions []func()) {
	var wg sync.WaitGroup
	wg.Add(2)
	defer wg.Wait()
	executeActions := func(actions []func()) {
		defer wg.Done()
		for _, a := range actions {
			a()
		}
	}
	go executeActions(ownerActions)
	go executeActions(dogActions)
}
```

### Conference lunch

```go
package main

import (
	"fmt"
	"log"
)

// the number of attendees we need to serve lunch to
const consumerCount = 300

// foodCourses represents the types of resources to pass to the consumers
var foodCourses = []string{
	"Caprese Salad",
	"Spaghetti Carbonara",
	"Vanilla Panna Cotta",
}

// takeLunch is the consumer function for the lunch simulation
// Change the signature of this function as required
func takeLunch(name string, in []chan string, done chan<- struct{}) {
	for _, ch := range in {
		log.Printf("%s eats %s.\n", name, <-ch)
	}
	done <- struct{}{}
}

// serveLunch is the producer function for the lunch simulation.
// Change the signature of this function as required
func serveLunch(course string, out chan<- string, done <-chan struct{}) {
	for {
		select {
		case out <- course:
		case <-done:
			return
		}
	}
}

func main() {
	log.Printf("Welcome to the conference lunch! Serving %d attendees.\n",
		consumerCount)
	var courses []chan string
	doneEating := make(chan struct{})
	doneServing := make(chan struct{})
	for _, c := range foodCourses {
		ch := make(chan string)
		courses = append(courses, ch)
		go serveLunch(c, ch, doneServing)
	}
	for i := 0; i < consumerCount; i++ {
		name := fmt.Sprintf("Attendee %d", i)
		go takeLunch(name, courses, doneEating)
	}

	for i := 0; i < consumerCount; i++ {
		<-doneEating
	}
	close(doneServing)
}
```

### Silent auction

```go

```
