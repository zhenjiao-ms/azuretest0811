---
title: Job preparation and cleanup in Batch | Microsoft Azure
description: Employ job-level preparation tasks to minimize data transfer to Azure Batch compute nodes, and release tasks for node cleanup at job completion.
services: batch
documentationcenter: .net
author: mmacy
manager: timlt
editor: ''

ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: big-compute
ms.date: 06/22/2016
ms.author: marsma

---
# Run job preparation and completion tasks on Azure Batch compute nodes
Azure Batch jobs often require some form of setup prior to execution, as well as some sort of post-job maintenance after the job's tasks are completed. Batch provides the mechanisms for this preparation and maintenance in the form of optional job preparation and job release tasks.

Before any of a job's tasks run, the **job preparation task** runs on all compute nodes that are scheduled to run tasks. Once the job is completed, the **job release task** runs on each node in the pool that executed at least one task. As with normal Batch tasks, you can specify a job preparation or release task's command line to be invoked when that task is run. These special tasks offer other familiar task features such as file download, elevated execution, custom environment variables, maximum execution duration, retry count, and file retention time.

In the following sections, you'll find out how to use these two special task types by using the [JobPreparationTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx) class and [JobReleaseTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobreleasetask.aspx) class in the [Batch .NET](http://msdn.microsoft.com/library/azure/mt348682.aspx) API.

> [!TIP]
> Job preparation and release tasks are especially helpful in "shared pool" environments—those environments in which a pool of compute nodes persists between job runs, and is shared between many different jobs.
> 
> 

## When to use job preparation and release tasks
Any time you need to prepare nodes with job-specific configuration or data (and clean up or persist task result data) is a good time to use job preparation and release tasks. Examples of such situations are:

**Transfer of common task data**

Batch jobs often require a common set of data as input for the job's tasks. For example, in daily risk analysis calculations, market data is job specific, yet common to all of the tasks in the job. This market data, often several gigabytes in size, should be downloaded to each compute node only once so that any task that runs on a node can use it. Use a **job preparation task** to download the data to each node before the execution of the job's other tasks.

**Job data deletion**

Within a shared pool environment in which a pool's compute nodes are not decommissioned between jobs, you may need to delete job data between runs—to conserve disk space on the nodes, or perhaps to satisfy your organization's security policies. Use a **job release task** to delete data that was downloaded by a job preparation task or generated during task execution.

**Log retention**

You might want to keep a copy of log files that your tasks generate, or perhaps crash dump files that can be generated by failed applications. Use a **job release task** in such cases to compress and upload this data to an [Azure Storage](https://azure.microsoft.com/services/storage/) account.

## Job preparation task
Prior to the execution of a job's tasks, the job preparation task is executed on each compute node that is scheduled to run a task. By default, the Batch service will wait for the job preparation task to be completed before running the tasks scheduled to execute on the node. However, you can configure the service not to wait. The job preparation task will run again on a compute node if the node restarts, but you can also disable this behavior.

The job preparation task is only executed on nodes that are scheduled to run a task. This prevents the unnecessary execution of a preparation task in case a node is not assigned a task. This can occur when the number of tasks for a job is less than the number of nodes in a pool. It also applies when [concurrent task execution](batch-parallel-node-tasks.md) is enabled, which leaves some nodes idle if the task count is lower than the total possible concurrent tasks. By not running the job preparation task on idle nodes, you can spend less money on data transfer charges.

> [!NOTE]
> [JobPreparationTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx) differs from [CloudPool.StartTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx) in that JobPreparationTask executes at the start of each job, whereas StartTask executes only when a compute node first joins a pool or restarts.
> 
> 

## Job release task
Once a job is marked as completed, the job release task is executed on each node in the pool that executed at least one task. You mark a job as completed by issuing a terminate request. The Batch service then sets the job state to *terminating*, terminates any active or running tasks associated with the job, and runs the job release task. The job then moves to the *completed* state.

> [!NOTE]
> Job deletion also executes the job release task. However, if a job has already been terminated, the release task is not run a second time if that job is subsequently deleted.
> 
> 

## Job prep and release tasks with Batch .NET
To use a job preparation task, you create and configure a [JobPreparationTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx) object and assign it to your job's [CloudJob.JobPreparationTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx) property. Similarly, initialize [JobReleaseTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobreleasetask.aspx) and assign it to your job's [CloudJob.JobReleaseTask](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx) property to set the job's release task.

In this code snippet, `myBatchClient` is a fully initialized instance of [BatchClient](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx), and `myPool` is an existing pool within the Batch account.

        // Create the CloudJob for CloudPool "myPool"
        CloudJob myJob = myBatchClient.JobOperations.CreateJob("JobPrepReleaseSampleJob",
                                                               new PoolInformation() { PoolId = "myPool" });

        // Specify the command lines for the job preparation and release tasks
        string jobPrepCmdLine = "cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
        string jobReleaseCmdLine = "cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";

        // Assign the job preparation task to the job
        myJob.JobPreparationTask = new JobPreparationTask { CommandLine = jobPrepCmdLine };

        // Assign the job release task to the job
        myJob.JobReleaseTask = new JobPreparationTask { CommandLine = jobReleaseCmdLine };

        await myJob.CommitAsync();

As mentioned above, the release task is executed when a job is terminated or deleted. Terminating a job with the Batch .NET API is performed by calling [JobOperations.TerminateJobAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx). Job deletion is performed with [JobOperations.DeleteJobAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.deletejobasync.aspx). Both of these actions are typically done when a job's tasks are completed or when a timeout that you've defined has been reached.

        // Terminate the job to mark it as Completed; this will initiate the Job Release Task on any node
        // that executed job tasks. Note that the Job Release Task is also executed when a job is deleted,
        // thus you need not call Terminate if you typically delete your jobs upon task completion.
        await myBatchClient.JobOperations.TerminateJobAsync("JobPrepReleaseSampleJob");

## Code sample on GitHub
Check out the [JobPrepRelease](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease) sample project on GitHub to see job preparation and release tasks in action. This console application does the following:

1. Creates a pool with two "small" nodes.
2. Creates a job with job preparation, release, and standard tasks.
3. Runs the job preparation task, which first writes the node ID to a text file in a node's "shared" directory.
4. Runs a task on each node that writes its task ID to the same text file.
5. Once all tasks are completed (or the timeout is reached), prints the contents of each node's text file to the console.
6. When the job is completed, runs the job release task to delete the file from the node.
7. Prints the exit codes of the job preparation and release tasks for each node on which they executed.
8. Pauses execution to allow confirmation of job and/or pool deletion.

Output from the sample application is similar to the following:

```
Attempting to create pool: JobPrepReleaseSamplePool
Created pool JobPrepReleaseSamplePool with 2 small nodes
Checking for existing job JobPrepReleaseSampleJob...
Job JobPrepReleaseSampleJob not found, creating...
Submitting tasks and awaiting completion...
All tasks completed.

Contents of shared\job_prep_and_release.txt on tvm-2434664350_1-20160623t173951z:
-------------------------------------------
tvm-2434664350_1-20160623t173951z tasks:
  task001
  task004
  task005
  task006

Contents of shared\job_prep_and_release.txt on tvm-2434664350_2-20160623t173951z:
-------------------------------------------
tvm-2434664350_2-20160623t173951z tasks:
  task008
  task002
  task003
  task007

Waiting for job JobPrepReleaseSampleJob to reach state Completed
...

tvm-2434664350_1-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

tvm-2434664350_2-20160623t173951z:
  Prep task exit code:    0
  Release task exit code: 0

Delete job? [yes] no
yes
Delete pool? [yes] no
yes

Sample complete, hit ENTER to exit...
```

> [!NOTE]
> Due to the variable creation and start time of nodes in a new pool (some nodes are ready for tasks before others), you may see different output. Specifically, because the tasks complete quickly, one of the pool's nodes may execute all of the job's tasks. If this occurs, you will notice that the job prep and release tasks do not exist for the node that executed no tasks.
> 
> 

### Inspect job preparation and release tasks in the Azure Portal
When you run the above sample application, you can use the [Azure portal](https://portal.azure.com) to view the properties of the job and its tasks, or even download the shared text file that is modified by the job's tasks.

The screenshot below shows the **Preparation tasks blade** in the Azure portal after a run of the sample application. Navigate to the *JobPrepReleaseSampleJob* properties after your tasks have completed (but before deleting your job and pool) and click **Preparation tasks** or **Release tasks** to view their properties.

![Job preparation properties in Azure portal](./media/batch-job-prep-release/portal-jobprep-01.png)

## Next steps
### Application packages
In addition to the job preparation task, you can also use the [application packages](batch-application-packages.md) feature of Batch to prepare compute nodes for task execution. This feature is especially useful for deploying applications that do not require running an installer, applications that contain many (100+) files, or applications that require strict version control.

### Installing applications and staging data
Check out the [Installing applications and staging data on Batch compute nodes](https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch) post in the Azure Batch forum for an overview of the various methods of preparing your nodes for running tasks. Written by one of the Azure Batch team members, this post is a good primer on the different ways to get files (including both applications and task input data) onto your compute nodes, and some special considerations to take into account for each method.

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[azure_storage]: https://azure.microsoft.com/services/storage/
[portal]: https://portal.azure.com
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]:https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_job_prep_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_job_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.deletejobasync.aspx
[net_job_terminate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_job_release]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobreleasetask.aspx
[net_job_release_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx

[net_list_certs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx
[net_list_compute_nodes]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listcomputenodes.aspx
[net_list_job_schedules]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobschedules.aspx
[net_list_jobprep_status]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobpreparationandreleasetaskstatus.aspx
[net_list_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[net_list_nodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listnodefiles.aspx
[net_list_pools]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx
[net_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobs.aspx
[net_list_task_files]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[net_list_tasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx

[1]: ./media/batch-job-prep-release/portal-jobprep-01.png
