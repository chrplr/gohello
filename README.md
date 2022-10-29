Example of deploying a go library on github
=======================================================

Create an _empty_ repository 'gohello' on http://github.com (do not add a README or LICENCE file,...)


Create the local folder for the library

    mkdir -p $GOPATH/github.com/chrplr/gohello
    cd $GOPATH/github.com/chrplr/gohello
    go mod init github.com/chrplr/gohello


Create the file `hello.go`:

```{go}
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


func hello(lang string) (greeting string, err error) {
	if gr, ok := Greetings[lang]; ok {
		return gr, nil
	} else {
		return "", errors.New("Unknown language")
	} 
}
```

test that it compiles

	go build .

Commit it

	git init
    git add hello.go go.mod README.md
	git commit -m 'first commit'
	git tag v0.0.1
	git branch -M main



Then push the local repo to github


	git remote add origin git@github.com:chrplr/gohello.git
	git push -u origin main



Using the module
-------------------

Create a new go project, that is, an empty folder


    mkdir myapp
    cd myapp

     

