## 服务引擎指南

### 简介 

服务框架是OFBiz  2.0  新增加的功能。服务定义为一段独立的逻辑程序，当多个服务组合在一起时可完成不同类型的业务需求。服务有很多类型：Workflow,  Rules,  Java,  SOAP,  BeanShell  等。Java  类型的服务更像一个静态方法实现的事件，然而使用服务框架就不会局限在Web应用程序中。服务需要使用Map传入参数，结果同样从Map中返回。这样很妙，因为Map可以被序列化并保存或者通过HTTP(SOAP)传输。服务通过HYPERLINK  "../../../Documents  and  Settings/ljc/×ÀÃæ/services.html"  \l  "ServiceDefinition"服务定义来定义并指派给具体的HYPERLINK  "../../../Documents  and  Settings/ljc/×ÀÃæ/services.html"  \l  "ServiceEngine"服务引擎。每个HYPERLINK  "../../../Documents  and  Settings/ljc/×ÀÃæ/services.html"  \l  "ServiceEngine"服务引擎  通过适当的方式负责调用服务定义。因为服务没有和  web应用程序绑定在一起，就允许在没有响应对象可用时仍可以执行。这就允许在指定时间由HYPERLINK  "../../../Documents  and  Settings/ljc/×ÀÃæ/services.html"  \l  "JobScheduler"工作调度程序在后台调用服务。

服务能调用其他服务。因此，将多个小的服务串联起来实现一个大的任务使重用更容易。在不同应用程序中使用的服务可以通过创建全局服务定义文件(只能创建一个)或者一个应用程序的特定服务(这样的服务受限制且只能用于这个应用程序)。
当在web应用程序中使用时，服务可以用于web事件，这允许时间在服务框架中(stay  small?)并重用现成的逻辑。同样，服务可以定义成  'exportable'，允许外部程序访问。目前，SOAP  EventHandler允许服务通过  SOAP  来产生。其他形式的远程访问将来会加入到框架中。

### Service  Dispatcher 

Service  Dispatcher将需要处理的服务分配给适当的HYPERLINK  "../../../Documents  and  Settings/Àî¼ª³É/×ÀÃæ/services.html"  \l  "ServiceEngine"服务引擎，在那里服务被调用。每个Entity  Delegator  都有一个具体的  ServiceDispatcher  。应用程序中如果有多个delegator  也应该有多个ServiceDispatcher  。通过LocalDispatcher  访问  ServiceDispatcher  。可能有多个LocalDispatcher关联到一个ServiceDispatcher上。每个LocalDispatcher  都是唯一命名的并包含自己的一些服务定义。当创建  LocalDispatcher  的一个实例，一个  HYPERLINK  "../../../Documents  and  Settings/Àî¼ª³É/×ÀÃæ/services.html"  \l  "DispatchContext"DispatchContext  实例也会被创建并被传给服务引擎。

一个  LocalDispatcher  和一个应用程序关联。应用程序永远不会直接和ServiceDispatcher对话。LocalDispatcher  包含一个API用来调用服务，这些服务通过ServiceDispather发送。然而，应用程序可能运行于不同的线程和实际的  ServiceDispatcher  中。

### Dispatch  Context 

DispatchContext  在LocalDispatcher实例之上由他创建的。这是运行时dispatcher  上下文环境。它包含每个dispatcher处理服务的必须信息。这个上下文包含到每个服务定义文件的引用。

### 服务引擎 

是服务实际被调用的地方。每个服务定义都要定义一个引擎名。引擎名定义在  servicesengine.xml  文件中当调用时通过GenericEngineFactory  实现。支持第三方引擎，但是必须实现  GenericEngine  接口。参照实体引擎配置指南看定义引擎的详细信息。调用同步和异步服务是引擎的工作。使用  HYPERLINK  "../../../Documents  and  Settings/Àî¼ª³É/×ÀÃæ/services.html"  \l  "JobScheduler"Job  Scheduler调用异步服务的引擎可以从GenericAsyncEngine派生得到。

### 服务定义

服务定义在服务定义文件中。有全局(global  )定义文件，所有服务派遣者都可以调用，同时也有只和单一服务派遣者相关联单独服务定义文件。当LocalDispatcher  被创建，他会传递指向服务定义文件的  Arils  的一个集合。这些文件由XML写成，并定义了调用一个服务的必须信息。请参照相关  HYPERLINK  "E:\\dtds\\services.dtd"DTD  文件。

服务定义有一个唯一名字，明确的服务引擎名，明确定义的输入输出参数。下面是个例子。

&lt;service  name="userLogin"  engine="java"
        	location="org.ofbiz.commonapp.security.login.LoginServices"  invoke="userLogin"&gt;
        &lt;description&gt;Authenticate  a  username/password;  create  a  UserLogin  object&lt;/description&gt;
        &lt;attribute  name="login.username"  type="String"  mode="IN"/&gt;
        &lt;attribute  name="login.password"  type="String"  mode="IN"/&gt;
        &lt;attribute  name="userLogin"  type="org.ofbiz.core.entity.GenericValue"  mode="OUT"  optional="true"/&gt;
&lt;/service&gt;

SERVICE  元素:

name  -  服务的唯一名字。 

engine  -  服务引擎的名字  (在  servicesengine.xml  中定义) 

location  -  服务类的包或其位置。 

invoke  -  服务的方法名。 

auth  -  服务是否需要验证(true/false) 

export  -  是否通过  SOAP/HTTP/JMS  (true/false)  访问。

validate  -  是否对下面属性的名字和类型进行验证(true/false) 

IMPLEMENTS  元素:

sevice  -  这个服务实现的服务的名字。所有属性都被继承。 

ATTRIBUTE  元素:

name  -  这个属性的名字 

type  -  对象的类型  (String,  java.util.Date,  等。) 

mode  -  这个参数是输入、输出或输入输出类型。(IN/OUT/INOUT) 

optional  -  这个参数是否可选(true/false) 

*下划线标注的值是默认值。

由上面可以看出服务名是  userLogin，使用  java  引擎。这个服务需要两个必须的输入参数：login.username  和  login.password。必须的参数在服务调用之前会做检验。如果参数和名字及对象类型不符服务就不会被调用。参数是否应该传给服务定义为optional。服务调用后，输出参数也被检验。只有需要的参数被检验，但是，如果传递了一个没有定义为可选的参数或者必须的参数没有通过校验，将会导致服务失败。这个服务没有要求输出参数，因此只是简单返回。

### 用法

服务框架内部的用法非常简单。在  Web  应用程序中，LocalDispatcher  被保存在  ServletContext  中，ServletContext  可以在事件中通过访问  Session  对象来访问。对不是基于  web  的应用程序仅仅创建了一个  GenericDispatcher。(在web.xml中可以找到)

GenericDelegator  delegator  =  GenericDelegator.getGenericDelegator("default");
LocalDispatcher  dispatcher  =  new  GenericDispatcher("UniqueName",  delegator);

现在我们有了dispatcher  ，可以用来调用服务。为了调用这个服务，为  context  创建一个  Map  包含一个输入参数  message,  然后调用这个服务：

Map  context  =  UtilMisc.toMap("message","This  is  a  test.");
Map  result  =  null;

try  {

    result  =  dispatcher.runSync("testScv",  context);
}
catch  (GenericServiceException  e)  {
    e.printStackTrace();
}
if  (result  !=  null)
    System.out.println("Result  from  service:  "  +  (String)  result.get("resp"));

现在查看控制台看测试服务的回复信息。

***  The  test  service  is  located  in  core/docs/examples/ServiceTest.java  you  must  compile  this  and  place  it  in  the  classpath.

安排一个服务在稍晚点时间运行或者重复使用：

//  This  example  will  schedule  a  job  to  run  now.
Map  context  =  UtilMisc.toMap("message","This  is  a  test.");
try  {
    long  startTime  =  (new  Date()).getTime();
    dispatcher.schedule("testScv",  context,  startTime);
}
catch  (GenericServiceException  e)  {
    e.printStackTrace();
}

//  This  example  will  schedule  a  service  to  run  now  and  repeat  once  every  5  seconds  a  total  of  10  times.
Map  context  =  UtilMisc.toMap("message","This  is  a  test.");
try  {
    long  startTime  =  (new  Date()).getTime();
    int  frequency  =  RecurrenceRule.SECONDLY;
    int  interval  =  5;
    int  count  =  10;
    dispatcher.schedule("testScv",  context,  startTime,  frequency,  interval,  count);
}
catch  (GenericServiceException  e)  {
    e.printStackTrace();
}

## 高级特性

服务引擎中加入了很多'高级'特性，在下面有例子、定义及信息。

### 接口

interface  服务引擎实现了在定义服务时可以共享同样的参数。一个接口服务不能被调用，而是为其他服务继承而定义的。每个接口服务都需要用interface  引擎来定义：

&lt;service  name="testInterface"  engine="interface"  location=""  invoke=""&gt;
        &lt;description&gt;A  test  interface  service&lt;/description&gt;
        &lt;attribute  name="partyId"  type="String"  mode="IN"/&gt;
        &lt;attribute  name="partyTypeId"  type="String"  mode="IN"/&gt;
        &lt;attribute  name="userLoginId"  type="org.ofbiz.core.entity.GenericValue"  mode="OUT"  optional="true"/&gt;
&lt;/service&gt;

**注意到location  和  invoke  和在DTD中定义为必须的，因此用做  interface  时使用空串。

现在定义一个服务来实现这个接口

&lt;service  name="testExample1"  engine="simple" 
    	location="org/ofbiz/commonapp/common/SomeTestFile.xml" 

invoke="testExample1"&gt;
        &lt;description&gt;A  test  service  which  implements  testInterface&lt;/description&gt;
        &lt;implements  service="testInterface"/&gt;     

&lt;/service&gt;

testExample1  服务将会和  testInterface  服务拥有完全一样的需要或可选的属性。任何实现testInterface  的服务都将继承其参数/属性。如果需要给指定的服务增加附加属性，可以在  implements  标签后面跟上  attribute  标签。可以在  implements  标签后面重定义某个属性达到重写一个属性的目的。

### ECAs

ECA  (Event  Condition  Action)  更象是一个触发器。当一个服务被调用时，会执查看是否为这个事件定义任何ECAs  。在验证之前，检验之前，事件在实际调用之前，在输出参数校验之前，在事务提交之前或者在服务返回之前包含进来。然后每个条件都会进行验证，如果全部返回为真，定义的动作就会执行。一个动作就是一个服务，该服务的参数必须已经存在于服务的上下文中。每个ECA可以定义的条件数或者动作数没有限制。

&lt;service-eca&gt;

        &lt;eca  service="testScv"  event="commit"&gt;
                &lt;condition  field-name="message"  operator="equals"  value="12345"/&gt;
                &lt;action  service="testBsh"  mode="sync"/&gt;
        &lt;/eca&gt;
&lt;/service-eca&gt;

eca  标签：

<table border><tr><td width=1102>属性名</td><td width=920>需要?</td><td width=6558>描述</td><tr><td width=1102>service</td><td width=920>Y</td><td width=6558>ECA关联的服务名</td><tr><td width=1102>event</td><td width=920>Y</td><td width=6558>ECA在哪个事件上或之前执行。事件有：auth, in-validate, out-validate, invoke, commit, 或者 return。</td><tr><td width=1102>run-on-error</td><td width=920>N</td><td width=6558>当有错误时是否执行ECA (默认为 false)</td></table>

eca  元素应该有0或多个condition/condition-field  元素，1或多个action元素。

condition  标签

<table border><tr><td width=963>属性名</td><td width=805>需要?</td><td width=6812>描述</td><tr><td width=963>map-name</td><td width=805>N</td><td width=6812>本服务上下文属性的名字，包含要检验的字段名字组成的Map。如果没有指定这个域名，就会使用服务上下文环境（env-name）。</td><tr><td width=963>field-name</td><td width=805>Y</td><td width=6812>要比较的map的名字。</td><tr><td width=963>operator</td><td width=805>Y</td><td width=6812>指定比较操作符，必须为 less, greater, less-equals, greater-equals, equals, not-equals, 或 contains 当中的一个。</td><tr><td width=963>value</td><td width=805>Y</td><td width=6812>字段要比较的值。必须为String,但是可以转换成其他类型。</td><tr><td width=963>type</td><td width=805>N</td><td width=6812>用来进行比较的数据类型。必须是 String, Double, Float, Long, Integer, Date, Time, 或 Timestamp 当中的一个。</td><tr><td width=963>format</td><td width=805>N</td><td width=6812>指定当将String 转换成其他类型(主要是Date, Time 和 Timestamp)时使用的格式说明。</td></table>

condition-field  标签

<table border><tr><td width=1196>属性名</td><td width=714>需要?</td><td width=6670>描述</td><tr><td width=1196>map-name</td><td width=714>N</td><td width=6670>本服务上下文属性的名字，包含要检验的字段名字组成的Map。如果没有指定这个域名，就会使用服务上下文环境（env-name）。</td><tr><td width=1196>field-name</td><td width=714>Y</td><td width=6670>要比较的map的名字。</td><tr><td width=1196>operator</td><td width=714>Y</td><td width=6670>指定比较操作符，必须为 less, greater, less-equals, greater-equals, equals, not-equals, 或 contains 当中的一个。</td><tr><td width=1196>to-map-name</td><td width=714>N</td><td width=6670>本服务上下文属性的名字，包含要与之比较的字段名字组成的Map。如果为空就会使用上面的map-name，如果map-name也为空，就会使用服务下文环境（env-name）。</td><tr><td width=1196>to-field-name</td><td width=714>N</td><td width=6670>要与之比较的Map中要比较的字段名，如果为空默认为上面field-name。</td><tr><td width=1196>type</td><td width=714>N</td><td width=6670>用来进行比较的数据类型。必须是 String, Double, Float, Long, Integer, Date, Time, 或 Timestamp 当中的一个。如果没指定默认为String。</td><tr><td width=1196>format</td><td width=714>N</td><td width=6670>指定当将String 转换成其他类型(主要是Date, Time 和 Timestamp)时使用的格式说明。</td></table>

action  标签

<table border><tr><td width=1446>属性名</td><td width=576>需要?</td><td width=6558>描述</td><tr><td width=1446>service</td><td width=576>N</td><td width=6558>本动作(action)要调用的服务名。</td><tr><td width=1446>mode</td><td width=576>Y</td><td width=6558>调用服务的方式，可以是sync 或 async。async actions 将不会更新 context 即使 result-to-context 设置为 true.</td><tr><td width=1446>result-to-context</td><td width=576>N</td><td width=6558>action 服务的执行结果是否更新服务的上下文(context)，默认为 true。</td><tr><td width=1446>ignore-error</td><td width=576>N</td><td width=6558>是否忽略本action 服务导致的错误，如果否原始服务就会失败。默认为 true。</td><tr><td width=1446>persist</td><td width=576>N</td><td width=6558>action 服务store/run。可以为 true 或 false。 只有当 mode 属性为 async 时才生效。默认为false。</td></table>

### 服务组

服务组是由多个服务组成的服务集，当调用初始化组服务时应该运行。使用组服务定义文件定义一个组服务，包含这个组服务所有服务需要的参数/属性。location  属性不需要，invoke  属性定义了要运行的组服务名。当这个组服务被调用时，组中定义的所有服务都会被调用。

组的定义很简单，他包含一个拥有多个  service  元素的  group  元素。group  元素包含一个  name  属性和一个  mode  属性，  mode   用来定义服务怎么执行。service   元素更像  ECA   中的action元素，不同之处在于  result-to-context  属性的默认值。

&lt;service-group&gt;
        &lt;group  name="testGroup"  send-mode="all"&gt;
                &lt;service  name="testScv"  mode="sync"/&gt;
                &lt;service  name="testBsh"  mode="sync"/&gt;
        &lt;/group&gt;
&lt;/service-group&gt;

group  标签

<table border><tr><td width=990>属性名</td><td width=748>需要?</td><td width=6842>描述</td><tr><td width=990>name</td><td width=748>Y</td><td width=6842>要调用的服务组的名字。</td><tr><td width=990>send-mode</td><td width=748>N</td><td width=6842>这些服务被调用的方式。可为：none, all, first-available, random, 或 round-robin。默认为all。</td></table>service  标签

<table border><tr><td width=1446>属性名</td><td width=747>需要?</td><td width=6387>描述</td><tr><td width=1446>service</td><td width=747>N</td><td width=6387>这个服务组要调用的服务名。</td><tr><td width=1446>mode</td><td width=747>Y</td><td width=6387>这个服务被调用的方式。可为sync 或 async。async 将不会更新 context 即使 result-to-context 设置为 true。</td><tr><td width=1446>result-to-context</td><td width=747>N</td><td width=6387>组服务执行的结果是否更新服务的上下文(context)，默认为 false.。</td></table>###4、路由服务(Route  services) 

路由服务使用路由服务引擎定义。当一个路由服务被调用时，不会执行调用，但是所有定义的  ECA  会在适当事件中运行。这种类型的服务不常用，但是通过利用ECA服务选项可以路由(  'route')  到其他服务。

### HTTP  服务

使用HTTP服务是调用定义在其他系统上远程服务的一种方法。本地定义应该和远程定义一致，但是引擎应该是http，location  应该是httpService  事件在远程系统上运行的完全URL，方法应该是远程系统上被调用运行的服务名。远程系统必须有挂在  HTTP服务上公允的  httpService  事件。默认情况下，commonapp  web  应用程序有用来接收服务请求的这样的事件。在远程系统上的服务必须将export属性设为true允许远程调用。HTTP  服务本质就是同步的。

### JMS  服务

JMS  服务和  HTTP  服务很相似，除了服务请求被发送到JMS  topic/queue。engine  属性应该设置为  jms，location  属性因该设置为在serviceengine.xml  文件中定义的  JMS  服务名(HYPERLINK  "../../../Documents  and  Settings/Àî¼ª³É/×ÀÃæ/serviceconfig.html"  \l  "JMS"服务配置)。方法应该是你请求要执行的远程系统上的  JMS  服务名。本质就是异步的。

## 服务引擎配置指南

### 简介

本篇文章描述服务引擎的设置。开始介绍总体思想，然后介绍serviceengine.xml的每部分并结识可用的元素及其用法。serviceengine.xml文件为不同用途提供有例子，文件位于ofbiz/commonapp/etc/serviceengine.xml。
服务引擎的设置通过一个叫做serviceengine.xml的简单XML文件来完成，必须位于  classpath某处。

###  验证

authorization  标签设置服务授权需要调用的服务。这个标签只有一个属性  service-name；属性值应该是用来授权的服务名。默认定义为使用通用OFBiz  userLogin  服务。 

###  线程池

工作调度器(job  scheduler)异步调用工作/服务。它包含池化的线程和几个请求线程。thread-pool  标签用来配置每个线程怎么操作。有如下属型可用。

<table border><tr><td width=1134>属性名</td><td width=746>需要?</td><td width=3854>描述</td><tr><td width=1134>Ttl</td><td width=746>Y</td><td width=3854>每个请求线程的存活时间。达到时间线程将被销毁。</td><tr><td width=1134>wait-millis</td><td width=746>Y</td><td width=3854>每个请求线程在检查通过运行前休眠的时间。</td><tr><td width=1134>jobs</td><td width=746>Y</td><td width=3854>每个请求线程在销毁之前可运行的工作数。</td><tr><td width=1134>min-threads</td><td width=746>Y</td><td width=3854>线程池中保持的请求线程的最小数。</td><tr><td width=1134>max-threads</td><td width=746>Y</td><td width=3854>线程池中将会创建请求线程的最大数。</td><tr><td width=1134>poll-enabled</td><td width=746>Y</td><td width=3854>为'true'scheduler 就会poll数据库来调度工作。</td><tr><td width=1134>poll-db-millis</td><td width=746>Y</td><td width=3854>如果线程池可用，本属性用来定义池化线程运行的频率。</td></table>

### 引擎定义

每一个  GenericEngine  接口的实现都需要在服务定义中定义，engine  标签有如下属性： 

<table border><tr><td width=572>属性名</td><td width=510>需要?</td><td width=2654>描述</td><tr><td width=572>name</td><td width=510>Y</td><td width=2654>服务引擎的名字。必须唯一。</td><tr><td width=572>class</td><td width=510>Y</td><td width=2654>GenericEngine 接口的实现类。</td></table>

### 资源加载器

resource-loader  标签用来设置一个指定的资源加载器以在其他地方加载XML文件和其他资源。有如下属性：

<table border><tr><td width=1112>属性名</td><td width=510>需要?</td><td width=6958>描述</td><tr><td width=1112>name</td><td width=510>Y</td><td width=6958>资源加载器的名字。用于其他标签的 'loader' 属性。</td><tr><td width=1112>class</td><td width=510>Y</td><td width=6958>通用抽象类 org.ofbiz.core.service.config.ResourceLoader 的扩展类。可用类包括 FileLoader, UrlLoader, 和 ClasspathLoader，同类 ResourceLoader 位于同一个包中。</td><tr><td width=1112>prepend-env</td><td width=510>N</td><td width=6958>Java环境属性的名称。用来放在全部路径(full location)比较靠前的地方，在前缀前面。可选。</td><tr><td width=1112>prefix</td><td width=510>N</td><td width=6958>当拼装全部路径(full location)时，放在前面的字符串。可选。如果使用了prepended 环境属性，将会置于其后并位于每个指定资源位置的前面。</td></table>

### 全局服务

global-services  标签用来定义服务定义文件的位置。有如下属性：

<table border><tr><td width=716>属性名</td><td width=510>需要?</td><td width=3753>描述</td><tr><td width=716>loader</td><td width=510>Y</td><td width=3753>前面 resource-loader 标签定义的资源加载器。</td><tr><td width=716>location</td><td width=510>Y</td><td width=3753>指明资源加载器加载资源要使用的文件的位置。</td></table>

### 服务组

service-groups  标签用来定义服务组定义文件的位置。有如下属性：

<table border><tr><td width=716>属性名</td><td width=510>需要?</td><td width=3753>描述</td><tr><td width=716>loader</td><td width=510>Y</td><td width=3753>前面 resource-loader 标签定义的资源加载器。</td><tr><td width=716>location</td><td width=510>Y</td><td width=3753>指明资源加载器加载资源要使用的文件的位置。</td></table>

### ECAs

service-ecas  标签用来定义服务条件触发动作定义文件的位置。有如下属性： 

<table border><tr><td width=716>属性名</td><td width=510>需要?</td><td width=3753>描述</td><tr><td width=716>loader</td><td width=510>Y</td><td width=3753>前面 resource-loader 标签定义的资源加载器。</td><tr><td width=716>location</td><td width=510>Y</td><td width=3753>指明资源加载器加载资源要使用的文件的位置。</td></table>
###  JMS

jms-service  标签为JMS定义服务的位置。 

<table border><tr><td width=990>属性名</td><td width=497>需要?</td><td width=7093>描述</td><tr><td width=990>name</td><td width=497>Y</td><td width=7093>JMS服务的名字，在服务定义中作为 location 的值。</td><tr><td width=990>send-mode</td><td width=497>Y</td><td width=7093>向定义的服务发送的模式有：none, all, first-available, random, round-robin, 或 least-load。</td></table>

jms-service  可以包含一个或多个  server  标签，  server  标签有如下属性：

<table border><tr><td width=1486>属性名</td><td width=510>需要?</td><td width=3953>描述</td><tr><td width=1486>jndi-server-name</td><td width=510>Y</td><td width=3953>在 jndiservers.xml 文件中定义的 JNDI 服务名字。</td><tr><td width=1486>jndi-name</td><td width=510>Y</td><td width=3953>在 JNDI 中为 JMS 工厂定义的名字。</td><tr><td width=1486>topic-queue</td><td width=510>Y</td><td width=3953>主题或队列( topic or queue)的名字。</td><tr><td width=1486>type</td><td width=510>Y</td><td width=3953>JMS 类型可能为主题或队列( topic or queue)。</td><tr><td width=1486>username</td><td width=510>Y</td><td width=3953>连接主题/队列(topic/queue)的用户名。</td><tr><td width=1486>password</td><td width=510>Y</td><td width=3953>连接主题/队列(topic/queue)的密码。</td><tr><td width=1486>listen</td><td width=510>Y</td><td width=3953>设置是否对主题/队列(topic/queue)起用监听。</td></table>
在  jndiservers.xml  文件中定义的  jndi-server  应该指出JMS  客户端  APIs  的位置。根据定义的  JMS  类型来决定使用  TopicConnectionFactory  或  QueueConnectionFactory  。JNDI  名字应该指出在  JNDI  中包含连接工厂实例的对象名字。
