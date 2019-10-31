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

后端项目的默认入口为`BaseSimplePageController<T>.importData()`，因此如果要实现默认的导入功能需要controller继承`com.belle.petrel.common.web.controller.BaseSimplePageController<T>`或其子类，如果未继承则要编写自定义的\`importData\(\)\`方法。

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

确定了是前端导入还是后端导入后，分别进入service层对应的方法，此处以常用的后端导入为示例，前端导入除了持久化和返回列表外其它操作都一致。

```java
    /**
     * 后端导入服务
     *
     * @param importReqDataVO 导入实体
     * @param systemUser      用户信息
     * @return
     */
    public Map<String, Object> importData(ImportReqDataVO importReqDataVO, SystemUser systemUser) {
        StringBuilder errMsgStr = new StringBuilder();

        // 导入的批量校验
        beforeImportData(importReqDataVO, systemUser);

        // 获取导入参数
        String colNamesStr = importReqDataVO.getColNames();
        String mustArrayStr = importReqDataVO.getMustArray();
        if (StringUtils.isEmpty(colNamesStr) || StringUtils.isEmpty(mustArrayStr)) {
            throw new MyselfMsgException("参数colNames与mustArray不能为空");
        }
        // 解析excel
        String[] colNames = colNamesStr.split(",");
        String[] mustArray = mustArrayStr.split(",");
        if (colNames.length != mustArray.length) {
            throw new MyselfMsgException("导入列colNames与是否必须mustArray的参数长度不一致");
        }

        //转换成导入数据
        ImportResolveVO<T> importData = ImportUtil.getImportData(importReqDataVO, this.entityClass, errMsgStr);

        //验证导入结果
        if (StringUtils.isNotEmpty(importData.getErrorMsg()) && importReqDataVO.getErrToExcel() != null && !importReqDataVO.getErrToExcel()) {
            //针对直接提示导入
            throw new ServiceException(importData.getErrorMsg());
            //
        } else if (StringUtils.isNotEmpty(importData.getErrorMsg()) && importReqDataVO.getErrToExcel() != null && importReqDataVO.getErrToExcel()) {
            //异常信息写入到Excel处理，不能直接抛出
            Map<String, Object> resultMap = new HashMap<>(3);
            ResultVO flag = new ResultVO();
            resultMap.put(ResultConstants.FLAG, flag);
            int successCount = importReqDataVO.getSuccessCount();
            String info = MessageUtils.message("base.simple.page.import.data", successCount, (importData.getEntityList().size() - successCount), errMsgStr.toString());
            flag.setRetCode(ResultVO.ERROR_CODE);
            flag.setRetMsg(info);
            flag.setRetDetail(importData.getRowErrMsg());
            return resultMap;
        }
        List<T> dataList = importData.getEntityList();
        if (Collections3.isEmpty(dataList)) {
            throw new MyselfMsgException("未解析到数据");
        }

        //导入中..
        importIng(importReqDataVO, importData, systemUser, errMsgStr);

        //导入后
        afterImportData(importReqDataVO, systemUser);

        // 处理导入结果信息
        Map<String, Object> resultMap = new HashMap<>();
        ResultVO flag = new ResultVO();
        resultMap.put(ResultConstants.FLAG, flag);
        int successCount = importReqDataVO.getSuccessCount();

        if (errMsgStr.length() > 0) {
            String info = MessageUtils.message("base.simple.page.import.data", successCount, (importData.getEntityList().size() - successCount), errMsgStr.toString());
            flag.setRetCode(ResultVO.ERROR_CODE);
            flag.setRetMsg(info);
            flag.setRetDetail(importData.getRowErrMsg());
            return resultMap;
        } else {
            String info = MessageUtils.message("base.simple.page.import.success.count", successCount);
            flag.setRetCode(ResultVO.SUCCESS_CODE);
            flag.setRetMsg(info);
            return resultMap;
        }

    }
```

首先提供了`beforeImportData()`的空实现钩子用于做导入前的校验，可按需要进行重写覆盖，如做参数覆盖等。

```java
    /**
     * 导入的批量校验
     * @param importResolveVO
     * @param systemUser
     */
    protected void beforeImportData(ImportReqDataVO importResolveVO, SystemUser systemUser) {
    }
```

随后通过`ImportUtil`将`ImportReqDataVO`中的`List<List<String>>`行集合转换为`ImportResolveVO`中的`List<T>`对象集合，转换过程中如果出现错误则停止下一步操作。

转换完成后，通过`importIng()`方法对对象集合进行数据库持久化，前端导入跳过此操作。

```java
    /**
     * 导入中
     * @param importReqDataVO 导入请求vo
     * @param importResolveVO 导入结果vo
     * @param systemUser      用户
     * @param errMsgStr       异常信息
     */
    protected void importIng(ImportReqDataVO importReqDataVO, ImportResolveVO<T> importResolveVO, SystemUser systemUser, StringBuilder errMsgStr) {
        // 导入具体逻辑
        for (T entity : importResolveVO.getEntityList()) {
            beforeImportOne(importReqDataVO, importResolveVO, entity, errMsgStr);
            importReqDataVO.setIndex(importReqDataVO.getIndex() + 1);
        }

        if (errMsgStr.length() > 0) {
            return;
        }

        importReqDataVO.setIndex(2);
        //设置默认值
        for (T entity : importResolveVO.getEntityList()) {
            // 新增前的逻辑处理
            BizUtils.setEntityDefaultField(entity, 0, systemUser, new Date());

            // 处理主表关联字段masterFields
            ImportUtil.processMasterFields(entity, importReqDataVO);

            // 上述逻辑都成立，才进行数据的插入
            getProxyService().insert(entity);
            importReqDataVO.setSuccessCount(importReqDataVO.getSuccessCount() + 1);

            // 处理行号
            importReqDataVO.setIndex(importReqDataVO.getIndex() + 1);
        }
    }
```

持久化之前，调用`beforeImportOne()`对每一行数据做进一步唯一性校验，检验通过后设置默认字段值并插入数据库。

```java
/**
     * 判断导入数据合法性
     * @param m       实体
     * @param reqData 导入请求
     * @param sb      导入结果信息
     * @return
     */
    public boolean beforeImportOne(ImportReqDataVO reqData, ImportResolveVO importResolveVO, T m, StringBuilder sb) {
        Map<String, String> errMsgMap = importResolveVO.getRowErrMsg();
        if (errMsgMap == null) {
            errMsgMap = new HashMap<>(10);
            importResolveVO.setRowErrMsg(errMsgMap);
        }
        // 检查导入唯一性
        String mainKeyStr = reqData.getUnionKey();
        if (StringUtils.isNotEmpty(mainKeyStr)) {
            String[] mainKey = mainKeyStr.split(",");
            EntityWrapper<T> entityWrapper = new EntityWrapper<>();
            for (String key : mainKey) {
                //获取数据库字段, tableField
                String fieldName = MybatisPlusUtil.humpToUnderline(m.getClass(), key);
                //获取value值
                Object value = ReflectUtils.invokeGetter(m, key);
                // 当value不为空的时候的才作为查询条件
                if (value != null) {
                    entityWrapper.eq(fieldName, value);
                }
            }
            // 当存在查询条件的时候才进行校验，避免lineId此类自增的id校验报错
            if (entityWrapper.isNotEmptyOfWhere()) {
                int count = this.selectCount(entityWrapper);
                if (count > 0) {
                    String infoMsg = MessageUtils.message("base.simple.page.import.row.exit", reqData.getIndex(), reqData.getUnionKeyName());
                    if (reqData.getValidateAll() != null && reqData.getValidateAll()) {
                        sb.append(infoMsg);
                        errMsgMap.put(reqData.getIndex() + ",", MessageUtils.message("base.simple.page.import.row.exit", reqData.getIndex(), reqData.getUnionKeyName()));
                    } else {
                        throw new ServiceException(infoMsg);
                    }
                    return true;
                }
            }
        }
        return true;
    }
```

持久化完或前端导入转换完后，提供`afterImportData()`钩子进行导入后处理，按 进行重写覆盖。对于前端导入还会提供转换后数据作为参数。

```java
    /**
     * 后端导入后事件
     * @param importResolveVO
     * @param systemUser
     */
    protected void afterImportData(ImportReqDataVO importResolveVO, SystemUser systemUser) {
        afterImportData(importResolveVO, systemUser, null);
    }
    
    /**
     * 前端导入后事件
     * @param importResolveVO
     * @param systemUser
     */
    protected void afterImportData(ImportReqDataVO importResolveVO, SystemUser systemUser, ImportResultVO importResultVO) {
    }
```

最后返回导入信息到服务中心。

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

