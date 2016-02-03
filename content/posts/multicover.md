---
title: "Coverage for multiple packages in go"
description: "Coverage for multiple packages in go"
date: "2014-05-06"
categories:
    - "go"
---

#### Prelude

There is awesome coverage in go. You can read about
it [here](http://blog.golang.org/cover). But also it has some limitations.
For example let's assume that we have next code structure:

```
src
├── pkg1
│   ├── pkg11
│   └── pkg12
└── pkg2
    ├── pkg21
    └── pkg22
```
`pkg2`, `pkg21`, `pkg22` uses `pkg1`, `pkg11` and `pkg12` in different cases.
So question is -- how we can compute **overall** coverage for our code base?

#### Generating cover profiles

Let's consider some possible go test commands with `-coveprofile`:

    go test -coverprofile=cover.out pkg2

tests run only for `pkg1` and cover profile generated only for pkg2

    go test -coverprofile=cover.out -coverpkg=./... pkg2

tests run only for `pkg2` and cover profile generated for all packages

    go test -coverprofile=cover.out -coverpkg=./... ./...

boo hoo: `cannot use test profile flag with multiple packages`

So, what we can do for running tests on all packages and get cover profile
for all packages?

#### Merging cover profiles

Now we able to get overall profile for each package individually.
It seems that we can merge this files. Profile file has next structure,
according to
[cover code](https://code.google.com/p/go/source/browse/cover/profile.go?repo=tools):

```
// First line is "mode: foo", where foo is "set", "count", or "atomic".
// Rest of file is in the format
//      encoding/base64/base64.go:34.44,37.40 3 1
// where the fields are: name.go:line.column,line.column numberOfStatements count
```

So, using magic of `awk` I found next solution to this problem:

```
go test -coverprofile=pkg1.cover.out -coverpkg=./... pkg1
go test -coverprofile=pkg11.cover.out -coverpkg=./... pkg1/pkg11
go test -coverprofile=pkg12.cover.out -coverpkg=./... pkg1/pkg12
go test -coverprofile=pkg2.cover.out -coverpkg=./... pkg2
go test -coverprofile=pkg21.cover.out -coverpkg=./... pkg2/pkg21
go test -coverprofile=pkg22.cover.out -coverpkg=./... pkg2/pkg22
echo "mode: set" > coverage.out && cat *.cover.out | grep -v mode: | sort -r | \
awk '{if($1 != last) {print $0;last=$1}}' >> coverage.out
```
The true meaning of last line I leave as an exercise for user :)
Now we have profile for all code, that was executed by all tests. We can use
this merged profile `coverage.out` for go cover tool:

```
go tool cover -html=coverage.out
```

or for generating [cobertura report](https://github.com/t-yuki/gocover-cobertura):

```
gocover-cobertura < coverage.txt > coverage.xml
```

#### Conclusion

Of course this solution is only workaround. And it works only for `mode: set`.
Similar logic must be embedded to cover tool. I am really hope that one day we
will be able to run

```
go test -coverprofile=cover.out -coverpkg=./... ./...
```
and leaning back in chair, enjoying perfect cover profiles.
