Solr全文检索，EHCache本地缓存，部分常用页面定制（403，404，500）
================
[![Release](https://img.shields.io/badge/build-springboot-green.svg)]()&nbsp;[![Build](https://img.shields.io/badge/release-2.0.0-blue.svg)]()&nbsp;[![Author](https://img.shields.io/badge/author-Xiaolong.Cao-yellow.svg)]()&nbsp;[![Time](https://img.shields.io/badge/time-2018.8.29-red.svg)]()&nbsp;

# Preface

> ### Frame新增对Solr全文检索引擎，EHCache缓存支持，分别在原生基础上进一步封装，提供CURD接口及实现；EHCache针对Mapper层数据缓存配置，Solr针对Oracl数据库同步配置；定制了部分常用的页面，诸如403，404，500。

完整产品介绍请参见 [Enjoyor-Springboot 框架](https://www.zybuluo.com/1528992118/note/1180138)，上期更新见[Frame-Update-Date-8.20](https://www.zybuluo.com/1528992118/note/1115387)，本次更新特性请参见 [Features 特性](#Features)，开发使用请参见 [Development](#Development)

<span id="Features"></span>
# Features
* ***Solr集成***
* **solr简介：**
> Solr采用`Lucene`搜索库为核心，提供全文索引和搜索开源企业平台，提供REST的HTTP/XML和JSON的API，本次168服务器上搭建的为`7.2.0`版本的solr，服务器地址: http://192.168.91.168:8983/solr

         
* **业务场景：** 
1. solr主要用于对其他存储系统中已有的一些数据做分析，查询，然后显示结果。当然它也可以直接存储数据，但是这不是它的强项
2. 对原有的数据做分析，需要对文本做一些处理（格式或者其他的），然后才导入到solr中，利用solr强大的搜索功能，找到自己想要的结果

* **使用须知：** 
solr中有个很重要的概念**core**，core是solr的一个索引库，可以理解为一个数据库，core可以根据需要，创建多个，其下有个**schema.xml**用于配置索引类型，可以理解为表结构，solr-5.5版本后没有了schema.xml，可以直接在在线配置schema，不再需要去改xml

* **Solr-CURD接口总览：**
  
   `public boolean save(String coreName, Object object)` 
   
   `public boolean save(String coreName, Map<String, Object> map)`
   
   `public boolean saveByReflect(String coreName, Object object)`
   
   `public int batchSave(String coreName, List<Object> list)`
   
   `public boolean deleteById(String coreName, String id)`
   
   `public int deleteById(String coreName, List<String> ids)`
   
   `public boolean deleteByQuery(String coreName, String query)`
   
   `public boolean deleteAll(String coreName)`
   
   `public SolrDocumentList selectAll(String coreName)`
   
   `public List<Object> selectAll(String coreName, Class<?> clazz)`
   
   `public SolrDocument selectById(String coreName, String id)`
   
   `public Object selectById(String coreName, String id, Class<?> clazz)`
   
   `public SolrDocumentList selectList(String coreName, String... filterQuerys)`
   
   `public List<Object> selectList(String coreName, Class<?> clazz, String... filterQuerys)`
   
   `public SolrDocumentList selectList(String coreName, String query, String... filterQuerys)`
   
   `public List<Object> selectList(String coreName, String query, Class<?> clazz, String... filterQuerys)`
   
   `public int selectCount(String coreName, String query, String... filterQuerys)`
   
   `public Page selectPage(String coreName, String query, Page page, String... filterQuerys)`
   
* 类位置：`com.enjoyor.soa.traffic.frame.support.solr.ISolrService`
* solr同步数据库，该场景可用于某些大数据表格，通过solr的全文索引技术，加快查询速度，其具体配置跳转 [Solr配置](#SolrConfig)

* **EHCache缓存**

* **EHCache简介：**
> 纯Java的进程内缓存框架，具有快速、精干等特点，可支持分布式，主要面向通用缓存,Java EE和轻量级容器。Spring提供了对缓存功能的抽象：即允许绑定不同的缓存解决方案（如Ehcache），但本身不直接提供缓存功能的实现。它支持注解方式使用缓存，非常方便。

* **业务场景：** 
1.单应用中直接对缓存进行存储读写，避免使用HashMap这种方式造成线程安全问题，或ConcurrentHashMap 造成的线程等待。
2.针对Dubbo-Server层进行缓存，提高某些Dubbo接口的查询速度。**（在使用前请确保已经完全理解缓存机制，避免其他内容更新后，查询的还是原来的旧数据）**
 
* **EHCache-CURD接口总览：**
  
  `public Object get(String cacheName, Object key);`
  
  `public boolean set(String cacheName, String key, Object value);`
  
  `public boolean removeCache(String cacheName);`
  
  `public boolean removeElment(String cacheName, String key);`

* 类位置：`com.enjoyor.soa.traffic.frame.support.cache.IEHCacheService`

* **定制页面**
* 这部分内容比较简单，简略叙述。针对原先系统自带的403，404，415及500错误返回页，定制了简易的新页面。
* 定制页面位置：`smart-traffic-frame/src/main/resources/public/error`
* 效果图，以404为例：
![404][1]
  
<span id="Development"></span>
# Development
## 1 Solr-CURD服务使用
### 1.1 使用准备
1.1.1 **使用须知**
> **使用前请确保solr服务器上有相应的core和core下对应的scheme已配置完成（可以类比为操作数据库前，必须创建数据库和相关表结构）,7.2版本可直接在相应core下在线完成配置，以168上的为例：http://192.168.91.168:8983/solr/#/template/schema**

1.1.2 **配置文件(application.properties)**

    spring.data.solr.host=http://192.168.91.168:8983/solr

1.1.2.1 配置文件说明

*spring.data.solr.host*   &nbsp;&nbsp;Solr服务器地址

1.1.3 **启动项注入配置类**

    @SpringBootApplication
    @Import({ SolrConfig.class })
    public class RestApplication 

### 1.2 使用

1.2.1 **com.enjoyor.soa.traffic.frame.support.solr.ISolrService**说明

**其下每个方法，都需要coreName，具体为索引库名，这里只说明一次；下面查询方法会反复出现`query`和`filterQuerys`，其用法可参考如下：**

>**基本查询**

 | 参数 | 说明 | 示例 |
 | ------ | ------ | ------ |
 | q (query) | 查询的关键字，此参数最为重 | q=id:1，默认为q=\*:* |
 | fq（filter query） | 过虑查询，提供一个可选的筛选器查询。返回在q查询符合结果中同时符合的fq条件的查询结果 | q=id:1&fq=sort:[1 TO 5]，找关键字id为1 的，并且sort是1到5之间的数据 |

>**Solr的检索运算符**

":"   &nbsp;&nbsp;指定字段查指定值，如返回所有值*:*

"?"   &nbsp;&nbsp;表示单个任意字符的通配

"*"   &nbsp;&nbsp;表示多个任意字符的通配（不能在检索的项开始使用*或者?符号）

"~"   &nbsp;&nbsp;表示模糊检索，如检索拼写类似于”roam”的项这样写：roam~将找到形如foam和roams的单       词；roam~0.8，检索返回相似度在0.8以上的记录。

AND、||  布尔操作符

OR、&&  布尔操作符

NOT、!、-（排除操作符不能单独与项使用构成查询）

"+"  存在操作符，要求符号”+”后的项必须在文档相应的域中存在²

( )  用于构成子查询

[]  包含范围检索，如检索某时间段记录，包含头尾，date:[201507 TO 201510]

{}  不包含范围检索，如检索某时间段记录，不包含头尾date:{201507 TO 201510}

　
<pre>
`public boolean save(String coreName, Object object)`
> 插入或修改数据：该方法用于被@Field注解的Object的保存,@Field用于适配Solr上的scheme.xml，如果有hibernate使用经验，应该很好理解。**(对于solr而言，其一个core相当于一个database,其下只冗余一张表，所以没有更新一说，save操作既是insert也是update)**

`public boolean save(String coreName, Map<String, Object> map)`
> 插入或修改数据：该方法比较容易理解，操作Map对象进行存储，map的key值具体对应solr服务器core端的字段名，value对应具体值，与前一个save方法相比，不需要创建具体对象。
    
`public boolean saveByReflect(String coreName,Object object)`
> 插入或修改数据：通过反射方式，根据对象类属性，自动完成适配，对应solr服务器core端的字段名，不需要@Field修饰每个类属性,若有@Field修饰,则优先采用Field中value值

`public int batchSave(String coreName, List<Object> list)`
> 批量存储(插入或和更新)：该方法为批量操作数据，被操作的Object需要@Field注解，用于匹配scheme.xml，返回操作成功记录数

 `public boolean deleteById(String coreName, String id)`
> 按Id号删除数据

`public int deleteById(String coreName, List<String> ids)`
> 按Id列表批量删除数据

`public boolean deleteByQuery(String coreName, String query)`
> 按自定义条件删除数据

` public boolean deleteAll(String coreName)`
> 删除索引库下所有数据，慎用

` public SolrDocumentList selectAll(String coreName)`
> 查询索引库下所有数据，返回结果为SolrDocumentList，其为SolrDocument的一个集合

`  public List<Object> selectAll(String coreName, Class<?> clazz)`
> 查询索引库下所有数据，返回结果为List<Object>，object类型根据clazz而具体实例化

`  public SolrDocument selectById(String coreName, String id)`
> 按id查询数据，返回结果为SolrDocument

`  public Object selectById(String coreName, String id, Class<?> clazz)`
> 按id查询数据，返回结果为根据clazz而具体实例化

`  public SolrDocumentList selectList(String coreName, String... filterQuerys)`
> 按过滤条件查询，filterQuerys可为null,为null则查询全部，返回结果为SolrDocumentList，其为SolrDocument的一个集合

`   public List<Object> selectList(String coreName, Class<?> clazz, String... filterQuerys)`
> 按过滤条件查询，filterQuerys可为null,为null则查询全部，返回结果为List<Object>，object类型根据clazz而具体实例化

`  public SolrDocumentList selectList(String coreName, String query, String... filterQuerys)`
> 按查询条件和过滤条件查询，query，filterQuerys可为null,为null则查询全部，返回结果为SolrDocumentList，其为SolrDocument的一个集合

`  List<Object> selectList(String coreName, String query, Class<?> clazz, String... filterQuerys)`
> 按查询条件和过滤条件查询，query，filterQuerys可为null,为null则查询全部，返回结果为List<Object>，object类型根据clazz而具体实例化

public int selectCount(String coreName, String query, String... filterQuerys)
> 按查询条件和过滤条件查询数量，query，filterQuerys可为null,为null则查询全部

public Page selectPage(String coreName, String query, Page page, String... filterQuerys)
按查询条件和过滤条件分页查询，query，filterQuerys可为null,为null则查询全部

</pre>



#### 1.2.2 使用案例：
    
    @Auth
    @RestController
    @RequestMapping({ "/api/template" })
    @Api("TemplateController相关api")
    public class TemplateController {
    
        @Autowired
        private ISolrService solrService;

        @GetMapping(value = { "/solrSelect/{id}" }, produces = { "application/json; 
        charset=UTF-8" })
        @ResponseBody
        @ApiOperation("按id查询")
        @ApiImplicitParam(name = "id", value = "id", defaultValue = "", paramType = "path", 
        required = true, dataType = "String")
        public ResultPojo solrSelect(@PathVariable("id") String id) {
             return new ResultPojo(solrService.selectById("template", id, SolrDemo.class));
        }

        @GetMapping(value = { "/solrSelectAll" }, produces = { "application/json; charset=UTF-8" 
        })
        @ResponseBody
        @ApiOperation("查询全部数据")
        public ResultPojo solrSelectAll() {
            return new ResultPojo(solrService.selectAll("template", SolrDemo.class));
        }

        @SuppressWarnings("rawtypes")
        @GetMapping(value = { "/solrSelectPage" }, produces = { "application/json; charset=UTF-8"
        })
        @ResponseBody
        @ApiOperation("分页查询")
        @ApiImplicitParam(name = "query", value = "query", paramType = "query", dataType = 
        "string")
        public ResultPojo solrSelectPage(Page page, @RequestParam(value = "query", required =    
        false) String query) {
            return new ResultPojo(solrService.selectPage("template", null, page, query));
        }

        @PostMapping(value = { "/solrInsert" }, consumes = { "application/json; charset=UTF-8" },
        produces = { "application/json; charset=UTF-8" })
        @ResponseBody
        @ApiOperation("添加数据")
        public ResultPojo solrInsert(@RequestBody SolrDemo solrDemo) {
             solrDemo.setCreateTime(new Date());
           return new ResultPojo(solrService.saveByReflect("template", solrDemo));
        }

    
        @DeleteMapping(value = { "/solrDelete/{id}" }, produces = { "application/json; 
        charset=UTF-8" })
        @ResponseBody
        @ApiOperation("按id删除数据")
        @ApiImplicitParam(name = "id", value = "id", paramType = "path", required = true, 
        dataType = "String")
        public ResultPojo solrDelete(@PathVariable("id") String id) {
           return new ResultPojo(solrService.deleteById("template", id));
        }

        @DeleteMapping(value = { "/solrDeleteAll/{coreName}" }, produces = { "application/json; 
        charset=UTF-8" })
        @ResponseBody
        @ApiOperation("删除全部数据")
        @ApiImplicitParam(name = "coreName", value = "coreName", paramType = "path", required = 
        true, dataType = "String")
        public ResultPojo solrDeleteAll(@PathVariable("coreName") String coreName) {
           return new ResultPojo(solrService.deleteAll(coreName));
        }
        
    }
    
    
## 2 EHCache本地缓存使用
### 2.1 使用准备
2.1.2 **配置文件(ehcache.xml)，位于项目resources目录下**

    <?xml version="1.0" encoding="UTF-8"?>
    <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">

        <diskStore path="java.io.tmpdir" /> 

        <defaultCache 
            eternal="false" 
            maxElementsInMemory="10000" 
            overflowToDisk="true" 
            diskPersistent="false"
            timeToIdleSeconds="120" 
            timeToLiveSeconds="120" 
            diskExpiryThreadIntervalSeconds="120" 
            memoryStoreEvictionPolicy="LRU">     
        </defaultCache>

        <cache 
            name="userCache" 
            eternal="false" 
            maxElementsInMemory="10" 
            overflowToDisk="true" 
            diskPersistent="false"
            timeToIdleSeconds="120" 
            timeToLiveSeconds="180" 
            memoryStoreEvictionPolicy="LRU" >
        </cache>
        
    </ehcache>

2.1.2.1 配置文件说明

**diskStore**   &nbsp;&nbsp;指定一个文件目录，当EHCache把数据写到硬盘上时，将把数据写到这个文件目录下，也可以指定为 `<diskStore path="D:\Data\ehcache" />`

**defaultCache**  &nbsp;&nbsp;默认的管理策略

**cache name="userCache"**  &nbsp;&nbsp;自定义的cache名，接下去的操作以以这个为准

**eternal**  &nbsp;&nbsp;true表示对象永不过期，此时会忽略timeToIdleSeconds和timeToLiveSeconds属性，默认为false

**maxElementsInMemory**  &nbsp;&nbsp;内存中最大缓存对象数，看着自己的heap大小来搞 

**overflowToDisk**  &nbsp;&nbsp;true表示当内存缓存的对象数目达到了maxElementsInMemory界限后， 会把溢出的对象写到硬盘缓存中。注意：如果缓存的对象要写入到硬盘中的话，则该对象必须实现了Serializable接口才行。

**diskPersistent**  &nbsp;&nbsp;是否缓存虚拟机重启期数据

**timeToIdleSeconds**  &nbsp;&nbsp;设定允许对象处于空闲状态的最长时间，以秒为单位。当对象自从最近一次被访问后， 如果处于空闲状态的时间超过了timeToIdleSeconds属性值，这个对象就会过期，EHCache将把它从缓存中清空。只有当eternal属性为false，该属性才有效。如果该属性值为0，则表示对象可以无限期地处于空闲状态

**timeToLiveSeconds**  &nbsp;&nbsp;设定对象允许存在于缓存中的最长时间，以秒为单位。当对象自从被存放到缓存中后， 如果处于缓存中的时间超过了timeToLiveSeconds属性值，这个对象就会过期，EHCache将把它从缓存中清除。只有当eternal属性为false，该属性才有效。如果该属性值为0，则表示对象可以无限期地存在于缓存中。timeToLiveSeconds必须大于timeToIdleSeconds属性，才有意义

**memoryStoreEvictionPolicy**     &nbsp;&nbsp;当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。可选策略有：**LRU**（最近最少使用，默认策略）、 **FIFO**（先进先出）、**LFU**（最少访问次数）

2.1.3 **启动项注入配置类**

    @SpringBootApplication
    @Import({ EhCacheConfig.class })
    public class RestApplication 

### 2.2 使用
2.2.1 **com.enjoyor.soa.traffic.frame.support.cache.IEHCacheService**说明
**其下每个方法，cacheName，具体为缓存名，具体在ehcache.xml以进行了配置**

 ` public Object get(String cacheName, Object key)`
> 获取缓存数据。


 `   public boolean set(String cacheName, String key, Object value)`
> 设置缓存数据，key值为缓存的键值。

`   public boolean removeCache(String cacheName)`
> 删除cacheName下所有缓存。

`   public boolean removeElment(String cacheName, String key)`
> 删除cacheName下具体key值对应的数据。

2.2.2 使用案例：
    @Auth
    @RestController
    @RequestMapping({ "/api/template" })
    @Api("TemplateController相关api")
    public class TemplateController {
    
         @Autowired
         private IEHCacheService eCacheService;

         @GetMapping(value = { "/cacheGet" }, produces = { "application/json; charset=UTF-8" })
         @ResponseBody
         @ApiOperation("查询数据")
         public ResultPojo cacheGet(@RequestParam(value = "cacheName", required = false) String 
         cacheName,@RequestParam(value = "key", required = false) String key) {
             return new ResultPojo(eCacheService.get(cacheName, key));
         }

         @PostMapping(value = { "/cacheSave" }, consumes = { "application/json; charset=UTF-8" },
         produces = {"application/json; charset=UTF-8" })
         @ResponseBody
         @ApiOperation("添加数据")
         public ResultPojo cacheSave(@RequestParam(value = "cacheName", required = false) String 
         cacheName,@RequestParam(value = "key", required = false) String key, @RequestBody     
         SolrDemo solrDemo) {
            return new ResultPojo(eCacheService.set(cacheName, key, solrDemo));
         }

         @DeleteMapping(value = { "/cacheDelete/{key}" }, produces = { "application/json;  
         charset=UTF-8" })
         @ResponseBody
         @ApiOperation("删除数据")
         @ApiImplicitParam(name = "key", value = "key", paramType = "path", required = true, 
         dataType = "String")
         public ResultPojo cacheDelete(@PathVariable("key") String key) {
            return new ResultPojo(eCacheService.removeElment("userCache", key));
         }
    
    }
    
### 2.3 dubbo-service（添加缓存，慎用）


    @SuppressWarnings("rawtypes")
    @Service(version = "1.0.0", timeout = 4000)
    public class DubboTemplateService implements IDubboTemplateService {
    
        @Autowired
        private ITemplateExampleService templateExampleService;
    
        @Cacheable(value="userCache",key="#id")
        public ResultPojo select(String id) {
            return new ResultPojo(templateExampleService.select(id));
        }
    
        @CachePut(key="'#templateDto.id", value="userCache")
        public ResultPojo insert(TemplateDto templateDto, String personName) {
            templateExampleService.insert((Template) DomainUtil.merge2Object(templateDto, new Template()), personName);
            return new ResultPojo();
        }
    
        @CachePut(key="'#templateDto.id", value="userCache")
        public ResultPojo edit(TemplateDto templateDto, String personName) {
            templateExampleService.edit((Template) DomainUtil.merge2Object(templateDto, new Template()), personName);
            return new ResultPojo();
        }
    
        @CacheEvict(key="#id", value="userCache")
        public ResultPojo delete(String id, String personName) {
            templateExampleService.delete(id, personName);
            return new ResultPojo();
        }
        
    }

2.3.1 注解说明：

**2.3.1.1 @Cacheable**
 >@Cacheable可以标记在一个方法上，也可以标记在一个类上。当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。对于一个支持缓存的方法，Spring会在其被调用后将其返回值缓存起来，以保证下次利用同样的参数来执行该方法时可以直接从缓存中获取结果，而不需要再次执行该方法。Spring在缓存方法的返回值时是以键值对进行缓存的，值就是方法的返回结果，至于键的话，Spring又支持两种策略，默认策略和自定义策略，这个稍后会进行说明。需要注意的是当一个支持缓存的方法在对象内部被调用时是不会触发缓存功能的。@Cacheable可以指定三个属性，value、key和condition
 
***2.3.1.2 @CachePut*** 
>在支持Spring Cache的环境下，对于使用@Cacheable标注的方法，Spring在每次执行前都会检查Cache中是否存在相同key的缓存元素，如果存在就不再执行该方法，而是直接从缓存中获取结果进行返回，否则才会执行并将返回结果存入指定的缓存中。@CachePut也可以声明一个方法支持缓存功能。与@Cacheable不同的是使用@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。
 
***2.3.1.3 @CacheEvict*** 
> @CacheEvict是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。@CacheEvict可以指定的属性有value、key、condition、allEntries和beforeInvocation。其中value、key和condition的语义与@Cacheable对应的属性类似。即value表示清除操作是发生在哪些Cache上的（对应Cache的名称）；key表示需要清除的是哪个key，如未指定则会使用默认策略生成的key；condition表示清除操作发生的条件。下面我们来介绍一下新出现的两个属性allEntries和beforeInvocation。

**详细资料请参考：https://blog.csdn.net/dreamhai/article/details/80642010**

<span id="SolrConfig"></span>
## 3 Solr数据库同步(以7.2版本为例)
### 3.1 使用准备
3.1.1 **相关下载及安装**

Solr下载地址：http://archive.apache.org/dist/lucene/solr/
Solr安装信息：http://www.cnblogs.com/tony-zt/p/9260017.html

**以上信息包含solr的下载及简单安装，比较基础，网上资源较多，这里只是给一个参考，我选取的7.2版本的Solr，7.4版本的启动会有日志报错。**

3.1.2 **数据库同步（这个配置有些地方会比较坑，故整理出来供参考）**
***注：以下配置，默认是以Jerry启动，若以Tomcat启动，请参考其他。***

***1,导包，测试表创建:***

>将oracle的驱动包导入到【YourDisk- Address\solr-7.2.0\server\solr-webapp\webapp\WEB-INF\lib】下，再将【YourDisk- Address\solr-7.2.0\dist】下的solr-dataimporthandler-7.2.0.jar和solr-dataimporthandler-extras-7.2.0.jar也复制到【YourDisk- Address\solr-7.2.0\server\solr-webapp\webapp\WEB-INF\lib】下 

![solr_1.png-39.6kB][2]


>在oracle数据库中建立一张新的表，表名随意，并添加一些数据进去，方面后面查询，这一步就不多说了，以下是我创建的表：
    
    create table TEMPLATE
    (
      id          VARCHAR2(30) not null,
      code        VARCHAR2(30) not null,
      type        VARCHAR2(30) not null,
      create_time DATE not null,
      update_time DATE,
      content     VARCHAR2(30) not null,
      delete_time DATE
    )
    comment on table TEMPLATE
      is '模板表（测试用）';
      
      
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('9', 'add', 'aa', to_date('24-08-2018 00:01:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('24-08-2018 11:00:00', 'dd-mm-yyyy hh24:mi:ss'), '1', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('10', 'BBB', 'qqq', to_date('24-08-2018 15:01:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('24-08-2018 15:01:00', 'dd-mm-yyyy hh24:mi:ss'), '12312', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('2', 'code_2', 'test', to_date('29-05-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试2', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('3', 'code_3', 'test', to_date('29-05-2018 14:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试3', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('4', 'code_4', 'test', to_date('29-05-2018 15:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试4', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('5', 'code_5', 'test', to_date('29-05-2018 16:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试5', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('6', 'code_6', 'test', to_date('30-05-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试6', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('7', 'code_7', 'test', to_date('30-05-2018 14:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试7', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('8', 'code_8', 'test', to_date('30-05-2018 16:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试8', null);
    insert into TEMPLATE (id, code, type, create_time, update_time, content, delete_time)
    values ('1', 'code_1', 'test', to_date('29-05-2018 11:00:00', 'dd-mm-yyyy hh24:mi:ss'), to_date('23-08-2018 13:00:00', 'dd-mm-yyyy hh24:mi:ss'), '测试1', null);

***2，修改配置文件（以下以我Template索引库为例，还没配置的可以参考上面2.1.1的安装下载地址）:***

>修改【YourDisk- Address\solr-7.2.0\server\solr\template\conf】目录下的solrconfig.xml文件，在<requestHandler name="/select" class="solr.SearchHandler">之上添加如下代码： 
      
       <requestHandler name="/dataimport" class="solr.DataImportHandler"> 
          <lst name="defaults"> 
            <str name="config">data-config.xml</str> 
          </lst> 
       </requestHandler>	

  
  ***3，在【YourDisk- Address\solr-7.2.0\server\solr\template\conf】目录下新建一个data-config.xml文件，并添加如下内容：***
 
>注：如果要用到分批导入数据，则必须配置**deltaImportQuery**和**deltaQuery**，**deltaQuer**y方法默认只能查询一个主键参数，不要多写，另外，注意 `pk="ID"，${dih.delta.ID}，<field column="ID" name="ID" />`  这几个值需要一致
  
        <dataConfig>
            <dataSource type="JdbcDataSource" driver="oracle.jdbc.driver.OracleDriver" 
             url="jdbc:oracle:thin:@192.168.91.153:1521:yfzx" user="prod_tiom" 
             password="prod_tiom" />
            <document name="tiom">
                <entity name="templat" pk="ID" 
        		 query="select ID,CODE,TYPE,CONTENT,TO_CHAR(UPDATE_TIME, 'YYYY-MM-DD HH24:MI:SS')                   as UPDATETIME from TEMPLATE"
        		 deltaImportQuery="select ID,CODE,TYPE,CONTENT,TO_CHAR(UPDATE_TIME, 'YYYY-MM-DD 
        	 	  HH24:MI:SS') as UPDATETIME from TEMPLATE where ID='${dih.delta.ID}'" 
                 deltaQuery="select ID FROM TEMPLATE where TO_CHAR(UPDATE_TIME, 'YYYY-MM-DD 
                  HH24:MI:SS') > '${dataimporter.last_index_time}'">
                    <field column="ID" name="ID" />  
                    <field column="code" name="CODE" /> 
                    <field column="type" name="TYPE" />  
                    <field column="content" name="CONTENT" />   			
                    <field column="updateTime" name="UPDATETIME" />     					
                </entity>
            </document>
        </dataConfig>


***4 重新启动Solr,选择template,点击左侧栏的schema，创建与data-config.xml相对应的Filed字段，再点击左侧栏的Dataimport，点击Execute，会显示indexing…，再点击Refresh Status，数据就导入进来了***
![solr2.png-51.2kB][3]
![solr3.png-40.7kB][4]
***5 结果查看***
![solr4.png-51.9kB][5]


[1]: http://static.zybuluo.com/1528992118/92a7gvm7lxhoaz7ozyb2ophy/404.png
[2]: http://static.zybuluo.com/1528992118/nbjj5ns70o95ig00joanqllk/solr_1.png
[3]: http://static.zybuluo.com/1528992118/7d6d8pe8u1zryduvzj0vn2d8/solr2.png
[4]: http://static.zybuluo.com/1528992118/9s47xjlmn44wj7oa54f0804z/solr3.png
[5]: http://static.zybuluo.com/1528992118/5kss38ljdpmuvo7l7op9plo8/solr4.png
