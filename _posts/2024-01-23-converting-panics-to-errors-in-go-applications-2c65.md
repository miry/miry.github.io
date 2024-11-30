---
url: https://dev.to/miry/converting-panics-to-errors-in-go-applications-2c65
canonical_url: https://dev.to/miry/converting-panics-to-errors-in-go-applications-2c65
title: Converting Panics to Errors in Go Applications
slug: converting-panics-to-errors-in-go-applications-2c65
description: This article explores various techniques in Golang to recover from panics
  in different situations.
tags:
- go
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2024-01-23-converting-panics-to-errors-in-go-applications-2c65-cover_image-1m2g9etk91c5enaxfyge.jpg)

# Converting Panics to Errors in Go Applications


> Foto von <a href="https://unsplash.com/de/@thesollers?utm_content=creditCopyText">Anton Darius</a> auf <a href="https://unsplash.com/de/fotos/nahaufnahme-des-lagerfeuers-00Dy3T-PtE8">Unsplash</a>

Golang is a popular language that makes it easy for beginners to start writing concurrent applications. Often, panics (or exceptions) are processed in the same way as errors, leading to duplicated handling for both. Errors should ideally be handled in a single place to eliminate redundancy in managing panics and errors.

This article explores various techniques to recover from panics in different situations.

In most Golang conventions, functions that return an `error` object are commonly used.
Let's explore this with a simple example:

```golang
// panic.go
package main

import (
  "fmt"
  "os"
)

func main() {
  err := run()
  if err != nil {
    fmt.Printf("error: `%+v`\n", err)
    os.Exit(1)
  }
  fmt.Println("success!")
}

func run() error {
  return nil
}
```

Ensure that it works: `go run panic.go`

## Handling Exceptions in a Function

Introduce a bit of distubance and recover from it with `defer` and `recover`.

```patch
 func run() error {
+ defer func() {
+   if excp := recover(); excp != nil {
+     fmt.Printf("catch: `%+v`\n", excp)
+   }
+ }()
+
+ fmt.Println("Panicking!")
+ panic("boom!")
+
  return nil
 }
```

The program will output:

```console
Panicking!
catch: `boom!`
success
```


This approach handles unexpected exceptions, but existed with success. However, it's desirable to process errors and exceptions in the `main` function in the same way, indicating the presence of an `error`.

Using the rule from **"Defer, Panic, and Recover"**[^1]: `3. Deferred functions may read and assign to the returning function’s named return values.`. 

```patch
 import (
+ "errors"
  "fmt"
  "os"
 )

...

-func run() error {
+func run() (err error) {
  defer func() {
    if excp := recover(); excp != nil {
      fmt.Printf("catch: `%+v`\n", excp)
+     switch v := excp.(type) {
+     case string:
+       err = errors.New(v)
+     case error:
+       err = v
+     default:
+       err = errors.New(fmt.Sprint(v))
+     }
+   }
  }()
 
  fmt.Println("Panicking!")
  panic("boom!")
 
- return nil
+ return
 }
```

> make sure change the function signature to has a named return value `err error` and return without a value.

The `recover()` function returns an `interface{}` object. If there's an exception that isn't an `error` type, it initializes `error` object. Once you have the correct type, assign to `err`  instance.

This modified program will output:

```console
Panicking!
catch: `boom!`
error: `boom!`
exit status 1
```

The exit code now indicates that the program finished with error code. As an experiment, you can change the panic argument to other types such as `error` or `int`.

## Handling Exceptions in a Goroutine

Wrap the panic statement with the `go` statement and give it some time to allow the scheduler to pick up the goroutine:

```diff
import (
  "errors"
  "fmt"
  "os"
+ "time"
 )
 
 ...
 
- fmt.Println("Panicking!")
- panic("boom!")
+ blocker := make(chan struct{})
+ go func() {
+   fmt.Println("Panicking!")
+   panic("boom!")
+   blocker <- struct{}{}
+ }()
+
+ select {
+ case <-blocker:
+   panic("this branch should not happen")
+ case <-time.After(time.Second):
+   fmt.Printf("goroutine timeouted! err = `%+v`\n", err)
+ }
 
  return
 }
```

In this case, the panic wasn't caught in the child goroutine:

```console
Panicking!
panic: boom!

goroutine 34 [running]:
main.run.func2()
	.../panic.go:124 +0x68
created by main.run in goroutine 1
	.../panic.go:122 +0xa4
exit status 2
```

As a solution, you can duplicate the recovery inside the goroutine function:


```diff
  blocker := make(chan struct{})
  go func() {
+   defer func() {
+     if excp := recover(); excp != nil {
+       fmt.Printf("catch in goroutine: `%+v`\n", excp)
+       switch v := excp.(type) {
+       case string:
+         err = errors.New(v)
+       case error:
+         err = v
+       default:
+         err = errors.New(fmt.Sprint(v))
+       }
+     }
+   }()
+
    fmt.Println("Panicking!")
    panic("boom!")
    blocker <- struct{}{}
```

Output:

```console
Panicking!
catch in goroutine: `boom!`
goroutine timeouted! err = `boom!`
error: `boom!`
exit status 1
```

The modified function `run` would be as follows:

```golang
func run() (err error) {
	blocker := make(chan struct{})
	go func() {
		defer func() {
			if excp := recover(); excp != nil {
				fmt.Printf("catch in goroutine: `%+v`\n", excp)
				switch v := excp.(type) {
				case string:
					err = errors.New(v)
				case error:
					err = v
				default:
					err = errors.New(fmt.Sprint(v))
				}
			}
		}()

		fmt.Println("Panicking!")
		panic("boom!")
		blocker <- struct{}{}
	}()

	select {
	case <-blocker:
		panic("this branch should not happen")
	case <-time.After(time.Second):
		fmt.Printf("goroutine timeouted! err = `%+v`\n", err)
	}

	return
}
```

While the resulting code may not be optimal, propagating exceptions for improved processing is crucial. This article explores various Go language tricks, to enhance error handling and exception management in your programs.

I've uploaded the video, along with my clarification, to Vimeo[^2].

[![thumbnail](/assets/2024-01-23-converting-panics-to-errors-in-go-applications-2c65-ub1owhez2z1kpe4yw9yj.png)](https://vimeo.com/906475629)

## References

[^1]: [The Go Blog: Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)
[^2]: [Video: Walkthrough Converting Panics to Errors in Go Applications](https://vimeo.com/906475629)


