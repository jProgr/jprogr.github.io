---
title: "Go library for the observer abstraction: events"
permalink: /events-release
custom_date: "230112"
---

# Go library for the observer abstraction: events

jProgr/events is a library that I built, mostly for fun, that provides the observer abstraction in Go. Basically one registers event IDs, simple strings, and later one can fire events, wrappers around an event ID and some data, that are handled by listeners, functions that receive the event data.

The code is open source, if you want to contribute or just take a look:

- GitHub: [https://github.com/jProgr/events](https://github.com/jProgr/events).
- Godoc: [https://pkg.go.dev/github.com/jProgr/events](https://pkg.go.dev/github.com/jProgr/events).

## Release history

### 1.0.1
#### Changed
- Move some files to ease importing through go tools.

### 1.0.0
#### Added
- Added event dispatcher.
