Hackathon 2017
==============

Welcome to hackathon 2017!

Download the kickoff deck for more information on the business problem, the hackathon agenda, the data, and more! Find it in the data folder: [Hackathon 2017 kickoff deck](https://github.com/tresata/hackathon2017/blob/master/data/hackathon_kickoff_deck2017.pdf)

Table of Content: 

* [Getting Started](https://github.com/tresata/hackathon2017#getting-started)
* [Machines](https://github.com/tresata/hackathon2017#machines)
* [HDFS] (https://github.com/tresata/hackathon2017#hdfs)
* [Data] (https://github.com/tresata/hackathon2017#data)
* [Hive] (https://github.com/tresata/hackathon2017#hive)
* [Spark] (https://github.com/tresata/hackathon2017#spark)
* [pySpark] (https://github.com/tresata/hackathon2017#pyspark)
* [Anaconda] (https://github.com/tresata/hackathon2017#anaconda)
* [Scalding] (https://github.com/tresata/hackathon2017#scalding)
* [Tresata Software] (https://github.com/tresata/hackathon2017#tresata-software)
* [Resource Manager] (https://github.com/tresata/hackathon2017#resource-manager)

## Getting Started

You can obtain a username and login information from one of the Tresata representatives or from sponsor's room.

NOTE: You can get the IP addresses from Slack (Infrastruture Room)

ssh into a server where you can access the BBBS data.

    > ssh <username>@hack02.datachambers.com OR
    > ssh <username>@hack03.datachambers.com OR
    > ssh <username>@hack04.datachambers.com OR
    > ssh <username>@hack05.datachambers.com

and enter the password you were given.

We made Hive, Spark, pySpark, R and Anaconda command-line interfaces available, and included a tool to compile and run simple Scalding or Spark scripts on-the-fly.

## Machines

We have a Hadoop cluster with one master and four slaves. The slaves have 32 cores, 6 X 1TB data drives, and 128GB of RAM each. You will have ssh access to the slaves.

Please spread yourselves out across the machines.

## HDFS
To access your HDFS location, you need to use hadoop fs commands (some reference: http://www.folkstalk.com/2013/09/hadoop-fs-shell-command-example-tutorial.html). For example, to take a look at your home directory on HDFS, use

    > hadoop fs -ls

or

    > hadoop fs -ls /user/username

## Data

**Data Disctonary** : It's present on /home/shared/data-dictionary and Slack Channel for DATA

You can find the data on HDFS in the /data folder, which contains the full data set with/without headers

    <To be filled in>

The sample files are also available on the local file system:

    /home/shared/


## Hive

In hive the following tables are available:

       <To be filled in>

Give Hive a whirl and run a sample query:

    > hive

Try pasting the following query into the hive command-line interface:

    hive> show tables;
    OK
    <To be filled in>
    hive> select * from ht_transactions limit 10;

This will return all the fields for the first ten items in the 'ht_transactions' table.

If you'd like to create a file from the command line, you can use a create table command:

    hive> create table test row format delimited fields terminated by '|' stored as textfile as select * from default.ht_transactions limit 10;

You can then extract the table from the hive warehouse for a table named test:

    hadoop fs -text /user/hive/warehouse/test/*.snappy > textfile

## Spark

**Spark-shell** can be found at /usr/local/lib/spark/bin

Now give the Spark-shell a test:

    > /usr/local/lib/spark/bin/spark-shell --num-executors 4 --executor-cores 1 --executor-memory 1G


Read in the data and run a simple query that calcuates the unique count of Donor_ID:

    val df = sqlContext.read.parquet("/data/full-dataset-parquet/donations.parquet")
    df.groupBy("Donor_ID").count().collect()


Note that for your "production" run on the full dataset you might want to increase resources used on the cluster:

    --num-executors 2 --executor-cores 2 --executor-memory 2G

Keep in mind that a spark-shell takes up these resources on the cluster even when you do not use them so please do not keep a spark-shell with "production" resources open unused.  


## pySpark

**pySpark** can be found at /usr/local/lib/spark/bin


You can also do the same query using a python version of the Spark shell.

    >  /usr/local/lib/spark/bin/pyspark --num-executors 4 --executor-cores 1 --executor-memory 1G

Read in the data and run a simple query that calcuates the unique count of Donor_ID:

    df = sqlContext.read.parquet("/data/full-dataset-parquet/donations.parquet")
    df.groupBy("Donor_ID").count().collect()

Note that for your "production" run on the full dataset you might want to increase resources used on the cluster:

    --num-executors 2 --executor-cores 2 --executor-memory 2G

Keep in mind that a pyspark takes up these resources on the cluster even when you do not use them so please do not keep a pyspark shell (interpreter) with "production" resources open unused.  


## Anaconda
Anaconda is a completely free Python distribution from [Continuum Analytics](https://www.continuum.io). It includes more than 400 of the most popular Python packages for science, math, engineering, and data analysis. See [the packages included with Anaconda](http://docs.continuum.io/anaconda/pkg-docs).


Getting failimar with conda: http://conda.pydata.org/docs/using/index.html


## Scalding

In addition to the Hive and Spark shells, we're also packaging eval-tool and df-eval-tool. These are tools to compile and run Scalding and Spark scripts without having to create a project. If you create a file called test.scala with the following contents:

    import com.twitter.scalding._
    import com.tresata.scalding.Dsl._
    import com.tresata.scalding.util.ScaldingUtil

    (args: Args) => {
      new Job(args) {
        ScaldingUtil.sourceFromArg(args("input"))
          .groupBy('Donor_ID) { _
            .size('Donation_Count)
          } 
          .write(ScaldingUtil.sourceFromArg(args("output")))
      }
   } 


you can run a query on the data set sample from the command-line:

    > eval-tool test.scala --hdfs --input bsv%/data/sample-dataset/donations_sample.bsv --output bsv%donation_counts.bsv

This will generate a bar-separated file called 'donation_counts' in your HDFS home directory, containing the Donor numbers along with their total counts.

df-eval-tool

    spark_eval_example.scala
    import com.twitter.scalding.Args
    import com.tresata.spark.sql.Job
    import com.tresata.spark.sql.source.Source

    (args : Args) =>
      new Job(args) {
        override def run = {
          val fapi = Source.fromArg(args, "input").read(minPartitions = 1000).fieldsApi
    
          fapi
            .groupBy('Donor_ID) { _
              .size('Donation_Count)
            }
            .write(Source.fromArg(args, "output"))
        }
      }

run df-eval-tool

    > df-eval-tool test.scala --input bsv%donations.bsv --output bsv%donation_counts.bsv

## Tresata Software

#### TREK
Data Inventory Engine built specifically to catalog, profile and report data ontology, quality and format attributes for all data in Hadoop. TREK rapidly profiles and inventories “as-is” data stored in Hadoop across all rows and columns to create an informed view of all valuable enterprise data feeds stored in a single Hadoop cluster.

#### TIDES
A real time and fully distributed data asset visualization, discovery, query ad aggregation engine with interactive web interface.

TREK AND TIDES can we accessed via https://hack01.datachambers.com:5601

For login, it's the same username and password you use or SSH.

## Resource Manager

http://hack01.datachambers.com:8088/
