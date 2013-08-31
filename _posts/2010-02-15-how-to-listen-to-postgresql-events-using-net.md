---
layout: post
title: How to listen to PostgreSQL events using .NET?
categories:
- coding
tags:
- .NET
- c#
- coding
- databases
- npgsql
- postgresql
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
---
In a recent project I needed to monitor PostgreSQL database. Needless to say I never worked with this database and I had a hard time finding anything interesting on the net. Finally after asking a <a href="http://stackoverflow.com/questions/2147353/how-to-make-postgresql-trigger-and-c-windows-service-work-together" target="_blank">question</a> on <a title="StackOverflow" href="http://stackoverflow.com" target="_blank">StackOverflow</a> I was directed to SQL notify mechanism. I've spend some time going through the documentation and I want to share what I've found out.

Before you begin you will need .NET data provider for PostgreSQL which you can download <a href="http://npgsql.projects.postgresql.org/" target="_blank">here</a>.

<strong>Some theory</strong>

Many databases support something that is called <em>triggers</em> - it's like .NET event, you tell it when it should execute a function. To add a <em>trigger</em> you need to execute some SQL code which you can throw at you database from you favourite database management tool or by executing NpgsqlCommand.ExecuteNonQuery() from you .NET application.

<strong>The code</strong>

Before you create your trigger you need to define a function that you want execute based on the event.
{% highlight sql %}CREATE OR REPLACE FUNCTION notify_demo() RETURNS TRIGGER AS '
BEGIN
	NOTIFY demoApp;
	RETURN NULL;
END;
' LANGUAGE plpgsql;{% endhighlight %}
The only thing to remember is that function cannot take any arguments and that it needs to return TRIGGER. In the function itself we use <em>notify</em> and set the name of the notification (later we will <em>listen</em> for a notification with the same name).
At this point we are ready to create the trigger.
{% highlight sql %}CREATE TRIGGER demo AFTER UPDATE
   ON users FOR EACH ROW
   EXECUTE PROCEDURE notify_demo();{% endhighlight %}
It easy to see that we create a trigger named <em>demo</em> which will execute function <em>notify_demo()</em> after update on table <em>users</em>. We can set the trigger to execute before or after any of the fallowing events: <em>insert</em>, <em>update</em>, <em>delete </em>(all of which work on rows) and <em>truncate </em>(which works on statements).

In my project function and the trigger are installed by the setup process by executing <em>NpgsqlCommand.ExecuteNonQuery().</em>

When we have our trigger in place it's time to write some .NET code :)
{% highlight csharp %}using System;
using Npgsql;

namespace TriggerDemo
{
    class Program
    {
        static void Main(string[] args)
        {

            string connString = "Server=127.0.0.1;Port=5432;User Id=postgres;Password=pass;SyncNotification=true";
            NpgsqlConnection conn = new NpgsqlConnection(connString);

            try
            {
                conn.Open();
                NpgsqlCommand cmd = new NpgsqlCommand("listen demoApp;", conn);
                cmd.ExecuteNonQuery();
                conn.Notification += new NotificationEventHandler(conn_Notification);
                Console.ReadLine();
                conn.Close();
            }
            catch (NpgsqlException ex)
            {
                Console.WriteLine(ex.Message);
                Console.ReadLine();
            }
        }

        static void conn_Notification(object sender, NpgsqlNotificationEventArgs e)
        {
            Console.WriteLine("Row was updated");
        }
    }
}{% endhighlight %}
In the code you can see standard usage of ADO.NET (if you don't know what it is ask uncle Google - there are some great tutorials out there). The things you should notice are the fallowing:
<ul>
	<li><em>SyncNotification=true</em> - what it does is opens another thread and using TCP based connection will regularly ask the database if anything interesting happen. This means that you can easily put your database monitoring application on a different machine until the network connection is present and both machines can see each other. One thing you need to remember is that when connection has <em>SyncNotification</em> set to true you cannot execute other queries inside of the notification handler function or it will hang npgsql - you need to create a new connection.</li>
	<li><em>listen demoApp </em>- this command listens to particular named trigger (in our case "demoApp")</li>
	<li><em>conn.Notification += new NotificationEventHandler(conn_Notification); </em> - well till now you should know this drill ;) we connect a delegate to an event.</li>
</ul>
Congratulations - your program is now monitoring changes in PostgreSQL database.
