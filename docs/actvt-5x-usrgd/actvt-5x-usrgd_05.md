# Chapter 10\. JPA

你可以使用 JPA 实体作为流程变量，并且可以这样做：

*   基于流程变量更新已有的 JPA 实体，它可以在用户任务的表单中填写或者由服务任务生成。
*   重用已有的领域模型不需要编写显示的服务获取实体或者更新实体的值。
*   根据已有实体的属性做出判断（网关即分支聚合）。
*   ...

# Requirements 要求

## 10.1\. Requirements 要求

只支持符合以下要求的实体：

*   实体应该使用 JPA 注解进行配置，我们支持字段和属性访问两种方式。映射的超级累类也能够被使用。
*   实体中应该有一个使用 `@Id` 注解的主键，不支持复合主键 (`@EmbeddedId` 和 `@IdClass`)。Id 字段/属性能够使用 JPA 规范支持的任意类型： 原生态数据类型和他们的包装类型（boolean 除外），`String`, `BigInteger`, `BigDecimal`, `java.util.Date` 和 `java.sql.Date`.

# Configuration 配置

## 10.2\. Configuration 配置

为了能够使用 JPA 的实体，引擎必须有一个对 `EntityManagerFactory`的引用。 这可以通过配置引用或者提供一个持久化单元名称。作为变量的 JPA 实体将会被自动检测并进行相应的处理

下面例子中的配置是使用 jpaPersistenceUnitName：

```java
<bean id="processEngineConfiguration"
  class="org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">

<!-- Database configurations -->
<property name="databaseSchemaUpdate" value="true" />
<property name="jdbcUrl" value="jdbc:h2:mem:JpaVariableTest;DB_CLOSE_DELAY=1000" />

<property name="jpaPersistenceUnitName" value="activiti-jpa-pu" />
<property name="jpaHandleTransaction" value="true" />
<property name="jpaCloseEntityManager" value="true" />

<!-- job executor configurations -->
<property name="jobExecutorActivate" value="false" />

<!-- mail server configurations -->
<property name="mailServerPort" value="5025" />
</bean> 
```

接下来例子中的配置提供了一个我们自定义的 `EntityManagerFactory`(在这个例子中，使用了 OpenJPA 实体管理器)。注意该代码片段仅仅包含与例子相关的 beans，去掉了其他 beans。OpenJPA 实体管理的完整并可以使用的例子可以在 activiti-spring-examples(`/activiti-spring/src/test/java/org/activiti/spring/test/jpa/JPASpringTest.java`)中找到。

```java
<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
  <property name="persistenceUnitManager" ref="pum"/>
  <property name="jpaVendorAdapter">
    <bean class="org.springframework.orm.jpa.vendor.OpenJpaVendorAdapter">
      <property name="databasePlatform" value="org.apache.openjpa.jdbc.sql.H2Dictionary" />
    </bean>
  </property>
</bean>

<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
  <property name="dataSource" ref="dataSource" />
  <property name="transactionManager" ref="transactionManager" />
  <property name="databaseSchemaUpdate" value="true" />
  <property name="jpaEntityManagerFactory" ref="entityManagerFactory" />
  <property name="jpaHandleTransaction" value="true" />
  <property name="jpaCloseEntityManager" value="true" />
  <property name="jobExecutorActivate" value="false" />
</bean> 
```

同样的配置也可以在编程式创建一个引擎时完成，例如：

```java
ProcessEngine processEngine = ProcessEngineConfiguration
.createProcessEngineConfigurationFromResourceDefault()
.setJpaPersistenceUnitName("activiti-pu")
.buildProcessEngine(); 
```

配置属性：

*   jpaPersistenceUnitName: `使用持久化单元的名称（要确保该持久化单元在类路径下是可用的）。根据该规范，默认的路径是/META-INF/persistence.xml)。要么使用 jpaEntityManagerFactory` 或者`jpaPersistenceUnitName`。
*   jpaEntityManagerFactory: `一个实现了 javax.persistence.EntityManagerFactory 的 bean 的引用`。它将被用来加载实体并且刷新更新。要么使用 jpaEntityManagerFactory 或者 jpaPersistenceUnitName。
*   jpaHandleTransaction: 在被使用的 EntityManager 实例上，该标记表示流程引擎是否需要开始和提交/回滚事物。当使用 Java 事物 API（JTA）时，设置为 false。
*   jpaCloseEntityManager: `该标记表示流程引擎是否应该关闭从 EntityManagerFactory` 获取的 EntityManager 的实例。当 EntityManager 是由容器管理的时候需要设置为 false（例如 当使用并不是单一事物作用域的扩展持久化上下文的时候）。

# Usage 使用

## 10.3\. Usage 用法

### 10.3.1\. 简单例子

使用 JPA 变量的例子可以在 JPAVariableTest 中找到。我们将会一步一步的解释 `JPAVariableTest.testUpdateJPAEntityValues`。

首先，我们需要创建一个基于 `META-INF/persistence.xml` 的 EntityManagerFactory 作为我们的持久化单元。它包含持久化单元中所有的类和一些供应商特定的配置。

我们将使用一个简单的实体作为测试，其中包含有一个 id 和 `String` 类型的 value 属性，这也将会被持久化。在允许测试之前，我们创建一个实体并且保存它。

```java
@Entity(name = "JPA_ENTITY_FIELD")
public class FieldAccessJPAEntity {

  @Id
  @Column(name = "ID_")
  private Long id;

  private String value;

  public FieldAccessJPAEntity() {
    // Empty constructor needed for JPA
  }

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getValue() {
    return value;
  }

  public void setValue(String value) {
    this.value = value;
  }
} 
```

我们开始一个新的流程实例，添加实体作为变量。与其它的变量一样，它们存储在引擎的持久存储区。当下次这个变量被请求，它会从基于类和 Id 的存储的 `EntityManager` 中加载。

```java
Map<String, Object> variables = new HashMap<String, Object>();
variables.put("entityToUpdate", entityToUpdate);

ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("UpdateJPAValuesProcess", variables); 
```

在我们的流程定义的第一个节点包含一个 `serviceTask` 将调用方法 在 `entityToUpdate` 上 `setValue`，它解析为 JPA 变量，我们启动流程实例并将从相关联的当前引擎的上下文“EntityManager”进行加载。

```java
<serviceTask id='theTask' name='updateJPAEntityTask'
  activiti:expression="${entityToUpdate.setValue('updatedValue')}" /> 
```

当 service-task 完成后，流程实例等在流程定义中定义的 userTask，这使我们能够检查流程实例。在这一点上，EntityManager 已刷新并更改到实体已经被推到数据库。当我们得到变量 entityToUpdate 值时，它再次加载，我们得到实体，并将实体中的属性 `value` 设置到 `updatedValue` 。

```java
// Servicetask in process 'UpdateJPAValuesProcess' should have set value on entityToUpdate.
Object updatedEntity = runtimeService.getVariable(processInstance.getId(), "entityToUpdate");
assertTrue(updatedEntity instanceof FieldAccessJPAEntity);
assertEquals("updatedValue", ((FieldAccessJPAEntity)updatedEntity).getValue()); 
```

### 10.3.2\. 查询 JPA 处理变量

可以查询 `ProcessInstances` 和 `Execution` 包含 JPA 实体作为变量值。注意 只有 在 ProcessInstanceQuery 和 ExecutionQuery ，`variableValueEquals(name, entity)` 是支持 JPA 实体的 。 方法 `variableValueNotEquals`, `variableValueGreaterThan`, `variableValueGreaterThanOrEqual`, `variableValueLessThan`和 `variableValueLessThanOrEqual` 不支持，并且当一个 JPA 实体传递作为值时，会抛出 `ActivitiException`。

```java
ProcessInstance result = runtimeService.createProcessInstanceQuery()
    .variableValueEquals("entityToQuery", entityToQuery).singleResult(); 
```

### 10.3.3\. 高级例子：使用 Spring bean 和 JPA

JPASpringTest 例子可以在 activiti-spring-examples 中找到。它的使用场景如下：

*   存在一个使用 JPA 实体的 Spring bean，用于存储贷款请求。
*   使用 Activiti，我们可以利用那些由现有的 bean 获得的已有的实体，并将其作为变量在流程中使用。 流程是按以下步骤进行定义的：
    *   服务任务，利用已有的 LoanRequestBean 使用启动流程时接收的变量（比如，可以来源于开始的表单）来创建新的 LoanRequest。使用 activiti:resultVariable（它以一个变量来存储表达式结果）将创建出来的实体以变量进行存储。
    *   用户任务，让经理查看请求，批准/不批准，结果存储到一个 boolean 类型的变量 approvedByManager 上。
    *   服务任务，用来更新贷款请求实体以使该实体与流程同步。
    *   根据实体属性 approved 的值，利用排他分支来决定下一步选择哪条路径：当请求被批准时，流程将结束；否则，会另有一个任务（发送拒绝信），这样就可以由拒绝信手动地通知用户了。

请注意此流程不包含任何表单，所以其只用于单元测试

![](img/eac62021.png)

```java
<?xml version="1.0" encoding="UTF-8"?>
<definitions id="taskAssigneeExample"

  targetNamespace="org.activiti.examples">

  <process id="LoanRequestProcess" name="Process creating and handling loan request">
    <startEvent id='theStart' />
    <sequenceFlow id='flow1' sourceRef='theStart' targetRef='createLoanRequest' />

    <serviceTask id='createLoanRequest' name='Create loan request'
      activiti:expression="${loanRequestBean.newLoanRequest(customerName, amount)}"
      activiti:resultVariable="loanRequest"/>
    <sequenceFlow id='flow2' sourceRef='createLoanRequest' targetRef='approveTask' />

    <userTask id="approveTask" name="Approve request" />
    <sequenceFlow id='flow3' sourceRef='approveTask' targetRef='approveOrDissaprove' />

    <serviceTask id='approveOrDissaprove' name='Store decision'
      activiti:expression="${loanRequest.setApproved(approvedByManager)}" />
    <sequenceFlow id='flow4' sourceRef='approveOrDissaprove' targetRef='exclusiveGw' />

    <exclusiveGateway id="exclusiveGw" name="Exclusive Gateway approval" />
    <sequenceFlow id="endFlow1" sourceRef="exclusiveGw" targetRef="theEnd">
      <conditionExpression xsi:type="tFormalExpression">${loanRequest.approved}</conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="endFlow2" sourceRef="exclusiveGw" targetRef="sendRejectionLetter">
      <conditionExpression xsi:type="tFormalExpression">${!loanRequest.approved}</conditionExpression>
    </sequenceFlow>

    <userTask id="sendRejectionLetter" name="Send rejection letter" />
    <sequenceFlow id='flow5' sourceRef='sendRejectionLetter' targetRef='theOtherEnd' />

    <endEvent id='theEnd' />
    <endEvent id='theOtherEnd' />
  </process>

</definitions> 
```

虽然上面的例子很简单，但确实展示出了结合 Spring 和参数化方法表达式来使用 JPA 的强大。此流程根本不需要定制 java 代码（当然了，除 Spring bean 外），大大加速了部署