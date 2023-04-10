# Momento Cost Estimator

This repo contains tools used for making Momento cost comparisons.

[![Launch Stack](https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?stackName=momento-cost-estimator&templateURL=https://raw.githubusercontent.com/momentohq/momento-cost-estimator/main/MomentoCostEstimator_EC-Redis.yml)


## Momento equivalent cost - Amazon ElastiCache for Redis

This is a [CloudFormation template](https://github.com/momentohq/momento-cost-estimator/blob/main/MomentoCostEstimator_EC-Redis.yml)
which creates a CloudWatch dashboard to show you an estimated equivalent cost for an ElastiCache for Redis cluster if you were supporting
the application load with Momento Serverless Cache instead.

### Deploying the dashboard

Use CloudFormation to create a stack by either...

- copy and paste the template from https://raw.githubusercontent.com/momentohq/momento-cost-estimator/main/MomentoCostEstimator_EC-Redis.yml
- download this file https://raw.githubusercontent.com/momentohq/momento-cost-estimator/main/MomentoCostEstimator_EC-Redis.yml and point CloudFormation to your local copy

You can deploy the stack from any region, as CloudWatch dashboards are global resources. When deploying the stack, you'll be
asked to name the stack - this can be anything but a meaningful name is recommended so that you'll know what it represents later.
You'll also be asked to provide the name of the ElastiCache for Redis cluster and its AWS region. Examples might be "MyRedisCluster" and "us-west-2" or "game-prod" and "us-east-1". It's important to get these right or the metrics won't be correct. If you get it wrong, you can just delete the stack and start over.

Once the stack deployment is complete, browse over to the CloudWatch console, and select your new dashboard.

### Understanding the graphed metrics and comparing costs

The dashboard widget created is a graph of three time series - read throughput (bytes), write throughput (bytes) and estimated Momento cost (US$). By default this will show per-minute granularity over the most recent hour. But you can change this around as you wish. To get a number for comparison, try
setting the granularity (the period) to one month. This will give you a monthly estimated cost for Momento Serverless Cache that you can compare against either a [calculated ElastiCache cost based on number of nodes and node types](https://calculator.aws/) or a cost as determined from your AWS billing.

### Known limitations

- The cost estimate is based on an assessment of data volume from the NetworkBytesIn and NetworkBytesOut metrics produced by ElastiCache for each node. ElastiCache nodes exchange a lot of data as part of basic operation (even if zero requests are being made to Redis). The metric math attempts to compensate for this at the per-minute level, but if you are analyzing a low-traffic (or completely idle) cluster you'll likely see some estimated Momento cost. In reality you'll not be assessed any consumption at all for Momento Serverless Cache if you are not driving any load. This is probably okay as it tends to over-estimate the Momento cost, which is preferred.
- Currently this does not return correct results for a single-node cluster. There must be one or more replica nodes for the metric math to work.
- Behavior for a sharded (cluster-mode enabled) ElastiCache for Redis configuration is untested - may not work correctly.
- The metric math search identifies nodes to include for the given cluster using a naming pattern - it expects all nodes to be named according to the cluster name with a postfix of dash (and some additional characters). This fits with the standard behavior of ElastiCache when creating clusters, but if you've added nodes with naming that doesn't fit this pattern, your results from this tool will be incorrect.

### Sharing results and improving the tool

At Momento, we're interested in helping you to achieve cost savings and simpler operations via our truly serverless product. This tool makes a best effort attempt to give an estimate of cost for Momento Serverless Cache based on available metric from an ElastiCache for Redis cluster. We expect that it will generally be an over-estimate, and we also believe that for many clusters, our serverless product's multi-tenancy and the way we meter for billing (purely on actual consumption - nothing to "provision") will offer a significant cost savings.

We'd love to hear about your results from use of this tool. Did it produce results that seem correct? Were you surprised at the potential for cost savings? Does the tool seem broken for your particular configuration? Please reach out to petenaylor@momentohq.com directly, or post your thoughts/feedback on twitter if you prefer (@momentohq, @pj_naylor). If you find bugs or want to contribute improvements that would be most welcome - please work with us using the github tooling here in the repository.
