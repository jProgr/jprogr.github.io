---
title: "Go performance recipes"
permalink: /go-performance-recipes
custom_date: "240308"
updated_date: "240420"
---

# Go performance recipes

This is a list of recipes to improve the performance of Go code that I have used or encountered whilst working on projects. I keep this list updated.

Contents:
- [Recipes](#recipes).
    - [Use `strings.Builder` to concatenate many strings (3h9)](#recipe-3h9).
    - [Set the size of the backing array on `strings.Builder` (a85)](#recipe-a85).
    - [Use `reflect.TypeOf` instead of the `fmt` package for a string representation of a type (71t)](#recipe-71t).
    - [Use `sync.Pool` for big and reusable objects to reduce GC work (us4)](#recipe-us4).
    - [Try a buffered channel instead of `sync.Pool` (7ya)](#recipe-7ya).
    - [Replace defer calls for normal ones (x9i)](#recipe-x9i).
- [General performance improvements](#general-performance-improvements).
    - [Specify length or capacity on slices (shu)](#recipe-shu).

Not every recipe will apply to every codebase nor you should try to use every single one of them. First you should measure, measure, measure. Pinpoint where is the performance bottleneck and then use any of these recipes. Remember to measure, measure, measure before and after changes.

In general, the recipes are listed in order of impact; how much time they save:

Impact on performance:
- High.
- Low.
- Slim.

The impact should not be taken verbatim. The actual impact may vary wildly, it all depends on what is the code actually doing on what hardware. The impact is solely a guideline on which recipes to implement first. Implementation is the second aspect of these recipes: some are easy to implement, a change in a line somewhere, others require full rewrites of vast parts of the logic.

Implementation difficulty:
- Trivial.
- Easy.
- Difficult.

Head straight for the recipes that have a high impact and are trivial, easy to implement. If you still need more performance, prove it. Then you may reach for the low impact. After measuring, and only after running out of options, you may reach for the difficult to implement and slim gains. These last recipes are really for extreme cases.

Remember to measure, measure, measure.

## Recipes

### Use `strings.Builder` to concatenate many strings (3h9)
{: #recipe-3h9}

- Impact: Low.
- Implementation: Trivial.

If you are concatenating many strings, use [`strings.Builder`](https://pkg.go.dev/strings#Builder) instead of just joining them with the `+` operator. Don't just use it as a general replacement for the `+` operator, or if your are just concatenating every once in a while.

```go
var sb strings.Builder

for i := 0; i < 1000; i++ {
    sb.WriteString("a")
}

fmt.Println(sb.String())
```

See:
- [Set the size of the backing array on `strings.Builder` (a85)](#recipe-a85).

### Set the size of the backing array on `strings.Builder` (a85)
{: #recipe-a85}

- Impact: Low.
- Implementation: Trivial.

`strings.Builder` uses a slice to store the strings passed to it, when the final result is needed, they are just joined together. Given that we are working with an array behind the scenes, after implementing [recipe 3h9](#recipe-3h9), you may squeeze some more performance by avoiding the backing array of `strings.Builder` to grow many times by using the [`Grow()`](https://pkg.go.dev/strings#Builder.Grow) method before starting to write strings. If you have an idea of the final size of the string, in bytes, set it. If not, you may still do it with an educated guess by sacrificing space for better performance.

See:
- [Use `strings.Builder` to concatenate many strings (3h9)](#recipe-3h9).
- [Specify length or capacity on slices (shu)](#recipe-shu).

### Use `reflect.TypeOf` instead of the `fmt` package for a string representation of a type (71t)
{: #recipe-71t}

- Impact: Slim.
- Implementation: Easy.

Although both `fmt.Sprintf("%T", value)` and `reflect.TypeOf(value).String()` result in a string representation of a type, the one from the `reflect` package is way faster because does not have to parse a string. `fmt` also uses `reflect`, but after the parsing.

### Use `sync.Pool` for big and reusable objects to reduce GC work (us4)
{: #recipe-us4}

- Impact: Slim.
- Implementation: Easy.

If you have identified that the bottleneck is GC runs due to a couple of big objects that sometimes are needed in some parts of the codebase, you can try using `sync.Pool`. The idea behind this struct is storing big objects that are costly to make and are sometimes used but not that much to always keep them in memory, but not so little that it is better to make them each time they are needed. The problem it solves is very specific and due to the overhead it has it may not even actually help.

To use it define a function that will be called when a new object is needed:

```go
var bufPool = sync.Pool{
    New: func() any {
        // The Pool's New function should generally only return pointer
        // types, since a pointer can be put into the return interface
        // value without an allocation:
        return new(bytes.Buffer)
    },
}
```

Then get the object by calling `Get()`. When you are done with it, put it back with `Put()`:

```go
func Log(w io.Writer, key, val string) {
    b := bufPool.Get().(*bytes.Buffer)
    b.Reset()
    
    // Work with the buffer...

    bufPool.Put(b)
}
```

Keep in mind that the object that is given to you by calling `Get()` can be either new or an old one that was sent back to the pool. So reset it (or clean it) before using it or after putting it back (depending on which is faster). The objects that live in the pool are not guaranteed to live whilst the pool is alive. They may be freed by the GC at any time.

The only good usage example I know of is the one in the `fmt` package. There is a struct called `pp` that stores a printer state. Due to `fmt`'s multiple functions that format and print stuff, there may be many print calls by many goroutines, but also not that many may be needed all the time:

```go
// pp is used to store a printer's state and is reused with sync.Pool to avoid allocations.
type pp struct {
    // ...
}

var ppFree = sync.Pool{
    New: func() any { return new(pp) },
}

// newPrinter allocates a new pp struct or grabs a cached one.
func newPrinter() *pp {
    p := ppFree.Get().(*pp)
    p.panicking = false
    p.erroring = false
    p.wrapErrs = false
    p.fmt.init(&p.buf)
    return p
}

// free saves used pp structs in ppFree; avoids an allocation per invocation.
func (p *pp) free() {
    // Proper usage of a sync.Pool requires each entry to have approximately
    // the same memory cost. To obtain this property when the stored type
    // contains a variably-sized buffer, we add a hard limit on the maximum
    // buffer to place back in the pool. If the buffer is larger than the
    // limit, we drop the buffer and recycle just the printer.
    //
    // See https://golang.org/issue/23199.
    if cap(p.buf) > 64*1024 {
        p.buf = nil
    } else {
        p.buf = p.buf[:0]
    }
    if cap(p.wrappedErrs) > 8 {
        p.wrappedErrs = nil
    }

    p.arg = nil
    p.value = reflect.Value{}
    p.wrappedErrs = p.wrappedErrs[:0]
    ppFree.Put(p)
}
```

And here it is one part where it is used:

```go
// Fprintf formats according to a format specifier and writes to w.
// It returns the number of bytes written and any write error encountered.
func Fprintf(w io.Writer, format string, a ...any) (n int, err error) {
    p := newPrinter()
    p.doPrintf(format, a)
    n, err = w.Write(p.buf)
    p.free()
    return
}
```

See:
- [Source code of the `fmt` package where `sync.Pool` is used](https://github.com/golang/go/blob/d8e47e257e40ab03c5eaf2316eaea4cb83e650c3/src/fmt/print.go#L120).
- [`sync.Pool` documentation](https://pkg.go.dev/sync#Pool).
- [Try a buffered channel instead of `sync.Pool` (7ya)](#recipe-7ya).

### Try a buffered channel instead of `sync.Pool` (7ya)
{: #recipe-7ya}

- Impact: Slim.
- Implementation: Easy.

If you are using, or tried, `sync.Pool` to reduce GC pressure and didn't get the desired results (maybe better GC performance but worst performance of the code due to the overhead of `sync.Pool`) you can try an alternative and simpler memory pool: a buffered channel. Create a channel, fill it and receive from the channel whenever you need the object. When you are done with it, send it back to the channel:

```go
type BigObject struct {
    // ...
}

// Create a pool that can hold 10 objects (or whatever you need).
pool := make(chan *BigObject, 10)

// Fill it
for i := 0; i < cap(pool); i++ {
    bo := &BigObject{}
    pool <- bo
}

// Use them.
wg := sync.WaitGroup{}
for i := 0; i < 100; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        // Get one object.
        bo := <-pool
        // Return after working with it.
        defer func() { pool <- bo }()
        
        // Work with objects.
    }()
}
wg.Wait()
```

There some some differences with the `sync.Pool` approach:

- The number of objects is set. `sync.Pool` does not have a limit. You may need that, you may not.
- This is not a strictly faster version of `sync.Pool`. From what I've read it varies. Measure.
- If you need an object and the channel is empty, it will block until some other goroutine sends back an object. At the same time, if another goroutine, somehow (bugs do happen), tries to put an object whilst the channel is full, it will block. You can avoid this by using `select`'s when getting and putting back.
- Unlike with `sync.Pool`, the objects are not freed by the GC whilst the channel is in use.
- You have more control on the pool state. You can fill it at the start, you can create the objects when needed, you can create an algorithm that fills or empties the channel based on usage.

See:
- [Stack Overflow answer that describes this approach](https://stackoverflow.com/a/38506367).
- [Use `sync.Pool` for big and reusable objects to reduce GC work (us4)](#recipe-us4).

### Replace defer calls for normal ones (x9i)
{: #recipe-x9i}

- Impact: Slim.
- Implementation: Difficult.

`defer` calls are very useful, but they do add a bit of overhead to the calling code. If you have identified a really hot path with a defer call, move it to the end of the function and just call it like any other function. Do keep in mind that it may not be called if your function returns early, so you have to manage that. If you have many `defer`, due to a loop or something else, you have to keep track of each of them and call them in order before returning. Avoid implementing your own general `defer` for this last case, this may add more overhead than the highly optimized one offered by the language.

## General performance improvements

These recipes are to be used whenever possible. They are good practices to have while working on Go code that are usually easy to implement.

### Specify capacity on slices (shu)
{: #recipe-shu}

Whenever possible, specify the capacity when making a new slice, especially if you are going to use `append()` a lot. A slice is really just a view into an array, and arrays are fixed size. Whenever a slice is made, for example:

```go
slice := make([]int, 5, 10)
```

The length, 5, is the size of the slice. It is the number of elements that the slice holds. Using `make()`, initializes to the [zero value](https://yourbasic.org/golang/default-zero-value/) of the type specified. The capacity, 10, is the size of the backing array. If you append more elements than the ones that the backing array can hold, a new array of double the size will be created, every element will be copied over and the slice will now point to this new array. These are costly operations that should be avoided whenever possible. Especially in loops.

```go
slice := make([]int, 0, 2) // The slice is empty, the backing array has capacity for two elements.
slice = append(slice, 1, 2) // The slice and array are full.
slice = append(slice, 3) // Adding another element, makes a new array in the background, copies every element over and makes the slice point to the new array. The old array may create more work to the GC.
```

## Notes

- [Armadillo framework for software development](/armadillo-framework).
- [100 Go Mistakes and How to Avoid Them](https://100go.co/book/).
