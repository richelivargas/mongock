# Version 3.x

![](https://raw.githubusercontent.com/cloudyrock/mongock/master/misc/logo-new.png)

**Mongock** is a java MongoDB tool for tracking, managing and applying database schema changes accross all your environments based on a coding approach.

## Last news ‼ ‼ 💥 💥 

{% hint style="success" %}
**3.3.2** is released with minor bug fixes
{% endhint %}

{% hint style="success" %}
**We are working on version 4!!**...Take a look to [this example](https://github.com/cloudyrock/mongock-integration-tests/tree/master/mongock-spring-v5/mongock-spring5-springdata3-it) and give the version **4.0.8.alpha** a try
{% endhint %}

## Why Mongock

There are several good reasons to use Mongock in your project. Here we give you some of them:

* Solid solution which really works.
* **Works well with sharded collections**: Unlike other similar projects using javascript, which requires `db.eval()`. [Documentation](https://docs.mongodb.com/manual/reference/method/db.eval/#sharded-data)
* Works with [Mongo Atlas](https://www.mongodb.com/cloud/atlas).
* Distributed solution with solid locking mechanism.
* We are very responsive for community license and maximum 2 business days for professional license.
* Well maintained and regularly updated.
* Used by several tech companies in different industries.
* Can be used together with most, if not all, frameworks.
* Provides great integration for [Spring](https://spring.io/), allowing you to inject any dependency you want to your changeLog method.
* We walk with you to production. Get more information about our support model at dev@cloudyrock.io

## Contributing

If you would like to contribute to Mongock project, please read [how to contribute](community/CONTRIBUTING.md) for details on our collaboration process and standards.

## Add a dependency

Mongock can be used standalone or with [Spring](https://spring.io/)/[Spring boot](https://spring.io/projects/spring-boot). When Using `mongock-spring` you don't need to import `mongock-core`, as it's already imported out of the box.

#### With Maven

```markup
<!-- To use standalone -->
<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-core</artifactId>
  <version>{LATEST_VERSION}</version>
</dependency>

<!-- To use with Spring -->
<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-spring</artifactId>
  <version>{LATEST_VERSION}</version>
</dependency>
```

#### With Gradle

```groovy
compile 'org.javassist:javassist:3.18.2-GA'          // workround for ${javassist.version} placeholder issue*
compile 'com.github.cloudyrock.mongock:mongock-core:{LATEST_VERSION}'    // standalone
compile 'com.github.cloudyrock.mongock:mongock-spring:{LATEST_VERSION}'  // with Spring (in addition to mongock-core)
```

## Key concepts

| Concept | Mongock Module | Type | Description |
| :--- | :--- | :--- | :--- |
| `@ChangeLog` | Common | Annotation | Annotation used to annotate a class that contains changeSet methods |
| `@ChangeSet` | Common | Annotation | Annotation used to annotate a a method inside a class annotated with `@ChangeLog` and contains the code for a specific change |
| `Mongock` | Standalone | Class | It's the class in charge of running the migration as standalone, independently from any framework |
| `MongockBuiler` | Standalone | Class | It's the class used to build a Mongock instance |
| `SpringMongock` | Spring | Class | A Mongock extension for [Spring framework](https://spring.io/), allowing what a normal Mongock runner does, plus the use of MongoTemplate, SpringData and Spring Profiles |
| `SpringMongockBuiler` | Spring | Class | It's the class used to build a SpringMongock instance |
| `SpringBootMongock` | Spring boot | Class | A SpringMongock extension for [Spring boot](https://spring.io/projects/spring-boot) framework, allowing what a SpringMongock runner does, plus Spring ApplicationContext for custom injection in your ChangeSet methods |
| `SpringBootMongockBuiler` | Spring boot | Class | It's the class used to build a SpringBootMongock instance |

## How it works

How Mongock works is very simple. The migration consists in three main elements: Your changes\(based on ChangeLog classes that contains one or more ChangeSet methods\), Mongock builder and Mongock runner. To understand these terms, please refer to section [Key concepts]().

Basic steps for you to understand the process are: 1. Create your ChangeLog class with ChangeSet methods in a package, which we'll call it "changes\_package" 2. Build the Mongock runner with Mongock Builder. Among other parameters, you will need to provide your "changes\_package" 3. Run Mongock object built in the previous step. You will see that this step will be automatically performed when used in frameworks like [Spring](https://spring.io/), [JHipster](https://www.jhipster.tech/), etc.

## Creating change logs

`ChangeLog` contains a bunch of `ChangeSet`s. `ChangeSet` is a single task \(set of instructions made on a database\). In other words `ChangeLog` is a class annotated with `@ChangeLog` and containing methods annotated with `@ChangeSet`.

```java
package com.example.yourapp.changelogs;

@ChangeLog
public class DatabaseChangelog {

  @ChangeSet(order = "001", id = "someChangeId", author = "testAuthor")
  public void importantWorkToDo(DB db){
     // task implementation
  }


}
```

### @ChangeLog

Class with change sets must be annotated by `@ChangeLog`. There can be more than one change log class but in that case `order` argument should be provided:

```java
@ChangeLog(order = "001")
public class DatabaseChangelog {
  //...
}
```

ChangeLogs are sorted alphabetically by `order` argument and changesets are applied due to this order.

### @ChangeSet

Method annotated by @ChangeSet is taken and applied to the database. History of applied change sets is stored in a collection called `dbchangelog` \(by default\) in your MongoDB

#### Annotation parameters:

| Parameter | Type | Mandatory | Considerations | Description |
| :--- | :--- | :--- | :--- | :--- |
| `id` | String | YES | Must be Unique | Name of a change set, **must be unique** for all change logs in a database |
| `author` | String | YES | N/A | Author of a change set |
| `order` | String | Optional | Alphabetic order | String for sorting change sets in one changelog. Sorting in alphabetical order, ascending. It can be a number, a date etc. |
| `runAlways` | Boolean | Optional | default: false | ChangeSet will always be executed but only first execution event will be stored in dbchangelog collection |
| `version` | String | Optional | default: "0" | efines a version on which this changeset is relate to. E.g. "0.1" means, this changeset should be applied to schema version 0.1 of your MongoDB. See [Defining ChangeSet methods with versions]() for more information. |

#### Defining ChangeSet methods

Method annotated by `@ChangeSet` can have one of the following definition:

```java
@ChangeSet(order = "001", id = "someChangeWithoutArgs", author = "testAuthor")
public void someChange1() {
   // method without arguments can do some non-db changes
}

@ChangeSet(order = "002", id = "someChangeWithMongoDatabase", author = "testAuthor")
public void someChange2(MongoDatabase db) {
  // type: com.mongodb.client.MongoDatabase : original MongoDB driver v. 3.x, operations allowed by driver are possible
  // example: 
  MongoCollection<Document> mycollection = db.getCollection("mycollection");
  Document doc = new Document("testName", "example").append("test", "1");
  mycollection.insertOne(doc);
}

@ChangeSet(order = "005", id = "someChangeWithSpringDataTemplate", author = "testAuthor")
public void someChange5(MongoTemplate mongoTemplate) {
  // type: org.springframework.data.mongodb.core.MongoTemplate
  // Spring Data integration allows using MongoTemplate in the ChangeSet
  // example:
  mongoTemplate.save(myEntity);
}

@ChangeSet(order = "006", id = "someChangeWithSpringDataTemplate", author = "testAuthor")
public void someChange6(MongoTemplate mongoTemplate, Environment environment) {
  // type: org.springframework.data.mongodb.core.MongoTemplate
  // type: org.springframework.core.env.Environment
  // Spring Data integration allows using MongoTemplate and Environment in the ChangeSet
}
```

#### Defining ChangeSet methods with versions

Method annotated by `@ChangeSet` have also the possibility to contain a version. This is a useful feature from a consultancy point of view. The more descriptive scenario is where a software provider has several customers who he provides his software to. The clients may be using different versions of the software at the same time. So when he install the product in a customer, the changeSets need to be applied depending on the product version. With this solution, he can tag every changeSet with his product version and will tell mongock which version range to apply.

```java
@ChangeSet(order = "001", id = "someChangeToVersionOne", author = "testAuthor", version = "1")
public void someChange1(MongoDatabase db) {
}

@ChangeSet(order = "002", id = "someChangeToVersionOneDotOne", author = "testAuthor", version = "1.1")
public void someChange2(MongoDatabase db) {
}

@ChangeSet(order = "003", id = "someChangeToVersionTwoDotFiveDotOne", author = "testAuthor", systemVersion = "2.5.1")
public void someChange3(MongoDatabase db) {
}

@ChangeSet(order = "004", id = "someChangeToVersionTwoDotFiveDotFive", author = "testAuthor", systemVersion = "2.5.5")
public void someChange5(MongoDatabase db) {
}

@ChangeSet(order = "005", id = "someChangeToVersionTwoDotSix", author = "testAuthor", systemVersion = "2.6")
public void someChange6(MongoDatabase db) {
}
```

With specifying versions you are able to upgrade to specific versions:

```java
  MongoClient mongoclient = new MongoClient(new MongoClientURI("yourDbName", yourMongoClientBuilder));
  Mongock runner=  new MongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setLockQuickConfig()
      .setStartSystemVersion("1")
      .setEndSystemVersion("2.5.5")
      .build();
  runner.execute();         //  ------> starts migration changesets from systemVersion 1 to 2.5.5
```

This example will execute `ChangeSet` 1, 2 and 3, because the specified systemVersion in the changeset should be greater equals the `startSystemVersion` and lower than `endSystemVersion`.

## When use each module

### When use Mongock standalone

Mongock standalone\(`mongock-core`\) is the original Mongock implementation. It provides the main features and while it still very easy to run, it does not rely on any framework to be executed, so that responsibility belongs to you.

As mentioned, it's recommended when you are running an standalone java application without any framework or, even using a framework, you want to have the control of when Mongock is executed and you don't need any extra feature provided by Spring, such as bean injection, profiles, converters, etc.

### When use SpringMongock

This is a specialisation of Mongock for [Spring](https://spring.io/) users. SpringMongock runner implements the interface [InitializingBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html) provided by [Spring](https://spring.io/). So by instantiating the Mongock runner as singleton bean in the spring context, the framework will take care of the dependencies and will execute at startup.

By using SpringMongock you can also take advantage of the Spring Profile's benefits and, if using MongoTemplate, all the SpringData features are implicitly provided.

> **Note:** We really recommend using [MongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) for building SpringMongock runner. It provides a better integration with [Spring](https://spring.io/) and it's the only way of taking advantage of SpringData, so you can use features like Mongo [CustomConversions](https://docs.spring.io/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/convert/CustomConversions.html), etc.

### When use SpringBootMongock

The main benefit of using SpringBoot integration is that it provides a totally flexible way to inject dependencies, so you can inject any object to your changeSets by using [Spring boot](https://spring.io/projects/spring-boot) [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html).

In order to use this feature you need to instantiate the SpringBootMongock runner and provide the required configuration. Mongock will run as an [ApplicationRunner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/ApplicationRunner.html) within [Spring boot](https://spring.io/projects/spring-boot). In terms of execution, it will be very similar to the standard Spring implementation, the key difference is that ApplicationRunner beans run _after_ \(as opposed to during\) the context is fully initialized.

> **Note:** We really recommend using [MongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) for building SpringBootMongock runner. It provides a better integration with Spring boot and it's the only way of taking advantage of SpringData, so you can use features like Mongo [CustomConversions](https://docs.spring.io/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/convert/CustomConversions.html), etc.
>
> **Note:** Using this implementation means you need all the dependencies in your changeLogs\(parameters in methods annotated with `@ChangeSet`\) declared as Spring beans.
>
> **Note:** In order to have your beans injected into your changeSet methods, you need to provide the [SpringBoot](https://spring.io/projects/spring-boot) [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) at building time.
>
> **Important note:** Currently\(we are working on it, please take a look to our [Roadmap]()\) the dependencies injected by the ApplicationContext \(other than [MongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html) and [MongoDatabase](http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/client/MongoDatabase.html)\) won't be covered by the lock. This means that if you are accessing to Mongo through a different mechanism to the ones mentioned, the lock synchronization is not guaranteed as Mongock only ensures synchronization when Mongo is accessed via either [MongoTemplate](https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html), [MongoDatabase](http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/client/MongoDatabase.html) or [DB](http://mongodb.github.io/mongo-java-driver/3.6/javadoc/?com/mongodb/DB.html). For more information, please consult the [lock section]()

## Build parameters

### Common build parameters

| Parameter | Injection type | Default | Description |
| :--- | :--- | :--- | :--- |
| `mongoClient` | constructor | N/A | New database connection client API. Recommended for standalone use over `legacyMongoClient` |
| `legacyMongoClient` | constructor | N/A | Old database connection client API. Deprecated |
| `databaseName` | constructor | Mandatory | Database name |
| `changeLogScanPackage` | constructor | Mandatory | Package path where the changeLogs are located |
| `enabled` | setter | `true` | Migration process will run only if this option is set to true |
| `changeLogCollectionName` | setter | `mongockChangeLog` | ChangeLog collection name |
| `lockCollectionName` | setter | `mongockLock` | Lock collection name |
| `metadata` | setter | empty Map | Map for custom data you want to attach to your migration |
| `throwExceptionIfCannotObtainLock` | String | `false` | Mongock will throw MongockException if lock can not be obtained |
| `lockAcquiredForMinutes` | setter | 24 \* 60\(1 day\) | Indicates how long the lock will be hold once acquired in minutes |
| `maxWaitingForLockMinutes` | setter | 3 | Indicates max time in minutes to wait for the lock in each try |
| `maxTries` | setter | 1 | Number of times Mongock will try to acquire the lock |
| `startSystemVersion` | setter | `"0"` | System version to start with \(see [Defining ChangeSet methods with versions\(\)]()\) |
| `endSystemVersion` | setter | Integer.MAX\_VALUE | System version to end with \(see [Defining ChangeSet methods with versions\(\)]()\) |

### SpringMongock build parameters

Provides all the commons parameters plus:

| Parameter | Injection type | Default | Description |
| :--- | :--- | :--- | :--- |
| `mongoTemplate` | constructor | N/A | Spring MongoTemplate. Recommended for Spring and Spring boot use. Without this, specific Spring feature related to Mongo, like Spring converters, repositories and so on, won't work |
| `springEnvironment` | setter | null | Spring Environment. Recommended for Spring. Without this Spring Profiles won't work. Not needed in Spring boot if ApplicationContext is provided |

### SpringBootMongock build parameters

Provides all the commons and Spring parameters plus:

| Parameter | Injection type | Default | Description |
| :--- | :--- | :--- | :--- |
| `springContext` | setter | null | Spring Application context. Recommended. Required for bean injections into ChangeSet methods. If this is provided, Spring Environment is not needed. |

## Build and run

Once we have seen how to create our migration classes in [Creating change logs](), we need to build and run Mongock. As mentioned, we have three modes: standalone, Spring and Spring boot

> **Note:** For backward compatibility, the lock is configured by default with too long values\(24 hours\). This is really basic strategy which can produce unexpected behaviour like not been able to execute Mongock in the next 24hours if there is any error and the lock is not released. For this reason we RECOMMEND to activate it by either setting the values manually with `setLockConfig` or with the default values for a proper lock mechanism with `setLockQuickConfig`

by default the lock is configured with values\(24 hours\) that in practice is like . This means that the lock is there but the time is taken is so long that it's like you have no lock, or even worse, if anything happens and the lock is not released, the next execution won't be able to run before 24 hours.

### Build and run standalone

For more attributes, please see [Build parameters]()

```java
  Mongock runner=  new MongockBuilder(mongoclient, "databaseName", "changeLogScanPackage")
      .setLockQuickConfig() OR .setLockConfig(3, 4, 3) 
      ...other setters
      .build();
  runner.execute();         //  ------> starts migration changeSets straight away
```

### Build and run SpringMongock

If you use Spring, mongock can be instantiated as a singleton bean in the Spring context. In this case the migration process will be executed automatically on startup. You can use MongoClient, like with standalone, but we recommend using MongoTemplate.

For more attributes, please see [Build parameters]()

```java
@Bean
public SpringMongock springMongock(MongoTemplate mongoTemplate, Environment springEnvironment) {
  return new SpringMongockBuilder(mongoTemplate, "changeLogScanPackage")
        .setSpringEnvironment(springEnvironment)
      .setLockQuickConfig() OR .setLockConfig(3, 4, 3) 
       ...other setters
      .build();
}
```

### Build and run SpringBootMongock

You can see the benefits of using SpringBootMongock in [this section]() By injecting the Spring ApplicationRunner, you are implicitly injecting the Spring Environment, so you don't need to do it manually.

As mentioned in previous sections, with SpringBootMongock now you can use your beans in your changeSets, please see [this section]() to discover how

For more attributes, please see [Build parameters]()

```java
@Bean
public SpringBootMongock mongock(MongoTemplate mongoTemplate, ApplicationContext springContext) {
  return new SpringBootMongockBuilder(mongoTemplate, "changeLogScanPackage")
      .setApplicationContext(springContext) 
      .setLockQuickConfig() OR .setLockConfig(3, 4, 3) 
      .build();
}
```

## Spring specific features

### Injecting custom dependencies with Spring Boot

Right now this is only possible by using SpringBootMongock\(no SpringMongock\) and injecting Spring Application Context to your builder as shown below. As explained in section [Build and run SpringBootMongock](), once you have injected the Spring ApplicationContext, you can use your beans in Mongock changeSet methods via method parameter. Don't use @autowired annotation.

For example having a springdata repository 'PersonRepository' in your project, that you wish to use in your changeSet, you can use it like follow

```java
@ChangeLog(order = "1")
@Profile("test")
public class ChangelogForTestEnv{
  @ChangeSet(author = "testuser", id = "myTestChangest", order = "01")
  public void testingEnvOnly(MongoTemplate template, PersonRepository repository){
    List<Person> allPersons = repository.findAll();
  } 
}
```

> **Note:** You shouldn't use the repository to write to Mongo, as it won't be covered by the lock.\(this feature which allows you to use your own repositories freely in your changeSet methods will be available in future releases\)

### Using Spring profiles

This feature available for both Springs builders, SpringMongock and SpringBootMongock

**mongock** accepts Spring's `org.springframework.context.annotation.Profile` annotation. If a change log or change set class is annotated with `@Profile`, then it is activated for current application profiles.

> Mongock will soon support the new Profile expression approach from Spring 5 in

#### Annotating ChangeLogs and ChangeSets with Profile

_Example 1_: annotated change set will be invoked for a `dev` profile

```java
@Profile("dev")
@ChangeSet(author = "testuser", id = "myDevChangest", order = "01")
public void devEnvOnly(MongoDatabase db){
  // ...
}
```

_Example 2_: all change sets in a changelog will be invoked for a `test` profile

```java
@ChangeLog(order = "1")
@Profile("test")
public class ChangelogForTestEnv{
  @ChangeSet(author = "testuser", id = "myTestChangest", order = "01")
  public void testingEnvOnly(MongoDatabase db){
    // ...
  } 
}
```

#### Enabling Profile annotation on Mongock

This feature available for both Springs builders, SpringMongock and SpringBootMongock

To enable the `@Profile` integration, please inject `org.springframework.core.env.Environment` to your runner.

```java
@Bean 
@Autowired
public SpringMongock mongock(Environment environment) {
  SpringMongock runner = new SpringMongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
      .setSpringEnvironment(environment)
      .setLockQuickConfig()
      .build();

  //... etc
}
```

## Adding metadata

Sometimes there is the need of adding some extra information to the mongockChangeLog documents at execution time. This is address by Mongock allowing to set a Map object to the MongockBuilder\(core, Spring and Springboot\) with the metadata which will be added later to each mongockChangeLog document inserted in the Mongock 'transaction'.

```java
Map<String, Object> metadata = new HashMap<>();
metadata.put("string_key", "string_value");

Mongock runner = new MongockBuilder(mongoclient, "yourDbName", "com.package.to.be.scanned.for.changesets")
    .setLockQuickConfig()
    .withMetadata(metadata)
    .build();

  //... etc
}
```

## Parallel process

Currently Mongock does not support parallel process in the same JVM. This means if you are thinking to create two or more Mongock instances in order to run them simultaneously, this wil produce unexpected results, specially related to lock synchronization.

However, this feature is in our road map for the next release. This roadmap and the expected release date will be published soon.

## Configuring Lock

In order to execute the changelogs, mongock needs to manage the lock to ensure only one instance executes a changelog at a time. By default the lock is reserved 24 hours and, in case the lock is held by another mongock instance, will ignore the execution and no exception will be sent, unless the parameter throwExceptionIfCannotObtainLock is set to true.

There are 3 parameters to configure:

| Parameter | Description |
| :--- | :--- |
| `lockAcquiredForMinutes` | Number of minutes mongock will acquire the lock for. It will refresh the lock when is close to be expired anyway. |
| `maxTries` | Max tries when the lock is held by another mongock instance. |
| `maxWaitingForLockMinutes` | Max minutes mongock will wait for the lock in every try. |

To configure these parameters there are two methods: setLockConfig and `setLockConfig` and `setLockQuickConfig`. Both will set the parameter throwExceptionIfCannotObtainLock to true.

```java
 @Bean @Autowired
 public Mongock mongock(Environment environment) {
   Mongock runner = new mongock(uri);
   runner.setLockConfig(5, 6, 3);
 }
```

or quick config with 3 minutes for lockAcquiredFor, 3 max tries and 4 minutes for maxWaitingForLock

```java
  @Bean @Autowired
  public Mongock mongock(Environment environment) {
    Mongock runner = new mongock(uri);
    runner.setLockQuickConfig();
  }
```

## Known issues

### Mongo java driver conflicts

**mongock** depends on `mongo-java-driver`. If your application has mongo-java-driver dependency too, there could be library conflicts in some cases.

**Exception**:

```text
com.mongodb.WriteConcernException: { "serverUsed" : "localhost" , 
"err" : "invalid ns to index" , "code" : 10096 , "n" : 0 , 
"connectionId" : 955 , "ok" : 1.0}
```

**Workaround**:

You can exclude mongo-java-driver from **mongock** and use your dependency only. Maven and gradle examples below:

```markup
<!-- pom.xml -->
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.4.0</version>
</dependency>

<dependency>
  <groupId>com.github.cloudyrock.mongock</groupId>
  <artifactId>mongock-core</artifactId>
  <version>{LATEST_VERSION}</version>
  <exclusions>
    <exclusion>
      <groupId>org.mongodb</groupId>
      <artifactId>mongo-java-driver</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

```text
    // build.gradle
    compile "org.mongodb:mongo-java-driver:3.4.0"
    compile("com.github.cloudyrock.mongock:mongock:{LATEST_VERSION}") {
        exclude group: 'org.mongodb', module: 'mongo-java-driver'
    }
```

### Mongo transaction limitations

> **Note:** We are working on providing transaction support and UNDO operations. However until then, this will still be relevant and even with that, will remain relevant for older MongoDB versions without transaction support. Please checkout our [roadmap]()

Due to Mongo limitations, there is no way to provide atomicity at ChangelogSet level. So a Changelog could need more than one execution to be finished, as any interruption could happen, leaving the changelog in a inconsistent state. If that happens, the next time mongock is executed it will try to finish the changelog execution, but it could already be half executed.

For this reason, the developer in charge of the changelog's design, should make sure that:

* **Changelog is idempotent**: As changelog can be interrupted at any time, it will need to be executed again. 
* **Changelog is Backward compatible \(If high availability is required\)**: While the migration process is taking place, 

  the old version of the software is still running. During this time could happen\(and probably will\) that the old version 

  of the software is dealing with the new version of the data. Could even happen that the data is a mix between old and 

  new version. This means the software must still work regardless of the status of the database. In case the developer is aware of 

  this and still decides to provide a non-backward-compatible changeSet, he should know it's a detriment to high 

  availability.

* **Changelog reduces its execution time in every iteration**: This is harder to explain. As said, a changelog can be 

  interrupted at any time. This means an specific changelog needs to be re-run. In the undesired scenario where the 

  changelog's execution time is grater than the interruption time\(could be Kubernetes initial delay\), that changelog won't 

  be ever finished. So the changelog needs to be developed in such a way that every iteration reduces its execution time, 

  so eventually, after some iterations, the changelog finished. 

* **Changelog's execution time is shorter than interruption time**: In case the previous condition cannot be ensured, 

  could be enough if the changelog's execution time is shorter than the interruption time. This is not ideal as the 

  execution time depends on the machine, but in most case could be enough.

## Code of conduct

Please read the [code of conduct](community/CODE_OF_CONDUCT.md) for details on our code of conduct.

## LICENSE

Mongock propject is licensed under the [Apache License Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html). See the [LICENSE](LICENSE.md) file for details

