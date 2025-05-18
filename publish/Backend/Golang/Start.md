## Variables and constants

```go
//nums
	//default nums
	var a int8 = -1
	var b uint8 = 2
	var c byte = 3  // byte - synonim uint8
	var d int16 = -4
	var f uint16 = 5
	var g int32 = -6
	var h rune = -7     // rune - synonim int32
	var j uint32 = 8
	var k int64 = -9
	var l uint64 = 10
	var m int = 102
	var n uint = 105
	
	//nums with floating point
	var a float32 = 4.5
	var b float64 = 0.23
	
	//complex nums like floating nums
	var f complex64 = 1+2i
	var g complex128 = 4+3i
	
	
//lines
var name string = "Hello NotPineapple"

//true/false
var a bool = true
a = false

//implicit typing
a := "AHHAHAHAHHA" //string
b := 123 //int
c := true //bool

// constant declaration
const a string = "NotPineapple"
```
## If, for, switch, range, errors, panic, recover

```go
a := true

// if example
if a {
	fmt.Println("a is exists")
}

//for example
for {
	fmt.Println("wow, it's infinity for loop")
}

//switch example
day := "вторник"

switch day {
case "понедельник":
    fmt.Println("Сегодня понедельник. Начало рабочей недели!")
case "вторник":
    fmt.Println("Сегодня вторник. Работаем дальше!")
case "среда":
    fmt.Println("Сегодня среда. Половина недели позади!")
case "четверг":
    fmt.Println("Сегодня четверг. Почти к выходным!")
case "пятница":
    fmt.Println("Сегодня пятница. Скоро выходные!")
case "суббота", "воскресенье":
    fmt.Println("Сегодня выходной! Время отдохнуть!")
default:
    fmt.Println("Неизвестный день.")
}

//range example
b := []int{3, 2, 4, 19, 100, 134}

for index, value := range b {
	fmt.Println("iterator index: " + index + ", value: " + value)
}

// error example
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

someValue, err := divide(1, 0) // retun err

//panic example
if err != nil {
	panic("you gay")
}

//recover example

//recover needs for recover from panics

func riskyOperation() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in riskyOperation:", r)
        }
    }()
    panic("an unexpected error occurred")
}

func main() {
    riskyOperation()
    fmt.Println("Program continues running")
}
```