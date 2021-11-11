# ace-commit-failure
Projects to illustrate commit failure issues

## General approach

The flows use a JavaCompute node to access JDBC, so the flow will try to commit the
JDBC changes (along with the MQPUT from the MQOutput node) when the flow ends. This
would normally be successfuly, but if the flows are invoked with a query parameter of
"fail=true" then the Java code does
```
if ( fail )
{
  conn.rollback();
  conn.close();
}
```
which causes a commit failure when the flow tries to call JDBC
```
2021-11-11 22:15:29.415546: BIP4362E: Java node error: Error committing Transaction for JDBC DataSourceName TEAJDBC. 
Exception in thread "Thread-20" 2021-11-11 22:15:29.415     41 <com.ibm.broker.plugin.MbRecoverableException class:com.ibm.broker.jdbctype4.localtrxn.JDBCType4SinglePhaseTrxnHandlerErrors@7134e551 method:JDBCType4SinglePhaseTrxnTable::commit source:BIPmsgs key:6268 >
2021-11-11 22:15:29.416     41  at com.ibm.broker.jdbctype4.localtrxn.JDBCType4SinglePhaseTrxnHandlerErrors.throwException(JDBCType4SinglePhaseTrxnHandlerErrors.java:213)
2021-11-11 22:15:29.416     41  at com.ibm.broker.jdbctype4.localtrxn.JDBCType4SinglePhaseTrxnTable.commit(JDBCType4SinglePhaseTrxnTable.java:1013)
2021-11-11 22:15:29.417     41  at com.ibm.broker.jdbctype4.localtrxn.JDBCType4SinglePhaseTrxnTable.accessThreadToTransactionTable(JDBCType4SinglePhaseTrxnTable.java:236)
2021-11-11 22:15:29.417     41  at com.ibm.broker.jdbctype4.localtrxn.JDBCType4SinglePhaseTrxnHandler.commitTrxnBranch(JDBCType4SinglePhaseTrxnHandler.java:565)
2021-11-11 22:15:29.418658: BIP2695E: Thread number '19870' with name 'TestFlow(Callable Input)' failed to commit a resource of type 'JDBC'. Other resources have been committed. 
2021-11-11 22:15:29.418786: BIP6265E: A problem was encountered when committing a transaction with the JDBC Datasource 'TEAJDBC'. 
```

## 01-CommitFailureNotPropagated

This flow shows the issue, in that even when invoked with fail=true the trace node is
still never called because the commit failure happens in the background.

![01 flow](01-CommitFailureNotPropagated.png)

## 02-WithCallableFlow

This flow has an extra stage added in, and would trigger the trace node if a commit 
occurs in the callable flow as long as the flow is running in XA (coordinated transaction)
mode. Without XA coordination, this flow behaves the same way as the 01 flow.

![02 flow](02-WithCallableFlow.png)

