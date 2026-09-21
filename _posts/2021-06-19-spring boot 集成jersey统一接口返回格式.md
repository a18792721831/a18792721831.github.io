---
layout: post
title: "spring boot 集成jersey统一接口返回格式"
date: 2021-06-19 16:17:00 +0800
categories: [jersey统一返回, 统一接口返回, jersey统一异常处理, jersey实现过滤器, 规范jersey的接口返回]
description: "spring boot 集成jersey统一接口返回格式1. 统一接口返回格式的需要2. 后台如何开发2.1 接口定义统一对象2.2 过滤器统一封装3. jersey中统一异常处理4. 在jersey中注册1. 统一接口返回格式的需要对于接口开发来说，接口的返回值，明确的告诉了调用者，调用接口后，将会返回的数据是什么。所以，接口定义中，接口的返回值总是与接口的返回值一一对应。但是如果是前端人员调用的话，就不太又好了。前端希望有一个统一格式的返回，这样可以根据返回的数据做不同的逻辑。比如返回：{_jersey接口"
keywords: jersey统一返回, 统一接口返回, jersey统一异常处理, jersey实现过滤器, 规范jersey的接口返回
---

> **文章信息**
> - 原文链接：https://jiayq.blog.csdn.net/article/details/118055471
> - 发布时间：2021-06-19 16:17:00
> - 阅读量：722
> - 分类：微服务同时被 3 个专栏收录, 订阅专栏, spring boot, spring
> - 标签：#jersey统一返回, #统一接口返回, #jersey统一异常处理, #jersey实现过滤器, #规范jersey的接口返回

## 摘要

文章浏览阅读722次。spring boot 集成jersey统一接口返回格式1. 统一接口返回格式的需要2. 后台如何开发2.1 接口定义统一对象2.2 过滤器统一封装3. jersey中统一异常处理4. 在jersey中注册1. 统一接口返回格式的需要对于接口开发来说，接口的返回值，明确的告诉了调用者，调用接口后，将会返回的数据是什么。所以，接口定义中，接口的返回值总是与接口的返回值一一对应。但是如果是前端人员调用的话，就不太又好了。前端希望有一个统一格式的返回，这样可以根据返回的数据做不同的逻辑。比如返回：{_jersey接口

---

#### spring boot 集成jersey统一接口返回格式

  * 1\. 统一接口返回格式的需要
  * 2\. 后台如何开发
  *     * 2.1 接口定义统一对象
    * 2.2 过滤器统一封装
  * 3\. jersey中统一异常处理
  * 4\. 返回数据统一json化
  * 5\. 在jersey中注册

## 1\. 统一接口返回格式的需要

对于接口开发来说，接口的返回值，明确的告诉了调用者，调用接口后，将会返回的数据是什么。

所以，接口定义中，接口的返回值总是与接口的返回值一一对应。

但是如果是前端人员调用的话，就不太又好了。

前端希望有一个统一格式的返回，这样可以根据返回的数据做不同的逻辑。

比如返回：
    
    
    {
        "code":200,
        "message":"success",
        "data": {
            "name":"xiaomei",
            "age":18
        }
    }
    

这样前端调用人员可以根据code或者message来决定是否从data中取值。

如果code不是200，那么就不会从data中取数据。

也有可能是：
    
    
    {
        "code":404,
        "message":"path not found",
        "data": null
    }
    

这是一个异常的返回。

前端人员发现code不是200，就不会从data中获取数据，就不会发生空指针问题了。

## 2\. 后台如何开发

对于前端人员的这种需求，后台开发如何满足呢？

### 2.1 接口定义统一对象

我们可以在接口定义的时候，定义一个统一的对象：
    
    
    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public class Result {
    
        private String code;
    
        private String message;
    
        private Object data;
    
    }
    

然后定义接口的时候，全部的接口返回都是Result。

这样就能满足这个要求了。

嗯。。。。

仔细想想这样好吗？

这样是能满足前端人员的需要，但是，后台人员开发起来就是别样的别扭。

所有的接口都是返回Result，这什么玩意。

接口的返回值是需要体现一部分接口的逻辑的。

比如接口返回对象，接口返回单个属性等等。

### 2.2 过滤器统一封装

接口的返回值，就应该是体现接口逻辑的返回值，而不是统一对象。

在jersey中，我们写一个过滤器，用于返回值的封装。
    
    
    @Slf4j
    @Provider
    @Consumes({MediaType.APPLICATION_JSON, MediaType.WILDCARD})
    @Produces({MediaType.APPLICATION_JSON, MediaType.WILDCARD})
    @ConditionalOnProperty(value = "spring.jersey.application-path")
    @ConstrainedTo(SERVER)
    @Component
    public class ResultJsonFilter implements ContainerResponseFilter {
    
        @Context
        private HttpServletRequest servletRequest;
    
        @Override
        public void filter(ContainerRequestContext containerRequestContext, ContainerResponseContext containerResponseContext) throws IOException {
            if (Optional.ofNullable(containerResponseContext.getEntity()).isPresent() && (
                    containerResponseContext.getEntityClass().isAssignableFrom(ExceptionEntity.class) ||
                            containerResponseContext.getEntityClass().isAssignableFrom(Exception.class) ||
                            containerResponseContext.getEntityClass().isAssignableFrom(Error.class))) {
                log.error(servletRequest.getRemoteAddr() + ":" + servletRequest.getRemotePort() + " => " + servletRequest.getRequestURI() + " - " + servletRequest.getHeader("sessionId"));
                return;
            } else if (Optional.ofNullable(containerResponseContext.getEntity()).isEmpty() ||
                    containerResponseContext.getEntityClass().isAssignableFrom(String.class)) {
                log.info(servletRequest.getRemoteAddr() + ":" + servletRequest.getRemotePort() + " => " + servletRequest.getRequestURI() + " - " + servletRequest.getHeader("sessionId"));
                containerResponseContext.setEntity(Result.builder()
                        .code("200")
                        .message("ok")
                        .data(new ObjectMapper().writeValueAsString(containerResponseContext.getEntity()))
                        .build());
            } else {
                containerResponseContext.setEntity(Result.builder()
                        .code("200")
                        .message("ok")
                        .data(containerResponseContext.getEntity())
                        .build());
            }
        }
    }
    

如果返回的是异常，而且不是业务异常，不是我们自己定义的已知异常，那么对返回值不做任何封装。

如果返回的是字符串，需要单独做一次判断，因为不管是什么返回，最终都是依托字符串返回的(只是针对返回是json)，所以需要单独处理。

如果返回的是我们的类，那么，使用统一对象封装。

或许你有疑问，我们的业务异常，已知异常，怎么没有封装？

异常走的是异常的处理，这里是正常的处理。

## 3\. jersey中统一异常处理

在jersey中，因为做了统一格式返回，所以，对于已知的异常和业务异常，我们也需要处理。
    
    
    @Component
    @Slf4j
    public class QuanBackExceptionMapper implements ExceptionMapper<Exception>, ApplicationContextAware {
    
        private ApplicationContext applicationContext;
    
        @Autowired
        private MessageSource messageSource;
    
        @Override
        public Response toResponse(Exception exception) {
            log.trace(exception.getMessage(), exception);
            // 这里是业务异常。业务异常是自己定义的根异常，根异常继承RuntimeExcecption
            if (exception instanceof QuanBackRuntimeException) {
                QuanBackRuntimeException msrRuntimeException = (QuanBackRuntimeException) exception;
                if (msrRuntimeException.isWriteLog() && !msrRuntimeException.isShowStackTrace()) {
                    log.info(msrRuntimeException.getMessage());
                } else if (msrRuntimeException.isWriteLog() && msrRuntimeException.isShowStackTrace()) {
                    log.error(msrRuntimeException.getMessage(), msrRuntimeException);
                }
                // 对于异常码，需要做国际化替换
                String i18nMessage = StringUtils.hasText(msrRuntimeException.getI18nMessage()) ?
                        msrRuntimeException.getI18nMessage() : getI18n(msrRuntimeException.getMessage(),
                        CollectionUtils.isEmpty(msrRuntimeException.getArguments()) ? null :
                                msrRuntimeException.getArguments().toArray());
                // 如果是业务异常，那么code就不是200了，而是异常码。
                // message也是异常码的国际化信息
                return Response.status(Response.Status.OK)
                        .entity(ExceptionEntity.builder().
                                code(msrRuntimeException.getMessage())
                                .message(i18nMessage)
                                .data(msrRuntimeException.getArguments())
                                .build())
                        .type(MediaType.APPLICATION_JSON)
                        .build();
            }
            // login exception
            // 如果是验证异常(用户没有登录，不能访问，验证时通过异常，快速失败)
            if (exception instanceof ExecutionException ||
                    exception instanceof CacheLoader.InvalidCacheLoadException) {
                String i18nMessage = getI18n("please.login", null);
                return Response.status(HttpStatus.NON_AUTHORITATIVE_INFORMATION.value())
                        .entity(ExceptionEntity.builder().
                                code("please.login")
                                .message(i18nMessage)
                                .data(null)
                                .build())
                        .type(MediaType.APPLICATION_JSON)
                        .status(HttpStatus.NON_AUTHORITATIVE_INFORMATION.value())
                        .build();
            }
            if (exception instanceof WebApplicationException) {
                log.error(exception.getMessage(), exception);
                WebApplicationException webApplicationException = (WebApplicationException) exception;
                return Response.status(webApplicationException.getResponse().getStatus())
                        .entity(ExceptionEntity.builder().
                                code(webApplicationException.getMessage())
                                .message(webApplicationException.getMessage())
                                .build())
                        .type(MediaType.APPLICATION_JSON).build();
            }
            log.error(exception.getMessage(), exception);
            return Response.status(ExceptionHttpStatus.INTERNAL_SERVER_ERROR.value()).entity(
                    ExceptionEntity.builder().
                            code(ExceptionHttpStatus.INTERNAL_SERVER_ERROR.code())
                            .message(exception.getClass().toString())
                            .build()).type(MediaType.APPLICATION_JSON).build();
        }
    
    
        private String getI18n(String code, Object[] args) {
            return applicationContext.getMessage(code, args, "", Locale.CHINA);
        }
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.applicationContext = applicationContext;
        }
    }
    

业务根异常：业务根异常做了一些工具方法。
    
    
    public class QuanBackRuntimeException extends RuntimeException implements IThowableMessage, ArgumentsAware {
        public static final int ERROR_MESSAGE = 0;
        public static final int INFORMATION_MESSAGE = 1;
        public static final int WARNING_MESSAGE = 2;
    
        private static final long serialVersionUID = 3978421433929970999L;
    
        private String[] codes;
        private List<String> arguments;
        private String defaultMessage = "Error.";
        private int messageLevel = ERROR_MESSAGE;
        private boolean showStackTrace = true;
        private boolean writeLog = true;
    
        private String i18nMessage;
    
        private Locale locale;
    
        public QuanBackRuntimeException() {
            this((String) null);
        }
    
        public QuanBackRuntimeException(Throwable ex) {
            this(null, ERROR_MESSAGE, false, true, ex);
        }
    
        public QuanBackRuntimeException(String code, String... arguments) {
            this(code, ERROR_MESSAGE, false, true, null, arguments);
        }
    
        public QuanBackRuntimeException(String code, int messageLevel, boolean showStackTrace, boolean writeLog) {
            this(code, messageLevel, showStackTrace, writeLog, null);
        }
    
        public QuanBackRuntimeException(String code, Throwable ex) {
            this(code, ERROR_MESSAGE, false, true, ex);
        }
    
        public QuanBackRuntimeException(String code, boolean showStackTrace, boolean writeLog, Throwable ex, String...
                arguments) {
            this(code, ERROR_MESSAGE, showStackTrace, writeLog, ex, arguments);
        }
    
    
        public QuanBackRuntimeException(String code, int messageLevel, boolean showStackTrace, boolean writeLog, Throwable ex,
                                        String... arguments) {
            super(code, ex);
            this.defaultMessage = code;
            this.showStackTrace = showStackTrace;
            this.messageLevel = messageLevel;
            this.writeLog = writeLog;
            this.arguments = arguments == null ? new ArrayList() : new ArrayList(Arrays.asList(arguments));
            this.locale = Locale.CHINA;
        }
    
        public QuanBackRuntimeException(String code, int messageLevel, boolean showStackTrace,
                                        boolean writeLog, Throwable ex) {
            super(code, ex);
            if (ex instanceof MessageSourceResolvable) {
                MessageSourceResolvable re = (MessageSourceResolvable) ex;
                this.codes = re.getCodes();
            }
            pushCode(code);
            this.defaultMessage = code;
            this.showStackTrace = showStackTrace;
            this.messageLevel = messageLevel;
            this.writeLog = writeLog;
            this.arguments = arguments == null ? new ArrayList() : new ArrayList(Arrays.asList(arguments));
            this.locale = Locale.CHINA;
        }
    
        public void pushCode(String code) {
            if (code == null) {
                return;
            }
            String[] newCodes;
            if (this.codes != null) {
                newCodes = new String[this.codes.length + 1];
                System.arraycopy(this.codes, 0, newCodes, 1, this.codes.length);
            } else {
                newCodes = new String[1];
            }
            newCodes[0] = code;
            this.codes = newCodes;
        }
    
        public static QuanBackRuntimeException newInfo(String code, String... arguments) {
            return new QuanBackRuntimeException(code, INFORMATION_MESSAGE, false, true, null, arguments);
        }
    
        public static QuanBackRuntimeException newWarning(String code, String... arguments) {
            return new QuanBackRuntimeException(code, WARNING_MESSAGE, false, true, null, arguments);
        }
    
        public static QuanBackRuntimeException newError(String code, String... arguments) {
            return new QuanBackRuntimeException(code, ERROR_MESSAGE, false, true, null, arguments);
        }
    
        public String[] getCodes() {
            return codes;
        }
    
        public List<String> getArguments() {
            return arguments;
        }
    
        public String getArgumentListString() {
            return arguments != null ? arguments.stream().reduce((s1, s2) -> s1 + ";" + s2).orElse("") : "";
        }
    
    
        public void setArguments(String[] arguments) {
            this.arguments.addAll(Arrays.asList(arguments));
        }
    
        @Override
        public void setArguments(Object[] arguments) {
            this.arguments.addAll(Arrays.asList((String[]) arguments));
        }
    
        public String getDefaultMessage() {
            return defaultMessage;
        }
    
        @Override
        public int getMessageLevel() {
            return messageLevel;
        }
    
        @Override
        public boolean isShowStackTrace() {
            return showStackTrace;
        }
    
        public void setShowStackTrace(boolean showStackTrace) {
            this.showStackTrace = showStackTrace;
            if (!showStackTrace) {
                this.writeLog = false;
            }
        }
    
        @Override
        public boolean isWriteLog() {
            return writeLog;
        }
    
        @Override
        public StackTraceElement[] getStackTrace() {
            return super.getStackTrace();
        }
    
        @Override
        public String toString() {
            return super.toString() + " , args = " + this.arguments;
        }
    
        public String getI18nMessage() {
            return i18nMessage;
        }
    
        public void setI18nMessage(String i18nMessage) {
            this.i18nMessage = i18nMessage;
        }
    
        @Override
        public String[] getMsrArguments() {
            return (String[]) arguments.toArray();
        }
    }
    

两个用于业务的接口，仅供参考：
    
    
    public interface IThowableMessage extends ArgumentsAware {
    
        /**
         * getMessageLevel.
         *
         * @return int
         */
        int getMessageLevel();
    
        /**
         * isShowStackTrace.
         *
         * @return boolean
         */
        boolean isShowStackTrace();
    
        /**
         * isWriteLog.
         *
         * @return boolean
         */
        boolean isWriteLog();
    
        /**
         * 国际化参数中 需要国际化的参数。
         *
         * @return 需要国际化的参数。
         */
        String[] getMsrArguments();
    
    }
    
    
    
    public interface ArgumentsAware {
        /**
         * setArguments.
         *
         * @param arguments Object[]
         */
    
        void setArguments(Object[] arguments);
    
    }
    

## 4\. 返回数据统一json化

因为json的简单直观性，所以一般接口的返回值都是json。

json字符串返回，就需要我们将返回值都手动转为json字符串，这样对开发来说，非常的不友好。

不仅仅是返回值以json的格式返回，请求，我们也希望用json请求。

但是在接口实现中手动将json串转为对象，这也太麻烦了。

幸好jersey支持配置数据的序列化类。
    
    
    @Provider
    @Consumes({MediaType.APPLICATION_JSON, MediaType.WILDCARD})
    @Produces({MediaType.APPLICATION_JSON, MediaType.WILDCARD})
    @ConditionalOnProperty(value = "spring.jersey.application-path")
    @ConstrainedTo(SERVER)
    @Component
    public class JerseyJacksonJaxbJsonProvider extends JacksonJaxbJsonProvider {
    
        public JerseyJacksonJaxbJsonProvider() {
            super();
        }
    
        @Override
        public boolean isWriteable(Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType) {
            // 支持单个参数以字符串直接传输
            // 请求
            if (MediaType.APPLICATION_JSON_TYPE.equals(mediaType) ||
                    MediaType.TEXT_PLAIN_TYPE.equals(mediaType)) {
                return true;
            }
            return super.isWriteable(type, genericType, annotations, mediaType);
        }
    
        @Override
        public boolean isReadable(Class<?> type, Type genericType, Annotation[] annotations, MediaType mediaType) {
            // 返回，因为是统一格式，所以都是json
            // 即使是单个属性，最终也会封装为统一对象
            if (MediaType.APPLICATION_JSON_TYPE.equals(mediaType)) {
                return true;
            }
            return super.isReadable(type, genericType, annotations, mediaType);
        }
    }
    

## 5\. 在jersey中注册

我们定义了一个过滤器和一个异常处理器，但是目前这些还不会生效。

因为过滤器和异常处理器还未注册到jersey中。
    
    
    @Configuration
    // 项目访问的统一前缀
    @ApplicationPath("/quanback")
    public class JerseyConfig extends ResourceConfig implements ApplicationContextAware {
    
        private ApplicationContext applicationContext;
    
        @PostConstruct
        public void init() {
            // 注册异常处理器
            register(QuanBackExceptionMapper.class);
            // 注册过滤器
            register(ResultJsonFilter.class);
            // 注册json转化器
            register(JerseyJacksonJaxbJsonProvider.class);
            property("jersey.config.workers.legacyOrdering", true);
            registerClasses(JerseyServiceAutoScanner.getPublishJerseyServiceClasses(applicationContext, "com.quanback"));
            setApplicationName("test");
        }
    
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.applicationContext = applicationContext;
        }
    }
