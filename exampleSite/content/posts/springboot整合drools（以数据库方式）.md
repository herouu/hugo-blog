---
title: Drools_2：springboot整合drools（以数据库方式）
date: 2018-12-17 23:42:29
tags: ["Drools"]

---

&emsp;&emsp;在实际应用中，很少使用.drl 文件的方式进行规则文件的编写，更多的是采用数据库的方式，将规则放到数据库的某一个字段里

<!--more-->

这里为了演示功能，简单的建了一张表。

### 建表语句如下

```sql
CREATE TABLE `drools_rules` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET latin1 DEFAULT NULL,
  `rule` text,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

### rule 字段存放规则，规则如下

```java
package base.rules;
dialect  "mvel"
import top.alertcode.example.traindrools.drools.entity.Product
rule "product apple"
    when
        proc:Product(name=="apple")
    then
        proc.setDiscount(45678);
        proc.setPrint(proc.getName()+":"+proc.getDiscount());
end
```

### 验证规则

#### 从数据库查询规则

&emsp;&emsp;使用 mybatis,druid 作为连接池，这里不再赘述。查询接口如下：

```java
package top.alertcode.example.traindrools.drools.mapper;

import org.apache.ibatis.annotations.Select;
import top.alertcode.example.traindrools.drools.entity.RuleEntity;

public interface RulesMapper {

  @Select("select * from drools_rules where id=#{id}")
   RuleEntity getRule(int id);

}
```

#### 规则实体类

```java
import lombok.Data;

/**
 * @author alertcode
 * @date 2018-12-17
 * @copyright alertcode.top
 */
@Data
public class RuleEntity {

  private Integer id;
  private String name;
  private String rule;
}
```

#### 主要代码

```java

/**
 * 经过规则验证后的对象
 *
 * @param id
 *     the id
 * @param product
 *     the product
 * @return the db product
 */
public Product getDBProduct(int id, Product product) {
  RuleEntity rule = rulesMapper.getRule(id);
  Assert.notNull(rule, "没有找到数据");
  return (Product) getDatabaseObject(rule, product);
}

/**
 * 根据字符串生成规则文件，KieSession进行读取的主要代码
 *
 * @param rule
 * @param object
 */
private Object getDatabaseObject(RuleEntity rule, Object object) {
  KieServices kieServices = KieServices.Factory.get();
  KieFileSystem kfs = kieServices.newKieFileSystem();
  kfs.write("src/main/resources/rules/rules" + System.currentTimeMillis() + ".drl",
      rule.getRule().getBytes());
  KieBuilder kieBuilder = kieServices.newKieBuilder(kfs).buildAll();
  Results results = kieBuilder.getResults();
  if (results.hasMessages(Level.ERROR)) {
    throw new IllegalStateException("### errors ###");
  }
  KieContainer kieContainer = kieServices
      .newKieContainer(kieServices.getRepository().getDefaultReleaseId());
  KieSession kieSession = kieContainer.newKieSession();
  kieSession.insert(object);
  kieSession.fireAllRules();
  kieSession.dispose();
  return object;
}
```

#### junit 验证

```java
@Test
public void getRule() {
    Product product = new Product();
    product.setName("apple");
    Product product1 = myShopService.getDBProduct(1, product);
    System.out.println(product1.toString());
    Assert.assertEquals(45678, product1.getDiscount());
```

#### 结果

![验证结果](https://bj.bcebos.com/v1/alertcode-blog/Drools_2：springboot整合drools（以数据库方式）/drools数据库-1.png)
