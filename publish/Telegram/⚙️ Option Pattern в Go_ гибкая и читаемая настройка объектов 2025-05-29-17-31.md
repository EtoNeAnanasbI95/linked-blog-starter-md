**Forwarded from [Golang вопросы собеседований](https://t.me/golang_interview/1192)**

**⚙️ Option Pattern в Go: гибкая и читаемая настройка объектов**

В статье Leapcell ["Option Pattern in Go: Advanced Parameter Handling"](https://dev.to/leapcell/option-pattern-in-go-advanced-parameter-handling-15hf) рассматривается эффективный способ управления параметрами функций и конструкторов в Go.:contentReference[oaicite:7]{index=7}

## 🧩 Проблема

:contentReference[oaicite:9]{index=9}:contentReference[oaicite:11]{index=11}

```

func NewServer(addr string, port int, timeout time.Duration, maxConn int, protocol string) *Server {
    // ...
}

```

**Недостатки:**
- :contentReference[oaicite:13]{index=13}
- :contentReference[oaicite:16]{index=16}
- :contentReference[oaicite:19]{index=19}
- :contentReference[oaicite:22]{index=22}:contentReference[oaicite:24]{index=24}

## 💡 Решение: Option Pattern

:contentReference[oaicite:26]{index=26}:contentReference[oaicite:28]{index=28}

```

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


:contentReference[oaicite:30]{index=30}:contentReference[oaicite:32]{index=32}

```

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

```

server := NewServer("localhost",
    WithTimeout(60*time.Second),
    WithMaxConn(500),
)

```

## 🛠️ Дополнительные возможности

- **Валидация параметров:**

```

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

```

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

✅ Преимущества

- :contentReference[oaicite:34]{index=34}
- :contentReference[oaicite:37]{index=37}
- :contentReference[oaicite:40]{index=40}
- :contentReference[oaicite:43]{index=43}:contentReference[oaicite:45]{index=45}

:contentReference[oaicite:47]{index=47}:contentReference[oaicite:49]{index=49}

📌 [Читать](https://dev.to/leapcell/option-pattern-in-go-advanced-parameter-handling-15hf)