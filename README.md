# Jenkins API Client for Go

[![GoDoc](https://godoc.org/github.com/daixijun/jenkins?status.svg)](https://godoc.org/github.com/daixijun/jenkins)
[![Go Report Cart](https://goreportcard.com/badge/github.com/daixijun/jenkins)](https://goreportcard.com/report/github.com/daixijun/jenkins)
[![Build Status](https://travis-ci.org/daixijun/jenkins.svg?branch=master)](https://travis-ci.org/daixijun/jenkins)

## About

Jenkins is the most popular Open Source Continuous Integration system. This Library will help you interact with Jenkins in a more developer-friendly way.

These are some of the features that are currently implemented:

* Get information on test-results of completed/failed build
* Ability to query Nodes, and manipulate them. Start, Stop, set Offline.
* Ability to query Jobs, and manipulate them.
* Get Plugins, Builds, Artifacts, Fingerprints
* Validate Fingerprints of Artifacts
* Get Current Queue, Cancel Tasks
* etc. For all methods go to GoDoc Reference.

## Installation

    go get github.com/daixijun/jenkins

## Usage

```go
import "github.com/daixijun/jenkins"

j, err := jenkins.NewJenkins("http://localhost:8080/", "admin", "admin")
// Provide CA certificate if server is using self-signed certificate
// caCert, _ := ioutil.ReadFile("/tmp/ca.crt")
// j.Requester.CACert = caCert


if err != nil {
  panic("Something Went Wrong")
}

build, err := j.GetJob("job_name")
if err != nil {
  panic("Job Does Not Exist")
}

lastSuccessBuild, err := build.GetLastSuccessfulBuild()
if err != nil {
  panic("Last SuccessBuild does not exist")
}

duration := lastSuccessBuild.GetDuration()

job, err := j.GetJob("jobname")

if err != nil {
  panic("Job does not exist")
}

job.Rename("SomeotherJobName")

configString := `<?xml version='1.0' encoding='UTF-8'?>
<project>
  <actions/>
  <description></description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers class="vector"/>
  <concurrentBuild>false</concurrentBuild>
  <builders/>
  <publishers/>
  <buildWrappers/>
</project>`

j.CreateJob(configString, "someNewJobsName")


```

API Reference: https://godoc.org/github.com/daixijun/jenkins

## Examples

For all of the examples below first create a jenkins object
```go
import "github.com/daixijun/jenkins"

j, _ := jenkins.NewJenkins("http://localhost:8080/", "admin", "admin")
```

or if you don't need authentication:

```go
j, _ := jenkins.NewJenkins("http://localhost:8080/", "", "")
```


### Check Status of all nodes

```go
nodes := j.GetAllNodes()

for _, node := range nodes {

  // Fetch Node Data
  node.Poll()
	if node.IsOnline() {
		fmt.Println("Node is Online")
	}
}

```

### Get all Builds for specific Job, and check their status

```go
jobName := "someJob"
builds, err := j.GetAllBuildIds(jobName)

if err != nil {
  panic(err)
}

for _, build := range builds {
  buildId := build.Number
  data, err := j.GetBuild(jobName, buildId)

  if err != nil {
    panic(err)
  }

	if "SUCCESS" == data.GetResult() {
		fmt.Println("This build succeeded")
	}
}

// Get Last Successful/Failed/Stable Build for a Job
job, err := j.GetJob("someJob")

if err != nil {
  panic(err)
}

job.GetLastSuccessfulBuild()
job.GetLastStableBuild()

```

### Get Current Tasks in Queue, and the reason why they're in the queue

```go

tasks := jenkins.GetQueue()

for _, task := range tasks {
	fmt.Println(task.GetWhy())
}

```

### Create View and add Jobs to it

```go

view, err := j.CreateView("test_view", jenkins.LIST_VIEW)

if err != nil {
  panic(err)
}

status, err := view.AddJob("jobName")

if status != nil {
  fmt.Println("Job has been added to view")
}

```

### Create nested Folders and create Jobs in them

```go

// Create parent folder
pFolder, err := j.CreateFolder("parentFolder")
if err != nil {
  panic(err)
}

// Create child folder in parent folder
cFolder, err := j.CreateFolder("childFolder", pFolder.GetName())
if err != nil {
  panic(err)
}

// Create job in child folder
configString := `<?xml version='1.0' encoding='UTF-8'?>
<project>
  <actions/>
  <description></description>
  <keepDependencies>false</keepDependencies>
  <properties/>
  <scm class="hudson.scm.NullSCM"/>
  <canRoam>true</canRoam>
  <disabled>false</disabled>
  <blockBuildWhenDownstreamBuilding>false</blockBuildWhenDownstreamBuilding>
  <blockBuildWhenUpstreamBuilding>false</blockBuildWhenUpstreamBuilding>
  <triggers class="vector"/>
  <concurrentBuild>false</concurrentBuild>
  <builders/>
  <publishers/>
  <buildWrappers/>
</project>`

job, err := j.CreateJobInFolder(configString, "jobInFolder", pFolder.GetName(), cFolder.GetName())
if err != nil {
  panic(err)
}

if job != nil {
	fmt.Println("Job has been created in child folder")
}

```

### Get All Artifacts for a Build and Save them to a folder

```go

job, _ := j.GetJob("job")
build, _ := job.GetBuild(1)
artifacts := build.GetArtifacts()

for _, a := range artifacts {
	a.SaveToDir("/tmp")
}

```

### To always get fresh data use the .Poll() method

```go

job, _ := j.GetJob("job")
job.Poll()

build, _ := job.getBuild(1)
build.Poll()

```

## Testing

    go test

## Contribute

All Contributions are welcome. The todo list is on the bottom of this README. Feel free to send a pull request.

## TODO

Although the basic features are implemented there are many optional features that are on the todo list.

* Kerberos Authentication
* CLI Tool
* Rewrite some (all?) iterators with channels

## LICENSE

Apache License 2.0
