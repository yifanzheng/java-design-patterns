## 外观模式

外观模式(Facade Design Pattern)，在 GoF 的《设计模式》一书中是这样定义的：为子系统中的一组接口提供统一的接口。此模式定义了一个高层接口，这个接口使子系统更易于使用。

简而言之，外观模式提供了到复杂子系统的简化接口。这里的“子系统”，可以是一个完整的系统，也可以是更细粒度的类或者模块。

### 外观模式解析

下面我们我们通过一个例子来解释外观模式：假设我们有一个带有一组接口的应用程序，以使用 MySQL / SQLServer 数据库并生成不同类型的报告，比如 HTML 报告，PDF 报告等。

因此，我们将使用不同的接口集来处理不同类型的数据库。 然后，客户端应用程序可以使用这些接口来获取所需的数据库连接并生成报告。

具体代码实现如下：

首先，创建两个帮助程序接口，分别是 MySQLHelper 和 SQLServerHelper。
```java
/**
 * MySQLHelper
 */
public class MySQLHelper {
	
	public static Connection getMySQLDBConnection(){
		// 获取 MySQL 数据库连接对象
		return null;
	}
	
	public void generateMySQLPDFReport(String tableName, Connection con){
		// 从数据库中获取数据并生成 PDF 报告
	}
	
	public void generateMySqlHTMLReport(String tableName, Connection con){
		// 从数据库中获取数据并生成 HTML 报告
	}
}

/**
 * SQLServerHelper
 */
public class SQLServerHelper {

	public static Connection getSQLServerDBConnection(){
		// 获取 SQLServer 数据库连接对象
		return null;
	}
	
	public void generateSQLServerPDFReport(String tableName, Connection con){
		// 从数据库中获取数据并生成 PDF 报告
	}
	
	public void generateSQLServerHTMLReport(String tableName, Connection con){
		// 从数据库中获取数据并生成 HTML 报告
	}
	
}
```

然后，创建 Facade 类，为 MySQLHelper 和 SQLServerHelper 提供统一的界面。

```java
/**
 * HelperFacade
 */
public class HelperFacade {

	public static void generateReport(DBTypes dbType, ReportTypes reportType, String tableName){
		Connection con = null;
		switch (dbType){
		case MYSQL: 
		  // 将两个接口整合到一个接口中
			con = MySQLHelper.getMySQLDBConnection();
			MySQLHelper mySqlHelper = new MySQLHelper();
			switch(reportType){
			case HTML:
				mySqlHelper.generateMySQLHTMLReport(tableName, con);
				break;
			case PDF:
				mySqlHelper.generateMySQLPDFReport(tableName, con);
				break;
			}
			break;
		case ORACLE: 
			con = SQLServerHelper.getOracleDBConnection();
			SQLServerHelper sqlServerHelper = new SQLServerHelper();
			switch(reportType){
			case HTML:
				sqlServerHelper.generateSQLServerHTMLReport(tableName, con);
				break;
			case PDF:
				sqlServerHelper.generateSQLServerPDFReport(tableName, con);
				break;
			}
			break;
		}
		
	}
	
	public static enum DBTypes{
    MYSQL, SQLSERVER;
	}
	
	public static enum ReportTypes{
		HTML, PDF;
	}
} 
```

最后，创建客户端程序，分别使用 Facade 模式和不使用 Facade 模式生成报告。

```java
public class Client {

	public static void main(String[] args) {
		String tableName = "Student";
		
		// 不使用 Facade 模式生成报告
		Connection con = MySqlHelper.getMySqlDBConnection();
		MySqlHelper mySqlHelper = new MySqlHelper();
		mySqlHelper.generateMySqlHTMLReport(tableName, con);
		
		Connection con1 = OracleHelper.getOracleDBConnection();
		OracleHelper oracleHelper = new OracleHelper();
		oracleHelper.generateOraclePDFReport(tableName, con1);
		
		// 使用 Facade 模式生成报告
		HelperFacade.generateReport(HelperFacade.DBTypes.MYSQL, HelperFacade.ReportTypes.HTML, tableName);
		HelperFacade.generateReport(HelperFacade.DBTypes.ORACLE, HelperFacade.ReportTypes.PDF, tableName);
	}
}
```
从代码中我们可以知道，在不使用外观模式的情况下，客户端使用时先要获取数据库连接（DB Connection），然后使用连接生成报告。然而，在使用外观模式下，客户端就避免了这些过多的逻辑调用，我们只需要调用一个方法就可以生成报告了，提高了系统的易用性。

### 外观模式应用场景举例

外观模式的应用场景很多，除了上面提到的”让子系统更加易用“外，实际上，它除了解决易用性问题，还能解决其他很多问题。这里列举两个个常用的场景，大家可以进行参考。

**1.解决易用性问题**

外观模式可以用来封装系统的底层实现，隐藏系统的复杂性，提供一组更加简单易用、更高层的接口。比如，Linux 的 Shell 命令，实际上可以看作一种外观模式的应用。它继续封装系统调用，提供更加友好、简单的命令，让我们可以直接通过执行命令来跟操作系统交互。

实际上，从隐藏实现复杂性，提供更易用接口这个目的来看，外观模式符合迪米特法则（最少知道原则）和接口隔离原则：两个有交互的系统，只暴露有限的必要的接口。除此之外，外观模式还也符合封装、抽象的设计思想，提供更抽象的接口，封装底层实现细节。

**2.解决性能问题**

假设有 A 和 B 两个系统，它们之间时通过网络通信的。如果 A 系统完成某个完成某个业务功能需要依次调用 B 系统的 a、b、c三个接口，因自身业务的特点，不支持并发调用这三个接口。如果我们现在 A 系统的响应速度比较慢，排查之后发现，是因为过多的接口调用过多的网络通信。针对这种情况，我们就可以利用外观模式，让 B 系统提供一个包裹 a、b、c 三个接口调用的接口 d。A 系统调用一次接口 d，来获取到所有想要的数据，将网络通信的次数从 3 次减少到 1 次，也就提高了 A 系统的响应速度。

### 总结

外观设计模式更像是客户端应用程序的助手，它不会对客户端隐藏子系统接口，是否使用 Facade 完全取决于客户端代码。   

当客户端与抽象的实现类之间存在许多依赖关系，可以引入外观模式让子系统与客户端和其他子系统分离，从而提高子系统的独立性和可移植性。**外观模式的优点显而易见，客户端不再需要关注实例化时应该使用哪个实现类，直接调用外观模式提供的方法就可以了**。