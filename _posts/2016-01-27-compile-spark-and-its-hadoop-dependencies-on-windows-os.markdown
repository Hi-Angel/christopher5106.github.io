---
layout: post
title:  "Compile Spark on Windows"
date:   2016-01-27 23:00:51
categories: big data
---

Spark has become the main big data tool, very easy to use as well as very powerful. Built on top of some Hadoop classes, Spark offers the use of the distributed memory (RDD) as if you were working on a single machine, and 3 REPL shells `spark-shell`, `pyspark` and `sparkR` for their respective Scala, Python and R languages. It is possible to submit a script with `spark-submit` command and to develop or test locally with `--master local[1]` option, before launching on a [cluster of hundred of instances such as EMR](//christopher5106.github.io/big/data/2016/01/19/computation-power-as-you-need-with-EMR-auto-termination-cluster-example-random-forest-python.html).

Here I recompile Spark on Windows since it avoids problems one could encounter with Windows binaries, such as software version mismatches. Loading and compiling all the required dependencies on a slow network and with standard hardware may require a day or so. The steps are the following :

- Download and install [Java Development Kit 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) in a path such as **C:\Java** (it has to be a folder with spaces)

- Download and install [Python 2.7.11](https://www.python.org/downloads/).

- Add **C:\Python27\;C:\Java** to your `Path` environment variable, **C:\Java** to your `JAVA_HOME` env var.

Check everything works well :

{% highlight bash %}
where java
#C:\Java\bin\java.exe
#C:\Windows\System32\java.exe
{% endhighlight %}

our install arrives first

{% highlight bash %}
java -version
#java version "1.7.0_79"
#Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
#Java Hotspot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)
{% endhighlight %}

Java should be 64bit to increase memory above 2G. Try javac

{% highlight bash %}
javac
{% endhighlight %}

I also had to change the memory options to `-Xmx2048m` (instead of 516m) in **C:\Program Files (x86)\sbt\conf\sbtconfig.txt**.

{% highlight bash %}
python --version
# Python 2.7.11
{% endhighlight %}

- Download and compile Spark :

{% highlight bash %}
git clone git://github.com/apache/spark.git
sbt package
sbt assembly
{% endhighlight %}

that will create the Spark assembly JAR.

<a name="workaround" />

- There is a small bug for Pyspark

In the *main function* in **python/pyspark/worker.py**, add the two last lines to the *process function* :

{% highlight python %}
def process():
  iterator = deserializer.load_stream(infile)
  serializer.dump_stream(func(split_index, iterator), outfile)
  for obj in iterator:
    pass
{% endhighlight %}

Rebuild the pyspark.zip in the python/lib folder or [download it here]({{ site.url }}/img/pyspark.zip).

Now it's done. Launch Pypark:

{% highlight bash %}
bin\pyspark --master local[1]
{% endhighlight %}
