# Quick Start

1. Job config file
    
    Prepare a job config file as described [here](../user/training.md), for example, `exampleJob.json`.

2. Authentication
    
    HTTP POST your username and password to get an access token from:

        http://restserver/api/v1/token
        ```
        For example, with [curl](https://curl.haxx.se/), you can execute below command line:
        ```sh
        curl -H "Content-Type: application/x-www-form-urlencoded" \
             -X POST http://restserver/api/v1/token \
             -d "username=YOUR_USERNAME" -d "password=YOUR_PASSWORD"
        ```
    
    3. Submit a job
    
        HTTP POST the config file as json with access token in header to:
        ```
        http://restserver/api/v1/user/:username/jobs
        ```
        For example, you can execute below command line:
        ```sh
        curl -H "Content-Type: application/json" \
             -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
             -X POST http://restserver/api/v1/user/:username/jobs \
             -d @exampleJob.json
        ```
    
    4. Monitor the job
    
        Check the list of jobs at:
        ```
        http://restserver/api/v1/jobs
        ```
        or
        ```
        http://restserver/api/v1/user/:username/jobs
        ```
        Check your exampleJob status at:
        ```
        http://restserver/api/v1/user/:username/jobs/exampleJob
        ```
        Get the job config JSON content:
        ```
        http://restserver/api/v1/user/:username/jobs/exampleJob/config
        ```
        Get the job's SSH info:
        ```
        http://restserver/api/v1/user/:username/jobs/exampleJob/ssh
        ```
    
    # RestAPI
    
    ## Root URI
    
    Configure the rest server port in [services-configuration.yaml](../../../examples/cluster-configuration/services-configuration.yaml).
    
    ## API Details
    
    ### `POST token`
    
    Authenticated and get an access token in the system.
    
    *Request*
    

POST /api/v1/token

    <br />*Parameters*
    

{ "username": "your username", "password": "your password", "expiration": "expiration time in seconds" }

    <br />*Response if succeeded*
    

Status: 200

{ "token": "your access token", "user": "username", "admin": true if user is admin }

    <br />*Response if user does not exist*
    

Status: 400

{ "code": "NoUserError", "message": "User $username is not found." }

    <br />*Response if password is incorrect*
    

Status: 400

{ "code": "IncorrectPassworkError", "message": "Password is incorrect." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `PUT user`
    
    Update a user in the system.
    Administrator can add user or change other user's password; user can change his own password.
    
    *Request*
    

PUT /api/v1/user Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "username": "username in [_A-Za-z0-9]+ format", "password": "password at least 6 characters", "admin": true | false, "modify": true | false }

    <br />*Response if succeeded*
    

Status: 201

{ "message": "update successfully" }

    <br />*Response if not authorized*
    

Status: 401

{ "code": "UnauthorizedUserError", "message": "Guest is not allowed to do this operation." }

    <br />*Response if current user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if updated user does not exist*
    

Status: 404

{ "code": "NoUserError", "message": "User $username is not found." }

    <br />*Response if created user has a duplicate name*
    

Status: 409

{ "code": "ConflictUserError", "message": "User name $username already exists." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `DELETE user` (administrator only)
    
    Remove a user in the system.
    
    *Request*
    

DELETE /api/v1/user Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "username": "username to be removed" }

    <br />*Response if succeeded*
    

Status: 200

{ "message": "remove successfully" }

    <br />*Response if not authorized*
    

Status: 401

{ "code": "UnauthorizedUserError", "message": "Guest is not allowed to do this operation." }

    <br />*Response if user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if an admin will be removed*
    

Status: 403

{ "code": "RemoveAdminError", "message": "Admin $username is not allowed to remove." }

    <br />*Response if updated user does not exist*
    

Status: 404

{ "code": "NoUserError", "message": "User $username is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `PUT user/:username/virtualClusters` (administrator only)
    
    Administrators can update user's virtual cluster. Administrators can access all virtual clusters, all users can access default virtual cluster.
    
    *Request*
    

PUT /api/v1/user/:username/virtualClusters Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "virtualClusters": "virtual cluster list separated by commas (e.g. vc1,vc2)" }

    <br />*Response if succeeded*
    

Status: 201

{ "message": "update user virtual clusters successfully" }

    <br />*Response if the virtual cluster does not exist.*
    

Status: 400

{ "code": "NoVirtualClusterError", "message": "Virtual cluster $vcname is not found." }

    <br />*Response if not authorized*
    

Status: 401

{ "code": "UnauthorizedUserError", "message": "Guest is not allowed to do this operation." }

    <br />*Response if user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if user does not exist.*
    

Status: 404

{ "code": "NoUserError", "message": "User $username is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET jobs`
    
    Get the list of jobs.
    
    *Request*
    

GET /api/v1/jobs

    <br />*Parameters*
    

{ "username": "filter jobs with username" }

    <br />*Response if succeeded*
    

Status: 200

{ [ ... ] }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET user/:username/jobs`
    
    Get the list of jobs of user.
    
    *Request*
    

GET /api/v1/user/:username/jobs

    <br />*Response if succeeded*
    

Status: 200

{ [ ... ] }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET user/:username/jobs/:jobName`
    
    Get job status in the system.
    
    *Request*
    

GET /api/v1/user/:username/jobs/:jobName

    <br />*Response if succeeded*
    

Status: 200

{ name: "jobName", jobStatus: { username: "username", virtualCluster: "virtualCluster", state: "jobState", // raw frameworkState from frameworklauncher subState: "frameworkState", createdTime: "createdTimestamp", completedTime: "completedTimestamp", executionType: "executionType", // Sum of succeededRetriedCount, transientNormalRetriedCount, // transientConflictRetriedCount, nonTransientRetriedCount, // and unKnownRetriedCount retries: retriedCount, appId: "applicationId", appProgress: "applicationProgress", appTrackingUrl: "applicationTrackingUrl", appLaunchedTime: "applicationLaunchedTimestamp", appCompletedTime: "applicationCompletedTimestamp", appExitCode: applicationExitCode, appExitDiagnostics: "applicationExitDiagnostics" appExitType: "applicationExitType" }, taskRoles: { // Name-details map "taskRoleName": { taskRoleStatus: { name: "taskRoleName" }, taskStatuses: { taskIndex: taskIndex, containerId: "containerId", containerIp: "containerIp", containerPorts: { // Protocol-port map "protocol": "portNumber" }, containerGpus: containerGpus, containerLog: containerLogHttpAddress, } }, ... } }

    <br />*Response if the job does not exist*
    

Status: 404

{ "code": "NoJobError", "message": "Job $jobname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `POST user/:username/jobs`
    
    Submit a job in the system.
    
    *Request*
    

POST /api/v1/user/:username/jobs Authorization: Bearer <access_token>

    <br />*Parameters*
    
    [job config json](../job_tutorial.md)
    
    *Response if succeeded*
    

Status: 202

{ "message": "update job $jobName successfully" }

    <br />*Response if the virtual cluster does not exist.*
    

Status: 400

{ "code": "NoVirtualClusterError", "message": "Virtual cluster $vcname is not found." }

    <br />*Response if user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "User $username is not allowed to add job to $vcname }

    <br />*Response if there is a duplicated job submission*
    

Status: 409

{ "code": "ConflictJobError", "message": "Job name $jobname already exists." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET user/:username/jobs/:jobName/config`
    
    Get job config JSON content.
    
    *Request*
    

GET /api/v1/user/:username/jobs/:jobName/config

    <br />*Response if succeeded*
    

Status: 200

{ "jobName": "test", "image": "pai.run.tensorflow", ... }

    <br />*Response if the job does not exist*
    

Status: 404

{ "code": "NoJobError", "message": "Job $jobname is not found." }

    <br />*Response if the job config does not exist*
    

Status: 404

{ "code": "NoJobConfigError", "message": "Config of job $jobname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET user/:username/jobs/:jobName/ssh`
    
    Get job SSH info.
    
    *Request*
    

GET /api/v1/user/:username/jobs/:jobName/ssh

    <br />*Response if succeeded*
    

Status: 200

{ "containers": [ { "id": "<container id>", "sshIp": "<ip to access the container's ssh service>", "sshPort": "<port to access the container's ssh service>" }, ... ], "keyPair": { "folderPath": "HDFS path to the job's ssh folder", "publicKeyFileName": "file name of the public key file", "privateKeyFileName": "file name of the private key file", "privateKeyDirectDownloadLink": "HTTP URL to download the private key file" } }

    <br />*Response if the job does not exist*
    

Status: 404

{ "code": "NoJobError", "message": "Job $jobname is not found." }

    <br />*Response if the job SSH info does not exist*
    

Status: 404

{ "code": "NoJobSshInfoError", "message": "SSH info of job $jobname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `PUT user/:username/jobs/:jobName/executionType`
    
    Start or stop a job.
    
    *Request*
    

PUT /api/v1/user/:username/jobs/:jobName/executionType Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "value": "START" | "STOP" }

    <br />*Response if succeeded*
    

Status: 200

{ "message": "execute job $jobName successfully" }

    <br />*Response if the job does not exist*
    

Status: 404

{ "code": "NoJobError", "message": "Job $jobname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET virtual-clusters`
    
    Get the list of virtual clusters.
    
    *Request*
    

GET /api/v1/virtual-clusters

    <br />*Response if succeeded*
    

Status: 200

{ "vc1": { } ... }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `GET virtual-clusters/:vcName`
    
    Get virtual cluster status in the system.
    
    *Request*
    

GET /api/v1/virtual-clusters/:vcName

    <br />*Response if succeeded*
    

Status: 200

{ //capacity percentage this virtual cluster can use of entire cluster "capacity":50, //max capacity percentage this virtual cluster can use of entire cluster "maxCapacity":100, // used capacity percentage this virtual cluster can use of entire cluster "usedCapacity":0, "numActiveJobs":0, "numJobs":0, "numPendingJobs":0, "resourcesUsed":{ "memory":0, "vCores":0, "GPUs":0 }, "state":"running" }

    <br />*Response if the virtual cluster does not exist*
    

Status: 404

{ "code": "NoVirtualClusterError", "message": "Virtual cluster $vcname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `PUT virtual-clusters/:vcName` (administrator only)
    
    Add or update virtual cluster quota in the system, don't allow to operate "default" vc.
    
    *Request*
    

PUT /api/v1/virtual-clusters/:vcName Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "vcCapacity": new capacity, "vcMaxCapacity": new max capacity, range of [vcCapacity, 100] }

    <br />*Response if succeeded*
    

Status: 201

{ "message": "Update vc: $vcName to capacity: $vcCapacity successfully." }

    <br />*Response if try to update "default" vc*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Don't allow to update default vc" }

    <br />*Response if current user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if no enough quota*
    

Status: 403

{ "code": "NoEnoughQuotaError", "message": "No enough quota in default vc." }

    <br />*Response if "default" virtual cluster does not exist*
    

Status: 404

{ "code": "NoVirtualClusterError", "message": "Default virtual cluster is not found, can't allocate or free resource." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `DELETE virtual-clusters/:vcName` (administrator only)
    
    remove virtual cluster in the system, don't allow to operate "default" vc.
    
    *Request*
    

DELETE /api/v1/virtual-clusters/:vcName Authorization: Bearer <access_token>

    <br />*Response if succeeded*
    

Status: 201

{ "message": "Remove vc: $vcName successfully." }

    <br />*Response if current user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if try to update "default" vc*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Don't allow to remove default vc." }

    <br />*Response if the virtual cluster does not exist*
    

Status: 404

{ "code": "NoVirtualClusterError", "message": "Virtual cluster $vcname is not found." }

    <br />*Response if "default" virtual cluster does not exist*
    

Status: 404

{ "code": "NoVirtualClusterError", "message": "Default virtual cluster is not found, can't allocate or free resource." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" }

    <br />### `PUT virtual-clusters/:vcName/status` (administrator only)
    
    Change virtual cluster status, don't allow to operate "default" vc.
    
    *Request*
    

PUT /api/v1/virtual-clusters/:vcName/status Authorization: Bearer <access_token>

    <br />*Parameters*
    

{ "vcStatus": "running" | "stopped" }

    <br />*Response if succeeded*
    

Status: 201

{ "message": "Update vc: $vcName to status: $vcStatus successfully." }

    <br />*Response if try to update "default" vc*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Don't allow to update default vc" }

    <br />*Response if current user has no permission*
    

Status: 403

{ "code": "ForbiddenUserError", "message": "Non-admin is not allow to do this operation." }

    <br />*Response if the virtual cluster does not exist*
    

Status: 404

{ "code": "NoVirtualClusterError", "message": "Virtual cluster $vcname is not found." }

    <br />*Response if a server error occured*
    

Status: 500

{ "code": "UnknownError", "message": "*Upstream error messages*" } ```

## About legacy jobs

从此版本开始，会启用 [Framework ACL](../../../subprojects/frameworklauncher/yarn/doc/USERMANUAL_zh_CN.md#Framework_ACL) ，Job 的命名空间会包含创建者的用户名。 jobs will have a namespace with job-creater's username. However there were still some jobs created before the version upgrade, which has no namespaces. They are called "legacy jobs", which can be retrieved, stopped, but cannot be created. To figure out them, there is a "legacy: true" field of them in list apis.

In the next versions, all operations of legacy jobs may be disabled, so please re-create them as namespaced job as soon as possible.