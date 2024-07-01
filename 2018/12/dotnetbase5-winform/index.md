# DotNet基础 Winform


在windows form开发过程中还是有很多坑需要注意，包括一些重要代码记不得，在这个文件中进行汇总更新。

## 命名规则

&#43; M结尾表示model
&#43; A结尾表示消息
&#43; Object表示 ,底层接口
&#43; Presenter表示，逻辑类
&#43; Transaction表示，具体逻辑
&#43; View表示界面接口
&#43; Helper：表示静态函数
&#43; Statements：表示字符串
&#43; E表示enum
&#43; ~BTN按钮
&#43; 私有变量m_
&#43; 获得Get
&#43; 建立Build
&#43; 生成Generate
&#43; listbox 为 LB


## 一个项目体验

```C#
using System;
using System.Windows.Forms;
namespace AerationSystem
{
    static class Program
    {
        /// &lt;summary&gt;
        /// 应用程序的主入口点。
        /// &lt;/summary&gt;
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            GetAllFrom getform = new GetAllFrom();
            getform.MakeLoginForm();
            getform.login.ShowDialog();
            if(getform.login.DialogResult==DialogResult.OK)
            {
                getform.MakeAllForm();
                Application.Run(getform.currentMain);
                //Application.Run(new Form1());
            }
        }
    }
}
```

读取xml标签类

```C#
public class XMLHelper
{
    /// &lt;summary&gt;
    /// 读取多行同一标签属性
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;filename&#34;&gt;地址&lt;/param&gt;
    /// &lt;param name=&#34;nodeflag&#34;&gt;标签&lt;/param&gt;
    /// &lt;param name=&#34;strflag&#34;&gt;属性&lt;/param&gt;
    /// &lt;returns&gt;多行同一标签属性&lt;/returns&gt;
    public static string[] ReadMultipleTagOneAttribute(string filename,string nodeflag, string strflag)
    {
        try
        {
            XmlDocument xl = new XmlDocument();
            xl.Load(filename);
            XmlNodeList xnl = xl.GetElementsByTagName(&#34;appSettings&#34;)[0].ChildNodes;
            List&lt;string&gt; vs = new List&lt;string&gt;();
            foreach (XmlNode cn in xnl)
            {
                if (cn.Name.Equals(nodeflag))
                {
                    vs.Add(cn.Attributes.GetNamedItem(strflag).Value);
                }
            }
            return vs.ToArray();

        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.StackTrace &#43; ex.Message);
        }
        return null;
    }
    /// &lt;summary&gt;
    /// 读取同一标签多个属性,如果有多个同标签则返回null
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;filename&#34;&gt;地址&lt;/param&gt;
    /// &lt;param name=&#34;nodeflag&#34;&gt;标签&lt;/param&gt;
    /// &lt;param name=&#34;strflag&#34;&gt;多个属性&lt;/param&gt;
    /// &lt;returns&gt;多个属性值&lt;/returns&gt;
    public static string[] ReadMultipleAttributeOneTag(string filename, string nodeflag, string[] strflag)
    {

        try
        {
            XmlDocument xl = new XmlDocument();
            xl.Load(filename);
            XmlNodeList xnl = xl.GetElementsByTagName(&#34;appSettings&#34;)[0].ChildNodes;
            List&lt;string&gt; vs = new List&lt;string&gt;();
            int tagcount = 0;
            foreach (XmlNode cn in xnl)
            {
                if (cn.Name.Equals(nodeflag))
                {
                    foreach (string s in strflag)
                    {
                        vs.Add(cn.Attributes.GetNamedItem(s).Value);
                    }
                    tagcount&#43;&#43;;
                }
            }
            if (tagcount &gt; 1)
            {
                throw new Exception(&#34;出现多个相同标签&#34;);
            }
            return vs.ToArray();

        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.StackTrace &#43; ex.Message);
        }
        return null;
    }
}
```

SQLHelper类

```C#
public class SQLDatabaseHelper
{
    /// &lt;summary&gt;
    /// 返回多个表
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dbname&#34;&gt;数据库URL&lt;/param&gt;
    /// &lt;param name=&#34;sqlstr&#34;&gt;执行语句&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static DataSet QueryDs(String dbname, String sqlstr)
    {
        OleDbConnection Connection = new OleDbConnection(dbname);
        try
        {
            Connection.Open();
            OleDbDataAdapter oleDbAdapter = new OleDbDataAdapter(sqlstr, Connection);

            DataSet ds = new DataSet();
            oleDbAdapter.Fill(ds);
            return ds;
        }
        catch (Exception ex)
        {
            Console.WriteLine(&#34;打开数据库连接异常:&#34; &#43; ex.Message &#43; &#34;\r\n&#34;);
        }
        finally
        {

            Connection.Close();
        }
        return null;
    }
    /// &lt;summary&gt;
    /// 执行语句返会受影响函数
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dbname&#34;&gt;数据库URL&lt;/param&gt;
    /// &lt;param name=&#34;sqlstr&#34;&gt;执行语句&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static int Execute(String dbname, String sqlstr)
    {
        OleDbConnection Connection = new OleDbConnection(dbname);
        try
        {
            Connection.Open();
            OleDbCommand oleDbCommand = new OleDbCommand(sqlstr, Connection);
            return oleDbCommand.ExecuteNonQuery();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
        finally
        {
            Connection.Close();
        }
        return 0;
    }
    /// &lt;summary&gt;
    /// 执行多条语句
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dbname&#34;&gt;数据库url&lt;/param&gt;
    /// &lt;param name=&#34;sqlstrs&#34;&gt;受影响的行数&lt;/param&gt;
    /// &lt;returns&gt;&lt;/returns&gt;
    public static int ExecuteAll(string dbname, string[] sqlstrs)
    {
        int count = 0;
        foreach (string s in sqlstrs)
        {
            count = count &#43; Execute(dbname, s);
        }
        return count;
    }
    /// &lt;summary&gt;
    /// 返回表
    /// &lt;/summary&gt;
    /// &lt;param name=&#34;dbname&#34;&gt;数据库URL&lt;/param&gt;
    /// &lt;param name=&#34;sqlstr&#34;&gt;执行语句&lt;/param&gt;
    /// &lt;returns&gt;受影响行数&lt;/returns&gt;
    public static DataTable QueryDt(String dbname, String sqlstr)
    {
        OleDbConnection Connection = new OleDbConnection(dbname);
        try
        {
            Connection.Open();
            OleDbDataAdapter oleDbAdapter = new OleDbDataAdapter(sqlstr, Connection);
            OleDbCommandBuilder oleDbBuilder = new OleDbCommandBuilder(oleDbAdapter);
            DataSet ds = new DataSet();
            oleDbAdapter.Fill(ds);
            return ds.Tables[0];
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
        finally
        {
            Connection.Close();
        }
        return null;
    }

}
```

```C#
private  ScadaModeM rowToScadaMode(DataRow dataRow)
{
    
    if (dataRow != null)
    {
        ScadaModeM scadaMode = new ScadaModeM();
        if (dataRow.Table.Columns.Contains(&#34;Name&#34;)) scadaMode.Name = dataRow[&#34;Name&#34;].ToString();
        if (dataRow.Table.Columns.Contains(&#34;ConsoleID&#34;)) scadaMode.Code = dataRow[&#34;ConsoleID&#34;].ToString();
        if (dataRow.Table.Columns.Contains(&#34;ConsoleLink&#34;)) scadaMode.ImagePath = dataRow[&#34;ConsoleLink&#34;].ToString();
        if (dataRow.Table.Columns.Contains(&#34;morder&#34;)) scadaMode.Moder = dataRow[&#34;morder&#34;].ToString();
        if (dataRow.Table.Columns.Contains(&#34;active&#34;)) scadaMode.Active = dataRow[&#34;active&#34;].ToString();
        return scadaMode;
    }
    return null;
}
```

正则表达式提取值

```C#
private SQLDataBaseM strToSqlDatabase(string str)
{
    string catalog, server, username, userpassword;
    Match mt = Regex.Match(str, @&#34;server=\w&#43;\.\w&#43;\.\w&#43;\.\w&#43;\,?(\w&#43;)?&#34;);
    if (!mt.Success)
    {
        return null;
    }
    server = mt.Value.Substring(7);
    mt = Regex.Match(str, @&#34;id=\w&#43;&#34;);
    if (!mt.Success)
    {
        return null;
    }
    username = mt.Value.Substring(3);
    mt = Regex.Match(str, @&#34;password=\w&#43;@?\w&#43;&#34;);
    if (!mt.Success)
    {
        return null;
    }
    userpassword = mt.Value.Substring(9);
    mt = Regex.Match(str, @&#34;catalog=\w&#43;&#34;);
    if (!mt.Success)
    {
        return null;
    }
    catalog = mt.Value.Substring(8);
    return new SQLDataBaseM(server, catalog, username, userpassword);
}
```

```C#
public class SQLStatements
{
    /// &lt;summary&gt;
    /// 通过pointcode获得point
    /// &lt;/summary&gt;
    public static string GetPointByCodeStr = &#34;select * from TB_MeasurePoint where MPointCode=&#39;{0}&#39;&#34;;
    public static string GetUserByNameAndPasswordStr = &#34;SELECT *  FROM TB_User where name=&#39;{0}&#39; and password=&#39;{1}&#39;&#34;;

    public static string GetAllFactoryAreaStr = &#34;select distinct BizType from TB_MeasurePoint where active = &#39;启用&#39;&#34;;

    public static string GetAllpointTypesStr = &#34;select distinct SignalType from TB_MeasurePoint where active = &#39;启用&#39;&#34;;

    public static string GetAllEquipmentTypesStr = &#34;select distinct scdtype from TB_MeasurePoint where active = &#39;启用&#39;&#34;;
    /// &lt;summary&gt;
    /// 获得钱100个point的值
    /// &lt;/summary&gt;
    public static string GetTop100PintsByWhereStr = &#34;select top 100 * from TB_MeasurePoint where {0} AND ParmValue &gt; -9999 order by measuredt desc&#34;;
    /// &lt;summary&gt;
    /// 通过约束获得measurepoint
    /// &lt;/summary&gt;
    public static string GetPintsByWhereStr = &#34;select * from TB_MeasurePoint where {0}  order by measuredt desc&#34;;

    /// &lt;summary&gt;
    /// 获得前100个数据
    /// &lt;/summary&gt;
    public static string GetPartlyParasWhereStr = &#34;select top 100 * from tb_mp_{0} where {1} AND ParmValue &gt; -9999 order by measuredt desc&#34;;
    /// &lt;summary&gt;
    /// 查询数据获得翻页效果，第一个是code，第二个是（页数-1）*100，第三个是约束
    /// &lt;/summary&gt;
    public static string GetParasWhereByPageStr = &#34; select top 100 * from tb_mp_{0} where ItemID not in (select top {1} ItemID from tb_mp_{2} where {3} AND ParmValue &gt; -9999 order by measuredt desc )  AND  {4} AND ParmValue &gt; -9999 order by measuredt desc &#34;;
    /// &lt;summary&gt;
    /// 数量，最大值，最小值，平均值
    /// &lt;/summary&gt;
    public static string GetCountMaxMinAvgParaStr = &#34;SELECT COUNT(ParmValue), MAX(ParmValue),MIN(ParmValue),AVG(ParmValue) FROM TB_MP_{0} where {1}  AND ParmValue &gt; -9999&#34;;

    /// &lt;summary&gt;
    /// 将点位变成对应数据写到历史数据里
    /// &lt;/summary&gt;
    public static string GetSpeciPointsByWhereStr = &#34;select MPOINTID,MPOINTCODE,PARMNAME,ISNULL(ParmName, &#39;未知&#39;) &#43;&#39;[&#39;&#43;MPOINTCODE&#43;&#39;]&#39; as 测量点,alarmmax as 超标上限,alarmmin as 超标下限,forcemax as 纵轴上限,forcemin as 纵轴下限,spanrange as 量程,rate,SignalType,Unit,NumTail,biztype,scdtype,morder from TB_MeasurePoint where {0} AND ParmValue &gt; -9999 order by measuredt desc&#34;;
    /// &lt;summary&gt;
    /// 获得point的第二个值
    /// &lt;/summary&gt;
    public static string GetSecondValueByCodeStr = &#34;select top 2 ParmValue from tb_mp_{0} where  ParmValue &gt; -9999 order by measuredt desc&#34;;
    /// &lt;summary&gt;
    /// 获得点的最新的数据时间
    /// &lt;/summary&gt;
    public static string GetLastDateTimeByCodeStr = &#34;select top 1 measuredt  from tb_mp_{0} where ParmValue &gt; -9999 order by measuredt desc&#34;;

    public static string GetPointDataByCodeStr = &#34;select  * from tb_mp_{0} where {1} AND ParmValue &gt; -9999 order by measuredt desc&#34;;
}
```

```C#
DateTime.Now.ToString(&#34;yyyy/MM/dd HH:mm:ss&#34;);
```

生成报表，需要加载Excel.dll

```C#
public void GenerateReportTable(SearchConstrainA sca)
{
    if (sca != null)
    {
        string startpath = Application.StartupPath &#43; &#34;\\日报表\\&#34;;
        if (!Directory.Exists(startpath))
        {
            Directory.CreateDirectory(startpath);
        }
        startpath = startpath &#43; &#34;&#34; &#43; sca.reporttime.ToString() &#43; sca.PumpType.ToString()&#43;&#34;.xls&#34;;
        ExcelHelper.CopyExcel(startpath);
        DataTable dt = database.GetDataFromSC(sca);
        if (dt == null)
        {
            allFrom.form.ShowMessageBox(sca.reporttime &#43; &#34;没有数据&#34;);
            return;
        }
        allFrom.form.ShowMessageBox(&#34;写入参数&#34;, ExcelHelper.FillExcel(dt, startpath, sca));

    }
}
```

```C#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace PunchReport
{
    public class ExcelHelper
    {
        public static void CopyExcel(string targetpath)
        {
            Excel.Application xApp = new Excel.ApplicationClass();
            xApp.WindowState = Excel.XlWindowState.xlMinimized;
            xApp.DisplayAlerts = false;
            xApp.Visible = false;
            object MissingValue = Type.Missing;
            Excel.Workbook xBook = xApp.Workbooks._Open(@&#34;&#34; &#43; Application.StartupPath &#43; &#34;\\report.xls&#34;,
                    MissingValue, MissingValue, MissingValue, MissingValue
                    , MissingValue, MissingValue, MissingValue, MissingValue
                    , MissingValue, MissingValue, MissingValue, MissingValue);
            xBook._SaveAs(@&#34;&#34; &#43; targetpath, MissingValue, MissingValue, MissingValue, MissingValue, MissingValue, Excel.XlSaveAsAccessMode.xlNoChange, MissingValue, MissingValue, MissingValue, MissingValue);
            xBook = null;
            xApp.Quit();
            Kill(xApp);

            xApp = null;
        }

        public static bool FillExcel(DataTable dt, string targetpath,SearchConstrainA sca)
        {
            if (dt == null &amp;&amp; dt.Rows.Count &lt; 0) return false;
            object MissingValue = Type.Missing;
            Excel.Application xApp = new Excel.ApplicationClass();
            Excel.Workbook xBook;
            xApp.WindowState = Excel.XlWindowState.xlMinimized;
            xApp.DisplayAlerts = false;
            xApp.Visible = false;
            try
            {
                xBook = xApp.Workbooks._Open(@&#34;&#34; &#43; targetpath,
                       MissingValue, MissingValue, MissingValue, MissingValue
                       , MissingValue, MissingValue, MissingValue, MissingValue
                       , MissingValue, MissingValue, MissingValue, MissingValue);

                Excel.Worksheet xSheet = (Excel.Worksheet)xBook.Sheets[&#34;统计表&#34;];
                Excel.Range tilterng = xSheet.get_Range(&#34;A1&#34;, MissingValue);
                tilterng.Value2 = sca.PumpType.ToString() &#43; &#34;日报表&#34;;
                Excel.Range tmprng = xSheet.get_Range(&#34;A2&#34;, MissingValue);
                tmprng.Value2 = &#34;报表日期:  &#34; &#43; DateTime.Parse(sca.reporttime).ToString(&#34;yyyy年MM月dd日&#34;);

                for (int j = 0; j &lt; dt.Rows.Count; j&#43;&#43;)
                {
                    for (int i = 0; i &lt; 4; i&#43;&#43;)
                    {
                        tmprng = xSheet.get_Range(getColName(&#34;A&#34;, i) &#43; (4 &#43; j).ToString(), MissingValue);
                        bool a = true;
                        if (tmprng.HasFormula.Equals(a))
                        {
                            continue;
                        }
                        else
                        {
                            tmprng.Value2 = dt.Rows[j][i].ToString();
                        }
                    }

                }
                xBook.Save();
                return true;
            }
            catch (Exception ex)
            {
                BackLogHelper.LogWrite(ex.StackTrace &#43; ex.Message);
                return false;
            }
            finally
            {
                xBook = null;
                xApp.Quit();
                Kill(xApp);
                xApp = null;
            }
            
        }

        private static string getColName(string strTemp, int inc)
        {
            int i = strTemp.Length;
            char[] ca = new char[i];
            ca = strTemp.ToCharArray();

            int w = i - 1;

            int AddResult = (int)ca[w];
            AddResult &#43;= inc;

            if (AddResult &gt;= 91)//需要进一
            {
                ca[w] = (char)(AddResult - 91 &#43; 65);
                if (w &gt; 0)
                {
                    ca[w - 1] = (char)((int)ca[w - 1] &#43; 1);
                    strTemp = ca[w - 1].ToString() &#43; ca[w].ToString();
                }
                else
                {
                    strTemp = &#34;A&#34; &#43; ca[w].ToString();
                }
            }
            else
            {
                strTemp = &#34;&#34;;
                ca[w] = (char)(AddResult);
                while (w &gt;= 0)
                {
                    strTemp = ca[w].ToString() &#43; strTemp;
                    w--;
                }
            }
            return strTemp;
        }

        [DllImport(&#34;User32.dll&#34;, CharSet = CharSet.Auto)]
        public static extern int GetWindowThreadProcessId(IntPtr hwnd, out int ID);
        public static void Kill(Excel.Application xlapp)
        {

            try
            {
                IntPtr app = new IntPtr(xlapp.Hwnd);  //得到这个句柄，具体作用是得到这块内存入口 
                int processid = 0;
                GetWindowThreadProcessId(app, out processid);  //得到本进程唯一标志k
                System.Diagnostics.Process p = System.Diagnostics.Process.GetProcessById(processid);
                p.Kill();    //关闭进程k
            }
            catch
            { }
        }
    }
}
```

使用interop.excel.dll

```C#
/// &lt;summary&gt;
/// Summary description for lib_Excel.
/// &lt;/summary&gt;
public class lib_Excel
{
	public lib_Excel()
	{
		//
		// TODO: Add constructor logic here
		//
	}

	//DataView导入Excel
	public static void DoExport(DataView dv,string strExcelFileName)
	{
		Excel.Application excel= new Excel.Application();
        
		//            Excel.Workbook obj=new Excel.WorkbookClass();
		//            obj.SaveAs(&#34;c:\zn.xls&#34;,Excel.XlFileFormat.xlExcel9795,null,null,false,false,Excel.XlSaveAsAccessMode.xlNoChange,null,null,null,null);

		int rowIndex=1;
		int colIndex=0;

		excel.Application.Workbooks.Add(true);
        
		System.Data.DataTable table=dv.Table ;
		foreach(DataColumn col in table.Columns)
		{
			colIndex&#43;&#43;;    
			excel.Cells[1,colIndex]=col.ColumnName;                
		}
		

		foreach(DataRow row in table.Rows)
		{
			rowIndex&#43;&#43;;
			colIndex=0;
			foreach(DataColumn col in table.Columns)
			{
				colIndex&#43;&#43;;
				excel.Cells[rowIndex,colIndex]=row[col.ColumnName].ToString();
			}
		}
		excel.Visible=false;    
		//   excel.Sheets[0] = &#34;sss&#34;; ///////////////////////////////?????????????????????//
		excel.ActiveWorkbook._SaveAs(strExcelFileName,Excel.XlFileFormat.xlExcel9795,null,null,false,false,Excel.XlSaveAsAccessMode.xlNoChange,null,null,null,null);
		//_SaveAs 比 SaveAs少一个参数
		//_SaveAs 是兼容以前的版本， SaveAs 是office 2003 的新版本
        
        
		//wkbNew.SaveAs strBookName


		//excel.Save(strExcelFileName);
		excel.Quit();
		excel=null;
        
		GC.Collect();//垃圾回收
	}

	//Excel导入DataSet
	public static DataSet ExcelToDS(string Path) 
	{ 
		string strConn = &#34;Provider=Microsoft.Jet.OLEDB.4.0;&#34; &#43;&#34;Data Source=&#34;&#43; Path &#43;&#34;;&#34;&#43;&#34;Extended Properties=Excel 8.0;&#34;; 
		OleDbConnection conn = new OleDbConnection(strConn); 
		conn.Open(); 
		string strExcel = &#34;&#34;; 
		OleDbDataAdapter myCommand = null; 
		DataSet ds = null; 
		strExcel=&#34;select * from [SQL$]&#34;; 
		myCommand = new OleDbDataAdapter(strExcel, strConn); 
		ds = new DataSet(); 
		myCommand.Fill(ds,&#34;table1&#34;); 
		return ds; 
	}
    public static DataTable ExcelToDT(string Path)
    {
        string strConn = &#34;Provider=Microsoft.Jet.OLEDB.4.0;&#34; &#43; &#34;Data Source=&#34; &#43; Path &#43; &#34;;&#34; &#43; &#34;Extended Properties=&#39;Excel 8.0;HDR=NO;IMEX=1&#39;;&#34;;
        OleDbConnection conn = new OleDbConnection(strConn);
        conn.Open();
        DataTable dt = new DataTable();
        string strExcel = &#34;select * from [SQL$]&#34;;
        OleDbDataAdapter myCommand = new OleDbDataAdapter(strExcel, strConn);
        myCommand.Fill(dt);
        foreach (DataRow row in dt.Rows)
        {
            if (!(row[0].ToString()).Equals(&#34;序号&#34;))
            {
                row.Delete();
            }
            else
            {
                break;
            }
        }
        dt = dt.GetChanges(DataRowState.Unchanged);
        for (int i = 0; i &lt; dt.Columns.Count; i&#43;&#43;)
        {
            dt.Columns[i].ColumnName = dt.Rows[0][i].ToString();
        }
        dt.Rows[0].Delete();
        dt = dt.GetChanges(DataRowState.Unchanged);
        return dt;
    }
	
}
```




后台记录log函数

```C#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace PunchReport
{
    public class BackLogHelper
    {
		private static object lockObj = new object();//线程同步锁
        public static void LogWrite(String memo)
        {
            try
            {
				lock (lockObj)
                {
	                StreamWriter sr;
	                //String filename = DateTime.Now.ToShortDateString() &#43; &#34;.txt&#34;;
	                String filename = DateTime.Now.ToString(&#34;yyyy-MM-dd&#34;) &#43; &#34;.txt&#34;;
	                if (!Directory.Exists(Application.StartupPath &#43; &#34;\\logs\\log\\&#34;))
	                {
	                    Directory.CreateDirectory(Application.StartupPath &#43; &#34;\\logs\\log\\&#34;);
	                }
	                if (!File.Exists(Application.StartupPath &#43; &#34;\\logs\\log\\&#34; &#43; filename))
	                {
	                    sr = File.CreateText(Application.StartupPath &#43; &#34;\\logs\\log\\&#34; &#43; filename);
	                    sr.Close();
	                }
	                sr = File.AppendText(Application.StartupPath &#43; &#34;\\logs\\log\\&#34; &#43; filename);
	                sr.WriteLine(memo);
	                sr.Close();
				}
            }
            catch (System.Exception ex)
            {
                Console.WriteLine(&#34;写入日志异常:&#34; &#43; ex);
            }
        }
    }
}
```

```C#
&lt;?xml version=&#34;1.0&#34; encoding=&#34;utf-8&#34; ?&gt;
&lt;configuration&gt;
    &lt;startup&gt; 
        &lt;supportedRuntime version=&#34;v4.0&#34; sku=&#34;.NETFramework,Version=v4.6.1&#34; /&gt;
    &lt;/startup&gt;
  &lt;appSettings&gt;
    &lt;add key = &#34;SQLName&#34; value=&#34;EIP_PRD_JSXJ&#34;/&gt;
    &lt;add key = &#34;SQLIP&#34; value=&#34;127.0.0.1&#34;/&gt;
    &lt;add key = &#34;UserID&#34; value=&#34;sa&#34;/&gt;
    &lt;add key=&#34;SearchTable&#34; value=&#34;KQview&#34;/&gt;
    &lt;add key=&#34;Remove&#34; value=&#34;巡检A组,巡检B组&#34;/&gt;
  &lt;/appSettings&gt;
&lt;/configuration&gt;

string sqlname = ConfigurationManager.AppSettings[&#34;SQLName&#34;].ToString();
string sqlip = ConfigurationManager.AppSettings[&#34;SQLIP&#34;].ToString();
string userid = ConfigurationManager.AppSettings[&#34;UserID&#34;].ToString();
string userpassword = ConfigurationManager.AppSettings[&#34;UserPassword&#34;].ToString();
string removesstr= ConfigurationManager.AppSettings[&#34;Remove&#34;].ToString();

```

控件处于编辑模式

```C#
 // 1、定义一个枚举类型，描述光标状态
private enum EnumMousePointPosition
{
    MouseSizeNone = 0, //&#39;无
    MouseSizeRight = 1, //&#39;拉伸右边框
    MouseSizeLeft = 2, //&#39;拉伸左边框
    MouseSizeBottom = 3, //&#39;拉伸下边框
    MouseSizeTop = 4, //&#39;拉伸上边框
    MouseSizeTopLeft = 5, //&#39;拉伸左上角
    MouseSizeTopRight = 6, //&#39;拉伸右上角
    MouseSizeBottomLeft = 7, //&#39;拉伸左下角
    MouseSizeBottomRight = 8, //&#39;拉伸右下角
    MouseDrag = 9  // &#39;鼠标拖动
}
//2、定义几个变量
const int Band = 5;
const int MinWidth = 10;
const int MinHeight = 10;
private EnumMousePointPosition m_MousePointPosition;
private Point p, p1;
//3、定义自己的MyMouseDown事件
private void MyMouseDown(object sender, System.Windows.Forms.MouseEventArgs e)
{
    p.X = e.X;
    p.Y = e.Y;
    p1.X = e.X;
    p1.Y = e.Y;
}
//4、定义自己的MyMouseLeave事件
private void MyMouseLeave(object sender, System.EventArgs e)
{
    m_MousePointPosition = EnumMousePointPosition.MouseSizeNone;
    this.Cursor = Cursors.Arrow;
}
//5、设计一个函数，确定光标在控件不同位置的样式
private EnumMousePointPosition MousePointPosition(Size size, System.Windows.Forms.MouseEventArgs e)
{

    if ((e.X &gt;= -1 * Band) | (e.X &lt;= size.Width) | (e.Y &gt;= -1 * Band) | (e.Y &lt;= size.Height))
    {
        if (e.X &lt; Band)
        {
            if (e.Y &lt; Band) { return EnumMousePointPosition.MouseSizeTopLeft; }
            else
            {
                if (e.Y &gt; -1 * Band &#43; size.Height)
                { return EnumMousePointPosition.MouseSizeBottomLeft; }
                else
                { return EnumMousePointPosition.MouseSizeLeft; }
            }
        }
        else
        {
            if (e.X &gt; -1 * Band &#43; size.Width)
            {
                if (e.Y &lt; Band)
                { return EnumMousePointPosition.MouseSizeTopRight; }
                else
                {
                    if (e.Y &gt; -1 * Band &#43; size.Height)
                    { return EnumMousePointPosition.MouseSizeBottomRight; }
                    else
                    { return EnumMousePointPosition.MouseSizeRight; }
                }
            }
            else
            {
                if (e.Y &lt; Band)
                { return EnumMousePointPosition.MouseSizeTop; }
                else
                {
                    if (e.Y &gt; -1 * Band &#43; size.Height)
                    { return EnumMousePointPosition.MouseSizeBottom; }
                    else
                    { return EnumMousePointPosition.MouseDrag; }
                }
            }
        }
    }
    else
    { return EnumMousePointPosition.MouseSizeNone; }
}
//6、定义自己的MyMouseMove事件，在这个事件里，会使用上面设计的函数
private void MyMouseMove(object sender, System.Windows.Forms.MouseEventArgs e)
{
    Control lCtrl = (sender as Control);

    if (e.Button == MouseButtons.Left)
    {
        switch (m_MousePointPosition)
        {
            case EnumMousePointPosition.MouseDrag:
                lCtrl.Left = lCtrl.Left &#43; e.X - p.X;
                lCtrl.Top = lCtrl.Top &#43; e.Y - p.Y;
                break;
            case EnumMousePointPosition.MouseSizeBottom:
                lCtrl.Height = lCtrl.Height &#43; e.Y - p1.Y;
                p1.X = e.X;
                p1.Y = e.Y; //&#39;记录光标拖动的当前点
                break;
            case EnumMousePointPosition.MouseSizeBottomRight:
                lCtrl.Width = lCtrl.Width &#43; e.X - p1.X;
                lCtrl.Height = lCtrl.Height &#43; e.Y - p1.Y;
                p1.X = e.X;
                p1.Y = e.Y; //&#39;记录光标拖动的当前点
                break;
            case EnumMousePointPosition.MouseSizeRight:
                lCtrl.Width = lCtrl.Width &#43; e.X - p1.X;
                //      lCtrl.Height = lCtrl.Height &#43; e.Y - p1.Y;
                p1.X = e.X;
                p1.Y = e.Y; //&#39;记录光标拖动的当前点
                break;
            case EnumMousePointPosition.MouseSizeTop:
                lCtrl.Top = lCtrl.Top &#43; (e.Y - p.Y);
                lCtrl.Height = lCtrl.Height - (e.Y - p.Y);
                break;
            case EnumMousePointPosition.MouseSizeLeft:
                lCtrl.Left = lCtrl.Left &#43; e.X - p.X;
                lCtrl.Width = lCtrl.Width - (e.X - p.X);
                break;
            case EnumMousePointPosition.MouseSizeBottomLeft:
                lCtrl.Left = lCtrl.Left &#43; e.X - p.X;
                lCtrl.Width = lCtrl.Width - (e.X - p.X);
                lCtrl.Height = lCtrl.Height &#43; e.Y - p1.Y;
                p1.X = e.X;
                p1.Y = e.Y; //&#39;记录光标拖动的当前点
                break;
            case EnumMousePointPosition.MouseSizeTopRight:
                lCtrl.Top = lCtrl.Top &#43; (e.Y - p.Y);
                lCtrl.Width = lCtrl.Width &#43; (e.X - p1.X);
                lCtrl.Height = lCtrl.Height - (e.Y - p.Y);
                p1.X = e.X;
                p1.Y = e.Y; //&#39;记录光标拖动的当前点
                break;
            case EnumMousePointPosition.MouseSizeTopLeft:
                lCtrl.Left = lCtrl.Left &#43; e.X - p.X;
                lCtrl.Top = lCtrl.Top &#43; (e.Y - p.Y);
                lCtrl.Width = lCtrl.Width - (e.X - p.X);
                lCtrl.Height = lCtrl.Height - (e.Y - p.Y);
                break;
            default:
                break;
        }
        if (lCtrl.Width &lt; MinWidth) lCtrl.Width = MinWidth;
        if (lCtrl.Height &lt; MinHeight) lCtrl.Height = MinHeight;

    }
    else
    {
        m_MousePointPosition = MousePointPosition(lCtrl.Size, e);  //&#39;判断光标的位置状态
        switch (m_MousePointPosition)  //&#39;改变光标
        {
            case EnumMousePointPosition.MouseSizeNone:
                this.Cursor = Cursors.Arrow;       //&#39;箭头
                break;
            case EnumMousePointPosition.MouseDrag:
                // this.Cursor = Cursors.SizeAll;     //&#39;四方向
                this.Cursor = Cursors.Hand;     //&#39;四方向
                break;
            case EnumMousePointPosition.MouseSizeBottom:
                this.Cursor = Cursors.SizeNS;      //&#39;南北
                break;
            case EnumMousePointPosition.MouseSizeTop:
                this.Cursor = Cursors.SizeNS;      //&#39;南北
                break;
            case EnumMousePointPosition.MouseSizeLeft:
                this.Cursor = Cursors.SizeWE;      //&#39;东西
                break;
            case EnumMousePointPosition.MouseSizeRight:
                this.Cursor = Cursors.SizeWE;      //&#39;东西
                break;
            case EnumMousePointPosition.MouseSizeBottomLeft:
                this.Cursor = Cursors.SizeNESW;    //&#39;东北到南西
                break;
            case EnumMousePointPosition.MouseSizeBottomRight:
                this.Cursor = Cursors.SizeNWSE;    //&#39;东南到西北
                break;
            case EnumMousePointPosition.MouseSizeTopLeft:
                this.Cursor = Cursors.SizeNWSE;    //&#39;东南到西北
                break;
            case EnumMousePointPosition.MouseSizeTopRight:
                this.Cursor = Cursors.SizeNESW;    //&#39;东北到南西
                break;
            default:
                break;
        }
    }

}
public void form_load()
{
	for(int i = 0; i &lt; this.panel1.Controls.Count; i&#43;&#43;)   
	 {   
	  this.panel1.Controls[i].MouseDown&#43;= new System.Windows.Forms.MouseEventHandler(MyMouseDown);  
	  this.panel1.Controls[i].MouseLeave&#43;= new System.EventHandler(MyMouseLeave);  
	  this.panel1.Controls[i].MouseMove &#43;= new System.Windows.Forms.MouseEventHandler(MyMouseMove);  
	 }
}  
```

```C#
public class FormBase : Form
{
	/// &lt;summary&gt;
	/// 处理 Windows 消息。
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;m&#34;&gt;要处理的 WindowsMessage。&lt;/param&gt;
	protected override void WndProc(ref Message m)
	{
		try
		{
			switch (m.Msg)
			{
				case (int)WindowsMessage.WM_NCPAINT:
				case (int)WindowsMessage.WM_NCCALCSIZE:
					break;
				case (int)WindowsMessage.WM_NCACTIVATE:
					if (m.WParam == (IntPtr)0)
					{
						m.Result = (IntPtr)1;
					}
					break;
				case (int)WindowsMessage.WM_NCHITTEST://移动鼠标、按住或释放鼠标时释放
					base.WndProc(ref m);
					this.WmNcHitTest(ref m);
					break;
				default:
					base.WndProc(ref m);
					break;
			}
		}
		catch { }
	}
	/// &lt;summary&gt;
	/// 拖动窗口大小
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;m&#34;&gt;&lt;/param&gt;
	private void WmNcHitTest(ref Message m)
	{
		if (this.WindowState != FormWindowState.Maximized)
		{
			int wparam = m.LParam.ToInt32();

			Point point = new Point(
				NativeMethods.LOWORD(wparam),
				NativeMethods.HIWORD(wparam));

			point = PointToClient(point);
			if (_isResize)
			{
				if (point.X &lt;= 3)
				{
					if (point.Y &lt;= 3)
						m.Result = (IntPtr)WM_NCHITTEST.HTTOPLEFT;
					else if (point.Y &gt; Height - 3)
						m.Result = (IntPtr)WM_NCHITTEST.HTBOTTOMLEFT;
					else
						m.Result = (IntPtr)WM_NCHITTEST.HTLEFT;
				}
				else if (point.X &gt;= Width - 3)
				{
					if (point.Y &lt;= 3)
						m.Result = (IntPtr)WM_NCHITTEST.HTTOPRIGHT;
					else if (point.Y &gt;= Height - 3)
						m.Result = (IntPtr)WM_NCHITTEST.HTBOTTOMRIGHT;
					else
						m.Result = (IntPtr)WM_NCHITTEST.HTRIGHT;
				}
				else if (point.Y &lt;= 3)
				{
					m.Result = (IntPtr)WM_NCHITTEST.HTTOP;
				}
				else if (point.Y &gt;= Height - 3)
				{
					m.Result = (IntPtr)WM_NCHITTEST.HTBOTTOM;
				}
				else
				{
					base.WndProc(ref m);
				}
			}
			else
			{
				base.WndProc(ref m);
			}
		}
	}
	//最大化
	protected override void OnMouseDown(MouseEventArgs e)
	{
		base.OnMouseDown(e);
		Point point = e.Location;
		if (e.Button == MouseButtons.Left)
		{
			if (this.CloseRect.Contains(point))
				this.CloseState = EMouseState.Down;
			else if (this.MiniRect.Contains(point))
				this.MinState = EMouseState.Down;
			else if (this.MaxRect.Contains(point))
				this.MaxState = EMouseState.Down;
		}
		if (this.WindowState != FormWindowState.Maximized)
		{
			if (e.Button == MouseButtons.Left &amp;&amp; !this.SysBtnRect.Contains(e.Location))//没有双击按钮
			{
				NativeMethods.ReleaseCapture();
				NativeMethods.SendMessage(Handle, 274, 61440 &#43; 9, 0);
			}
		}
	}
	/// &lt;summary&gt;
	/// &lt;para&gt;该函数从当前线程中的窗口释放鼠标捕获，并恢复通常的鼠标输入处理。捕获鼠标的窗口接收所有&lt;/para&gt;
	/// &lt;para&gt;的鼠标输入（无论光标的位置在哪里），除非点击鼠标键时，光标热点在另一个线程的窗口中。&lt;/para&gt;
	/// &lt;/summary&gt;
	[DllImport(&#34;user32.dll&#34;)]
	public static extern int ReleaseCapture();
	/// &lt;summary&gt;
	/// &lt;para&gt;该函数从当前线程中的窗口释放鼠标捕获，并恢复通常的鼠标输入处理。捕获鼠标的窗口接收所有&lt;/para&gt;
	/// &lt;para&gt;的鼠标输入（无论光标的位置在哪里），除非点击鼠标键时，光标热点在另一个线程的窗口中。&lt;/para&gt;
	/// &lt;/summary&gt;
	[DllImport(&#34;user32.dll&#34;)]
	public static extern int ReleaseCapture();

	#region SendMessage

	/// &lt;summary&gt;
	/// &lt;para&gt;该函数将指定的消息发送到一个或多个窗口。&lt;/para&gt;
	/// &lt;para&gt;此函数为指定的窗口调用窗口程序直到窗口程序处理完消息再返回。&lt;/para&gt;
	/// &lt;para&gt;而函数PostMessage不同，将一个消息寄送到一个线程的消息队列后立即返回。&lt;/para&gt;
	/// return 返回值 : 指定消息处理的结果，依赖于所发送的消息。
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;hWnd&#34;&gt;要接收消息的那个窗口的句柄&lt;/param&gt;
	/// &lt;param name=&#34;Msg&#34;&gt;消息的标识符&lt;/param&gt;
	/// &lt;param name=&#34;wParam&#34;&gt;具体取决于消息&lt;/param&gt;
	/// &lt;param name=&#34;lParam&#34;&gt;具体取决于消息&lt;/param&gt;
	[DllImport(&#34;user32.dll&#34;)]
	public static extern IntPtr SendMessage(IntPtr hWnd, uint Msg, IntPtr wParam, IntPtr lParam);

	/// &lt;summary&gt;
	/// &lt;para&gt;该函数将指定的消息发送到一个或多个窗口。&lt;/para&gt;
	/// &lt;para&gt;此函数为指定的窗口调用窗口程序直到窗口程序处理完消息再返回。&lt;/para&gt;
	/// &lt;para&gt;而函数PostMessage不同，将一个消息寄送到一个线程的消息队列后立即返回。&lt;/para&gt;
	/// return 返回值 : 指定消息处理的结果，依赖于所发送的消息。
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;hWnd&#34;&gt;要接收消息的那个窗口的句柄&lt;/param&gt;
	/// &lt;param name=&#34;Msg&#34;&gt;消息的标识符&lt;/param&gt;
	/// &lt;param name=&#34;wParam&#34;&gt;具体取决于消息&lt;/param&gt;
	/// &lt;param name=&#34;lParam&#34;&gt;具体取决于消息&lt;/param&gt;
	[DllImport(&#34;User32.dll&#34;, CharSet = CharSet.Auto, EntryPoint = &#34;SendMessageA&#34;)]
	public static extern IntPtr SendMessage(IntPtr hWnd, int Msg, int wParam, int lParam);
	/// &lt;summary&gt;
	/// 
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
	/// &lt;returns&gt;&lt;/returns&gt;
	public static int LOWORD(int value)
	{
		return value &amp; 0xFFFF;
	}
	/// &lt;summary&gt;
	/// 
	/// &lt;/summary&gt;
	/// &lt;param name=&#34;value&#34;&gt;&lt;/param&gt;
	/// &lt;returns&gt;&lt;/returns&gt;
	public static int HIWORD(int value)
	{
		return value &gt;&gt; 16;
	}
	/// &lt;summary&gt;
    /// 发送到一个窗口，以确定鼠标在窗口的哪一部分，对应于一个特定的屏幕坐标
    /// &lt;/summary&gt;
    
}
public enum WM_NCHITTEST : int
{
	/// &lt;summary&gt;
	/// 在屏幕背景或窗口之间的分界线
	/// &lt;/summary&gt;
	HTERROR = -2,
	/// &lt;summary&gt;
	/// 在目前一个窗口，其他窗口覆盖在同一个线程
	/// （该消息将被发送到相关窗口在同一个线程，直到其中一个返回一个代码，是不是HTTRANSPARENT）
	/// &lt;/summary&gt;
	HTTRANSPARENT = -1,
	/// &lt;summary&gt;
	/// 在屏幕背景或窗口之间的分界线上
	/// &lt;/summary&gt;
	HTNOWHERE = 0,
	/// &lt;summary&gt;
	/// 在客户端区域
	/// &lt;/summary&gt;
	HTCLIENT = 1,
	/// &lt;summary&gt;
	/// 在标题栏
	/// &lt;/summary&gt;
	HTCAPTION = 2,
	/// &lt;summary&gt;
	/// 在窗口菜单中，或在一个子窗口的关闭按钮
	/// &lt;/summary&gt;
	HTSYSMENU = 3,
	/// &lt;summary&gt;
	/// 在大小框（与HTGROWBO相同）
	/// &lt;/summary&gt;
	HTSIZE = 4,
	/// &lt;summary&gt;
	/// 在大小框（与HTSIZE相同）
	/// &lt;/summary&gt;
	HTGROWBOX = 4,
	/// &lt;summary&gt;
	/// 在一个菜单
	/// &lt;/summary&gt;
	HTMENU = 5,
	/// &lt;summary&gt;
	/// 在水平滚动条
	/// &lt;/summary&gt;
	HTHSCROLL = 6,
	/// &lt;summary&gt;
	/// 在垂直滚动条
	/// &lt;/summary&gt;
	HTVSCROLL = 7,
	/// &lt;summary&gt;
	/// 在最小化按钮
	/// &lt;/summary&gt;
	HTREDUCE = 8,
	/// &lt;summary&gt;
	/// 在最小化按钮
	/// &lt;/summary&gt;
	HTMINBUTTON = 8,
	/// &lt;summary&gt;
	/// 在最大化按钮
	/// &lt;/summary&gt;
	HTMAXBUTTON = 9,
	/// &lt;summary&gt;
	/// 在最大化按钮
	/// &lt;/summary&gt;
	HTZOOM = 9,
	/// &lt;summary&gt;
	/// 在左边框可调整大小的窗口
	/// &lt;/summary&gt;
	HTLEFT = 10,
	/// &lt;summary&gt;
	/// 在一个可调整大小的窗口的右边框
	/// &lt;/summary&gt;
	HTRIGHT = 11,
	/// &lt;summary&gt;
	/// 在窗口的上边框水平线上
	/// &lt;/summary&gt;
	HTTOP = 12,
	/// &lt;summary&gt;
	/// 在窗口的左上边框
	/// &lt;/summary&gt;
	HTTOPLEFT = 13,
	/// &lt;summary&gt;
	/// 在窗口的右上边框
	/// &lt;/summary&gt;
	HTTOPRIGHT = 14,
	/// &lt;summary&gt;
	/// （用户可以在较低的水平边界可调整大小的窗口单击鼠标，改变窗口的垂直大小）
	/// &lt;/summary&gt;
	HTBOTTOM = 15,
	/// &lt;summary&gt;
	/// 在左下角的边框可调整大小的窗口（用户可以通过点击鼠标来调整窗口的大小，对角）
	/// &lt;/summary&gt;
	HTBOTTOMLEFT = 16,
	/// &lt;summary&gt;
	/// 在右下角的边框可调整大小的窗口（用户可以通过点击鼠标来调整窗口的大小，对角）
	/// &lt;/summary&gt;
	HTBOTTOMRIGHT = 17,
	/// &lt;summary&gt;
	/// 在一个不具有缩放边框的窗口
	/// &lt;/summary&gt;
	HTBORDER = 18,
	/// &lt;summary&gt;
	/// 在关闭按钮
	/// &lt;/summary&gt;
	HTCLOSE = 20,
	/// &lt;summary&gt;
	/// 在帮助按钮
	/// &lt;/summary&gt;
	HTHELP = 21,
}
```

通过反射从XML中序列化成元数据读取数据

```C#
private void readXMLCoach()
{
    FileStream fs = new FileStream(CoachFileNane,FileMode.Open);//CoachFileNane文件名字
    XmlDictionaryReader reader =
    XmlDictionaryReader.CreateTextReader(fs, new XmlDictionaryReaderQuotas());
    DataContractSerializer ser = new DataContractSerializer(typeof(ProgramConfiguration));
     ConfigurationValue =
    (ProgramConfiguration)ser.ReadObject(reader, true);//ConfigurationValue读取的类
    reader.Close();
    fs.Close();
}
```

将类序列化成元数据进行写入

```C#
private void writeXMLCoach()
{
    try
    {
        FileStream writer = new FileStream(CoachFileNane, FileMode.Create);
        DataContractSerializer ser =
        new DataContractSerializer(typeof(ProgramConfiguration));
        ser.WriteObject(writer, ConfigurationValue);
        writer.Close();
    }
    catch(Exception ex)
    {
        updataLabel(&#34;写入缓存失败：&#34; &#43; ex.Message);
    }
}
```

```C#
pathTestBtn.PerformClick();//生产按钮响应事件
```

## 自定义控件

&#43; Screen.PrimaryScreen.WorkingArea.Width;获取桌面宽度，hight高度
&#43; this.size程序宽度
&#43; base.Invalidate(this.MenuRect)重绘矩形区域
&#43; this.FormBorderStyle = FormBorderStyle.None;无边框
&#43; this.SetStyle(
  ControlStyles.AllPaintingInWmPaint |
  ControlStyles.OptimizedDoubleBuffer |
  ControlStyles.ResizeRedraw |
  ControlStyles.Selectable |
  ControlStyles.ContainerControl |
  ControlStyles.UserPaint, true);
  this.SetStyle(ControlStyles.Opaque, false);
  this.UpdateStyles();
  绘制控件样式
&#43; Graphics g = e.Graphics;获取画布，如果是创建的使用完后要注销
&#43; FormWindowState.Maximized窗口状态
&#43; FormStartPosition.CenterParent窗体开启位置
&#43; 继承form的重载类中WndProc有窗体循环，在该循环中提前捕获消息进行拦截。
&#43; 设置背景图片 totalpic.ImageLocation = System.Windows.Forms.Application.StartupPath &#43; &#34;\\img\\&#34; &#43; comboBox2.Text.ToString() &#43; &#34;.jpg&#34;;

&#43; 使得panel1不可见的时候panel2填满panel1的区域，让panel1的dock设置为top，panel2的dock设置为fill。


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/12/dotnetbase5-winform/  

