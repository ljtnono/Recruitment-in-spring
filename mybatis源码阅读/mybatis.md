## mybatis源代码阅读

参考 Mybatis 3 源代码深度解析-江荣波

### 第三章mybatis常用工具类

#### SQL工具类

SQL工具类是mybatis下jdbc包中一个工具类，用来动态拼接SQL，其上有一个AbstractSQL类，主要功能是在这个类中实现，类图如下：

![SQL类图](assets/SQL%E7%B1%BB%E5%9B%BE.png)

代码示例：

```java
package cn.ljtnono.study.capter03;

import org.apache.ibatis.jdbc.SQL;
import org.junit.Test;

/**
 * 使用SQL类拼接动态SQL语句
 *
 * @author Ling, Jiatong
 * Date: 2021/2/6 8:52
 */
public class SQLExample01 {

    @Test
    public void example01() {
        String dic_vul = new SQL() {
            {
                // 使用多个SELECT语句会自动拼接到一个SELECT语句中
                SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME");
                SELECT("P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON");
                // 使用多个FROM语句会自动拼接到一个FROM语句中去
                FROM("PERSON P");
                FROM("ACCOUNT A");
                INNER_JOIN("DEPARTMENT D on D.ID = P.DEPARTMENT_ID");
                INNER_JOIN("COMPANY C on D.COMPANY_ID = C.ID");
                // where语句被多次调用也会拼接为一个语句，默认使用AND进行拼接
                WHERE("P.ID = A.ID");
                WHERE("P.FIRST_NAME like ?");
                OR();
                WHERE("P.LAST_NAME like ?");
                GROUP_BY("P.ID");
                HAVING("P.LAST_NAME like ?");
                // 多次order by 语句默认使用逗号隔开
                ORDER_BY("P.ID");
                ORDER_BY("P.FULL_NAME");
            }
        }.toString();
        System.out.println(dic_vul);
    }

    @Test
    public void testInsertSql() {
        String insertSql = new SQL() {
            {
                INSERT_INTO("PERSON");
                VALUES("ID, FIRST_NAME", "#{id}, #{firstName}").VALUES("LAST_NAME", "#{lastName}");
            }
        }.toString();
        System.out.println(insertSql);
    }

    @Test
    public void testDeleteSql() {
        String deleteSql = new SQL() {
            {
                DELETE_FROM("PERSON");
                WHERE("ID = #{id}");
            }
        }.toString();
        System.out.println(deleteSql);
    }

    @Test
    public void testUpdateSql() {
        String updateSql = new SQL() {
            {
                UPDATE("PERSON");
                SET("FIRST_NAME = #{firstName}");
                WHERE("ID = #{id}");
            }
        }.toString();
        System.out.println(updateSql);
    }
}

```

SQL类继承至AbstractSQL类，只重写了该类的getSelf()方法：

```java
public class SQL extends AbstractSQL<SQL> {

  @Override
  public SQL getSelf() {
    return this;
  }

}
```

所有的功能由AbstractSQL类完成，AbstractSQL类汇总维护了一个SQLStatement内部类的实例和一系列构造SQL语句的方法。所有的语句基本都是独立存放在一个ArrayList中。最后通过sqlClause方法拼接SQL语句。

```java
private void sqlClause(SafeAppendable builder, String keyword, List<String> parts, String open, String close,
                               String conjunction) {
            if (!parts.isEmpty()) {
                if (!builder.isEmpty()) {
                    builder.append("\n");
                }
                builder.append(keyword);
                builder.append(" ");
                builder.append(open);
                String last = "________";
                for (int i = 0, n = parts.size(); i < n; i++) {
                    String part = parts.get(i);
                    if (i > 0 && !part.equals(AND) && !part.equals(OR) && !last.equals(AND) && !last.equals(OR)) {
                        builder.append(conjunction);
                    }
                    builder.append(part);
                    last = part;
                }
                builder.append(close);
            }
        }
```

#### ScriptRunner工具类

ScriptRunner工具类用于执行SQL脚本，ScriptRunner工具类需要一个Connection对象作为构造函数参数，其中比较有价值的是ScriptRunner类可以通过设置各种属性来控制SQL脚本的执行，其中比较重要的属性罗列如下：

```java
// SQL异常是否中断程序执行
private boolean stopOnError;
// 是否抛出SQLWarning警告（jdk中定义的一个运行时异常）
private boolean throwWarning;
// 是否自动提交
private boolean autoCommit;
// 属性为true时，批量执行文件中的SQL语句，为false时逐条执行SQL语句，默认情况下，SQL语句以分号分割
private boolean sendFullScript;
// 是否去除Windows系统换行符中的\r
private boolean remoteCRs;
// 设置Statement属性是否支持转义处理
private boolean escapeProcessing = true;
// 日志输出位置，默认标准输入输出，即控制台
private boolean PrintWriter logWriter = new PrintWriter(System.out);
// 错误日志输出位置，默认控制台
private boolean PrintWriter errorLogWriter = new PrintWriter(System.error);
// 脚本文件SQL语句中的分隔符，默认为分号
private String delimiter = DEFAULT_DELIMITER;
// 是否支持SQL语句分隔符，单独占一行
private boolean fullLineDelimiter;
```

这些属性都有对应的setter方法进行设置。

在ScriptRunner类中仅提供了一个runScript()方法用于执行SQL脚本文件。



