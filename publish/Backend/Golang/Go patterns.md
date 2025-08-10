```table-of-contents
```
# Option Pattern

## Проблема

```go

func NewServer(addr string, port int, timeout time.Duration, maxConn int, protocol string) *Server {
    // ...
}

```
## Решение
```go
type Server struct {
    addr     string
    port     int
    timeout  time.Duration
    maxConn  int
    protocol string
}

type Option func(*Server)

func WithTimeout(t time.Duration) Option {
    return func(s *Server) {
        s.timeout = t
    }
}

func WithMaxConn(max int) Option {
    return func(s *Server) {
        s.maxConn = max
    }
}

```

```go

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{
        addr:     addr,
        port:     8080,
        timeout:  30 * time.Second,
        maxConn:  100,
        protocol: "tcp",
    }

    for _, opt := range opts {
        opt(s)
    }
    return s
}

```

**Использование:**
```go
server := NewServer("localhost",
    WithTimeout(60*time.Second),
    WithMaxConn(500),
)

```
## Дополнительные возможности
- **Валидация параметров:**
```go
func WithPort(port int) Option {
    return func(s *Server) {
        if port < 0 || port > 65535 {
            panic("invalid port number")
        }
        s.port = port
    }
}

```
- **Группировка опций:**
```go
type NetworkOptions struct {
    Protocol string
    Timeout  time.Duration
}

func WithNetworkOptions(opts NetworkOptions) Option {
    return func(s *Server) {
        s.protocol = opts.Protocol
        s.timeout = opts.Timeout
    }
}

```