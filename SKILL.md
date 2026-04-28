---
name: uber-go-guide
description: Executable Go coding standard extracted from Uber Go Style Guide for automated code review and refactoring.
license: MIT
compatibility: Golang
metadata:
  author: kaiiyvwu
  version: "1.0"
  source: "https://github.com/uber-go/guide"
  generatedBy: "1.0.0"
---

# Skill: go-style-uber

## Description

Executable Go coding rules extracted from the Uber Go Style Guide for automated code review, static analysis, and AI-assisted refactoring.

---

## Rule Categories

- [1. Error Handling](#1-error-handling)
- [2. Interface Design](#2-interface-design)
- [3. Concurrency](#3-concurrency)
- [4. Memory Safety](#4-memory-safety)
- [5. API Design](#5-api-design)
- [6. Code Style](#6-code-style)
- [7. Testing](#7-testing)
- [8. Performance](#8-performance)
- [9. Structural Patterns](#9-structural-patterns)
- [Meta Rules](#meta-rules)

---

## 1. Error Handling

### RULE-ERR-001: Return errors instead of panicking

- **ID**: RULE-ERR-001
- **Name**: No panic for error handling
- **Description**: Production code MUST return errors via function return values. Panics MUST only be used for truly irrecoverable conditions (e.g., nil dereference). Panic/recover MUST NOT be used as an error handling strategy.
- **Bad**: `panic("an argument is required")`
- **Good**: `return errors.New("an argument is required")`
- **Fix Strategy**: Replace `panic(msg)` with `return errors.New(msg)` or `return fmt.Errorf(...)` and propagate the error up the call stack.

### RULE-ERR-002: Use `t.Fatal` over panic in tests

- **ID**: RULE-ERR-002
- **Name**: No panic in test setup
- **Description**: Test code MUST use `t.Fatal` or `t.FailNow` instead of `panic` to signal test failures.
- **Bad**: `panic("failed to set up test")`
- **Good**: `t.Fatal("failed to set up test")`
- **Fix Strategy**: Replace `panic(msg)` with `t.Fatal(msg)` inside test functions.

### RULE-ERR-003: Handle errors only once

- **ID**: RULE-ERR-003
- **Name**: Single error handling per error
- **Description**: Each error MUST be handled exactly once. Code MUST NOT both log an error and return it, as this causes duplicate noise in logs.
- **Bad**: `log.Printf(..., err); return err`
- **Good**: `return fmt.Errorf("context: %w", err)`
- **Fix Strategy**: Remove the logging and wrap the error with context using `fmt.Errorf` before returning; OR log the error and degrade gracefully without returning it.

### RULE-ERR-004: Wrap errors with context using `%w`

- **ID**: RULE-ERR-004
- **Name**: Error wrapping with context
- **Description**: When propagating errors with added context, MUST use `fmt.Errorf` with `%w` verb to preserve the error chain for `errors.Is`/`errors.As` matching. Use `%v` only when intentionally obfuscating the underlying error.
- **Bad**: `return fmt.Errorf("failed to create store: %v", err)`
- **Good**: `return fmt.Errorf("new store: %w", err)`
- **Fix Strategy**: Replace `%v` with `%w` in `fmt.Errorf` calls that wrap errors, unless the caller must not access the underlying error.

### RULE-ERR-005: Keep error context messages succinct

- **ID**: RULE-ERR-005
- **Name**: No redundant error prefixes
- **Description**: Error context messages MUST NOT include redundant phrases such as "failed to". Use the operation name directly.
- **Bad**: `fmt.Errorf("failed to create new store: %w", err)`
- **Good**: `fmt.Errorf("new store: %w", err)`
- **Fix Strategy**: Remove "failed to" or similar redundant prefixes from error wrapping messages.

### RULE-ERR-006: Use sentinel errors for matchable static errors

- **ID**: RULE-ERR-006
- **Name**: Sentinel error variables for matching
- **Description**: When callers MUST match an error with `errors.Is`, the error MUST be declared as a package-level `var` using `errors.New`. When the error message is dynamic and callers need to match it, a custom error type MUST be used.
- **Bad**: `return errors.New("could not open")` (inline, unexported)
- **Good**: `var ErrCouldNotOpen = errors.New("could not open")` and return `ErrCouldNotOpen`
- **Fix Strategy**: Extract inline `errors.New` calls to package-level variables when the error needs to be matchable by callers.

### RULE-ERR-007: Name error values with `Err` prefix, error types with `Error` suffix

- **ID**: RULE-ERR-007
- **Name**: Error naming convention
- **Description**: Error sentinel variables MUST use `Err` prefix (exported) or `err` prefix (unexported). Custom error types MUST use `Error` suffix.
- **Bad**: `var BrokenLink = errors.New(...)`
- **Bad**: `type NotFound struct { ... }`
- **Good**: `var ErrBrokenLink = errors.New(...)`
- **Good**: `type NotFoundError struct { ... }`
- **Fix Strategy**: Rename error variables to `Err`/`err` prefix and error types to `Error` suffix.

### RULE-ERR-008: Restrict `os.Exit`/`log.Fatal` to `main()` only

- **ID**: RULE-ERR-008
- **Name**: Exit only in main
- **Description**: `os.Exit` and `log.Fatal*` MUST be called only in `main()`. All other functions MUST return errors to signal failure.
- **Bad**: `func readFile(p string) string { ...; log.Fatal(err) }`
- **Good**: `func readFile(p string) (string, error) { ...; return "", err }`
- **Fix Strategy**: Change function signature to return an error; move `log.Fatal`/`os.Exit` to `main()`.

### RULE-ERR-009: Exit at most once in `main()`

- **ID**: RULE-ERR-009
- **Name**: Single exit point in main
- **Description**: `os.Exit` or `log.Fatal` MUST be called at most once in `main()`. All program logic MUST be extracted into a `run()` or similar function that returns an error.
- **Bad**: Multiple `log.Fatal()` calls scattered in `main()`
- **Good**: `func main() { if err := run(); err != nil { log.Fatal(err) } }`
- **Fix Strategy**: Extract business logic into a separate function; call `log.Fatal` or `os.Exit` once based on its return value.

### RULE-ERR-010: Handle type assertion failures with comma-ok

- **ID**: RULE-ERR-010
- **Name**: Safe type assertions
- **Description**: Type assertions MUST use the comma-ok idiom. The single-return form `i.(T)` MUST NOT be used.
- **Bad**: `t := i.(string)`
- **Good**: `t, ok := i.(string); if !ok { /* handle */ }`
- **Fix Strategy**: Replace `x := i.(T)` with `x, ok := i.(T)` and add error handling for the failure case.

---

## 2. Interface Design

### RULE-IFACE-001: Do not use pointers to interfaces

- **ID**: RULE-IFACE-001
- **Name**: No pointer to interface
- **Description**: Pointers to interfaces MUST NOT be used. Interfaces MUST be passed as values; the underlying data can still be a pointer.
- **Bad**: `func Foo(i *interface{})`
- **Good**: `func Foo(i interface{})`
- **Fix Strategy**: Remove the `*` from the interface parameter type.

### RULE-IFACE-002: Verify interface compliance at compile time

- **ID**: RULE-IFACE-002
- **Name**: Compile-time interface check
- **Description**: Exported types that implement specific interfaces MUST include a compile-time assertion: `var _ Interface = (*Type)(nil)` or `var _ Interface = Type{}`.
- **Bad**: `type Handler struct{}; func (h *Handler) ServeHTTP(...) {}` (no assertion)
- **Good**: `var _ http.Handler = (*Handler)(nil)`
- **Fix Strategy**: Add `var _ Interface = (*Type)(nil)` immediately after the type declaration.

### RULE-IFACE-003: Do not embed types in public structs

- **ID**: RULE-IFACE-003
- **Name**: No type embedding in public structs
- **Description**: Public structs MUST NOT embed other types (structs or interfaces). Use a named field and delegate methods explicitly. This applies to both struct and interface embedding.
- **Bad**: `type ConcreteList struct { *AbstractList }`
- **Good**: `type ConcreteList struct { list *AbstractList }` with explicit delegate methods
- **Fix Strategy**: Replace the embedded type with a named unexported field; write explicit delegate methods for each required method.

### RULE-IFACE-004: Interfaces should be defined by the consumer

- **ID**: RULE-IFACE-004
- **Name**: Consumer-defined interfaces
- **Description**: Interfaces MUST be defined in the package that consumes them, not in the package that implements them. Keep interfaces small and focused.
- **Fix Strategy**: Move interface definitions to the consuming package; keep the implementing package concrete.

---

## 3. Concurrency

### RULE-CONC-001: Use `go.uber.org/atomic` for atomic operations

- **ID**: RULE-CONC-001
- **Name**: Type-safe atomics
- **Description**: Atomic operations MUST use `go.uber.org/atomic` types instead of raw `sync/atomic` functions operating on primitive types. Direct reads of fields intended to be atomic MUST NOT occur.
- **Bad**: `type foo struct { running int32 }; f.running == 1`
- **Good**: `type foo struct { running atomic.Bool }; f.running.Load()`
- **Fix Strategy**: Replace `int32`/`int64` atomic fields with `atomic.Int32`/`atomic.Int64`/`atomic.Bool` types; replace direct reads with `.Load()` calls.

### RULE-CONC-002: Channel size MUST be zero or one

- **ID**: RULE-CONC-002
- **Name**: Buffered channel limit
- **Description**: Channels MUST have a buffer size of 0 (unbuffered) or 1. Any other size MUST be justified with documented reasoning about backpressure behavior.
- **Bad**: `c := make(chan int, 64)`
- **Good**: `c := make(chan int, 1)` or `c := make(chan int)`
- **Fix Strategy**: Change buffer size to 0 or 1; if larger buffer is required, document the justification.

### RULE-CONC-003: Every goroutine MUST have a controlled lifetime

- **ID**: RULE-CONC-003
- **Name**: No fire-and-forget goroutines
- **Description**: Every goroutine MUST have a predictable termination condition or a mechanism to signal it to stop. There MUST be a way to wait for goroutine completion.
- **Bad**: `go func() { for { flush(); time.Sleep(d) } }()`
- **Good**: Use `stop`/`done` channels or `sync.WaitGroup` with a cancel mechanism
- **Fix Strategy**: Add a stop channel; use `select` with the stop channel; provide a `Close()`/`Stop()` method to signal shutdown and wait for completion.

### RULE-CONC-004: Wait for goroutines to exit

- **ID**: RULE-CONC-004
- **Name**: Goroutine lifecycle tracking
- **Description**: Code that spawns goroutines MUST wait for them to exit before returning. Use `sync.WaitGroup` for multiple goroutines or `chan struct{}` for single goroutines.
- **Bad**: Spawning goroutines without tracking completion
- **Good**: `var wg sync.WaitGroup; wg.Add(1); go func() { defer wg.Done(); ... }(); wg.Wait()`
- **Fix Strategy**: Add `sync.WaitGroup` or done channel to track and wait for goroutine completion.

### RULE-CONC-005: No goroutines in `init()`

- **ID**: RULE-CONC-005
- **Name**: No goroutine in init
- **Description**: `init()` functions MUST NOT spawn goroutines. Background goroutines MUST be managed by an explicit constructor that returns an object with a `Close()`/`Stop()` method.
- **Bad**: `func init() { go doWork() }`
- **Good**: `func NewWorker() *Worker { go w.doWork(); return w }` with `w.Shutdown()`
- **Fix Strategy**: Move goroutine spawning to a constructor; add a shutdown method that signals and waits for the goroutine to stop.

### RULE-CONC-006: Use defer for mutex unlocks

- **ID**: RULE-CONC-006
- **Name**: Defer mutex cleanup
- **Description**: `Mutex.Unlock()` and `RWMutex.Unlock()` MUST use `defer` immediately after the `Lock()` call. Manual unlock at multiple return points is prohibited.
- **Bad**: `p.Lock(); ...; p.Unlock(); return x`
- **Good**: `p.Lock(); defer p.Unlock()`
- **Fix Strategy**: Add `defer p.Unlock()` immediately after `p.Lock()` and remove all manual `Unlock()` calls.

### RULE-CONC-007: Do not embed mutexes in structs

- **ID**: RULE-CONC-007
- **Name**: No mutex embedding
- **Description**: `sync.Mutex` and `sync.RWMutex` MUST NOT be embedded in structs. They MUST be declared as named non-pointer fields.
- **Bad**: `type SMap struct { sync.Mutex; data map[string]string }`
- **Good**: `type SMap struct { mu sync.Mutex; data map[string]string }`
- **Fix Strategy**: Replace the embedded `sync.Mutex` with a named field `mu sync.Mutex`; update all `Lock()`/`Unlock()` calls to use `mu.Lock()`/`mu.Unlock()`.

### RULE-CONC-008: Use zero-value mutexes

- **ID**: RULE-CONC-008
- **Name**: Zero-value mutex, no pointer
- **Description**: Mutex fields MUST use the zero-value form (`var mu sync.Mutex`), not `new(sync.Mutex)` or pointer types.
- **Bad**: `mu := new(sync.Mutex)`
- **Good**: `var mu sync.Mutex`
- **Fix Strategy**: Replace `new(sync.Mutex)` with `var mu sync.Mutex`; remove pointer indirection.

---

## 4. Memory Safety

### RULE-MEM-001: Copy slices and maps at struct boundaries (receiving)

- **ID**: RULE-MEM-001
- **Name**: Defensive copy on receive
- **Description**: When storing a slice or map received as a function argument into a struct, a deep copy MUST be made. Direct assignment of external slices/maps to internal fields is prohibited.
- **Bad**: `func (d *Driver) SetTrips(trips []Trip) { d.trips = trips }`
- **Good**: `func (d *Driver) SetTrips(trips []Trip) { d.trips = make([]Trip, len(trips)); copy(d.trips, trips) }`
- **Fix Strategy**: Create a new slice/map with `make` and `copy` the incoming data into it before storing.

### RULE-MEM-002: Copy slices and maps at struct boundaries (returning)

- **ID**: RULE-MEM-002
- **Name**: Defensive copy on return
- **Description**: Methods returning internal slices or maps MUST return a copy, not a reference to the internal data. This prevents external mutation of internal state.
- **Bad**: `func (s *Stats) Snapshot() map[string]int { return s.counters }`
- **Good**: `result := make(map[string]int, len(s.counters)); for k, v := range s.counters { result[k] = v }; return result`
- **Fix Strategy**: Create a new container, copy internal data into it, and return the copy.

### RULE-MEM-003: Avoid mutable globals

- **ID**: RULE-MEM-003
- **Name**: No mutable package-level state
- **Description**: Package-level mutable variables (including function pointers) MUST NOT be used. Dependency injection MUST be used instead.
- **Bad**: `var _timeNow = time.Now`
- **Good**: `type signer struct { now func() time.Time }`
- **Fix Strategy**: Move mutable state into a struct field; inject dependencies via constructor or struct initialization.

### RULE-MEM-004: Avoid repeated string-to-byte conversions

- **ID**: RULE-MEM-004
- **Name**: Cache byte slice conversions
- **Description**: Fixed string-to-`[]byte` conversions MUST NOT be performed inside loops. The conversion MUST be done once outside the loop.
- **Bad**: `for { w.Write([]byte("Hello")) }`
- **Good**: `data := []byte("Hello"); for { w.Write(data) }`
- **Fix Strategy**: Move `[]byte(s)` conversion before the loop and assign to a variable.

---

## 5. API Design

### RULE-API-001: Use functional options for optional constructor parameters

- **ID**: RULE-API-001
- **Name**: Functional options pattern
- **Description**: Public constructors with three or more optional parameters MUST use the functional options pattern. An `Option` interface with an unexported `apply` method MUST be used.
- **Bad**: `func Open(addr string, cache bool, logger *zap.Logger) (*Conn, error)`
- **Good**: `func Open(addr string, opts ...Option) (*Conn, error)` with `WithCache`, `WithLogger`
- **Fix Strategy**: Extract optional parameters into an `Option` interface; create `WithX` helper functions; accept `...Option` variadic parameter.

### RULE-API-002: Use `time.Time` for time instants in APIs

- **ID**: RULE-API-002
- **Name**: time.Time for instants
- **Description**: API functions MUST use `time.Time` for time instants, not integers or strings. Time comparison MUST use `time.Time` methods (`Before`, `After`, `Equal`).
- **Bad**: `func isActive(now, start, stop int) bool`
- **Good**: `func isActive(now, start, stop time.Time) bool`
- **Fix Strategy**: Replace integer/string time parameters with `time.Time`; use `Before`/`After`/`Equal` for comparisons.

### RULE-API-003: Use `time.Duration` for time periods in APIs

- **ID**: RULE-API-003
- **Name**: time.Duration for periods
- **Description**: API functions MUST use `time.Duration` for time durations, not raw integers.
- **Bad**: `func poll(delay int)`
- **Good**: `func poll(delay time.Duration)`
- **Fix Strategy**: Replace integer duration parameters with `time.Duration`.

### RULE-API-004: Include time unit in field names when using numeric types

- **ID**: RULE-API-004
- **Name**: Unit suffix for time fields
- **Description**: When `time.Duration` cannot be used in external APIs (e.g., JSON), numeric time fields MUST include the unit in the field name.
- **Bad**: `Interval int \`json:"interval"\``
- **Good**: `IntervalMillis int \`json:"intervalMillis"\``
- **Fix Strategy**: Rename the field to include the unit suffix (e.g., `Millis`, `Seconds`).

### RULE-API-005: Use RFC 3339 for time strings

- **ID**: RULE-API-005
- **Name**: RFC 3339 time format
- **Description**: When `time.Time` cannot be used in external APIs, timestamps MUST use RFC 3339 string format via `time.RFC3339`.
- **Fix Strategy**: Use `time.Format(time.RFC3339)` and `time.Parse(time.RFC3339, s)` for string time representations.

### RULE-API-006: Use struct tags on marshaled fields

- **ID**: RULE-API-006
- **Name**: Struct tags for serialization
- **Description**: Struct fields that are serialized to JSON, YAML, or other tag-based formats MUST be annotated with the relevant struct tag.
- **Bad**: `type Stock struct { Price int; Name string }`
- **Good**: `type Stock struct { Price int \`json:"price"\`; Name string \`json:"name"\` }`
- **Fix Strategy**: Add appropriate struct tags (e.g., `json:"fieldname"`) to all serializable fields.

---

## 6. Code Style

### RULE-STYLE-001: Use MixedCaps for function names

- **ID**: RULE-STYLE-001
- **Name**: MixedCaps naming
- **Description**: Function names MUST use MixedCaps convention. Underscores in function names are prohibited except in test functions (`TestMyFunction_WhatIsBeingTested`).
- **Fix Strategy**: Rename functions to MixedCaps; use underscores only in test function names for grouping.

### RULE-STYLE-002: Package names must be lowercase, singular, and descriptive

- **ID**: RULE-STYLE-002
- **Name**: Package naming
- **Description**: Package names MUST be all lowercase with no capitals or underscores. Names MUST NOT be plural. Names such as "common", "util", "shared", or "lib" are prohibited.
- **Bad**: `package utils`, `package commonServices`
- **Good**: `package httputil`, `package user`
- **Fix Strategy**: Rename package to a short, descriptive, singular, lowercase name.

### RULE-STYLE-003: Do not shadow built-in identifiers

- **ID**: RULE-STYLE-003
- **Name**: No builtin name shadowing
- **Description**: Variables, fields, and parameters MUST NOT use names that shadow Go's predeclared identifiers (e.g., `error`, `string`, `int`, `true`, `false`, `nil`, `func`, `len`, `cap`, `copy`, etc.).
- **Bad**: `var error string; type Foo struct { error error; string string }`
- **Good**: `var errMsg string; type Foo struct { err error; str string }`
- **Fix Strategy**: Rename the variable/field to avoid collision with predeclared identifiers.

### RULE-STYLE-004: Prefix unexported globals with underscore

- **ID**: RULE-STYLE-004
- **Name**: Underscore prefix for unexported globals
- **Description**: Unexported package-level `var` and `const` declarations MUST be prefixed with `_`. Exception: unexported error values use `err` prefix without underscore.
- **Bad**: `const defaultPort = 8080`
- **Good**: `const _defaultPort = 8080`
- **Fix Strategy**: Add `_` prefix to unexported top-level variables and constants.

### RULE-STYLE-005: Group related declarations

- **ID**: RULE-STYLE-005
- **Name**: Declaration grouping
- **Description**: Related `const`, `var`, `type`, and `import` declarations MUST be grouped using parenthesized syntax. Unrelated declarations MUST NOT be grouped together.
- **Bad**: `const a = 1; const b = 2; const EnvVar = "X"` (all in one group)
- **Good**: `const (a = 1; b = 2)` separate from `const EnvVar = "X"`
- **Fix Strategy**: Group related declarations in parenthesized blocks; separate unrelated ones.

### RULE-STYLE-006: Group imports into two groups

- **ID**: RULE-STYLE-006
- **Name**: Import grouping
- **Description**: Imports MUST be grouped into two groups separated by a blank line: (1) standard library, (2) everything else (third-party).
- **Bad**: `import ( "fmt"; "go.uber.org/atomic"; "os" )`
- **Good**: `import ( "fmt"; "os"; \n "go.uber.org/atomic" )`
- **Fix Strategy**: Reorder imports: stdlib first, blank line, then third-party packages.

### RULE-STYLE-007: Use import aliases only when necessary

- **ID**: RULE-STYLE-007
- **Name**: Minimal import aliases
- **Description**: Import aliases MUST be used ONLY when the package name does not match the last element of the import path, or when there is a direct name conflict. Unnecessary aliases are prohibited.
- **Bad**: `import runtimetrace "runtime/trace"` (when no conflict exists)
- **Good**: `import "runtime/trace"` OR `import nettrace "golang.net/x/trace"` (conflict resolution)
- **Fix Strategy**: Remove unnecessary import aliases; keep aliases only for conflicts or mismatched package names.

### RULE-STYLE-008: Use field names when initializing structs

- **ID**: RULE-STYLE-008
- **Name**: Named struct initialization
- **Description**: Struct initialization MUST use field names. Positional initialization is prohibited. Exception: test table structs with 3 or fewer fields.
- **Bad**: `User{"John", "Doe", true}`
- **Good**: `User{FirstName: "John", LastName: "Doe", Admin: true}`
- **Fix Strategy**: Add field names to all struct literal initializations.

### RULE-STYLE-009: Omit zero-value fields in struct literals

- **ID**: RULE-STYLE-009
- **Name**: Omit zero-value struct fields
- **Description**: When initializing structs with named fields, fields set to their zero value MUST be omitted unless they provide meaningful context.
- **Bad**: `User{FirstName: "John", MiddleName: "", Admin: false}`
- **Good**: `User{FirstName: "John"}`
- **Fix Strategy**: Remove zero-value field entries from struct literals where they add no meaningful context.

### RULE-STYLE-010: Use `var` for zero-value struct declarations

- **ID**: RULE-STYLE-010
- **Name**: var form for zero structs
- **Description**: When declaring a struct with all zero-value fields, MUST use `var` form, not `T{}`.
- **Bad**: `user := User{}`
- **Good**: `var user User`
- **Fix Strategy**: Replace `name := T{}` with `var name T`.

### RULE-STYLE-011: Use `&T{}` instead of `new(T)`

- **ID**: RULE-STYLE-011
- **Name**: No new for structs
- **Description**: Struct pointer initialization MUST use `&T{}` instead of `new(T)`.
- **Bad**: `sptr := new(T); sptr.Name = "bar"`
- **Good**: `sptr := &T{Name: "bar"}`
- **Fix Strategy**: Replace `new(T)` with `&T{}` and initialize fields in the literal.

### RULE-STYLE-012: Declare format strings as `const`

- **ID**: RULE-STYLE-012
- **Name**: Const format strings
- **Description**: Format strings used with `Printf`-style functions MUST be declared as `const` when stored in variables.
- **Bad**: `msg := "unexpected values %v, %v\n"; fmt.Printf(msg, 1, 2)`
- **Good**: `const msg = "unexpected values %v, %v\n"; fmt.Printf(msg, 1, 2)`
- **Fix Strategy**: Change `var`/`:=` format string declarations to `const`.

### RULE-STYLE-013: Name Printf-style functions ending with `f`

- **ID**: RULE-STYLE-013
- **Name**: Printf-style function naming
- **Description**: Custom `Printf`-style functions MUST be named ending with `f` (e.g., `Wrapf`, not `Wrap`) to enable `go vet` format string checking.
- **Bad**: `func Wrap(msg string, args ...interface{})`
- **Good**: `func Wrapf(msg string, args ...interface{})`
- **Fix Strategy**: Rename the function to end with `f`.

### RULE-STYLE-014: Start enums at 1, not 0

- **ID**: RULE-STYLE-014
- **Name**: Non-zero enum start
- **Description**: Enumeration constants defined with `iota` MUST start at 1 (using `iota + 1`). The zero value MUST be reserved as the default/unset state unless the zero value is a valid, desired default.
- **Bad**: `const ( Add Operation = iota; Subtract; Multiply )`
- **Good**: `const ( Add Operation = iota + 1; Subtract; Multiply )`
- **Fix Strategy**: Change `iota` to `iota + 1` in the enum declaration.

### RULE-STYLE-015: Use `make()` for empty maps, map literals for fixed elements

- **ID**: RULE-STYLE-015
- **Name**: Map initialization style
- **Description**: Empty maps and programmatically populated maps MUST use `make()`. Maps with a fixed set of known elements MUST use map literals.
- **Bad**: `m := make(map[string]int, 3); m["a"]=1; m["b"]=2; m["c"]=3`
- **Good**: `m := map[string]int{"a": 1, "b": 2, "c": 3}`
- **Fix Strategy**: Convert sequential `make` + assignments to a map literal when elements are known at compile time.

### RULE-STYLE-016: Use `var` for empty slice declarations

- **ID**: RULE-STYLE-016
- **Name**: Nil slice declaration
- **Description**: Empty slices that will be populated later MUST be declared with `var s []T`, not `s := []T{}` or `s := make([]T, 0)`.
- **Bad**: `nums := []int{}`
- **Good**: `var nums []int`
- **Fix Strategy**: Replace `s := []T{}` with `var s []T`.

### RULE-STYLE-017: Use `len(s) == 0` to check for empty slices

- **ID**: RULE-STYLE-017
- **Name**: Len check for slices
- **Description**: To check if a slice is empty, MUST use `len(s) == 0`. MUST NOT compare `s == nil` for emptiness checking.
- **Bad**: `return s == nil`
- **Good**: `return len(s) == 0`
- **Fix Strategy**: Replace `s == nil` emptiness checks with `len(s) == 0`.

### RULE-STYLE-018: Return `nil` for empty slices

- **ID**: RULE-STYLE-018
- **Name**: Return nil for empty slices
- **Description**: Functions returning empty slices MUST return `nil`, not `[]T{}`.
- **Bad**: `return []int{}`
- **Good**: `return nil`
- **Fix Strategy**: Replace `return []T{}` with `return nil`.

### RULE-STYLE-019: Use `strconv` over `fmt` for primitive conversions

- **ID**: RULE-STYLE-019
- **Name**: Strconv for primitives
- **Description**: Conversions between primitives and strings MUST use `strconv` (e.g., `strconv.Itoa`, `strconv.FormatInt`), not `fmt.Sprintf` or `fmt.Sprint`.
- **Bad**: `s := fmt.Sprint(i)`
- **Good**: `s := strconv.Itoa(i)`
- **Fix Strategy**: Replace `fmt.Sprint`/`fmt.Sprintf("%d", ...)` with `strconv.Itoa`/`strconv.FormatInt`.

### RULE-STYLE-020: Use raw string literals to avoid escaping

- **ID**: RULE-STYLE-020
- **Name**: Raw string literals
- **Description**: Strings containing quotes, backslashes, or special characters MUST use raw string literals (backticks) to avoid escape sequences.
- **Bad**: `"unknown name:\"test\""`
- **Good**: `` `unknown error:"test"` ``
- **Fix Strategy**: Replace double-quoted strings with backtick strings when they contain escaped characters.

### RULE-STYLE-021: Avoid naked parameters in function calls

- **ID**: RULE-STYLE-021
- **Name**: Comment naked parameters
- **Description**: Function calls with non-obvious parameter values (especially `bool`) MUST include C-style comments (`/* ... */`) indicating the parameter name.
- **Bad**: `printInfo("foo", true, true)`
- **Good**: `printInfo("foo", true /* isLocal */, true /* done */)`
- **Fix Strategy**: Add `/* paramName */` comments after each non-obvious argument.

### RULE-STYLE-022: Reduce nesting by handling errors first

- **ID**: RULE-STYLE-022
- **Name**: Early return for errors
- **Description**: Code MUST handle error cases and special conditions first, returning early or continuing the loop, to reduce nesting depth.
- **Bad**: `if ok { if err == nil { ... } else { return err } } else { log(...) }`
- **Good**: `if !ok { log(...); continue }; if err != nil { return err }; ...`
- **Fix Strategy**: Invert conditions; handle error/edge cases first with early return or continue.

### RULE-STYLE-023: Eliminate unnecessary else branches

- **ID**: RULE-STYLE-023
- **Name**: No unnecessary else
- **Description**: When a variable is set in both branches of an `if/else`, the else branch MUST be eliminated by using an early return or default value assignment.
- **Bad**: `var a int; if b { a = 100 } else { a = 10 }`
- **Good**: `a := 10; if b { a = 100 }`
- **Fix Strategy**: Initialize with the default value; use `if` without `else` to override.

### RULE-STYLE-024: Use `:=` for local variable declarations with explicit values

- **ID**: RULE-STYLE-024
- **Name**: Short variable declaration
- **Description**: Local variables set to explicit values MUST use short declaration `:=`. The `var` keyword MUST be used only for zero-value declarations.
- **Bad**: `var s = "foo"`
- **Good**: `s := "foo"`
- **Fix Strategy**: Replace `var x = value` with `x := value`.

### RULE-STYLE-025: Reduce variable scope

- **ID**: RULE-STYLE-025
- **Name**: Minimal variable scope
- **Description**: Variables and constants MUST be declared in the narrowest possible scope. This MUST NOT conflict with the reduce-nesting rule.
- **Bad**: `err := os.WriteFile(...); if err != nil { return err }`
- **Good**: `if err := os.WriteFile(...); err != nil { return err }`
- **Fix Strategy**: Move variable declarations into `if` init statements when the variable is only used within the `if` block.

### RULE-STYLE-026: Top-level var declarations must omit type when redundant

- **ID**: RULE-STYLE-026
- **Name**: Omit redundant type in top-level var
- **Description**: At the top level, `var` declarations MUST NOT specify the type when the expression's type matches the desired type. Type MUST be specified only when different.
- **Bad**: `var _s string = F()` (where F returns string)
- **Good**: `var _s = F()`
- **Fix Strategy**: Remove redundant type annotation from top-level var declarations.

### RULE-STYLE-027: Place embedded types at the top of struct fields

- **ID**: RULE-STYLE-027
- **Name**: Embedded fields first
- **Description**: When embedding types in structs (where embedding is justified), embedded fields MUST appear at the top of the field list, separated from regular fields by a blank line.
- **Bad**: `type Client struct { version int; http.Client }`
- **Good**: `type Client struct { http.Client; \n version int }`
- **Fix Strategy**: Move embedded type declarations to the top of the struct field list; add a blank line separator.

### RULE-STYLE-028: Order functions by call order and group by receiver

- **ID**: RULE-STYLE-028
- **Name**: Function ordering
- **Description**: In a file, definitions MUST appear in order: type definitions, then `newXYZ()`/`NewXYZ()`, then methods grouped by receiver, then plain utility functions. Exported functions appear before unexported.
- **Fix Strategy**: Reorder file contents: structs first, constructors next, methods grouped by receiver, then utility functions at the end.

### RULE-STYLE-029: Soft line length limit of 99 characters

- **ID**: RULE-STYLE-029
- **Name**: Line length limit
- **Description**: Lines of code SHOULD aim for a maximum of 99 characters. Code is allowed to exceed this limit when wrapping would harm readability.
- **Fix Strategy**: Wrap long lines at logical boundaries (after operators, before arguments).

---

## 7. Testing

### RULE-TEST-001: Use table-driven tests for repetitive test logic

- **ID**: RULE-TEST-001
- **Name**: Table-driven tests
- **Description**: Tests that verify the same logic across multiple inputs/outputs MUST use table-driven tests. Test case slices MUST be named `tests` and each case `tt`. Input fields MUST use `give` prefix; output fields MUST use `want` prefix.
- **Fix Strategy**: Convert repetitive test cases to a `[]struct{...}` table with `t.Run` subtests.

### RULE-TEST-002: Avoid complex conditional logic in table tests

- **ID**: RULE-TEST-002
- **Name**: Simple table tests
- **Description**: Table tests MUST NOT contain complex conditional logic, branching mock setup, or function fields. Complex tests MUST be split into separate `Test...` functions. All table fields MUST be used in all test cases.
- **Bad**: Table with `shouldCallX`, `shouldCallY`, `giveXErr` fields and `if` branching in subtests
- **Good**: Separate `TestShouldCallX` and `TestShouldCallY` functions
- **Fix Strategy**: Split complex table tests into multiple focused test functions or simpler tables.

### RULE-TEST-003: Capture loop variables in parallel tests

- **ID**: RULE-TEST-003
- **Name**: Parallel test loop variable capture
- **Description**: When using `t.Parallel()` inside subtests, the loop variable MUST be explicitly captured within the loop scope to prevent data races.
- **Fix Strategy**: Use `tt := tt` inside the for loop body when `t.Parallel()` is called.

---

## 8. Performance

### RULE-PERF-001: Specify container capacity hints

- **ID**: RULE-PERF-001
- **Name**: Container capacity hints
- **Description**: Slices and maps MUST specify capacity hints in `make()` when the size is known or can be estimated at initialization time.
- **Bad**: `make([]int, 0)` followed by known number of appends
- **Good**: `make([]int, 0, size)` or `make(map[K]V, hint)`
- **Fix Strategy**: Add capacity argument to `make()` calls when size is predictable.

### RULE-PERF-002: Use `strconv` over `fmt` for primitive-to-string

- **ID**: RULE-PERF-002
- **Name**: Fast primitive conversion
- **Description**: In hot paths, primitive-to-string conversions MUST use `strconv` instead of `fmt.Sprintf`. (Cross-references RULE-STYLE-019.)
- **Fix Strategy**: Replace `fmt.Sprint`/`fmt.Sprintf` with `strconv.Itoa`/`strconv.FormatFloat` etc.

---

## 9. Structural Patterns

### RULE-STRUCT-001: Avoid `init()` functions

- **ID**: RULE-STRUCT-001
- **Name**: No init functions
- **Description**: `init()` MUST be avoided. When unavoidable, `init()` MUST be: deterministic, independent of other `init()` ordering, free of global/environment state access, and free of I/O.
- **Bad**: `func init() { raw, _ := os.ReadFile("config.yaml"); yaml.Unmarshal(raw, &_config) }`
- **Good**: `func loadConfig() Config { raw, err := os.ReadFile(...); ...; return config }`
- **Fix Strategy**: Move init logic to an explicit function called from `main()` or a constructor.

### RULE-STRUCT-002: Use defer for resource cleanup

- **ID**: RULE-STRUCT-002
- **Name**: Defer for cleanup
- **Description**: Resource cleanup (file closes, lock releases, response body closes) MUST use `defer` immediately after resource acquisition.
- **Fix Strategy**: Add `defer resource.Close()` / `defer mutex.Unlock()` immediately after acquisition.

---

## Meta Rules

These principles represent the core design philosophy underlying all rules:

1. **Explicit over implicit**: Every behavior should be visible in code. No hidden side effects, no magic.
2. **Control resource lifetimes**: Every resource (goroutine, file, lock, connection) must have a clear owner and a deterministic cleanup path.
3. **Avoid shared mutable state**: Package-level mutable state is prohibited. State must be injected and scoped.
4. **Errors are values**: Errors must be returned, not panicked. They must be handled exactly once and wrapped with context.
5. **Interfaces belong to consumers**: Define interfaces where they are used, not where they are implemented. Keep them minimal.
6. **Consistency above all**: Within a package, a single consistent style must be applied. Inconsistency is worse than any specific convention.
7. **Reduce cognitive load**: Minimize nesting, reduce scope, handle errors early. The happy path should read linearly.
8. **Defensive at boundaries**: Copy data at API boundaries. Do not expose internal state through return values or shared references.
9. **Zero values must be useful**: Types should be designed so their zero value is immediately usable without initialization.
10. **Test readability matters**: Tests are documentation. Table tests must be simple, focused, and free of branching logic.

---