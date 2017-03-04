## 服务定义

### 服务定义文件

ofbiz-component.xml：所有的服务定义文件在每个组件的ofbi-component.xml文件中

例:framework/common/ofbiz-component.xml

<code>
 &lt;entity-resource  type="model"  reader-name="main"  loader="main"  location="entitydef/entitymodel.xml"/&gt; 
 &lt;entity-resource  type="model"  reader-name="main"  loader="main"  location="entitydef/entitymodel_olap.xml"/&gt;   
&lt;entity-resource  type="group"  reader-name="main"  loader="main"  location="entitydef/entitygroup_olap.xml"/&gt;   
 &lt;entity-resource  type="data"  reader-name="seed"  loader="main"   location="data/CommonSecurityData.xml"/&gt;   
 &lt;entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CommonSystemPropertyData.xml"/&gt;   
 &lt;entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CommonTypeData.xml"/&gt;   
 &lt;entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CountryCodeData.xml"/&gt;   
 &lt;entity-resource  type="data"  reader-name="seed"  loader="main"  location="data/CurrencyData.xml"/&gt;   
</code>

服务参数

**IN：输入参数**
类似于java方法中的入参一样，一个ofibiz服务同样需要声明入参，声明方式为在attribute中添加属性mode="IN"

例：
<code>
&lt;attribute  name="partyId"  type="String"  mode="IN"  optional="true"/&gt;   
</code>

**OUT：输出参数**

类似于java方法中会有返回值一样，一个ofbiz服务也需要声明输出参数，声明方式为在attribute中添加属性mode="OUT"

例：
<code>
&lt;attribute  name="noteId"  type="String"  mode="OUT"/&gt;   
</code>

**INOUT：输入输出参数**

为了方便起见，可以将一个参数既作为输入参数，又作为输出参数，声明方式为在attribute中添加属性mode="INOUT"

例：
<code>
&lt;attribute  name="nodeId"  type="String"  mode="INOUT"/&gt;   
</code>

**隐含参数 **

如同servlet内置对象，在ofbiz服务中有少数参数已经在框架中定义了，无需声明就可以在服务中使用

<table border><tr><td width=2871>name</td><td width=4580>type</td><td width=854>mode</td><td width=1320>optional</td><td width=4580>说明</td><tr><td width=2871>responseMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>返回信息（success/error/fail)</td><tr><td width=2871>errorMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>错误信息</td><tr><td width=2871>errorMessageList</td><td width=4580>java.util.List</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>错误信息列表</td><tr><td width=2871>successMessage</td><td width=4580>String</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>成功信息</td><tr><td width=2871>successMessageList</td><td width=4580>java.util.List</td><td width=854>OUT</td><td width=1320>true</td><td width=4580>成功信息列表</td><tr><td width=2871>userLogin</td><td width=4580>org.ofbiz.entity.GenericValue</td><td width=854>INOUT</td><td width=1320>true</td><tr><tr><td width=2871>locale</td><td width=4580>java.util.Locale</td><td width=854>INOUT</td><td width=1320>true</td><td width=4580>国际化本地对象</td></table> 

**可选/必选参数**

服务的参数可以是可选，也可以是必须。必须输入的参数在服务调用之前会做检验，如果输入参数的名字及对象类型与声明不符，服务就不会被调用（报错）。必须的输出参数在服务调用之后会被检验，如果输出参数的名字及对象类型与声明不符，会导致服务失败（报错）。只有声明为必须的参数才被检验，同样如果你传入了一个没有意义的参数，也将导致服务调用失败。声明方式为在attribute中添加属性optional="true"/optional="false"（缺省）


例：

<code>
&lt;attribute  name="partyId"  type="String"  mode="IN"  optional="true"/&gt;   
&lt;attribute  name="partyId"  type="String"  mode="IN"  optional="false"/&gt;   
</code>


---

## Interface服务引擎

interface服务引擎实现了在定义服务时可以共享同样的参数。interface服务是不可以被调用的，它仅作为其他服务继承而定义。每个接口服务都需要用interface服务引擎来定义

例：applications/order/servicedef/services.xml

<code>
&lt;service  name="massOrderChangeInterface"  engine="interface"  location=""  invoke=""&gt;   
 &lt;description&gt;Interface  for  Mass  Order  Change  Services&lt;/description&gt;   
 &lt;attribute  name="orderIdList"  type="List"  mode="IN"  optional="false"/&gt;   
&lt;/service&gt;   
</code>

继承上述的接口来实现新的服务

例：applications/order/servicedef/services.xml

<code>
&lt;service  name="massPickOrders"  engine="java"  transaction-timeout="300"   
 location="org.ofbiz.order.order.OrderServices"  invoke="massPickOrders"  auth="true"&gt;   
 &lt;implements  service="massOrderChangeInterface"/&gt;   
 &lt;/service&gt;   
 &lt;service  name="massChangeOrderApproved"  engine="java"  transaction-timeout="300"   
 location="org.ofbiz.order.order.OrderServices"  invoke="massChangeApproved"  auth="true"&gt;   
 &lt;implements  service="massOrderChangeInterface"/&gt;   
 &lt;/service&gt;   
 &lt;service  name="massProcessOrders"  engine="java"  transaction-timeout="300"   
 location="org.ofbiz.order.order.OrderServices"  invoke="massProcessOrders"  auth="true"&gt;   
 &lt;implements  service="massOrderChangeInterface"/&gt;   
 &lt;/service&gt;   
</code>

然后我们分别查看java类org.ofbiz.order.order.OrderServices中的方法massPickOrders、massChangeApproved、massProcessOrders

代码片中都存在如下代码，可以获取参数接口定义的变量orderIdList

<code>
  List&lt;String&gt;  orderIds  =  UtilGenerics.checkList(context.get("orderIdList"));   
</code>


覆盖接口，覆盖接口中的属性或新增属性

例：applications/order/servicedef/services.xml

<code>
&lt;service  name="massPrintOrders"  engine="java"  transaction-timeout="300"   
location="org.ofbiz.order.order.OrderServices"  invoke="massPrintOrders"  auth="true"&gt;   
&lt;implements  service="massOrderChangeInterface"/&gt;   
 &lt;attribute  name="screenLocation"  type="String"  mode="IN"  optional="false"/&gt;   
&lt;attribute  name="printerName"  type="String"  mode="IN"  optional="true"/&gt;   
&lt;/service&gt; 
</code>
 

查看代码java类org.ofbiz.order.order.OrderServices中的方法massPrintOrders

<code>
String  screenLocation  =  (String)  context.get("screenLocation");
String  printerName  =  (String)  context.get("printerName");
//  make  the  list  per  facility
 List&lt;String&gt;  orderIds  =  UtilGenerics.checkList(context.get("orderIdList"));
</code>

---


## 实现服务

实现一个服务可以用一个或多个服务引擎来实现，这里描述下常用的实现方法

### java：用java代码实现服务

首先定义java类型的服务

服务定义：applications/party/servicedef/services_view.xml

<code>
 &lt;service  name="getPerson"  engine="java"
 location="org.ofbiz.party.party.PartyServices"  invoke="getPerson"&gt;
&lt;description&gt;Gets  a  person  entity  from  the  cache/database&lt;/description&gt;
&lt;attribute  name="partyId"  type="String"  mode="IN"/&gt;
 &lt;attribute  name="lookupPerson"  type="org.ofbiz.entity.GenericValue"  mode="OUT"/&gt;
&lt;/service&gt;
</code>


** engine：  java **

location：指向带包路径的类名

invoke：静态的方法名称

代码

1、参数通过Map方式传递给java方法

2、在java事件中GenericValue类型的userLogin、Locale类型的locale作为属性加入到request中

3、第一个参数DispatchContext包含访问数据库、调用其他服务的工具

4、从java代码中如下访问便利的对象

<code>
GenericValue  userLogin  =  (GenericValue)context.get("userLogin");
Locale  locale  =  (Locale)context.get("locale");
Delegator  delegator  =  dctx.getDelegator();
</code>


** 安全和访问控制**

例：详细参考/applications/order/src/org/ofbiz/order/order/OrderLookupServices.java

<code>
public  static  Map&lt;String,  Object&gt;  findOrders(DispatchContext  dctx,  Map&lt;String,  ? extends  Object&gt;  context)  {
Security  security  =  dctx.getSecurity();
 //  check  security  flag  for  purchase  orders
boolean  canViewPo  =  security.hasEntityPermission("ORDERMGR",  "_PURCHASE_VIEW",  userLogin);
 if  (!canViewPo)  {
   conditions.add(EntityCondition.makeCondition("orderTypeId",  EntityOperator.NOT_EQUAL,  "PURCHASE_ORDER"));
 }
}
</code>


** 服务返回 **

服务必须返回一个map，这个map至少包含一个responseMessage

ModelService.RESPOND_SUCCESS

ModelService.RESPOND_ERROR

ModelService.RESPOND_FAIL

simple:使用minilang开发的服务详细使用方法参考后续的《minilang开发  学习笔记  》系列

脚本：applications/party/servicedef/services.xml

<code>
 &lt;!--  Party  Relationship  services  --&gt;<br/>
&lt;service  name="createPartyRelationship"  default-entity-name="PartyRelationship"  engine="simple"
location="component://party/script/org/ofbiz/party/party/PartyServices.xml"  invoke="createPartyRelationship"  auth="true"&gt;<br/>
 &lt;description&gt;
 Create  a  Relationship  between  two  Parties;
 if  partyIdFrom  is  not  specified  the  partyId  of  the  current  userLogin  will  be  used;
 if  roleTypeIds  are  not  specified  they  will  default  to  "_NA_".
 If  a  partyIdFrom  is  passed  in,  it  will  be  used  if  the  userLogin  has  PARTYMGR_REL_CREATE  permission.
 &lt;/description&gt;<br/>
 &lt;permission-service  service-name="partyRelationshipPermissionCheck"  main-action="CREATE"/&gt;<br/>
 &lt;auto-attributes  include="pk"  mode="IN"  optional="true"/&gt;<br/>
 &lt;auto-attributes  include="nonpk"  mode="IN"  optional="true"/&gt;
 &lt;override  name="partyIdTo"  optional="false"/&gt;
&lt;/service&gt;
</code>


** engine：  simple **

location：实现文件的全路径

invoke：simple-method

<code>
&lt;!--  PartyRelationship  services  --&gt;<br/>
&lt;simple-method  method-name="createPartyRelationship"  short-description="createPartyRelationship"&gt;<br/>
&lt;if-empty  field="parameters.roleTypeIdFrom"&gt;&lt;set  field="parameters.roleTypeIdFrom"  value="_NA_"/&gt;&lt;/if-empty&gt;<br/>
&lt;if-empty  field="parameters.roleTypeIdTo"&gt;&lt;set  field="parameters.roleTypeIdTo"  value="_NA_"/&gt;&lt;/if-empty&gt;<br/>
&lt;if-empty  field="parameters.partyIdFrom"&gt;&lt;set  field="parameters.partyIdFrom"  from-field="userLogin.partyId"/&gt;&lt;/if-empty&gt;<br/>
 &lt;if-empty  field="parameters.fromDate"&gt;&lt;now-timestamp  field="parameters.fromDate"/&gt;&lt;/if-empty&gt;<br/>
 &lt;!--  check  if  not  already  exist  --&gt;<br/>
&lt;entity-and  entity-name="PartyRelationship"  list="partyRels"  filter-by-date="true"&gt;
 &lt;field-map  field-name="partyIdFrom"  from-field="parameters.partyIdFrom"/&gt;
&lt;field-map  field-name="roleTypeIdFrom"  from-field="parameters.roleTypeIdFrom"/&gt;
 &lt;field-map  field-name="roleTypeIdTo"  from-field="parameters.roleTypeIdTo"/&gt;
&lt;/entity-and&gt;<br/>
 &lt;if-empty  field="partyRels"&gt;
 &lt;make-value  value-field="newEntity"  entity-name="PartyRelationship"/&gt;
&lt;set-pk-fields  map="parameters"  value-field="newEntity"/&gt;
 &lt;set-nonpk-fields  map="parameters"  value-field="newEntity"/&gt;
&lt;create-value  value-field="newEntity"/&gt;
 &lt;/if-empty&gt;
&lt;/simple-method&gt;
</code>
 
** entity-auto服务 **

例：specialpurpose/example/servicedef/services.xml

<code>
 &lt;!--  Example  &amp;  Related  Services  --&gt;<br/>
 &lt;service  name="createExample"  default-entity-name="Example"  engine="entity-auto"  invoke="create"  auth="true"&gt;
 &lt;description&gt;Create  a  Example&lt;/description&gt;
&lt;permission-service  service-name="exampleGenericPermission"  main-action="CREATE"/&gt;
&lt;auto-attributes  include="pk"  mode="OUT"  optional="false"/&gt;
&lt;auto-attributes  include="nonpk"  mode="IN"  optional="true"/&gt;
 &lt;override  name="exampleTypeId"  optional="false"/&gt;
 &lt;override  name="statusId"  optional="false"/&gt;
 &lt;override  name="exampleName"  optional="false"/&gt;
&lt;/service&gt;
</code>


### RMI服务

例：applications/accounting/servicedef/services_rita.xml

<code>
 &lt;!--  RiTA  (Remote)  Implementations  --&gt;<br/>
 &lt;service  name="ritaCCAuthRemote"  engine="rmi"
 location="rita-rmi"  invoke="ritaCCAuth"&gt;
   &lt;description&gt;RiTA  Credit  Card  Pre-Authorization/Sale&lt;/description&gt;
   &lt;implements  service="ccAuthInterface"/&gt;
 &lt;/service&gt;


** engine：rmi ** 

location：rita-rmi这个是是在serviceengine.xml中配置的

&lt;service-location  name="rita-rmi"  location="rmi://localhost:1099/RMIDispatcher"/&gt;
invoke：ritaCCAuth  服务名，该服务如下，也在applications/accounting/servicedef/services_rita.xml中
&lt;!--  RiTA  (Local)  Implementations  --&gt;
 &lt;service  name="ritaCCAuth"  engine="java"  export="true"
 location="org.ofbiz.accounting.thirdparty.gosoftware.RitaServices"  invoke="ccAuth"&gt;
   &lt;description&gt;RiTA  Credit  Card  Pre-Authorization/Sale&lt;/description&gt;
   &lt;implements  service="ccAuthInterface"/&gt;
 &lt;/service&gt;


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


&lt;service-group  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://ofbiz.apache.org/dtds/service-group.xsd"&gt;
 &lt;group  name="updateWorkEffortAndAssoc"  send-mode="all"    &gt;
  &lt;invoke  name="updateWorkEffort"  mode="sync"/&gt;
   &lt;invoke  name="updateWorkEffortAssoc"  mode="sync"/&gt;
 &lt;/group&gt;
 &lt;group  name="createWorkEffortRequestItemAndRequestItem"  send-mode="all"    &gt;
   &lt;invoke  name="checkCustRequestItemExists"  mode="sync"    result-to-context="true"/&gt;
   &lt;invoke  name="createWorkEffortRequestItem"  mode="sync"/&gt;
 &lt;/group&gt;
&lt;/service-group&gt;


 

## 调用服务

调用一个存在的服务是非常简单的，可以通过webtools的Service  Engine  Tools查看具体的服务信息

### 使用java方式调用服务

例：applications/content/src/org/ofbiz/content/search/SearchEvents.java

<code>
public  static  String  indexTree(HttpServletRequest  request,  HttpServletResponse  response)  {
        Map&lt;String,  Object&gt;  result;
        Map&lt;String,  Object&gt;  serviceInMap  =  FastMap.newInstance();
        HttpSession  session  =  request.getSession();
        GenericValue  userLogin  =  (GenericValue)session.getAttribute("userLogin");
        serviceInMap.put("userLogin",  userLogin);
        LocalDispatcher  dispatcher  =  (LocalDispatcher)  request.getAttribute("dispatcher");
        Map&lt;String,  Object&gt;  paramMap  =  UtilHttp.getParameterMap(request);
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
       List&lt;String&gt;  badIndexList  =  UtilGenerics.checkList(result.get("badIndexList"));
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
</code>

 
调用服务

<code>
     try  {
       result  =  dispatcher.runSync("indexTree",  serviceInMap);
          }  catch  (GenericServiceException  e)  {
       String  errorMsg  =  "Error  calling  the  indexTree  service."  +  e.toString();
       Debug.logError(e,  errorMsg,  module);
       request.setAttribute("_ERROR_MESSAGE_",  errorMsg  +  e.toString());
       return  "error";
      }
</code>

 

如上调用了indxTree服务，该服务在applications/content/servicedef/services.xml中被定义

<code>
 &lt;service  name="indexTree"  auth="true"  engine="java"  validate="true"  transaction-timeout="7200"
  location="org.ofbiz.content.search.SearchServices"  invoke="indexTree"&gt;
   &lt;description&gt;Index  content  under  publish  point&lt;/description&gt;
   &lt;attribute  mode="IN"  name="contentId"  optional="false"  type="String"/&gt;
   &lt;attribute  mode="OUT"  name="badIndexList"  optional="true"  type="List"/&gt;
   &lt;attribute  mode="OUT"  name="goodIndexCount"  optional="true"  type="Integer"/&gt;
 &lt;/service&gt;
</code>
 

 

###  smiple-method：使用simple-method方式调用服务

例：applications/accounting/script/org/ofbiz/accounting/payment/PaymentServices.xml

<code>
 &lt;simple-method  method-name="createPaymentAndApplication"  short-description="Create  a  payment  and  a  payment  application  for  the  full  amount"&gt;
   &lt;set-service-fields  service-name="createPayment"  map="parameters"  to-map="createPaymentInMap"/&gt;
   &lt;call-service  service-name="createPayment"  in-map-name="createPaymentInMap"&gt;
 &lt;result-to-field  field="paymentId"  result-name="paymentId"/&gt;
  &lt;/call-service&gt;
   &lt;check-errors/&gt;
   &lt;set-service-fields  service-name="createPaymentApplication"  map="parameters"  to-map="createPaymentAppInMap"/&gt;
   &lt;set  field="createPaymentAppInMap.paymentId"  from-field="paymentId"/&gt;
   &lt;set  field="createPaymentAppInMap.amountApplied"  from-field="parameters.amount"/&gt;
   &lt;call-service  service-name="createPaymentApplication"  in-map-name="createPaymentAppInMap"&gt;
 &lt;result-to-field  field="paymentApplicationId"  result-name="paymentApplicationId"/&gt;
   &lt;/call-service&gt;
   &lt;check-errors/&gt;
   &lt;field-to-result  field="paymentId"  result-name="paymentId"/&gt;
   &lt;field-to-result  field="paymentApplicationId"  result-name="paymentApplicationId"/&gt;
 &lt;/simple-method&gt;
</code>

 

调用服务

<code>
   &lt;call-service  service-name="createPayment"  in-map-name="createPaymentInMap"&gt;
 &lt;result-to-field  field="paymentId"  result-name="paymentId"/&gt;
   &lt;/call-service&gt;
</code>
 

该服务createPayment定义于applications/accounting/script/org/ofbiz/accounting/payment/PaymentServices.xml

<code> 
 &lt;simple-method  method-name="createPayment"  short-description="Create  a  Payment"&gt;
   &lt;if&gt;
 &lt;condition&gt;
  &lt;and&gt;
    &lt;not&gt;&lt;if-has-permission  permission="PAY_INFO"  action="_CREATE"/&gt;&lt;/not&gt;
    &lt;not&gt;&lt;if-compare-field  field="userLogin.partyId"  to-field="parameters.partyIdFrom"  operator="equals"/&gt;&lt;/not&gt;
    &lt;not&gt;&lt;if-compare-field  field="userLogin.partyId"  to-field="parameters.partyIdTo"  operator="equals"/&gt;&lt;/not&gt;
  &lt;/and&gt;
 &lt;/condition&gt;
 &lt;then&gt;
  &lt;add-error&gt;
    &lt;fail-property  resource="AccountingUiLabels"  property="AccountingCreatePaymentPermissionError"/&gt;
  &lt;/add-error&gt;
 &lt;/then&gt;
   &lt;/if&gt;
   &lt;check-errors/&gt;
   &lt;make-value  entity-name="Payment"  value-field="payment"/&gt;
   &lt;if-empty  field="parameters.paymentId"&gt;
 &lt;sequenced-id  sequence-name="Payment"  field="payment.paymentId"/&gt;
&lt;else&gt;
  &lt;set  field="payment.paymentId"  from-field="parameters.paymentId"/&gt;
 &lt;/else&gt;
   &lt;/if-empty&gt;
   &lt;field-to-result  field="payment.paymentId"  result-name="paymentId"/&gt;
   &lt;if-not-empty  field="parameters.paymentMethodId"&gt;
 &lt;entity-one  entity-name="PaymentMethod"  value-field="paymentMethod"&gt;
  &lt;field-map  field-name="paymentMethodId"  from-field="parameters.paymentMethodId"/&gt;
 &lt;/entity-one&gt;
 &lt;if-compare-field  field="parameters.paymentMethodTypeId"  operator="not-equals"  to-field="paymentMethod.paymentMethodTypeId"&gt;
  &lt;log  level="info"  message="Replacing  passed  payment  method  type  [${parameters.paymentMethodTypeId}]  with  payment  method  type  [${paymentMethod.paymentMethodTypeId}]  for  payment  method  [${parameters.paymentMethodId}]"/&gt;
  &lt;set  field="parameters.paymentMethodTypeId"  from-field="paymentMethod.paymentMethodTypeId"/&gt;
 &lt;/if-compare-field&gt;
   &lt;/if-not-empty&gt;
   &lt;if-not-empty  field="parameters.paymentPreferenceId"&gt;
 &lt;entity-one  value-field="orderPaymentPreference"  entity-name="OrderPaymentPreference"&gt;
  &lt;field-map  field-name="orderPaymentPreferenceId"  from-field="parameters.paymentPreferenceId"/&gt;
 &lt;/entity-one&gt;
 &lt;if-empty  field="parameters.paymentMethodId"&gt;
 &lt;set  field="parameters.paymentMethodId"  from-field="orderPaymentPreference.paymentMethodId"/&gt;
 &lt;/if-empty&gt;
 &lt;if-empty  field="parameters.paymentMethodTypeId"&gt;
  &lt;set  field="parameters.paymentMethodTypeId"  from-field="orderPaymentPreference.paymentMethodTypeId"/&gt;
 &lt;/if-empty&gt;
   &lt;/if-not-empty&gt;
   &lt;if-empty  field="parameters.paymentMethodTypeId"&gt;
 &lt;add-error&gt;
  &lt;fail-property  resource="AccountingUiLabels"  property="AccountingPaymentMethodIdPaymentMethodTypeIdNullError"/&gt;
 &lt;/add-error&gt;
   &lt;/if-empty&gt;
   &lt;set-nonpk-fields  map="parameters"  value-field="payment"/&gt;
   &lt;if-empty  field="payment.effectiveDate"&gt;
 &lt;now-timestamp  field="payment.effectiveDate"/&gt;
   &lt;/if-empty&gt;
   &lt;create-value  value-field="payment"/&gt;
 &lt;/simple-method&gt;
</code>

 

### 异步方式调用

<code>
 &lt;simple-method  method-name="sendInvoicePerEmail"  short-description="Send  an  invoice  per  Email"&gt;
   &lt;set-service-fields  service-name="sendMailFromScreen"  map="parameters"  to-map="emailParams"/&gt;
   &lt;set  field="emailParams.xslfoAttachScreenLocation"  value="component://accounting/widget/AccountingPrintScreens.xml#InvoicePDF"/&gt;
   &lt;set  field="emailParams.bodyParameters.invoiceId"  from-field="parameters.invoiceId"/&gt;
  &lt;set  field="emailParams.bodyParameters.userLogin"  from-field="parameters.userLogin"/&gt;
   &lt;set  field="emailParams.bodyParameters.other"  from-field="parameters.other"/&gt;&lt;!--  to  to  print  in  'other  currency'  --&gt;
   &lt;call-service-asynch  service-name="sendMailFromScreen"  in-map-name="emailParams"/&gt;
   &lt;property-to-field  resource="AccountingUiLabels"  property="AccountingEmailScheduledToSend"  field="successMessage"/&gt;
 &lt;/simple-method&gt;
</code>

 

异步调用代码

<code>
    &lt;call-service-asynch  service-name="sendMailFromScreen"  in-map-name="emailParams"/&gt;
</code>
 

 

### 服务中的事务申明

一个服务是否使用事务，是否必须是独立的事务，以及事务的超时时间设置，均可以在服务定义时声明，或者在通过服务引擎调用服务时指定（传入）

#### 服务定义时申明事务

例：applications/order/servicedef/services.xml
<code>
 &lt;service  name="sendOrderConfirmation"  engine="java"  require-new-transaction="true"  max-retry="3"
 location="org.ofbiz.order.order.OrderServices"  invoke="sendOrderConfirmNotification"&gt;
   &lt;description&gt;Send  a  order  confirmation&lt;/description&gt;
   &lt;implements  service="orderNotificationInterface"/&gt;
 &lt;/service&gt;
</code>
 

#### 在服务调用时指定

java代码
<code>
dispatcher.runSync(servicename,  context,transactionTimeOut,requireNewTransaction);
dispatcher.runAsync(servicename,  context,requester,persist,transactionTimeOut,requireNewTransaction);
</code>
 


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
