# Excel导入

校验日期：

---

## 导入流程

1. 前台请求导入导出服务中心（//TODO 查看前端导入文档）。
2. 服务中心内解析excel获取导入信息，再调用具体业务工程接口（默认接口/importData）。
3. 业务工程内进行数据校验等处理，并持久化到数据库。

---

## 流程详解

> 此详解仅包含从服务中心进入到业务项目后端的后续流程，从业务项目前端到服务中心的流程请//TODO查看前端导入文档。

### BaseSimplePageController

        后端项目的默认入口为`BaseSimplePageController<T>.importData()`，因此如果要实现默认的导入功能需要controller继承`com.belle.petrel.common.web.controller.BaseSimplePageController<T>`或其子类，如果未继承则要编写自定义的`importData()`方法。

```java
    @ApiOperation(value = "导入数据")
    @PostMapping("/importData")
    @ResponseBody
    public Map<String, Object> importData(HttpServletRequest request, @RequestBody ImportReqDataVO reqData) {
        if (reqData.getDataList() == null || reqData.getDataList().isEmpty()) {
            ResultMapUtils.getErrorResultMap(MessageUtils.message("base.page.import.data.null"));
        }

        Map resultMap;
        if (reqData.getFrontImport() != null && reqData.getFrontImport()) {
            resultMap = getBaseSimplePageService().frontImport(reqData, ThreadLocals.getCurrentUser());
        } else {
            resultMap = getBaseSimplePageService().importData(reqData, ThreadLocals.getCurrentUser());
        }

        return resultMap;
    }
```

        方法内根据`frontImport`的值判断为前端导入还是后端导入，两者的区别是：前端导入校验完数据后直接返回服务中心，再由服务中心原样返回给前端做进一步处理展示；后端导入校验完数据后会将数据持久化到数据库中，仅返回处理结果。

### BaseSimplePageService

//TODO

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
"[{\"field\":\"msCode\",\"value\":\"G01\"}]
"
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

* 由服务中心判断是否为异步，业务项目内不需要关注。

#### errToExcel

```
true/false
```

* 是否将异常信息写入到excel
* 当为true时返回异常信息。
* 当为false或null时直接抛出异常信息。
* 只有为异步导入时有效。

#### url

```
"https://dev-petrel.belle.net.cn/petrel/oms-e-api/bmExpressTactics/importData"
```

* 当前导入数据时的服务接口
* 用于前端指定后端导入接口，后端项目内不需要关注。

#### moduleName

```

```

* 模块名称。
* 服务中心发出通知消息时提示用，后端项目内不需要关注。

#### dataList

```
[[{"columnKey":"columnValue1"},{"rowNum":1}],[{"columnKey":"columnValue2"},{"rowNum":2}]]
```

* 解析成列表后的excel数据。

#### index

```
2
```

* 当前行号。

#### successCount

```
100
```

* 成功条数。

#### fileId

```
""
```

* excel文件id。
* 只在导入导出中心内使用，不传递到业务项目中，目前默认为空，业务项目不需要关注。

---

## 业务场景

//TODO









