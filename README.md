[![GoDoc](https://godoc.org/github.com/gunk/gunk?status.svg)](https://godoc.org/github.com/gunk/gunk)
[![Build Status](https://travis-ci.org/gunk/gunk.svg?branch=master)](https://travis-ci.org/gunk/gunk)

# Gunk

Gunk is an acronym for "Gunk Unified N-terface Kompiler".

Gunk primarily works as a frontend for `protobuf`'s, `protoc` compiler. `gunk`
aims is to provide a way to work with `protobuf` files / projects in the same
way as the Go programming language's toolchain allows for working with Go
projects.

Gunk provides an alternative Go-derived syntax for defining `protobuf`'s, that
is simpler and easier to work with. Additionally, for developers familar with
the Go programming language will be instantly comfortable with Gunk files and
syntax.

### Contents

* [Installing](#installing)
* [Usage](#usage)
* [Contributing](#contributing)
* [How it works](#how-it-works)

## Installing

Gunk requires [Go][go-project] to be installed on your machine. To install Go,
follow the [installation instructions][go-install] for your specific operating
systems.

Gunk can then be installed in the usual Go fashion, with Go 1.11 or later:

	go get -u github.com/gunk/gunk

You'll also need `protoc` version 3.6 or later to run Gunk.

## Usage

### Syntax

The aim of Gunk is to provide Go-compatible syntax that can be natively read
and handled by the `go/*` package. As such, Gunk definitions are a subset of
the Go programming language:

```
package message

import "github.com/gunk/opt/http"

// Message is a Echo message.
type Message struct {
	// Msg holds a message.
	Msg  string `pb:"1" json:"msg"`
	Code int    `pb:"2" json:"code"`
}

// Util is a utility service.
type Util interface {
	// Echo echoes a message.
	//
	// +gunk http.Match{
	//		Method:	"POST",
	// 		Path:	"/v1/echo",
	// 		Body:	"*",
	//	}
	Echo(Message) Message
}
```

### Working with `*.gunk` files

Working with the `gunk` command line tool should be instantly recognizable to
experienced Go developers:

	gunk generate ./examples/util

### More information

Please see [the GoDoc API page][godoc] for a full API listing.

## How it works

Gunk works by using Go's standard library `go/ast` package (and related
packages) to parse all `*.gunk` files in a "Gunk Package" (and applying the
same kinds of rules as Go package), building the appropos protobuf messages. In
turn, those are passed to `protoc-gen-*` tools, along with any passed options.

## Contributing

Issues and pull requests are welcome. Building Gunk is simple:

```bash
$ git clone https://github.com/gunk/gunk
$ cd gunk
$ GO111MODULE=on go build
$ ./gunk
usage: gunk [<flags>] <command> [<args> ...]

Gunk Unified N-terface Kompiler command-line tool.

[...]
```

This project uses [go modules][go-modules] for managing dependencies, which
comes with Go 1.11 and above. Before sending a PR, remember to run `mod tidy`:

    GO111MODULE=on go mod tidy

## Gunk Config

Gunk is configurable by a `.gunkconfig` file. The `.gunkconfig` file or files
can be placed anywhere in the project and Gunk will attempt to find these
`.gunkconfig` files and merge them together. Gunk starts looking in the package
path and will continue looking up directories for any `.gunkconfig` until it
reaches the project root (the project root is where `.git` file or directory
or a `go.mod` file is).

Gunk uses an ini-style config file for declaring how Gunk should operate.
Each `[generate]` or `[generate <lang>]` section in the `.gunkconfig`
is a separate protoc generator run. A generate section can look like
the following:

```
[generate]
command=protoc-gen-go

[generate]
out=v1/js
protoc=js
```

`command` is a `protoc-gen-*` generator, it should be the binary name
of the generator to be run. The binary should be findable on your
`PATH`, otherwise the path to the binary can be used.

`protoc` will make Gunk call `protoc` with the out cli parameter
specified; `protoc=js` will call `protoc --js_out=OUT_PATH`

`out` is the file location the generated files should be outputted.

Any other key-values specified in a `generate` section will be
passed as plugin parameters to `protoc` or the `protoc-gen-*` generator.

You can also use a shortened generate section syntax:

```
[generate go]

[generate js]
out=v1/js
```

This is equivalent to:

```
[generate]
command=protoc-gen-go

[generate]
out=v1/js
protoc=js
```

### Example

```
# Example .gunkconfig for Go, grpc-gateway, Python and JS
[generate go]
out=v1/go
plugins=grpc

[generate]
out=v1/go
command=protoc-gen-grpc-gateway
logtostderr=true

[generate python]
out=v1/python

[generate js]
out=v1/js
import_style=commonjs
binary
```

## Convert

Gunk provides functionality to convert proto files or folders to a Gunk file.

    gunk convert path/to/file.proto other/path/to/file.proto
    gunk convert ./path/to/proto/dir

This will convert your proto file to the equivalent Gunk file.

## Gunk Annotations

Gunk provides annotations to configure proto options as well as other external
annotations, such as Google Api Annotations. Gunk annotations can be found at
[github.com/gunk/opt](https://github.com/gunk/opt).

Gunk annotations are specified as comments in Gunk code, the annotations should
be valid Go code prefixed with `+gunk`.

Gunk has annotations for all built-in proto options.

### Example

```
// +gunk java.Package("com.example.message")
// +gunk java.MultipleFiles(true)
package message

import (
    "github.com/gunk/opt/http"
    "github.com/gunk/opt/file/java"
)

type Util interface {
	// +gunk http.Match{
	//		Method:	"POST",
	// 		Path:	"/v1/echo",
	// 		Body:	"*",
	//	}
	Echo()
}
```

`java.Package` will set the proto file option [java_package](https://github.com/golang/protobuf/blob/882cf97a83ad205fd22af574246a3bc647d7a7d2/protoc-gen-go/descriptor/descriptor.proto#L324)

`http.Match` will generate Google Api Annotations and add them as extensions that will
can be used for any protoc generators, such as `grpc-gateway`.

[go-install]: https://golang.org/doc/install
[go-modules]: https://github.com/golang/go/wiki/Modules
[go-project]: https://golang.org/project
