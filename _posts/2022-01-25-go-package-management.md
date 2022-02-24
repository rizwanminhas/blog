---
layout: post
author: Rizwan Minhas
title: Go Package Management
excerpt: Organize code with Go packages. 
---

In this post I will show how to organize Go code in multiple files using same package and using different packages.

First create a go module using `go mod init myapp`.

Then create the following files:

1- `myapp/file1.go` with the following content:

```go
package main

import "fmt"

func demo() {
	fmt.Println("message from file1")
}

```

2- `myapp/utils/file2.go` with the following content:

```go
package utils

import "fmt"

func Demo() {
	fmt.Println("message from utils/file2.go Demo()")
}
```

3- Finally create the `myapp/main.go` with the following content:

```go
package main

import (
    "fmt"
    "myapp/utils"
)

func main() {
    demo()
    utils.Demo()
}
```

4- Now navigate to the root of `myapp` and execute `go run .` and you will see the following output:

```
message from file1
message from utils/file2.go Demo()
```

### How does it work?
1. `file1.go` uses the same `main` package hence you can call its function simply using `demo()`. **NOTE:** Function name is lower case.
2. `file2.go` uses a different `utils` package hence you will first have to import it using `myapp/utils` i.e. module name from the `go.mod` file and then the directory name i.e. `utils`. **NOTE:** The function name `Demo` in `file2.go` is also upper-case because it is a public function.
3. Finally instead of just running `main.go` I need to run all the files whose functions are getting called hence using `go run .`.
