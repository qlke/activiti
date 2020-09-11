# activiti
workflow introduce

# 概述

activiti是一个工作流引擎，用于流程控制，保存流程中间结果，追溯流程历史等

# 概念和表结构

   Activiti数据库支持：

      Activiti的后台是有数据库的支持，所有的表都以ACT_开头。 第二部分是表示表的用途的两个字母标识。 用途也和服务的API对应。

      ACT_RE_*: ‘RE’表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。

      ACT_RU_*: ‘RU’表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。

      ACT_ID_*: ‘ID’表示identity。 这些表包含身份信息，比如用户，组等等。

      ACT_HI_*: ‘HI’表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
      
      ACT_GE_*: 通用数据， 用于不同场景下，如存放资源文件。

   工作流引擎

　　　　ProcessEngine对象，这是Activiti工作的核心。负责生成流程运行时的各种实例及数据、监控和管理流程的运行。

　　    BPMN 业务流程建模与标注（Business Process Model and Notation，BPMN)，描述流程的基本符号，包括这些图元如何组合成一个业务流程图（Business Process Diagram）

　　资源库流程规则表
  
　　　　1) act_re_deployment 部署信息表

　　　　2) act_re_model 流程设计模型部署表

　　　　3) act_re_procdef 流程定义数据表

　　运行时数据库表
  
　　　　1) act_ru_execution 运行时流程执行实例表

　　　　2) act_ru_identitylink 运行时流程人员表，主要存储任务节点与参与者的相关信息

　　　　3) act_ru_task 运行时任务节点表

　　　　4) act_ru_variable 运行时流程变量数据表

　　历史数据库表
  
　　　　1) act_hi_actinst 历史节点表

　　　　2) act_hi_attachment 历史附件表

　　　　3) act_hi_comment 历史意见表

　　　　4) act_hi_identitylink 历史流程人员表

　　　　5) act_hi_detail 历史详情表，提供历史变量的查询

　　　　6) act_hi_procinst 历史流程实例表

　　　　7) act_hi_taskinst 历史任务实例表

　　　　8) act_hi_varinst 历史变量表

　　组织机构表
  
　　　　1) act_id_group 用户组信息表

　　　　2) act_id_info 用户扩展信息表

　　　　3) act_id_membership 用户与用户组对应信息表

　　　　4) act_id_user 用户信息表

　　　　这四张表很常见，基本的组织机构管理，关于用户认证方面建议还是自己开发一套，组件自带的功能太简单，使用中有很多需求难以满足

　　通用数据表
  
　　　　1) act_ge_bytearray 二进制数据表

　　　　2) act_ge_property 属性数据表存储整个流程引擎级别的数据,初始化表结构时，会默认插入三条记录

    初始化

　  activiti.cfg.xml文件用于配置数据库，编写main方法执行
    
    activiti.cfg.xml（activiti的配置文件）
    
    Activiti核心配置文件，配置流程引擎创建工具的基本参数和数据库连接池参数
    
   示例,生成25张表：
    
       <?xml version="1.0" encoding="UTF-8"?>
       <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置数据源 -->
    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url"
                  value="jdbc:mysql://localhost:3306/activiti?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=utf8&amp;nullCatalogMeansCurrent=true&amp;serverTimezone=GMT"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="dataSource" ref="dataSource"></property>
        <property name="databaseSchemaUpdate" value="true"></property>
    </bean>
</beans>

 main:
     
     @Test
     public void testGenTable2() {
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        System.out.println(processEngine);
    }
    
idea配置画图（.bpmn）插件

    目前idea中不存在这款插件，需要到https://plugins.jetbrains.com/ 搜索actiBPM然后下载，再添加到idea中，重启。
　　　　

　　主要接口

　　　　RepositoryService

　　　　　　管理流程定义

　　　　RuntimeService

　　　　　　执行管理，包括启动、推进、删除流程实例等操作

　　　　TaskService

　　　　　　任务管理

　　　　HistoryService

　　　　　　历史管理(执行完的数据的管理)

　　　　IdentityService

　　　　　　组织机构管理

　　　　FormService

　　　　　　一个可选服务，任务表单管理
      
   activiti简单实用：
   
      private ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();

    /**
     * 流程部署
     */
    @Test
    public void processDeploy() {
        RepositoryService repositoryService = processEngine.getRepositoryService();
        Deployment deployment = repositoryService
                .createDeployment().name("请假申请")
                .addClasspathResource("bpmn/leave1.bpmn")
                .deploy();
        System.out.println("部署成功。");
        System.out.println("流程部署名称：" + deployment.getName());
        System.out.println("流程部署ID：" + deployment.getId());
    }

    /**
     * 启动流程
     */
    @Test
    public void startProcess() {
        RuntimeService runtimeService = processEngine.getRuntimeService();
        runtimeService.startProcessInstanceById("审批流程:2:2503");
        System.out.println("流程启动成功。");
    }

    /**
     * 查询任务
     */
    @Test
    public void queryTask() {
        TaskService taskService = processEngine.getTaskService();
        List<Task> tasks = taskService.createTaskQuery().list();
        for (Task t : tasks) {
            System.out.println("任务ID：" + t.getId());
            System.out.println("流程实例ID：" + t.getProcessInstanceId());
            System.out.println("执行实例ID：" + t.getExecutionId());
            System.out.println("流程定义ID：" + t.getProcessDefinitionId());
            System.out.println("任务名称：" + t.getName());
            System.out.println("任务办理人：" + t.getAssignee());
        }
    }

    /**
     * 设置任务办理人
     */
    @Test
    public void setTaskAssignee(){
        TaskService taskService = processEngine.getTaskService();
        taskService.setAssignee("5005","qlke");
    }

    /**
     * 办理任务
     */
    @Test
    public void doTask(){
         TaskService taskService = processEngine.getTaskService();
         taskService.complete("5005");//任务id：act_ru_task任务节点表
        System.out.println("审核通过。");
    }

工作流使用第一步是用插件画流程图，最后保存为bpmn文件，里面是xml定义，然后deploy该文件即在数据库中插入相应的数据，然后在代码中调用相应task，等处理完业务代码后决定是完成该task还是终止流程，如果完成则下一步业务可以启动。
