````c#

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Oracle.DataAccess.Client;

namespace RedisSync
{
    
    public class LBSDatabaseChangeListener
    {
        OracleDependency dep;
        OracleConnection conn;
        //注册通知

        public LBSDatabaseChangeListener()
        {
            //设置App的监听端口，即使用哪个端口接收Change Notification。
            //OracleDependency.Port = 49500;
            //string cs = "data source=222.185.255.10:38189/oralnx;persist security info=True;user id=lbs;password=lbs";
            //string cs = "data source=221.226.150.180:1521/ORALNX;persist security info=True;user id=lbs;password=lbs";
            string cs = "data source=221.226.150.180:1521/ORALNX;persist security info=True;user id=lbsbus;password=lbsbus";
            //string cs = "data source=202.102.108.23:12521/oralnx;persist security info=True;user id=lbs;password=lbs";
            conn = new OracleConnection(cs);
            conn.Open();
        }

        //注册
        public void Register()
        {
            //SELECT T.*, ROWID FROM " + table.getName()+ " T WHERE 1=2"
            OracleCommand cmd = new OracleCommand("SELECT T.*, ROWID FROM ATTACHFILES T ", conn);//WHERE 1=2
            //绑定OracleDependency实例与OracleCommand实例
            dep = new OracleDependency(cmd);
            //指定Notification是object-based还是query-based，前者表示表（本例中为tab_cn）中任意数据变化时都会发出Notification；后者提供更细粒度的Notification，例如可以在前面的sql语句中加上where子句，从而指定Notification只针对查询结果里的数据，而不是全表。
            dep.QueryBasedNotification = false;
            //是否在Notification中包含变化数据对应的RowId
            dep.RowidInfo = OracleRowidInfo.Include;
            //指定收到Notification后的事件处理方法
            dep.OnChange += new OnChangeEventHandler(OnNotificaton);
            //是否在一次Notification后立即移除此次注册
            cmd.Notification.IsNotifiedOnce = false;
            //此次注册的超时时间（秒），超过此时间，注册将被自动移除。0表示不超时。
            cmd.Notification.Timeout = 360000;
            //False表示Notification将被存于内存中，True表示存于数据库中，选择True可以保证即便数据库重启之后，消息仍然不会丢失
            cmd.Notification.IsPersistent = true;
            //
            OracleDataReader odr = cmd.ExecuteReader();
            //
            //this.rtb1.AppendText("Registration completed. " + DateTime.Now.ToLongTimeString() + Environment.NewLine);

            Console.WriteLine("Registration completed. " + DateTime.Now.ToLongTimeString() + Environment.NewLine);
        }


        //解除注册
        public void UnRegister()
        {
            //注销
            dep.RemoveRegistration(conn);
            //this.rtb1.AppendText("Registration Removed. " + DateTime.Now.ToLongTimeString() + Environment.NewLine);
             Console.WriteLine("Registration Removed. " + DateTime.Now.ToLongTimeString() + Environment.NewLine);
        }
        public static void OnNotificaton(object src, OracleNotificationEventArgs arg)
        {
            Console.WriteLine("-------------Notificaton Received :  " + DateTime.Now.ToLongTimeString() + "------------------------. " + DateTime.Now.ToLongTimeString() + Environment.NewLine);
            Console.WriteLine("  Changed data(rowid): " + arg.Details.Rows[0]["rowid"].ToString() + Environment.NewLine);
            Console.WriteLine("  Changed data(ResourceName): " + arg.Details.Rows[0]["ResourceName"].ToString() + Environment.NewLine);
            Console.WriteLine("  Changed data(status): " + arg.Info.ToString() + Environment.NewLine);
            Console.WriteLine("  Changed data(Count): " + arg.Details.Rows.Count + Environment.NewLine);
           // Console.WriteLine("  Changed data(Count): " + arg.Details. + Environment.NewLine);
            //可以从arg.Details中获得通知的具体信息，比如变化数据的RowId
            //DataTable dt = arg.Details;
            ////......
            //this.rtb1.Dispatcher.BeginInvoke(
            //    DispatcherPriority.Normal,
            //    new Action(() =>
            //    {
            //        this.rtb1.AppendText("Notification Received. " + DateTime.Now.ToLongTimeString() + "  Changed data(rowid): " + arg.Details.Rows[0]["rowid"].ToString() + Environment.NewLine);
            //    }));
        }
    }
}
````
