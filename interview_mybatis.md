<h1> MyBaits </h1>

**Table of Contents**
<!-- TOC -->

- [1. #{}和${}的区别是什么？](#1-和的区别是什么)
- [2. Xml映射文件中，除了常见的select|insert|updae|delete标签之外，还有哪些标签？](#2-xml映射文件中除了常见的selectinsertupdaedelete标签之外还有哪些标签)
- [3. 最佳实践中，通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？](#3-最佳实践中通常一个xml映射文件都会写一个dao接口与之对应请问这个dao接口的工作原理是什么dao接口里的方法参数不同时方法能重载吗)
- [4. Mybatis是如何进行分页的？分页插件的原理是什么？](#4-mybatis是如何进行分页的分页插件的原理是什么)
- [5. Mybatis执行批量插入，能返回数据库主键列表吗？](#5-mybatis执行批量插入能返回数据库主键列表吗)
- [6. Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？](#6-mybatis动态sql是做什么的都有哪些动态sql能简述一下动态sql的执行原理不)
- [7. Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？](#7-mybatis是如何将sql执行结果封装为目标对象并返回的都有哪些映射形式)
- [8. Mybatis能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。](#8-mybatis能执行一对一一对多的关联查询吗都有哪些实现方式以及它们之间的区别)
- [12. Mybatis中如何执行批处理？](#12-mybatis中如何执行批处理)
- [13. Mybatis都有哪些Executor执行器？它们之间的区别是什么？](#13-mybatis都有哪些executor执行器它们之间的区别是什么)
- [16、Mybatis映射文件中，如果A标签通过include引用了B标签的内容，请问，B标签能否定义在A标签的后面，还是说必须定义在A标签的前面？](#16mybatis映射文件中如果a标签通过include引用了b标签的内容请问b标签能否定义在a标签的后面还是说必须定义在a标签的前面)
- [17. 简述Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系？](#17-简述mybatis的xml映射文件和mybatis内部数据结构之间的映射关系)
- [18. 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？](#18-为什么说mybatis是半自动orm映射工具它与全自动的区别在哪里)

<!-- /TOC -->

# 1. #{}和${}的区别是什么？

`${}` 是Properties文件中的变量占位符，它可以用于标签属性值和sql内部，属于静态文本替换，比如`${driver}`会被静态替换为`com.mysql.jdbc.Driver`。`#{}`是sql的参数占位符，Mybatis会将sql中的`#{}`替换为`?`号，在sql执行前会使用PreparedStatement的参数设置方法，按序给sql的?号占位符设置参数值，比如ps.setInt(0, parameterValue)，`#{item.name}`的取值方式为使用反射从参数对象中获取item对象的name属性值，相当于param.getItem().getName()。

# 2. Xml映射文件中，除了常见的select|insert|updae|delete标签之外，还有哪些标签？

还有很多其他的标签，`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态sql的9个标签，trim|where|set|foreach|if|choose|when|otherwise|bind等，其中`<sql>`为sql片段标签，通过`<include>`标签引入sql片段，`<selectKey>`为不支持自增的主键生成策略标签。

# 3. 最佳实践中，通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement，举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面id = findStudentById的MappedStatement。在Mybatis中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个MappedStatement对象。

Dao接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。

# 4. Mybatis是如何进行分页的？分页插件的原理是什么？

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

举例：select * from student，拦截sql后重写为：select t.* from （select * from student）t limit 0，10

# 5. Mybatis执行批量插入，能返回数据库主键列表吗？

能，JDBC都能，Mybatis当然也能。

# 6. Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

Mybatis动态sql可以让我们在Xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能，Mybatis提供了9种动态sql标签trim|where|set|foreach|if|choose|when|otherwise|bind。

其执行原理为，使用OGNL从sql参数对象中计算表达式的值，根据表达式的值动态拼接sql，以此来完成动态sql的功能。

# 7. Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

1. 使用`<resultMap>`标签，逐一定义列名和对象属性名之间的映射关系。
2. 使用sql列的别名功能，将列别名书写为对象属性名，比如T_NAME AS NAME，对象属性名一般是name，小写，但是列名不区分大小写，Mybatis会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成T_NAME AS NaMe，Mybatis一样可以正常工作。
3. 使用驼峰标识

有了列名与属性名的映射关系后，Mybatis通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

# 8. Mybatis能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。

能，Mybatis不仅可以执行一对一、一对多的关联查询，还可以执行多对一，多对多的关联查询，多对一查询，其实就是一对一查询，只需要把selectOne()修改为selectList()即可；多对多查询，其实就是一对多查询，只需要把selectOne()修改为selectList()即可。

关联对象查询，有两种实现方式，一种是单独发送一个sql去查询关联对象，赋给主对象，然后返回主对象。另一种是使用嵌套查询，嵌套查询的含义为使用join查询，一部分列是A对象的属性值，另外一部分列是关联对象B的属性值，好处是只发一个sql查询，就可以把主对象和其关联对象查出来。

那么问题来了，join查询出来100条记录，如何确定主对象是5个，而不是100个？其去重复的原理是`<resultMap>`标签内的`<id>`子标签，指定了唯一确定一条记录的id列，Mybatis根据`<id>`列值来完成100条记录的去重复功能，`<id>`可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的，尽管一般情况下，只有主对象会有重复记录，关联对象一般不会重复。

举例：下面join查询出来6条记录，一、二列是Teacher对象列，第三列为Student对象列，Mybatis去重复处理后，结果为1个老师6个学生，而不是6个老师6个学生。


|       t_id |   t_name     |    s_id |
|:---        |:---          |:---     |
|          1 | teacher      |      38 |
|          1 | teacher      |      39 |
|          1 | teacher      |      40 |
|          1 | teacher      |      41 |
|          1 | teacher      |      42 |
|          1 | teacher      |      43 |

# 9. Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

答：Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。

它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。


# 10. Mybatis中如何执行批处理？

使用BatchExecutor完成批处理。

# 11. Mybatis都有哪些Executor执行器？它们之间的区别是什么？

Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。

BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

# 12. 简述Mybatis的Xml映射文件和Mybatis内部数据结构之间的映射关系？

Mybatis将所有Xml配置信息都封装到All-In-One重量级对象Configuration内部。在Xml映射文件中，`<parameterMap>`标签会被解析为ParameterMap对象，其每个子元素会被解析为ParameterMapping对象。`<resultMap>`标签会被解析为ResultMap对象，其每个子元素会被解析为ResultMapping对象。每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签均会被解析为MappedStatement对象，标签内的sql会被解析为BoundSql对象。

# 13. 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。
