- if 判断
- choose (when, otherwise) 分支选择
- trim 字符串截取 ( where 封装查询条件 , set 封装修改条件)
- foreach 遍历集合 



# 4.1 `<if>+<where>`

新建接口`EmployeeMapperDynamicSQL`和对应的配置文件。注意将配置文件注册到 mybatis 的配置文件中去。 

```java
public interface EmployeeMapperDynamicSQL {
    public List<Employee> getEmpsByConditionIf(Employee employee);
}
```

需求：**查询员工，携带了哪个字段，查询条件就带上哪个字段的值** 

```xml
<select id="getEmpsByConditionIf" resultType="bean.Employee">
    select * from tb1_employee where
    <!--test:判断表达式（OGNL表达式，参照官方文档）
        从参数中取值进行判断
        遇见特殊符号应该去写转义字符：&->&amp; "->&quot;
        -->
    <if test="id!=null">
        id=#{id}
    </if>
    <if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
        and last_name like #{lastName}
    </if>
    <if test="email!=null and email.trim()!=&quot;&quot;">
        and email=#{email}
    </if>
    <!--ONGL 会进行字符串与数字的转换判断，如："0"==0-->
    <if test="gender==0 or gender==1">
        and gender=#{gender}
    </if>
</select>
```

测试：

```java
public class Mybatis {  
    public void dynamicSql() throws IOException {
        /**获取 SqlSessionFactory的自定义方法 */
        SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
            EmployeeMapperDynamicSQL mapper = sqlSession.getMapper(EmployeeMapperDynamicSQL.class);
            Employee employee = new Employee(3,"%x%","",null);
            List<Employee> emps1 = mapper.getEmpsByConditionIf(employee);
            for (Employee e:emps1){
                System.out.println(e);
            }
        } finally {
            sqlSession.close();
        }
    }
}
```

上面代码会出现的问题：如果某些条件缺失，可能SQL的拼装会出现问题，如果上面没有 id，那么拼装后的SQL会是：`"... where and last_name like ..."`。

解决方法(两种)：

1. 给 where 后面加上 `1=1`，以后的条件都是 `and...`
2. MyBatis 可以使用 `<where> `标签来将所有的查询条件包括在内。MyBatis会将`<where>`标签中拼装的sql多出来的 and 或者 or 去掉
   - 注意：`<where>`标签只会将每个语句最开始的and或or去掉，如果写在后面（ id=#{id} and）则仍会拼装错误。请按下面写。

```xml
<select id="getEmpsByConditionIf" resultType="bean.Employee">
    select * from tb1_employee where
    <where>
        <if test="id!=null">
            id=#{id}
        </if>
        <if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
            and last_name like #{lastName}
        </if>
        <if test="email!=null and email.trim()!=&quot;&quot;">
            and email=#{email}
        </if>
        <if test="gender==0 or gender==1">
            and gender=#{gender}
        </if>
    </where>
</select>
```



# 4.2 `<trim>+<if>`

**使用较少** 

之前的代码中如果将and或or写在语句的后面，使用`<where>`标签会出错，可使用`<trim>`标签自定义字符串截取的规则。 

- `<trim>`标签体中是整个字符串拼接后的结果
  - prefix 属性：给拼接后的整个字符串加一个前缀        
  - prefixOverrides 属性：前缀覆盖，去掉整个字符串前面多余的字符        
  - suffix 属性：给拼接后的整个字符串加一个后缀        
  - suffixOverrides属性： 后缀覆盖，去掉整个字符串后面多余的字符 



需求：**查询员工，携带了哪个字段，查询条件就带上哪个字段的值**

```java
public interface EmployeeMapperDynamicSQL {
	public List<Employee> getEmpsByConditionTrim(Employee employee);
}
```

```xml
<select id="getEmpsByConditionTrim" resultType="bean.Employee">
    select * from tb1_employee
    <trim prefix="where" suffixOverrides="and">
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
            last_name like #{lastName} and
        </if>
        <if test="email!=null and email.trim()!=&quot;&quot;">
            email=#{email} and
        </if>
        <if test="gender==0 or gender==1">
            gender=#{gender}
        </if>
    </trim>
</select>
```

测试：这里仅写了不同的部分

```java
//测试 trim
Employee employee = new Employee(3,"%x%","",null);
List<Employee> emps2 = mapper.getEmpsByConditionTrim(employee);
for (Employee e:emps2){
        System.out.println(e);
}
```



# 4.3 `<choose>`

需求：**如果带了 id 就用 id 查，如果带了 lastName 就用 lastName 查。。。不再是将所有条件拼装，而是*只会进入一个*。** 

接下来使用的方式类似于：带了break的switch-case 

```java
public interface EmployeeMapperDynamicSQL {
    //测试 choose
    public List<Employee> getEmpsByConditionChoose(Employee employee);
}
```

```xml
<select id="getEmpsByConditionChoose" resultType="bean.Employee">
    select * from tb1_employee
    <where>
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="lastName!=null">
                last_name like #{lastName}
            </when>
            <when test="email!=null">
                email=#{email}
            </when>
            <otherwise>
                gender=0
            </otherwise>
        </choose>
    </where>
</select>
```

```java
EmployeeMapperDynamicSQL mapper = sqlSession.getMapper(EmployeeMapperDynamicSQL.class);
Employee employee = new Employee(1,"%x%","",null);
        //测试choose。虽然有两个选择条件但只有 id 作为条件进行了查询 
List<Employee> empsByConditionChoose = mapper.getEmpsByConditionChoose(employee);
for (Employee e:empsByConditionChoose)
        System.out.println(e);
```



# 4.4 `<set>+<if>`动态更新

需求：根据传入的参数所携带的信息来进行更新 

```java
public interface EmployeeMapperDynamicSQL {
    //测试 set
    public void updateEmp(Employee employee);
}
```

```xml
<update id="updateEmp">
    update tb1_employee
    <set>
        <if test="lastName!=null">
            last_name=#{lastName},
        </if>
        <if test="email!=null">
            email=#{email},
        </if>
        <if test="gender==0 or gender==1">
            gender=#{gender}
        </if>
    </set>
    where id=#{id}
</update>
```

如果没有使用`<set>`标签，由于每一个参数设置后面的逗号，当只设置部分参数时会出现拼装错误。也可以使用`<trim>`标签来实现: 

```xml
<update id="updateEmp">
    update tb1_employee
    <trim prefix="set" suffixOverrides=",">
        <if test="lastName!=null">
            last_name=#{lastName},
        </if>
        <if test="email!=null">
            email=#{email},
        </if>
        <if test="gender==0 or gender==1">
            gender=#{gender}
        </if>
    </trim>
    where id=#{id}
</update>
```

```java
EmployeeMapperDynamicSQL mapper = sqlSession.getMapper(EmployeeMapperDynamicSQL.class);
Employee employee = new Employee(1,"bx","bx@126.com",null);
//测试 set
mapper.updateEmp(employee);
sqlSession.commit(); //提交
```



# 4.5 `<foreach>`

`<foreach>`标签：

- collection属性: 指定要遍历的集合：list类型的参数会特殊处理封装在map中，map的key就叫list
- item属性: 将当前遍历出的元素赋值给指定的变量。使用 #{变量名} 就能取出变量的值也就是当前遍历出的元素
- separator属性: 每个元素之间的分隔符。每两组标签体中的数据之间加分隔符
- open属性: 遍历出所有结果拼接一个开始的字符
- close属性: 遍历出所有结果拼接一个结束的字符
- index属性：索引。
  - 遍历list的时候index是list的索引，item就是当前的值
  - 遍历map的时候index是mao的key，item就是map的值 



- 遍历查询

  - 需求：查询给定的多个id所对应的员工信息`select * from tb1_employee where id in (1,2,3,4)`

  ```java
  public interface EmployeeMapperDynamicSQL {
      //测试 foreach
      public List<Employee> getEmpsByConditionForeach(@Param("ids") List<Integer> ids);
  }
  ```

  ```xml
  <select id="getEmpsByConditionForeach" resultType="bean.Employee">
      select * from tb1_employee
      <foreach collection="ids" item="item_id" separator="," open="where id in(" close=")">
          #{item_id}
      </foreach>
  </select>
  ```

  说明：如果没有 open 和 close 属性的话，写成以下形式： 

  ```xml
  <select id="getEmpsByConditionForeach" resultType="bean.Employee">
      select * from tb1_employee where id in(
      <foreach collection="ids" item="item_id" separator="," >
          #{item_id}
      </foreach>
      ）
  </select>
  ```

  ```java
  //测试 foreach
  List<Employee> emps = mapper.getEmpsByConditionForeach( Arrays.asList(1, 2, 3, 4));
  for (Employee e:emps)
      System.out.println(e);
  ```



- 批量保存

  ```java
  public interface EmployeeMapperDynamicSQL {
      //测试 foreach 批量保存
      public void addEmps(@Param("emps") List<Employee> emps);
  }
  ```

  方式一：使用 mysql支持的 values(),(),() 语法。 更推荐。但是Oracle是不支持这种语法的 

  ```xml
  <!--foreach批量保存-->
  <!--MySQL下批量保存，可以foreach遍历，mysql支持 values(),(),() 语法-->
  <insert id="addEmps">
      insert into tb1_employee(last_name,email,gender,d_id) values
      <foreach collection="emps" item="emp" separator=",">  注意这里的级联属性的逐层获取
          ( #{emp.lastName} , #{emp.email} , #{emp.gender} , #{emp.dept.id} )
      </foreach>
  </insert>
  ```

  方式二：多次发出插入请求。这种方式需要数据库连接属性 `allowMultiQueries` 的支持：

  - 这种分号分隔多个sql可以用于其他的批量操作（删除、修改） 
  - 注意：由于MySQL的语法中默认是不支持在一条语句使用`；`来分隔多条查询的，所以需要先在db.properties 中设置`allowMultiQueries`属性： 

  ```properties
  url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useSSL=false&allowMultiQueries=true
  ```

  ```xml
  <insert id="addEmps">
      <foreach collection="emps" item="emp" separator=";">
          insert into tb1_employee(last_name,email,gender,d_id)
          values (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
      </foreach>
  </insert>
  ```

  ```java
  /**测试 foreach 批量保存*/
  public void batchSave() throws IOException {
      SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
      SqlSession sqlSession = sqlSessionFactory.openSession();
      try{
          EmployeeMapperDynamicSQL mapper = sqlSession.getMapper(EmployeeMapperDynamicSQL.class);
          List<Employee> emps = new ArrayList<>();
          emps.add(new Employee(null,"tom","tom@qq.cpm","0",new Department(1)));
          emps.add(new Employee(null,"rose","rose@qq.cpm","1",new Department(2)));
          emps.add(new Employee(null,"caimi","caimi@qq.cpm","1",new Department(1)));
          mapper.addEmps(emps);
          //提交
          sqlSession.commit();
      } finally {
          sqlSession.close();
      }
  }
  ```

# 4.6 内置参数

不只是方法传递过来的参数可以被用来判断、取值。MyBatis默认还有两个内置参数：

1. `_parameter`：代表整个参数：
   - 单个参数：_parameter 就是这个参数
   - 多个参数：参数会被封装为一个 map，_parameter 就是代表这个 map
2. `_databaseId`：如果配置了全局配置的` <DatabaseIdProvider> `标签，则代表当前数据库的别名。

```java
public interface EmployeeMapperDynamicSQL {
    //测试两个内置参数
    public List<Employee> getEmpsTestInnerParameters(Employee employee);
}
```

```xml
<select id="getEmpsTestInnerParameters" resultType="bean.Employee">
    <if test="_databaseId=='mysql'">
        select * from tb1_employee
        <if test="_parameter!=null">
            where last_name=#{_parameter.lastName}
        </if>
    </if>
    <if test="_databaseId=='oracle'">
        select * from employees
    </if>
</select>
```



# 4.7 `<bind>`动态绑定

可以将 OGNL 表达式的值绑定到一个变量中，方便后来引用这个变量的值。不推荐使用下面的<bind>中将传过来的参数左右两侧添加上`%`，使得变成了模糊查询。

```xml
<select id="getEmpsTestInnerParameters" resultType="bean.Employee">
    <bind name="_lastName" value="'%'+lastName+'%'"/>
    <if test="_databaseId=='mysql'">
        select * from tb1_employee
        <if test="_parameter!=null">
            where last_name like #{_lastName}    //这里调用的是<bind>标签中定义的变量
            </if>
        </if>
        <if test="_databaseId=='oracle'">
            select * from employees
            <if test="_parameter!=null">
                where last_name like #{_parameter.lastName}
            </if>
        </if>
</select>
```



# 4.8 `<sql>+<include>`

`<sql> `标签抽取可重用的sql片段，方便后面使用 `<include>`标签引用。 

1. 可以将经常要查询的列名或插入用的列名抽取出来方便引用；
2. `<include>`还可以自定义一些`<property>`，`<sql>`  标签内部就能使用自定义的属性：只能用`${ }` ，不能用`#{ }`。



- 使用`<sql>`标签抽取可重用的 sql 片段： 

  ```xml
  <sql id="insertColumn">
      <if test="_databaseId='oracle'">
          employee_id,last_name,email,${testColumn}
      </if>
      <if test="_databaseId='mysql'">
          last_name,email,gender,d_id
      </if>
  </sql>
  ```

- 使用`<include>` 标签引用外部定义的 sql： 

  ```xml
  <insert id="addEmps">
      insert into tb1_employee( 
      <include refid="insertColumn">
          <property name="testColumn" value="abc"></property>
      </include>
      )
      values
      <foreach collection="emps" item="emp" separator=",">
          (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
      </foreach>
  </insert>
  ```

