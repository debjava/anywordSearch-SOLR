![A22.png](https://s19.postimg.org/5hb03dwkj/A22.png) How to Search anything in SOLR
==============

Introduction
============
We know that **SOLR** is blazingly fast enterprise search platform. It provides many facilities to retrieve the data from solr index. However we may encounter a very common business requirements that “how to search anything from SOLR”. Interestingly we know how to search for a particular data from SOLR console, but this requirement is somewhat like Google type search ie. Search anything. In this post I will show you how to achieve this.

Technology Stack
================
The following framework/s and tool/s have been used in this current sample application.

<table border="1">
  <tr>
    <th>Name</th>
    <th>Version</th> 
  </tr>
  <tr>
    <td>Java</td>
    <td>1.8</td> 
  </tr>
  <tr>
    <td>SOLR</td>
    <td>6.1</td> 
  </tr>
  <tr>
    <td>Firefox REST Client Addon</td>
    <td>2.0.5</td> 
  </tr>
</table>


What does it do?
===============
In this small tutorial project, we will perform the following tasks.

* **Create a SOLR Core.**
* **Create some data specific to Employees working in a project.**
* **Push some data in SOLR Core for indexing using REST Calls.**
* **Search any keyword which is present in any of the fields.**
* **Delete data from SOLR index using REST calls.**

# Project Structure
The basic structure for SOLR core is given below.
The name of the solr core is **anywordSearch**.

![SOLR-Anykeyword-Search-proj-View-1.png](https://s19.postimg.org/v6lmx8en7/SOLR_Anykeyword_Search_proj_View_1.png)

In this directory structure, the following three files are important to create a solr core.

1. **managed-schema** (This is the actual schema for SOLR data to be indexed).
2. **solrconfig.xml** (This is the internal configuration file used for SOLR, we will modify as per the requirement)
3. **elevate.xml** (A minor modification is required to avoid error **Error initializing QueryElevationComponent**)
4. **core.properties** (Information to SOLR about the configuration)

Build and Installation
===
Before we proceed for setup a core in SOLR, I assume that you have at least some exposure to SOLR.
Let us see what kind of data we are going to store in SOLR Index system.
We are going to store some employee information as given below. The json structure is given below.

		{  
   			"empId":4,
   			"firstName":"John",
   			"lastName":"Abraham",
   			"designation":"QA Lead",
   			"location":"Bangalore",
   			"projectName":"Media and Entertainment"
		}

The objective of this kind of data is to find the details specific to firstName, lastName,designation, location etc.
The requirement will be something like "Find out all the employees working in Bangalore". So in this case, we will use our keyword as Bangalore only in SOLR search and let SOLR search from its indexing system.

## Files Setup ##
We need to modify few files to achieve this feature.

1). **managed-schema**

Add the following the contents in the file.

```xml

	 <!-- Provide below all the field to field mappings -->
    <field name="firstName" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="lastName" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="designation" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="location" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="projectName" type="string" indexed="true" stored="true" multiValued="false"/>
    
    <!-- For all kinds of keyword search -->
    <field name="allText_s" type="text_general" indexed="true" stored="false" multiValued="true"/>
    
    <!-- Copy all the required fields to specific field to search anything --> 
    <copyField source="firstName" dest="allText_s"/>
	 <copyField source="lastName" dest="allText_s"/>
	 <copyField source="designation" dest="allText_s"/>
	 <copyField source="location" dest="allText_s"/>
	 <copyField source="projectName" dest="allText_s"/>
    
```
In the above case, we have created a dummy field called **"allText_s"** to keep track of all kinds of keywords from various fields mentioned in the schema.

2). **solrconfig.xml**

Add or Modify the following lines in the above mentioned file.

```xml
			<requestHandler name="/select" class="solr.SearchHandler">
			     <lst name="defaults">
			       <str name="echoParams">explicit</str>
			       <int name="rows">10</int>
			       <str name="df">allText_s</str>
			     </lst>    
			</requestHandler>
```

This above code will provide information about what to search and it instructructs SOLR that search any word stored in the field **"allText_s"** and retrieve the data accordingly.

3). **elevate.xml**

Make sure that the file looks like the below given to avoid unnecessary SOLR Exception.

```xml
			<elevate>
				 <query text="foo bar">
				  <!-- <doc id="1" /> -->
				  <!-- <doc id="2" /> -->
				  <!-- <doc id="3" /> -->
				 </query>
				 
				 <!-- <query text="ipod"> -->
				   <!-- <doc id="MA147LL/A" />   -->
				   <!-- <doc id="IW-02" exclude="true" />  -->
				 <!-- </query> -->
				 
			</elevate>
```

3). **core.properties**

Add the following information to this properties file.

			name=anywordSearch
			config=solrconfig.xml
			schema=schema.xml
			dataDir=data

## Execution ##
Afte all modifications, follow the steps.

1). Copy the entire directory **anywordSearch** to **SOLR_HOME/server/solr**.
2). Now go to bin directory of SOLR_HOME in command prompt and start the SOLR using the following command.

		solr start -f (To start in foreground mode, good and easy for debugging)
		                or
		solr start (To start in background mode)

> **Note:**
There are commands to start solr, refer to actual SOLR wiki for more details

## Data population ##
Once we start the SOLR server, browse the url [http://localhost:8983/solr/#/](http://localhost:8983/solr/#/), you will be able to see the core as **anywordSearch** in section Core Selector in SOLR Admin console.

SOLR provides REST calls to populate the data in its indexing system.

Let us populate some data now.
Open your Firefox rest client and provide the following information to make REST call.

**HTTP Method** : POST

**URL** : [http://localhost:8983/solr/anywordSearch/update?commit=true](http://localhost:8983/solr/anywordSearch/update?commit=true)

**Header** : Content-Type=application/json

**Request Body**

			[
				{  
				   "empId":1,
				   "firstName":"Deb",
				   "lastName":"Mishra",
				   "designation":"Technical Architect",
				   "location":"Bangalore",
				   "projectName":"Complex Processing System"
				},
				{  
				   "empId":2,
				   "firstName":"Sambit",
				   "lastName":"Mishra",
				   "designation":"Technical Manager",
				   "location":"Chennai",
				   "projectName":"Payroll Management System"
				},
				{  
				   "empId":3,
				   "firstName":"Rajesh",
				   "lastName":"Padhi",
				   "designation":"Project Lead",
				   "location":"Phoenix",
				   "projectName":"Digital System"
				},
				{  
				   "empId":4,
				   "firstName":"John",
				   "lastName":"Abraham",
				   "designation":"QA Lead",
				   "location":"Bangalore",
				   "projectName":"Media and Entertainment"
				},
				{  
				   "empId":5,
				   "firstName":"Vidya",
				   "lastName":"Balan",
				   "designation":"Technical Architect",
				   "location":"Mumbai",
				   "projectName":"Media and Entertainment"
				}
			]
			

Now go back to SOLR Admin console, click on query and search everything. Now you will be able to see all the data what we posted just now.

Now search for a particular keyword like **Bangalore** and you will be happy to see the required data.

For convenience , I provide below about how to delete all data from SOLR index.

**HTTP Method** : POST

**URL** : [http://localhost:8983/solr/anywordSearch/update?commit=true](http://localhost:8983/solr/anywordSearch/update?commit=true)

**Header** : Content-Type=application/xml

**Request Body**

		<delete><query>*:*</query></delete>
		
For simplicity, I provide below the complete url to search any kind of word.

[http://localhost:8983/solr/anywordSearch/select?indent=on&q=bangalore&wt=json](http://localhost:8983/solr/anywordSearch/select?indent=on&q=bangalore&wt=json)		


Bugs and Feedback
=================
This is a simple sample project about searching any kind of word in SOLR.
There may be some bugs or error in documentation, please report to me at debadatta.mishra@gmail.com

Further Reading and References
==============================

[SOLR Tutorials](https://lucene.apache.org/solr/resources.html#tutorials)

[SOLR Offcial WIKI](https://wiki.apache.org/solr)

Contributor
====
@Author : **Debadatta Mishra**

Conclusion
==========
Hope you have enjoyed my post, try to learn and explore more and share with all.