# 代码生成器

校验日期：//TODO

---

* 优势
  * 使用操作简单，提供可视化界面。
  * 快速生成基础代码。
  * 自动升级，拥有最新特性功能。
  * 支持跨平台，目前测试windows/mac。
* 环境
  * 依赖jdk-1.8
* 依赖配置
  * \[generatorConfig.xml\] 配置数据库链接、Entity层、Dao层、Service层、ServiceImpl层、Controller层
  * \[petrel-gcode-pom.xml\] 自动升级配置，依赖maven
  * \[petrel-gcode.bat\] 、\[petrel-gcode.sh\] 运行命令，依赖petrel-gcode-pom.xml

---

## 使用方法

1. 在指定目录位置创建如下配置文件（[代码生成器配置文件模板](/kuo-zhan/dai-ma-sheng-cheng-qi/dai-ma-sheng-cheng-qi-pei-zhi-wen-jian-shuo-ming.md)）：![](/assets/代码生成器配置文件放置路径.png)

2. 打开文件所在目录，点击.bat/bsh启动代码生成器：  
   ![](/assets/代码生成器界面.png)

3. 顶部选择要生成的表类型（//TODO 查看单表多表单据区别）。

4. 左侧双击表名，将其置入已选列表区。

5. 已选列表区内双击表名，展示出可生成属性。

6. 编辑字段，调整字段名称、主键属性。

7. 选择要生成模板区块\[Entity层、Dao层、Service层、Controller层\]。

8. 点击生成，将会把相应代码放入配置文件中指定的目录。

> 已选择表必须双击出现字段属性内容,否则不会生成相应的代码。



