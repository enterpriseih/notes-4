# 消息体格式

```java
/****************************************************************
* message model
****************************************************************/
message Entry {
   /**协议头部信息**/
 optional Header                   header              = 1;
 
   /**打散后的事件类型**/
 optional EntryType             entryType        = 2 [default = ROWDATA];
 
   /**传输的二进制数组**/
 optional bytes             storeValue           = 3;
 
   /**additional info**/
 optional int64                    batchId             = 4;
 
   /**additional info**/
 optional int64                inId            = 5;
 
   /**additional info**/
 optional string                   ip             = 6;
}
 
/**message Header**/
message Header {
   /**协议的版本号**/
 optional int32                 version             = 1 [default = 1];
 
   /**binlog/redolog 文件名**/
 optional string                logfileName          = 2;
 
   /**binlog/redolog 文件的偏移位置**/
 optional int64                 logfileOffset     = 3;
 
   /**服务端serverId**/
 optional   int64           serverId           = 4;
 
   /** 变更数据的编码 **/
 optional string                serverenCode      = 5;
 
   /**变更数据的执行时间 **/
 optional int64             executeTime          = 6;
 
   /** 变更数据的来源**/
 optional Type              sourceType       = 7 [default = MYSQL];
 
   /** 变更数据的schemaname**/
 optional string                schemaName       = 8;
 
   /**变更数据的tablename**/
 optional string                tableName        = 9;
 
   /**每个event的长度**/
 optional int64             eventLength         = 10;
 
   /**数据变更类型**/
 optional EventType              eventType        = 11 [default = UPDATE];
 
   /**预留扩展**/
 repeated Pair              props           = 12;
}
 
/**每个字段的数据结构**/
message Column {
   /**字段下标**/
 optional int32    index        =     1;
 
   /**字段java中类型**/
 optional int32        sqlType          =     2;
 
   /**字段名称(忽略大小写)，在mysql中是没有的**/
 optional string       name         =     3;
 
   /**是否是主键**/
 optional bool     isKey        =     4;
 
   /**如果EventType=UPDATE,用于标识这个字段值是否有修改**/
 optional bool     updated          =     5;
 
   /** 标识是否为空 **/
 optional bool     isNull       =     6 [default = false];
 
   /**预留扩展**/
 repeated Pair     props        =     7;
 
   /** 字段值,timestamp,Datetime是一个时间格式的文本 **/
 optional string       value        =     8;
 
   /** 对应数据对象原始长度 **/
 optional int32    length       =     9;
 
   /**字段mysql类型**/
 optional string       mysqlType     =     10;
}
 
message RowData {
 
   /** 字段信息，增量数据(修改前,删除前) **/
 repeated Column          beforeColumns      =     1;
 
   /** 字段信息，增量数据(修改后,新增后) **/
 repeated Column          afterColumns   =     2;
 
   /**预留扩展**/
 repeated Pair        props        =     3;
}
 
/**message row 每行变更数据的数据结构**/
message RowChange {
 
   /**tableId,由数据库产生**/
 optional int64       tableId          =     1;
 
   /**数据变更类型**/
 optional EventType        eventType     =     2 [default = UPDATE];
 
   /** 标识是否是ddl语句 **/
 optional bool        isDdl        =     10 [default = false];
 
   /** ddl/query的sql语句 **/
 optional string          sql          =     11;
 
   /** 一次数据库变更可能存在多行 **/
 repeated RowData      rowDatas      =     12;
 
   /**预留扩展**/
 repeated Pair        props        =     13;
 
   /** ddl/query的schemaName，会存在跨库ddl，
   需要保留执行ddl的当前schemaName **/
 optional string          ddlSchemaName  =     14;
}
 
/**开始事务的一些信息**/
message TransactionBegin{
 
   /**已废弃，请使用header里的executeTime**/
 optional int64       executeTime       =     1;
 
   /**已废弃，Begin里不提供事务id**/
 optional string          transactionId  =     2;
 
   /**预留扩展**/
 repeated Pair        props        =     3;
 
   /**执行的thread Id**/
 optional int64       threadId      =     4;
}
 
/**结束事务的一些信息**/
message TransactionEnd{
 
   /**已废弃，请使用header里的executeTime**/
 optional int64       executeTime       =     1;
 
   /**事务号**/
 optional string          transactionId  =     2;
 
   /**预留扩展**/
 repeated Pair        props        =     3;
}
 
/**预留扩展**/
message Pair{
   optional string       key             =        1;
   optional string       value        =        2;
}
 
/**打散后的事件类型，主要用于标识事务的开始，变更数据，结束**/
enum EntryType{
   TRANSACTIONBEGIN      =     1;
   ROWDATA                =     2;
   TRANSACTIONEND       =     3;
   /** 心跳类型，内部使用，外部暂不可见，可忽略 **/
 HEARTBEAT           =     4;
}
 
/** 事件类型 **/
enum EventType {
    INSERT        =     1;
    UPDATE        =     2;
    DELETE        =     3;
    CREATE    =     4;
    ALTER     =     5;
    ERASE     =     6;
    QUERY     =     7;
    TRUNCATE   =     8;
    RENAME        =     9;
    /**CREATE INDEX**/
 CINDEX    =     10;
    DINDEX        =     11;
}
 
/**数据库类型**/
enum Type {
    ORACLE    =     1;
    MYSQL     =     2;
    PGSQL     =     3;
}
```



# 读取转换api

```java
// 从二进制消息体中获取entry
WaveEntry.Entry entry = WaveEntry.Entry.parseFrom(message.getData());

// 还可以获得表名
String tableName = entry.getHeader().getTableName();

// 获取entry类型, TRANSACTIONBEGIN, ROWDATA, TRANSACTIONEND
entry.getEntryType();

// 获取rowChange, 每行变更数据的数据结构
WaveEntry.RowChange rowChange = 
    WaveEntry.RowChange.parseFrom(entry.getStoreValue());

// 获取eventType
WaveEntry.EventType eventType = rowChange.getEventType();

// 一次数据库变更可能存在多行
for (WaveEntry.RowData rowData : rowChange.getRowDatasList()) {
    
    List<WaveEntry.Column> beforeColumns = rowData.getBeforeColumnsList();
    List<WaveEntry.Column> afterColumns = rowData.getAfterColumnsList();
    
    Map<String, Object> beforeMap = convertColumn2Map(beforeColumns);
    Map<String, Object> afterMap = convertColumn2Map(afterColumns);
    
}


protected Map<String, Object> convertColumn2Map(List<WaveEntry.Column> columns) {
    if (CollectionUtils.isEmpty(columns)) {
        return null;
    }
    Map<String, Object> columnMap = new HashMap<>(columns.size());
    for (WaveEntry.Column column : columns) {
        // 只读取name和value，name是字段名，value是字段值
        // 此处的字段是该条记录的所有字段
        columnMap.put(column.getName(), column.getValue());
    }
    return columnMap;
}



```

