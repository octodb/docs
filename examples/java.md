OctoDB in Java
==============

The OctoDB JDBC driver is based on the [SQLite JDBC](https://github.com/xerial/sqlite-jdbc)

You can download the free version of OctoDB JDBC:

[octodb-jdbc-3.36.0-free.jar](http://octodb.io/download/octodb-jdbc-3.36.0-free.jar)

The full version is available with the commercial license


### Using

Compile and run your java application with the commands:

#### On Linux and Mac

    javac -classpath ".:octodb-jdbc-3.36.0.jar" <your_app.java>
    java -classpath ".:octodb-jdbc-3.36.0.jar" <your_app>

#### On Windows

    javac -classpath ".;octodb-jdbc-3.36.0.jar" <your_app.java>
    java -classpath ".;octodb-jdbc-3.36.0.jar" <your_app>


### Example Code

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import org.json.*;

public class SecondaryNode
{
  public static void main(String[] args)
  {
    Connection connection = null;
    try
    {
      // create a database connection
      String uri = "file:sec.db?node=secondary&connect=tcp://127.0.0.1:1234";
      connection = DriverManager.getConnection("jdbc:sqlite:" + uri);

      // print some text
      System.out.println("secondary database open");

      Statement statement = connection.createStatement();
      // check if the db is ready
      while (true) {
        ResultSet rs = statement.executeQuery("PRAGMA sync_status");
        rs.next();
        JSONObject obj = new JSONObject(rs.getString(1));
        if (obj.getBoolean("db_is_ready")) break;
        Thread.sleep(250);
      }

      // print some text
      System.out.println("secondary database ready");

      // now we can access the db
      start_access(connection);

      // exiting the app
      connection.close();
    }
    catch(SQLException e)
    {
      // if the error message is "out of memory",
      // it probably means no database file is found
      System.err.println(e.getMessage());
    }
    catch (Exception e)
    {
      System.out.println(e);
    }
    finally
    {
      try
      {
        if(connection != null)
          connection.close();
      }
      catch(SQLException e)
      {
        // connection close failed.
        System.err.println(e);
      }
    }
  }
}
```
