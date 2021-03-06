<properties linkid="manage-services-hdinsight-using-blob-store-hdinsight" urlDisplayName="Using Windows Azure Blob Store with HDInsight" pageTitle="Using Blob Store with HDInsight - Windows Azure tutorial" metaKeywords="hdinsight blob, asv, hdinsight asv, hdinsight azure, wasb, wasbs, hdinsight, hdinsight storage" metaDescription="How to use Windows Azure Blob Store with HDInsight" umbracoNaviHide="0" disqusComments="1" writer="sburgess" editor="mollybos" manager="paulettm" />


#Using Windows Azure Blob storage with HDInsight




Windows Azure HDInsight Service supports both Hadoop Distributed Files System (HDFS) and Windows Azure Blob storage for storing data. Windows Azure Blob storage is a robust, general purpose Windows Azure storage solution. Windows Azure Blob storage provides a full featured HDFS file system interface for Blob storage that provides a seamless experience to customers by enabling the full set of components in the Hadoop ecosystem to operate (by default) directly on the data managed by Blog Storage. Blob storage is not just a low cost solution. Storing data in Blob storage enables the HDInsight clusters used for computation to be safely deleted without losing user data. 

<div class="dev-callout"> 
<b>Note</b> 
<p>Most HDFS commands such as ls, copyFromLocal, mkdir etc. will still work as expected. Only the commands that are specific to the native HDFS implementation (which is referred to as DFS) such as fschk and dfsadmin will show different behavior on Windows Azure Blob storage.</p> 
</div>

For information on provision a HDInsight cluster, see [Getting Started with Windows Azure HDInsight Service][hdinsight-getting-started].

##In this article

* [The HDInsight service storage architecture](#architecture)
* [Benefits of Windows Azure Blob storage](#benefits)
* [Preparing container for Windows Azure Blob storage](#preparingblobstorage)
* [Addressing files in Windows Azure Blob storage](#addressing)
* [Next steps](#nextsteps)

##<a id="architecture"></a>The HDInsight service storage architecture
The following diagram provides an abstract view of the HDInsight Service's storage architecture:

![HDI.ASVArch](../Media/HDI.ASVArch.png "HDInsight Storage Architecture")
  
The HDInsight Service provides access to the distributed file system that is locally attached to the compute nodes. This file system can be accessed using the fully qualified URI. For example: 

	hdfs://<namenodehost>/<path>

In addition, HDInsight Service provides the ability to access data stored in Blob storage. The syntax to access Windows Azure Blob storage is:

	WASB[s]://[<container>@]<accountname>.blob.core.windows.net/<path>


Hadoop supports a notion of default file system. The default file system implies a default scheme and authority; it can also be used to resolve relative paths. During the HDInsight provision process, user must specify a Blob storage container used as the default file system. 


Other than the Blob storage container designated as the default file system, you can also access containers that reside in the same Windows Azure storage account or different Windows Azure storage accounts located on the same data center as the HDInsight cluster:

* **Container in the same storage account:** Because the account name and key are stored in the core-site.xml, you have full access to the files in the container.
* **Container in a different storage account with the *public container* or the *public blob* access level:** you have read only permission to the files in the container.

	<div class="dev-callout"> 
	<b>Note</b> 
	<p>Public Container allows you to get a list of all blobs available in that container and get container metadata. Public Blob allows  you to access the blobs only if you know the exact url. For more information, see <a href="http://msdn.microsoft.com/en-us/library/windowsazure/dd179354.aspx">Restrict Access to Containers and Blobs</a>.</p> 
	</div>

* **Container in a different storage account with the *private* access levels:** you must add those containers during the provision process.


Blob storage containers store data as key/value pairs, and there is no directory hierarchy. However the ‘/’ character can be used within the key name to make it appear as if a file is stored within a directory structure. For example, a blob’s key may be ‘input/log1.txt’. No actual ‘input’ directory exists, but due to the presence of the ‘/’ character in the key name, it has the appearance of a file path.










##<a id="benefits"></a>Benefits of Windows Azure Blob storage
The implied performance cost of not having compute and storage co-located is mitigated by the way the compute clusters are provisioned close to the storage account resources inside the Windows Azure data center, where the high speed network makes it very efficient for the compute nodes to access the data inside Blob storage.

There are several benefits associated with storing the data in Blob storage instead of HDFS:

* **Data reuse and sharing:** The data in HDFS is located inside the compute cluster. Only the applications that have access to the compute cluster can use the data using HDFS API. The data in Blob storage can be accessed either through the HDFS APIs or through the [Blob Storage REST APIs](http://msdn.microsoft.com/en-us/library/windowsazure/dd135733.aspx). Thus, a larger set of applications (including other HDInsight clusters) and tools can be used to produce and consume the data.
* **Data archiving:** Storing data in Blob storage enables the HDInsight clusters used for computation to be safely deleted without losing user data. 
* **Data storage cost:** Storing data in DFS for the long term will be more costly than storing the data in Blob storage since the cost of a compute cluster is higher than the cost of a Blob storage container. In addition, since the data does not have to be reloaded for every compute cluster generation, you are saving data loading costs as well.
* **Elastic scale-out:** While HDFS provides you with a scaled-out file system, the scale is determined by the number of nodes that you provision for your cluster. Changing the scale can become a more complicated process than relying on the Blob storage's elastic scaling capabilities that you get automatically.
* **Geo replication:** Your Blob storage containers can be geo replicated through the Azure Portal. While this gives you geographic recovery and data redundancy, a fail-over to the geo replicated location will severely impact your performance and may incur additional costs. So our recommendation is to choose the geo replication wisely and only if the value of the data is worth the additional costs.

Certain Map-Reduce jobs and packages may create intermediate results that you don't really want to store in the Blob storage container. In that case, you can still elect to store the data in the local HDFS file system. In fact, HDInsight uses DFS for several of these intermediate results in Hive jobs and other processes. 





##<a id="preparingblobstorage"></a>Preparing container for Windows Azure Blob storage
To use blobs, you first create a [Windows Azure storage account](/en-us/manage/services/storage/how-to-create-a-storage-account/). As part of this, you specify a Windows Azure data center that will store the objects you create using this account. This data center must also used to host the HDInsight cluster. Wherever it lives, each blob you create belongs to some container in your storage account. This container may be an artitary Blob storage container created outside of HDInsight, or it may be a container that is created for an HDInsight cluster. 



**Provision the container used as the default file system**

When provisioning an HDInsight cluster from Windows Azure Management Portal, there are two options: *quick create* and *custom create*. Using either of the options, a Windows Azure Storage account must be created beforehand.  For instructions, see [How to Create a Storage Account]( /en-us/manage/services/storage/how-to-create-a-storage-account/). 

Using the quick create option, you can choose an existing storage account. The provision process will create a new container with the same name as the HDInsight cluster name. This container will be used as the default file system.

![HDI.QuickCreate](../Media/HDI.QuickCreateCluster.png "HDInsight Cluster Quick Create")
 
Using the custom create, you can either choose an existing Blob storage container or create a new container to be provisioned as the default file system. The new container has the same name as the HDInsight cluster name.

![HDI.CustomCreateStorageAccount](../Media/HDI.CustomCreateStorageAccount.png "Custom Create Storage Account")





























**Using APIs to create containers**

Creating an HDInsight default file system can be done by creating a new Blob storage container through the commonly-used APIs in a storage account for which core-site.xml contains the storage key. In addition, you can also create a new container by referring to it in an HDFS file system command. For example:

	hadoop fs -mkdir wasbs://<newcontainer>@<accountname>.blob.core.windows.net/<newdirectory>

The example command will not only create the new directory *newdirectory* but, if it doesn't exist, will also create a new container called *newcontainer*.










##<a id="addressing"></a>Addressing files in Blob storage

The URI scheme for accessing files in Blob storage is: 

	WASB[S]://[<container>@]<accountname>.blob.core.windows.net/<path>

The URI scheme provides both unencrypted access with the WASB: prefix, and SSL encrypted access with WASBS. We recommend using WASBS wherever possible, even when accessing data that lives inside the same Windows Azure data center.
	
The &lt;container&gt; identifies the name of the Blob storage container. If no container name is specified but the domain is, then it refers to the [root container](http://msdn.microsoft.com/en-us/library/windowsazure/ee395424.aspx) of the domain's storage account. Note that root containers are read-only.
	
The &lt;accountname&gt; identifies the storage account name. Fully Qualified Domain Name (FQDN) is required.
	
If neither the container nor the accountname has been specified, then the default file system is used.
	
The &lt;path&gt; is the file or directory HDFS path name. Since Blob storage containers are just a key-value store, there is no true hierarchical file system. A / inside an Azure Blob's key is interpreted as a directory separator. Thus, if a blob's key is input/log1.txt, then it is the file log1.txt inside the directory input.

For example:

	wasbs://dailylogs@myaccount.blob.core.windows.net/input/log1.txt
	
refers to the file log1.txt in the directory input on the Blob storage container dailylogs at the location myaccount.blob.core.windows.net using SSL.
	
	wasbs://myaccount.blob.core.windows.net/result.txt
	
refers to the file result.txt on the read-only WASB file system in the root container at the location myaccount.blob.core.windows.net that gets accessed through SSL. Note that wasb://myaccount.blob.core.windows.net/output/result.txt will result in an exception, because Blob storage does not allow / inside path names in the root container to avoid ambiguities between paths and folder names. 
	
	wasb:///output/result.txt 
	
refers to the file result.txt in the output directory on the default file system.

You must specify the FQDN when using SSL. The following command will return an error:

	wasbs:///output/result.txt 

Instead, you must use the following command:

	wasbs://dailylogs@myaccount.blob.core.windows.net/output/result.txt 

Because HDInsight uses a Blob storage container as the default file system, you can refer to files and directories inside the default file system using relative or absolute paths. For example, the following statement will list all top-level directories and files of the default file system:

	hadoop fs -ls /output/result.txt

**Mapping a Blob storage URI to an WASB URI**

Given a Blob storage URI, you may need to be able to create the WASB URI. The mapping is straight forward. 


To access an file (or folder) at 

<table border="1">
<tr><th>Blob storage URI</th><th>WASB URI</th></tr>

<tr><td>http[s]://&lt;account&gt;/&lt;path-name&gt;</td><td>wasb[s]://&lt;account&gt;.blob.core.windows.net/&lt;path-name&gt;</td></tr>

<tr><td>http[s]://&lt;account&gt;/&lt;container&gt;/&lt;path-name&gt;</td><td>wasb[s]://&lt;container&gt;@&lt;account&gt;.blob.core.windows.net/<br />&lt;path-name&gt;</td></tr>

<tr><td>http[s]://&lt;account&gt;/&lt;container&gt;/&lt;path-name&gt;<br/>

where account and container are the values used for specifying the default file system.</td><td>wasb:///&lt;path-name&gt;<br/> wasbs://&lt;container&gt;@&lt;account&gt;.blob.core.windows.net/<br />&lt;path-name&gt;</td></tr>

<tr><td>https://&lt;account&gt;.blob.core.windows.net/<br />dailylogs/input/log1.txt</td><td>wasbs://&lt;container&gt;@&lt;account&gt;.blob.core.windows.net/<br />input/log1.txt</td></tr>

</table>

##<a id="nextsteps"></a>Next steps

In this article, you learned how to use Blob storage with HDInsight and that Blob storage is a fundamental component of the HDInsight Service. This will allow you to build scalable, long-term archiving data acquisition solutions with Windows Azure Blob storage and use HDInsight to unlock the information inside the stored data.

Now you understand how to use Windows Azure Blob storage. To learn more, see the following articles:

* [Getting Started with Windows Azure HDInsight Service][hdinsight-getting-started]
* [Upload data to HDInsight][hdinsight-upload-data]
* [Using Hive with HDInsight][hdinsight-hive]
* [Using Pig with HDInsight][hdinsight-pig]

[hdinsight-getting-started]: /en-us/manage/services/hdinsight/get-started-hdinsight/
[hdinsight-upload-data]: /en-us/manage/services/hdinsight/howto-upload-data-to-hdinsight/
[hdinsight-mapreduce]: /en-us/manage/services/hdinsight/using-mapreduce-with-hdinsight/
[hdinsight-hive]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
[hdinsight-pig]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
