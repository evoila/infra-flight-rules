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
