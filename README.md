# Globally Unique ID Generator

Forked from https://github.com/rs/xid

Package xid is a globally unique id generator library, ready to be used safely directly in your server code.

- 6-byte value representing the nanoseconds since the Unix epoch,
- 6-byte random value

The string representation is using base32 hex (w/o padding) for better space efficiency
when stored in that form (20 bytes). The hex variant of base32 is used to retain the
sortable property of the id.

Xid doesn't use base64 because case sensitivity and the 2 non alphanum chars may be an
issue when transported as a string between various systems. Base36 wasn't retained either
because 1/ it's not standard 2/ the resulting size is not predictable (not bit aligned)
and 3/ it would not remain sortable. To validate a base32 `xid`, expect a 20 chars long,
all lowercase sequence of `a` to `v` letters and `0` to `9` numbers (`[0-9a-v]{20}`).

UUIDs are 16 bytes (128 bits) and 36 chars as string representation. Twitter Snowflake
ids are 8 bytes (64 bits) but require machine/data-center configuration and/or central
generator servers. xid stands in between with 12 bytes (96 bits) and a more compact
URL-safe string representation (20 chars). No configuration or central generator server
is required so it can be used directly in server's code.

| Name        | Binary Size | String Size    | Features
|-------------|-------------|----------------|----------------
| [UUID]      | 16 bytes    | 36 chars       | configuration free, not sortable
| [shortuuid] | 16 bytes    | 22 chars       | configuration free, not sortable
| [Snowflake] | 8 bytes     | up to 20 chars | needs machin/DC configuration, needs central server, sortable
| [MongoID]   | 12 bytes    | 24 chars       | configuration free, sortable
| xid         | 12 bytes    | 20 chars       | configuration free, sortable

[UUID]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[shortuuid]: https://github.com/stochastic-technologies/shortuuid
[Snowflake]: https://blog.twitter.com/2010/announcing-snowflake
[MongoID]: https://docs.mongodb.org/manual/reference/object-id/

Features:

- Size: 12 bytes (96 bits), smaller than UUID, larger than snowflake
- Base32 hex encoded by default (20 chars when transported as printable string, still sortable)
- Non configured, you don't need set a unique machine and/or data center id
- K-ordered
- Embedded time with 6 byte precision
- Lock-free (i.e.: unlike UUIDv1 and v2)

Best used with [xlog](https://github.com/rs/xlog)'s
[RequestIDHandler](https://godoc.org/github.com/rs/xlog#RequestIDHandler).

References:

- http://www.slideshare.net/davegardnerisme/unique-id-generation-in-distributed-systems
- https://en.wikipedia.org/wiki/Universally_unique_identifier
- https://blog.twitter.com/2010/announcing-snowflake

## Install

    go get github.com/JoinVerse/xid

## Usage

```go
guid := xid.New()

println(guid.String())
// Output: 9m4e2mr0ui3e8a215n4g
```

Get `xid` embedded info:

```go
guid.Time()
guid.Counter()
```

## Security Note

This implementation uses an `/dev/urandom' PRNG to add an unpredictable generation feature.

**NOTE**: If you use this library in an IaaS environment, we recommend to check which PRNG is used. If you use GPC, AWS
or Azure IaaS, check if your linux distribution has installed the `rng-tools` package to use a dedicated RNG hardware. 

Additional resources:
+ [Container-Optimized OS from Google - Security Considerations](https://www.chromium.org/developers/design-documents/chaps-technical-design#TOC-Security-Considerations)
+ [Intel DRNG](https://software.intel.com/en-us/articles/intel-digital-random-number-generator-drng-software-implementation-guide)
+ [Analysis to Linux default PRNG](http://users.ics.aalto.fi/arock/slides/slides_devrand.pdf)
+ [Alternative to default PRNG implementation](http://www.issihosts.com/haveged/)

## Benchmark

Benchmark against Go [Maxim Bublis](https://github.com/satori)'s [UUID](https://github.com/satori/go.uuid).

```
BenchmarkXID        	20000000	        91.1 ns/op	      32 B/op	       1 allocs/op
BenchmarkXID-2      	20000000	        55.9 ns/op	      32 B/op	       1 allocs/op
BenchmarkXID-4      	50000000	        32.3 ns/op	      32 B/op	       1 allocs/op
BenchmarkUUIDv1     	10000000	       204 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv1-2   	10000000	       160 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv1-4   	10000000	       195 ns/op	      48 B/op	       1 allocs/op
BenchmarkUUIDv4     	 1000000	      1503 ns/op	      64 B/op	       2 allocs/op
BenchmarkUUIDv4-2   	 1000000	      1427 ns/op	      64 B/op	       2 allocs/op
BenchmarkUUIDv4-4   	 1000000	      1452 ns/op	      64 B/op	       2 allocs/op
```

Note: UUIDv1 requires a global lock, hence the performence degrading as we add more CPUs.
