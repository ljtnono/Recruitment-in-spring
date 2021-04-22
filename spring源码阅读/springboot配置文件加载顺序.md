## **1、默认搜索顺序和优先级顺序**

springboot**默认**会自动从以下四个路径加载配置文件

- **file:./config/\*/**
  jar包外部同级的config目录和其子目录的配置文件（**只包括第一层子目录**，config子目录的优先级比config目录优先级低，不同子目录，按照目录名升序优先级降低）
- **file:./**
  jar包外部同级目录的配置文件（不包含子目录）
- **classpath:/config/**
  jar包内部classes目录下config目录下的配置文件（不包含子目录）
- **classpath:/**
  jar包内部classes目录下的配置文件（不包含子目录）

springboot默认只加载**application.properties**或者**application.yml**文件，**优先级从上往下依次降低，如果是同一属性值，优先级高的会覆盖优先级低的，如果是不同属性则互补**

以上顺序是**优先级顺序**，**springboot的搜索顺序是相反的，**也就是说springboot默认的搜索顺序为

**1、classpath:/**

**2、classpath:/config/**

**3、file:./**

**4、file:./config/\*/**

**springboot加载配置文件遵循后来者胜利规则，在其他条件不变的情况下，始终以最后搜索到的配置作为最后配置**

## **2、yml与properties的优先级**

在同一目录下的**properties文件优先级高于yml文件，不同目录下以目录优先级高的为主**

**3、特定场景的配置文件（spring.profiles.active）**

如果设定了**spring.profiles.active**的值，那么application-{profile}文件的优先级要高于application文件的优先级（**无视目录之间的优先级差异，无视yml和properties后缀名的优先级差异**）

如果没有设定**spring.profiles.active**的值，**springboot会默认设置一个值，这个值是default**，也就是说默认情况下，application-default的优先级就比application的优先级高

### **3.1 多profile场景**

如果设定了多个spring,profiles.active值（以逗号隔开），那么**后面设定的优先级高于前面**

例如：如果设置了**spring.profiles.active=prd,default，**那么优先级依次为

**1、application-default文件**

**2、application-prd文件**

**3、application文件**

### 3.2 在不同文件中定义profile场景

springboot会按照**搜索顺序**找到**最先定义spring.profiles.active**的值放入一个set中，当其他非spring.profiles.active文件加载完毕后，再去加载spring.profiles.active值指定的配置文件

## **4、命令行参数优先级最高**

命令行参数指定环境变量优先级最高，总是会覆盖其他来源设置的环境变量，命令行多次设定同一个环境变量，后面设定的覆盖前面的

## **5、命令行参数指定配置文件目录**

springboot提供了两个命令行参数用于指定配置文件目录，分别是**--spring.config.location**和**--spring.config.additional-location**

- **--spring.config.location**
  如果指定了此命令行参数，**那么不会再加载默认目录下的配置文件**，且优先级按照指定位置的先后**依次升高**
  例如：**java -jar xxx.jar --spring.config.location=classpath:custom-config/,file:./custom-config/**
  那么优先级依次为
  **1、file:./custom-config/**
  **2、classpath:custom-config/**
- **--spring.config.additional-location**
  如果指定了此命令行参数，springboot会额外加载指定目录下的配置文件，且优先级按照指定位置的先后**依次升高**，并且指定路径下的配置文件优先于默认路径下的配置文件
  例如：**java -jar xxx.jar --spring.config.additional-location=classpath:/custom-config/,file:./custom-config/**
  那么优先级依次为
  **1、file:./custom-config/**
  **2、classpath:/custom-config/**
  **3、file:./config/\*/**
  **4、file:./**
  **5、classpath:/config/**
  **6、classpath:/**





## **补充：**

**1、修改配置文件名**

springboot默认配置文件名为application,可以通过**--spring.config.name**命令行参数修改配置文件的名字

example:  **java -jar myproject.jar --spring.config.name=myproject**

**2、使用systemctl方式启动时，需要主动指定额外读取配置文件路径，否则不会读取相对路径下config文件夹下的配置文件**

 --spring.config.additional-location=/oss/OSSWebBackend/config/