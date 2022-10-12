+++
date = "2021-05-17T12:30:48+08:00"
title = "golang errors (after 1.13)"
description = "golang errors (after 1.13)"
tags = ["golang", "errors"]
topics = ["golang", "errors"]
slug = "go-errors"
+++

Since go 1.13, go has gone one step further in terms of error handling. Errors can be wrapped, bubbled up and act upon.

We will here take a step back on error handling in go, from the perspective of a library.

## What data _could_ we have in the error?

Inspired by a [rust talk](https://www.youtube.com/watch?v=rAF8mLI0naQ) from [Jan Lusby](https://github.com/yaahc) about errors, here are a few things that error could contain:

- The reason of the error
- Context of the error: struct field, struct surrounding values...
- Stack trace: which part of the code triggered this error
- How to fix the error: suggest a way to fix it

## Where are errors displayed?

- In the logs of an application
- In front of the end user

## Ways to define errors after golang 1.13

### a) inline, with no type definition

```golang
 return fmt.Errorf("bad int %d", i)
```

(+)

- simple to write
- no runtime penalty

(-)

- impossible to handle properly as you can't match and react accordingly

### b) sentinel errors

```golang
var errBadInt = errors.New("bad int")
return fmt.Errorf("%w %d", i)
```

(+)

- match on error possible

(-)

- runtime (init) penalty
- binary size penalty
- global vars (fragile)

### c) structs, with defined context and string formatting

```golang

type errBadInt struct {
   i int
}

func (e *errBadInt) Error() string {
	return fmt.Errorf("bad int: %d", i)
}

func newErrBadInt(i int) *errBadInt {
    return &errBadInt{i}
}
```

## More tricks used by libs out there

### [upspin](https://github.com/upspin/upspin/blob/master/errors/errors.go) by Rob Pike himself

- smart use of const and redefine the Error struct with context
- return stack in debug mode

### [big cache](https://github.com/allegro/bigcache/blob/16762172f2ee433caf5bae1904cdbc11dcebffeb/iterator.go)

- interesting usage of const + error interface to avoid global vars

## So what should we do?

### For simple errors

Keep the errors as they are, but turn them into const thanks to the bigcache trick.
It keeps the nice opportunity to use errors.Is and removes the initialization cost.
No wrapping when there is no additional context.
Context is not stack trace.
Externally available errors should be handling all errors, unwrap them and repackage them again.

```golang
type marshalError string

func (me marshalError) Error() string {
	return string(me)
}

const ErrBadInt = marshalError("bad int")

func testErr() error {
	return fmt.Errorf("%w", ErrBadInt)
}
```

### For more complex errors

```golang
type badValueError struct {
key   string
value interface{}
}

func (bv *badValueError) Error() string {
	return fmt.Sprintf("bad value %v for key %s", bv.value, bv.key)
}

func testErr() error {
	v := 3
	k := "test"

	return badValueError{key: k, value: v}
}
```

## Handling errors

### Error as values

[testable in the playground](https://play.golang.org/p/gzCFQLfaGdL)

```golang
package main

import (
	"errors"
	"fmt"
	"log"
)

type marshalError string

func (me marshalError) Error() string {
	return string(me)
}

const errBadInt = marshalError("bad int")

func testErr() error {
	return fmt.Errorf("%w", errBadInt)
}

func main() {
	err := testErr()

	if errors.Is(err, errBadInt) {
		log.Printf("%v", err)
	} else {
		log.Println("wrong")
	}

	var e marshalError

	if errors.As(err, &e) {
		log.Printf("%v", err)
	} else {
		log.Println("wrong")
	}
}
```

### Error as struct

[on the playground](https://play.golang.org/p/XE7L2KWf3Au)

```golang
package main

import (
	"errors"
	"fmt"
	"log"
)

type badValueError struct {
	key   string
	value interface{}
}

func (bv *badValueError) Error() string {
	return fmt.Sprintf("bad value %v for key %s", bv.value, bv.key)
}

func testErr() error {
	v := 3
	k := "test"

		return &badValueError{key: k, value: v}
}

func main() {
	err := testErr()

	var e *badValueError

	if errors.As(err, &e) {
		log.Printf("%v", err)
	} else {
		log.Println("wrong")
	}
}
```

## Further thoughts

### Panic or not panic?

I tend to agree with [uber recommendation](https://github.com/uber-go/guide/blob/master/style.md) to not panic in a library, especially when you're expected to most likely handle user data. It allows the user of the lib to do whatever they want with the returned error.

That said, it's important to actually notify the library user if the library is used improperly in cases that are supposedly "not possible" by panicking.
The lib user is always able to handle these panics through a recover.

### Wrapping or not wrapping?

from [go1.13 post on errors](https://blog.golang.org/go1.13-errors#TOC_3.4.):

    "wrapping an error makes that error part of your API. If you don't want to commit to supporting that error as part of your API in the future, you shouldn't wrap the error."

It also makes no sense to wrap unexported errors to the external user of the API, as the user has no way of testing/matching the error.
External methods hence should not return internal errors and return their own exported type, catered at the lib user.

[See what wrapping allow and not allow you to do](https://go.dev/play/p/eMVNaeGV5QM):

```golang
package main

import (
	"errors"
	"fmt"
)

type AlphaError struct {
	err   error
	ratio int
}

func (e *AlphaError) Error() string {
	return fmt.Sprintf("%v %d", e.err, e.ratio)
}

type BetaError struct {
	err error
	key string
}

func (e *BetaError) Error() string {
	return fmt.Sprintf("%v %s", e.err, e.key)
}

func (e *BetaError) Unwrap() error {
	return e.err
}

type GammaError struct {
	name string
}

func (e *GammaError) Error() string {
	return fmt.Sprintf("%s", e.name)
}

func main() {
	err1 := &AlphaError{err: &GammaError{name: "pollux"}, ratio: 1}
	err2 := &BetaError{err: &GammaError{name: "pollux"}, key: "123"}

	var (
		alphaError *AlphaError
		betaError  *BetaError
		gammaError *GammaError
	)

	if errors.As(err1, &alphaError) { // true
		fmt.Println("alpha is alpha")
	}
	if errors.As(err1, &gammaError) { // false
		fmt.Println("alpha is gamma")
	}

	if errors.As(err2, &betaError) { // true
		fmt.Println("beta is beta")
	}
	if errors.As(err2, &gammaError) { // true
		fmt.Println("beta is gamma")
	}
}
```

### Never inline errors?

Simple cases don't require to define sentinel values and a simple string is often enough to start with.
If later on, you need to match on the error value or type, you can change this to an actual sentinel value.

That said, I think any error part of your API should be typed, as you want the lib user to be able to handle it according to its need.
