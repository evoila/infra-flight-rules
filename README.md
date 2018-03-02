# Bosh flight rules 
## What are "flight rules"?

A guide for astronauts  about what to do when things go wrong.

>Flight Rules are the hard-earned body of knowledge recorded in manuals that list, step-by-step, what to do if X occurs, and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures. [...]

— Chris Hadfield, An Astronaut's Guide to Life.

## If something goes wrong

### With Blobs

#### Cannot find blob named XXX
	 Building a release from directory '/Users/yremmet/DEV/bosh/mongodb-bosh-release':
	 Cannot find blob named 'create-replset/6ff7a835c431e94d47476107f8c93c2d5c27ed62' with SHA1 '0692dd85bc1486d8b1b1d076f1a43911de02aa8f'

	 Exit code 1

When creating a new release, this is propably due to some sha mismatch after you recated your bosh environment.
Delete `.dev_builds` and try again.

#### Missing Blob
	  - Constructing packages from directory:
      - Reading package from '/workspace/releases/osb-kafka-deployment/packages/zookeeper':
          Collecting package files:
            Missing files for pattern 'zookeeper/zookeeper-3.4.10.tar.gz'
You propably forgot a step when adding the new blob. 

1. `bosh add-blob`   
To add blob to local blob store.
2. `bosh upload-blobs`   
To upload local blobs to the remote blob store
3. `git commit`
To add the blob ids to 

### With deployments

#### Job not found in template table
	L Error: Job 'node_exporter' not found in Template table
This often occurs when an incorrect version of the release is deployed. Check the if the version you are deploing contains the mentioned job.   
If your deployment is pointing to the version `latest`, make sure the director doesn't have a higher version of your release than you create.

### Colocated job is already added to instance group
	L Error: Colocated job ‘node_exporter’ is already added to the instance group ‘postgres’.
Check the runtime configuration. The mentioned job will run on all VMs.

### Bad file descriptor
Trying to upload release and getting: 

```shell
Uploading release file:
  Performing request POST 'https://172.16.106.4:25555/releases?':
    Performing POST request:
      Seeking to beginning of seekable request body during attempt 0: seek /root/.bosh/tmp/bosh-platform-disk-TarballCompressor-CompressSpecificFilesInDir232448384: bad file descriptor
```

Actual problem: environment variables for Bosh director not set:

```shell
Exit code 1
root@host:/workspace/releases/osb-bosh-postgresql# bosh vms
Expected non-empty Director URL

Exit code 1
Check the runtime configuration. The mentioned job will run on all VMs
```
Solution: login

### Credhub cannot generate password
```shell
Preparing deployment: Preparing deployment (00:00:01)
                   L Error: Config Server failed to generate value for '/Bosh Lite Director/kubo/kubo-admin-password' with type 'password'. Error: ''
Task 15 | 21:30:09 | Error: Config Server failed to generate value for '/Bosh Lite Director/kubo/kubo-admin-password' with type 'password'. Error: ''
````

It turns out that the path for credhub is not allowed to contain spaces:

````shell
horst:kubo-deployment jhiemer$ credhub generate -n '/Bosh Lite Director/kubo/kubo-admin-password' -t 'password'
Credential names may only include alpha, numeric, hyphen, underscore, and forward-slash characters. Please update and retry your request.
````
Works:

````shell
horst:kubo-deployment jhiemer$ credhub generate -n '/BoshLiteDirector/kubo/kubo-admin-password' -t 'password'
id: f829e692-d98e-400e-8b14-9ac20d558c81
name: /BoshLiteDirector/kubo/kubo-admin-password
type: password
value: *******************
version_created_at: 2017-12-30T21:50:33Z
````
### Script not working (e.g. pre-start script)
```shell
Updating instance my-instances: my-instances/0d08eec9-9d4e-4507-b4fe-ba0b7b20f628 (0) (canary) (00:00:14)
L Error: Action Failed get_task: Task 3a65c031-3bd6-4cae-44cc-ab7cdc58e871 result: 1 of 2 pre-start scripts failed. Failed Jobs: my-cool-job. Successful Jobs: bosh-dns.
````

In most cases, this means you missed to switch to bash at the start of your script. Add the following to the beginning of your script and all works out well...

```shell
#!/usr/bin/env bash
````
