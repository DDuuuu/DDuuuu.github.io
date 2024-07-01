# 敏捷软件开发原则模式与实践 咖啡机


这本书的实例非常好，给了我非常多的启发。主要讲了两个实例，咖啡机和薪水支付实例，咖啡机实例比较简单并没有用什么设计模式，薪水支付实例用了很多设计模式，包括后面的打包等。

## 咖啡机实例

做一个使用咖啡机的软件，驱动接口已经被写好。

咖啡机的硬件包括：

&#43; 加热器加热棒（开关）
&#43; 保温盘加热棒（开关）
&#43; 保温盘传感器（保温盘空，杯子空，杯子不空）
&#43; 加热器传感器（有水，没水）
&#43; 冲煮按钮（开关）
&#43; 指示灯（开关）
&#43; 减压阀门（开关）

咖啡机的冲煮流程：

&#43; 咖啡机一次煮12杯咖啡，
&#43; 咖啡加入过滤器，过滤器加入支架，支架放到咖啡机。
&#43; 倒入12杯水到滤水器，按下冲煮，水杯加热至沸腾，水蒸气碰洒到过滤器，形成水滴到咖啡壶，咖啡壶发现有水保温，
&#43; 拿走水杯，停止工作。

### 方案一

建立一个咖啡机超类，关联各个硬件类。这个方案是非常丑陋的，这不是根据行为划分的，有些类，比如light没有任何变量，仅仅调用了驱动接口，这种类叫水蒸气类。没有意义

![](/designpattern/gU6xVKQ-1719811783363-66.png)


### 方案二

按照行为来划分，要确定一些类有没有意义，只需要确定**谁使用他们**，而且要到业务底层去看。把问题的本质和细节分离，忘掉一切，**最根本的问题**是如何煮咖啡。如何煮咖啡，将热水倒到咖啡上，把冲泡好的咖啡收集起来。水从哪来？咖啡放到哪里？那么就有两个类：热水类和支架类，大大多数人会考虑热水流到支架中，这是比较错误的，**软件的行为应该按照软件的行为给出而不是基于一些物理关系**。还需要考虑用户界面，这样就有三个类。

&#43; 谁使用
&#43; 最根本的问题
&#43; 软件行为而不是物理行为

#### 用例

1. 按下冲煮，启动水流，支架做好准备，都准备好就开始煮咖啡
2. 接受器具准备好没有
3. 冲煮完成
4. 咖啡喝完

![](/designpattern/dqRYN3x-1719811783363-67.png)

Containment Vessel:支架和保温壶

Resume:恢复 

a，b，c，d表示四种逻辑：

&#43; a 表示：用户按下冲煮，确保支架中有咖啡壶放在保温杯上，热水器中已经加满了水，然后才开始煮咖啡
&#43; b 表示：如果正在股咖啡的过程中咖啡壶被拿走，则必须中断咖啡流，停止送热水，再次放回咖啡壶继续煮咖啡
&#43; c 表示：热水器中传感器告诉我们水用完了就停止煮咖啡，同时告诉用户和支架（保温盘）已经停止煮咖啡
&#43; d 表示：冲煮结束时并且一个空的咖啡壶放在支架上（保温盘），热水器应该知道这个消息，同时用户也应该知道这个消息

这样整个咖啡机的抽象就完成了，按职责划分，各司其职。这三个抽象类不能知道任何关于咖啡机的任何信息。这就是依赖倒置原则。

系统的控制流如何检测传感器呢？是选择线程还是轮询。最好的总是假设消息都是可以异步发送的，就像存在有独立的线程一样，把使用轮询还是线程的决策推迟到最后一刻。
这样设置了一个接口，main()程序就待在一个循环中，不停地一遍遍调用这个方法实现轮询。

```C#
public static void Main(string[] args)
{
	CoffeeMakerAPI api = new M4CoffeeMakerAPI();
	M4UserInterface ui = new M4UserInterface(api);
	M4HotWaterSOurce hws = new M4HotWaterSOurce(api);
	M4ContainmentVessel cv = new M4ContainmentVessel(api);
	ui.Init(hws,cv);
	hws.Init(ui,cv);
	cv.Init(hws,ui);
	while(true)
	{
		ui.Poll();
		hws.Poll();
		cv.Poll();
	}
}
```


![](/designpattern/9amR9CG-1719811783364-68.png)


依赖倒置，不允许高层的咖啡制作中依赖底层实现。

## 薪水支付实例

该案例主要有做一个薪水支付系统，主要有三类员工

&#43; 临时工：基本时薪，超过8小时加班时间1.5倍工资，每天有考勤卡，每周5结算。
&#43; 正式员工：月薪，每个月最后一天结算。
&#43; 经理：月薪，每月最后一天结算，有项目提成，每隔一周的周五结算，加入公会扣会费。
&#43; 公会会费分服务费和会费：会费每周都有从薪水中扣除，服务费从下个月薪水中扣除。
&#43; 薪水支付方式：可以选择支票邮寄到家，支票保存自取，直接存入银行账号。
&#43; 薪水支付每天运行一次，在当天为相应的雇员进行支付，上一次支付到本次支付应付的数额。

### 用例：

&#43; 新增雇员：雇员号，姓名，地址。（零时工，正式员工，经理）
&#43; 删除雇员：雇员号
&#43; 登记考勤卡：雇员号，日期，小时
&#43; 登记销售凭条：雇员号，日期，销售记录
&#43; 登记公会服务费：公会成员，服务费用
&#43; 更改雇员细则：更改姓名，更改地址，更改每小时报酬，更改薪水，更改提成，更改支付方式，加入公会，离开公会

### 设计类和结构：

通过迭代的方式进行实现。数据库是实现细节，应该尽推迟数据库的设计。通过用例来推导出应该有哪些类。

#### 从用例的角度设计

&#43; 新增雇员

![](/designpattern/4sgdRCQ-1719811783364-69.png)

Hourly:小时工

Commissioned：正式员工

Balarid：经理

把每一项工作划分为自己的类中。这样有可能会创建三个雇员类，但是分析一下就会发现变化的东西太多了，正式由于雇员变化的东西引发雇员类型的改变，只需要将变化的东西抽象出来，在更改雇员细则时改变这些变化的东西就可以改变雇员类型。

&#43; 登记考勤卡

考勤卡和雇员应该是聚合的关系
![](/designpattern/xnDY9RM-1719811783364-70.png)

&#43; 登记销售凭条

销售凭条和雇员也应该是聚合的关系
![](/designpattern/HplaBfD-1719811783364-71.png)

&#43; 登机工会服务费

工会服务费维护着工会会员的编号，因此系统必须要把工会成员和雇员标识联系起俩，推迟这一行为。公会成员和服务费也是聚合的关系
![](/designpattern/brpPGqR-1719811783364-72.png)

&#43; 更改雇员细则

这是由多个更改策略组合而成。

![](/designpattern/ZuREWHC-1601256937615-1719811783364-73.png)

最后各个类之间的关系

#### 从程序运行的角度补充细节

&#43; 运行薪水支付系统：找到所有进行支付的雇员，确定扣款额，根据他们的支付方式支付。
&#43; 抽象出变化的东西：雇员的支付类别抽象，支付时间抽象

![](/designpattern/SpfhQMB-1601256966037-1719811783364-74.png)
![](/designpattern/sIDmVyG-1719811783364-75.png)

&#43; 工会服务费抽象。Affillation:联系

![](/designpattern/b4wSIST-1719811783364-76.png)
![](/designpattern/bfmWeG3-1719811783364-77.png)

### 实现

&#43; 事务

事务是使用命令模式。
![](/designpattern/nBYrNSk-1719811783364-78.png)

&#43; 增加雇员事务，雇员有三个类型，所以使用模板模式来实现增加雇员，此处模板模式的唯一任务就是创建对象

![](/designpattern/aOp2NGw-1719811783364-79.png)

![image-20240701133000020](/designpattern/image-20240701133000020.png)

```C#
public abstract class AddEmployeeTransaction : Transaction
{
	private readonly int empid;
	private readonly string name;
	private readonly string address;

	public AddEmployeeTransaction(int empid, 
		string name, string address, PayrollDatabase database)
		: base (database)
	{
		this.empid = empid;
		this.name = name;
		this.address = address;
	}

	protected abstract 
		PaymentClassification MakeClassification();
	protected abstract 
		PaymentSchedule MakeSchedule();

	public override void Execute()
	{
		PaymentClassification pc = MakeClassification();
		PaymentSchedule ps = MakeSchedule();
		PaymentMethod pm = new HoldMethod();

		Employee e = new Employee(empid, name, address);
		e.Classification = pc;
		e.Schedule = ps;
		e.Method = pm;
		database.AddEmployee(e);
	}

	public override string ToString()
	{
		return String.Format(&#34;{0}  id:{1}   name:{2}   address:{3}&#34;, GetType().Name, empid, name,address);
	} 
}

public class AddSalariedEmployee : AddEmployeeTransaction
{
	private readonly double salary;

	public AddSalariedEmployee(int id, string name, string address, double salary, PayrollDatabase database) 
		: base(id, name, address, database)
	{
		this.salary = salary;
	}

	protected override 
		PaymentClassification MakeClassification()
	{
		return new SalariedClassification(salary);
	}

	protected override PaymentSchedule MakeSchedule()
	{
		return new MonthlySchedule();
	}
}
```

&#43; 删除雇员

提供雇员id，去数据库删除雇员，没啥好说的。

&#43; 考勤卡、销售凭条、服务费

考勤卡：需要参数，雇员id,日期，工作时间

![](/designpattern/HKh6NFO-1719811783365-80.png)
![](/designpattern/Q5vkdbO-1719811783365-81.png)

```C#
public class TimeCard
{
	private readonly DateTime date;
	private readonly double hours;

	public TimeCard(DateTime date, double hours)
	{
		this.date = date;
		this.hours = hours;
	}

	public double Hours
	{
		get { return hours; }
	}

	public DateTime Date
	{
		get { return date; }
	}
}
public class TimeCardTransaction : Transaction
{
	private readonly DateTime date;
	private readonly double hours;
	private readonly int empId;

	public TimeCardTransaction(DateTime date, double hours, int empId, PayrollDatabase database)
		: base(database)
	{
		this.date = date;
		this.hours = hours;
		this.empId = empId;
	}

	public override void Execute()
	{
		Employee e = database.GetEmployee(empId);

		if (e != null)
		{
			HourlyClassification hc =
				e.Classification as HourlyClassification;

			if (hc != null)
				hc.AddTimeCard(new TimeCard(date, hours));
			else
				throw new ApplicationException(
					&#34;Tried to add timecard to&#34; &#43;
						&#34;non-hourly employee&#34;);
		}
		else
			throw new ApplicationException(
				&#34;No such employee.&#34;);
	}
}
```

其他两种与这类似

&#43; 更改雇员属性

更改雇员属性由多个事务集合而成

![](/designpattern/aVSGZwo-1719811783365-82.png)
![](/designpattern/gWlg4Xy-1719811783365-83.png)
![](/designpattern/uUHuGdd-1719811783366-84.png)
![](/designpattern/o7yQ9r5-1719811783366-85.png)

改名字事务：

```C#
public abstract class ChangeEmployeeTransaction : Transaction
{
	private readonly int empId;

	public ChangeEmployeeTransaction(int empId, PayrollDatabase database)
		: base (database)
	{
		this.empId = empId;
	}

	public override void Execute()
	{
		Employee e = database.GetEmployee(empId);
		
		if(e != null)
			Change(e);
		else
			throw new ApplicationException(
				&#34;No such employee.&#34;);
	}

	protected abstract void Change(Employee e);
}
public class ChangeNameTransaction 
	: ChangeEmployeeTransaction
{
	private readonly string newName;

	public ChangeNameTransaction(int id, string newName, PayrollDatabase database)
		: base(id, database)
	{
		this.newName = newName;
	}

	protected override void Change(Employee e)
	{
		e.Name = newName;
	}
}
```

更改雇员类别

![](/designpattern/GeW3N7q-1719811783366-86.png)
![](/designpattern/joOQrUG-1719811783366-87.png)
![](/designpattern/kDPETN3-1719811783366-88.png)
![](/designpattern/Y4oG5M0-1719811783366-89.png)

```C#
public abstract class ChangeClassificationTransaction
	: ChangeEmployeeTransaction
{
	public ChangeClassificationTransaction(int id, PayrollDatabase database)
		: base (id, database)
	{}

	protected override void Change(Employee e)
	{
		e.Classification = Classification;
		e.Schedule = Schedule;
	}

	protected abstract 
		PaymentClassification Classification { get; }
	protected abstract PaymentSchedule Schedule { get; }
}
public class ChangeHourlyTransaction 
	: ChangeClassificationTransaction
{
	private readonly double hourlyRate;

	public ChangeHourlyTransaction(int id, double hourlyRate, PayrollDatabase database)
		: base(id, database)
	{
		this.hourlyRate = hourlyRate;
	}

	protected override PaymentClassification Classification
	{
		get { return new HourlyClassification(hourlyRate); }
	}

	protected override PaymentSchedule Schedule
	{
		get { return new WeeklySchedule(); }
	}
}
public class ChangeSalariedTransaction : ChangeClassificationTransaction
{
	private readonly double salary;

	public ChangeSalariedTransaction(int id, double salary, PayrollDatabase database)
		: base(id, database)
	{
		this.salary = salary;
	}

	protected override PaymentClassification Classification
	{
		get { return new SalariedClassification(salary); }
	}

	protected override PaymentSchedule Schedule
	{
		get { return new MonthlySchedule(); }
	}
}
public class ChangeCommissionedTransaction
	: ChangeClassificationTransaction
{
	private readonly double baseSalary;
	private readonly double commissionRate;

	public ChangeCommissionedTransaction(int id, double baseSalary, double commissionRate, PayrollDatabase database)
		: base(id, database)
	{
		this.baseSalary = baseSalary;
		this.commissionRate = commissionRate;
	}

	protected override PaymentClassification Classification
	{
		get { return new CommissionClassification(baseSalary, commissionRate); }
	}

	protected override PaymentSchedule Schedule
	{
		get { return new BiWeeklySchedule(); }
	}
}
```

改变方法和改变工会实现方式基本与改变雇佣类别相似

改变支付方法：

```C#
public abstract class ChangeMethodTransaction : ChangeEmployeeTransaction
{
	public ChangeMethodTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{}

	protected override void Change(Employee e)
	{
		PaymentMethod method = Method;
		e.Method = method;
	}

	protected abstract PaymentMethod Method { get; }
}
public class ChangeMailTransaction : ChangeMethodTransaction
{
	public ChangeMailTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{
	}

	protected override PaymentMethod Method
	{
		get { return new MailMethod(&#34;3.14 Pi St&#34;); }
	}

}
public class ChangeHoldTransaction : ChangeMethodTransaction
{
	public ChangeHoldTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{
	}

	protected override PaymentMethod Method
	{
		get { return new HoldMethod(); }
	}

}
public class ChangeDirectTransaction : ChangeMethodTransaction
{
	public ChangeDirectTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{
	}

	protected override PaymentMethod Method
	{
		get { return new DirectDepositMethod(&#34;Bank -1&#34;, &#34;123&#34;); }
	}

}
```

改变工会：

```C#
public abstract class ChangeAffiliationTransaction : ChangeEmployeeTransaction
{
	public ChangeAffiliationTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{}

	protected override void Change(Employee e)
	{
		RecordMembership(e);
		Affiliation affiliation = Affiliation;
		e.Affiliation = affiliation;
	}

	protected abstract Affiliation Affiliation { get; }
	protected abstract void RecordMembership(Employee e);
}
public class ChangeUnaffiliatedTransaction : ChangeAffiliationTransaction
{
	public ChangeUnaffiliatedTransaction(int empId, PayrollDatabase database)
		: base(empId, database)
	{}

	protected override Affiliation Affiliation
	{
		get { return new NoAffiliation(); }
	}

	protected override void RecordMembership(Employee e)
	{
		Affiliation affiliation = e.Affiliation;
		if(affiliation is UnionAffiliation)
		{
			UnionAffiliation unionAffiliation = 
				affiliation as UnionAffiliation;
			int memberId = unionAffiliation.MemberId;
			database.RemoveUnionMember(memberId);
		}
	}
}
public class ChangeMemberTransaction : ChangeAffiliationTransaction
{
	private readonly int memberId;
	private readonly double dues;

	public ChangeMemberTransaction(int empId, int memberId, double dues, PayrollDatabase database)
		: base(empId, database)
	{
		this.memberId = memberId;
		this.dues = dues;
	}

	protected override Affiliation Affiliation
	{
		get { return new UnionAffiliation(memberId, dues); }
	}

	protected override void RecordMembership(Employee e)
	{
		database.AddUnionMember(memberId, e);
	}
}
```

&#43; 支付薪水

![](/designpattern/VYRcwVf-1719811783366-90.png)
![](/designpattern/6Xk883H-1719811783366-91.png)
![](/designpattern/d3XMQJP-1719811783366-92.png)
![](/designpattern/4LJ7uRl-1719811783366-93.png)
![](/designpattern/vPEAMS0-1719811783366-94.png)


支付月薪：

```C#
public class PaydayTransaction : Transaction
{
	private readonly DateTime payDate;
	private Hashtable paychecks = new Hashtable();

	public PaydayTransaction(DateTime payDate, PayrollDatabase database)
		: base (database)
	{
		this.payDate = payDate;
	}

	public override void Execute()
	{
		ArrayList empIds = database.GetAllEmployeeIds();
		  
		foreach(int empId in empIds)
		{
			Employee employee = database.GetEmployee(empId);
			if (employee.IsPayDate(payDate)) 
			{
				DateTime startDate = 
					employee.GetPayPeriodStartDate(payDate);
				Paycheck pc = new Paycheck(startDate, payDate);
				paychecks[empId] = pc;
				employee.Payday(pc);
			}
		}
	}

	public Paycheck GetPaycheck(int empId)
	{
		return paychecks[empId] as Paycheck;
	}
}
public class MonthlySchedule : PaymentSchedule
{
	private bool IsLastDayOfMonth(DateTime date)
	{
		int m1 = date.Month;
		int m2 = date.AddDays(1).Month;
		return (m1 != m2);
	}

	public bool IsPayDate(DateTime payDate)
	{
		return IsLastDayOfMonth(payDate);
	}

	public DateTime GetPayPeriodStartDate(DateTime date)
	{
		int days = 0;
		while(date.AddDays(days - 1).Month == date.Month)
			days--;

		return date.AddDays(days);
	}

	public override string ToString()
	{
		return &#34;monthly&#34;;
	}
}
```

其中有paycheck类。该类有日期，总薪酬，服务扣费，实际薪酬

&#43; 临时工


```C#
public class HourlyClassification : PaymentClassification
{
    private double hourlyRate;

    private Hashtable timeCards = new Hashtable();

    public HourlyClassification(double rate)
    {
        this.hourlyRate = rate;
    }

    public double HourlyRate
    {
        get { return hourlyRate; }
    }

    public TimeCard GetTimeCard(DateTime date)
    {
        return timeCards[date] as TimeCard;
    }

    public void AddTimeCard(TimeCard card)
    {
        timeCards[card.Date] = card;
    }

    public override double CalculatePay(Paycheck paycheck)
    {
        double totalPay = 0.0;
        foreach(TimeCard timeCard in timeCards.Values)
        {
            if(DateUtil.IsInPayPeriod(timeCard.Date, 
                paycheck.PayPeriodStartDate, 
                paycheck.PayPeriodEndDate))
                totalPay &#43;= CalculatePayForTimeCard(timeCard);
        }
        return totalPay;
    }

    private double CalculatePayForTimeCard(TimeCard card)
    {
        double overtimeHours = Math.Max(0.0, card.Hours - 8);
        double normalHours = card.Hours - overtimeHours;
        return hourlyRate * normalHours &#43; 
            hourlyRate * 1.5 * overtimeHours;
    }

    public override string ToString()
    {
        return String.Format(&#34;${0}/hr&#34;, hourlyRate);
    }
}
public class WeeklySchedule : PaymentSchedule
{
    public bool IsPayDate(DateTime payDate)
    {
        return payDate.DayOfWeek == DayOfWeek.Friday;
    }

    public DateTime GetPayPeriodStartDate(DateTime date)
    {
        return date.AddDays(-6);
    }

    public override string ToString()
    {
        return &#34;weekly&#34;;
    }
}

public class Employee
{
    private readonly int empid;
    private string name;
    private readonly string address;
    private PaymentClassification classification;
    private PaymentSchedule schedule;	
    private PaymentMethod method;
    private Affiliation affiliation = new NoAffiliation();

    public Employee(int empid, string name, string address)
    {
        this.empid = empid;
        this.name = name;
        this.address = address;
    }

    public string Name
    {
        get { return name; }
        set { name = value; }
    }

    public string Address
    {
        get { return address; }
    }

    public PaymentClassification Classification
    {
        get { return classification; }
        set { classification = value; }
    }

    public PaymentSchedule Schedule
    {
        get { return schedule; }
        set { schedule = value; }
    }

    public PaymentMethod Method
    {
        get { return method; }
        set { method = value; }
    }

    public Affiliation Affiliation
    {
        get { return affiliation; }
        set { affiliation = value; }
    }

    public bool IsPayDate(DateTime date)
    {
        return schedule.IsPayDate(date);
    }

    public void Payday(Paycheck paycheck)
    {
        //计算总的薪资
        double grossPay = classification.CalculatePay(paycheck);
        //计算扣除薪资
        double deductions = affiliation.CalculateDeductions(paycheck);
        //计算实际薪资
        double netPay = grossPay - deductions;
        paycheck.GrossPay = grossPay;
        paycheck.Deductions = deductions;
        paycheck.NetPay = netPay;
        //通过支付方式支付
        method.Pay(paycheck);
    }

    public DateTime GetPayPeriodStartDate(DateTime date)
    {
        return schedule.GetPayPeriodStartDate(date);
    }

    public int EmpId
    {
        get { return empid; }
    }

    public override string ToString()
    {
        StringBuilder builder = new StringBuilder();
        builder.Append(&#34;Emp#: &#34;).Append(empid).Append(&#34;   &#34;);
        builder.Append(name).Append(&#34;   &#34;);
        builder.Append(address).Append(&#34;   &#34;);
        builder.Append(&#34;Paid &#34;).Append(classification).Append(&#34; &#34;);
        builder.Append(schedule);
        builder.Append(&#34; by &#34;).Append(method);
        return builder.ToString();
    }
}

public class UnionAffiliation : Affiliation
{
    private Hashtable charges = new Hashtable();
    private int memberId;
    private readonly double dues;

    public UnionAffiliation(int memberId, double dues)
    {
        this.memberId = memberId;
        this.dues = dues;
    }

    public UnionAffiliation()
        : this(-1, 0.0)
    {}

    public ServiceCharge GetServiceCharge(DateTime time)
    {
        return charges[time] as ServiceCharge;
    }

    public void AddServiceCharge(ServiceCharge sc)
    {
        charges[sc.Time] = sc;
    }

    public double Dues
    {
        get { return dues;	}
    }

    public int MemberId
    {
        get { return memberId; }
    }

    public double CalculateDeductions(Paycheck paycheck)
    {
        double totalDues = 0;

        int fridays = NumberOfFridaysInPayPeriod(
            paycheck.PayPeriodStartDate, paycheck.PayPeriodEndDate);
        totalDues = dues * fridays;

        foreach(ServiceCharge charge in charges.Values)
        {
            if(DateUtil.IsInPayPeriod(charge.Time,
                paycheck.PayPeriodStartDate,
                paycheck.PayPeriodEndDate))
            totalDues &#43;= charge.Amount;
        }

        return totalDues;
    }

    private int NumberOfFridaysInPayPeriod(
        DateTime payPeriodStart, DateTime payPeriodEnd)
    {
        int fridays = 0;
        for (DateTime day = payPeriodStart; 
            day &lt;= payPeriodEnd; day = day.AddDays(1)) 
        {
            if (day.DayOfWeek == DayOfWeek.Friday)
                fridays&#43;&#43;;
        }
        return fridays;
    }
}
```

&#43; 主程序

![](/designpattern/TSzQlyi-1719811783366-95.png)



![](/designpattern/Uo40WBA-1719811783366-96.png)

## 总结

到目前为止基本功能已经实现，仅仅只是用了模板，空值，命令等设计模式，下一篇将会进一步使用更多的设计模式进行打包处理。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/09/designpattern2-case1/  

