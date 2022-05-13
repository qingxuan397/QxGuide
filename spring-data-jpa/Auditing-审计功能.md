## 一、数据库审计

数据库审计是指当数据库有记录变更时，可以记录数据库的变更时间和变更人等，这样以后出问题回溯问责也比较方便。对于审计表记录的变更可以两种方式，一种是建立一张审计表专门用于记录，另一种是在数据库增加字段。本文所讨论的是第二种方案。

那如何在新增、修改、删除的时候同时增加记录呢？如果每张表都单独记录，代码就会显得很冗余。更好的方式应该是做切面或者事件监听，当数据有变更时统一进行记录。



## 二、Spring Data JPA审计

`Spring Data JPA`为我们提供了方便的`Audit`功能，通过四个注解来标记字段：

(1) @CreatedBy： 创建人，在这个实体被insert的时候，会设置值。

(2) @CreatedDate：创建时间，在这个实体被 insert的时候，会设置值

(3) @LastModifiedBy： 最后修改人，在这个实体被更新的时候，会设置值

(4) @LastModifiedDate： 最后修改时间，在这个实体被更新的时候，会设置值



## 三、创建审计类

```java
@Data
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class Auditable<U> {
    @CreatedBy
    @Column(name = "created_by")
    private U createdBy;
 
    @CreatedDate
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_date", updatable = false, nullable = true)
    private Date createdDate;
 
    @LastModifiedBy
    @Column(name = "last_modified_by")
    private U lastModifiedBy;
 
    @LastModifiedDate
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "last_modified_date", nullable = true)
    private Date lastModifiedDate;
}
```



数据库表字段设计：

```sql
  `created_date` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `last_modified_date` datetime(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间',
  `last_modified_by` varchar(32) DEFAULT NULL COMMENT '更新人',
```



`@MappedSuperclass`可以让其它子实体类继承相关的字段和属性；

`@EntityListeners`设置监听类，会对`新增`和`修改`进行回调处理。



## 四、继承审计类

```java
@Data
@Entity
@Table(name = "t_user")
public class User extends Auditable<String> {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long userId;
    private String name;
    private String email;
    private String country;
    private String website;
}
```



## 五、如何填充审计信息

有以下两种写法， 如果在ApplicationContext中注册了多个实现，则可以通过显式设置@EnableJpaAuditing的auditorAwareRef属性来选择要使用的实现。

### 方式一（推荐）：

```java
@Configuration
@EnableJpaAuditing(auditorAwareRef = "auditorProvider")
public class JpaConfig {
 
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            // 从上下文获取登录用户信息
            String username = "system";
            SecurityContext context = SecurityContextHolder.getContext();
            if (context != null) {
                Authentication authentication = context.getAuthentication();
                if (authentication != null) {
                    username = authentication.getName();
                }
            }
 
            return Optional.ofNullable(username);
        };
    }
}
```



### 方式二：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;

@Configuration
@EnableJpaAuditing
public class JpaConfig {
 
}
```



实现AuditorAware接口

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.data.domain.AuditorAware;

@Configuration
public class MacTokenAuditorAware implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
         // 从上下文获取登录用户信息
        String username = "system";
        SecurityContext context = SecurityContextHolder.getContext();
        if (context != null) {
            Authentication authentication = context.getAuthentication();
            if (authentication != null) {
                username = authentication.getName();
            }
        }

        return Optional.ofNullable(username);
    }
    
//    @Override
//    public Optional<String> getCurrentAuditor() {
//        //从token 中获取 用户名
//        ServletRequestAttributes servletRequestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
//           if (servletRequestAttributes != null) {
//            HttpServletRequest request = servletRequestAttributes.getRequest();
//            String token = request.getHeader("token");
//            //1.从token 中解析
//            //2.从网关下发得到
//            String loginUserInfo = request.getHeader("loginUserInfo");
//            //模拟获取到 操作人id 返回
//            return Optional.of(-2L);
//        }
//           return Optional.empty();
//    }
}
```



这里配置的是通过`Spring Security`的`Context`来获取登陆用户的名字，当然可以有其它方案，如获取请求头的某个字段等。

注意注解`@EnableJpaAuditing`开启了审计功能。



## 附录

### Auditing 原理解析

**源码的关系图**

![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/8cfa5800-4186-11e8-94d7-4b37be7eacf0.jpg)



### @MappedSuperclass

#### 分析

(1) 该注解标注在类上，用来标识父类；

(2) 该注解标识的类不能映射到数据库，被标识的类的属性必须通过子类来映射；

(3) 该注解标识了类之后，不能再有@Entity和@Table注解标识该类

(4) 标识有该注解的类，通常属性上用以下注解@Id @GeneratedVale(strategy=GenerationType.IDENTITY) 、

@CreateDate 、 @CreateBy、@LastModifiedBy、@LastModifiedDate等用在父类上，子类继承该父类即可，并在子类标注@Table和@Entity



### @EntityListeners

#### 源码

```java
/*
 * Copyright (c) 2008, 2019 Oracle and/or its affiliates. All rights reserved.
 *
 * This program and the accompanying materials are made available under the
 * terms of the Eclipse Public License v. 2.0 which is available at
 * http://www.eclipse.org/legal/epl-2.0,
 * or the Eclipse Distribution License v. 1.0 which is available at
 * http://www.eclipse.org/org/documents/edl-v10.php.
 *
 * SPDX-License-Identifier: EPL-2.0 OR BSD-3-Clause
 */

// Contributors:
//     Linda DeMichiel - 2.1
//     Linda DeMichiel - 2.0


package javax.persistence;

import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * Specifies the callback listener classes to be used for an 
 * entity or mapped superclass. This annotation may be applied 
 * to an entity class or mapped superclass.
 *
 * @since 1.0
 */
@Target({TYPE}) 
@Retention(RUNTIME)
public @interface EntityListeners {

    /** The callback listener classes */
    Class[] value();
}

```

#### 分析

从源码的注释中可以很清楚的了解到该注解的作用，简单翻译如下：该注解用于指定Entity或者superclass上的回调监听类。该注解可以用于Entity或者superclass上。



### @EnableJpaAuditing

#### 源码

```java
/*
 * Copyright 2013-2021 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.springframework.data.jpa.repository.config;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.context.annotation.Import;
import org.springframework.data.auditing.DateTimeProvider;
import org.springframework.data.domain.AuditorAware;

/**
 * Annotation to enable auditing in JPA via annotation configuration.
 *
 * @author Thomas Darimont
 * @author Oliver Gierke
 */
@Inherited
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(JpaAuditingRegistrar.class)
public @interface EnableJpaAuditing {

	/**
	 * Configures the {@link AuditorAware} bean to be used to lookup the current principal.
	 *
	 * @return
	 */
	String auditorAwareRef() default "";

	/**
	 * Configures whether the creation and modification dates are set. Defaults to {@literal true}.
	 *
	 * @return
	 */
	boolean setDates() default true;

	/**
	 * Configures whether the entity shall be marked as modified on creation. Defaults to {@literal true}.
	 *
	 * @return
	 */
	boolean modifyOnCreate() default true;

	/**
	 * Configures a {@link DateTimeProvider} bean name that allows customizing the {@link java.time.temporal.TemporalAccessor} to be
	 * used for setting creation and modification dates.
	 *
	 * @return
	 */
	String dateTimeProviderRef() default "";
}

```

#### 分析

原文:

If you expose a bean of type `AuditorAware` to the `ApplicationContext`, the auditing infrastructure automatically picks it up and uses it to determine the current user to be set on domain types. If you have multiple implementations registered in the `ApplicationContext`, you can select the one to be used by explicitly setting the `auditorAwareRef` attribute of `@EnableJpaAuditing`.

翻译:

如果将AuditorAware类型的bean公开给ApplicationContext，则审计基础结构会自动选择它并使用它来确定要在域类型上设置的当前用户。 如果在ApplicationContext中注册了多个实现，则可以通过显式设置@EnableJpaAuditing的auditorAwareRef属性来选择要使用的实现。



### AuditingEntityListener.class

#### 源码

```java
/*
 * Copyright 2008-2021 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package org.springframework.data.jpa.domain.support;

import javax.persistence.PrePersist;
import javax.persistence.PreUpdate;

import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.data.auditing.AuditingHandler;
import org.springframework.data.domain.Auditable;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;

/**
 * JPA entity listener to capture auditing information on persiting and updating entities. To get this one flying be
 * sure you configure it as entity listener in your {@code orm.xml} as follows:
 *
 * <pre>
 * &lt;persistence-unit-metadata&gt;
 *     &lt;persistence-unit-defaults&gt;
 *         &lt;entity-listeners&gt;
 *             &lt;entity-listener class="org.springframework.data.jpa.domain.support.AuditingEntityListener" /&gt;
 *         &lt;/entity-listeners&gt;
 *     &lt;/persistence-unit-defaults&gt;
 * &lt;/persistence-unit-metadata&gt;
 * </pre>
 *
 * After that it's just a matter of activating auditing in your Spring config:
 *
 * <pre>
 * &#064;Configuration
 * &#064;EnableJpaAuditing
 * class ApplicationConfig {
 *
 * }
 * </pre>
 *
 * <pre>
 * &lt;jpa:auditing auditor-aware-ref="yourAuditorAwarebean" /&gt;
 * </pre>
 *
 * @author Oliver Gierke
 * @author Thomas Darimont
 * @author Christoph Strobl
 * @author Mark Paluch
 */
@Configurable
public class AuditingEntityListener {

	private @Nullable ObjectFactory<AuditingHandler> handler;

	/**
	 * Configures the {@link AuditingHandler} to be used to set the current auditor on the domain types touched.
	 *
	 * @param auditingHandler must not be {@literal null}.
	 */
	public void setAuditingHandler(ObjectFactory<AuditingHandler> auditingHandler) {

		Assert.notNull(auditingHandler, "AuditingHandler must not be null!");
		this.handler = auditingHandler;
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * persist events.
	 *
	 * @param target
	 */
	@PrePersist
	public void touchForCreate(Object target) {

		Assert.notNull(target, "Entity must not be null!");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markCreated(target);
			}
		}
	}

	/**
	 * Sets modification and creation date and auditor on the target object in case it implements {@link Auditable} on
	 * update events.
	 *
	 * @param target
	 */
	@PreUpdate
	public void touchForUpdate(Object target) {

		Assert.notNull(target, "Entity must not be null!");

		if (handler != null) {

			AuditingHandler object = handler.getObject();
			if (object != null) {
				object.markModified(target);
			}
		}
	}
}
```

#### 分析

同样的从该类的注释也可以了解到该类的作用：这是一个JPA Entity Listener，用于捕获监听信息，当Entity发生持久化和更新操作时。