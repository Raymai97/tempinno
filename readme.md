## Android key components

- Linux kernel
  - responsible for device, driver, memory and power management

- Native Libraries
  - such as WebKit, OpenGL and SQLite

- Android Runtime
  - consists of core libraries and Dalvik VM

- Application Framework
  - provide Android APIs such as UI, telephony, resources, locations and package managers

- Applications
  - powered by Application Framework
  - such as Contact, Camera, Phone, Calculator app

## Misc Definition of terms

- GSON
  - a Java library for converting JSON representation into Java Object and vice versa

- OHA / Open Handset Alliance
  - an association whose goal is to
    - develop open standards for mobile devices
    - promote innovation in mobile phones
    - provide a better experience for consumers at a lower cost

- BI / Business Intelligence
  - refers to the use of computing technologies for the identification, discovery and analysis of business data - such as sales revenue, costs and incomes.

- OLAP / Online Analytical Processing

- OLTP / Online Transaction Processing

- SSAS / SQL Server Analysis Services

- API / Application Programming Interface.

- FXML / FX Markup Language.

- RIA / Rich Internet Application.
  - has many characteristics of desktop application software
  - typically delivered via extensive JS, site-specific browser and browser plug-in.

- Scene Builder
  - a freely available GUI design tool for JavaFX application
  - generates FXML code of layout designed by users, allows creating robust GUI without coding

- JavaFX Scene Graph
  - A hierarchical tree of nodes that models the visual elements of JavaFX’s GUI.

## Multi-Dimensional Model

- Just like Entity-Relationship Model (ERM) is for Relational Database;  
Multi-Dimensional Model (MDM) is for Data Warehouse.

- Every MDM consists of
  - one **Fact Table** that stores **measures**
  - one or more **Dimension Table** that store **dimensions**

- **Dimension Table**
  - has **one primary key**
  - has one or more fields that describe the dimension closely
  - has **non-key** columns that are usually **textual** descriptive columns for reporting use

- **Fact Table**
  - links one or more Dimension Table by having a **composite primary key** made up of foreign keys (primary key of every Dimension Table it links)
  - has **non-key** columns that are usually **numeric** columns for use in calculations, which are called **measures** or **measure group**

## OLAP functions

OLAP stands for Online Analytical Processing.

When OLAP functions are combined with standard SQL within the data warehouse, they provide the ability to **analyze large amounts of historical, business transactions** from the past through the present. Also, they provide the ability to **project possible future values**.

## MOLAP, ROLAP, HOLAP

SSAS offers three storage types for Cubes, Partitions and Dimensions.

- MOLAP stands for Multidimensional OLAP.
  - MOLAP is designed to offer **best query performance** to the users.
  - Both data & aggregations are stored in optimized format in the cube. The data inside the cube will refresh only when the cube is processed, hence **high latency**.

- ROLAP stands for Relational OLAP.
  - ROLAP is designed to offer **minimal latency** to the users.
  - Both data & aggregations are stored in relational format. Since the data is always in sync with the source relational database, data cannot be optimized beforehand, hence **worst query performance**.

- HOLAP stands for Hybrid OLAP.
  - HOLAP is a storage type between MOLAP and ROLAP.
  - **Data** are stored in **relational** format, like ROLAP.
  - **Aggregations** are stored in **multidimensional** format, like MOLAP.
  - When source database changes, it sends notification to SSAS so that SSAS will process the aggregation again.
  - It offers **minimal latency** and **moderate query performance**.

## T-SQL and GROUP BY extensions

- T-SQL stands for **Transact-SQL**, it is MS SQL Server primary Relational Database Language, a proprietary extension of standard SQL.

- It supports many Aggregate Functions and elementary grouping (with GROUP BY clause) but as its name implies, it is designed for OLTP, not OLAP.

### OPENJSON function

Syntax:

```
OPENJSON( jsonExpression [, path ] )  [ <with_clause> ]

<with_clause> ::= WITH ( { colName type [ column_path ] [ AS JSON ] } [ ,...n ] )
```

- `path` specify a specific object/array in the JSON
- `with` specify rowset mapping

Example usage:

```sql
DECLARE @json NVARCHAR(MAX) = N'[  
  {  
    "Order": {  
      "Number":"SO43659",  
      "Date":"2011-05-31T00:00:00"  
    },  
    "AccountNumber":"AW29825",  
    "Item": {  
      "Price":2024.9940,  
      "Quantity":1  
    }  
  },  
  {  
    "Order": {  
      "Number":"SO43661",  
      "Date":"2011-06-01T00:00:00"  
    },  
    "AccountNumber":"AW73565",  
    "Item": {  
      "Price":2024.9940,  
      "Quantity":3  
    }  
  }
]';

SELECT *
FROM OPENJSON ( @json )  
WITH (   
	Number   varchar(200)   '$.Order.Number',  
	Date     datetime       '$.Order.Date',  
	Customer varchar(200)   '$.AccountNumber',  
	Quantity int            '$.Item.Quantity',  
	[Order]  nvarchar(MAX)  AS JSON  
);
```

which outputs

| Number | Date | Customer | Quantity | Order |
|--------|------|----------|----------|-------|
| SO43659 | 2011-05-31T00:00:00 | AW29825 | 1 | {"Number":"SO43659","Date":"2011-05-31T00:00:00"} |
| SO43661 | 2011-06-01T00:00:00 | AW73565 | 3 | {"Number":"SO43661","Date":"2011-06-01T00:00:00"} |

### OPENXML function

Syntax:

```
OPENXML( idoc , rowpattern , [ flags ] )   
[ WITH ( SchemaDeclaration | TableName ) ]  
```

- `idoc` specify internal representation of an XML document, which can be created with `sp_xml_preparedocument`.
- `rowpattern` identify the nodes to be processed as rows.
- `flags` specify the mapping used between the XML data and the relational rowset, and how the spill-over column is filled.
  - `0` : Defaults to attribute-centric mapping.
  - `1` : Use the attribute-centric mapping.
  - `2` : Use the element-centric mapping.

Example usage:

```sql
DECLARE @idoc int, @doc varchar(1000);

SET @doc = '
<ROOT>
<Customer CustomerID="VINET" ContactName="Paul Henriot">
   <Order CustomerID="VINET" EmployeeID="5" OrderDate="1996-07-04T00:00:00">
      <OrderDetail OrderID="10248" ProductID="11" Quantity="12"/>
      <OrderDetail OrderID="10248" ProductID="42" Quantity="10"/>
   </Order>
</Customer>
<Customer CustomerID="LILAS" ContactName="Carlos Gonzlez">
   <Order CustomerID="LILAS" EmployeeID="3" OrderDate="1996-08-16T00:00:00">
      <OrderDetail OrderID="10283" ProductID="72" Quantity="3"/>
   </Order>
</Customer>
</ROOT>';

-- Create an internal representation of the XML document.
EXEC sp_xml_preparedocument @idoc OUTPUT, @doc;

-- Execute a SELECT statement that uses the OPENXML rowset provider.
SELECT *
FROM OPENXML(@idoc, '/ROOT/Customer', 1)  
WITH (
	CustomerID  varchar(10),
	ContactName varchar(20)
);
```

which outputs

|CustomerID |ContactName   |
|-----------|--------------|
|VINET      |Paul Henriot  |
|LILAS      |Carlos Gonzlez|

## XML representation of Table

|  Name  |  Value  |
|--------|---------|
|myColor |#DC143C  |
|second  |toSecond |
|minutes |toMinutes|
|calc    |Calculate|

```xml
<record>
	<myColor>#DC143C</myColor>
	<second>toSecond</second>
	<minutes>toMinutes</minutes>
	<calc>Calculate</calc>
</record>
```


## JavaBean naming convention

- `set` method (setter)
  - Always named as field's name prefixed with `set`.
  - Expects argument of same type.
  - Example:
  ```java
  int userAge = 0;
  public void setUserAge(int newVal) {
  	this.userAge = newVal;
  }
  ```
- `get` method (getter)
  - Always named as field's name prefixed with `get`.
  - Returns value of same type.
  - Example:
  ```java
  int userAge = 0;
  public int getUserAge() {
  	return userAge;
  }
  ```

Specific in JavaFX Property
- `get property` method
  - Always named as field's name suffixed with `Property`.
  - Returns `Property` class of same type.
  - Example:
  ```java
  private IntegerProperty userAge = new SimpleIntegerProperty();
  public IntegerProperty userAgeProperty() {
  	return userAge;
  }
  ```

## JavaFX Binding

> A mechanism for expressing links between **Property** objects. Any change occured in one object will be echoed in other automatically.

There are 2 binding mechanisms in JavaFX.

- High-Level API
  - Quickest and easiest way to use bindings.
  - Consist of 2 components:
    - Fluent API
      - Provide methods on the dependency object.
    - `Bindings` class
      - Provide static factory methods.
- Low-Level API
  - For developers who need more flexibility or better performance.

### Example of Fluent API

> **Dependency** refer to object that a binding depends on.

Consider: `num1` (dependency) `num2` (dependency) `sum` (binding)

The following code bind the result of `num1 + num2` to `sum`. Note the usage of `.add()` here.

```java
import javafx.beans.property.IntegerProperty;
import javafx.beans.property.SimpleIntegerProperty;
import javafx.beans.binding.NumberBinding;
public class Main {
	public static void main(String[] args) {
		IntegerProperty num1 = new SimpleIntegerProperty(1);
		IntegerProperty num2 = new SimpleIntegerProperty(2);
		NumberBinding sum = num1.add(num2);
		assert sum.getValue().intValue() == 3;
		num1.set(2);
		assert sum.getValue().intValue() == 4;
	}
}
```

### Example of `Bindings` class

Base on Fluent API example, change `num1.add(num2)` to `Bindings.add(num1, num2)`.

```java
// ...
import javafx.beans.binding.Bindings;
public class Main {
	public static void main(String[] args) {
    	// ...
		NumberBinding sum = Bindings.add(num1, num2);
		// ...
	}
}
```

### Example of Low-Level API

As you see, High-Level API is limited to simple operation like `add` and `multiply`. If you want to use custom operation, simply define your own `Binding` class. Base on Fluent API example, create anonymous class that inherits `IntegerBinding` as below.

```java
// ...
public class Main {
	public static void main(String[] args) {
    	// ...
		NumberBinding sum = new IntegerBinding() {
			{
				super.bind(num1, num2); // make sure bind these
			}
			@Override
			protected int computeValue() {
				final int n1 = num1.get(), n2 = num2.get();
				return (n1 + n2) * n2 - n1; // custom operation here
			}
		};
		// ...
	}
}
```

Note that we create anonymous class that inherits `IntegerBinding` instead of `NumberBinding`, since latter is abstract class that acts as base for other numeric binding classes such as `DoubleBinding`. If you inherit `NumberBinding`, good luck implementing all the abstract methods.

### JavaFX `ChangeListener` interface

- A `ChangeListener` is notified when the value of an `ObservableValue` (usually a `Property` class) changes.

- `ObservableValue` provides `addListener` and `removeListener` methods for adding and removing a `ChangeListener`.

## JavaFX Property

Two types:
- `Simple` prefix
  - can get and set value
  - such as `SimpleIntegerProperty`
- `ReadOnly` prefix
  - can get value only
  - such as `ReadOnlyIntegerProperty`

### Example of `SimpleStringProperty`

```java
import javafx.beans.property.SimpleStringProperty;
import javafx.beans.property.StringProperty;
class SomeClass {
	void SomeMethod() {
		StringProperty text  = new SimpleStringProperty("Initial value");
		// assert text.get() == "Initial value"
		text.set("New value");
		// assert text.get() == "New value"
	}
}
```

Read only property needs two property class, one for internal, one for external. Internal for read and write, external for read only. In JavaFX, internal is called `ReadOnlyWhateverWrapper`, external is called `ReadOnlyWhateverProperty`, where `Whatever` is the data type name like `String`.

Example usage of `ReadOnlyStringWrapper` and `ReadOnlyStringProperty`:
```java
import javafx.beans.property.ReadOnlyStringProperty;
import javafx.beans.property.ReadOnlyStringWrapper;
class SomeClass {
	// ctor (Object bean, String name, String initialValue)
	ReadOnlyStringWrapper userName = new ReadOnlyStringWrapper(this, "userName", "");

	// Usage: String userName = x.userNameProperty().get();
	public final ReadOnlyStringProperty userNameProperty() {
		return userName.getReadOnlyProperty();
	}

	private final void whenNeedToUpdateName() {
		userName.set("New name");
	}
}
```

## JavaFX Concurrent
- `Task` class
  - for implementing asynchronous tasks (background threads)
- `Service` class
  - for executing and managing Tasks
- `Worker` interface
  - `enum Worker.State`
    - `READY` Worker is not running and ready to be executed.
    - `SCHEDULED` Worker has been scheduled for execution, but currently not running.
    - `RUNNING` Worker is currently running.
    - `CANCELLED` Worker has been cancelled via the Worker.cancel() method.
    - `FAILED` Worker has failed, such as when exception thrown.
    - `SUCCEEDED` Worker has completed successfully, and valid `value` has been set.
- `WorkerStateEvent` class
  - tells event when the state of a `Worker` implementation changes

## JavaFX Layout

- AnchorPane
  - anchors the nodes at a particular distance from the pane
- FlowPane
  - wraps all the nodes in a flow
  - horz wraps pane elements at its height
  - vert wraps pane elements at its width
- GridPane
  - arranges the nodes as a grid of rows and columns
- HBox
  - arranges the nodes in a single horizontal row
- VBox
  - arranges the nodes in a single vertical row

## JavaFX Life Cycle

The `javafx.application.Application` class provides methods that define the life cycle of a JavaFX application.

```java
public void init();
public abstract void start(Stage primaryStage);
public void stop();
```

- `init`
  - called after `Application` object instantiated.
  - for application-specific initialization.
  - at this stage JavaFX framework is not ready yet for creating GUI.

- `start`
  - called after JavaFX framework has created GUI thread.
  - for application to init a `Scene` and assign it to the `primaryStage`.

- `stop`
  - called when application is about to terminate.
  - for application to cleanup, free any allocated resource.
