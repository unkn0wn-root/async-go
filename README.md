# Async

```go
import "github.com/unkn0wn-root/async-go"
```

Package `async` provides asynchronous primitives and utilities.

## Usage

`Future` is a generic type that represents a value that will be resolved at some point in
the future.

```go
type User struct {
	ID   int
	Name string
}

fut := new(async.Future[User])

// Simulate long computation or IO by sleeping before and resolving the future.
go func() {
	time.Sleep(500 * time.Millisecond)
	user := User{ID: 1, Name: "Tom Aslam"}

	async.ResolveFuture(fut, user, nil)
}()

// Block until the future is resolved.
user, err := fut.Value()

fmt.Println(user, err)
// output: {1 Tom Aslam} <nil>
```


### Custom Futures & `Latch`

`Latch` is a synchronization primitive that can be used to block until a desired state is reached.
It is useful for implementing custom future types.

```go
type User struct {
	ID   int
	Name string
}

type UserFuture struct {
	async.Latch
	User User
	Err  error
}

func ResolveUser(fut *UserFuture, user User, err error) {
	async.Resolve(fut, func() {
		fut.User, fut.Err = user, err
	})
}

// This example demonstrates how Latch can be embedded into a custom type to create a
// Future implementation.
func ExampleLatch_customType() {
	userFut := new(UserFuture)

	// Simulate long computation or IO by sleeping before and resolving the future.
	go func() {
		time.Sleep(500 * time.Millisecond)
		user := User{ID: 1, Name: "Tom Aslam"}

		ResolveUser(userFut, user, nil)
	}()

	// Block until the future is resolved.
	async.Await(userFut)

	fmt.Println(userFut.User, userFut.Err)
	// output: {1 Tom Aslam} <nil>
}
```
