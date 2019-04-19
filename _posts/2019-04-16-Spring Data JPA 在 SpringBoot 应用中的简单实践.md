---
layout:     post
title:      Spring Data JPA 在 SpringBoot 应用中的简单实践
subtitle:   及资料汇总
date:       2019-04-16
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_62a.jpg
catalog: true
tags:
    - spring
---

## 1.引用maven依赖

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
```

## 2.添加配置

添加`@EnableJpaRepositories`注解，开启JPA功能

```java
@SpringBootApplication
@EnableJpaRepositories
public class SpringdataDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringdataDemoApplication.class, args);
    }
}
```

## 3.定义实体类

使用`@Entity`对实体类进行注解

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;


@Entity(name = "t_user")
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Data
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "`t_name`")
    private String name;

    /**
     * 不使用 @Column ，默认数据库字段名就是 age
     */
    private Integer age;

    /**
     * 枚举 
     */
    @Enumerated(EnumType.STRING)
    UserStatus status;

}
```

## 4.仓库类

#### 4.1 基本仓库 Repository

```java

import org.springframework.data.repository.Repository;

public interface UserRepository extends Repository<User, Long> {

}
```
这样就可以使用`UserRepository`就是持久化操作了，
但是若仅仅继承`Repository`接口，需要自己编写很多方法。


#### 4.2 CrudRepository

`CrudRepository`接口封装了常用的增删改查方法：
```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
	<S extends T> S save(S entity);
	<S extends T> Iterable<S> saveAll(Iterable<S> entities);

	Optional<T> findById(ID id);
	boolean existsById(ID id);
	Iterable<T> findAll();
	Iterable<T> findAllById(Iterable<ID> ids);

	long count();

	void deleteById(ID id);
	void delete(T entity);
	void deleteAll(Iterable<? extends T> entities);
	void deleteAll();
}

```

使用案例：
```java
import org.springframework.data.repository.CrudRepository;

public interface UserRepository extends CrudRepository<User, Long> {

}
```

#### 4.3 PagingAndSortingRepository

`PagingAndSortingRepository`提供了基本的分页方法，并继承了`CrudRepository`提供的增删改查方法

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	Iterable<T> findAll(Sort sort);

	Page<T> findAll(Pageable pageable);
}
```

使用案例：
```java
import org.springframework.data.repository.PagingAndSortingRepository;

public interface UserRepository extends PagingAndSortingRepository<User, Long> {

}
```


#### 4.4 JpaRepository

```java
/**
 * JPA specific extension of {@link org.springframework.data.repository.Repository}.
 *
 * @author Oliver Gierke
 * @author Christoph Strobl
 * @author Mark Paluch
 */
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.CrudRepository#findAll()
	 */
	List<T> findAll();

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.PagingAndSortingRepository#findAll(org.springframework.data.domain.Sort)
	 */
	List<T> findAll(Sort sort);

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.CrudRepository#findAll(java.lang.Iterable)
	 */
	List<T> findAllById(Iterable<ID> ids);

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.CrudRepository#save(java.lang.Iterable)
	 */
	<S extends T> List<S> saveAll(Iterable<S> entities);

	/**
	 * Flushes all pending changes to the database.
	 */
	void flush();

	/**
	 * Saves an entity and flushes changes instantly.
	 *
	 * @param entity
	 * @return the saved entity
	 */
	<S extends T> S saveAndFlush(S entity);

	/**
	 * Deletes the given entities in a batch which means it will create a single {@link Query}. Assume that we will clear
	 * the {@link javax.persistence.EntityManager} after the call.
	 *
	 * @param entities
	 */
	void deleteInBatch(Iterable<T> entities);

	/**
	 * Deletes all entities in a batch call.
	 */
	void deleteAllInBatch();

	/**
	 * Returns a reference to the entity with the given identifier. Depending on how the JPA persistence provider is
	 * implemented this is very likely to always return an instance and throw an
	 * {@link javax.persistence.EntityNotFoundException} on first access. Some of them will reject invalid identifiers
	 * immediately.
	 *
	 * @param id must not be {@literal null}.
	 * @return a reference to the entity with the given identifier.
	 * @see EntityManager#getReference(Class, Object) for details on when an exception is thrown.
	 */
	T getOne(ID id);

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.query.QueryByExampleExecutor#findAll(org.springframework.data.domain.Example)
	 */
	@Override
	<S extends T> List<S> findAll(Example<S> example);

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.query.QueryByExampleExecutor#findAll(org.springframework.data.domain.Example, org.springframework.data.domain.Sort)
	 */
	@Override
	<S extends T> List<S> findAll(Example<S> example, Sort sort);
}

```

```java
/**
 * Interface to allow execution of Query by Example {@link Example} instances.
 *
 * @param <T>
 * @author Mark Paluch
 * @author Christoph Strobl
 * @since 1.12
 */
public interface QueryByExampleExecutor<T> {

	/**
	 * Returns a single entity matching the given {@link Example} or {@literal null} if none was found.
	 *
	 * @param example must not be {@literal null}.
	 * @return a single entity matching the given {@link Example} or {@link Optional#empty()} if none was found.
	 * @throws org.springframework.dao.IncorrectResultSizeDataAccessException if the Example yields more than one result.
	 */
	<S extends T> Optional<S> findOne(Example<S> example);

	/**
	 * Returns all entities matching the given {@link Example}. In case no match could be found an empty {@link Iterable}
	 * is returned.
	 *
	 * @param example must not be {@literal null}.
	 * @return all entities matching the given {@link Example}.
	 */
	<S extends T> Iterable<S> findAll(Example<S> example);

	/**
	 * Returns all entities matching the given {@link Example} applying the given {@link Sort}. In case no match could be
	 * found an empty {@link Iterable} is returned.
	 *
	 * @param example must not be {@literal null}.
	 * @param sort the {@link Sort} specification to sort the results by, must not be {@literal null}.
	 * @return all entities matching the given {@link Example}.
	 * @since 1.10
	 */
	<S extends T> Iterable<S> findAll(Example<S> example, Sort sort);

	/**
	 * Returns a {@link Page} of entities matching the given {@link Example}. In case no match could be found, an empty
	 * {@link Page} is returned.
	 *
	 * @param example must not be {@literal null}.
	 * @param pageable can be {@literal null}.
	 * @return a {@link Page} of entities matching the given {@link Example}.
	 */
	<S extends T> Page<S> findAll(Example<S> example, Pageable pageable);

	/**
	 * Returns the number of instances matching the given {@link Example}.
	 *
	 * @param example the {@link Example} to count instances for. Must not be {@literal null}.
	 * @return the number of instances matching the {@link Example}.
	 */
	<S extends T> long count(Example<S> example);

	/**
	 * Checks whether the data store contains elements that match the given {@link Example}.
	 *
	 * @param example the {@link Example} to use for the existence check. Must not be {@literal null}.
	 * @return {@literal true} if the data store contains elements that match the given {@link Example}.
	 */
	<S extends T> boolean exists(Example<S> example);
}

```

使用案例：
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {

}
```



## 5.测试用例

```java

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringdataDemoApplicationTests {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void test1() {
        User user = User.builder()
                .name("gouzi")
                .age(25)
                .status(UserStatus.LEVEL1)
                .build();
        userRepository.save(user);
    }

    @Test
    public void test2() {
        //
        userRepository.findAllByName("Tom").forEach(System.out::println);
    }

}
```

## 6.自定义持久化语句

#### 6.1 方法名方式

```java
public interface UserRepository extends Repository<User, Long> {

  long countByLastname(String lastname);

  long deleteByLastname(String lastname);

  List<User> removeByLastname(String lastname);

  List<User> findByLastname(String lastname);

  User findByEmailAddress(String emailAddress);
  User findByEmailAddress(EmailAddress emailAddress);

  User getByEmailAddress(EmailAddress emailAddress);                    

  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAdress);          

  Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress);

  List<User> findByLastname(String lastname);

  User findByEmailAddress(String emailAddress);

  List<User> findByEmailAddressAndLastname(String emailAddress, String lastname);
}
```

拓展：
[https://docs.spring.io/spring-data/jpa/docs/2.1.6.RELEASE/reference/html/#jpa.query-methods.query-creation](https://docs.spring.io/spring-data/jpa/docs/2.1.6.RELEASE/reference/html/#jpa.query-methods.query-creation)#Table 3. Supported keywords inside method names

关键字 | 案例 | 含义
| ------ | ------ | ------ |
And | findByLastnameAndFirstname | … where x.lastname = ?1 and x.firstname = ?2
Or | findByLastnameOrFirstname | … where x.lastname = ?1 or x.firstname = ?2
Is,Equals | findByFirstname,findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1
Between | findByStartDateBetween | … where x.startDate between ?1 and ?2
LessThan | findByAgeLessThan | … where x.age < ?1
LessThanEqual | findByAgeLessThanEqual | … where x.age <= ?1
GreaterThan | findByAgeGreaterThan | … where x.age > ?1
GreaterThanEqual | findByAgeGreaterThanEqual | … where x.age >= ?1
After | findByStartDateAfter | … where x.startDate > ?1
Before | findByStartDateBefore | … where x.startDate < ?1
IsNull | findByAgeIsNull | … where x.age is null
IsNotNull,NotNull | findByAge(Is)NotNull | … where x.age not null
Like | findByFirstnameLike | … where x.firstname like ?1
NotLike | findByFirstnameNotLike | … where x.firstname not like ?1
StartingWith | findByFirstnameStartingWith | … where x.firstname like ?1 (parameter bound with appended %)
EndingWith | findByFirstnameEndingWith | … where x.firstname like ?1 (parameter bound with prepended %)
Containing | findByFirstnameContaining | … where x.firstname like ?1 (parameter bound wrapped in %)
OrderBy | findByAgeOrderByLastnameDesc | … where x.age = ?1 order by x.lastname desc
Not | findByLastnameNot | … where x.lastname <> ?1
In | findByAgeIn(Collection<Age> ages) | … where x.age in ?1
NotIn | findByAgeNotIn(Collection<Age> ages) | … where x.age not in ?1
True | findByActiveTrue() | … where x.active = true
False | findByActiveFalse() | … where x.active = false
IgnoreCase | findByFirstnameIgnoreCase | … where UPPER(x.firstame) = UPPER(?1)


#### 6.2 编写SQL

```java
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.emailAddress = ?1")
  User findByEmailAddress(String emailAddress);

  @Query("select u from User u where u.firstname like %?1")
  List<User> findByFirstnameEndsWith(String firstname);

  @Query(value = "SELECT * FROM USERS WHERE EMAIL_ADDRESS = ?1", nativeQuery = true)
  User findByEmailAddress(String emailAddress);

  @Query(value = "SELECT * FROM USERS WHERE LASTNAME = ?1",
    countQuery = "SELECT count(*) FROM USERS WHERE LASTNAME = ?1",
    nativeQuery = true)
  Page<User> findByLastname(String lastname, Pageable pageable);

  @Query("select u from User u where u.lastname like ?1%")
  List<User> findByAndSort(String lastname, Sort sort);

  @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
  List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);

  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);

  @Query("select u from #{#entityName} u where u.lastname = ?1")
  List<User> findByLastname(String lastname);


  @Modifying
  @Query("delete from User u where user.role.id = ?1")
  void deleteInBulkByRoleId(long roleId);

  @QueryHints(value = { @QueryHint(name = "name", value = "value")},
              forCounting = false)
  Page<User> findByLastname(String lastname, Pageable pageable);

  @Override
  @Transactional(timeout = 10)
  public List<User> findAll();

  @Modifying
  @Transactional
  @Query("delete from User u where u.active = false")
  void deleteInactiveUsers();

  // Plain query method
  @Lock(LockModeType.READ)
  List<User> findByLastname(String lastname);

  // Redeclaration of a CRUD method
  @Lock(LockModeType.READ);
  List<User> findAll();
```


## 附录

- [https://docs.spring.io/spring-data/jpa/docs/2.1.6.RELEASE/reference/html/](https://docs.spring.io/spring-data/jpa/docs/2.1.6.RELEASE/reference/html/)
- https://mp.weixin.qq.com/s/Uw3wkiqD16av3tVW-yiTDQ