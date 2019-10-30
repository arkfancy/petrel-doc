# 编码规则

## 使用场景

* 后端生成单据时，取单据编号
* 生成指定规则的全局序列

## 使用方法

#### 创建编码规则

1. 页面打开**【新物流系统】**-【**集成中心**】-【**系统设置**】-【**编码规则**】。![](/assets/编码规则主页面.png)
2. 点击【新建】按钮，创建编码规则后并保存。![](/assets/编码规则新增页面.png)

> 查看[编码规则字段说明。](/kuo-zhan/bian-ma-gui-ze/bian-ma-gui-ze-zi-duan-shuo-ming.md)

#### 获取编码规则

1. 引入编码规则所需依赖：
   ```
       <dependency>
           <groupId>com.belle</groupId>
           <artifactId>petrel-itg-client</artifactId>
           <version>last-version</version>
       </dependency>
   ```
2. 在需要获取编码规则的类中声明调用接口：

3. 


