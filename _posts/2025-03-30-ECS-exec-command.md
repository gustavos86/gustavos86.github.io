---
title: AWS ECS Exec to access a container's CLI running in Fargate
date: 2024-12-08 15:30:00 -0700
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

If it happens to be enabled, run the following command to get a `bash` prompt:

```bash
aws ecs execute-command  \
    --region <AWS_REGIOM> \
    --cluster <ECS_CLUSTER_NAME> \
    --tasks <ECS_TASK_ID>
    --container <CONTAINER_NAME> \
    --command "/bin/bash" \
    --interactive
```

## Resources

- [NEW – Using Amazon ECS Exec to access your containers on AWS Fargate and Amazon EC2](https://aws.amazon.com/blogs/containers/new-using-amazon-ecs-exec-access-your-containers-fargate-ec2/)
- [Install the Session Manager plugin on macOS](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-macos-overview.html)
- [Verify the Session Manager plugin installation](https://docs.aws.amazon.com/systems-manager/latest/userguide/install-plugin-verify.html)
- [Monitor Amazon ECS containers with ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html)
- [Updating an ECS service automatically using the CLI via Lambda](https://repost.aws/questions/QUkzW33fTNTw2g9oJNMoANdA/updating-an-ecs-service-automatically-using-the-cli-via-lambda)
