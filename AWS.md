# Hosting on Amazon AWS (EC2/ECS)

#### Step 1. Install the amazon ecs-cli
[Install ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)

#### Step 2. Configure your profile
`ecs-cli configure profile --profile-name profile_name --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY`

See (https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_Configuration.html) for more information

#### Step 3. Configure your cluster
`ecs-cli configure --cluster victron --default-launch-type EC2 --region us-east-1 --config-name victron`

#### Step 4.  Create your cluster 
`ecs-cli up --keypair ECS --capability-iam --size 1 --instance-type t2.medium --cluster-config victron`

#### Step 5. Deploy to the cluster

`ecs-cli compose --ecs-params ecs-params.yml -f docker-compose-ecs.yaml up --create-log-groups --cluster-config victron`

#### Step 6. Access Grafana
Go to the new EC2 instance to get the public ip address
Then go to http://EC2_PUBLIC_IP

#### Step 7. To cleanup and bring down the cluster
`ecs-cli down --cluster-config victron`
