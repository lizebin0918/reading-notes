# Spring Boot学习

## 原理
  * 约定优于配置的提现，并没有引入新的技术
  * AutoConfiguration 自动装配
  * Starter
  * Actuator
  * SpringBoot CLI

## SpringBootApplication
  * EnableAutoConfiguration
  	* AutoConfigurationPackage
  	* @Import({AutoConfigurationImportSelector.class})
  	* SPI扩展点
  	  * A项目依赖B项目，B项目新建文件：resources/META-INF/spring.factories打包，A项目就可以加载B项目的bean
  	  * ConditionOnXXXX:条件注解
  * ComponentScan
  * SpringBootConfiguration->Configuration
  	* 组件化实例，包含了多个实例