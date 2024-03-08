---
title: "Go performance recipes"
permalink: /go-performance-recipes
custom_date: "240308"
---

# Go performance recipes

This is a list of recipes to improve the performance of Go code that I have used or encountered whilst working on projects. I keep this list updated.

Contents:
- [Recipes](#recipes).
    - [Use `strings.Builder` to concatenate many strings (3h9)](#recipe-3h9).
    - [Set the size of the backing array on `strings.Builder` (a85)](#recipe-a85).
    - [Use `reflect.TypeOf` instead of the `fmt` package for a string representation of a type (71t)](#recipe-71t).
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
