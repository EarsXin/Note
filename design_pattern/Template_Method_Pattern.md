# 03 - 模板方法模式

模板模式通常又叫模板方法模式(Template Method Pattern)是指定义一个算法的骨 架，并允许子类为一个或者多个步骤提供实现

模板方法使得子类可以在不改变算法结 构的情况下，重新定义算法的某些步骤，属于行为型设计模式。

## 应用场景

1. 一次性实现一个算法的不变的部分，并将可变的行为留给子类来实现。
2. 各子类中公共的行为被提取出来并集中到一个公共的父类中，从而避免代码重复。

## 实际应用场景（JDBC 操作）

创建一个模板类 JdbcTemplate，封装所有的 JDBC 操作。以查询为例，每次查询的表不 同，返回的数据结构也就不一样。我们针对不同的数据，都要封装成不同的实体对象。 而每个实体封装的逻辑都是不一样的，但封装前和封装后的处理流程是不变的，因此， 我们可以使用模板方法模式来设计这样的业务场景

```java
public interface RowMapper<T> {
    T mapRow(ResultSet rs,int rowNum) throws Exception;
}
```

再创建封装了所有处理流程的抽象类 JdbcTemplate

```java
public abstract class JdbcTemplate {
    private DataSource dataSource;

    public JdbcTemplate(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public List<?> executeQuery(String sql, RowMapper<?> rowMapper, Object[] values){
        try {
            //1、获取连接
            Connection conn = this.getConnection();
            //2、创建语句集
            PreparedStatement pstm = this.createPrepareStatement(conn,sql);
            //3、执行语句集
            ResultSet rs = this.executeQuery(pstm,values);
            //4、处理结果集
            List<?> result = this.paresResultSet(rs,rowMapper);
            //5、关闭结果集
            this.closeResultSet(rs);
            //6、关闭语句集
            this.closeStatement(pstm);
            //7、关闭连接
            this.closeConnection(conn);
            return result;
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }

    protected void closeConnection(Connection conn) throws Exception {
        //数据库连接池，我们不是关闭
        conn.close();
    }

    protected void closeStatement(PreparedStatement pstm) throws Exception {
        pstm.close();
    }

    protected void closeResultSet(ResultSet rs) throws Exception {
        rs.close();
    }

    protected List<?> paresResultSet(ResultSet rs, RowMapper<?> rowMapper) throws Exception {
        List<Object> result = new ArrayList<Object>();
        int rowNum = 1;
        while (rs.next()){
            result.add(rowMapper.mapRow(rs,rowNum ++));
        }
        return result;
    }

    protected ResultSet executeQuery(PreparedStatement pstm, Object[] values) throws Exception {
        for (int i = 0; i < values.length; i++) {
            pstm.setObject(i,values[i]);
        }
        return pstm.executeQuery();
    }

    protected PreparedStatement createPrepareStatement(Connection conn, String sql) throws Exception {
        return conn.prepareStatement(sql);
    }

    public Connection getConnection() throws Exception {
        return this.dataSource.getConnection();
    }
}
```

实体类 Member

```java
public class Member {

    private String username;
    private String password;
    private String nickname;
    private int age;
    private String addr;

    // getter/setter ...
}
```

创建数据库操作类 MemberDao

```java
public class MemberDao extends JdbcTemplate {
    public MemberDao(DataSource dataSource) {
        super(dataSource);
    }

    public List<?> selectAll(){
        String sql = "select * from t_member";
        return super.executeQuery(sql, new RowMapper<Member>() {
            public Member mapRow(ResultSet rs, int rowNum) throws Exception {
                Member member = new Member();
                //字段过多，原型模式
                member.setUsername(rs.getString("username"));
                member.setPassword(rs.getString("password"));
                member.setAge(rs.getInt("age"));
                member.setAddr(rs.getString("addr"));
                return member;
            }
        },null);
    }
}
```

测试类

```java
public class MemberDaoTest {
    public static void main(String[] args) {
        MemberDao memberDao = new MemberDao(null);
        List<?> result = memberDao.selectAll();
        System.out.println(result);
    }
}
```


## 优缺点
优点: 
1. 利用模板方法将相同处理逻辑的代码放到抽象父类中，可以提高代码的复用性。 
2. 将不同的代码不同的子类中，通过对子类的扩展增加新的行为，提高代码的扩展性。 
3. 把不变的行为写在父类上，去除子类的重复代码，提供了一个很好的代码复用平台， 符合开闭原则。

缺点: 
1. 类数目的增加，每一个抽象类都需要一个子类来实现，这样导致类的个数增加。 
2. 类数量的增加，间接地增加了系统实现的复杂度。 
3. 继承关系自身缺点，如果父类添加新的抽象方法，所有子类都要改一遍。


