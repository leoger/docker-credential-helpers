## Introduction

docker-credential-helpers is a suite of programs to use native stores to keep Docker credentials safe.

## Installation

Go to the [Releases](https://github.com/docker/docker-credential-helpers/releases) page and download the binary that works better for you. Put that binary in your `$PATH`, so Docker can find it.

### Building from scratch

The programs in this repository are written with the Go programming language. These instructions assume that you have previous knowledge about the language and you have it installed in your machine.

1 - Download the source and put it in your `$GOPATH` with `go get`.

```
$ go get github.com/docker/docker-credential-helpers
```

2 - Use `make` to build the program you want. That will leave an executable in the `bin` directory inside the repository.

```
$ cd $GOPATH/docker/docker-credentials-helpers
$ make osxkeychain
```

3 - Put that binary in your `$PATH`, so Docker can find it.

## Usage

### With the Docker Engine

Set the `credsStore` option in your `.docker/config.json` file with the suffix of the program you want to use. For instance, set it to `osxkeychain` if you want to use `docker-credential-osxkeychain`.

```json
{
  "credsStore": "osxkeychain"
}
```

### With other command line applications

The sub-package [client](https://godoc.org/github.com/docker/docker-credential-helpers/client) includes
functions to call external programs from your own command line applications.

There are three things you need to know if you need to interact with a helper:

1. The name of the program to execute, for instance `docker-credential-osxkeychain`.
2. The server address to identify the credentials, for instance `https://example.com`.
3. The username and secret to store, when you want to store credentials.

You can see examples of each function in the [client](https://godoc.org/github.com/docker/docker-credential-helpers/client) documentation.

### Available programs

1. osxkeychain: Provides a helper to use the OS X keychain as credentials store.
2. secretservice: Provides a helper to use the D-Bus secret service as credentials store.
3. wincred: Provides a helper to use Windows credentials manager as store.
4. pass: Provides a helper to use `pass` as credentials store.

#### Note

`pass` needs to be configured for `docker-credential-pass` to work properly.
It must be initialized with a `gpg2` key ID. Make sure your GPG key exists is in `gpg2` keyring as `pass` uses `gpg2` instead of the regular `gpg`.

## Development

A credential helper can be any program that can take command line arguments, read from the standard input and write to standard output. We use the first argument in the command line to differentiate the kind of command to execute. 

### Commands
There are four valid values for the command: `store`, `get`, `erase`, `list`.

- `store`: Adds credentials to the keychain. 
  - stdin: a [Credentials JSON document](#Credentials_JSON_document)
  - stdout: none
- `get`: Retrieves credentials from the keychain.
  - stdin: raw string value of the `ServerURL` field of the stored credentials
  - stdout: a [Credentials JSON document](#Credentials_JSON_document)
- `erase`: Removes credentials from the keychain.
  - stdin: raw string value of the `ServerURL` field of the stored credentials
  - stdout: none
- `list`: Lists stored credentials. 
  - stdin: none
  - stdout: a JSON map where each key is a `ServerURL` and its value is the corresponding `Username`. NOTE: current implementations exhibit an irregularity where `ServerURL` **only** includes the protocol prefix (e.g. "https://") in the output of the `list` command.)

### Credentials JSON document
The Credentials JSON document is simply a JSON serialization of the [Credentials struct](credentials/credentials.go). In other words, a JSON object with string-valued keys `ServerURL`, `Username` and `Secret`. NOTE: The value of `ServerURL` for current implementations is just a hostname with no protocol prefix, rather than a full "URL", as in the following example.

Example: 
```json
{
  "ServerURL":"private-registry.example.com",
  "Username":"alice",
  "Secret":"mdpDMeQv8b"
}
```

## Credentials libraries
This repository also includes libraries to implement new credentials programs in Go. Adding a new helper program is pretty easy. You can see how the OS X keychain helper works in the [osxkeychain](osxkeychain) directory.

1. Implement the interface `credentials.Helper` in `YOUR_PACKAGE/YOUR_PACKAGE_$GOOS.go`
2. Create a main program in `YOUR_PACKAGE/cmd/main_$GOOS.go`.
3. Add make tasks to build your program and run tests.

## License

MIT. See [LICENSE](LICENSE) for more information.
