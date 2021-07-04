# carbites

[![Build](https://github.com/alanshaw/go-carbites/actions/workflows/main.yml/badge.svg)](https://github.com/alanshaw/go-carbites/actions/workflows/main.yml)
[![Standard README](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg)](https://github.com/RichardLitt/standard-readme)
[![Go Report Card](https://goreportcard.com/badge/github.com/alanshaw/go-carbites)](https://goreportcard.com/report/github.com/alanshaw/go-carbites)

Chunking for [CAR files](https://ipld.io/specs/transport/car/). Split a single CAR into multiple CARs.

## Install

```sh
go get github.com/alanshaw/go-carbites
```

## Usage

Carbites supports 2 different strategies:

1. **Simple** (default) - fast but naive, only the first CAR output has a root CID, subsequent CARs have a placeholder "empty" CID. The first CAR output has roots in the header, subsequent CARs have an empty root CID [`bafkqaaa`](https://cid.ipfs.io/#bafkqaaa) as [recommended](https://ipld.io/specs/transport/car/carv1/#number-of-roots).
2. **Treewalk** - walks the DAG to pack sub-graphs into each CAR file that is output. Every CAR has the same root CID, but contains a different portion of the DAG. Every CAR file has the _same_ root CID but a different portion of the DAG. The DAG is traversed from the root node and each block is decoded and links extracted in order to determine which sub-graph to include in each CAR.

```go
package main

import (
	"io"
	"github.com/alanshaw/go-carbites"
)

func main() {
	out := make(chan io.Reader)

	go func() {
		var i int
		for r := range out {
			b, _ := ioutil.ReadAll(r)
			ioutil.WriteFile(fmt.Sprintf("chunk-%d.car", i), b, 0644)
			i++
		}
	}()

	var reader io.Reader
	targetSize := 1000 // 1kb chunks
	strategy := carbites.Simple // also carbites.Treewalk
	err := carbites.Split(context.Background(), reader, targetSize, strategy, out)
}

```

## API

[pkg.go.dev Reference](https://pkg.go.dev/github.com/alanshaw/go-carbites)

## Contribute

Feel free to dive in! [Open an issue](https://github.com/alanshaw/go-carbites/issues/new) or submit PRs.

## License

Dual-licensed under [MIT + Apache 2.0](https://github.com/alanshaw/go-carbites/blob/main/LICENSE.md)
