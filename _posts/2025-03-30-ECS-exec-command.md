---
title: AWS ECS Exec to access a container's CLI running in Fargate
date: 2025-03-30 16:30:00 -0700
categories: [AWS, AWS CLI]
tags: [aws, aws cli]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/ecs.png)

I was looking for the way to get a `bash` prompt to a Container running on **ECS Fargate** for debugging & troubleshooting purposes.

Yes, it is possible. However, the tricky part is that the `ExecuteCommand` API must already by enabled on the **Task**. From the [documentation](https://aws.amazon.com/blogs/containers/new-using-amazon-ecs-exec-access-your-containers-fargate-ec2/):

*If a task is deployed or a service is created without the `--enable-execute-command` flag, you will need to redeploy the task (with `run-task`) or update the service (with `update-service`) with these opt-in settings to be able to exec into the container.*

Even more, it seems we cannot enable `ExecuteCommand` API from the AWS Management Console. Also from the [documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html):

*ECS Exec is not currently supported using the AWS Management Console.*

Using the AWS CLI, this command can be used to see the value of the `ExecuteCommand` API for a specific **Task**:

```bash
aws ecs describe-tasks \
    --region <AWS_REGIOM> \
    --cluster <ECS_CLUSTER_NAME> \
    --tasks <ECS_TASK_ID>
```

Output:

```
. . . 
            "enableExecuteCommand": false,
. . .
```

In this case, the `ExecuteCommand` API is **disabled**.

## Enabling the ExecuteCommand API

You need to update the service including the `--enable-execute-command` parameter.

```bash
aws ecs update-service \
    --region <AWS_REGIOM> \
    --cluster <ECS_CLUSTER_NAME> \
    --service <SERVICE_NAME> \
    --task-definition <TASK_DEFINITION_NAME> \
    --enable-execute-command \
    --force-new-deployment
```

However, you may run into the following error message when running the command if the **Task Definition** does not have a **Task Role** attached.

```bash
An error occurred (InvalidParameterException) when calling the UpdateService operation: The service couldn't be updated because a valid taskRoleArn is not being used. Specify a valid task role in your task definition and try again.
```

To solve this, create a new revision of the **Task Definition** attaching a **Task Role** from the **AWS Management Console**

Making sure there is a **Task Role** attached to the **Task Definition**:

![]({{ site.baseurl }}/images/2025/03-30-ECS-exec-command/01-Task-Role.png)

Source: [Can not update my existing ECS service to enable execute command #6242](https://github.com/aws/aws-cli/issues/6242)

We should see the **ECS Exec** parameters enabled on **Tasks** running on the **service**.

![]({{ site.baseurl }}/images/2025/03-30-ECS-exec-command/02-ECS-Exec.png)


----

Once the service is updated, run the following command:

```bash
aws ecs execute-command  \
    --region <AWS_REGIOM> \
    --cluster <ECS_CLUSTER_NAME> \
    --tasks <ECS_TASK_ID>
    --container <CONTAINER_NAME> \
    --command "/bin/bash" \
    --interactive
```

This time, you may run into the following errror in case the **Task Role** attached does not have the right permissions.

```bash
An error occurred (TargetNotConnectedException) when calling the ExecuteCommand operation: The execute command failed due to an internal error. Try again later.
```

To solve this, you can go attach an **inline policy** to the **Task Role**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssmmessages:CreateControlChannel",
        "ssmmessages:CreateDataChannel",
        "ssmmessages:OpenControlChannel",
        "ssmmessages:OpenDataChannel"
      ],
      "Resource": "*"
    }
  ]
}
```

**IMPORTANT:** This change may take several minutes to take effect.

Source: [How do I resolve the error "An error occurred (TargetNotConnectedException) when calling the ExecuteCommand operation" in Amazon ECS?](https://repost.aws/knowledge-center/ecs-error-execute-command)

----

## Finanlly, we see results

We can know get a promnpt from the container.

```bash
$ aws ecs execute-command  \
    --region us-east-1 \
    --cluster MyECScluster \
    --task 54022f2f28d342bfb4e31c480f127e4a \
    --container nginxdemos-hello-Container \
    --command "/bin/sh" \
    --interactive

The Session Manager plugin was installed successfully. Use the AWS CLI to start a session.


Starting session with SessionId: ecs-execute-command-abz5brhab9o72lttnt6hj3fo8a
/ # pwd
/
/ # whoami
root
/ #
```

**NOTE:** `/bin/bash` may not be available, in such case try with `/bin/sh` since you may see this error:

```bash
----------ERROR-------
Unable to start command: Failed to start pty: fork/exec /bin/bash: no such file or directory
```

## Resources

- [NEW – Using Amazon ECS Exec to access your containers on AWS Fargate and Amazon EC2](https://aws.amazon.com/blogs/containers/new-using-amazon-ecs-exec-access-your-containers-fargate-ec2/)
- [Install the Session Manager plugin on macOS](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-macos-overview.html)
- [Verify the Session Manager plugin installation](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-verify.html)
- [Monitor Amazon ECS containers with ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html)
- [Updating an ECS service automatically using the CLI via Lambda](https://repost.aws/questions/QUkzW33fTNTw2g9oJNMoANdA/updating-an-ecs-service-automatically-using-the-cli-via-lambda)
- [Can not update my existing ECS service to enable execute command #6242](https://github.com/aws/aws-cli/issues/6242)
- [How do I resolve the error "An error occurred (TargetNotConnectedException) when calling the ExecuteCommand operation" in Amazon ECS?](https://repost.aws/knowledge-center/ecs-error-execute-command)