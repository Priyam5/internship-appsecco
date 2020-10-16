# DAST scan

##  Objective 

This section aims to perform a DAST scan on [angular-realworld-example-app](https://github.com/gothinkster/angular-realworld-example-app) and generate a report to provide a solution to the 4th point of the [problem statement](https://cloud-native.netlify.app/problem-statement/) under Task 1.

## Installing the application manually

Firstly, I installed the application manually and ran it on my browser to know how it works. So I cloned the application in my terminal
```
git clone https://github.com/gothinkster/angular-realworld-example-app.git
```
* Install npm

```
sudo apt update
sudo apt install nodejs
sudo apt install npm
nodejs -v
```

* Install Yarn (https://classic.yarnpkg.com/en/docs/install/#debian-stable)
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install yarn
export PATH="$PATH:`yarn global bin`"
yarn install
yarn -version
```
* Install Angular CLI(https://angular.io/cli)
```
npm install -g @angular/cli
```
* Got an error as on running `ng serve` opens editor instead of loading local URL. This is the terminal editor on the 'ng' alias. I uninstalled it with:
```
sudo apt purge ng-common ng-latin
```
Now again I ran `ng serve` and in the browser I typed `localhost:4200` (4200 is default port). The application was successfully installed and window that opened is shown below:
![](Images/angular%20app.png)

## Installing the application through Docker

I firstly cloned the application folder and made a file `Dockerfile`. In this I used a node [image](https://hub.docker.com/_/node)
```
nano Dockerfile
```
I written this code in Dockerfile
```
#getting base image
FROM node

MAINTAINER Priyam Singh <2020priyamsingh@gmail.com>

RUN apt-get update
COPY . /src
WORKDIR /src

#Installing Angular CLI
RUN npm install
RUN npm install -y -g @angular-devkit/build-angular
RUN npm install -y -g @angular/cli

EXPOSE 4200
CMD ["ng", "serve", "--host", "0.0.0.0"]
```
* My application was not running on browser but it was getting compiled because I made a mistake that I was not writing "--host", "0.0.0.0" (--host 0.0.0.0 to listen to all the interfaces from the container).
* I was facing many errors such of packages getting failed so I removed my code of Yarn and only installed with Angular CLI

After this I build the image
```
docker build -t angular5:latest .
```

Then ran the container
```
docker run --rm --name docker5 -p 1234:4200 angular5:latest
```
On the browser I opened `localhost:1234` it worked and the below window got opened.
![](Images/application%20docker%20running.png)


## Installing application through AWS

### Installing AWS CLI in terminal
I followed this official link for the installation of AWS CLI(https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html#cliv2-linux-install)

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
I got the version output as shown below. It shows AWS CLI is successfully installed.
```
aws-cli/2.0.56 Python/3.7.3 Linux/5.3.0-64-generic exe/x86_64.ubuntu.19
```

### setting up AWS profile

(https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)
```
aws configure
AWS Access Key ID [****************4529]: <Enter the ID>
AWS Secret Access Key [None]: <Enter the Access Key>
Default region name [None]: us-east-2
Default output format [None]: json
```
aws sts get-caller-identity
"UserId": "AIDAVXTWDFXQ2SJNKMMAV",
    "Account": "394310921697",
    "Arn": "arn:aws:iam::394310921697:user/priyam"

### ECR 

Amazon Elastic Container Registry (ECR) is a fully-managed Docker container registry that makes it easy for developers to store, manage, and deploy Docker container images. Amazon ECR eliminates the need to operate your own container repositories or worry about scaling the underlying infrastructure. Amazon ECR hosts your images in a highly available and scalable architecture, allowing you to reliably deploy containers for your applications. 

#### Creating an ECR Repository

* Open the Amazon ECR console
* In the navigation pane, choose Repositories. 
* On the Repositories page, choose Create repository. 
* Repository name, enter a unique name for your repository.
* For Tag immutability, choose the tag mutability setting for the repository. Repositories configured with immutable tags will prevent image tags from being overwritten.
* For Scan on push, choose the image scanning setting for the repository. Repositories configured to scan on push will start an image scan whenever an image is pushed, otherwise image scans need to be started manually.
* For KMS encryption, choose whether to enable encryption of the images in the repository using AWS Key Management Service. 

#### Deleting an ECR repository

To delete a repository

* Open the Amazon ECR console.

* From the navigation bar, choose the Region that contains the repository to delete.

* In the navigation pane, choose Repositories.

* On the Repositories page, select the repository to delete and choose Delete.

* In the Delete repository_name window, verify that the selected repositories should be deleted and choose Delete. 

#### Pushing an ECR Repository

When be create a repository it shows commands for pushing. So we have have to follow these commands and we can easily push the image to our ECR Repository.
```
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 394310921697.dkr.ecr.us-east-2.amazonaws.com 

docker tag angular5:latest 394310921697.dkr.ecr.us-east-2.amazonaws.com/angular-app-repo:latest

docker push 394310921697.dkr.ecr.us-east-2.amazonaws.com/angular-app-repo:latest
```

### Adding AWS through GitHub Actions

secrets(https://github.com/marketplace/actions/configure-aws-credentials-action-for-github-actions)

In the YML file I also added the AWS CLI portion
```
- name: Installing AWS CLI
      run: |
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.DEMO_ID }}
        aws-secret-access-key: ${{ secrets.DEMO_K }}
        aws-region: us-east-2

    - name: Pushing image to AWS
      run: |
        aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 394310921697.dkr.ecr.us-east-2.amazonaws.com     
        docker tag angular5:latest 394310921697.dkr.ecr.us-east-2.amazonaws.com/angular-app-repo:latest
        docker push 394310921697.dkr.ecr.us-east-2.amazonaws.com/angular-app-repo:latest
```
The image got successfully pushed to ECR.