<properties linkid="manage-services-hdinsight-using-client-library" urlDisplayName="Using the HDInsight Service Client Library" pageTitle="How to use the HDInsight Client Library - Windows Azure guidance" metaKeywords="hdinsight library, hdinsight .net, hdinsight .net azure" metaDescription="Learn how to use the .Net client Libraries for HDInsight." umbracoNaviHide="0" disqusComments="1" writer="sburgess" editor="mollybos" manager="paulettm" />

<div chunk="../chunks/hdinsight-left-nav.md" />

#Using the HDInsight Service Client Library#

The HDInsight Service library provides set of .NET client libraries that makes it easier to work with HDInsight in .NET. In this tutorial you will learn how to get the client library and use it to build simple .NET based Hadoop program to run Hive queries. 

To enable the HDInsight preview, click [here](https://account.windowsazure.com/PreviewFeatures).

## In this Article

* [Downloading an installing the library](#install)
* [Preparing for the tutorial](#prepare)
* [Creating and Runing a .NET program](#create)
* [Next Steps](#nextsteps)

##<a id="install"></a> Downloading and Installing the Library##

You can install latest published build of the library from [NuGet](http://nuget.codeplex.com/wikipage?title=Getting%20Started). The library includes the following components:

* **MapReduce library:** This library simplifies writing MapReduce jobs in .NET languages using the Hadoop streaming interface.
* **LINQ to Hive client library:** This library translates C# or F# LINQ queries into HiveQL queries and executes them on the Hadoop cluster. This library can execute arbitrary HiveQL queries from a .NET program as well.
* **WebClient library:** This library contains client libraries for *WebHDFS* and *WebHCat*.

	* **WebHDFS client library:** It works with files in HDFS and Windows Azure Blog Storage
	* **WebHCat client library:** It manages scheduling and execution of jobs in HDInsight cluster
	
The NuGet syntax to install the libraries:

		install-package Microsoft.Hadoop.MapReduce
		install-package Microsoft.Hadoop.Hive 
		install-package Microsoft.Hadoop.WebClient 
			
These commands add .NET libraries and references to them to the current Visual Studio project.

##<a id="prepare"></a> Preparing for the Tutorial

You must have a [Windows Azure subscription][free-trial], and a [Windows Azure Storage Account][create-storage-account] before you can proceed. You must also know your Windows Azure storage account name and account key. For the instructions on how to  get this information, see the *How to: View, copy and regenerate storage access keys* section of [How to Manage Storage Accounts](/en-us/manage/services/storage/how-to-manage-a-storage-account/).


You must also download the Actors.txt file used in this tutorial. Perform the following steps to download this file to your development environment:

1. Create a C:\Tutorials folder on your local computer.
2. Download [Actors.txt](http://www.microsoft.com/en-us/download/details.aspx?id=37003), and save the file to the C:\Tutorials folder.

##<a id="create"></a>Creating and Executing a .NET Program

In this section you will learn how to upload files to Hadoop cluster programmatically and how to execute Hive jobs using LINQ to Hive.

1. Open Visual Studio 2012.
2. From the File menu, click **New**, and then click **Project**.
3. From New Project, type or select the following values:

	<table>
	<tr><th>Property</th><th>Value</th></tr>
	<tr><td>Category</td><td>Templates/Visual C#/Windows</td></tr>
	<tr><td>Template</td><td>Console Application</td></tr>
	<tr><td>Name</td><td>SimpleHiveJob</td></tr>
	</table>

4. Click **OK** to create the project.


3. From the **Tools** menu, click **Library Package Manager**, click **Package Manager Console**.
4. Run the following commands in the console to install the packages.

		install-package Microsoft.Hadoop.Hive -pre 
		install-package Microsoft.Hadoop.WebClient -pre 

	These commands add .NET libraries and references to them to the current Visual Studio project.



6. From Solution Explorer, double-click **Program.cs** to open it.
7. Add the following using statements to the top of the file:

		using Microsoft.Hadoop.WebHDFS.Adapters;
		using Microsoft.Hadoop.WebHDFS;
		using Microsoft.Hadoop.Hive;
	
7. In the Main() function, copy and paste the following code:
		
		// Upload actors.txt to Blob Storage
		var asvAccount = [Storage account name];
		var asvKey = [Storage account key];
		var asvContainer = [Container name];
		var localFile = "C:/Tutorials/Actors.txt"
		var hadoopUser = [Hadoop user name];
		var hadoopUserPassword = [Hadoop user password];
		var clusterURI = [HDInsight cluster URL]; //"http://HDInsightCluster Name.azurehdinsight.net";
		
		var storageAdapter = new BlobStorageAdapter(asvAccount, asvKey, asvContainer, true);
		var HDFSClient = new WebHDFSClient(hadoopUser, storageAdapter);
		
		HDFSClient.CreateDirectory("/user/hadoop/MovieData").Wait();
		HDFSClient.CreateFile(localFile, "/user/hadoop/MovieData/Actors.txt").Wait();
		
		// Create Hive table from actors.txt
		var hiveConnection = new HiveConnection(
		  new System.Uri(clusterURI),
		  hadoopUser, 
		  hadoopUserPassword,
		  asvAccount, 
		  asvKey);
		
		// Drop any existing tables called Actors
		//hiveConnection.GetTable<HiveRow>("Actors").Drop();
		string command =
		  @"CREATE TABLE Actors(
		            MovieId string, 
		            ActorId string,
		            Name string, 
		            AwardsCount int) 
		            row format delimited fields 
		        terminated by ',';";
		hiveConnection.ExecuteHiveQuery(command).Wait();
		
		command =
		  @"LOAD DATA INPATH 
		        '/user/hadoop/MovieData/Actors.txt'
		    OVERWRITE INTO TABLE Actors;";
		hiveConnection.ExecuteHiveQuery(command).Wait();
		
		// Execute Hive query
		var result = hiveConnection.ExecuteQuery(@"select name, awardscount
		          from actors sort by awardscount desc
		          limit 1");
		result.Wait();
		Console.WriteLine(result.Result.ReadToEnd());
		Console.ReadLine();

9. Press **F5** to run the program.

##<a id="nextsteps"></a>Next Steps
Now you understand how to create a .NET application using HDInsight client SDK. To learn more, see the following articles:

* [Getting Started with Windows Azure HDInsight Service](/en-us/manage/services/hdinsight/get-started-hdinsight/)
* [Using Pig with HDInsight][hdinsight-pig] 
* [Using MapReduce with HDInsight][hdinsight-mapreduce]
* [Using Hive with HDInsight](/en-us/manage/services/hdinsight/using-hive/)

[hdinsight-pig]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
[hdinsight-mapreduce]: /en-us/manage/services/hdinsight/using-mapreduce-with-hdinsight/



[free-trial]: http://www.windowsazure.com/en-us/pricing/free-trial/
[create-storage-account]: http://www.windowsazure.com/en-us/manage/services/storage/how-to-create-a-storage-account/


