## 服务定义

### 服务定义文件

ofbiz-component.xml：所有的服务定义文件在每个组件的ofbi-component.xml文件中

例:framework/common/ofbiz-component.xml


     <entity-resource  type="model"  reader-name="main"  loader="main"  location="entitydef/entitymodel.xml"/> 
     <entity-resource  type="model"  reader-name="main"  loader="main"  location="entitydef/entitymodel_olap.xml"/>   
     <entity-resource  type="group"  reader-name="main"  loader="main"  location="entitydef/entitygroup_olap.xml"/>   
     <entity-resource  type="data"  reader-name="seed"  loader="main"   location="data/CommonSecurityData.xml"/>   
     <entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CommonSystemPropertyData.xml"/>   
     <entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CommonTypeData.xml"/>   
     <entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CountryCodeData.xml"/>   
     <entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CurrencyData.xml"/>   


服务参数

**IN：输入参数**
类似于java方法中的入参一样，一个ofibiz服务同样需要声明入参，声明方式为在attribute中添加属性mode="IN"

例：

    <attribute  name="partyId"  type="String"  mode="IN"  optional="true"/>   


**OUT：输出参数**

类似于java方法中会有返回值一样，一个ofbiz服务也需要声明输出参数，声明方式为在attribute中添加属性mode="OUT"

例：

     <attribute  name="noteId"  type="String"  mode="OUT"/>   

 **INOUT：输入输出参数**

为了方便起见，可以将一个参数既作为输入参数，又作为输出参数，声明方式为在attribute中添加属性mode="INOUT"

例：

<attribute  name="nodeId"  type="String"  mode="INOUT"/>   


**隐含参数 **

如同servlet内置对象，在ofbiz服务中有少数参数已经在框架中定义了，无需声明就可以在服务中使用

<table border><tr><td width=2871>name</td><td width=4580>type</td><td width=854>mode</td><td width=1320>optional</td><td width=4580>说明</td><tr><td width=2871>responseMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>返回信息（success/error/fail)</td><tr><td width=2871>errorMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>错误信息</td><tr><td width=2871>errorMessageList</td><td width=4580>java.util.List</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>错误信息列表</td><tr><td width=2871>successMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>成功信息</td><tr><td width=2871>successMessageList</td><td width=4580>java.util.List</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>成功信息列表</td><tr><td width=2871>userLogin</td><td width=4580>org.ofbiz.entity.GenericValue</td><td width=854>INOUT</td><td width=1320>true</td><tr><tr><td width=2871>locale</td><td width=4580>java.util.Locale</td><td width=854>INOUT</td><td width=1320>true</td><td width=4580>国际化本地对象</td></table> 

**可选/必选参数**

服务的参数可以是可选，也可以是必须。必须输入的参数在服务调用之前会做检验，如果输入参数的名字及对象类型与声明不符，服务就不会被调用（报错）。必须的输出参数在服务调用之后会被检验，如果输出参数的名字及对象类型与声明不符，会导致服务失败（报错）。只有声明为必须的参数才被检验，同样如果你传入了一个没有意义的参数，也将导致服务调用失败。声明方式为在attribute中添加属性optional="true"/optional="false"（缺省）


例：


    <attribute  name="partyId"  type="String"  mode="IN"  optional="true"/>   
    <attribute  name="partyId"  type="String"  mode="IN"  optional="false"/>   

---

## Interface服务引擎

interface服务引擎实现了在定义服务时可以共享同样的参数。interface服务是不可以被调用的，它仅作为其他服务继承而定义。每个接口服务都需要用interface服务引擎来定义

例：applications/order/servicedef/services.xml


    <service  name="massOrderChangeInterface"  engine="interface"  location=""  invoke="">   
     <description>Interface  for  Mass  Order  Change  Services</description>   
     <attribute  name="orderIdList"  type="List"  mode="IN"  optional="false"/>   
    </service>   


继承上述的接口来实现新的服务

例：applications/order/servicedef/services.xml


     <service  name="massPickOrders"  engine="java"  transaction-timeout="300"   
     location="org.ofbiz.order.order.OrderServices"  invoke="massPickOrders"  auth="true">   
     <implements  service="massOrderChangeInterface"/>   
     </service>   
     <service  name="massChangeOrderApproved"  engine="java"  transaction-timeout="300"   
     location="org.ofbiz.order.order.OrderServices"  invoke="massChangeApproved"  auth="true">   
     <implements  service="massOrderChangeInterface"/>   
     </service>   
     <service  name="massProcessOrders"  engine="java"  transaction-timeout="300"   
     location="org.ofbiz.order.order.OrderServices"  invoke="massProcessOrders"  auth="true">   
     <implements  service="massOrderChangeInterface"/>   
     </service>   


然后我们分别查看java类org.ofbiz.order.order.OrderServices中的方法massPickOrders、massChangeApproved、massProcessOrders

代码片中都存在如下代码，可以获取参数接口定义的变量orderIdList


    List<String>  orderIds  =  UtilGenerics.checkList(context.get("orderIdList"));   



覆盖接口，覆盖接口中的属性或新增属性

例：applications/order/servicedef/services.xml


    <service  name="massPrintOrders"  engine="java"  transaction-timeout="300"   
        location="org.ofbiz.order.order.OrderServices"  invoke="massPrintOrders"  auth="true">   
    <implements  service="massOrderChangeInterface"/>   
     <attribute  name="screenLocation"  type="String"  mode="IN"  optional="false"/>   
    <attribute  name="printerName"  type="String"  mode="IN"  optional="true"/>   
    </service> 

 

 查看代码java类org.ofbiz.order.order.OrderServices中的方法massPrintOrders


    String  screenLocation  =  (String)  context.get("screenLocation");
    String  printerName  =  (String)  context.get("printerName");
    //  make  the  list  per  facility
     List<String>  orderIds  =  UtilGenerics.checkList(context.get("orderIdList"));


---


## 实现服务

实现一个服务可以用一个或多个服务引擎来实现，这里描述下常用的实现方法

### java：用java代码实现服务

首先定义java类型的服务

服务定义：applications/party/servicedef/services_view.xml

    <service  name="getPerson"  engine="java"
     location="org.ofbiz.party.party.PartyServices"  invoke="getPerson">
    <description>Gets  a  person  entity  from  the  cache/database</description>
    <attribute  name="partyId"  type="String"  mode="IN"/>
     <attribute  name="lookupPerson"  type="org.ofbiz.entity.GenericValue"  mode="OUT"/>
    </service>



** engine：  java **

location：指向带包路径的类名

invoke：静态的方法名称

代码

1、参数通过Map方式传递给java方法

2、在java事件中GenericValue类型的userLogin、Locale类型的locale作为属性加入到request中

3、第一个参数DispatchContext包含访问数据库、调用其他服务的工具

4、从java代码中如下访问便利的对象


    GenericValue  userLogin  =  (GenericValue)context.get("userLogin");
    Locale  locale  =  (Locale)context.get("locale");
    Delegator  delegator  =  dctx.getDelegator();



** 安全和访问控制**

例：详细参考/applications/order/src/org/ofbiz/order/order/OrderLookupServices.java


    public  static  Map<String,  Object>  findOrders(DispatchContext  dctx,
        Map<String,  ? extends  Object>  context)  {
    Security  security  =  dctx.getSecurity();
     //  check  security  flag  for  purchase  orders
    boolean  canViewPo  =  security.hasEntityPermission("ORDERMGR",  "_PURCHASE_VIEW",  userLogin);
     if  (!canViewPo)  {
       conditions.add(EntityCondition.makeCondition("orderTypeId",  EntityOperator.NOT_EQUAL,  "PURCHASE_ORDER"));
     }
    }


** 服务返回  **

服务必须返回一个map，这个map至少包含一个responseMessage

ModelService.RESPOND_SUCCESS

ModelService.RESPOND_ERROR

ModelService.RESPOND_FAIL

simple:使用minilang开发的服务详细使用方法参考后续的《minilang开发  学习笔记  》系列  
脚本：applications/party/servicedef/services.xml  


     <!--  Party  Relationship  services  --><br/>
    <service  name="createPartyRelationship"  default-entity-name="PartyRelationship"  engine="simple"
    location="component://party/script/org/ofbiz/party/party/PartyServices.xml"  invoke="createPartyRelationship"  auth="true"><br/>
     <description>
     Create  a  Relationship  between  two  Parties;
     if  partyIdFrom  is  not  specified  the  partyId  of  the  current  userLogin  will  be  used;
     if  roleTypeIds  are  not  specified  they  will  default  to  "_NA_".
     If  a  partyIdFrom  is  passed  in,  it  will  be  used  if  the  userLogin  has  PARTYMGR_REL_CREATE  permission.
     </description><br/>
     <permission-service  service-name="partyRelationshipPermissionCheck"  main-action="CREATE"/><br/>
     <auto-attributes  include="pk"  mode="IN"  optional="true"/><br/>
     <auto-attributes  include="nonpk"  mode="IN"  optional="true"/>
     <override  name="partyIdTo"  optional="false"/>
    </service>

** engine：  simple **

location：实现文件的全路径

invoke：simple-method


    <!--  PartyRelationship  services  --><br/>
    <simple-method  method-name="createPartyRelationship"  short-description="createPartyRelationship"><br/>
    <if-empty  field="parameters.roleTypeIdFrom"><set  field="parameters.roleTypeIdFrom"  value="_NA_"/></if-empty><br/>
    <if-empty  field="parameters.roleTypeIdTo"><set  field="parameters.roleTypeIdTo"  value="_NA_"/></if-empty><br/>
    <if-empty  field="parameters.partyIdFrom"><set  field="parameters.partyIdFrom"  from-field="userLogin.partyId"/></if-empty><br/>
    <if-empty  field="parameters.fromDate"><now-timestamp  field="parameters.fromDate"/></if-empty><br/>
     <!--  check  if  not  already  exist  --><br/>
    <entity-and  entity-name="PartyRelationship"  list="partyRels"  filter-by-date="true">
     <field-map  field-name="partyIdFrom"  from-field="parameters.partyIdFrom"/>
    <field-map  field-name="roleTypeIdFrom"  from-field="parameters.roleTypeIdFrom"/>
     <field-map  field-name="roleTypeIdTo"  from-field="parameters.roleTypeIdTo"/>
    </entity-and><br/>
    <if-empty  field="partyRels">
     <make-value  value-field="newEntity"  entity-name="PartyRelationship"/>
    <set-pk-fields  map="parameters"  value-field="newEntity"/>
     <set-nonpk-fields  map="parameters"  value-field="newEntity"/>
    <create-value  value-field="newEntity"/>
     </if-empty>
    </simple-method>

 
** entity-auto服务 **

例：specialpurpose/example/servicedef/services.xml


     <!--  Example  &amp;  Related  Services  --><br/>
    <service  name="createExample"  default-entity-name="Example"  engine="entity-auto"  invoke="create"  auth="true">
     <description>Create  a  Example</description>
    <permission-service  service-name="exampleGenericPermission"  main-action="CREATE"/>
    <auto-attributes  include="pk"  mode="OUT"  optional="false"/>
    <auto-attributes  include="nonpk"  mode="IN"  optional="true"/>
     <override  name="exampleTypeId"  optional="false"/>
     <override  name="statusId"  optional="false"/>
     <override  name="exampleName"  optional="false"/>
    </service>



### RMI服务

例：applications/accounting/servicedef/services_rita.xml


    <!--  RiTA  (Remote)  Implementations  --><br/>
     <service  name="ritaCCAuthRemote"  engine="rmi"
     location="rita-rmi"  invoke="ritaCCAuth">
       <description>RiTA  Credit  Card  Pre-Authorization/Sale</description>
       <implements  service="ccAuthInterface"/>
     </service>


** engine：rmi **

location：rita-rmi这个是是在serviceengine.xml中配置的

    <service-location  name="rita-rmi"  location="rmi://localhost:1099/RMIDispatcher"/>
    invoke：ritaCCAuth  服务名，该服务如下，也在applications/accounting/servicedef/services_rita.xml中
    <!--  RiTA  (Local)  Implementations  -->
     <service  name="ritaCCAuth"  engine="java"  export="true"
     location="org.ofbiz.accounting.thirdparty.gosoftware.RitaServices"  invoke="ccAuth">
       <description>RiTA  Credit  Card  Pre-Authorization/Sale</description>
       <implements  service="ccAuthInterface"/>
     </service>


设置为可被远程调用  export=“true”

 

### route  服务

这类类型不常用，路由服务使用路由引擎定义，当一个路由服务被调用时，不会执行调用，但是所有定义的ECA会在适当事件中运行。通过利用ECA服务选项可以路由（‘route‘）到其他服务

 

###  http服务

使用http服务是调用定义在其他系统上远程服务的一种方法。本地定义应该和远程定义一致，但是引擎应该是http，location应该是httpService事件在远程系统上运行的完全URL，方法应该是远程系统上被调用运行的服务名。远程系统必须有挂在HTTP服务上公允的httpService事件。默认情况下，commonapp  web应用程序有用来接收服务请求的这样的事件。在远程系统上的服务必须将export属性设为true允许远程调用。HTTP服务本质就是同步的。

 

###  JMS服务

JMS服务和HTTP服务很相似，除了服务请求被发送到JMS  topic/queue。engine属性应该设置为jms，location属性应该设置为在serviceengine.xml文件中定义的JMS服务名。方法应该是你请求要执行的远程系统上的JMS服务名。本质就是异步

 

### 服务组

服务组由多个服务组成的集合。

组的定义：包含一个拥有多个service元素的group元素。group元素包含name属性和mode属性

mode用来定义服务怎么执行

service：类似于ECA中的action元素，不同之处在于resutl-to-context属性的默认值不同。

例：applications/workeffort/servicedef/service_groups.xml


    <service-group  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="http://ofbiz.apache.org/dtds/service-group.xsd">
     <group  name="updateWorkEffortAndAssoc"  send-mode="all"    >
      <invoke  name="updateWorkEffort"  mode="sync"/>
       <invoke  name="updateWorkEffortAssoc"  mode="sync"/>
     </group>
     <group  name="createWorkEffortRequestItemAndRequestItem"  send-mode="all"    >
       <invoke  name="checkCustRequestItemExists"  mode="sync"    result-to-context="true"/>
       <invoke  name="createWorkEffortRequestItem"  mode="sync"/>
     </group>
    </service-group>


 

## 调用服务

调用一个存在的服务是非常简单的，可以通过webtools的Service  Engine  Tools查看具体的服务信息

### 使用java方式调用服务

例：applications/content/src/org/ofbiz/content/search/SearchEvents.java


     public  static  String  indexTree(HttpServletRequest  request,  HttpServletResponse  response)  {
        Map<String,  Object>  result;
        Map<String,  Object>  serviceInMap  =  FastMap.newInstance();
        HttpSession  session  =  request.getSession();
        GenericValue  userLogin  =  (GenericValue)session.getAttribute("userLogin");
        serviceInMap.put("userLogin",  userLogin);
        LocalDispatcher  dispatcher  =  (LocalDispatcher)  request.getAttribute("dispatcher");
        Map<String,  Object>  paramMap  =  UtilHttp.getParameterMap(request);
        String  siteId  =  (String)paramMap.get("contentId");
      serviceInMap.put("contentId",  siteId);
      try  {
 			  result  =  dispatcher.runSync("indexTree",  serviceInMap);
      }  catch  (GenericServiceException  e)  {
           String  errorMsg  =  "Error  calling  the  indexTree  service."  +  e.toString();
           Debug.logError(e,  errorMsg,  module);
           request.setAttribute("_ERROR_MESSAGE_",  errorMsg  +  e.toString());
           return  "error";
      }
      String  errMsg  =  ServiceUtil.getErrorMessage(result);
      if  (Debug.infoOn())  Debug.logInfo("errMsg:"  +  errMsg,  module);
      if  (Debug.infoOn())  Debug.logInfo("result:"  +  result,  module);
      if  (UtilValidate.isEmpty(errMsg))  {
       List<String>  badIndexList  =  UtilGenerics.checkList(result.get("badIndexList"));
       if  (Debug.infoOn())  Debug.logInfo("badIndexList:"  +  badIndexList,  module);
       String  badIndexMsg  =  StringUtil.join(badIndexList,  "\n")  +  badIndexList.size()  +  "  entities  not  indexed";
       Integer  goodIndexCount  =  (Integer)result.get("goodIndexCount");
       String  goodIndexMsg  =  goodIndexCount  +  "  entities  indexed.";
       if  (Debug.infoOn())  Debug.logInfo("goodIndexCount:"  +  goodIndexCount,  module);
       ServiceUtil.setMessages(request,  badIndexMsg,  goodIndexMsg,  null);
       return  "success";
          }  else  {
       ServiceUtil.setMessages(request,  errMsg,  null,  null);
       return  "error";
          }
       }


 
调用服务


     try  {
       result  =  dispatcher.runSync("indexTree",  serviceInMap);
          }  catch  (GenericServiceException  e)  {
       String  errorMsg  =  "Error  calling  the  indexTree  service."  +  e.toString();
       Debug.logError(e,  errorMsg,  module);
       request.setAttribute("_ERROR_MESSAGE_",  errorMsg  +  e.toString());
       return  "error";
      }


 

如上调用了indxTree服务，该服务在applications/content/servicedef/services.xml中被定义


     <service  name="indexTree"  auth="true"  engine="java"  validate="true"  transaction-timeout="7200"
      location="org.ofbiz.content.search.SearchServices"  invoke="indexTree">
       <description>Index  content  under  publish  point</description>
       <attribute  mode="IN"  name="contentId"  optional="false"  type="String"/>
       <attribute  mode="OUT"  name="badIndexList"  optional="true"  type="List"/>
       <attribute  mode="OUT"  name="goodIndexCount"  optional="true"  type="Integer"/>
     </service>

 

 

###  smiple-method：使用simple-method方式调用服务

例：applications/accounting/script/org/ofbiz/accounting/payment/PaymentServices.xml

     <simple-method  method-name="createPaymentAndApplication"  short-description="Create  a  payment  and  a  payment  application  for  the  full  amount">
     <set-service-fields  service-name="createPayment"  map="parameters"  to-map="createPaymentInMap"/>
     <call-service  service-name="createPayment"  in-map-name="createPaymentInMap">
   <result-to-field  field="paymentId"  result-name="paymentId"/>
    </call-service>
     <check-errors/>
     <set-service-fields  service-name="createPaymentApplication"  map="parameters"  to-map="createPaymentAppInMap"/>
     <set  field="createPaymentAppInMap.paymentId"  from-field="paymentId"/>
     <set  field="createPaymentAppInMap.amountApplied"  from-field="parameters.amount"/>
     <call-service  service-name="createPaymentApplication"  in-map-name="createPaymentAppInMap">
   <result-to-field  field="paymentApplicationId"  result-name="paymentApplicationId"/>
     </call-service>
     <check-errors/>
     <field-to-result  field="paymentId"  result-name="paymentId"/>
     <field-to-result  field="paymentApplicationId"  result-name="paymentApplicationId"/>
    </simple-method>


 

调用服务


    <call-service  service-name="createPayment"  in-map-name="createPaymentInMap">
    <result-to-field  field="paymentId"  result-name="paymentId"/>
    </call-service>

 

该服务createPayment定义于applications/accounting/script/org/ofbiz/accounting/payment/PaymentServices.xml


     <simple-method  method-name="createPayment"  short-description="Create  a  Payment">
       <if>
     <condition>
      <and>
        <not><if-has-permission  permission="PAY_INFO"  action="_CREATE"/></not>
        <not><if-compare-field  field="userLogin.partyId"  to-field="parameters.partyIdFrom"  operator="equals"/></not>
        <not><if-compare-field  field="userLogin.partyId"  to-field="parameters.partyIdTo"  operator="equals"/></not>
      </and>
     </condition>
     <then>
      <add-error>
        <fail-property  resource="AccountingUiLabels"  property="AccountingCreatePaymentPermissionError"/>
      </add-error>
     </then>
       </if>
       <check-errors/>
       <make-value  entity-name="Payment"  value-field="payment"/>
       <if-empty  field="parameters.paymentId">
     <sequenced-id  sequence-name="Payment"  field="payment.paymentId"/>
    <else>
      <set  field="payment.paymentId"  from-field="parameters.paymentId"/>
     </else>
       </if-empty>
       <field-to-result  field="payment.paymentId"  result-name="paymentId"/>
       <if-not-empty  field="parameters.paymentMethodId">
     <entity-one  entity-name="PaymentMethod"  value-field="paymentMethod">
      <field-map  field-name="paymentMethodId"  from-field="parameters.paymentMethodId"/>
     </entity-one>
     <if-compare-field  field="parameters.paymentMethodTypeId"  operator="not-equals"  to-field="paymentMethod.paymentMethodTypeId">
      <log  level="info"  message="Replacing  passed  payment  method  type  [${parameters.paymentMethodTypeId}]  with  payment  method  type  [${paymentMethod.paymentMethodTypeId}]  for  payment  method  [${parameters.paymentMethodId}]"/>
      <set  field="parameters.paymentMethodTypeId"  from-field="paymentMethod.paymentMethodTypeId"/>
     </if-compare-field>
       </if-not-empty>
       <if-not-empty  field="parameters.paymentPreferenceId">
     <entity-one  value-field="orderPaymentPreference"  entity-name="OrderPaymentPreference">
      <field-map  field-name="orderPaymentPreferenceId"  from-field="parameters.paymentPreferenceId"/>
     </entity-one>
     <if-empty  field="parameters.paymentMethodId">
     <set  field="parameters.paymentMethodId"  from-field="orderPaymentPreference.paymentMethodId"/>
     </if-empty>
     <if-empty  field="parameters.paymentMethodTypeId">
      <set  field="parameters.paymentMethodTypeId"  from-field="orderPaymentPreference.paymentMethodTypeId"/>
     </if-empty>
       </if-not-empty>
       <if-empty  field="parameters.paymentMethodTypeId">
     <add-error>
      <fail-property  resource="AccountingUiLabels"  property="AccountingPaymentMethodIdPaymentMethodTypeIdNullError"/>
     </add-error>
       </if-empty>
       <set-nonpk-fields  map="parameters"  value-field="payment"/>
       <if-empty  field="payment.effectiveDate">
     <now-timestamp  field="payment.effectiveDate"/>
       </if-empty>
       <create-value  value-field="payment"/>
     </simple-method>


 

### 异步方式调用


    <simple-method  method-name="sendInvoicePerEmail"  short-description="Send  an  invoice  per  Email">
       <set-service-fields  service-name="sendMailFromScreen"  map="parameters"  to-map="emailParams"/>
       <set  field="emailParams.xslfoAttachScreenLocation"  value="component://accounting/widget/AccountingPrintScreens.xml#InvoicePDF"/>
       <set  field="emailParams.bodyParameters.invoiceId"  from-field="parameters.invoiceId"/>
      <set  field="emailParams.bodyParameters.userLogin"  from-field="parameters.userLogin"/>
       <set  field="emailParams.bodyParameters.other"  from-field="parameters.other"/><!--  to  to  print  in  'other  currency'  -->
       <call-service-asynch  service-name="sendMailFromScreen"  in-map-name="emailParams"/>
       <property-to-field  resource="AccountingUiLabels"  property="AccountingEmailScheduledToSend"  field="successMessage"/>
     </simple-method>


 

异步调用代码


     <call-service-asynch  service-name="sendMailFromScreen"  in-map-name="emailParams"/>

 

 

### 服务中的事务申明

一个服务是否使用事务，是否必须是独立的事务，以及事务的超时时间设置，均可以在服务定义时声明，或者在通过服务引擎调用服务时指定（传入）

#### 服务定义时申明事务

例：applications/order/servicedef/services.xml

     <service  name="sendOrderConfirmation"  engine="java"  require-new-transaction="true"  max-retry="3"
     location="org.ofbiz.order.order.OrderServices"  invoke="sendOrderConfirmNotification">
       <description>Send  a  order  confirmation</description>
       <implements  service="orderNotificationInterface"/>
     </service>

 

#### 在服务调用时指定

java代码

    dispatcher.runSync(servicename,  context,transactionTimeOut,requireNewTransaction);
    dispatcher.runAsync(servicename,  context,requester,persist,transactionTimeOut,requireNewTransaction);

 
 

#### 事务的管理规则

1、调用服务时传入的参数（是否为独立事务、事务的超时时间）的优先级高于服务定义时定义的参数
2、若服务在定义时声明了不使用事务（use-transaction="false")，那么无论是定义什么的独立事务和超市时间，还是调用服务的时候定义了独立事务和超市时间，都将不会生效
3、服务被申明为不使用事务，则所有的默认为每次操作位一个独立的事务

 

#### 事务的提交规则

1、没有在服务内部显示操作事务的情况下，事务由服务引擎来管理，当服务返回error时，回滚事务；当服务返回failure或success时提交事务
2、服务间调用使用同一个事务时，内层服务返回failure或success，不提交事务，在最外层服务返回时方提交
3、服务间调用使用同一个事务时，任何一个服务返回error，事务均会回滚。
4、服务间调用使用独立事务时，一个事务回滚或提交不影响其他事务，但一个事务不能访问另一个事务尚未提交的数据
5、服务执行超时，回滚事务

---
## 服务超时

1、服务执行是否超时由服务引擎来控制，超时时间，在定义服务时指定，或调用服务的时候指定，若不指定，默认时间为60秒
2、这个超时时间为服务执行的总时间，包括服务内部执行的其他操作时间的总和
3、如果没有开启事务，则配置超时时间也不会生效。
 

### EEAC、SEAC、MCA概念

EEAC：在实体上触发一个服务的调用

SEAC：当条件满足时，调用另外一个服务

MCA：启动javamail-container
