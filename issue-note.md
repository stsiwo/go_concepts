# Issue Note

## Testing Prerequisites

1. use suite so that we can use lifecycle methods.
2. prepare test data in datbase for each test case if necessary
3. production code usualy contains Tx.
4. cleanup the database after each testing (e.g., truncate all tables) 

## Issues

### should I use Tx for each test case rather than non-Tx?

if database/sql package has a feature nested Tx or save points, I can wrap a Tx in production code with another Tx of the test case. However, it does not support that yet.

check [this](https://github.com/golang/go/issues/7898)

currently, use non-Tx for prep and clean up.

### Go Mod remove necessary dependencies sometimes

**error**: $ go run github.com/99designs/gqlgen generate
/go/pkg/mod/github.com/99designs/gqlgen@v0.14.0/cmd/gen.go:9:2: missing go.sum entry for module providing package github.com/urfave/cli/v2 (imported by github.com/99designs/gqlgen/cmd); to add:
        go get github.com/99designs/gqlgen/cmd@v0.14.0
        
this is because Go internally run 'go mod tidy' to remove unnecessary dependencies from your app (e.g., the one that you don't import), so this also causes the above error. the cmd dependency is necessary for generate graphql files but it is not imported to any file. 

**solutions**: 

reference [here](https://github.com/99designs/gqlgen/issues/1483)
        
