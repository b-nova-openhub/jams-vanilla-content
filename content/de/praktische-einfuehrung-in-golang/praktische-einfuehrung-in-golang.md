<b-nova-content-header>
version: 6
title: Praktische Einführung in Go
description: Go ist eine beliebte Sprache im Cloud-Umfeld. Go könnte schon bald der neue Standard für Microservices und Container-fähigen Applikationen sein.
ogImage: 20210526-og-apache-kafka.png
date: 2020-10-20
author: rschneider
categories: cloud, tech
tags: golang, microservices, handson, foss
showComments: true
publish: true
</b-nova-content-header>

<a href="https://golang.org/" target="_blank">Go</a> ist eine beliebte Sprache im Cloud-Umfeld. Viele bekannte
Anwendungen, sind aus der
<a href="https://thenewstack.io/go-the-programming-language-of-the-cloud" target="_blank">Cloud nicht mehr
wegzudenken</a>, wie beispielsweise _Docker_, _Kubernetes_, _Istio_ oder auch _Terraform_, wurden in Go geschrieben.
Dies bezeugt auch
<a href="https://github.com/spf13" target="_blank">Steve Francia</a>, Product-Owner von Go bei Google, <a
href="https://thenewstack.io/go-the-programming-language-of-the-cloud/" target="_blank">in einem Interview aus dem Jahr
2019</a> worin er eine bewusste Ausrichtung von Go in das Cloud-Umfeld thematisiert und auch das neu
entwickelte <a href="https://github.com/google/go-cloud" target="_blank">Go Cloud Development Kit</a>
vorstellt. Es scheint somit angebracht sich die _Lingua franca_ mal etwas genauer anzuschauen.

### First things first

Go (auch _Golang_ genannt) ist eine kompilierbare Sprache, die Nebenläufigkeit (Concurrency) unterstützt und über eine
automatische Speicherbereinigung (Garbage collection) verfügt. Zudem ist der Sprachsyntax minimalistisch und orientiert
sich am Hardware-nahen C. Die <a href="https://golang.org/ref/spec" target="_blank">Go Programming Language
Specification</a> ist gerade mal 50-Seiten lang. Die Idee bei Go ist eine kleinstmögliche Anzahl an
einfachen, <a href="http://www.catb.org/~esr/writings/taoup/html/ch04s02.html#orthogonality" target="_blank">
orthogonalen</a> Instruktionen bereitzustellen, die sich in eine überschaubare Anzahl von Patterns zusammenbauen lassen.
Dies mindert die Gesamtkomplexität. Somit ist es einfacher Code zu schreiben, zu verstehen und zu warten, da es oft nur
einen bestimmten Weg gibt.

Die Features von Go ergeben eine Sprache die sich besonders gut für **skalierbare**, **cluster-fähige**
Applikationen eignen, die darauf ausgelegt sind genau eine Aufgabe besonders gut und effizient zu lösen. Der Go-Compiler
generiert kleine Binärartefakte die kleine Footprints von Docker-Images garantieren. Ausserdem stellt das _Go Cloud
Development Kit_ eine Library bereit die gängige, Cloud-typische Operationen wie das Lesen von Blob Storage (AWS S3)
oder Health-Checks abdeckt. Aus diesen Gründen eignet sich Go für **Microservices**.

### Snippet unter der Lupe

Um gleich ein Gefühl für die Sprache zu bekommen, lassen Sie uns ein zwei kurze Code-Beispiele unter die Lupe nehmen,
die die Features von Golang veranschaulichen. Beim ersten Beispiel geht es um eine einfache objekt-orientierte Klasse,
die Deklaration von einem Datentyp und Funktionen aufzeigt. Das zweite Beispiel zeigt auf, wie man Nebenläufigkeit in
Golang mit _Goroutines_ und _Channels_ benutzt. Die Snippets enthalten Kommentare inline die den Ablauf erklären.

##### Eine einfache objekt-orientiere Klasse

Das folgende Snippet implementiert einen simplen, abstrakten Datentyp `Stack` im Package `collection` mit Golang:

```golang
// Dies ist ein Kommentar. Das Snippet ist Teil von der Klasse 'collection'
package collection

// Der Wert Null von Stack ist eine leeres Array 'data'
type Stack struct {
	data []string
}

// Die Funktion Push() addiert 'x' in das Array 'data' im Stack 's'
func (s *Stack) Push(x string) {
	s.data = append(s.data, x)
}

// Die Funktion Pop() entfernt das oberste Element im Array 'data' im Stack 's'
func (s *Stack) Pop() string {
	n := len(s.data) - 1
	res := s.data[n]
	// nötig um einen Memory-Leak zu vermeiden
	s.data[n] = ""
	s.data = s.data[:n]
	return res
}

// Die Funktion Size() gibt die Anzahl von Element im Array 'data' im Stack 's' zurück
func (s *Stack) Size() int {
	return len(s.data)
}
```

##### Implementation von Goroutines und Channels

Die Nebenläufigkeit bei Go beruht auf Goroutines und Channels. Eine Goroutine ist ein paralleler Thread. Channels sind
Verbindungen die Goroutines miteinander kommunizieren lassen. Damit kann man gewisse Teile eines Programms nebeneinander
und ab anderen Punkten synchron laufen lassen.

Das folgende Snippet schreibt konstant `"ping"` aus. Die Funktion `pinger()` schreibt `"ping"` in die
Channel-Variable `c`. Die Funktion `printer()` gibt all Sekunde den Inhalt von `c` über eine Zwischenvariable `msg`
aus. Mit `Enter`kann man die konstante Ausgabe stoppen.

```golang
package main

import (
	"fmt"
	"time"
)

func pinger(c chan string) {
	for i := 0; ; i++ {
		c <- "ping"
	}
}

func printer(c chan string) {
	for {
		msg := <-c
		fmt.Println(msg)
		time.Sleep(time.Second * 1)
	}
}

func main() {
	var c chan string = make(chan string)

	go pinger(c)
	go printer(c)

	var input string
	fmt.Scanln(&input)
}
```

Der `<-`-Operator sendet Werte in die Channel-Variable `c`. Per `:= <-`-Operator wird der Wert aus der Channel-Variable
wieder ausgelesen. Dieser Wert ist Routine-aware und ist somit zeitlich immer mit dem aktuellen Wert synchron.

### Microservice mit Go

Die Basics von Go sind erläutert, jetzt lassen Sie uns die Hände ein wenig schmutzig machen. Ein einfacher Microservice
stellt üblicherweise eine Reihe von Daten über eine API per `REST-Schnitstelle` bereit. In unserem Fall wird ein Array
von Kaktus-Objekten `[]Cactus` über die Schnittstelle bereitgestellt.

#### Requirements

Um das folgende Hands-on durchzuführen, werden folgende Tools vorausgesetzt:

- Go-Compiler <a href="https://golang.org/doc/install" target="_blank">installiert</a>
- Editor (IntelliJ, VSCode, Atom, vim, ...)
- Postman um die API zu testen
- 5 Minuten Ihrer Zeit

**Resolve** zuerst die externe Library `gorilla/mux` mit folgendem Befehl:

```shell script
$ go get -u github.com/gorilla/mux
```

**Erstellen Sie** danach eine Datei mit den Namen `cactusService.go` in einem beliebigen Verzeichnis. Die go-Datei muss
folgenden Inhalt haben:

```golang
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"strconv"

	"github.com/gorilla/mux"
)

type Cactus struct {
	ID          string `json:"id"`
	ImageNumber string `json:"imageNumber"`
	Name        string `json:"name"`
	Features    string `json:"features"`
}

var cacti []Cactus

func getCacti(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(cacti)
}

func getCactus(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	params := mux.Vars(r)
	for _, item := range cacti {
		if item.ID == params["id"] {
			json.NewEncoder(w).Encode(item)
			return
		}
	}
}

func createCactus(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	var newCactus Cactus
	json.NewDecoder(r.Body).Decode(&newCactus)
	newCactus.ID = strconv.Itoa(len(cacti) + 1)
	cacti = append(cacti, newCactus)
	json.NewEncoder(w).Encode(newCactus)
}

func updateCactus(w http.ResponseWriter, r *http.Request) {
	// tbd
}

func deleteCactus(w http.ResponseWriter, r *http.Request) {
	// tbd
}

func main() {
	cacti = append(cacti, Cactus{ID: "1", ImageNumber: "8", Name: "San Pedro Cactus", Features: "small, middle-sized,
		tasty, red"}, Cactus{ID: "2", ImageNumber: "6", Name: "Palo Alto Cactus", Features: "big, tall, juicy, green"})

		router := mux.NewRouter()

		router.HandleFunc("/cactus", getCacti).Methods("GET")
		router.HandleFunc("/cactus", createCactus).Methods("POST")
		router.HandleFunc("/cactus/{id}", getCactus).Methods("GET")
		router.HandleFunc("/cactus/{id}", updateCactus).Methods("POST")
		router.HandleFunc("/cactus/{id}", deleteCactus).Methods("DELETE")

		log.Fatal(http.ListenAndServe(":5000", router))
	}
```

Jetzt kann die go-Datei wie folgt kompiliert werden. Führen Sie dazu den Befehl im gleichen Verzeichnis aus:

```shell script
$ go build -o cactusService
```

Das daraus resultierende Binary ist **6.7 MB** gross. Jetzt kann unter `localhost:5000/cactus/` die REST-Schnittstelle
aufgerufen werden. Dazu kann Postman genutzt werden, um komfortabel die Abfrage-Parameter einzustellen.

Glückwunsch, Ihr erster Microservice in Go ist funktional! Jetzt gilt es dieses Wissen weiter zu vertiefen und einen
entsprechenden Use-Case bei Ihnen im Betrieb zu erzielen. Mit Go lassen sich wartbare, prägnante Services schreiben die
mit kleinem Footprints und den _Best Breeds_ der Cloud-Welt trumpfen.

#### Weiterführende Links:

##### Web-Frameworks

<a href="https://gobuffalo.io/en/" target="_blank">Buffalo | A Go web development eco-system, designed to make your life
easier</a>

<a href="https://github.com/revel/revel" target="_blank">Revel Framework | A high productivity, full-stack web framework
for the Go language</a>

##### Lernmaterial zu Go

<a href="https://www.educative.io/blog/golang-tutorial" target="_blank">Educative | Getting started with Golang: a
tutorial for beginners</a>

<a href="https://golang.org/doc/code.html" target="_blank">Golang | How to Write Go Code</a>