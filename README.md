### 事务传播

**@Transactional(propagation = Propagation.REQUIRES_NEW)**

<font face="黑体">asdfasdfasd</font>

```java
protected void doBegin(Object transaction, TransactionDefinition definition) {

DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;

Connection con = null;

try {

if (!txObject.hasConnectionHolder() ||

txObject.getConnectionHolder().isSynchronizedWithTransaction()) {

Connection newCon = obtainDataSource().getConnection();

if (logger.isDebugEnabled()) {

logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");

}

txObject.setConnectionHolder(new ConnectionHolder(newCon), true);

}

txObject.getConnectionHolder().setSynchronizedWithTransaction(true);

con = txObject.getConnectionHolder().getConnection();

Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);

txObject.setPreviousIsolationLevel(previousIsolationLevel);

txObject.setReadOnly(definition.isReadOnly());

// Switch to manual commit if necessary. This is very expensive in some JDBC drivers,

// so we don't want to do it unnecessarily (for example if we've explicitly

// configured the connection pool to set it already).

if (con.getAutoCommit()) {

txObject.setMustRestoreAutoCommit(true);

if (logger.isDebugEnabled()) {

logger.debug("Switching JDBC Connection [" + con + "] to manual commit");

}

con.setAutoCommit(false);

}

prepareTransactionalConnection(con, definition);

txObject.getConnectionHolder().setTransactionActive(true);

int timeout = determineTimeout(definition);

if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {

txObject.getConnectionHolder().setTimeoutInSeconds(timeout);

}

// Bind the connection holder to the thread.

if (txObject.isNewConnectionHolder()) {

TransactionSynchronizationManager.bindResource(obtainDataSource(), txObject.getConnectionHolder());

}

}

catch (Throwable ex) {

if (txObject.isNewConnectionHolder()) {

DataSourceUtils.releaseConnection(con, obtainDataSource());

txObject.setConnectionHolder(null, false);

}

throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);

}

}
```
