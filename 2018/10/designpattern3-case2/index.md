# 敏捷软件开发原则模式与实践 薪水支付


上一章中对薪水支付案例的用例和类做了详细的阐述，在本篇会介绍薪水支付案例包的划分和数据库，UI的设计。

## 包的划分

一个错误包的划分

![](/designpattern/VJJaTih.png)

为什么这个包是错误的：

&#43; 如果对classifications更改就要影响payrolldatabase更改，还会迫使transactions更改，tansactions重新发布和编译测试就是不负责的，transactions没有共享封闭性，每个类都有自己变化的敏感，所以发布的频率非常高，是不合理的。


调整一下：

![](/designpattern/dCTxgiK.png)

将具体类和具体类打包，抽象类和抽象类打包，交互类单独打包。这已经是一个比较好打包设计了。

类的组件应该要符合共同重用原则，payrolldamain中的类没有形成最小的可重用单元，transaction类不必和组件中的其他类一起重用，可以把transaction迁移到transactionapplication类中

![](/designpattern/Mc7GuUc.png)

这样的划分太精细了，是否有这样的必要需要整体来看。


![](/designpattern/m1comVF.png)

![](/designpattern/y2tuVhK.png)

![](/designpattern/Q5hXV24.png)

最终包的结构：

![](/designpattern/1BOrTBm.png)

![](/designpattern/MZKLWCw.png)

![](/designpattern/xTFcgwa.png)

## 数据库的设计

emplogee是核心

![](/designpattern/fNHzXua.png)

完成这个设计需要进行重构：

&#43; 提取出payrolldatabase接口，


  ```C#
  public interface PayrollDatabase
  {
  	void AddEmployee(Employee employee);
  	Employee GetEmployee(int id);
  	void DeleteEmployee(int id);
  	void AddUnionMember(int id, Employee e);
  	Employee GetUnionMember(int id);
  	void RemoveUnionMember(int memberId);
  	ArrayList GetAllEmployeeIds();
  	IList GetAllEmployees();
  }
  ```

&#43; 内存表实例：

  ```C#
  public class InMemoryPayrollDatabase : PayrollDatabase
  {
  	private static Hashtable employees = new Hashtable();
  	private static Hashtable unionMembers = new Hashtable();
  
  	public void AddEmployee(Employee employee)
  	{
  		employees[employee.EmpId] = employee;
  	}
  
  	// etc...
  	public Employee GetEmployee(int id)
  	{
  		return employees[id] as Employee;
  	}
  
  	public void DeleteEmployee(int id)
  	{
  		employees.Remove(id);
  	}
  
  	public void AddUnionMember(int id, Employee e)
  	{
  		unionMembers[id] = e;
  	}
  
  	public Employee GetUnionMember(int id)
  	{
  		return unionMembers[id] as Employee;
  	}
  
  	public void RemoveUnionMember(int memberId)
  	{
  		unionMembers.Remove(memberId);
  	}
  
  	public ArrayList GetAllEmployeeIds()
  	{
  		return new ArrayList(employees.Keys);
  	}
  
  	public IList GetAllEmployees()
  	{
  		return new ArrayList(employees.Values);
  	}
  
  	public void Clear()
  	{
  		employees.Clear();
  		unionMembers.Clear();
  	}
  }
  ```

&#43; 数据库

  ```C#
  public class SqlPayrollDatabase : PayrollDatabase
  {
  	private SqlConnection connection;
  
  	public SqlPayrollDatabase()
  	{
  		connection = new SqlConnection(&#34;Initial Catalog=Payroll;Data Source=localhost;user id=sa;password=abc&#34;);
  		connection.Open();
  	}
  
  	~SqlPayrollDatabase()
  	{
  		connection.Close();
  	}
  
  	public void AddEmployee(Employee employee)
  	{
          //增加员工策略
  		SaveEmployeeOperation operation = new SaveEmployeeOperation(employee, connection);
  		operation.Execute();
  	}
  
  	public Employee GetEmployee(int id)
  	{
          //数据库事务
  		LoadEmployeeOperation loadOperation = new LoadEmployeeOperation(id, connection);
  		loadOperation.Execute();
  		return loadOperation.Employee;
  	}
  
  	public void DeleteEmployee(int id)
  	{
  		throw new NotImplementedException();
  	}
  
  	public void AddUnionMember(int id, Employee e)
  	{
  		throw new NotImplementedException();
  	}
  
  	public Employee GetUnionMember(int id)
  	{
  		throw new NotImplementedException();
  	}
  
  	public void RemoveUnionMember(int memberId)
  	{
  		throw new NotImplementedException();
  	}
  
  	public ArrayList GetAllEmployeeIds()
  	{
  		throw new NotImplementedException();
  	}
  
  	public IList GetAllEmployees()
  	{
  		throw new NotImplementedException();
  	}
  
  }
  ```

&#43; 如果插入雇佣记录成功，但是支付记录失败，为了解决这个问题而使用事务的方式。

  ```C#
  public class SaveEmployeeOperation
  {
  	private readonly Employee employee;
  	private readonly SqlConnection connection;
  	
  	private string methodCode;
  	private string classificationCode;
  	private SqlCommand insertPaymentMethodCommand;
  	private SqlCommand insertEmployeeCommand;
  	private SqlCommand insertClassificationCommand;
  
  	public SaveEmployeeOperation(Employee employee, SqlConnection connection)
  	{
  		this.employee = employee;
  		this.connection = connection;
  	}
  
  	public void Execute()
  	{
  		PrepareToSavePaymentMethod(employee);
  		PrepareToSaveClassification(employee);
  		PrepareToSaveEmployee(employee);
  
  		SqlTransaction transaction = connection.BeginTransaction(&#34;Save Employee&#34;);
  		try
  		{
  			ExecuteCommand(insertEmployeeCommand, transaction);
  			ExecuteCommand(insertPaymentMethodCommand, transaction);
  			ExecuteCommand(insertClassificationCommand, transaction);
  			transaction.Commit();
  		}
  		catch(Exception e)
  		{
  			transaction.Rollback();
  			throw e;
  		}
  	}
  ```


  ```C#
  	private void ExecuteCommand(SqlCommand command, SqlTransaction transaction)
  	{
  		if(command != null)
  		{
  			command.Connection = connection;
  			command.Transaction = transaction;
  			command.ExecuteNonQuery();
  		}
  	}
  
  	private void PrepareToSaveEmployee(Employee employee)
  	{
  		string sql = &#34;insert into Employee values (&#34; &#43;
  			&#34;@EmpId, @Name, @Address, @ScheduleType, &#34; &#43;
  			&#34;@PaymentMethodType, @PaymentClassificationType)&#34;;
  		insertEmployeeCommand = new SqlCommand(sql);
  
  		this.insertEmployeeCommand.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		this.insertEmployeeCommand.Parameters.Add(&#34;@Name&#34;, employee.Name);
  		this.insertEmployeeCommand.Parameters.Add(&#34;@Address&#34;, employee.Address);
  		this.insertEmployeeCommand.Parameters.Add(&#34;@ScheduleType&#34;, ScheduleCode(employee.Schedule));
  		this.insertEmployeeCommand.Parameters.Add(&#34;@PaymentMethodType&#34;, methodCode);
  		this.insertEmployeeCommand.Parameters.Add(&#34;@PaymentClassificationType&#34;, classificationCode);
  	}
  
  	private void PrepareToSavePaymentMethod(Employee employee)
  	{
  		PaymentMethod method = employee.Method;
  		if(method is HoldMethod)
  			methodCode = &#34;hold&#34;;
  		else if(method is DirectDepositMethod)
  		{
  			methodCode = &#34;directdeposit&#34;;
  			DirectDepositMethod ddMethod = method as DirectDepositMethod;
  			insertPaymentMethodCommand = CreateInsertDirectDepositCommand(ddMethod, employee);
  		}
  		else if(method is MailMethod)
  		{
  			methodCode = &#34;mail&#34;;
  			MailMethod mailMethod = method as MailMethod;
  			insertPaymentMethodCommand = CreateInsertMailMethodCommand(mailMethod, employee);
  		}
  		else
  			methodCode = &#34;unknown&#34;;
  	}
  
  	private SqlCommand CreateInsertDirectDepositCommand(DirectDepositMethod ddMethod, Employee employee)
  	{
  		string sql = &#34;insert into DirectDepositAccount values (@Bank, @Account, @EmpId)&#34;;
  		SqlCommand command = new SqlCommand(sql);
  		command.Parameters.Add(&#34;@Bank&#34;, ddMethod.Bank);
  		command.Parameters.Add(&#34;@Account&#34;, ddMethod.AccountNumber);
  		command.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		return command;
  	}		
  	
  	private SqlCommand CreateInsertMailMethodCommand(MailMethod mailMethod, Employee employee)
  	{
  		string sql = &#34;insert into PaycheckAddress values (@Address, @EmpId)&#34;;
  		SqlCommand command = new SqlCommand(sql);
  		command.Parameters.Add(&#34;@Address&#34;, mailMethod.Address);
  		command.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		return command;
  	}
  
  	private void PrepareToSaveClassification(Employee employee)
  	{
  		PaymentClassification classification = employee.Classification;
  		if(classification is HourlyClassification)
  		{
  			classificationCode = &#34;hourly&#34;;
  			HourlyClassification hourlyClassification = classification as HourlyClassification;
  			insertClassificationCommand = CreateInsertHourlyClassificationCommand(hourlyClassification, employee);
  		}
  		else if(classification is SalariedClassification)
  		{
  			classificationCode = &#34;salary&#34;;
  			SalariedClassification salariedClassification = classification as SalariedClassification;
  			insertClassificationCommand = CreateInsertSalariedClassificationCommand(salariedClassification, employee);
  		}
  		else if(classification is CommissionClassification)
  		{
  			classificationCode = &#34;commission&#34;;
  			CommissionClassification commissionClassification = classification as CommissionClassification;
  			insertClassificationCommand = CreateInsertCommissionClassificationCommand(commissionClassification, employee);
  		}
  		else
  			classificationCode = &#34;unknown&#34;;
  		
  	}
  
  	private SqlCommand CreateInsertHourlyClassificationCommand(HourlyClassification classification, Employee employee)
  	{
  		string sql = &#34;insert into HourlyClassification values (@HourlyRate, @EmpId)&#34;;
  		SqlCommand command = new SqlCommand(sql);
  		command.Parameters.Add(&#34;@HourlyRate&#34;, classification.HourlyRate);
  		command.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		return command;
  	}
  
  	private SqlCommand CreateInsertSalariedClassificationCommand(SalariedClassification classification, Employee employee)
  	{
  		string sql = &#34;insert into SalariedClassification values (@Salary, @EmpId)&#34;;
  		SqlCommand command = new SqlCommand(sql);
  		command.Parameters.Add(&#34;@Salary&#34;, classification.Salary);
  		command.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		return command;
  	}
  
  	private SqlCommand CreateInsertCommissionClassificationCommand(CommissionClassification classification, Employee employee)
  	{
  		string sql = &#34;insert into CommissionedClassification values (@Salary, @Commission, @EmpId)&#34;;
  		SqlCommand command = new SqlCommand(sql);
  		command.Parameters.Add(&#34;@Salary&#34;, classification.BaseRate);
  		command.Parameters.Add(&#34;@Commission&#34;, classification.CommissionRate);
  		command.Parameters.Add(&#34;@EmpId&#34;, employee.EmpId);
  		return command;
  	}
  
  	private static string ScheduleCode(PaymentSchedule schedule)
  	{
  		if(schedule is MonthlySchedule)
  			return &#34;monthly&#34;;
  		if(schedule is WeeklySchedule)
  			return &#34;weekly&#34;;
  		if(schedule is BiWeeklySchedule)
  			return &#34;biweekly&#34;;
  		else
  			return &#34;unknown&#34;;
  	}
  }
  ```

## 界面设计

界面设计时最好使得业务行为和UI分离，这里使用model view presenter模式（MVP）

&#43; model：实体层和数据库交互
&#43; view：界面层
&#43; presenter：业务处理层

MVP的作用是解耦界面、业务和实体的关系

&#43; 在presenter中主动使用view，界面的形态都是由presenter去控制，就是在presenter中去注册view事件，当用户触发事件时，这个事件会通过view传递到presenter中，并通过presenter调用model数据方法，最后presenter 调用引用的view实例去改变界面的形态。

![](/designpattern/4uKkRBZ.png)

```C#
public class AddEmployeePresenter
{
	private TransactionContainer transactionContainer;
	private AddEmployeeView view;
	private PayrollDatabase database;

	private int empId;
	private string name;
	private string address;
	private bool isHourly;
	private double hourlyRate;
	private bool isSalary;
	private double salary;
	private bool isCommission;
	private double commissionSalary;
	private double commission;

	public AddEmployeePresenter(AddEmployeeView view, 
		TransactionContainer container, 
		PayrollDatabase database)
	{
		this.view = view;
		this.transactionContainer = container;
		this.database = database;
	}

	public int EmpId
	{
		get { return empId; }
		set
		{
			empId = value;
			UpdateView();
		}
	}

	public string Name
	{
		get { return name; }
		set
		{
			name = value;
			UpdateView();
		}
	}

	public string Address
	{
		get { return address; }
		set
		{
			address = value;
			UpdateView();
		}
	}

	public bool IsHourly
	{
		get { return isHourly; }
		set
		{
			isHourly = value;
			UpdateView();
		}
	}

	public double HourlyRate
	{
		get { return hourlyRate; }
		set
		{
			hourlyRate = value;
			UpdateView();
		}
	}

	public bool IsSalary
	{
		get { return isSalary; }
		set
		{
			isSalary = value;
			UpdateView();
		}
	}

	public double Salary
	{
		get { return salary; }
		set
		{
			salary = value;
			UpdateView();
		}
	}

	public bool IsCommission
	{
		get { return isCommission; }
		set
		{
			isCommission = value;
			UpdateView();
		}
	}

	public double CommissionSalary
	{
		get { return commissionSalary; }
		set
		{
			commissionSalary = value;
			UpdateView();
		}
	}

	public double Commission
	{
		get { return commission; }
		set
		{
			commission = value;
			UpdateView();
		}
	}

	private void UpdateView()
	{
		if(AllInformationIsCollected())
			view.SubmitEnabled = true;
		else
			view.SubmitEnabled = false;
	}

	public bool AllInformationIsCollected()
	{
		bool result = true;
		result &amp;= empId &gt; 0;
		result &amp;= name != null &amp;&amp; name.Length &gt; 0;
		result &amp;= address != null &amp;&amp; address.Length &gt; 0;
		result &amp;= isHourly || isSalary || isCommission;
		if(isHourly)
			result &amp;= hourlyRate &gt; 0;
		else if(isSalary)
			result &amp;= salary &gt; 0;
		else if(isCommission)
		{
			result &amp;= commission &gt; 0;
			result &amp;= commissionSalary &gt; 0;
		}
		return result;
	}

	public TransactionContainer TransactionContainer
	{
		get { return transactionContainer; }
	}

	public virtual void AddEmployee()
	{
		transactionContainer.Add(CreateTransaction());
	}

	public Transaction CreateTransaction()
	{
		if(isHourly)
			return new AddHourlyEmployee(
				empId, name, address, hourlyRate, database);
		else if(isSalary)
			return new AddSalariedEmployee(
				empId, name, address, salary, database);
		else
			return new AddCommissionedEmployee(
				empId, name, address, commissionSalary, 
				commission, database);
	}
}

public interface ViewLoader
{
	void LoadPayrollView();
	void LoadAddEmployeeView(
		TransactionContainer transactionContainer);
}

public class WindowViewLoader : ViewLoader
{
	private readonly PayrollDatabase database;
	private Form lastLoadedView;

	public WindowViewLoader(PayrollDatabase database)
	{
		this.database = database;
	}

	public void LoadPayrollView()
	{
		PayrollWindow view = new PayrollWindow();
		PayrollPresenter presenter = 
			new PayrollPresenter(database, this);

		view.Presenter = presenter;
		presenter.View = view; // 相互关联

		LoadView(view);
	}

	public void LoadAddEmployeeView(
		TransactionContainer transactionContainer)
	{
		AddEmployeeWindow view = new AddEmployeeWindow();
		AddEmployeePresenter presenter = 
			new AddEmployeePresenter(view, 
			transactionContainer, database);

		view.Presenter = presenter;

		LoadView(view);
	}

	private void LoadView(Form view)
	{
		view.Show();
		lastLoadedView = view;
	}
    /// &lt;summary&gt;
    /// 最新的form
    /// &lt;/summary&gt;
	public Form LastLoadedView
	{
		get { return lastLoadedView; }
	}
}

public class PayrollMain
{
	public static void Main(string[] args)
	{
		PayrollDatabase database = 
			new InMemoryPayrollDatabase();
		WindowViewLoader viewLoader = 
			new WindowViewLoader(database);
		
		viewLoader.LoadPayrollView();
		Application.Run(viewLoader.LastLoadedView);
	}
}

public class PayrollPresenter
{
	private PayrollView view;
	private readonly PayrollDatabase database;
	private readonly ViewLoader viewLoader;
	private TransactionContainer transactionContainer;

	public PayrollPresenter(PayrollDatabase database,
		ViewLoader viewLoader)
	{
		//this.view = view;
		this.database = database;
		this.viewLoader = viewLoader;
		TransactionContainer.AddAction addAction = 
			new TransactionContainer.AddAction(TransactionAdded);
		transactionContainer = new TransactionContainer(addAction);
	}

	public PayrollView View
	{
		get { return view; }
		set { view = value; }
	}

	public TransactionContainer TransactionContainer
	{
		get { return transactionContainer; }
	}

	public void TransactionAdded()
	{
		UpdateTransactionsTextBox();
	}

	private void UpdateTransactionsTextBox()
	{
		StringBuilder builder = new StringBuilder();
		foreach(Transaction transaction in 
			transactionContainer.Transactions)
		{
			builder.Append(transaction.ToString());
			builder.Append(Environment.NewLine);
		}
		view.TransactionsText = builder.ToString();
	}

	public PayrollDatabase Database
	{
		get { return database; }
	}

	public virtual void AddEmployeeActionInvoked()
	{
		viewLoader.LoadAddEmployeeView(transactionContainer);
	}

	public virtual void RunTransactions()
	{
		foreach(Transaction transaction in 
			transactionContainer.Transactions)
			transaction.Execute();

		transactionContainer.Clear();
		UpdateTransactionsTextBox();
		UpdateEmployeesTextBox();
	}

	private void UpdateEmployeesTextBox()
	{
		StringBuilder builder = new StringBuilder();
		foreach(Employee employee in database.GetAllEmployees())
		{
			builder.Append(employee.ToString());
			builder.Append(Environment.NewLine);
		}
		view.EmployeesText = builder.ToString();
	}
}

public class TransactionContainer
{
	public delegate void AddAction();

	private IList transactions = new ArrayList();
	private AddAction addAction;

	public TransactionContainer(AddAction action)
	{
		addAction = action;
	}

	public IList Transactions
	{
		get { return transactions; }
	}

	public void Add(Transaction transaction)
	{
		transactions.Add(transaction);
		if(addAction != null)
			addAction();
	}

	public void Clear()
	{
		transactions.Clear();
	}
}

public class AddEmployeeWindow : Form, AddEmployeeView
{
	public System.Windows.Forms.TextBox empIdTextBox;
	private System.Windows.Forms.Label empIdLabel;
	private System.Windows.Forms.Label nameLabel;
	public System.Windows.Forms.TextBox nameTextBox;
	private System.Windows.Forms.Label addressLabel;
	public System.Windows.Forms.TextBox addressTextBox;
	public System.Windows.Forms.RadioButton hourlyRadioButton;
	public System.Windows.Forms.RadioButton salaryRadioButton;
	public System.Windows.Forms.RadioButton commissionRadioButton;
	private System.Windows.Forms.Label hourlyRateLabel;
	public System.Windows.Forms.TextBox hourlyRateTextBox;
	private System.Windows.Forms.Label salaryLabel;
	public System.Windows.Forms.TextBox salaryTextBox;
	private System.Windows.Forms.Label commissionSalaryLabel;
	public System.Windows.Forms.TextBox commissionSalaryTextBox;
	private System.Windows.Forms.Label commissionLabel;
	public System.Windows.Forms.TextBox commissionTextBox;
	private System.Windows.Forms.TextBox textBox2;
	private System.Windows.Forms.Label label1;
	private System.ComponentModel.Container components = null;
	public System.Windows.Forms.Button submitButton;
	private AddEmployeePresenter presenter;

	public AddEmployeeWindow()
	{
		InitializeComponent();
	}

	protected override void Dispose( bool disposing )
	{
		if( disposing )
		{
			if(components != null)
			{
				components.Dispose();
			}
		}
		base.Dispose( disposing );
	}
	public AddEmployeePresenter Presenter
	{
		get { return presenter; }
		set { presenter = value; }
	}

	private void hourlyRadioButton_CheckedChanged(
		object sender, System.EventArgs e)
	{
		hourlyRateTextBox.Enabled = hourlyRadioButton.Checked;
		presenter.IsHourly = hourlyRadioButton.Checked;
	}

	private void salaryRadioButton_CheckedChanged(
		object sender, System.EventArgs e)
	{
		salaryTextBox.Enabled = salaryRadioButton.Checked;
		presenter.IsSalary = salaryRadioButton.Checked;
	}

	private void commissionRadioButton_CheckedChanged(
		object sender, System.EventArgs e)
	{
		commissionSalaryTextBox.Enabled = 
			commissionRadioButton.Checked;
		commissionTextBox.Enabled = 
			commissionRadioButton.Checked;
		presenter.IsCommission = 
			commissionRadioButton.Checked;
	}

	private void empIdTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.EmpId = AsInt(empIdTextBox.Text);
	}

	private void nameTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.Name = nameTextBox.Text;
	}

	private void addressTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.Address = addressTextBox.Text;
	}

	private void hourlyRateTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.HourlyRate = AsDouble(hourlyRateTextBox.Text);
	}

	private void salaryTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.Salary = AsDouble(salaryTextBox.Text);
	}

	private void commissionSalaryTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.CommissionSalary = 
			AsDouble(commissionSalaryTextBox.Text);
	}

	private void commissionTextBox_TextChanged(
		object sender, System.EventArgs e)
	{
		presenter.Commission = AsDouble(commissionTextBox.Text);
	}

	private void addEmployeeButton_Click(
		object sender, System.EventArgs e)
	{
		presenter.AddEmployee();
		this.Close();
	}

	private double AsDouble(string text)
	{
		try
		{
			return Double.Parse(text);
		}
		catch (Exception)
		{
			return 0.0;
		}
	}

	private int AsInt(string text)
	{
		try
		{
			return Int32.Parse(text);
		}
		catch (Exception)
		{
			return 0;
		}
	}

	public bool SubmitEnabled
	{
		set { submitButton.Enabled = value; }
	}
}
```


完毕



---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/10/designpattern3-case2/  

