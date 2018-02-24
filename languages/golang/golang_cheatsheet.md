# Go Cheatsheet

## Important Commands

- `go build` compiles
- `go run` compiles and executes
- `go fmt` formats all the code in each file in the current directory
- `go install` compiles and "installs" a package
- `go get` downloads the raw source code of someone else's package
- `go test` runs any tests associated with the current project

## Package

- Very first line must be package definition
- Types of packages
  - Executable: Generates a file that we can run
  - Reusable: Code used as 'helpers'. Good place to put reusable logic
- `package main` means we creating an executable package
  - The main name is special
  - Build will spit out executable file
  - Must have a func called 'main'
- there are builtin package, which have predeclared identifiers that can be accessed anywhere without importing: https://golang.org/pkg/builtin/

## import

- `import fmt` give my package all functionalities from package `fmt`

## func

- `func` is a function in go

    func main() {
      // Function body here
    }

- func called `init()` will be run only once at the beginning

## organizing main.go file

- *from top to bottom*
- Package declaration
- Import other packages that we need
- Declare functions

## var

- If we assign variable after initializing right away, type must be omitted (let go infer it)
- We can initialize variable outside of any function definition. But we cannot assign it right away, we can only assign inside a function

## array and slice

- Array: Fixed length list of things
- Slice: An array that can grow or shrink. Slice internal data representation is an array.
- Array and slice elements must be of same type

    // Make a slice of string, which have length of 10 and capacity of 100.
    // Length means the length of the slice.
    // Capacity means the length of the underlying array
    xs := make([]string, 10, 100)

    // This will return in 'index our of range' error, because it's outside
    // the initialized slice length
    xs[10] = 1

    // append will increase length (but not the capacity) of the slice
    // append return modified copy (the original is immutable)
    xs = append(xs, 10)

    // range <var> is a function that return index and current element
    // it can be used as an iterator
    i, val := range xs

    // In 'for' block initialization, the initialized variable scope is only valid within the for block.
    // Therefore we must re-initialize the 'i' variable below on the next for block.
    //
    // This is also true if we initialize some variables in an 'if' block.
    for i := 0; i < len(xs); i++ {
      // code
    }

## OO vs Go

- in OO, we may create a `Deck` class that have `card` property and `print`, `shuffle` and `saveToFile` methods.
- in Go, we create a new type `deck`, which is a slice of string `[]string` and create functions with `deck` as a receiver.

## type

- There are two kinds of type
  1. Concrete type
  2. Interface type

### type conversion

    []byte("Hi there!")

### receiver function A.K.A method

- Example of receiver function:

    func (d deck) print()

- Any variable of type `deck` now get access to the `print` method
- d is the receiver name (self/this object)
- the go convention is to use the first letter of the type as the receiver name
- the `d` can be omitted if we're not using it within the function, example:

    func (deck) print()

## pointer

- A star in front of type is completely different with a star in front of variable
- A star in front of type x means: a pointer to type x (star is type description)
- A star in front of variable means: value / dereferencing (star is operator)
- in go, if we defined a pointer as a receiver. Go will autodetect whether the receiver is a value or a pointer. (shortcut)
- slice is already a pointer (to array). Even if it is copied in memory, it will still point to the same array
- slice is categorized as `reference types`
- there are also other `reference types`: maps, channels, pointers, functions
- other category is called `value types`

## map

- both the key and the value are statically typed (must be the same type)

## map vs struct

### map

- All keys must be the same type
- All values must be the same type
- Keys are indexed - we can iterate over them
- Reference type
- Don't need to know all the keys at compile time
- Used to represent a collection of related properties
- Accessing map:

    // when we access a map, the map will actually return a second value.
    // the second value is a boolean, it will be true if the value returned are valid
    // it will return false if the value returned is a zero value or not set (please check this)
    _, ok := dbUsers[un]

### struct

- No keys, instead you have to define fields at compile time
- Values can be of different type
- Keys don't support indexing
- Value type
- You need to know all the different fields at compile time
- Used to represent a "thing" with a lot of different properties
- **anonymous** struct is available (struct without explicit type declaration)

## goroutine

    go <function-call>

- concurrency is not parallelism
- main go routine can exit prematurely if it's not have anything else to do (even if the child go routine still working)
- goroutine can also be called upon anonymous function, ex:

    go func() {
      // method definition
    }()

## channels

- Used to communicate between go routine (the only way)
- object that passed through a channel should be of the same type and declared during the initialization

    c := make(chan string)

- Receiving message from a channel is a **blocking call**

    c <- "Some message"
    fmt.Println(<- c)

- `range` can also be used to iterate a channel
- never, ever refer to the same variable from different go routine
- the program will not exit if there are uncaught messages in channels

## interface

### background

- We know that:
  - Every value has a type
  - Every method has to specify the type of its arguments
- So does that mean:
  - Every method we ever write has to be rewritten to accommodate different types even if the logic in it is identical?

- Example #1:

    func (f []float64) print()
    func (s []string) print()
    func (i []int) print()

  - let's say that the print method contains logic for printing the receiver into screen.
  - There will be duplication of logic in implementation of method signature above.

- Example #2:

    type englishBot struct

    func (eb englishBot) getGreeting() string
    func (eb englishBot) printGreeting()

    type spanishBot struct

    func (sb spanishBot) getGreeting() string
    func (sb spanishBot) printGreeting()

  - The duplication of logic can be imagined in the above example

### interface in Go

- Interface alleviate this by allowing types to implement its method signature
- When every methods on an interface are implemented by a type, the type can serve as concrete type of that interface
- The fulfillment of an interface is **implicit**, meaning there is no special syntax to tell that a type wants to implement an interface
- Interface can then be used as arguments in method definition
- Interface is a significant feature in Go that help in code-reuse and doing polymorphism

- Example:

    type human interface {
      speak()
    }

- if any type implement speak(), that type will become `human`.
- The fulfillment of interface is implicit.

- Notes about interfaces
  - interfaces are not generic types (go does not have generic types)
  - interfaces are a contract to help us manage types
  - an interface can contains another interfaces (nested)
  - interfaces are tough. Step #1 is understanding how to read them

- Example #2:
    - To address code duplication in previous section, we can use interface.

    // using interface
    type bot interface {
      getGreeting() string
      printGreeting()
    }

    func (b bot) printGreeting()

- Example #3 - Interface in Go stdlib:
  - The `Reader` interface
    - one of the most common interface in Go stdlib
    - allows us to provide abstraction to different sources of input

    // Source of Data -> Reader -> []byte
    type Reader interface {
      // p = will be mutated within the method
      // n = how many bytes that just read
      // err = nil if there is no error
      Read(p []byte) (n int, err error)
    }

    - the thing that wants to read a data, pass an empty `[]byte]` to thing that it wants to read, which implements the `Reader` interface.
    - the method `Read` will then inject the content into the empty `[]byte`
    - example of using the `Read` method

    bs := make([]byte, 99999)
    res.Body.Read(bs)
    fmt.Println(string(bs)

    - the example above seems 'hackish', there are other way, for example using the `io.Copy` method

    io.Copy(os.Stdout, res.Body)

    // definition of Copy function
    // copyBuffer create a bs as buffer, make([]byte, 32*1024)
    func Copy(dst Writer, src Reader) (written int64, err error) {
      return copyBuffer(dst, src, nil)
    }

  - The `Writer` interface

    // []byte -> Writer -> Some form of output
    type Writer interface {
      // p = truly only used as a source of input (not mutated)
      // n = how many bytes that just read
      // err = nil if there is no error
      Write(p []byte) (n int, err error)
    }

### empty interface

- Exist special interface in Go, which called the empty interface.
- Empty interface is basically an interface without any method signatures.
- Because fulfillment of an interface in Go is implicit, it means that every type fulfill the empty interface.
- It means that the empty interface can be used as a type of an argument in a function, it can be written as `interface{}`

## templates

### static templates

    import "text/template"

    // ParseFiles is used to parse templates from list of filenames
    tpl, err := template.ParseFiles(filenames)

    // We can also use ParseGlob to glob templates from a directory
    tpl, err := tpl.ParseGlob(directory)

    // To avoid tedious if err != nil code, we can use template.Must
    // template.Must does error checking for ParseFiles or ParseGlob
    tpl := template.Must(t *Template, err error)

    // To execute (write) the template and merge them with data,
    // we can use the Execute method.
    // method Execute writes every templates inside the container tpl
    tpl.Execute(writer, data)

    // method ExecuteTemplate needs one extra attribute: the specific
    // template filename to execute
    tpl.ExecuteTemplate(writer, filename, data)

### functions and methods

- functions can be imported into templates using `Funcs(fm FuncMap)`

    tpl := template.Must(template.New("").Funcs(fm).ParseFiles(filename))
    tpl.ExecuteTemplate(writer, filename, data)

- methods from a struct are automatically available if you passed its instance to the template
- But it's best to only calls formatting-related functions or methods only; not for processing or data access in template

### pipelining

    {{ . | fdbl | fsq | fsqrt }}

### html templates

    import "html/template"

- html template has all the functionality that text template has, but introduce some specific html functionalities
- one of the functionality is to escape unsafe contents (e.g. cross site scripting).
- Escaping unsafe contents will be done automatically during execution.

### predefined global functions

- You can find the list of predefined global functions, which we can access freely without importing here: https://golang.org/pkg/text/template/

## time

    import "time"

    t := time.Now()
    t.Format("2006-01-02")

- Reference time, use this when formatting
  `01/02 03:04:05PM '06' -0700`

## server

### synonymous terms

- router
- request router
- multiplexer
- mux
- servemux
- server
- http router
- http request router
- http multiplexer
- http mux
- http servemux
- http server

### HTTP request

- See RFC7231 for details
- Note: I think this is important for API & versioning

- client make request: request line (Method URI HTTP_version<CRLF>)
                       headers
                       body

- server returns response: status line (HTTP_version status_code reason_phrase<CRLF>)
                         headers
                         body

- https://golang.org/pkg/net/http/#pkg-constants see HTTP status codes

### HTTP in Go

- See full documentation here: https://golang.org/pkg/net/http

    type Handler interface {
      ServeHTTP(ResponseWriter, *Request)
    }

- if your type implement `ServeHTTP` above, then your type can be a `Handler`
- handler is an input for important function: `http.ListenAndServe`

- http.ListenAndServe

    func ListenAndServe(addr string, handler Handler) error

- http.ListenAndServeTLS

    func ListenAndServeTLS(addr string, certFile, keyFile string, handler Handler) error

### routing

#### stl ServeMux

    mux := http.NewServeMux()
    mux.Handle("/dog/", handler)
    mux.Handle("/cat", handler)

- We also have defaultServeMux, which will be used when we passed nil into the second argument of http.ListenAndServe

    mux.Handle(addr string, handler Handler)
    mux.HandleFunc(addr string, handler func(ResponseWriter, *Request))

#### julien schmidt's router

- http://godoc.org search: "httprouter"
- julien schmidt's router is very performant
- be aware with forward slash, the behaviour is different with stl ServeMux

### Passing values and files

- POST: values through body
- GET: values through URL

    // FormValue returns the first value for the named component of the query.
    // POST and PUT body parameters take precedence over URL query string values.
    // FormValue calls ParseMultipartForm and ParseForm if necessary and ignores any errors returned by these functions.
    // If key is not present, FormValue returns the empty string.
    // To access multiple values of the same key, call ParseForm and then inspect Request.Form directly.
    req.FormValue(key string) string

    // PostFormValue similar with FormValue but ignore URL query parameters.
    req.PostFormValue(key string) string

    // FormFile returns the first file for the provided form key.
    // Make sure that the encoding type used for sending body is 'multipart/form-data'
    // <form method="POST" enctype="multipart/form-data">
    req.FormFile(key string) (multipart.File, *multipart.FileHeader, error)

### Enctype

- `enctype="application/x-www-form-urlencoded"`: the default for form, ampersand separated key-value on body
- `enctype="multipart/form-data"`: if on your form you need to upload even only one file
- `enctype="text/plain"`: introduced on HTML5 for debugging purpos ***never use it for production!***

### Error handling

    // Catch an error, print the error to log and exit the server
    log.Fatal(http.ListenAndServe(":8080", http.FileServer(http.Dir("."))))

    // Error replies to the request with the specified error message and HTTP code.
    // StatusCode constants are available on http.*
    http.Error(ResponseWriter, Msg, StatusCode)

    // NotFoundHandler returns a simple request handler that replies to each request
    // with a “404 page not found” reply.
    http.Handle("/favicon.ico", http.NotFoundHandler())

### Redirects

- Status codes for redirects
  - 301 - Moved Permanently: The browser won't ask at that address anymore
  - 303 - See other: Changes method to GET
  - 307 - Temporary redirect: Preserve the method

    // manually set redirect
    w.Header().Set("Location", "/")
    w.WriteHeader(http.StatusSeeOther)

    // shorthand for the above method
    http.Redirect(w, req, "/", http.StatusSeeOther)

### State

#### Cookies

- Cookies are little files located on the client machine, which server can write into (if the client allows it)
- key-value pair separated by '='
- cookies will always be sent every client request. Cookies are uniquely identified by domain
- if cookie is disabled by the client, we can embed information in the URL but make sure that we use HTTPS

- Check https://godoc.org/net/http#Cookie for writing/reading cookie in Go.

    http.SetCookie(w, &http.Cookie{
        Name: "my-cookie",
        Value: "some value",
    })

    c, err := req.Cookie("my-cookie")
    if err == http.ErrNoCookie {
      //
    }

- If the `Expires` or `MaxAge` field isn't set, then the cookie is deleted when the browser is closed. This is colloquically known as a "session cookie".
- `Expires` set the exact time when the cookie expires, Expires is **Deprecated**
- `MaxAge` sets how long the cookie should live (in seconds), and is the prefered method for setting cookies lifetime.
- There are more attributes that can be set, please check documentation for details.
- We can also delete cookie by setting `MaxAge = -1`

    c, err := req.Cookie("my-cookie")
    if err == http.ErrNoCookie {
      //
    }
    c.MaxAge = -1 //delete cookie
    http.SetCookie(w, c)
    http.Redirect(w, req, "/", http.StatusSeeOther)

#### Sessions

- Session is a unique identifier that can be used by the server for identifying a client, the client must always shown it on every requests.
- Session can be stored in cookie or other client-side persistence method.
- Session can be expired by:
  1. Setting MaxAge to specific value (but control on client)
  2. Timestamping user last activity on session table on database, with periodic check by background process and delete as necessary (control on server)

## persistence

- http://godoc.org search: "sql"
- https://godoc.org/database/sql
- sql package must be used in conjunction with a database driver. See http://golang.org/s/sqldrivers

## JSON

- http://godoc.org search: "json"
- https://blog.golang.org/json-and-go
- https://godoc.org/encoding/json
- we can use this webapp to determine go structure of a json: https://mholt.github.io/json-to-go/
- Marshal & unmarshal vs encode & decode
- we can add tags to the struct, to map different field names between json and struct.

    type AutoGenerated []struct {
      Precision string `json:"precision"`
      Latitude float64 `json:"Latitude"`
      Longitude float64 `json:"Longitude"`
      Address string `json:"Address"`
      City string `json:"City"`
      State string `json:"State"`
      Zip string `json:"Zip"`
      Country string `json:"Country"`
    }

- Field should be uppercase (visible), otherwise it won't be marshalled
- unmarshalling null value will result in zero value in go

## code organization

- You can distribute code into multiple files, as long as the package have same name and on the same folders
- But if we want to use `go run`, then we must specify all files to run. It's better if we use `go build` if our project spans multiple files.
- https://github.com/GoesToEleven/golang-web-dev/tree/master/045-code-organization
- http://www.alexedwards.net/blog/organising-database-access

## misc

### UUID

- http://godoc.org search: "uuid"
- https://godoc.org/github.com/satori/go.uuid
- https://godoc.org/github.com/nu7hatch/gouuid

    go get github.com/satori/go.uuid

### Bcrypt

- https://godoc.org/golang.org/x/crypto/bcrypt

    go get golang.org/x/crypto/bcrypt

- password should be []byte
- cost in bcrypt is for setting complexity for hashing algorithm

    import "golang.org/x/crypto/bcrypt"

    // Generate password
    bs, err := bcrypt.GenerateFromPassword([]byte(p), bcrypt.MinCost)

    // Compare password
    err := bcrypt.CompareHashAndPassword(u.Password, []byte(p))
