**Prerequisites:** Python; a local ssh key for AWS use

1. Visit AWS IAM via the AWS console. In IAM, create an AWS role for EC2 instances that both has access to Elastic Beanstalk and can write logs to CloudWatch. This role may include standard policies like `AmazonEC2ContainerRegistryReadOnly`, `AWSElasticBeanstalkWebTier`, `AWSElasticBeanstalkMulticontainerDocker`, `AWSElasticBeanstalkWorkerTier`. It should also include a CloudWatch policy that has the actions `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`, and `logs:DescribeLogStreams`.
2. [Download the Elastic Beanstalk CLI](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) with `pip install awsebcli --upgrade --user`.
3. Create a local directory, e.g. `simplified_eb/`, to house project files and create supporting configuration files. All of the following steps take place inside this local directory.
    ```sh
    $ mkdir simplified_eb && cd simplified_eb
    $ touch Dockerrun.aws.json
    $ mkdir .ebextensions
    $ touch .ebextensions/01_cloudwatch_agent_config.config
    ```
4. Alter the two files you just created with the following content. Make sure to update `<YOUR-TARGET-TAG>` in `Dockerrun.aws.json` with the Docker image tag you would like to deploy.
    ```
    // Dockerrun.aws.json
    {
      "AWSEBDockerrunVersion": 2,
      "containerDefinitions": [
        {
          "name": "circ-webapp",
          "image": "nypl/circ-webapp:<YOUR-TARGET-TAG>",
          "essential": true,
          "memory": 1536,
          "portMappings": [
            {
              "hostPort": 80,
              "containerPort": 80
            }
          ],
          "mountPoints": [
            {
              "sourceVolume": "awseb-logs-circ-webapp",
              "containerPath": "/var/log/uwsgi"
            }
          ]
        }
      ]
    }
    ```

    ```yaml
    # .ebextensions/01_cloudwatch_agent_config.config
    files:
      '/etc/awslogs/config/application_log.conf' :
        mode: "000644"
        owner: root
        group: root
        content: |
          [/var/log/containers/circ-webapp/uwsgi.log]
          file=/var/log/containers/circ-webapp/uwsgi.log*
          log_group_name=`{"Fn::Join":["/", ["/aws/elasticbeanstalk", { "Ref":"AWSEBEnvironmentName" }, "var/log/uwsgi/uwsgi.log"]]}`
          log_stream_name={instance_id}
    
    commands:
      restart_awslogs:
        command: service awslogs restart
    ```
5. Run `eb init` to configure the Elastic Beanstalk application. This will begin an interactive tool. Select your target region and create a new, `Multi-container Docker` environment, using the latest version. For debugging purposes, we recommend enabling SSH into the EC2 instance.
6. Use `eb create` to set the configuration for your target Elastic Beanstalk environment. This will kickstart another interactive tool, though you can also use CLI options. For example, if your application is called `simplye-circulation`, you may run the following command with the appropriate environment name (e.g. dev, qa, prod)
    ```sh
    $ eb create simplye-circulation-[environmentname] \
        --instance_type t2.small \
        --instance_profile loggable-beanstalk \
        --cname simplye-circulation-[environmentname] \
        --vpc.id target-vpc-id \
        --vpc.elbsubnets public-subnet-id-1,public-subnet-id-2 \
        --vpc.ec2subnets private-subnet-id-1,private-subnet-id-2 \
        --vpc.elbpublic \
        --tags Project=SimplyE, Foo=Bar \
        --keyname your-aws-ssh-keyname \
        --scale 2 \
        --envvars SIMPLIFIED_PRODUCTION_DATABASE="xxx",VAR_NAME_2="xxx"
    ```
    See [the `eb create` documentation](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-create.html) for more information. Be sure to configure the environment with the required `SIMPLIFIED_PRODUCTION_DATABASE` variable.
7. Run `eb deploy --label "a-versioned-label" --message "Your human-readable version description"` to deploy your the container image described in `Dockerrun.aws.json`. Documentation for `eb deploy` can be found [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-deploy.html).
8. Using the AWS console, check Elastic Beanstalk for application health and CloudWatch for uwsgi logs.
9. As you need to update the container version, make local changes to the image tag in `Dockerrun.aws.json` and redeploy with `eb deploy`. Application environments can also be duplicated and scaled in Elastic Beanstalk, as needed.