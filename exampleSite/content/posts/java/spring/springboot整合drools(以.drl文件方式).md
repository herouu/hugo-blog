---
title: Drools_1：springboot整合drools(以.drl文件方式)
date: 2018-12-15 16:10:39
tags: ["Drools"]

---

&emsp;&emsp;最近，在项目中，做还款的功能，需要根据还款方式（7 种）进行相应还款计算，需要判断的条件说多不多，说少不少。为了避免

```
if(){
    if(){
      ...if 嵌套
    }
  }
}
```

在一个方法内有多层 if 嵌套的情况，利用 java 设计模式-策略模式，根据还款方式的不同，去除了一层 if-else，然后将计算的部分放在一个 package,利用一个对象将计算的相关配置构建一个 po,代入 calculate(po)中,获取一个或多个结果对象用来处理业务逻辑。勉强使代码看起来简洁了一点，即时是这样，代码仍旧不是很美观，虽然功能是完成了，但是，假如判断的条件不断增加，if-else 也越来越多，代码也就变得难以理解与阅读。不禁提出了一个疑问，难道就没有什么框架或者工具，来替代我上面所做的工作吗？通过了解，发现通过规则引擎 drools 可以解决上面的问题，然后也就有了此文。

<!--more-->

&emsp;&emsp;了解之后，发现这个东西有两方面优势：1、其规则可以配置在数据库中，通过数据流的方式进行读取，比较适合规则经常变动的场景 2、可以通过 KIE 的 drools-workbench 管理规则（增加、修改），随着规则的越来越多，通过界面来管理得优势尽显。
&emsp;&emsp;什么是规则引擎？其实就是 when-then（if-else），即满足什么条件，做什么。

### springboot 中使用 drools

&emsp;&emsp;主要有两个方面：其一、drools 和 springboot 整合（通过.drl 文件配置规则或者通过数据库的方式配置规则）其二、drools 规则的方言（这是比较有用的部分，但是跟编码能力无关，即使不会代码的业务人员简单培训一下，感觉也可以配置规则）。自己会通过 3 篇文章简单介绍一下 drools,这是第一篇博文，以.drl 文件的方式配置规则，与 springboot 整合，剩下两篇分别是通过数据库方式整合 springboot、drools 规则方言。

#### 目录结构

![目录结构](<https://bj.bcebos.com/v1/alertcode-blog/Drools_1：springboot整合drools(以.drl文件方式)/springboot-drools-1.png>)

#### pom.xml

```xml
<!--Drools-->
   <dependency>
       <groupId>org.drools</groupId>
       <artifactId>drools-core</artifactId>
       <version>7.0.0.Final</version>
   </dependency>
   <dependency>
       <groupId>org.kie</groupId>
       <artifactId>kie-spring</artifactId>
       <version>7.0.0.Final</version>
   </dependency>
```

#### 实体类 product

&emsp;&emsp;3 个属性，规则文件根据 name 判断，然后向 discount、print 字段塞值。

```java
package top.alertcode.example.traindrools.drools.entity;

import lombok.Data;

@Data
public class Product {

    private String name;
    private int discount;
    private String print;

}
```

#### 配置 kmodule.xml

&emsp;&emsp;

```
<?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://www.drools.org/xsd/kmodule">
 <kbase name="productKB" packages="rules">     <!--packages 是.drl文件的 ‘package rules;’-->
       <ksession name="productSession"/>     <!-- ksession 负责po对象与规则进行交互的对象 -->
    </kbase>
</kmodule>
```

#### drl 规则文件 product.drl

```java
package rules; //与kmoudle package 关联
/*dialect设置规则当中要使用的语言类型 ，默认除了java还有mevl
  mvel是一种嵌入式脚本语言,在规则文件上可以用这种语言建立他们的断言、返回值、Eval和推论。
  mvel分解析模式(Interpreted Mode)和编译模式(Compiled Mode) */
dialect  "mvel"
//引入Product类
import top.alertcode.example.traindrools.drools.entity.Product
//规则名称
rule "product apple"
    when
//    判断name是否等于apple
//    proc 相当于product对象的引用
        proc:Product(name=="apple")
    then
//    满足条件进行处理的逻辑
        proc.setDiscount(15);
        proc.setPrint(proc.getName()+":"+proc.getDiscount());
end
rule "product orange"
    when
//    判断name是否等于orange
        proc:Product(name=="orange")
    then
        proc.setDiscount(20);
        proc.setPrint(proc.getName()+":"+proc.getDiscount());
     end

```

#### MyShopService

```java
package top.alertcode.example.traindrools.drools;

import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import top.alertcode.example.traindrools.drools.entity.Product;

@Service
public class MyShopService {

    /**
     * KieContainer： KieContainer就是一个KieBase的容器，可以根据kmodule.xml 里描述的KieBase信息来获取具体的KieSession
     */
    @Autowired
    private KieContainer kieContainer;

    public Product getProductDiscount(Product product) {
        KieSession kieSession = kieContainer.newKieSession("productSession");
        kieSession.insert(product);
        kieSession.fireAllRules();
        kieSession.dispose();
        return product;
    }
}
```

#### junit 验证

```java
package top.alertcode.example.traindrools.drools;

import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import top.alertcode.example.traindrools.TrainDroolsApplicationTests;
import top.alertcode.example.traindrools.drools.entity.Product;

import static org.junit.Assert.*;

public class MyShopServiceTest extends TrainDroolsApplicationTests {
    @Autowired
    private MyShopService myShopService;

    @Test
    public void getProductDiscount() {
        Product product = new Product();
        product.setName("apple");
        Product productDiscount = myShopService.getProductDiscount(product);
        System.out.println(productDiscount.toString());
        product.setName("orange");
        Product productDiscount1 = myShopService.getProductDiscount(product);
        System.out.println(productDiscount1.toString());
    }
}
```

#### 结果

```
Product(name=apple, discount=15, print=apple:15)
Product(name=orange, discount=20, print=orange:20)
```
