# 导入

校验日期：

---

## 导入流程

1. 前台请求导入导出服务中心（//TODO 查看前台导入文档）。
2. 导入中心内解析excel获取导入信息，再调用具体业务工程接口（默认接口/importData）。
3. 业务工程内进行数据校验等处理，并持久化到数据库。

---





---

## 参数说明

### ImportReqDataVO

#### colNames

```
"ownerNo,ownerName,storeNo,storeName,forwarderCode,forwarderName,expressFlag,expressFlagName,remarks"
```

* 实体类中的字段名称。
* 以逗号【,】分割，按顺序对应excel中的各列数据

#### mustArray

```
"true,false,true,false,true,false,true,false,true,true,false"
```

* 字段是否必须。
* 以逗号【,】分割，按顺序对应excel中的各列数据。
* 与[`colNames`](#colnames)中的长度要一致。
* 当全部非必须时可以为空。

#### unionKey

```
"ownerNo,brandDeptNo"
```

* 唯一约束列。
* 多列时以逗号【,】分割。
* 无需校验时可以为空。
* 性能消耗较大，导入数量过多时不建议使用。

#### unionKeyName

```
"XX唯一约束"
```

* 唯一约束名称。
* 目前仅在唯一约束错误信息展示中使用。

#### validateAll

```
true/false
```

* 是否验证全部列的唯一约束。
* 当`unionKey` 存在时才需要判断。
* 当为true时，验证完全部行后再返回所有的错误行信息。
* 当为false或null时，首次验证到错误行就返回错误信息。
* 不管值为哪个，只要有错误行存在，该批数据都不会导入。

#### masterFields

```
"[{\"field\":\"msCode\",\"value\":\"G01\"}]"
```

* 补充字段，用于补充导入模板中不存在的字段。
* 当主从表导入，且主从表间的外键不在导入模板中时，可以将外键字段放入此处，自动加入到每一行中。
* 当某一列为固定值，不需要在模板中显式写入时，可放入此处，自动加入到每一行中。

#### frontImport

```
false/true
```

* 是否前端导入。
* 当为true时，导入的数据不持久化到数据库，而是直接返回给前台使用。
* 当为false或null时，导入的数据正常持久化到数据库中。

#### params

```
"{\"type\":\"normal\"}"
```

* 扩展参数。
* 普通导入时直接通过params获取前端所传扩展参数字符串。
* 大数据量导入时会还需要在params中获取key=frontParams后才能得到扩展参数字符串。

#### async

```
true/false
```

* 是否异步导入。

* 由导入导出服务中心判断是否为异步，业务项目内不需要关注。

#### errToExcel

```
true/false
```

* 是否将异常信息写入到excel
* 当为true时返回异常信息。
* 当为false或null时直接抛出异常信息。
* 只有为异步导入时有效。  

    /\*\*

     \* 写入Excel数据的接口

     \*/

    private String url;



    /\*\*

     \* 模块名称

     \*/

    private String moduleName;



    //===================================================

    /\*\*

     \* excel解析后数据，后台使用

     \*/

    private List&lt;List&lt;String&gt;&gt; dataList;



    /\*\*

     \* 当前行

     \*/

    @JsonIgnore

    private Integer index = 2;



    /\*\*

     \* 成功条数

     \*/

    @JsonIgnore

    private Integer successCount = 0;

    /\*\*

     \* 导入Excel文件id

     \*/

    @JsonIgnore

    private String fileId;



