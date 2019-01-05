## Enjoyor-Springboot 框架

***历史版本***
日期| 版本号| 作者
- | :-: | -:
2018/6/13 | V 1.0.0| 曹晓龙

## 1.前言
### 1.1 ***什么是SpringBoot***

    springboot就是一个大框架里面包含了许许多多的东西，其中spring就是最核心的内容之一，当然就包含spring mvc。

    spring mvc 是只是spring 处理web层请求的一个模块。
    
    spring mvc < spring <springboot。
    
    spring boot 我理解就是把 spring spring mvc ,spring data jpa等等的一些常用基础框架组合起来，
    提供默认的配置，然后提供可插拔的设计，就是各种starter ，来方便开发者使用这一系列的技术
    
    
### 1.2 ***比较传统项目优势***
- Spring Boot可以建立独立的Spring应用程序；
- 内嵌了如Tomcat，Jetty这样的容器，也就是说可以直接跑起来
- 无需再像Spring那样搞一堆繁琐的xml文件的配置
- spring 官方为其提供了一系列默认配置，可以自动配置Spring，往往只需在其application.properties文件   中默认配置一些相关声明就可以

### 1.3 ***示例展示*** 

    现在有项目A(Springboot)，在他的pom配置文件中添加相关依赖：
    
     <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>1.5.9.RELEASE</version>
     </dependency>
     
#### 1.3.1 spring-boot-starter-web包含依赖：

![springboot1.png-9.6kB][1]

     可以从上述案例图片中看到，这个最基础的springboot依赖，他是带有start相关的，按照官方的说法，这个依赖就是所谓的插拔式配置，无需像传统的spring一样配置xml,它会采用约定默认的方式，注入相关bean。
     
     那么这种配置怎么修改呢？
     
     比如上述的依赖，它已经依赖了tomcat,官方一般会默认给定一个端口号，如果你想修改其端口，只需在application.properties配置文件中修改其默认端口，如下：
     server.port=10200
     
#### 1.3.2 传统的配置怎么办？
     
     那如果没有相关start依赖呢，类似redis，log...这种常用的一般都会有相关start-pom；万一没有，或者由于其他某些原因，那可以通过@Configuration注入相关bean

### 1.4 ***运行springboot项目（以demo-rest为例）***
     找到相关主程序main函数，右键run as Java Application 即可
     
### 1.5 ***发布，运行***
     直接maven打包发布，将编译获得的jar直接java -jar xx.jar 就可以成功的运行项目

### 1.6  ***对开发者而言，意味着什么***

    Spring家族发展到今天，已经很庞大了，作为一个开发者，如果想要使用Spring家族一系列的技术，需要一个一个的搞配置，然后还有个版本兼容性问题，其实挺麻烦的，偶尔也会有小坑出现，其实蛮影响开发进度,spring boot就是来解决这个问题，提供了一个解决方案吧，可以先不关心如何配置，可以快速的启动开发，进行业务逻辑编写，各种需要的技术，加入starter就配置好了，直接使用，可以说追求开箱即用的效果吧.

## 2.框架结构

### 2.1 框架总览

- smart-traffic-frame (boot相关配置)
     - com.enjoyor.soa.traffic.frame
           - annotation（自定义的一些注解）
           - aop（controller,service拦截,日志输出）
           - configure（核心配置,mybatis,redis,Swagger2等）
           - constant
           - domain（日志的相关bean）
           - enums
           - dubbo（启用dubbo注解相关配置，重写AnnotationBean）
           - handler
               - exception（全局异常处理,最后在controller抛出restful异常）
               - mybatis（地图相关处理类）
           - interceptor（controller单点拦截，mybatis-sql拦截输出，分页处理）
           - logger（日志入库相关mapper,提取至公共部分）
           - resolver（异常具体处理类）
           - security（单点相关bean,针对前端页面单点拦截fifter）
           - support 
               - mybatis(基础增删改查，分页等抽象实现类)
               - redis (redis实现类,已注入 )
               - util （依赖spring相关jar包的util,区分smart-util下的util）

- smart-traffic-api（dubbo接口2.0）

- smart-traffic-util（原则上绝不能引入spring和dubbo相关jar）

- smart-traffic-app（在原先的基础上，做了一层隔离，防止污染上层项目）
     - demo-rest (dubbo消费方，为前台提供controller)
     - demo-core (数据库操作)
     - demo-server （dubbo服务提供方）
     - demo-web （前台web页面）



### 2.2 构建详情
2.2.1 ***基础依赖（上层依赖）***
<pre>
    smart-traffic-util  最上层jar
    
    smart-traffic-api 依赖 smart-traffic-util
    
    smart-traffic-frame 依赖  smart-traffic-util

</pre>

2.2.2 ***项目依赖（以demo为例）***

<pre>
demo-rest 依赖smart-traffic-frame,smart-traffic-api

demo-core 依赖smart-traffic-frame

demo-server 依赖 demo-core,smart-traffic-api

demo-web 依赖smart-traffic-frame
</pre>

2.2.3 ***demo-rest架构***

- common(常用包，包括缓存，枚举，方法类等)
  - cache(本地缓存常量)
  - enums
  - util(本地util)
- controller
- dubbo
  - impl（原manager,组装dubbo服务，为controller直接调用）
  - invoke（其他项目dubbo调用）
- listener(dubbo回调监听) 

2.2.4 ***demo-core架构***

- constant
<pre>
自定义sql常量包
</pre>
- domain
<pre>
mybatis相关domain
</pre>
- mapper
<pre>
mybatis相关mapper实现，整合原先mapper和dao层为一层
</pre>
- service
<pre>
原service
</pre>

2.2.5 ***demo-server架构***

- dubbo
  - impl 
  - invoke
<pre>
dubbo服务实现，其他dubbo调用
</pre>



## 3.项目配置
***properties配置***

&nbsp;&nbsp;&nbsp;3.1 *demo-server*

-  **oracl(application.properties)**
<pre>
    spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
    spring.datasource.url=jdbc:oracle:thin:@192.168.91.153:1521:yfzx
    spring.datasource.username=prod_idcs
    spring.datasource.password=prod_idcs
</pre>

-  **dubbo(application.properties)**
<pre>
    spring.dubbo.application.name=tiom_server
    spring.dubbo.registry.address=zookeeper://192.168.91.121:2181
    spring.dubbo.protocol.name=dubbo
    spring.dubbo.protocol.port=23202
    spring.dubbo.scan=com.enjoyor.soa.traffic.server.demo.dubbo
</pre>

-  **redis(application.properties)**
<pre>
    spring.redis.database=0
    spring.redis.host=127.0.0.1
    spring.redis.port=6379
    spring.redis.password=
    spring.redis.pool.max-active=8
    spring.redis.pool.max-wait=-1
    spring.redis.pool.max-idle=8
    spring.redis.pool.min-idle=0
    spring.redis.timeout=5000
</pre>

-  **mybatis(mybatis.properties)**
<pre>
   #base
   mybatis.type.aliasesPackage=com.enjoyor.soa.traffic.**.domain
   mybatis.type.handlersPackage=com.enjoyor.soa.traffic.frame.handler.mybatis
   #mapper
   mybatis.mapper.location=classpath*:/mybatis/**/*Mapper.xml
   #page
   mybatis.page.interceptor.showSql=true
   mybatis.page.interceptor.databaseType=oracl
</pre>


&nbsp;&nbsp;&nbsp;3.2 *demo-rest*


-  **redis(application.properties)**
<pre>
    spring.redis.database=0
    spring.redis.host=127.0.0.1
    spring.redis.port=6379
    spring.redis.password=
    spring.redis.pool.max-active=8
    spring.redis.pool.max-wait=-1
    spring.redis.pool.max-idle=8
    spring.redis.pool.min-idle=0
    spring.redis.timeout=5000
</pre>

-  **dubbo(application.properties)**
<pre>
    spring.dubbo.application.name=tiom_rest
    spring.dubbo.registry.address=zookeeper://192.168.91.121:2181
    spring.dubbo.scan=com.enjoyor.soa.traffic.rest.demo.dubbo,com.enjoyor.soa.traffic.frame.interceptor
</pre>

-  **sso(application.properties)**
<pre>
    sso.client.filter[startUp]=1
    sso.client.filter[server]=http://192.168.91.152:8081/uums-server/
    sso.client.filter[checkUrl]=http://192.168.91.152:8081/uums-server/token/check
    sso.client.filter[systemKey]=ceefba99-847b-4291-aee7-42bccc22bb53
    sso.client.filter[urlPatterns]=*.html
</pre>

-  **swagger2(swagger2.properties)**
<pre>
    #token-accept
    swagger2.sso.authorizeUri=http://localhost:10200/tiom-rest/html/authorize/authorize.html
    #basePackage
    swagger2.auto.scan.basePackage=com.enjoyor.soa.traffic.rest.demo
</pre>

&nbsp;&nbsp;&nbsp;3.3 *demo-web*

-  **sso(application.properties)**

<pre>
    sso.client.filter[startUp]=1
    sso.client.filter[server]=http://192.168.91.152:8081/uums-server/
    sso.client.filter[checkUrl]=http://192.168.91.152:8081/uums-server/token/check
    sso.client.filter[systemKey]=ceefba99-847b-4291-aee7-42bccc22bb53
    sso.client.filter[urlPatterns]=*.html
</pre>

## 4.开发指南

### 4.1 redis主要类(包)介绍说明
##### RedisConfig.java(com.enjoyor.soa.traffic.frame.configure下):
      配置redis
##### com.enjoyor.soa.traffic.frame.support.redis包：

     

      主要提供以下几种方法：

      //获取数据，参数key
      public String get(String key);

      //设置数据，参数为key和具体设置的值
      public Boolean set(String key, String value);

      //设置过期时间,参数为key和过期时间毫秒
      public boolean expire(String key, long expire);

      //设置对象,参数为key和对象类型
      public Object getObject(String key,Class<?> clz);

      //获取对象,参数为key和需存放的值
      public boolean setObject(String key, Object object);

      //设置数组,参数为key和需存放的list
      public <T> boolean setList(String key, List<T> list);

      //读取数组,参数为key和list容器中对象的类型
      public <T> List<T> getList(String key, Class<T> clz);

      //删除数据,参数为key
      public abstract Boolean del(final String key);




##### com.enjoyor.soa.traffic.frame.springboot.redis.tool包：
      redis实现类包，已注入，具体方法如上

### 4.1.1 redis使用示例
         @Autowired
         private IRedisTool redisTool;

         ...

         //设置redis
         redisTool.set("key1","value1");
         List<User> users = ...(实际根据业务需求获取)
         redisTool.setList("key2",users);
         User user = new User();
         redisTool.setObject("key3",user);

         //获取redis
         String value1=redisTool.get("key1");
         List<User> users=redisTool.getList("key2",User.class);
         User user=redisTool.getList("key3",User.class);


### 4.1.2 redis使用总结

      IRedisTool：分别提供了三种基于redisTemplate的设置和获取redis方法


### 4.2 mybatis主要类(包)介绍说明
##### MyBatisConfig.java/MyBatisMapperScannerConfig.java(com.enjoyor.soa.traffic.frame.configure下:
      配置mybatis
##### com.enjoyor.soa.traffic.frame.support.mybatis.mapper包：

     mybatis抽象类包，未注入，具体业务需继承此类，实现自己的业务逻辑

     主要提供以下几种方法：

     public abstract String getSqlNamespace();

     public abstract void setSqlNamespace(String paramString);

     public abstract T selectById(String paramString);

     public abstract List<T> selectList(T paramT);

     public abstract List<T> selectAll();

     public abstract void insert(T paramT);

     public abstract void insertInBatch(List<T> paramList);

     public abstract int updateById(T paramT);

     public abstract int updateByIdSelective(T paramT);

     public abstract void updateInBatch(List<T> paramList);

     public abstract void updateInBatchSelective(List<T> paramList);

     public abstract int deleteById(String paramString);

     public abstract void deleteByIdInBatch(String[] paramArrayOfString);

     public abstract boolean checkFieldExists(String paramString1, String paramString2, String paramString3);
     
     // 分页相关
     public abstract Page selectPageInfo(Page page);

     public abstract Page selectPageInfo(String sqlId, Page page);

     public abstract Page selectPageInfo(String sqlId, String countSqlId, Page page);

     public abstract int selectTotalCount(String sqlId, Map<String, Object> map);



### 4.2.1 mybatis使用示例

         1.接口继承：
         public interface IOperationHisMapper extends IBaseMapper<OperationHis> {}


         2.具体实现：
         @Service
         @Transactional
         public class OperationHisManager extends BaseMapper<OperationHis> implements  
         IOperationHisMapper {}

         3.实际使用
         @Autowired
         private IOperationHisMapper operationHisMapper;

         ...

         operationHisMapper.selectById(hisId)


### 4.2.2 mybatis使用总结

      使用前需在具体业务模块下，声明相关xml:

     <select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.String">
      select
         <include refid="Base_Column_List" />
      from OPERATION_HIS
         where HIS_ID = #{hisId,jdbcType=VARCHAR}
     </select>


### 4.3 SSO主要类(包)介绍说明
##### SSOFilterConfig.java (com.enjoyor.soa.traffic.frame.configure下)
      web,rest项目filter拦截配置
##### SecurityWebConfig.java (com.enjoyor.soa.traffic.frame.configure下)
      rest项目controller拦截配置
##### com.enjoyor.soa.traffic.frame.security包：
      该包下主要配置了前台拦截所需的实现类,SSOFilter.java
##### com.enjoyor.soa.traffic.frame.interceptor包：
      该包下主要配置了后台拦截所需的实现类,SSOInterceptor.java
### 4.3.1 SSO web项目使用示例
      1.application.properties配置（详见3.3)；
      2.启动类中SSOFilterConfig.class引入；
### 4.3.2 SSO rest项目使用示例
      1.application.properties配置（详见3.2）；
      2.启动类中InterceptorScannerConfig.class, SecurityWebConfig.class引入；
      3.@Auth 注解使用（SSOInterceptor会拦截带有@Auth注解的controller）：
      
        @Auth
        @RestController
        @RequestMapping({ "/api/operationhis" })
        @Api("OperationHisController相关api")
        @CrossOrigin
        public class OperationHisController {}
        
### 4.4 restful 接口
-  **GET**
<pre>

    @Autowired
    private ITemplateConsumeService templateConsumeService;

    @GetMapping(value = { "/select/{id}" }, produces = { "application/json; charset=UTF-8" })
    @ResponseBody
    @ApiOperation(value = "根据hisId查询操作记录", notes = "查询数据库中某条操作记录")
    @ApiImplicitParam(name = "id", value = "id", defaultValue = "", paramType = "path", 
    required=true, dataType = "String")
    public ResultPojo select(@PathVariable("id") String id, HttpServletRequest request) {
        return templateConsumeService.select(id);
    }
</pre>

- **POST**
<pre>

    @Autowired
    private ITemplateConsumeService templateConsumeService;
    
    @PostMapping(value = { "/create" }, consumes = { "application/json; charset=UTF-8" },  
    produces = { "application/json; charset=UTF-8" })
    @ResponseBody
    @ApiOperation("创建数据")
    public ResultPojo create(@RequestBody TemplateDto templateDto, HttpServletRequest request) {
        return templateConsumeService.insert(templateDto,
                ((UserDto) request.getAttribute("currentUser")).getPersonName());
    }
</pre>

- **PUT**
<pre>

    @Autowired
    private ITemplateConsumeService templateConsumeService;
    
    @PutMapping(value = { "/update" }, consumes = { "application/json; charset=UTF-8" })
    @ResponseBody
    @ApiOperation("更新数据")
    public ResultPojo update(@RequestBody TemplateDto templateDto, HttpServletRequest request) {
        return templateConsumeService.edit(templateDto,
                ((UserDto) request.getAttribute("currentUser")).getPersonName());
    }

</pre>

- **DELETE**
<pre>

    @Autowired
    private ITemplateConsumeService templateConsumeService;
    
    @DeleteMapping(value = { "/delete/{id}" }, produces = { "application/json; charset=UTF-8" })
    @ResponseBody
    @ApiOperation("删除数据")
    @ApiImplicitParam(name = "id", value = "id", paramType = "path", required = true, dataType = 
    "String")
    public ResultPojo delete(@PathVariable("id") String id, HttpServletRequest request) {
        return templateConsumeService.delete(id, ((UserDto)   
        request.getAttribute("currentUser")).getPersonName());
    }

</pre>

### 4.5 Dubbo (注解方式)
- **相关包扫描配置(application.properties)**
<pre>
spring.dubbo.scan=com.enjoyor.soa.traffic.server.demo.dubbo
</pre>

#### 4.5.1 服务提供

- **使用注解示例**
<pre>
@Service(version = "1.0.0", timeout = 4000)
public class DubboTemplateService implements IDubboTemplateService{
   ...
}
</pre>

#### 4.5.2 服务消费

- **使用注解示例**
<pre>
@Component
public class TemplateConsumeService implements ITemplateConsumeService {

    @Reference(version = "1.0.0", timeout = 3000)
    private IDubboTemplateService dubboTemplateService;
    
    public ResultPojo select(String id) {
        return dubboTemplateService.select(id);
    }
    
    ...
    
}
</pre>

#### 4.5.3 CallBack调用

- **使用注解示例**
<pre>
@Service(version = "1.0.0", timeout = 4000, callbacks = 1000)
public class DubboTestCallbackService implements IDubboTestCallbackService {

    @SuppressWarnings({ "unchecked", "rawtypes" })
    private Map<String, Long> hbLastList = new HashMap();
    
    private final Map<String, CallbackListener> listeners = new ConcurrentHashMap<String, CallbackListener>();

    public DubboTestCallbackService() {
       ...
    }
    
    @InterfaceMethods(methods = { @Method(name = "addTestListener", retries = 0, timeout = 3000,
    arguments = { @Argument(index = 1, callback = true) }) })
    public void addTestListener(String listenerId, CallbackListener listener) {
        ...
    }
    
    public boolean removeTestListener(String listenerId) {
      ...
    }

    public boolean TestHeartBeat(String listenerId) {
        ...
    }

}

</pre>


## 5.Swagger2 API Test

### 5.1 swagger-ui(以demo-rest为例)
##### 5.1.1 访问路径
      http://localhost:10200/demo-rest/swagger-ui.html
##### 5.1.2 Token获取
      http://localhost:10200/demo-rest/html/authorize/authorize.html
##### 5.1.3 Token页面
      直接点击按钮，即可复制至剪贴板
##### 5.1.4 单点认证
      输入刚才获取的token
### 5.1.5 controller测试
      选择具体controller开始测试

***示例图***：

![springboot2.png-57.8kB][2]

[1]: http://static.zybuluo.com/1528992118/ueczvioywrtxjxnnnn9uboit/springboot1.png
[2]: http://static.zybuluo.com/1528992118/ymwkltkexicu8eeaz99oep2p/springboot2.png
