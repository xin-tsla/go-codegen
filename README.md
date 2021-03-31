# `go-codegen`, a simple code generation system

`go-codegen` is a simple template-based code generation system for go.  By
annotating structs with special-format fields, go-codegen will generate code
based upon templates provided alongside your package.

## Example usage

go-codegen works by building a catalog of available code templates, and then any
anonymous fields on a struct whose type match a template name will be invoked
with a struct that provides information about the struct that the template is
being invoked upon.

Take for example, the following go file:

```go
package main

import "fmt"

//go:generate go-codegen

// cmd is a template. Any type can have a template bound to it by creating the
// appropriate file on disk.
//
// aside: Blank interfaces are good to use for targeting templates
// as they do not affect the compiled package.
type cmd interface {
	Execute() (interface{}, error)
	MustExecute() interface{}
}

type HelloCommand struct {
	// HelloCommand needs to have the `cmd` template invoked upon it.
	// By mixing in cmd, we tell go-codegen so.
	cmd
	Name string
}

func (cmd *HelloCommand) Execute() (interface{}, error) {
	return "Hello, " + cmd.Name, nil
}

type GoodbyeCommand struct {
	cmd
	Name string
}

func (cmd *GoodbyeCommand) Execute() (interface{}, error) {
	return "Goodbye, " + cmd.Name, nil
}

func main() {
	var c cmd
	c = &HelloCommand{Name: "You"}
	fmt.Println(c.MustExecute())
	c = &GoodbyeCommand{Name: "You"}
	fmt.Println(c.MustExecute())
}

```

Notice that `HelloCommand` doesn't have a `MustExecute` method.  This code will
be generated by `go-codegen`.  Now we need to write the `cmd` template.

Create a new file named `cmd.tmpl` in the same package:

```go
// MustExecute behaves like Execute, but panics if an error occurs.
func (cmd *{{ .Name }}) MustExecute() interface{} {
	result, err := cmd.Execute()

  if err != nil {
    panic(err)
  }

  return result
}
```

Notice the `{{ .Name }}` expression:  It's just normal go template code.

Now, given both files, lets generate the code.  Run `go-codegen` in the package
directory, and you'll see a file called `main_generated.go` Whose content looks
like:

```go
package main

// MustExecute behaves like Execute, but panics if an error occurs.
func (cmd *HelloCommand) MustExecute() interface{} {
	result, err := cmd.Execute()

	if err != nil {
		panic(err)
	}

	return result
}

// MustExecute behaves like Execute, but panics if an error occurs.
func (cmd *GoodbyeCommand) MustExecute() interface{} {
	result, err := cmd.Execute()

	if err != nil {
		panic(err)
	}

	return result
}
```


## Installation

Presently, we only support installing from source.  You'll first need to installing
`gb`, used for building this project:

```bash
go get -u github.com/constabulary/gb/...
```

Then:
```bash
git clone git@github.com:nullstyle/go-codegen.git
cd go-codegen
gb build
cp ./bin/go-codegen /usr/local/bin
```

## Notes on template invocation

TODO

### Template Data

The "data" available in a template invocation is represented with a
`TemplateContext` struct.  You can see it [here](https://github.com/nullstyle/go-codegen/blob/master/src/github.com/nullstyle/go-codegen/template.go#L11).

### Finding templates

At present, a template is searched for within the same directory as a struct
invoking the template, using a name of the form `TemplateName.tmpl`.  For
example, invoking go-codegen in a directory that has a file named "FooBar.tmpl"
will cause that template to be invoked for any struct that has an anonymous
field with a type of `FooBar`.  `FooBar` may be any type: interface, struct or
otherwise.

### Arguments

TODO

### Adding Imports

A template may add additional imports into generated go giles by calling the
`AddImport` method on the `TemplateContext` like so:

```
{{ $.AddImport "net/http" }}
```

## TODO

- Add additional data about the invoking structs fields.
- Add more facilities to interpret struct tags:  Extract different named tags,
	for example.
- Template search paths.
- Better examples

## Example usages

Below is a list of some example usages of go-codegen.  Open up a github issue if
you would like to add your own.

- [go-horizon](https://github.com/stellar/go-horizon/blob/master/src/github.com/stellar/go-horizon/Action.tmpl) uses this to remove some boilerprate from their HTTP Handler code
- Some [examples from this project](https://github.com/nullstyle/go-codegen/tree/master/src/examples) illustrate simple usage

## Contributing

1. Fork it ( https://github.com/nullstyle/go-codegen/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
