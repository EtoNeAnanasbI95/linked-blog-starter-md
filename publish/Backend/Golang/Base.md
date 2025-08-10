```table-of-contents
```
## Basic syntax

- В го есть общепринятая практика, когда функция может вернуть сразу два значения. Но иногда один из этих значений нам не нужно, а компилятор го даст нам пиздюлей если увидит переменную которая никак не используетсяю Поэтому, разрабы придумали следующее — если одно из двух возвращаемых значений нам не нужно, то мы просто пишем _ вместо имени переменной
- В функции можно не только указать тип данных который мы возвращаем, но сразу объявить имя переменной которую надо будет вернуть
- В го классов по сути нет, но есть структуры, которые в какой-то мере могут являться их аналогами. Сами же структуры ведут себя как структуры из JS, но на них можно вешать методы
- Как такового ООП в го нет, но я бы не сказал что это так, потому что классы — это структуры с методами, а когда мы преисполнились солидом мы можем начать юзать интерфейсы, просто нам не надо наследовать структуры от них, достаточно просто засунуть в аргумент функции какой-либо интерфейс

## Duck typing

- Концепция объявления методов и полей у структур и пакетов. Всё что было объявлено с заглавной буквы — считается экспортируемым, всё что с маленькой — инкапсулированным
## Go agreements

- Go, like other languages, have some general agreements:
    1. _**Export variables**_ must be named with PascalCase
    2. _**Must functions.**_ Functions starts with “Must”, doesn’t return any error, they are just will be panic
    3. _**Test functions.**_ Names of all test functions should starts with “Test”
## Pointers

- Указатель это какая переменная которая содержит в себе адрес в памяти, к который можно разыменовать и изменить его значение, или просто его получить

```go
func changeMe(storke *string) {
	*stroke = "changed stroke"
}

func main() {
	stroke := "text"
	changeMe(&stroke)
	fmt.Println(stroke) // changed stroke
}
```

## Channels

- Channels — is a variable in the heap, that can be used by different go routines at any time. Example:
    
    ```go
    ch := make(chan int)
      // in this case, "make" allocated memory from the heap
      // and initializes a "chan int",
    	// which simply declares what the channel should be
    
    ch <- 1 // write a value in channel
    ch <- 2 // write some one value in channel
    
    fmt.Println(<-ch) // read value from channel (it's will return a 1)
    fmt.Println(<-ch) // read some one value from channel (it's will return a 2)
    ```
    

## Context and select statement

- Context is like “cancellation token” from c#, but better, because it has more features. It’s basically a channel that holds the information about a function can work or not. Context often uses with select statement.
    
- Select — is a statement like `switch`, but can work with channels
    
- Example:
    
    ```go
    func doWork(ctx context.Context) { // create a function, that constantly cheaks a contex condition
        for {
            select { // select statement, that work like swith, but use only in functions with context
            case <-ctx.Done():
                // context is canceled or time is over
                fmt.Println("Task canceled:", ctx.Err())
                return
            default:
                fmt.Println("Work...")
                time.Sleep(1 * time.Second) // Work simulation
            }
        }
    }
    
    func main() {
        // Create context with 3 seconds timeout
        ctx, cancel := context.WithTimeout(
    	    context.Background(),// context entry point (like a "cancelation token default state")
    	    3*time.Second
        )
        defer cancel() // free up resources
    
        // start go routine
        go doWork(ctx)
    
        // wait 5 seconds
        time.Sleep(5 * time.Second)
    }
    ```
    

## Packages and modules

- Every your .go file in project folders — is a little part of package, and this package should be named the same as parent folder.
- Packages add more structure to your project and declared for each file in project `package <name of your package>`
- Modules is a name of your application or library and declared in go.mod files

## Project structure

- Golang there are many different directories that a commonly used in go applications. For example:
    - Configs — is a folder for configuration files
    - Migrations — is a folder for sql scripts and sql migrations
    - cmd — is a folder for start up code for application
    - pkg — is a folder for .go files, that can be represented as a library or module
    - internal — just backend of your application
    - other…

## Struct tags

- In go, structs have some function called struct tags. Json tag exist for correct marshaling and unmarshaling json and yaml. For example:
    
    ```go
    type user struct {
    	ID   int    `json:"id"   yaml:"id"`
    	Name string `json:"name" yaml:"name"`
    }
    ```
    

## Interfaces

- Interfaces — is a very powerful and most used element in go. It a just declare a some methods that will be implemented in struct. This allow to make different realization a methods, and make your code more readable for other developers

## Generic methods

- Generics — is a methods, that can receive arguments of any types, and correctly work with them

```go
var numInt1 int = 0
var numInt2 int = -1

var numInt1 uint = 0
var numInt2 uint = 1

func WhoIsMore[T constraints.Ordered](a, b T) T {
    if a > b {
      return a  
    } else {
        return b
    }
}
```