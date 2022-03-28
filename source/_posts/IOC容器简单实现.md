---
title: IOC容器简单实现
date: 2022-03-28 14:37:13
tags: [Spring,IOC]
index_img: img/person.jpg
categories: Spring

---

Spring IOC 对象实例化模拟实现

<!-- more -->

## 梳理思路



1. 创建配置文件
2. 创建工厂类，使用dom4j解析配置文件
3. 反射创建对象，根据id放入全局的map
4. 根据配置id获取class实例



## 实现步骤



### 一、创建配置文件，并固定配置文件dom节点

创建my_spring.xml,并创建两个java类配置进去

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<beans>

    <bean id="userDao" class="org.liumf.dao.UserDao"></bean>
    <bean id="userService" class="org.liumf.service.UserService"></bean>

</beans>
```

### 二、创建工厂

- 创建java Bean，配置文件的属性值包含 id和class两个属性，我们需要一个java bean去承载

  ```java
  @Getter
  @Setter
  @NoArgsConstructor
  @AllArgsConstructor
  public class MyBean {
  
      private String id;
      private String clazz;
  }
  
  ```

  

- 根据spring的配置获取方式分析，我们需要一个工厂,同时需要个构造器去传入文件名称

  ```java
   ApplicationContext ac = new ClassPathXmlApplicationContext("springContext.xml");
  ```



- 创建工厂接口

  ```java
  public interface MyFactory {
       Object getBean(String id);
  }
  ```

  

- 工厂实现

  ```java
  /**
   * 1.通过构造器传入要解析的xml
   * 2.解析配置文件得到对应的id和class,并放入集合中
   * 3.遍历集合得到每一个对象，实例化对象，并存入map
   * 4.根据id获取bean
   */
  public class MyClassPathXmlApplicationContext implements MyFactory {
  
      // 全局map,存放id和实例化对象
      private final Map<String,Object> map  =new HashMap(16);
  
      // 存放id和clazz 对象的映射关系
      private final List<MyBean> list = new ArrayList<>(16);
  
      public MyClassPathXmlApplicationContext() {
      }
  
      /**
       * 解析配置文件，得到实例化对象
       * @param fileName
       */
      public MyClassPathXmlApplicationContext(String fileName) {
  
          // 1.解析xml
          parseXml(fileName);
          // 2.反射实例化对象
          instanceBean();
      }
      /**
       * 1. 解析xml
       * @param fileName
       */
      private void parseXml(String fileName){
          SAXReader reader =new SAXReader();
          // 获取对应配置文件的地址
          URL resourceURL = this.getClass().getClassLoader().getResource(fileName);
          try {
              Document doc = reader.read(resourceURL);
              XPath xPath = doc.createXPath("beans/bean");
              List<Element> elements = xPath.selectNodes(doc);
              if (!CollectionUtils.isEmpty(elements)){
                  for (Element element : elements) {
                     this.list.add(new MyBean(element.attributeValue("id"), element.attributeValue("class")));
                  }
              }
          } catch (DocumentException e) {
              e.printStackTrace();
          }
  
      }
  
      /**
       * 2.实例化bean，并放入map中
       */
      private void instanceBean()  {
          if (null !=list && list.size()>0){
              for (MyBean myBean : list) {
                  try{
                      map.put(myBean.getId(), Class.forName(myBean.getClazz()).getDeclaredConstructor().newInstance());
                  }catch (Exception e){
                      e.printStackTrace();
                  }
              }
  
          }
      }
      /**
       * 3.从map中获取指定的bean
       * @param id
       * @return
       */
      @Override
      public Object getBean(String id) {
          return map.get(id);
      }
  }
  
  
  ```



## 测试

```java

    public static void main(String[] args) {

        MyFactory context = new MyClassPathXmlApplicationContext("my_spring.xml");

        UserDao userDao = (UserDao)context.getBean("userDao");

        userDao.userDaoTest();

    }
```





​	 
