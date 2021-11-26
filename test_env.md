# Testing With Go

an overview of how to implement testing with go. 

## Prerequisites

1. use suite so that we can use lifecycle methods.
2. prepare test data in datbase for each test case if necessary
3. production code usualy contains Tx.
4. cleanup the database after each testing (e.g., truncate all tables) 

## Issues

### should I use Tx for each test case rather than non-Tx?

if database/sql package has a feature nested Tx or save points, I can wrap a Tx in production code with another Tx of the test case. However, it does not support that yet.

check [this](https://github.com/golang/go/issues/7898)

currently, use non-Tx for prep and clean up.
