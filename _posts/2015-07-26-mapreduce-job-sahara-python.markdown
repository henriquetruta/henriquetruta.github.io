---
layout: post
title: Launching a MapReduce Job in Sahara using python
date: '2015-07-26 18:22:07'
tags:
- sahara
- openstack
- big-data
- hadoop
- python
---

Sahara is OpenStack's Big Data as a Service solution, that provides an easy interface to create clusters and run jobs in platforms like Hadoop, Spark and Storm. Although most of its functionalities are available in OpenStack Dashboard (Horizon), it might be useful for a user to run jobs in an automated way, in cases when the user needs repetition or to schedule the execution to a given time.

We'll show how to run a Wordcount Hadoop job through Sahara python API.

## Before you start...

This tutorial assumes you have:

* A running Hadoop Cluster. A tutorial to launch one is available [here](http://docs.openstack.org/developer/sahara/devref/quickstart.html).
* A Wordcount MapReduce Job Binary uploaded and a Job Template Created. The binary is available [here](https://github.com/openstack/sahara/tree/master/etc/edp-examples/edp-mapreduce) and the tutorial to create a template [here](http://docs.openstack.org/developer/sahara/horizon/dashboard.user.guide.html#job-binaries) 
* A valid input Data Source, uploaded in Swift. Tutorial [here](http://docs.openstack.org/developer/sahara/horizon/dashboard.user.guide.html#data-sources)

## python-saharaclient
Each OpenStack service has its own python client, that provides a Python API and a Command Line Interface. We'll need to have [python-saharaclient](https://github.com/openstack/python-saharaclient) client installed. 

We recommend that you install the Master branch version, by doing:
```
$ git clone https://github.com/openstack/python-saharaclient.git
$ cd python-saharaclient
$ python setup.py install
```

To use the command-line interface, you must export environment variables, in order to get the client authenticated. The easiest way to do that is through Horizon. Once you're logged in, just go in Project > Compute > Access & Security > API Access > Download OpenStack RC File. Then, in the terminal, enter to the directory you've download the file and type:

```
$ source <file.sh>
```
Type your password after prompted and you're good to go. You can test if your installation and credentials were successful by typing:

```
$ sahara job-list
$ sahara data-source-list
$ sahara cluster-list
$ sahara help
```
## Getting it done
First, you must import some stuff and define the job and credentials variables. We are not using args, here. Just in-line code.
``` python
# Auth Data                                                                 
username = 'admin'                                                       
project_name = 'admin'
project_id = 'bdbc102bcc8e41569b9a0cb8119ddd73'                                        
password = "SuperSecret"                                                      
auth_url = "http://keystone_url:5000/v2.0"                                     
sahara_url = "http://sahara_url:8386/v1.1/%s" % project_id 
# Job attributes                                                            
cluster_id = "814b6691-ea64-4898-98d4-052c1ef16456" # Get it at 'sahara cluster-list'
job_id = "50e8d9b6-cf8c-41a2-9f73-808c4cef97ef" # 'sahara job-template-list'            
container_out_name = "wordcount" # A valid swift container                                           
input_ds_id = "2817b803-e8d9-480d-93d7-8710c62eabb3" # Input data source. Get it by 'sahara data-source-list'                      
exec_date = datetime.now().strftime('%Y%m%d_%H%M%S') # We're using timestamp as the output name              
output ="output_%s" % exec_date                                             
```
You're done with configuration. The next step is instantiate python-saharaclient.

```
# Client Instantiating
sahara = saharaclient(auth_url=auth_url,
                      sahara_url=sahara_url,
                      username=username,
                      api_key=password,
                      project_name=project_name)
```

From now on, this `sahara` object will be used to access Sahara API in pythoned way (which is much more user friendly than doing REST calls).

Let's now create the output data source.
```
data_source_out = sahara.data_sources.create("mydatasource_"+output,
                                             "This is a test data source",
                                             "swift",
                                             container_url,
                                             credential_user=username,
                                             credential_pass=password)
# We can check whether the job was correctly created
print data_source_out
```

Finally, creating the job configs dict and launching the job:
```
job_configs = {
    "configs" : {"mapred.mapper.class": "org.myorg.WordCount$Map",
                 "mapred.reducer.class": "org.myorg.WordCount$Reduce",
                 "mapred.mapoutput.key.class": "org.apache.hadoop.io.Text",
                 "mapred.mapoutput.value.class": "org.apache.hadoop.io.IntWritable"},
    "args": [],
    "params": {}}

job = sahara.job_executions.create(job_id,
                                   cluster_id,
                                   input_ds_id,
                                   output_ds_id,
                                   job_configs)
```

And, if everything went ok, you must have you job running at this point.

## Got an error?
You can always check your job status through `sahara job-list` or through Horizon. However, they are not very good at properly showing the error message. Better ways to seek the error are:

* `sahara --debug job-show --id <job_id>` and look for some error messages in the raw json that will be shown;
* See the hadoop job tracker or Oozie Web UI. The URLs are available at cluster's page in Horizon or in the CLI through `sahara cluster-show --id <cluster_id>`;
* If you have access to the node Sahara is running on, check the log, that's usually placed at /var/log/sahara.

The full code shown in this example is available [here](https://www.dropbox.com/s/ylq6ddb3yoh21gw/run_job.py?dl=0), and also includes some helper methods that track the job status, until it's finished or failed.

Did this work for you? If so, great. If it didn't, you can email :)

######Helpful links:
http://docs.openstack.org/developer/python-saharaclient/api.html#job-execution-ops
http://docs.openstack.org/developer/sahara/restapi/rest\_api\_v1.1\_EDP.html#execute-job
http://docs.openstack.org/developer/sahara/userdoc/edp.html



