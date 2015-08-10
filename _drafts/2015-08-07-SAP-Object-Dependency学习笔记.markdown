---
title:  "SAP Object Dependency 学习笔记"
date:   2015-08-07 11:51:00
description: SAP Object Dependency 笔记
---

## SAP Object Dependency General Information

### Object Dependency Type

1. Precondition Type
2. Selection conditions
3. Actions (obsolete)
4. Procedures
5. Constraints


### Global and Local Dependencies
The differences between global and local dependencies are as follows:

* Global dependencies are created centrally and can be assigned to several objects.
* Local dependencies are created for one object and can only be used with this object.

### Integration
You can use dependencies in the following components:

* Component CA – Classification System
* Component LO – Variant Configuration


## 1. Preconditions
用于隐藏不需要显示的特征与特征值，保证对象配置的一致性。在此种类型依赖的定义中，需要定义在何种条件下，指定的特征或特征值会被隐藏。当条件满足或没有违反时，触发此依赖。例如当指定的特征值被选择，或指定的特征没有分配值。

### Precondition for a Characteristic Value
现在有一个可配置物料自行车，并由两个特征MODEL与GEARS以及以下的特征值组成。


|特征|特征值|条件|
|:----|:-------|:-----|
|MODEL|Racing|
||Standard|
||Mountain|
||Tandem|
|GEARS|10|
||12|
||17|
||21|MODEL="Racing"|

只有当MODEL等于Racing时GEARS才能有选项21。

创建一个precondition，并填入源代码：MODEL eq ‘Racing’

系统检查特征值Racing是否分配给特征MODEL：

* 当MODEL设置的值不为Racing时，则特征值21不会显示；
* 当MODEL设置的值Racing时，特征值21将会显示；
* 当MODEL没有设置值时，特征值21仍然会显示，因为并没有违反设置的precondition。


若想当MODEL没有设置值时，特征值21应该隐藏。则可以按照如下设置precondition：
MODEL eq ‘Racing’ and Specified MODEL

### Precondition for a Characteristic
同样以BIKE为例，特征如下:

|特征|特征值|条件|
|:----|:-------|:-----|
|MODEL|Racing|
||Standard|
||Mountain|
||Tandem|
|TANDEM_SADDLE| |MODEL="Racing"|

当MODEL选择Tandem时，才会实现特征TANDEM_SADDLE。

按照如下代码创建precondition，并分配给特征TANDEM_DADDLE:
MODEL eq ‘Tandem’

系统检查特征MODEL是否设置值Tandem：

* 当已设置特征值Tandem，则显示特征TANDEM_SADDLE；
* 当MODEL已设置为另外的值，则特征TANDEM_SADDLE不会显示；
* 当MODEL没有设置值，则特征TANDEM_SADDLE仍然会显示，因为当前的选择未违反定义的precondition。

若想当MODEL没有选择值时，隐藏特征TANDEM_SADDLE，则可以设置precondition：
MODEL eq ‘Tandem’  and Specified MODEL


## 2. Selection conditions

用于确认与变式相关的对象被选中：

* 确定变式配置中的组件或操作
* 确定特征是否为必输

允许分配的对象类型：

* Characteristics
* BOM items
* Operations in task lists
* Sub-operations
* Sequences of operations
* Production resources/tools (PRTs)

当select condition明确符合条件时会生效。当出现以下情况时，select condition不会生效：

* 特征设置了不同于select condition定义的特征值
* 特征没有设置值

由于selection condition 比较简单，这里就不再举例详细描述，主要用途为：

* Selection Conditions for a BOM Item and Operation
* Selection Condition for a Characteristic

## 3. Procedures

procedure用于推断特征值，在推断特征值的功能上来看，与actions非常相像。但是，两者又有一些很重要的不同点：（Actions是旧的依赖类型，现在完全可以用Procedure代替）

|Procedure|Action|
|:-----------|:-------|
|Procedure能够覆盖被其他Procedure设置的默认值|Action不能够覆盖被其他Action设置的值
|Procedure可以设置特征的默认值，而此默认值可以被用户设置的值覆盖|用户不能覆盖Action设置的默认值
|用户可以定义同一个对象的对个Procedure的执行顺序|不能修改Action的执行顺序

可以为以下对象设置Procedure：

* 特征；
* 特征值；
* 可配置物料的配置文件(configuration profile of the configurable object)；
* 物料清单项目(BOM Items)，例如可以修改项目数量；
* 工作清单中的操作(Operation in task list)，例如可以修改标准值。

提示：可以将Procedure分配给配置配件，这样可在同一个地方管理Procedure。


### 如何使用Procedure

* 如果要推断特征值，需要在特征前加上变量 $SELF；
* 可以覆盖其他Procedure设置的特征值；
* Procedure都可以用于定价；
* 设置默认值；
{% highlight ruby %}
$SET_DEFAULT ($SELF, <characteristic>, <term>)；
#=> 默认值设置后，即使引起默认值的条件清除，此默认值仍然存在；
{% endhighlight %}


* 删除默认值；
{% highlight ruby %}
 $DEL_DEFAULT ($SELF, <characteristic>, <term>)；
#=> 删除默认之后，并不会自动删除根据默认值推导的值；
{% endhighlight %}

* 在多层配置结果中，计算某一特征的值的总和；
{% highlight ruby %}
 $SUM_PARTS ($SELF, <characteristic>)；
{% endhighlight %}

* Add the components of a BOM together
{% highlight ruby %}
$COUNT_PARTS (<$SELF>)；
{% endhighlight %}

* 对定价变式条件增加附加费；
{% highlight ruby %}
$SET_PRICING_FACTOR ($SELF, <characteristic>, <variant key>, <factor>)；
{% endhighlight %}

* 可以使用以下关键字（不能在Action中使用）；
{% highlight ruby %}
NOT SPECIFIED
NOT TYPE_OF
<multiple-value characteristic>NE<value>
{% endhighlight %}

### Procedure的执行顺序

每次设置特征值并确认后，系统将执行所有依赖定义。依赖按照如下顺序执行：

1. 执行多次Actions，直到没有特征值能够被推断；
2. 按照如下顺序执行一次Procedure：
	* 在配置文件中定义的顺序；
	* 为特征定义的Procedure；
	* 为特征值定义的Procedure；
3. Actions
4. 被Procedure设置的特征值会触发Action，因此所有的Actions将再次执行；
5. Preconditions
6. Selection condition for Characteristic

当所有的对象在OBJECT区域存在且所有在CONDITION定义的条件满足，Constraints会并行的在1-2点中执行。

## 4. Constraints

用于监控配置是否一致，主要功能如下：

* 描述完全不同的对象以及它们的特征之间的依赖关系；
* 保存配置满足一致性的条件信息；
* 不会单独分配给某些对象，而是将限制组合在依赖网中然后分配给配置文件；
* 在限制定义中，只需通过对象的一般信息来引用，不需加入$SELF、$ROOT与$PARENT等前缀来引用，而是通过输入类名称来引用其中的对象；
* 限制是声明式的依赖，其执行顺序以及触发点并不相关；
* 限制不会以固定的顺序执行，用户不能决定何时一个限制会被使用。

### 限制的结构(Structure of Constraints)

一共有四部分组成，每部分由一个关键词开始。

**OBJECTS**:

输入与此限制有关的对象。

**CONDITION**:

只有当这里输入的条件得到满足时，此限制才会被启用。此关键词可以省略，但是如果写入此关键词，则必须填入条件信息。

**RESTRICTIONS**:

这里输入使配置一致的对象与特征之间的关系，亦可对特征进行赋值，此部分为必输。

**INFERENCES**:

输入需要推导的特征值的特征，考虑到性能问题，建议仅输入确实需要推导的特征。

## 5. ACTIONS（官方已不建议使用，使用Procedure代替）









