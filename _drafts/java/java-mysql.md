private void empsInDept(Connection myConnect, int deptId) throws SQLException {

  CallableStatement cStmt = myConnect.prepareCall("{CALL sp_emps_in_dept(?)}");
  cStmt.setInt(1, deptId);
  cstm.registerOutParameter(3, Types.INTEGER); // 设置返回值类型 
  cStmt.execute();
  ResultSet rs1 = cStmt.getResultSet();
	
  while (rs1.next()) {
       System.out.println(rs1.getString("department_name") + " "
         + rs1.getString("location"));
  }
  rs1.close();

  /* process second result set */
  if (cStmt.getMoreResults()) {
       ResultSet rs2 = cStmt.getResultSet();
       while (rs2.next()) {
        System.out.println(rs2.getInt(1) + " " + rs2.getString(2) + " "
              + rs2.getString(3));
       }
       rs2.close();
  }
  cStmt.close();
}



package cn.outofmemory.test;

import java.sql.Connection;
import java.sql.DriverManager;

public class Mysql {
    public static void main(String arg[]) {
        try {
            Connection con = null; //定义一个MYSQL链接对象
            Class.forName("com.mysql.jdbc.Driver").newInstance(); //MYSQL驱动
            con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/test", "root", "root"); //链接本地MYSQL
            System.out.print("yes");
        } catch (Exception e) {
            System.out.print("MYSQL ERROR:" + e.getMessage());
        }

    }
}


Class.forName("com.mysql.jdbc.Driver").newInstance(); 我们链接的是MYSQL数据库，所以需要一个MYSQL的数据库驱动，如果你的环境中没有安装， 可以下载：mysql-connector-java-5.1.17-bin.jar JAR包，然后放进jdk1.6.0_37\jre\lib\ext 重启eclispe 就可以在JRE系统库中看到。

con = DriverManager.getConnection;("jdbc:mysql://127.0.0.1:3306/test", "root", "root"); 是链接数据库的语句， 返回Connection con;对象。参数格式：("jdbc:mysql://ip:端口/数据库名称", 用户名,密码)


##写入一条数据 

package main;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class Mysql {

    /**
     * 入口函数
     * @param arg
     */
    public static void main(String arg[]) {
        try {
            Connection con = null; //定义一个MYSQL链接对象
            Class.forName("com.mysql.jdbc.Driver").newInstance(); //MYSQL驱动
            con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/test", "root", "root"); //链接本地MYSQL

            Statement stmt; //创建声明
            stmt = con.createStatement();

            //新增一条数据
            stmt.executeUpdate("INSERT INTO user (username, password) VALUES ('init', '123456')");
            ResultSet res = stmt.executeQuery("select LAST_INSERT_ID()");
            int ret_id;
            if (res.next()) {
                ret_id = res.getInt(1);
                System.out.print(ret_id);
            }

        } catch (Exception e) {
            System.out.print("MYSQL ERROR:" + e.getMessage());
        }

    }
}

stmt.executeUpdate INSERT; DELETE; UPDATE;语句都用executeUpdate函数来操作 stmt.executeQuery SELECT;语句都用stmt.executeQuery函数来操作 ResultSet res = stmt.executeQuery;("select LAST;_INSERT_ID()"); 查询最后插入数据的ID号，返回ResultSet res;对象

##删除和更新数据

package main;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class Mysql {

    /**
     * 入口函数
     * @param arg
     */
    public static void main(String arg[]) {
        try {
            Connection con = null; //定义一个MYSQL链接对象
            Class.forName("com.mysql.jdbc.Driver").newInstance(); //MYSQL驱动
            con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/test", "root", "root"); //链接本地MYSQL

            Statement stmt; //创建声明
            stmt = con.createStatement();

            //新增一条数据
            stmt.executeUpdate("INSERT INTO user (username, password) VALUES ('init', '123456')");
            ResultSet res = stmt.executeQuery("select LAST_INSERT_ID()");
            int ret_id;
            if (res.next()) {
                ret_id = res.getInt(1);
                System.out.print(ret_id);
            }

            //删除一条数据
            String sql = "DELETE FROM user WHERE id = 1";
            long deleteRes = stmt.executeUpdate(sql); //如果为0则没有进行删除操作，如果大于0，则记录删除的条数
            System.out.print("DELETE:" + deleteRes);

            //更新一条数据
            String updateSql = "UPDATE user SET username = 'xxxx' WHERE id = 2";
            long updateRes = stmt.executeUpdate(updateSql);
            System.out.print("UPDATE:" + updateRes);

        } catch (Exception e) {
            System.out.print("MYSQL ERROR:" + e.getMessage());
        }

    }
}

删除和更新数据都使用stmt.executeUpdate函数。 删除和更新数据都会返回一个Long的结果，如果为0，则删除或者更新失败，如果大于0则是操作删除的记录数

##查询语句

package main;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.Statement;

public class Mysql {

    /**
     * 入口函数
     * @param arg
     */
    public static void main(String arg[]) {
        try {
            Connection con = null; //定义一个MYSQL链接对象
            Class.forName("com.mysql.jdbc.Driver").newInstance(); //MYSQL驱动
            con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/test", "root", "root"); //链接本地MYSQL

            Statement stmt; //创建声明
            stmt = con.createStatement();

            //新增一条数据
            stmt.executeUpdate("INSERT INTO user (username, password) VALUES ('init', '123456')");
            ResultSet res = stmt.executeQuery("select LAST_INSERT_ID()");
            int ret_id;
            if (res.next()) {
                ret_id = res.getInt(1);
                System.out.print(ret_id);
            }

            //删除一条数据
            String sql = "DELETE FROM user WHERE id = 1";
            long deleteRes = stmt.executeUpdate(sql); //如果为0则没有进行删除操作，如果大于0，则记录删除的条数
            System.out.print("DELETE:" + deleteRes);

            //更新一条数据
            String updateSql = "UPDATE user SET username = 'xxxx' WHERE id = 2";
            long updateRes = stmt.executeUpdate(updateSql);
            System.out.print("UPDATE:" + updateRes);

            //查询数据并输出
            String selectSql = "SELECT * FROM user";
            ResultSet selectRes = stmt.executeQuery(selectSql);
            while (selectRes.next()) { //循环输出结果集
                String username = selectRes.getString("username");
                String password = selectRes.getString("password");
                System.out.print("\r\n\r\n");
                System.out.print("username:" + username + "password:" + password);
            }

        } catch (Exception e) {
            System.out.print("MYSQL ERROR:" + e.getMessage());
        }

    }
}

查询语句使用stmt.executeQuery函数

    rs.absolute() //绝对位置，负数表示从后面数
    rs.first()第一条
    rs.last()最后一条
    rs.previoust()前一条
    rs.next()后一条
    rs.beforeFirst()第一条之前
    rs.afterLast()最后之后
    rs.isFirst(),rs.isLast(),rs.isBeforeFirst(),rs.isAfterLast


======================================

PreparedStatement pstmt = null; //4.建立数据库操作对象
　　String sql = null;//数据库操作语句
　　sql = "INSERT INTO 表名 (属性,属性,属性) VALUES (?,?,?)" ;//插入操作,可以是别的操作语句
　　pstmt = conn.prepareStatement(sql);
　　pstmt.setString(1,值) ;
　　pstmt.setString(2,值) ;
　　pstmt.setString(3,值) ;
　　pstmt.executeUpdate() ;



ResultSet rs = null;
　　sql = "select 属性,属性,属性 from 表名";
　　pstmt = conn.prepareStatement(sql);
　　rs = pstmt.executeQuery();
　　rs.getString("属性");具体方法看属性的类型,返回查询结果





package com.jdbc;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;
/**
 * JDBC 的工具类
 * 其中包含: 获取数据库连接, 关闭数据库资源等方法.
 */
public class JDBCTolls {
    public static void release(Statement sta,Connection conn){
        if(sta!=null)
            try {
                sta.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }finally{
                if(conn !=null)
                    try {
                        conn.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
            }
    }
    public static Connection getConnection(){
        //声明用到的变量 url 驱动类型 用户名和密码
                String driverClass =null;
                String url =null;
                String user = null;
                String password =null;
                Connection conn = null;
                try{
                    //通过读入文件得到信息
                    @SuppressWarnings("static-access")
                    InputStream in=    JDBCTolls.class.getClassLoader().getSystemResourceAsStream("jdbc.properties");
                    Properties properties = new Properties();
                    properties.load(in);
                    // 1. 准备获取连接的 4 个字符串: user, password, jdbcUrl, driverClass
                    driverClass=properties.getProperty("driver");
                    url = properties.getProperty("url");
                    user = properties.getProperty("user");
                    password = properties.getProperty("password");
                    // 2. 加载驱动: Class.forName(driverClass)
                    Class.forName(driverClass);
                    // 3. 调用
                    // DriverManager.getConnection(jdbcUrl, user, password)
                    // 获取数据库连接
                    conn = DriverManager.getConnection(url, user, password);
                    System.out.println(conn);
                }catch(ClassNotFoundException e){
                    e.printStackTrace();
                }catch(Exception e){
                    e.printStackTrace();
                }
        return conn;
    }
}
以下是在其他类中，我们可以写一个公共方法，用来修改数据库 ,可以执行insert，update，delete操作
    /**
     * 通用的方法，用来修改数据库 ,可以执行insert，update，delete操作，注意异常处理，最后要关闭操作释放资源
     * */
    @Test
    public void update(){
        Connection conn = null;
        Statement sta = null;
        String sql=" insert into article(title,ptime,content) values('ccca','111','ccac')";
        try {
             conn = JDBCTolls.getConnection();
             sta = conn.createStatement();
            sta.execute(sql);
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            JDBCTolls.release(sta, conn);
        }
    }

