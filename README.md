How to create libraries in Go
=====================================

Date: 2022-10-30

How to create your own libraries in Go is not obvious. The process is actually not at all complicated. Here, I show how to do it in practice. 

I proceed in three steps:

1. I briefly recapitulate how to create a local library inside a project
2. I show how to make a library a independent module that can be reused accross projects
3. I show how this independent module can be reused locally, or from the internet (that is, on github.com)

Thanks in advance for contacting me <christophe@pallier.org> if you notice any mistake or inaccuracy!

## How to how to create a local library inside a project. 

As code grows in complexity, it is always a good idea to split it in a main code and supporting libraries (sets of related objects and functions that focus on apurpose).

This is achieved, in Go, by spliting the code into different packages that can be imported from your main code using the `import` statement.

If you just want a library to be local and its source code distributed with your main application, it is as simple as creating a subfolder in your main  project folder. The following code generates a example project with a library `mylib`:


```
mkdir myapp
cd myapp

cat <<EOF >main.go
package main
	
import (
	   "fmt"
	   "myapp/mylib"
	)
	
func main() {
	    fmt.Println(mylib.Test())
}
EOF

go mod init myapp

# create the library
mkdir mylib
cat <<EOF >mylib/mylib.go
package mylib
func Test() string {
        return "this is mylib.Test"
}
EOF
	
go fmt ./...
```

After running the above code, the project has the following structure:

```
myapp/
├── go.mod
├── main.go
└── mylib
    └── mylib.go
```

To run the app, execute 

    go run . 
	
(inside `myapp` folder)

To generate a executable, use:

    go build .

To generate an executable to will be saved in `$GOPATH/bin`, use:

    go install .

Remarks:

- in `main.go`, you need to import `"myapp/mylib"` and not just `"mylib"`, even if the relative path is `./mylib`

- the library `mylib` is _not_ a Go module, only a package; there is no `go.mod` file inside `./mylib`

- since go 1.5, there is a reserved name for a special subfolder, `vendor`, which can hold copies of libraries used by the project. You do not need to specify `vendor` in the `import` statement. (see <https://blog.gopheracademy.com/advent-2015/vendor-folder/> for more information) 


## How to create an independent module

To make some folder containing packages an independent library in Go, you must tranform it into a  _module_, that is create a `go.mod` file at its root.

First, you must decide about the module's name, also known as the `module path`.

Because I plan to publish the module on github, I have chosen to name it `github.com/chrplr/gohello`,  (see <https://go.dev/doc/modules/managing-dependencies#naming_module> for advice about naming a module).

Note that `chrplr` is my username on github.com, therefore you need to change it and replace it with you own identifier in everything that follows. Otherwise, it will not work (unless you have managed to steal my credentials on github ;-) ).

Let us start by creating a local folder for the module and initialize the go.mod file with the module path:

    mkdir -p $GOPATH/src/github.com/chrplr/gohello
    cd $GOPATH/src/github.com/chrplr/gohello
	
	go mod init github.com/chrplr/gohello 

	
Remark:


- There is no obligation to create the folder `gohello` within `github.com/chrplr/gohello`, nor in `$GOPATH`; the folder can be created anywhere in your file system (the path to this folder _does not have to_ contain `github.com/chrplr`). It is just a convention to keep track of modules posted on github.com.

- If you are working inside `$GOPATH/src`, you can just type `go mod init`. Then, the module path will be automatically generated from the relative path.) 

Next, we create the file `hello.go` that will contain the library functions: 

```
cat <<EOF >hello.go
package gohello

import "errors"

var Greetings = map[string]string{
  "Chinese": "你好",
  "English": "Hello",
  "French": "Bonjour",
  "Hungarian": "Jó napot kívánok!",
  "Japanese":"こんにちは",
  "Russian":"привет",
  "Ukrainian":"привіт", 
  }


func Hello(lang string) (greeting string, err error) {
	if gr, ok := Greetings[lang]; ok {
		return gr, nil
	} else {
		return "", errors.New("Unknown language")
	} 
}
EOF

# Check that the code compiles
go build .
```

Remarks:

- the first line, `package gohello` is not the full module path (`github.com/chrplr/gohello`) but only the last bit.

- The name of this file does not matter! It could as well be named `main.go`, or `mylib.go`. What matter is the first line of the file that specifies the package. You can split your source code into several files that all start with the same package line):


## Link to github

Here, I assume that you have an account on <http://github.com>. 

First, you need to create a new _empty_ repository `gohello` on <http://github.com> (do not add a `README` nor a `LICENCE` file)

Second, you need to associate it to your local repository, that is, the folder containing your module, with the following commands:

	git init

    ID=chrplr  # replace by your own ID
	git remote add origin git@github.com:"$ID"/gohello.git

    # create first snapshot

    git add hello.go go.mod README.md
	git commit -m 'first commit'
	git tag v0.0.1
	git branch -M main

     # push the local repo to github:

	git push -u origin main


Remarks:

- remember to replace `chrplr` by your own github username remote repository name on the seconde line of code below. Use `git remote rm git@github.com:"/chrplr/gohello.git` if you have added my repo by mistake.

- Next time you make changes to the library's source code, you will use `git add ...`, `git commit ...` and `git push` to publish the new version on github.

## Importing the module in another project

Let's create a new go project that make use of our library:

```
mkdir ~/$GOPATH/src/myapp
cd ~/$GOPATH/src/myapp

cat <<EOF >main.go
package main

import (
	"fmt"
	"github.com/chrplr/gohello"
)

func main() {
	msg, err := gohello.Hello("Chinese")
	if err != nil {
		fmt.Errorf("%s", err)
	}
	fmt.Println(msg)
}
EOF

go mod init myapp
```


Now you have to decide if you want to use the local copy of the github.com/chrplr/gohello library, or the copy which is on the internet. 

### To use the version of the module which is on github

    go get github.com/chrplr/gohello
	cat go.mod
	go run .
	
This will download the `github.com/chrplr/gohello` module in `$GOPATH/pkg/mod/` and save this information in the `go.mod`.


### To use the local copy of the module in your `src` tree

To use the original module `github.com/chrplr/gohello`, assuming it is located in the the same folder as `myapp` (that is, `$GOPATH/src` for me), you can use:
	
	# Use the local copy of the module
    go mod edit -replace github.com/chrplr/gohello=../github.com/chrplr/gohello 
	go mod tidy
    cat go.mod
	go run .



### Run, build or install your app
	
	go run .
	go build .
	go install .
	
If you distribute `myapp`'s source code and used a local copy of the library, do not forget to clean `go.mod` by removing the line starting with `replace`.

## Final remarks

- The only difference between a library and an application is that a library does not contain a package `main`.

- Where do modules obtained form the Net with `go get` go? Answer: in `$GOPATH/pkg/mod`. If `GOPATH` does not exit, it is `$HOME/go`


For more information:

- <https://go.dev/doc/tutorial/create-module>
- To understand in details the algorithm use by Go to locate libraries, I recommend reading <https://lucasfcosta.com/2017/02/07/Understanding-Go-Dependency-Management.html> 
