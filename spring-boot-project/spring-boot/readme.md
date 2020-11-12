## 1、条件注解 `@Conditional` 派生

> SpringBoot自动配置是需要满足相应的条件才会自动配置,因此SpringBoot的自动配置大量应用了条件注解 `@ConditionalOnXXX`

- `@ConditionalOnBean` 容器存在指定 Bean

- `@ConditionalOnClass` 容器存在指定 Class

- `@ConditionalOnExpression` 基于 SpEL 表达式

- `@ConditionalOnJava` 基于 Jvm 版本

- `@ConditionalOnJndi` 在 JNDI 存在时查找指定位置

- `@ConditionalOnMissBean` 容器不存在指定 Bean

- `@ConditionalOnMissClass` 容器不存在指定 Class

- `@ConditionalOnNotWebApplication` 当前不是 web 项目

- `@ConditionalOnProperty` 属性值是否存在

- `@ConditionalOnResource` 类路径是否有指定资源

- `@ConditionalOnSingleCondidate` 指定 Bean 只存在一个，或虽有多个但指定首选

- `@ConditionalOnWebApplication` 当前是 web 项目

## 2、条件注解的实现

```java
// 1、实现spring 的 Condition 接口，并且重写 matches() 方法，
// 如果 @ConditionalOnLinux 的注解属性 environment 是 linux 就返回 true

public class LinuxCondition implements Condition {
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		// 获得注解 @ConditionalOnLinux 的所有属性
		List<AnnotationAttributes> allAnnotationAttributes = annotationAttributesFromMultiValueMap(
				metadata.getAllAnnotationAttributes(
						ConditionalOnLinux.class.getName()));
		for (AnnotationAttributes annotationAttributes : allAnnotationAttributes) {
			// 获得注解 @ConditionalOnLinux 的 environment 属性
			String environment = annotationAttributes.getString("environment");
			// 若 environment 等于 linux，则返回 true
			if ("linux".equals(environment)) {
				return true;
			}
		}
		return false;
	}
}
// 2、定制条件注解
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(LinuxCondition.class)
public @interface ConditionalOnLinux {
	// 标注是哪个环境
	String environment() default "";

}
// 3、使用注解
@Configuration
public class ConditionConfig {
    // 只有 `@ConditionalOnLinux` 的注解属性 `environment` 是 "linux" 时才会创建 bean
	@Bean
	@ConditionalOnLinux(environment = "linux")
	public Environment linuxEnvironment() {
		return new LinuxEnvironment();
	}
}
```

## 3、`@Condition` 接口

```java
@FunctionalInterface
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

- ConditionContext，顾名思义，主要是跟 Condition 的上下文有关，主要用来获取 Registry,BeanFactory,Environment,ResourceLoader 和 ClassLoader

- AnnotatedTypeMetadata，这个跟注解元数据有关，利用 AnnotatedTypeMetadata 可以拿到某个注解的一些元数据，而这些元数据就包含了某个注解里面的属性

## 4、`@SpringBootCondition` SpringBoot 条件注解的基类

- java/org/springframework/boot/autoconfigure/condition/SpringBootCondition.java 注解

---- `SpringBootCondition`
  |
  |- `FilteringSpringBootCondition`
  |  |
  |  |- `OnClassCondition`
  |  |
  |  |- `OnWebApplicationCondition`
  |   
  |- `OnJavaCondition`
  |
  |- `OnResourceCondition` 判断指定路径资源是否存在
  |
  |- `OnPropertyCondition`
  |
  |- `OnJndiCondition`
  |
  |- `OnExpressionCondition`
  |
  |- `OnBeanCondition`

