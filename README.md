# go-kd

Golang K-D tree implementation with duplicate coordinate support

See https://en.wikipedia.org/wiki/K-d_tree for more information.

## Testing

```bash
go test github.com/downflux/go-kd/... -bench . -benchmem -timeout=60m
```

## Example

```golang
package main

import (
	"fmt"

	"github.com/downflux/go-geometry/nd/hyperrectangle"
	"github.com/downflux/go-geometry/nd/vector"
	"github.com/downflux/go-kd/container"
	"github.com/downflux/go-kd/container/kd"
	"github.com/downflux/go-kd/point"

	ckd "github.com/downflux/go-kd/kd"
)

// P implements the point.P interface, which needs to provide a coordinate
// vector function P().
var _ point.P = &P{}

type P struct {
	p   vector.V
	tag string
}

func (p *P) P() vector.V     { return p.p }
func (p *P) Equal(q *P) bool { return vector.Within(p.P(), q.P()) && p.tag == q.tag }

func main() {
	data := []*P{
		&P{p: vector.V{1, 2}, tag: "A"},
		&P{p: vector.V{2, 100}, tag: "B"},
	}

	// Data is copy-constructed and may be read from outside the k-D tree.
	//
	// N.B.: We are casting the k-D tree into a container type, as this
	// allows for the user to easily switch between implementations. The
	// user may directly consume the kd package API instead.
	var t container.C[*P] = (*kd.KD[*P])(
		ckd.New[*P](ckd.O[*P]{
			Data: data,
			K:    2,
			N:    1,
		}),
	)

	fmt.Println("KNN search")
	for _, p := range t.KNN(
		/* v = */ vector.V{0, 0},
		/* k = */ 2,
		func(p *P) bool { return true }) {
		fmt.Println(p)
	}

	// Remove deletes the first data point at the given input coordinate and
	// matches the input check function.
	p, ok := t.Remove(data[0].P(), data[0].Equal)
	fmt.Printf("removed %v (found = %v)\n", p, ok)

	// RangeSearch returns all points within the k-D bounds and matches the
	// input filter function.
	fmt.Println("range search")
	for _, p := range t.RangeSearch(
		*hyperrectangle.New(
			/* min = */ vector.V{0, 0},
			/* max = */ vector.V{100, 100},
		),
		func(p *P) bool { return true },
	) {
		fmt.Println(p)
	}
}
```
