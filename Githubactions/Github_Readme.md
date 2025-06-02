**why we use github instead of jenkins ?**

In jenkins we will install jenkins in master and attached slaves servers to run the jobs

But in Github we have runners(similar to slave)

**shared runners:** Shared runners we can run the jobs it is completey free, but we cant have access to the backend.
           
           Ex: if we want to run any commands we need to write ubuntu_latest

**private runners:**  Here we can bring our own virtual machine and add this as a runner, it is complete isolated

          
**Setup**

**Step-1:** create a repository, push the code to the github 

![image](https://github.com/user-attachments/assets/7d588f7e-7053-4b5d-a099-5fc81d3676b1)

**Step-2:** For shared runners click on Actions and select the template for the new workflow

![image](https://github.com/user-attachments/assets/e858b7eb-c9ac-42f9-9e6b-e077b1871b91)

**Step-3:** write a cicd file using the template

CICD: Deploy into ECR 

name: Deploy to Amazon ECS

on:
  push:
    branches: [ "master" ]

env:
  AWS_REGION: ap-south-1
  AWS_ACCESS_KEY_ID: AKIA2RVWE6TFIMJ4F2G3
  AWS_SECRET_ACCESS_KEY: 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
  ECR_REPOSITORY: testing

jobs:
  compile:
  
    name: deploy on eks
    runs-on: ubuntu-latest

    steps:
    
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ env.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ env.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to Amazon ECR
      id: build-image
      env:
         ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
         IMAGE_TAG: testing_v${{ github.run_number }}
      run: |
         # Build a docker container and
         # push it to ECR so that it can
         docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
         docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
         echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

    - name: Set image tag
      run: echo "IMAGE_NAME=${{ steps.login-ecr.outputs.registry }}/my-repo:testing_v${{ github.run_number }}" >> $GITHUB_ENV

    - name: Deploy with updated image
      run: |
            envsubst < Kubernetes/deployment.yaml > Kubernetes/deployment-rendered.yaml
            cat Kubernetes/deployment-rendered.yaml  # Optional: Debug output
            kubectl apply -f Kubernetes/deployment-rendered.yaml
    
    # manifest file in different folder

    # - name: Download deployment.yaml from another repo
    #   run: |
    #    curl -o kubernetes/deployment.yaml https://raw.githubusercontent.com/other-user/k8s-repo/main/manifests/deployment.yaml


    # - name: Deploy Amazon ECS task definition
    #   uses: aws-actions/amazon-ecs-deploy-task-definition@v1
    #   with:
    #     task-definition: ${{ steps.task-def.outputs.task-definition }}
    #     service: ${{ env.ECS_SERVICE }}
    #     cluster: ${{ env.ECS_CLUSTER }}
    #     wait-for-service-stability: true

**Step-4:** Setup a secret variables for access key and secret key 

**click on project repository and click on settings, click on actions under security variables**

 ![image](https://github.com/user-attachments/assets/107eb804-1622-4de8-abb6-a3c465dbab29)

click on New repository secret, add access & secret key

![image](https://github.com/user-attachments/assets/1a0d4c5d-2fb2-458d-a57d-7dd4ef1d5f02)

![image](https://github.com/user-attachments/assets/433c9c91-be30-4125-aaca-12191c9d35f1)

![image](https://github.com/user-attachments/assets/c19adb46-7406-48b2-bf43-d1f2c46cb27f)

**Step-5:** Setup a self-hosted runner

**click on project repository and click on settings, click on actions under code and automation**

![image](https://github.com/user-attachments/assets/c5dfa6ad-de80-461a-b018-55c32791cf5b)

click on new self hosted runner login with ubuntu and run the commands in the server

![image](https://github.com/user-attachments/assets/3c3359b0-63d0-4a27-95b0-8ae41287d346)

![image](https://github.com/user-attachments/assets/4ecb5052-e6c6-48d9-9a9e-cc36bf119409)

Enter the runner name and label 

![image](https://github.com/user-attachments/assets/11df2afa-df74-47a8-b4a1-6feb140c9b97)

to start the runner 

./run.sh

![image](https://github.com/user-attachments/assets/e67b6a82-efd3-4ae3-8af3-ecf4b877e388)



