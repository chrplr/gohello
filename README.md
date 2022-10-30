How to create libraries in Go
=====================================

Date: 2022-10-30

Finding out how to create your own libraries in Go is not obvious. The process is actually not at all complicated. I show here how to do it in practice, proceeding in three steps:

1. I briefly recapitulate how to create a local library inside a project
2. I show how to make a library a independent module that can be reused across projects
3. I show how this independent module can be reused locally, or from the internet (that is, on github.com)


## How to how to create a local library inside a project. 

As code grows in complexity, it is always a good idea to split it in a main code and supporting libraries (sets of related objects and functions that focus on a purpose). This is achieved, in Go, by splitting the code into different _packages_ that can be imported from your main code using the `import` statement. If you just want the library to be local to a project, and distribute it along the source code of the main application, this is as simple as creating a subfolder in your main  project folder. For example, here is the structure of a trivial project using a local library `mylib`:


```
myapp/
├── go.mod
├── main.go
└── mylib
    └── mylib.go
```

which can be generated it from the following code (run it!):

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

To run the app, execute (inside `myapp` folder):

    go run . 
	

To generate a executable, use:

    go build .

To generate an executable to will be saved in `$GOPATH/bin`, use:

    go install .


The code of the main application is `main.go`. As it imports the library thanks to the line `import "myapp/mylib"`, it can use its function `Test()` (which is public as indicated its name starting with an uppercase letter). 

Remarks:

- Despite the fact that the relative path is `./mylib`, in `main.go`, you need to import `"myapp/mylib"` and not just `"mylib"`.

- The library `mylib` is _not_ a Go module, only a package; there is no `go.mod` file inside `./mylib`

- Since go 1.5, there is a reserved name for a special subfolder, `vendor`, which can hold copies of libraries used by the project. You do not need to specify `vendor` in the `import` statement. (see <https://blog.gopheracademy.com/advent-2015/vendor-folder/> for more information) 


## How to create an independent module

To create an _independent_ library in Go which can be reused in several applications, you must transform it into a _module_, that is, create a `go.mod` file at the root of the project.

Before anything, you must decide about the module's name, technically known as its _module path_.
As I plan to publish the module on github, I have chosen to name it `github.com/chrplr/gohello`


Remarks:

- `chrplr` is my username on <http://github.com>, therefore you need to change it and replace it with you own identifier in everything that follows. Otherwise, it will not work (unless you have managed to steal my credentials on github ;-) ).


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


### Link it github.com

This step is only if you want to publish your code on github so that users can download it using `go get github.com/chrplr/gohello`. It you just want to keep it on your local filesystem, you can skip to the next section. 

Here, I assume that you have an account on <http://github.com>, otherwise you will need to register on it (or an equivalent site handling git repositories).

Firstly, create a new _empty_ repository `gohello` on <http://github.com>, that is, do not add a `README` nor a `LICENCE` file.

Secondly, associate it to your local repository. From the root folder of your module, and run the following commands:

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

- Remember to replace `chrplr` by your own github.com username remote repository name on the second line of code below. Use `git remote rm git@github.com:"/chrplr/gohello.git` if you have added my repository by mistake.

- Next time you make changes to the library's source code, you will use `git add ...`, `git commit ...` and `git push` to publish the new version on github.com

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


Now you have to decide if you want to use the local copy of the `github.com/chrplr/gohello` library, or the copy which is on the internet. 

In order to use a version of the module made available on github.com, you must run:

    go get github.com/chrplr/gohello

This will download the `github.com/chrplr/gohello` module in `$GOPATH/pkg/mod/` and save relevant information in `go.mod`.

Alternatively, if you want to use the local copy of the module `github.com/chrplr/gohello`, assuming that it is located in the same folder as `myapp` (that is, `$GOPATH/src` for me), you can use:
	
     go mod edit -replace github.com/chrplr/gohello=../github.com/chrplr/gohello 


Now, to run, build or install your app:
	
	go run .
	go build .
	go install .
	

If you distribute `myapp`'s source code and have used a local copy of the library, do not forget to clean `go.mod` by removing the line starting with `replace`. and run `go mod tidy` to clean `go.mod`.


## To go further

- Check <https://go.dev/doc/modules/managing-dependencies#naming_module> for advice about naming a module.
- To understand in details the algorithm use by Go to locate libraries, I recommend reading <https://lucasfcosta.com/2017/02/07/Understanding-Go-Dependency-Management.html> 
