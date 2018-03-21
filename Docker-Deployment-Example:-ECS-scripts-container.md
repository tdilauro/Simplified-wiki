# `nypl/circ-scripts` on ECS

**Prerequisites:** a local ssh key for AWS use

## Creating the Cluster

1. Navigate to [the ECS console on AWS](https://console.aws.amazon.com/ecs/).
2. Select the "Task Definition" tab from the left-hand menu. Then select "Create New Task Definition".
3. Set the Task Definition Name (e.g. simplye-circulation-scripts), then scroll down and click the "Add container" button.
4. Configure the container and select "create" for the Task Definition. Required settings include:
    - _Container Name_ (e.g. circ-scripts)
    - _Image:_ nypl/circ-scripts:<YOUR_TARGET_TAG>
    - _Memory Limits:_ Soft Limit of at least 1536 MiB
    - _Essential:_ checked/true
    - _Environment Variables:_ `SIMPLIFIED_PRODUCTION_DATABASE` is required. `TZ` and `SIMPLIFIED_DB_TASK` are optional.
5. Navigate to the "Clusters" tab and select "Create Cluster". Select "EC2 Linux + Networking".
6. Configure and create the Cluster. Suggested settings include:
    - _Cluster name_ (e.g. simplye-circulation-scripts)
    - _Provisioning Model:_ On-Demand instance
    - _EC2 instance type:_ t2.small
    - _Number of instances:_ 1
    - _Key pair:_ your AWS ssh key
    - Appropriate VPC, subnet, security group, and IAM role selections
7. In your new Cluster's console, select "Create Service".
8. Configure and create the Service with the following suggested settings:
    - _Launch type:_ EC2
    - _Task definition:_ the name of the task you defined earlier
    - _Service name:_ (e.g. simplye-circulation-scripts)
    - _Number of tasks:_ 1
    - _Load balancer type:_ None
    - _Service Auto Scaling:_ Do not adjust the service's desired count.
    Upon creation, the task should start automatically.

## Updating the container
1. To deploy a new version, return to your Task Definition and select "Create new revision".
2. Update the tag on the selected Docker container image (e.g. `2.2.0` -> `2.2.1`) and any relevant environment variables before saving the new revision.
3. Navigate to the Cluster, select your Service, and click the "Update" button.
4. Update the Task Definition on your Service to have the appropriate new revision number. For example, `simplye-circulation-scripts:1` => `simplye-circulation-scripts:2`. Update the service.
5. In the Cluster console, navigate to the "Tasks" tab. Stop the running Task, which is the old Task Definition revision. A new Task, with the new Task Definition, should begin automatically.