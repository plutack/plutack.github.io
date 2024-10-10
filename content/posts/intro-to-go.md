---
title: "Learning Go: POV of a JavaScript Developer PT. 1"
date: 2024-09-25T18:42:50+01:00
draft: false
toc: false
images:
tags: 
  - learning
  - Go
---


As someone whose first programming language is dynamically typed, it only made sense to explore what the other side of the coin looks like. So, I started looking for a statically-typed language to learn. I already appreciated TypeScript, although some would argue it's more like a linter, so that was a natural stepping stone. In my search, I came across several choices like classic Java, C, OCaml, Go, and a few others.

Initially, I wanted to settle on Rust—and I did for a while—for a few reasons: Rust is well-known for encouraging memory-safe code, and it seems to be gaining traction lately. An [article](https://www.whitehouse.gov/oncd/briefing-room/2024/02/26/press-release-technical-report/) released by the U.S. government hinted at a shift towards using Rust more widely in modern software. As a software engineer looking for opportunities both locally and abroad, I figured Rust developers might be in high demand soon.

However, a few days in, it became clear that I was lacking the background knowledge needed to continue with Rust. After weighing the pros and cons, I decided to switch gears and learn Go instead. Go felt like the perfect balance—offering simplicity while coming from my JavaScript experience.

[**Go**](https://go.dev) is known for its straightforward syntax. It also handles some things, like garbage collection and memory allocation. 

## Why Go?

As a JavaScript developer exploring new languages, I found several reasons why Go stands out, especially as I transition from a dynamically typed language to a statically typed one. Here are some key advantages that resonate with me:

1. **Simplicity and Readability**

Go’s syntax is clean and straightforward, much like JavaScript. This makes it easy for someone like me, who is just starting to learn a new language, to pick it up quickly. The simplicity of Go helps reduce the mental overhead that often comes with learning complex syntax in other languages.

2. **Performance**

Being a compiled language, Go offers impressive performance. While JavaScript runs in a browser or on Node.js, Go compiles to machine code, allowing applications to run faster. This is particularly appealing for building high-performance applications and services and complements a topic I’m currently pondering: "what scaling for a large audience looks like," which I’m eager to explore.

3. **Single Executable**

One of the standout features of Go is that it compiles to a single binary executable. Unlike JavaScript, where users need Node.js installed, I can distribute my Go applications without requiring users to install anything extra. This makes deployment much easier, especially for command-line tools or server applications. Building CLI tools is an idea I am most certainly going to explore in the future and Go seems to be the better choice.

4. **Cross-Platform Compatibility**

Go allows you to build applications that run on various operating systems without much hassle. By setting `GOOS` and `GOARCH`, I can compile my code for Windows, macOS, or Linux from a single codebase. This flexibility is definitely a big plus.

5. **Built-in Concurrency**

As someone who has primarily worked with JavaScript’s asynchronous programming model, Go’s approach promises a better way to handle concurrency with goroutines and channels to write concurrent code, allowing for efficient task management. 

6. **Strong Standard Library**

Go has a robust standard library that provides a wide range of functionalities, including HTTP handling, file manipulation, and more—similar to Node.js built-in modules, but arguably more powerful. This means I can accomplish many tasks without relying heavily on external libraries, much like how I sometimes use Node.js core modules.


7. **Strong Typing and Error Handling**

Transitioning from JavaScript’s dynamic typing to Go’s static typing is a significant shift. However, I’ve seen how leveraging compile-time error catching in TypeScript can lead to more robust and reliable code, while also saving development time by reducing debugging efforts. Go’s explicit error handling introduces a new approach, leading me to explore the "errors as values vs exceptions" debate, and I’m eager to see which approach resonates with me more.

## Setting Up Go

Go has extensive setup guides for different systems on its website. As an Arch Linux user (btw 😎), installing Go was pretty simple since it’s available in the Arch Extra repository:

```sh
paru -S go 
```

In VS Code, I also installed the Go extension to work smoothly with the binary downloaded earlier. Although I plan to switch to the Neovim + Tmux combo in the future (more on that later).

## The Usual Ritual

As is customary when learning a new language, I had to perform the ritual of printing the first "Hello, World!"—though I tend to default to 'Foo Bar' now. Here's how I did it:

- First, I created a directory:
```sh
mkdir -p go/test
```

- Then, I navigated to the newly created directory:
```sh
cd go/test
```

- I opened the directory in VS Code:
```sh
code .
```

- Next, I initialized a Go module (similar to package.json in Node.js):
```sh
go mod init
```

- I created a Go file:
```sh
touch main.go
```

- Then, I wrote my first bit of Go code:
```go
package main

func main(){
	println("FOO BAR")
}
```

- Finally, I ran the code:
```sh
go run .
```

**Note:** Go looks for an entry file that satisfies three conditions:

1. The file has a `.go` extension.
2. It starts with the statement `package main`.
3. It declares a `main()` function.

## What’s Next?
In the upcoming posts, I will be delving into the fundamentals of Go and highlighting contrasts with JavaScript, such as:

- **Variable Declaration & Assignment**: How Go handles variables and types.
- **Pointers**: Go’s approach to pointers and memory management.
- **Functions**: Writing reusable functions, passing arguments, and returning values.
- **Go Routines**: A sneak peek into Go’s concurrency model.
