---
title: 微服务迁移过程需要注意的地方
date:  2022-06-22 16:39:23 +0800
category: Original
tags: [Microservices,RPC,Feign]
excerpt: 微服务迁移过程
---

* content
{:toc}

1. #### 【调用与响应】

   feign（Http远程方法调用）**标准化Response响应体**（这里定为BaseFeignResponse）

   * 用于适配**被调用端**Server的**响应体结构**，使其既能提供RPC的服务也能给前端提供服务

     ```java
     @Data
     public class BaseFeignResponse<T> implements Serializable {
     
         private static final long serialVersionUID = 1L;
     
         /**
          * 响应状态码
          */
         private String code;
     
         /**
          * 响应描述
          */
         private String msg;
     
         /**
          * 响应业务数据
          */
         private T data;
     
     }
     ```

     ```json
     {"code":"SUCCESS","msg":"success","data":"success"}
     ```

   * 定义好响应体之后，需要通过**切面**Aspect对RestController作**增强处理**：将调用结果转成目标体，并做好空值与异常的自适应

     ```java
     @Aspect
     @Component
     @Slf4j
     public class HadesFeignAspect {
     
         public static final String SUCCESS = "SUCCESS";
     
         @Pointcut("execution(* com.shop*hyiki*.plutus.app.hades.logistics.*.*(..))")
         private void returnResult() {
         }
     
         @Around("returnResult()")
         private Object handleReturnResult(ProceedingJoinPoint pjp) throws Throwable {
             try {
                 Object proceed = pjp.proceed();
                 /* null值 */
                 if (proceed == null) {
                     return null;
                 }
                 /* void */
                 Signature s = pjp.getSignature();
                 MethodSignature ms = (MethodSignature)s;
                 Method m = ms.getMethod();
                 if (m.getReturnType() == Void.class) {
                     return proceed;
                 }
                 /* Response */
                 if (m.getReturnType() == Response.class) {
                     return proceed;
                 }
                 /* 标准转换 */
                 BaseFeignResponse feignResponse = (BaseFeignResponse) proceed;
                 if (log.isDebugEnabled()) {
                     log.debug(feignResponse.toString());
                 }
                 if (Objects.equals(feignResponse.getCode(), SUCCESS)) {
                     return feignResponse;
                 } else {
                     throw new BizException(feignResponse.getCode(), feignResponse.getMsg());
                 }
             } catch (FeignException e) {
                 log.error("feign exception, body:{}", e.contentUTF8(), e);
                 throw new BizException("调用外部系统异常:" + e.getMessage());
             } catch (BizException e) {
                 log.error("feign bizException", e);
                 throw e;
             } catch (Exception e) {
                 log.error("feign system exception", e);
                 throw e;
             }
     
         }
     }
     ```

2. #### 【Response的解码器】executeAndDecode(RequestTemplate template)

   ```java
   public interface Decoder {
   
     /**
      * Decodes an http response into an object corresponding to its
      * {@link java.lang.reflect.Method#getGenericReturnType() generic return type}. If you need to
      * wrap exceptions, please do so via {@link DecodeException}.
      *
      * @param response the response to decode
      * @param type {@link java.lang.reflect.Method#getGenericReturnType() generic return type} of the
      *        method corresponding to this {@code response}.
      * @return instance of {@code type}
      * @throws IOException will be propagated safely to the caller.
      * @throws DecodeException when decoding failed due to a checked exception besides IOException.
      * @throws FeignException when decoding succeeds, but conveys the operation failed.
      */
     // ☆☆☆☆☆☆☆☆☆☆☆☆这句是关键代码☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
     Object decode(Response response, Type type) throws IOException, DecodeException, FeignException;
   
     /** Default implementation of {@code Decoder}. */
     public class Default extends StringDecoder {
   
       @Override
       public Object decode(Response response, Type type) throws IOException {
         if (response.status() == 404)
           return Util.emptyValueOf(type);
         if (response.body() == null)
           return null;
         if (byte[].class.equals(type)) {
           return Util.toByteArray(response.body().asInputStream());
         }
         return super.decode(response, type);
       }
     }
   }
   ```

   ```java
   /* FeignConfig - ErrorDecoder */
   @Bean
   public *hyiki*ErrorDecoder errorDecoder() {
     return new *hyiki*ErrorDecoder();
   }
   
   /**
   * 自定义错误解码器
   */
   public static class *hyiki*ErrorDecoder extends ErrorDecoder.Default {
   
     @Override
     public Exception decode(String methodKey, Response response) {
       if (response.status() == HttpServletResponse.SC_BAD_REQUEST) {
         // 自定义异常
         BizException exception = null;
         try {
           // 获取原始的返回内容
           String json = Util.toString(response.body().asReader());
           String msg = JSONUtil.parseObject(json, *hyiki*Response.class).map(*hyiki*Response::getMsg).orElse(json);
           // 抛出业务异常
           exception = new BizException(msg);
         } catch (IOException ex) {
           log.error(ex.getMessage(), ex);
         }
         return exception;
       }
       return super.decode(methodKey, response);
     }
   }
   ```

3. #### 【签名】在Feign客户端的FeignConfig发起**apply**，将Signature签名注入到RequestHeader当中

   ```java
   /* FeignConfig */
   @Override
   public void apply(RequestTemplate requestTemplate) {
   
     String timestamp = String.valueOf(System.currentTimeMillis());
     requestTemplate.header("key", dionysusServiceSign.getKey())
       .header("timestamp", timestamp)
       .header("signature", dionysusServiceSign.getSignature(timestamp));
   }
   ```

4. #### 【文件接收】关键点在于将**RPC的输出流**作为输入流，**asInputStream**写入**响应服务的输出流**

   * feign.Response ：An immutable response to an http invocation which only returns string content.
   * 只返回String内容的响应会视为feign.Response对象：文件下载

   ```java
   /** String -> feign.Response **/
   @Slf4j
   public class FeignUtils {
   
       /**
        * 通过feign下载文件 (feign.Response)
        * An immutable response to an http invocation which only returns string content.
        * 
        *
        * @param response
        * @param feignResponseSupplier
        * @throws IOException
        */
       public static void downloadFile(HttpServletResponse response, ResponseSupplier<Response> feignResponseSupplier) throws Exception {
           downloadFile(response, feignResponseSupplier.request());
       }
   
       /**
        * 通过feign下载文件
        *
        * @param response
        * @param serviceResponse
        * @throws IOException
        */
       public static void downloadFile(HttpServletResponse response, Response serviceResponse) throws IOException {
           InputStream inputStream = null;
           Response.Body body = serviceResponse.body();
           inputStream = body.asInputStream();
           BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
           response.setHeader("Content-Disposition", serviceResponse.headers().get("Content-Disposition").toString().replace("[", "").replace("]", ""));
           BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(response.getOutputStream());
           int length = 0;
           byte[] temp = new byte[1024 * 10];
           while ((length = bufferedInputStream.read(temp)) != -1) {
               bufferedOutputStream.write(temp, 0, length);
           }
           bufferedOutputStream.flush();
           bufferedOutputStream.close();
           bufferedInputStream.close();
           inputStream.close();
       }
   
       /**
        * 响应方法
        * @param
        */
       public interface ResponseSupplier<T> {
           /**
            * 下载文件
            *
            * @return
            * @throws Exception
            */
           T request() throws Exception;
       }
   }
   ```

5. #### 【文件上传】遇到的已知问题：the request was rejected because no multipart boundary was found

   * 分析问题：

     * StandardMultipartHttpServletRequest#parseRequest -> contentType: multipart/form-data

     * FileUploadBase#getBoundary -> return null

       ```java
       public byte[] getBoundary(String contentType) {
         ParameterParser parser = new ParameterParser();
         parser.setLowerCaseNames(true);
         Map<String, String> params = parser.parse(contentType, new char[]{';', ','});
         String boundaryStr = (String)params.get("boundary");
         if (boundaryStr == null) {
           return null;
         } else {
           byte[] boundary = boundaryStr.getBytes(StandardCharsets.ISO_8859_1);
           return boundary;
         }
       }
       ```

     * FileItemIteratorImpl#init -> throw Exception

       ```java
       multiPartBoundary = fileUploadBase.getBoundary(contentType);
       if (multiPartBoundary == null) {
         IOUtils.closeQuietly(input); // avoid possible resource leak
         throw new FileUploadException("the request was rejected because no multipart boundary was found");
       }
       ```

   * 解决问题：

     * **@PostMapping**注解属性添加**consumes**类型 & 文件类型前的**@RequestPart** 注解

       ```java
       @PostMapping(value = "/upload", consumes = MULTIPART_FORM_DATA_VALUE)
       @*hyiki*ResponseBody
       BaseFeignResponse<String> logisticsRead(@RequestPart("file") MultipartFile file,
                                               @RequestParam("courierCompanyId") Long courierCompanyId,
                                               @RequestParam("templateType") String templateType,
                                               @RequestParam(value = "sheetName", required = false) List<String> sheetName) throws IOException;
       ```

     * 【注意】值得注意的点是序列化Json作为日志输出时，流相关的属性不能被操作：因此需要作filter处理

       ```java
       Object[] parameters = pjp.getArgs();
       if (log.isInfoEnabled() && parameters != null) {
         List<Object> params = Arrays.stream(parameters)
           .filter(o -> !(o instanceof InputStreamSource)
                   && !(o instanceof ServletResponse)
                   && !(o instanceof ServletRequest))
           .collect(Collectors.toList());
         log.info("class:{}, method:{}, parameters :{}", className, methodName, JSON.toJSONString(params));
       }
       ```

6. #### 【server端服务关闭】server端为保证服务的稳定，当前服务节点从注册中心内摘除应**先于**Spring容器的关闭

   * 【环境架构】依赖组件：spring-boot + spring-cloud-starter-openfeign + spring-cloud-starter-zookeeper-all

     1. <spring-boot.version>2.4.1</spring-boot.version>
     2. <openfeign.version>3.0.0</openfeign.version>
        * spring-cloud-openfeign-core:3.0.0
        * spring-cloud-netflix-ribbon:3.0.0
     3. <zookeeper.version>3.0.0</zookeeper.version>

   * 【背景】服务下线**后于**容器关闭会导致的问题：
     请求进来，由于注册中心中还残留节点（未完全下线），**负载均衡会把请求分发给存活的服务节点，但是实际上该节点的容器已经关闭，无法提供正常服务**，最终导致这个请求会被拒绝，从而影响了用户的使用体验。

   * 【目标】**避免请求会被拒绝**，应在容器关闭前将节点从注册中心下线，那么在下线之前容器未完全关闭，服务正常提供，当节点真正下线后，负载均衡不会将请求分发到即将下线的节点时，此时才进行服务容器的关闭，这种做法可以支撑节点**优雅(graceful)**关闭，从而进行维护操作：代码更新发版、单节点的临时维护或事故。

   * 【做法】

     * 【前提】客户端的参数配置——服务节点列表的缓存问题
       值得注意：节点列表缓存间隔**ServerListRefreshInterval**
       换句话说，就是每一个时间间隔，客户端会从注册中心刷新一次服务节点列表；那么在缓存时间内，如果有节点进行下线，ribbon也会通过负载均衡将请求分发到下线节点上，导致出现服务拒绝异常。

       ```yaml
       # client
       feign:
         client:
           config:
             default:
               connectTimeout: 10000 #ms
               readTimeout: 60000 #ms
       
       ribbon:
         ServerListRefreshInterval: 1000
         ReadTimeout: 5000
         ConnectTimeout: 2000
         MaxAutoRetries: 0 #同一台实例最大重试次数,不包括首次调用
         MaxAutoRetriesNextServer: 0 #重试负载均衡其他的实例最大重试次数,不包括首次调用
         OkToRetryOnAllOperations: false  #是否所有操作都重试
       ```

     * 【事件监听】从Spring容器的事件监听入手：**ApplicationEvent**

       ```java
       package org.springframework.context;
       
       import java.util.EventObject;
       
       public abstract class ApplicationEvent extends EventObject {
           private static final long serialVersionUID = 7099057708183571937L;
           private final long timestamp = System.currentTimeMillis();
       
           public ApplicationEvent(Object source) {
               super(source);
           }
       
           public final long getTimestamp() {
               return this.timestamp;
           }
       }
       ```

     * 【监听入口】从抽象类**ApplicationEvent**中列出实现@Override

     * 【注入容器】定义**@Component**组件注入用于事件监听的Bean：示例代码

       ```java
       public class SpringEventListener implements ApplicationListener<E extends ApplicationEvent>, TomcatConnectorCustomizer
       ```

     * 【选择事件】Spring容器常用的事件监听：

       ```java
       public abstract class ApplicationContextEvent extends ApplicationEvent
       ```

       【**Refreshed 对应 Closed 、Started 对应 Stopped**】

       * **ContextRefreshedEvent**
         在**初始化或刷新ApplicationContext**时发布（例如，通过使用ConfigurableApplicationContext接口上的refresh（）方法）。在这里，“已初始化”是指所有Bean都已加载，检测到并激活了后处理器Bean，已预先实例化单例并且可以使用ApplicationContext对象。只要尚未关闭上下文，只要选定的ApplicationContext实际上支持这种“热”刷新，就可以多次触发刷新。例如，XmlWebApplicationContext支持热刷新，但GenericApplicationContext不支持。
       * **ContextStartedEvent**
         **使用ConfigurableApplicationContext接口上的start（）方法启动ApplicationContex**t时发布。此处，“已启动”表示所有Lifecycle bean都收到一个明确的启动信号。通常，此信号用于在显式停止后重新启动Bean，但也可以用于启动尚未配置为自动启动的组件（例如，尚未在初始化时启动的组件）。
       * **ContextStoppedEvent**
         **通过使用ConfigurableApplicationContext接口上的stop（）方法停止ApplicationContext**时发布。在这里，“已停止”表示所有Lifecycle bean都收到一个明确的停止信号。停止的上下文可以通过start（）调用重新启动。
       * **ContextClosedEvent**
         **通过使用ConfigurableApplicationContext接口上的close（）方法关闭ApplicationContext**时发布。在此，“封闭”表示所有单例Bean都被破坏。封闭的情境到了生命的尽头。无法刷新或重新启动。

     * 【解决方案】容器关闭前从注册中心摘除节点
       实际做法：【重点】借助**ContextClosedEvent**，在Spring容器关闭前摘除注册中心服务节点

       ```yaml
       server:
         port: 8080
       #  优雅关闭
         shutdown: graceful
       ```

       ```java
       @Configuration
       public class StopConfig {
       
           @Bean
           public SpringClosedListener gracefulShutdown() {
               return new SpringClosedListener();
           }
       
           @Bean
           public ConfigurableServletWebServerFactory webServerFactory(final SpringClosedListener springStopListener) {
               TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
               factory.addConnectorCustomizers(springStopListener);
               return factory;
           }
       }
       ```

       ```java
       public class SpringStopListener implements ApplicationListener<ContextClosedEvent>, TomcatConnectorCustomizer {
       
           @Autowired
           RedissonClient redisClient;
       
           @Autowired
           private ZookeeperServiceRegistry serviceRegistry;
       
           private static final int TIMEOUT = 30;
       
           /**
            * tomcat才填充，对于本地测试实例存在不启用tomcat
            */
           private volatile Connector connector;
       
           @Override
           public void customize(Connector connector) {
               this.connector = connector;
           }
       
           @Override
           public void onApplicationEvent(ContextClosedEvent event) {
               // Zookeeper 优雅下线服务，先摘除当前服务注册再关闭容器
       //        log.info("ZookeeperServiceRegistry close");
               serviceRegistry.close();
               // 等待5s，防止调用方的缓存导致拒绝访问：等待时间 > 缓存时间
               try {
                   Thread.sleep(5000);
               } catch (InterruptedException e) {
       //            log.error("InterruptedException", e);
               }
       
               Executor executor = null;
               if (Objects.nonNull(this.connector)) {
                   this.connector.pause();
                   executor = this.connector.getProtocolHandler().getExecutor();
               }
       
               // 释放redis锁
       //        RLock lock = redisClient.getLock("");
       //        if(lock.isLocked()) {
       //            lock.unlock();
       //        }
       
               
               // 优雅关闭线程池
               if (executor instanceof ThreadPoolExecutor) {
                   try {
                       ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
                       // 将状态设置为shutdown,不再接收新的请求，正在跑的任务会执行完
                       threadPoolExecutor.shutdown();
                       if (!threadPoolExecutor.awaitTermination(TIMEOUT, TimeUnit.SECONDS)) {
       //                    log.warn("Tomcat thread pool did not shut down gracefully within "
       //                            + TIMEOUT + " seconds. Proceeding with forceful shutdown");
       
                           // 如果在规定的时间30s之内还未处理完成，那么直接关闭
                           threadPoolExecutor.shutdownNow();
                           if (!threadPoolExecutor.awaitTermination(TIMEOUT, TimeUnit.SECONDS)) {
       //                        log.error("Tomcat thread pool did not terminate");
                           }
                       }
                   } catch (InterruptedException ex) {
                       Thread.currentThread().interrupt();
                   }
               }
       
           }
       }
       ```

##### 【RequestBody参数顺序】feign限制了参数顺序，RequestBody在前

##### 【PathVariable使用方式】 PathVariable的value必须写，且path上不能有格式，如(/{id:\\d+})不行，需要(/{id})

```java
/* Feign Client Service */
// @PostMapping("/{id:\\d+}")
@PostMapping("/{id}")
@*hyiki*ResponseBody
BaseFeignResponse<ActionEnum> update(@RequestBody @Valid RuleSaveOrUpdateDto ruleDto, @PathVariable("id") Long id);
```

##### 1. 【GET请求格式】feign如果不标识@RequestParam注解，即使指定了GET方法，feign依然会以POST方法发送请求

```java
/* Feign Client Service */
@GetMapping("/tracking/refresh")
@*hyiki*ResponseBody
BaseFeignResponse<LogisticsNumberDTO> refreshLogisticsTracking(@RequestParam("logisticsNumber") String logisticsNumber);
```

​ 2. 【@RequestBody报错】已知POST请求进行对象的传递，需要在传递的对象类前加@RequestBody注解

```java
/* Feign Client Service */
@RequestMapping(value = "/department/updateByExampleSelective",method = RequestMethod.PUT)
int updateByExampleSelective(@RequestBody final Department record, @RequestBody final DepartmentExample example);
```

报错信息：

```java
Caused by: java.lang.IllegalStateException: Method has too many Body parameters: 
```

解决方式：

* 使用@RequestParam代替@RequestBody，在某些地方是能够实现的，具体还得看状况对象

* 【不推荐】将接收参数定义为Map<String, Object>，而后使用map转object工具，转换成须要的对象。

7. #### fastxml解析会用xml的解析？？？FeignConfig配置fastjson解析

   ```java
   @Bean
   public Encoder feignEncoder(){
     return new SpringEncoder(feignHttpMessageConverter());
   }
   
   @Bean
   public Decoder feignDecoder(){
     return new SpringDecoder(feignHttpMessageConverter());
   }
   
   /**
        * feign和Springboot使用的都是jackson,可以都修改为fastjson解析方式
        */
   private ObjectFactory<HttpMessageConverters> feignHttpMessageConverter() {
     final HttpMessageConverters httpMessageConverters = new HttpMessageConverters(this.getFastJsonConverter());
     return () -> httpMessageConverters;
   }
   
   private FastJsonHttpMessageConverter getFastJsonConverter() {
     CustomFastJsonHttpMessageConverter converter = new CustomFastJsonHttpMessageConverter();
     List<MediaType> supportedMediaTypes = new ArrayList<>();
     MediaType mediaTypeJson = MediaType.valueOf(MediaType.APPLICATION_JSON_UTF8_VALUE);
     supportedMediaTypes.add(mediaTypeJson);
     FastJsonConfig config = new FastJsonConfig();
     config.getSerializeConfig().put(JSON.class, new SwaggerJsonSerializer());
     config.setSerializerFeatures(SerializerFeature.DisableCircularReferenceDetect);
     converter.setFastJsonConfig(config);
     converter.setSupportedMediaTypes(supportedMediaTypes);
     return converter;
   }
   ```

8. ##### Springboot-maven引入其他模块无法扫描到spring bean的问题（遇到过新建rpc模块无法扫描@FeignClient）

   * 解决方法：将A模块和B模块的Application置于相同路径下，例如com.xx下
   * 启动模块的包路径在com.*hyiki*.hades.***,新建模块的代码也需要放在这个路径下

9. **rpc 请求过程中使用DTO作为Get请求的参数需要在RPC Client类中，打上@SpringQueryMap注解**

   ```java
   /**
    * 单件成本变化记录弹窗
    */
   @GetMapping("listSkuCostLog")
   PageResponse<SkuCostLogListVO> listSkuCostLog(@SpringQueryMap SkuCostLogQuery query);
   ```

   * SpringQueryMap没有生效？

     * 配置请求端的Fegin，加强SpringQueryMap解析

       ```java
       @Configuration
       public class FeignConfiguration {
        @Bean
        public Feign.Builder feignBuilder() {
            return Feign.builder()
                    .queryMapEncoder(new BeanQueryMapEncoder())
                    .retryer(Retryer.NEVER_RETRY);
        }
       }
       ```

     * 使用Fegin加载该配置

       ```java
       @FeignClient(value = "test", configuration = FeignConfiguration.class)
       ```

     * 附带项目的FeignConfiguration

       ```java
       @Slf4j
       public class DionysusAdminClientConfiguration extends AbstractFeignClientConfiguration {
       
        @Value("${inner.gateway.dionysus-admin.signatureName}")
        private String sign;
       
        @Override
        public String getSignatureName() {
         return sign;
        }
       
        @Bean
        Logger.Level feignLogger() {
         return Logger.Level.FULL;
        }
       
        @Bean
        public ErrorDecoder errorDecoder() {
         return new RpcCommonConfig.ErrorDecoder();
        }
       
        @Bean
        public Feign.Builder feignBuilder() {
         return Feign.builder().queryMapEncoder(new BeanQueryMapEncoder());
        }
       }
       ```

10. Spring MVC 传递值的细节处理：Get请求，非基础类型的入参必须要打上@RequestParam 注解，这个注解作用是强制匹配入参

   ```java
   /**
   * skuId -> 单件成本
   */
   @GetMapping("cargoCostMap")
   @ResponseBody
   public Map<Long, BigDecimal> listCargoCostMapInTheSameWarehouse(@NotNull(message = "仓库必传") long warehouseId,
   @NotNull(message = "skuId必传") @RequestParam List<Long> skuIds) {
    return skuCostQueryApplicationService.listCargoCostMapInTheSameWarehouse(warehouseId, skuIds);
   }
   ```
