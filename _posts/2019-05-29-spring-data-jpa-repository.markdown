---
layout: post
title: "spring-data-jpa-repository explained"
date: "2019-05-30 23:22:25"
author: brunojensen
header-img: img/post-bg-02.jpg
---

On a recent experience working on a self-contained spring-boot based application, I faced some “problems”. What happened was that even with so many different ways to write a repository with spring-data-jpa, in some cases it was not enough and as a quick solution the development team introduced a concrete implementation of repositories with EntityManager injection. Does it work? Yeah, of course, but is there any other way to do it?

### Spring Data JPA

First, let’s explore more about how to use spring-data-jpa, I'll not going into details because you probably already know. If not, check [Spring Data reference](https://docs.spring.io/spring-data/jpa/docs/2.1.8.RELEASE/reference/html) There are three main solutions to write a query on a spring repository: Query methods, @Query annotation, and JpaSpecificationExecutor. Well, it’s looking good, why would I need more?
Query methods is very cool but can look ugly if you need to filter and sort by various fields; Query annotation is nice but when your queries starts to be more complex it doesn’t look so good, especially to maintain in the same repository and if you have the same query with different parameters, usually you need to write it twice or concatenate strings; Criteria Builder solve all the previous problems but how does likes it? And, just like JPQL, there are a few limitations on the query syntax and you might need a native query instead of.

### Spring data extension

Okay, what're the chances of this happen in a project? Well, it shouldn’t happen, but sometimes it does. And everybody knows about the JPA limitations regarding querying in case you need performance, sometimes there’s no option besides writing a native query.

And based on those problems I decided to write a spring-data-jpa extension which is based on the specification pattern similar to JpaSpecificationExecutor but allowing the developer to write Specifications classes with more flexibility on the filtering and with better maintainability of the query, whether is JPQL or native.

Am I using this extension in any project? Not yet, but it's almost in a good shape to try it out on production.

So, let’s contribute. I have some issues and improvements waiting for you on github :)

[https://github.com/brunojensen/spring-data-jpa-repository](https://github.com/brunojensen/spring-data-jpa-repository)

<h3>Project details: </h3>

[![Build Status](https://travis-ci.org/brunojensen/spring-data-jpa-repository.svg?branch=master)](https://travis-ci.org/brunojensen/spring-data-jpa-repository)

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=spring.data.repository%3Aspring-data-jpa-repository&metric=alert_status)](https://sonarcloud.io/dashboard?id=spring.data.repository%3Aspring-data-jpa-repository)

Demo:
[https://github.com/brunojensen/spring-data-repository-demo](https://github.com/brunojensen/spring-data-repository-demo)

Setup:

```java

    @EnableJpaRepositories(
        value = "jpa.model.package",
        repositoryBaseClass = RepositorySpecificationExecutorImpl.class)
    public class ... {

    }

```

Usage:

```java

    public interface PersonRepository extends RepositorySpecificationExecutor<Person, String> {
     // you still can use spring-data common Repository implementations
     // this framework is only giving you a boost
    }

```

```java
...

    public Page<Person> searchBy(final Person person, final Pageable pageable) {
        return repository.findAll(new ByPersonUsingTypedQuerySpecification(person), pageable);
    }

    public List<Person> searchBy(final Person person) {
        return repository.findAll(new ByPersonUsingTypedQuerySpecification(person));
    }

    /**
     * Uses {@link org.springframework.data.jpa.repository.support.SimpleJpaRepository}
     * implementation
     */
    public Person findById(String id) {
        return repository.findById(id).orElse(null);
    }

    /**
     * Uses {@link org.extension.spring.data.repository.specification.TypedNativeQuerySpecification}
     * combined with {@link org.extension.spring.data.repository.annotations.TypedAsSqlResultSetMapping}
     * to use SqlResultMapping to serialize the query result into the pre-defined object.
     */
    public List<PersonContactResultMapping> findAll2() {
        return repository.findAll(new ByPersonContactUsingTypedNativeSpecification(), PersonContactResultMapping.class);
    }

    /**
     * Still capable to quering using annotation {@link org.springframework.data.jpa.repository.Query} and
     * interface projection
     */
    public Page<PersonRecord> findAll(final Pageable pageable) {
        return repository.findPersonRecord(pageable);
    }

    /**
     * Uses {@link org.extension.spring.data.repository.specification.TypedNativeQuerySpecification}
     *      * for count
     */
    public List<Person> findAll() {
        return repository.findAll((TypedQuerySpecification<Person>) () -> "SELECT * FROM Person");
    }

    /**
     * Uses {@link org.extension.spring.data.repository.specification.TypedNativeQuerySpecification} with
     * SELECT * ... for counting.
     *
     * The query will be replaced with SELECT count(*)... at execution time.
     */
    public long count() {
        return repository.count(new ByPersonUsingTypedNativeSpecification());
    }

    /**
     * Uses {@link org.extension.spring.data.repository.specification.NativeQuerySpecification} with
     * SELECT * ... for counting.
     *
     * The query will be replaced with SELECT count(*)... at execution time.
     */
    public long countBy(final Person person) {
        return repository.count(new ByPersonUsingTypedQuerySpecification(person);
    }

...

```

[Comments here](https://github.com/brunojensen/brunojensen.github.com/issues)
