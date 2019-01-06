Frame（核心框架）文件操作，Druid数据源及SSO拦截排除更新
================

[![Release](https://img.shields.io/badge/build-springboot-green.svg)]()&nbsp;[![Build](https://img.shields.io/badge/release-2.0.0-blue.svg)]()&nbsp;[![Author](https://img.shields.io/badge/author-Xiaolong.Cao-yellow.svg)]()&nbsp;[![Time](https://img.shields.io/badge/time-2018.8.4-red.svg)]()&nbsp;


# Preface
> ### 本次文档针对Frame部分功能升级和完善，本次更新特性请参见 [Features 特性](#Features)，开发使用请参见 [Development](#Development)

<span id="Features"></span>
# Features
* ***轻量级文件处理功能***
  * 通过IFilesTool，实现对文件的上传，下载，删除功能
  
  * 不同于原先FTP方式，该功能基于服务器某文件夹（可配置指定）的映射，上传的文件位于应用所处服务器
  
  * 上传文件后，会返回一个可访问路径地址，针对各别.png，.txt文件不能在浏览器下载，会直接打开的问题， 提供了下载功能，将文件流输出值Response，通知浏览器下载
  
  * 类位置：com.enjoyor.soa.traffic.frame.support.file.IFilesTool

* ***排除特定Api的token验证***

  * 在特定场景下，某些API可能不需要Token验证，原先@Auth注解直接一刀切，注释在整个Controller上，拦截所有API这种做法欠妥

  * 本次更新，争对特定Controller下的某个API请求，@Auth注解新增excludes字段，可以有效排除具体API对于Token的验证

 * excludes字段是一个数组，支持多个API排除，排除的为Controller中的具体实现方法，而不是Mapping-URL

* ***Druid连接池说明***

 * frame中已依赖druid集成环境的starter包，直接在application.properties配置文件中进行相关设置，即可使用，无需其他操作。
  
<span id="Development"></span>
# Development
## 1 IFilesTool使用
### 1.1 使用准备
#### 1.1.1 配置文件

    file.handle.suffixs=jpg,png,word
    file.handle.file-dir=D://FileTest
    file.handle.file-addr=http://192.168.**.**:10200/tiom-rest
    file.handle.resource=fileResource/**
    file.handle.mapping-addr=fileResource
    
##### 1.1.1.2 配置文件说明
  *file.handle.suffixs*   &nbsp;&nbsp;表示支持的上传文件类型 
  *file.handle.file-dir*   &nbsp;&nbsp;表示文件上传的磁盘地址 
  *file.handle.file-addr*   &nbsp;&nbsp;表示文件上传成功后，返回应用所在服务器地址 
  *file.handle.resource*   &nbsp;&nbsp;结合上面的`file-dir`，表示`D://FileTest`下文件被`fileResource/**`映射 
*file.handle.mapping-addr* &nbsp;&nbsp;不能随便写，要和`file.handle.resource`制定的相同，如无必要`file.handle.file-addr`，`file.handle.resource`不需要修改

**备注：**实际开发中，需要修改的，基本只有`suffixs`，`file-dir`，`file-addr`

#### 1.1.2 启动项注入配置类

    @SpringBootApplication
    @Import({ FileHandleConfig.class })
    public class RestApplication 
    
### 1.2 使用

    @Auth(excludes = { "downloadFile" })
    @RestController
    @RequestMapping({ "/api/template" })
    @Api("TemplateController相关api")
    @CrossOrigin
    public class TemplateController{

        @Autowired
        private IFilesTool filesTool;

        @PostMapping(value = { "/uploadFile" }, consumes = "multipart/*",
        headers="content-type=multipart/form-data")
        @ApiOperation(value = "上传文件", notes = "上传文件")
        public ResultPojo uploadFile(@ApiParam(value = "上传的文件", required = true)
        MultipartFile multipartFile) {
             return filesTool.fileUpload(multipartFile);
        }  
    
        @DeleteMapping(value = { "/deleteFile" }, produces = { "application/json; charset=UTF-8" 
        })
        @ApiOperation(value = "删除文件", notes = "删除文件")
        public ResultPojo deleteFile(@RequestParam(value = "secondDir", required = true) String 
        secondDir,@RequestParam(value = "fileName", required = true) String fileName) {
           return filesTool.removeFile(secondDir, fileName);
        }
    
    
        @GetMapping("/downloadFile")
        @ApiOperation(value = "下载文件", notes = "下载文件")
        public void downloadFile(@RequestParam(value = "secondDir", required = true) String   
        secondDir,@RequestParam(value = "fileName", required = true) String fileName, 
        HttpServletResponse response) {
           filesTool.downloadFile(secondDir, fileName, response);
        }

    }
    
#### 1.2.1 使用说明 
**1）** 文件上传十分简单，直接调用注入的`IFilesTool` 的`fileUpload`方法，可结合**Swagger2-UI**页面进行测试，上传成功后会返回如下的`ResultPojo` ：

    {
      "appCode": "0",
      "dataBuffer": "成功",
      "resultList": "http://192.168.**.**:10200/tiom-rest/fileResource/20180804/2018080414400630
      .png"
    }
    
**2）** 文件删除，直接调用`IFilesTool`的`removeFile`方法，该方法只能删除之前通过配置文件配置的相关磁盘文件夹下的文件，也可以结合**Swagger2-UI**页面进行测试，不同于上传，此处需要输入2级文件夹名和文件名，拿之前上传的文件为例，删除成功会返回如下的`ResultPojo`：

     {
      "appCode": "0",
      "dataBuffer": "成功",
      "resultList": true
     }
**3）** 文件下载，主要是一些文件，如`PNG,TXT`等，输入全路径，浏览器会直接打开，这种情况下可调用`IFilesTool`的`downloadFile`方法，该方法也只能下载之前通过配置文件配置的相关磁盘文件夹下的文件，**不能结合Swagger2-UI页面进行测试**，所以需指定为GET方式，还需在`@Auth(excludes = { "downloadFile" })`中排除该方法的`token`验证，这样就能直接在浏览器输入API地址后进行测试，该方法能通过浏览器帮你完成下载。


## 2 @Auth中excludes的使用
### 2.1 直接以Controller为例
    @Auth(excludes = { "downloadFile" })
    @RestController
    @RequestMapping({ "/api/template" })
    @Api("TemplateController相关api")
    @CrossOrigin
    public class TemplateController{
    
        @Autowired
        private IFilesTool filesTool;
    
        @GetMapping("/downloadFile")
        @ApiOperation(value = "下载文件", notes = "下载文件")
        public void downloadFile(@RequestParam(value = "secondDir", required = true) String 
        secondDir,@RequestParam(value = "fileName", required = true) String fileName, 
        HttpServletResponse response) {
            filesTool.downloadFile(secondDir, fileName, response);
        }
    
    }
#### 2.1.1 说明
*@Auth(excludes = { "downloadFile" })*   &nbsp;&nbsp;**注解中excludes排除的是方法，而不是Mapping-Url**



## 3 Druid配置
### 由于在frame中使用了以下Maven依赖的缘故，所以无需进行其他的操作，只需要`application.properties`配置一些参数即可使用。
     <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>          
     </dependency>
### 3.1 application.properties配置
    #连接池配
    spring.datasource.druid.initial-size=1
    spring.datasource.druid.max-active=20
    spring.datasource.druid.min-idle=1
    # 配置获取连接等待超时的时间
    spring.datasource.druid.max-wait=60000 
    #打开PSCache，并且指定每个连接上PSCache的大小
    spring.datasource.druid.pool-prepared-statements=true
    spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
    #spring.datasource.druid.max-open-prepared-statements=和上面的等价
    spring.datasource.druid.validation-query=SELECT 'x'
    #spring.datasource.druid.validation-query-timeout=
    spring.datasource.druid.test-on-borrow=false
    spring.datasource.druid.test-on-return=false
    spring.datasource.druid.test-while-idle=true
    #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    spring.datasource.druid.time-between-eviction-runs-millis=60000
    #配置一个连接在池中最小生存的时间，单位是毫秒
    spring.datasource.druid.min-evictable-idle-time-millis=300000
    #spring.datasource.druid.max-evictable-idle-time-millis=
    #配置多个英文逗号分隔
    spring.datasource.druid.filters= stat,wall,log4j
    
    # WebStatFilter配置，说明请参考Druid Wiki，配置_配置WebStatFilter
    #是否启用StatFilter默认值true
    spring.datasource.druid.web-stat-filter.enabled=true
    spring.datasource.druid.web-stat-filter.url-pattern=/*
    spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*
    spring.datasource.druid.web-stat-filter.session-stat-enable=false
    spring.datasource.druid.web-stat-filter.session-stat-max-count=1000
    spring.datasource.druid.web-stat-filter.principal-session-name=admin
    spring.datasource.druid.web-stat-filter.principal-cookie-name=admin
    spring.datasource.druid.web-stat-filter.profile-enable=true

    # StatViewServlet配置
    #展示Druid的统计信息,StatViewServlet的用途包括：1.提供监控信息展示的html页面2.提供监控信息的JSON API
    #是否启用StatViewServlet默认值true
    spring.datasource.druid.stat-view-servlet.enabled=true
    #根据配置中的url-pattern来访问内置监控页面，如果是上面的配置，内置监控页面的首页是/druid/index.html例如：
    #http://110.76.43.235:9000/druid/index.html
    #http://110.76.43.235:8080/mini-web/druid/index.html
    spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
    #允许清空统计数据
    spring.datasource.druid.stat-view-servlet.reset-enable=true
    spring.datasource.druid.stat-view-servlet.login-username=admin
    spring.datasource.druid.stat-view-servlet.login-password=admin
    #StatViewSerlvet展示出来的监控信息比较敏感，是系统运行的内部情况，如果你需要做访问控制，可以配置allow和deny这两个参数
    #deny优先于allow，如果在deny列表中，就算在allow列表中，也会被拒绝。如果allow没有配置或者为空，则允许所有访问
    #配置的格式
    #<IP>
    #或者<IP>/<SUB_NET_MASK_size>其中128.242.127.1/24
    #24表示，前面24位是子网掩码，比对的时候，前面24位相同就匹配,不支持IPV6。
    spring.datasource.druid.stat-view-servlet.allow=
    spring.datasource.druid.stat-view-servlet.deny=128.242.127.1/24,128.242.128.1

    # Spring监控配置，说明请参考Druid Github Wiki，配置_Druid和Spring关联监控配置
    spring.datasource.druid.aop-patterns= #Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
    
    
### 3.2 application.properties说明
#### 通常来说，只需要修改`initialSize`、`minIdle`、`maxActive`，如果用**Oracle**，则把`poolPreparedStatements`配置为true，mysql可以配置为false。分库分表较多的数据库，建议配置为false。`removeabandoned`不建议在生产环境中打开，如果用SQL Server，建议追加配置。