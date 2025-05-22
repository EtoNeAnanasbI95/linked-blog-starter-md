# Memory
## Общее
- В контексте работы с памятью существует несколько терминов которые нужно знать
    1. Аллокация — это выделение памяти под новую переменную
    2. Эвакуация — это перенос данных из одного участка памяти в другой
## Выделение памяти
* Указатели в golang, занимают разное кол-во места в памяти.
	На x64 архитектуре, они занимают 8 байт
	На x32 архитектуре, они занимают 4 байта

x32 архитектура почти не используется, потому что она устарела

# Runtime

```go
package main

func init() { // run before main if init function is exist
}

func main() { // main function of package, wich is launched first if there is no init
}
```
# UI libraries

- Write pretty GUI in go lang is not a good idea, because it not what this language was designed for, but it has some libraries for this shit:
    1. **go-gtk**
    2. **qt**
    3. **fyne (best)**
    4. **walk**
    5. **gioui**

# API creation libraries

1. Echo (render templates, websocket)
2. Gin (most popular, just fast and easy)
3. chi (light weight, simple, very good for micro services)
4. net/http (base)
5. Os/exec (trash)
6. gRPC

### Adding

- I recommend to use the [https://github.com/githubnemo/CompileDaemon](https://github.com/githubnemo/CompileDaemon) util, she can help to develop any api

# DB/ORM libraries

1. GORM (most popular and very good library)
2. ent
3. go-pg
# bufio
* Стандартная библиотека bufio, нужна для работы с «буферами». Основная киллерфича этой библиотеки, это функция, bufio.NewBuffer(w io.writer | io.Reader | io.ReadWriter) она принимает в себя врайтер или ридер, и заворачивает его в буфер. Зачем? Чтобы повесить на переменную ридера/врайтера буферизацию
* Буферизация — это процесс, при котором данные временно накапливаются в памяти (в буфере), прежде чем они будут переданы в конечное место назначения (например, записаны в файл или отправлены по сети). Вместо того чтобы записывать каждый маленький кусочек данных напрямую, bufio.Writer собирает их в буфер и выполняет запись только тогда, когда буфер заполняется или вызывается метод Flush().
* Пример:
    Без буферизации: Если ты вызываешь w.Write([]byte("a")) 100 раз, это приведёт к 100 вызовам метода Write базового io.Writer (например, системным вызовам для записи в файл).
    
    С буферизацией: bufio.Writer собирает эти 100 байтов в буфер и выполняет запись одним большим куском, когда буфер заполняется или вызывается Flush().