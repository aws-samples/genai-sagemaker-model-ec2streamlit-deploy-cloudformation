## Deploy GenAI Model in Amazon Sagemaker & Streamlit app in EC2 using Cloudformation

## Description
This sample project serves as a building block for deploying using AWS CloudFormation a pre-trained, Foundation Model endpoint on Amazon SageMaker and hosting a Streamlit user interface on an Amazon Elastic Compute Cloud (EC2) instance to interact with the model. It provides a foundation for developers to provision the necessary infrastructure and build upon it according to their specific requirements. Falcon-7B is the model demonstrated in the sample but the process can be used to deploy other models available in the Amazon Sagemaker Jumpstart. 

This repository contains a simple example to deploy using Infrastructure as Code (IaC) service Cloudformation, refer [amazon-sagemaker-generativeai
](https://github.com/aws-samples/amazon-sagemaker-generativeai/tree/main) for elaborate options to train and deploy Generative AI models using notebooks and [generative-ai-sagemaker-cdk-demo](https://github.com/aws-samples/generative-ai-sagemaker-cdk-demo/tree/main) to automate using AWS Cloud Development Kit (AWS CDK).

## Prerequisites
1. An AWS Account, to deploy the infrastructure. You can find more instructions to create your account [here](https://aws.amazon.com/free).
2. Template is configured to run in us-east-1 and us-west-2 region. 
3. Default VPC with public subnet
4. Approximate deployment time is 15 minutes. 
5. Multiple resources, including SageMaker GPU instances and EC2 instances, will be provisioned and you will be responsible for the associated costs. Please monitor your AWS account usage carefully to ensure that resources are terminated when no longer required.

## Solution Overview

### Amazon Sagemaker Jumpstart
Amazon SageMaker JumpStart provides pre-trained, open-source models for a wide range of problem types to help you get started with ML. You can incrementally train and tune these models before deployment. JumpStart also provides solution templates that set up infrastructure for common use cases, and executable example notebooks for ML with [Amazon SageMaker](https://aws.amazon.com/sagemaker/).

With over 600 pre-trained models available and growing every day, JumpStart enables developers to quickly and easily incorporate cutting-edge ML techniques into their production workflows. You can access the pre-trained models, solution templates, and examples through the JumpStart landing page in Amazon SageMaker Studio. You can also access JumpStart models using the SageMaker Python SDK. For information about how to use JumpStart models programmatically, see [Use SageMaker JumpStart Algorithms with Pretrained Models](https://sagemaker.readthedocs.io/en/stable/overview.html#use-sagemaker-jumpstart-algorithms-with-pretrained-models).

In April 2023, AWS unveiled [Amazon Bedrock](https://aws.amazon.com/bedrock/), which provides a way to build generative AI-powered apps via pre-trained models from various providers. Amazon Bedrock also offers access to Titan foundation models, a family of models trained in-house by AWS. With the serverless experience of Amazon Bedrock, you can easily find the right model for your needs, get started quickly, privately customize FMs with your own data, and easily integrate and deploy them into your applications using the AWS tools and capabilities you’re familiar with (including integrations with SageMaker ML features like Amazon SageMaker Experiments to test different models and Amazon SageMaker Pipelines to manage your FMs at scale) without having to manage any infrastructure.

### Streamlit
Streamlit is an open-source Python framework to deliver interactive data apps. The repository contains base code for a Streamlit application that interacts with the deployed machine learning model on SageMaker. However, it is important to note that this code is not production-ready, and developers should thoroughly test and vet the application according to their security guidelines before deploying it to a production environment.

While this sample focuses on deploying the Streamlit application on an EC2 instance, it is recommended to consider using Amazon [Elastic Container Service (ECS)](https://github.com/aws-samples/streamlit-application-deployment-on-aws) or [AWS Fargate](https://github.com/aws-samples/streamlit-deploy/tree/main) as preferred options for hosting containerized applications. These services offer better scalability, load balancing, and resource management capabilities compared to running the application directly on an EC2 instance.

However, if there is a specific requirement to host the Streamlit application on an EC2 instance, this project can serve as a starting point. Developers can use the provided CloudFormation templates to provision the necessary infrastructure and then build upon it to meet their specific needs.

### Deploy a Large Language Model (Falcon-7B) in SageMaker with a Streamlit Web UI
[Falcon-7B](https://huggingface.co/tiiuae/falcon-7b-instruct) is a powerful and versatile language model developed by TII and is used for the text generation use cases. It is a 7-billion parameter causal decoder-only model trained on a massive dataset of text and code. The Falcon-7B can be used for a variety of tasks, including generating text, translating languages, writing different creative text formats, and answering your questions in an informative way. Refer Falcon-7B [Apache 2.0 license](https://huggingface.co/tiiuae/falcon-7b) for additional details.

The architecture consists of an EC2 instance running a Streamlit web UI and a SageMaker endpoint hosting the Falcon model. The Streamlit web UI serves as the user interface, allowing users to input text and receive responses from the Falcon model. The Streamlit application communicates with the SageMaker endpoint using HTTP requests.

![Solution Architecture Falcon](/images/genaisolutionarchitecture-falcon.png)

## Steps to Deploy the Sagemaker Jumpstart model Falcon-7B and Streamlit UI

### Step 1: Deploy cloudformation yaml

1. Click the Launch stack to directly deploy the application. Clicking the link will take you to the CloudFormation console within your signed-in AWS account.

> [!NOTE]  
> Ensure you are in the intended AWS Account, as you will be responsible for the associated costs.

|   Region   | txt-streamlitinfra |
| ---------- | ------------------ |
| us-east-1 or us-west-2  | [![launch-stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=StreamlitDeploy&templateURL=https://ws-assets-prod-iad-r-iad-ed304a55c2ca1aee.s3.us-east-1.amazonaws.com/c10312f0-aa83-4ba5-b908-599d80a75179/txt-streamlitinfra.yaml)|

> [!NOTE]  
> Currently, this template can only be deployed and run in us-west-2 (Oregon) or us-east-1 (N. Virginia) region.
> The EC2 instance is deployed in the default VPC

2. Click Next.
3. On the "Specify stack details" page of the CloudFormation template, update the necessary details and click "Next". In the "MyIP" field, enter your public IP address in CIDR notation. You can find your public IP address by visiting https://checkip.amazonaws.com/. For example, if your public IP address is 10.0.0.10, enter "10.0.0.10" in the "MyIP" field. This will allow inbound access to the simple app created by the CloudFormation stack from your specific IP address (as a /32) through the security group. Note that you can modify the security group rules later, after the environment is deployed, if needed.
4. On the Configure stack options page, keep all settings unchanged and click Next.
5. On the Review page, check and confirm whether the related content is correct.
6. Click Submit to start deploying the environment
7. Through the CloudFormation console, we can observe the current Stack's deployment progress and related items. During the deployment process, two associated Stacks will be created. In about 15 minutes, our Stack will be fully deployed, and the deployment status on the console will change to CREATE_COMPLETE.

To deploy via the command line, execute the following command:

aws cloudformation deploy \
  --template-file streamlitec2-sagemakermodel-cfndeploy.yaml \
  --stack-name sm-streamlit-infra1 \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --parameter-overrides MyIP=YOUR_IP_ADDRESS_HERE

### Step 2: Viewing the app

1. Search for "Cloudformation" in the services and click on the "Cloudformation" in the dropdown
   ![cloudformation](/images/startup-cloudformationsearch.png)

2. Choose the stack you deployed (e.g. txt-streamlitinfra) and Click on the "Outputs" tab.
   ![sdwebui](/images/startup-streaminfraurl.png)

3. Note the Value in "WebUIURL" - this will be the **Falcon 7b UI URL** to be used. Update the security group associated with the EC2 instance if the web UI is not accessible

### Step 3: Customise the deployment
1. To expand the cloudformation template to deploy other models available in the Sagemaker Jumpstart such as models from Hugging Face, follow the procedure [in this link](https://community.aws/content/2gw7NsgJM0H7RrlJL5sJQRQNJhD/fast-pre-trained-model-deployment-the-code-only-approach) to get the 'ModelDataUrl' and 'TrainingImage' details
2. Update the CloudFormation template with the new values, test the modified template by creating a new stack.
3. For additional regions, repeat the above steps with region-specific details


## Clean up deployment
- Open the CloudFormation console.
- Select the stack (e.g. txt-streamlitinfra) you created, and click **Delete**. Wait for the stack to be deleted.

## Security
See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License
This library is licensed under the MIT-0 License. See the LICENSE file.

