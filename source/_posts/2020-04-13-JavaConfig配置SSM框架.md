---
layout: post
title: JavaConfig 搭建 SSM 框架
categories: Web  
description: 通过采用纯JavaConfig 的方式来搭建Spring、Mybatis、springMVC框架的开发环境 
keywords: SSM, 框架 , JavaConfig 
---  
**摘要：** Spring在3.0版本之后支持了javaconfig的配置方式来代替原来的xml配置，javaconfig的配置简化了开发也为springboot的开发奠定了基础
<!-- more -->

> 开发环境：  
>
> ide：IDEA 2020.1  
> JDK：java8  
> Spring：5.2.4. RELEASE  
> Mybatis：3.5.3  
> maven：3.6.3  

# 1. Maven 项目构建 

* 通过 `idea` 构建基础的 `web` 项目这里不在赘述，不糊直接百度可以

* 添加需要的依赖：

``` xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring.version>5.2.4.RELEASE</spring.version>
    </properties>

    <dependencies>
        <!-- spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>aopalliance</groupId>
            <artifactId>aopalliance</artifactId>
            <version>1.0</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.2</version>
        </dependency>
        <!-- mybatis 和spring 整合包 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.1</version>
        </dependency>

        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!-- 连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.20</version>
        </dependency>
        <!-- 日志 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>org.logback-extensions</groupId>
            <artifactId>logback-ext-spring</artifactId>
            <version>0.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.25</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.9</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.9.8</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.9</version>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp.jstl</groupId>
            <artifactId>jstl-api</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.9</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.14.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

* 项目结构如下：

![img](https://raw.githubusercontent.com/AnonyStar/AnonyStar.github.io/master/images/post/ssm/ssm项目结构.png)

> 项目中 `web-inf/web.xml` 文件删除，不需要，对于 `web.xml` 文件，需要知道， `servlet3.0` 之后是支持注解的方式的，不需要xml配置，所以这里整个配置文件是不需要的。

> 如果用jsp动态页面要放在 webapp下，如果是静态页面可以直接放resource下，如果这样整个webapp目录也是没有任何作用了，可以删除，这里我们用jsp 所以保留了这个目录

# 2.spring上下文配置

在以前项目中我们会配置一个 `applicationContext.xml` 的文件，里面主要配置spring的上下文关系和bean实例这些都可以通过javaconfig来配置，我们在 `com.config` 包中创建 `SpringConfig` 类，主要实现如下：

``` JAVA
package com.config;

import org.springframework.context.annotation.*;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.stereotype.Controller;

/**
 * @author: anonystar
 * @time: 2020/3/29 19:38
 */
@Configuration
@ComponentScan(basePackages = "com.portal",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)
})
@Import({JdbcConfig.class})
public class SpringConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer(){
        return new PropertySourcesPlaceholderConfigurer();
    }

}

```

* `@Configuration` 注解是标记为配置类，用于初始化加载
* `@ComponentScan` 设置自动扫描的类 
* `@Import` 单独导入其他的配置文件，类似我们在xml中引入其他xml

# 3.mybatis与配置

我们将和数据库相关的全部配置到一个类中 `JdbcConfig` 

``` JAVA
package com.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.github.pagehelper.PageInterceptor;
import lombok.Data;
import org.apache.ibatis.plugin.Interceptor;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.mapper.MapperScannerConfigurer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
import java.util.Properties;

/**
 * 数据源配置  mybatis
 * @author: anonystar
 * @time: 2020/3/29 20:12
 */
@Configuration
@PropertySource("classpath:jdbc.properties")  //可以在外部文件中加载
@EnableTransactionManagement
@Data
@Component
public class JdbcConfig {
   // @Value("${jdbc.driver}")
    private String driver = "com.mysql.jdbc.Driver";
    //@Value("${jdbc.url}")
    private String url = "jdbc:mysql://localhost:3306/ocdb";
   // @Value("${jdbc.username}")
    private String username = "root";
    //@Value("${jdbc.password}")
    private String password = "root";

    @Bean(name = "dataSource")
    public DataSource dataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(url);
        druidDataSource.setDriverClassName(driver);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        return druidDataSource;
    }

    //spring-mybatis中替代myabtis的sqlSessionFactory的，
    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(){
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();

        //分页组件插入
        PageInterceptor pageInterceptor = new PageInterceptor();
        pageInterceptor.setProperties(new Properties());
        Interceptor[] interceptors = {pageInterceptor};
        bean.setPlugins(interceptors);
        bean.setDataSource(dataSource());
        return bean;
    }
    //事务管理配置
    @Bean
    public DataSourceTransactionManager dataSourceTransactionManager(){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource());
        return transactionManager;
    }
    //mapper映射配置
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer scannerConfigurer = new MapperScannerConfigurer();
        scannerConfigurer.setBasePackage("com.portal.dao");
        return scannerConfigurer;
    }

}

```

上面整个代码都是比较简单的，只要理解 `@Bean` 等价于原来xml中的 `<bean></bean>` 标签就可以了，其他和xml配置没多少区别

* 配置mybatis的扫描包时指定了包路径，那么我们的mapper映射文件也要和接口路径一致 名字一致 在resources目录下，不能放在Java目录下，maven不会编译在Java目录下的非java文件

# 4.springmvc配置

spring中提供了 `WebMvcConfigurationSupport` 类用于我们继承来实现springmvc的配置
springmvc我们需要配置的较多如下：
`WebMvcConfig.java` 

``` JAVA
package com.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.http.MediaType;
import org.springframework.http.converter.ByteArrayHttpMessageConverter;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.ResourceHttpMessageConverter;
import org.springframework.http.converter.StringHttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.http.converter.support.AllEncompassingFormHttpMessageConverter;
import org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter;
import org.springframework.http.converter.xml.SourceHttpMessageConverter;
import org.springframework.stereotype.Controller;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import java.nio.charset.Charset;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;
import java.util.TimeZone;

/**
 * @author: anonystar
 * @time: 2020/3/29 19:40
 */
@Configuration
@ComponentScan(basePackages = "com.portal",includeFilters ={
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)
} )
public class WebMvcConfig extends WebMvcConfigurationSupport {

    @Bean
    public ViewResolver viewResolver(){
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/pages/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    /**
     * 静态资源过滤
     * @param registry
     */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");

    }

    /**
     * 设置消息转换 请求头和响应的转换
     * @param converters
     */
    @Override
    protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Charset charset = Charset.forName("UTF-8");
        MappingJackson2HttpMessageConverter jacksonConverter =
                new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = jacksonConverter.getObjectMapper();
        objectMapper.setLocale(Locale.CANADA);
        objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8"));
        List<MediaType> mediaTypeList = new ArrayList<>();
        mediaTypeList.add(new MediaType("text", "plain", charset));
        mediaTypeList.add(new MediaType("application", "json", charset));
        jacksonConverter.setSupportedMediaTypes(mediaTypeList);

        StringHttpMessageConverter stringConverter = new StringHttpMessageConverter(charset);
        stringConverter.setWriteAcceptCharset(false);

        //保持以下顺序
        converters.add(jacksonConverter);//"application/json" "application/*+json"
        converters.add(new Jaxb2RootElementHttpMessageConverter());//"application/xml" "text/xml" "application/*+xml"
        converters.add(new SourceHttpMessageConverter<>());//"application/xml" "text/xml" "application/*+xml"
        converters.add(new AllEncompassingFormHttpMessageConverter());//"application/x-www-form-urlencoded" "multipart/form-data"
        converters.add(new ByteArrayHttpMessageConverter());//"application/octet-stream" "*/*"
        converters.add(stringConverter);//"text/plain" "*/*"
        converters.add(new ResourceHttpMessageConverter());//"*/*"
    }
}

```

* 这里需要注意静态资源的映射，我们将静态资源的更目录映射到 `/static/` 目录下
* 增加了springmvc对自定义对象的请求响应json方式
* 配置了视图解析器 jsp ，可以是其他的，可以参考springboot的设置

# 5. 替代web.xml配置 WebApplicationInitializer

我们在 `com.config` 目录下新建类 `WebApplicationInitializer` 用来实现 以前项目中 `web.xml` 的职责，首先我们明确 `web,xml` 主要的职责：

* servlet映射关系配置
* listener监听器配置
* filter 过滤器配置

在 `spring` 中我们知道 `DispatcherServlet` 是所有请求的入口，但是这个类并不是有带 `@WebServlet` 注解的, 所以是spring中提供了 `AbstractAnnotationConfigDispatcherServletInitializer` 这个类，我们只需要继承这个类，就可以实现 `web.xml` 的功能代码如下：

``` JAVA
package com.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * Servlet3.0以上
 * 代替web.xml 作为项目启动的 加载类
 * 1.可以加载过滤器
 * 2.监听器
 * 3.初始化servlet ->DispatcherServlet
 * 4.spring 容器配置文件加载
 *
 * @author: anonystar
 * @time: 2020/3/29 19:53
 */
public class WebApplicationInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * Specify {@code @Configuration} and/or {@code @Component} classes for the
     * {@linkplain #createRootApplicationContext() root application context}.
     *
     * @return the configuration for the root application context, or {@code null}
     * if creation and registration of a root context is not desired
     */
    /**
     * 指定spring容器配置文件
     *
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{SpringConfig.class};
    }

    /**
     * 指定springmvc的配置类
     * Specify {@code @Configuration} and/or {@code @Component} classes for the
     * {@linkplain #createServletApplicationContext() Servlet application context}.
     *
     * @return the configuration for the Servlet application context, or
     * {@code null} if all configuration is specified through root config classes.
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{WebMvcConfig.class};
    }

    /**
     * 请求拦截mapping
     * Specify the servlet mapping(s) for the {@code DispatcherServlet} &mdash;
     * for example {@code "/"}, {@code "/app"}, etc.
     *
     * @see #registerDispatcherServlet(ServletContext)
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}

```

> 到上面为止我们的基本环境已经搭建完成，接下来就是我们的开发

