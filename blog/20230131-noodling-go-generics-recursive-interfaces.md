# Noodling w/Generics and Recursive Interfaces, in Go

> January 31, 2022

I did a bit of noodling over the weekend in an attempt to figure out
how to write interface that abstracted over methods whose return
type is identical to the receiver's type.

For example, suppose we have a struct through which we log stuff:

```go
type StdoutLogger struct {
	out    ioutil.Writer
	outMu  sync.Mutex
	fields map[string]interface{}
}

func (n *StdoutLogger) WithFields(fields map[string]interface{}) (out *StdoutLogger) {
	for k, v := range fields {
		out.fields[k] = v
	}

	return
}

func (n *StdoutLogger) Infof(format string, args ...interface{}) {
	n.outMu.Lock()
	defer n.outMu.Unlock()

	s := fmt.Sprintf(format, args...)

	if len(n.fields) > 0 {
		s += " "
	}

	for k, v := range n.fields {
		s += fmt.Sprintf("%s=%+v", k, v)
	}

	n.out.Write([]byte(s))
}
```

...and we've got some other struct that we use during test which
ignores all requests to log stuff:

```go
type NoopLogger struct {}

func (n *NoopLogger) WithFields(fields map[string]interface{}) *NoopLogger {
	return n
}

func (n NoopLogger) Infof(format string, args ...interface{}) {
	return
}
```

## Before Go Generics

Before the Go "generics" feature was released, defining an interface
that abstracted over both structs was not possible ([link][1]). For 
example, if we had a pre-generics interface that looked like:

```go
type Logger interface {
	WithFields(fields map[string]interface{}) Logger
	Infof(format string, args ...interface{})
}
```

...there would be no way to satisfy it with types that had these
signatures:

```go
func (n *NoopLogger) WithFields(fields map[string]interface{}) *NoopLogger
func (n *StdoutLogger) WithFields(fields map[string]interface{}) *StdoutLogger
```

...because of the different return types of each struct's 
`WithFields` method.

## Polymorphic Interfaces

Now that generics have landed, we _can_ define an interface that 
abstracts over both of these structs:

```go
type Logger[T any] interface {
	WithFields(fields map[string]interface{}) T
	Infof(format string, args ...interface{})
}

var x Logger[*NoopLogger] = new(NoopLogger)
var y Logger[*StdoutLogger] = new(StdoutLogger)
```

Using that interface involves using generic "constraints" 
([link][3]), like so:

```go
type Config[T Logger[T]] struct {
	username string
	password string
	logger   T
}
```

What we're expressing with this `Config` struct is that the struct
owns a logger of type `T`, where `T` is constrained to any type
which satisfies the `Logger` interface (which itself is
polymorphic).

## Weak Type Inference

While it is certainly cool that we can now define these sorts of
polymorphic interfaces, Go's type inference is so weak as to make
use of those interfaces quite awkward. In my experiments, I 
frequently found it to be the case that I needed to explicitly set a 
type variable in a place where I would expect it to be inferred.

Suppose for a moment that we export a bunch of config-manipulating 
functions, and that one of those functions can be used to configure 
out application to use a logger of some type that the user provides.

```go
// Config is a struct which holds our application's configuration.
type Config[T Logger[T]] struct {
	username string
	password string
	logger   T
}

// WithUsername configures the system to use the provided username.
func WithUsername[T Logger[T]](username string) func(config *Config[T]) {
	return func(p *Config[T]) {
		p.username = username
	}
}

// WithPassword configures the system to use the provided password.
func WithPassword[T Logger[T]](password string) func(config *Config[T]) {
	return func(p *Config[T]) {
		p.password = password
	}
}

// WithLogger configures the system to use the provided logger.
func WithLogger[T Logger[T]](logger T) func(config *Config[T]) {
	return func(p *Config[T]) {
		p.logger = logger
	}
}

// NewConfig creates a new configuring from the provided options.
func NewConfig[T Logger[T]](opts ...func(config *Config[T])) Config[T] {
	cfg := Config[T]{}

	for _, opt := range opts {
		opt(&cfg)
	}

	return cfg
}
```

Suppose we wanted to construct a configuration that used our 
`NoopLogger`. In many other languages, we'd load all of our 
configuration functions into a monomorphized (i.e. non-polymorphic) 
slice, and then we'd pass that slice around:

```go
func main() {
	opts := []func(config *Config[*NoopLogger]){
		WithUsername("foo"),
		WithPassword("bar"),
		WithLogger(&NoopLogger{}),
	}

	cfg := NewConfig(opts...)

	fmt.Printf("cfg is: %+v\n", cfg)
}
```

Unfortunately, Go's type inference is not able to infer that `T` is 
`*NoopLogger` for the `WithUsername` and `WithPassword` functions, 
in spite of the fact that `opts` is not polymorphic.

If you try to run this code, you'll see something like ([link][2]):

```go
./prog.go:70:15: cannot infer T (prog.go:33:19)
./prog.go:71:15: cannot infer T (prog.go:40:19)

Go build failed.
```

To work around the lack of type inference, you'll need to explicitly
parameterize each function, like so:

```go
func main() {
	opts := []func(config *Config[*NoopLogger]){
		WithUsername[*NoopLogger]("foo"),
		WithPassword[*NoopLogger]("bar"),
		WithLogger(&NoopLogger{}),
	}

	cfg := NewConfig(opts...)

	fmt.Printf("cfg is: %+v\n", cfg)
}
```

Not particularly awesome.

## Conclusion

Go's generics implementation allows us to solve some problems that 
were previously difficult or impossible to solve, but the inference
algorithm is such that explicit parameterization is required in
places where one would expect types to be inferred.

[1]: https://tip.golang.org/doc/go1.17_spec#Interface_types
[2]: https://go.dev/play/p/sdb9qGPV7GI
[3]: https://go.dev/blog/intro-generics
