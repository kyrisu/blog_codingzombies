---
layout: post
title: Why SoapIgnore doesn't work with SoapFormatter?
categories:
- coding
tags:
- c#
- serialization
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
  _edit_last: '1'
---
This post was inspired by <a href="http://stackoverflow.com/questions/2283482/applying-soapignore-attribute-doesnt-take-any-effect-to-serialization-result/2285142#2285142" target="_blank">StackOverflow question</a> about using SoapIgnore in SoapFormatter.

First quick example from the question itself:
{% highlight csharp %}
[Serializable]
public class SampleClass
{
    [SoapIgnore]
    public Guid InstanceId
    {
        get;
        set;
    }
}

class Program
{
    static void Main()
    {

        SampleClass cl = new SampleClass { InstanceId = Guid.NewGuid() };
        SoapFormatter fm = new SoapFormatter();
        using (FileStream stream = new FileStream(string.Format("C:\\Temp\\{0}.inv", Guid.NewGuid().ToString().Replace("-", "")), FileMode.Create))
        {
            fm.Serialize(stream, cl);
        }
    }
}
{% endhighlight %}
.. this code will not work the way expected. Guid property won't be ignored, and created file will contain it's value. The problem with this code is that it's trying to use serialization designated for Remoting (<a href="http://msdn.microsoft.com/en-us/library/system.runtime.serialization.formatters.soap.aspx">System.Runtime.Serialization.Formatters.Soap</a> namespace) with the SoapIgnore (<a href="http://msdn.microsoft.com/en-us/library/system.xml.serialization.aspx">System.Xml.Serialization</a> namespace).

<strong>How it should be done then?</strong>

Depending on what you want to accomplish:

Use [<a href="http://msdn.microsoft.com/en-us/library/system.nonserializedattribute.aspx" target="_blank">NonSerialized</a>] (from System namespace) to prevent Guid property from serializing or..

use XML serialization to serialize the class like so:

{% highlight csharp %}
[Serializable]
public class SampleClass
{
    [SoapIgnore]
    public Guid InstanceId
    {
        get;
        set;
    }
}

class Program
{
    static void Main()
    {
        SampleClass cl = new SampleClass { InstanceId = Guid.NewGuid() };
        XmlTypeMapping xtm = new SoapReflectionImporter().ImportTypeMapping(typeof(SampleClass));
        XmlSerializer xs = new XmlSerializer(xtm);
        
        using (FileStream stream = new FileStream(string.Format("C:\\Temp\\{0}.inv", Guid.NewGuid().ToString().Replace("-", "")), FileMode.Create))
        {
            xs.Serialize(stream, cl);
        }
    }
}
{% endhighlight %}

Note at the <a href="http://msdn.microsoft.com/en-us/library/system.runtime.serialization.formatters.soap.soapformatter.aspx" target="_blank">MSDN </a>site states:
<blockquote>Beginning with the .NET Framework version 3.5, this class is obsolete. Use BinaryFormatter instead.</blockquote>
..so maybe it's time to stop using SoapFormatter anyway ;)
