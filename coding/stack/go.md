# Go Patterns

## Error Handling

### Always Check Errors
```go
result, err := doSomething()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}
```

### Custom Errors
```go
var ErrNotFound = errors.New("resource not found")

func GetUser(id string) (*User, error) {
    user, ok := users[id]
    if !ok {
        return nil, ErrNotFound
    }
    return user, nil
}

// Caller can check specific error
if errors.Is(err, ErrNotFound) {
    // handle not found
}
```

## Concurrency

### Goroutines with WaitGroup
```go
var wg sync.WaitGroup

for _, item := range items {
    wg.Add(1)
    go func(item Item) {
        defer wg.Done()
        process(item)
    }(item)
}

wg.Wait()
```

### Channels for Communication
```go
results := make(chan Result, len(items))

for _, item := range items {
    go func(item Item) {
        results <- process(item)
    }(item)
}

for range items {
    result := <-results
    // handle result
}
```

### Context for Cancellation
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

select {
case result := <-doWork(ctx):
    return result, nil
case <-ctx.Done():
    return nil, ctx.Err()
}
```

## Project Structure

```
project/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   ├── service/
│   └── repository/
├── pkg/
│   └── shared/
├── go.mod
└── go.sum
```

## Idioms

### Accept Interfaces, Return Structs
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

func Process(r Reader) (*Result, error) {
    // ...
}
```

### Defer for Cleanup
```go
func ReadFile(path string) ([]byte, error) {
    f, err := os.Open(path)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    
    return io.ReadAll(f)
}
```

### Table-Driven Tests
```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 1, 2, 3},
        {"negative", -1, -2, -3},
        {"zero", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("got %d, want %d", result, tt.expected)
            }
        })
    }
}
```
