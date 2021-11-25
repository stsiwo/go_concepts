# database/sql package note

## source

https://pkg.go.dev/database/sql

## notes

### sql.DB is thread-safe so it can be shared by as many GR as you want.

- sql.DB is a representation of a pool of database connections.
- don't need to close sql.DB.
- don't need to handle a connection. the DB automatically handles creating a Conn instance internally so that you can use db.Query/Exec/... without thinking about the connection.
- sql.Tx from sql.DB is bound to a single connection so it should be used for the connection (e.g., the current go routine) and don't share it with multiple connections.
- Use the APIs described in this section to manage transactions. Do not use transaction-related SQL statements such as BEGIN and COMMIT directly—doing so can leave your database in an unpredictable state, especially in concurrent programs.
- When using a transaction, take care not to call the non-transaction sql.DB methods directly, too, as those will execute outside the transaction, giving your code an inconsistent view of the state of the database or even causing deadlocks.

## issues

### SET FOREIGN_KEY_CHECKS=0; (non-Tx) not working when using Tx.

bg) i used to non-Tx db operation for perparing test data in db and the clean up, and i used to Tx for testing to production code. the flow is the following:

1. (non-Tx) prepare the test data in database (e.g., insert)
2. (Tx) test producion code
3. (non-Tx) clean up (e.g., truncate all tables) // also set SET FOREIGN_KEY_CHECKS=0; here but mysql gives me a foreign key constraint.

solutions)

- don't use non-Tx and Tx together.
- should use Tx for prep and clean up too.

