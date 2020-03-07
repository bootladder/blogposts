---
layout: post
title: 'Jenkins: Job DSL'
---
I don't know if it's my low-end VPS, java, or what, but Jenkins is **slow**.  
To create a new job, edit a job, really just about everything is slow.  
Right now I want to speed up:  creating a job, editing a job, viewing a job's definition.  
Obviously Job DSL is the answer.
  
[https://github.com/jenkinsci/job-dsl-plugin/wiki/Tutorial---Using-the-Jenkins-Job-DSL]
  
# Install JobDSL from Plugin Manager (this is obvious)
  
# Intro to Job DSL
First you manually create a job called the seed job.  In the build step,
instead of executing a script in the shell, select to run the Job DSL script.  
The Job DSL script is defining the new jobs that will be created as a result of
running this seed job.  
  
Run the seed job.  It generates jobs.  It does not run them, or check if they are defined sensibly.
Here is the console output.
```
Started by user engineering
Building in workspace /var/jenkins_home/workspace/job-dsl-1
Processing provided DSL script
Added items:
    GeneratedJob{name='DSL-Tutorial-1-Test'}
Finished: SUCCESS
```
Now go to the homepage of the seed job.  Notice it shows "Generated Items:" with the generated job.  
Now go to the homepage of Jenkins.  It shows the generated job, the seed job, and the existing manually created jobs.
  
# Huh... This does not help me!
Let's think this through.  What do I need?
* A Single Point of Truth (SPOT) for viewing and editing a job
* Atleast a file on the jenkins machine for the above.  Optional/better: git repo
* The SPOT must contain:
  * remote URL of git repo
  * Bitbucket, Github, Slack webhooks 
  * shell snippet for processing the output of a build
  
Looks like all the above is supported.  
What I don't like is:
* To edit job definitions, must edit the seed job
  * To solve this, I'll need to put the Job DSL script in the filesystem
* After editing job definitions, must re-generate the jobs
  * With the Job DSL script in the filesystem, I'll be editing
    the job in the filesystem, maybe without jenkins open.
    I can put a timer on the job trigger.  I should be able
    to stick a shell script in there which I can execute after editing,
    which should run the job to generate all the jobs.
  
OK, in the Job Configuration, change `Use the provided DSL script` to `Look on Filesystem`.  
It says the scripts must be located in the Workspace.  But wildcards `*` can be used.  
My Jenkins is in a docker container, but that should be OK since all the workspaces are
volume mapped.  
  
OK, I put a Job DSL script in there.  I then manually build the Seed Job and it fails.  
`ERROR: script not yet approved for use`  
OK, go to `Manage Jenkins --> In-process Script Approval`, and it's there.  
  
Cool!  Works as expected, including the deleting of the old job which I checked the box for.
```
Started by user engineering
Building in workspace /var/jenkins_home/workspace/job-dsl-1
Processing DSL script filesystemjob.groovy
Added items:
    GeneratedJob{name='Job-DSL-Filesystem-Defined'}
Unreferenced items:
    GeneratedJob{name='DSL-Tutorial-1-Test'}
Removed items:
    GeneratedJob{name='DSL-Tutorial-1-Test'}
Finished: SUCCESS
```
I'll have to figure out how to automate thru that security.  That's OK for now.  
  
# Using a git Repo
Created a repo, added the seedjob.groovy.  
**Note:  Jenkins needs the password to a private repo**  
Manually in Jenkins, add the job git repo with the groovy scripts.
This way, when building the seed job, the groovy for the jobs
come from git.
  
**OK never mind, this security thing is annoying**  
In the Seed Job Config, select `Use Groovy Sandbox`  
Install the `Authorize Project` plugin.  
Then a widget appears in Manage Jenkins --> Configure Global Security.  
I selected Run as User who triggered the build.  
Nope!  Must select `Run as Specific User`, and it's the jenkins user, not the system user
  
**I will need Bitbucket Webhook for the Seed Job**  
Unfortunately have to do this manually.  But I need it
so I can change the definition of a generated job.
  
**WTF?  Can't put dashes `-` in groovy filenames?**
  
# Result:  Recreated an Existing Job Configuration with Job DSL
```
job('ffn-loader-atmelice5898') {
    scm {
        git {
            remote {
                url('https://bitbucket.org/bootladder/debos_firmware.git');
            }
            branch('master');
            extensions {
                localBranch 'master'
            }
        }
    }
    triggers {
        bitbucketPush()
    }
    steps {
        shell('echo "Hello, world this is generated job dsl!"');
        shell('\
            ./build.sh && cd src/cmakebuild/bin/ && \
            pathtonewelf=$(ls *.elf) && \
            cp *.elf /var/jenkins_publish/ && \
            cd /var/jenkins_publish && \
            ln -f $pathtonewelf latest-debos_firmware && \
            echo $pathtonewelf > target/ffnbeagle-woodland-atmelice-J41800075898 \
        ')
    }
}
```
**What this does:**
* url:  use https:// , it seemed like git:// was not working Bitbucket push triggers
* localBranch 'master' does a local checkout of master branch
* bitbucketPush() triggers a build on the Bitbucket webhook
  
**The Shell bit**  
Builds the ELF, copies it to the publish directory
and echos the filename to a pointer file.  
The pointer file is curl'ed by a remote host in the lab.  
The pointer file says what firmware should be loaded on a particular target.  
You can see I'm identifying the target by `system-location-tool-serialnumber`
  
# For future reference: http://job-dsl.herokuapp.com/  

