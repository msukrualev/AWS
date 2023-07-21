# Web Server Without Using CloudFormation

- Go to EC2 service
- Launch a t2.micro Amazon Linux 2 instance
- Connect to the instance with SSH / Instance Connect
- Enter the following bash commands line by line, to install Apache Web Server

```bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

- check if Apache web server is installed and active using bash or browser
```bash
curl InstancePublicIP
```
- terminate instance

# Web Server using CloudFormation Template

- Go to Cloudformation service
- Click Create stack
- Select Create template in Designer
- Click Create template in designer
- Click Template at the bottom of the Designer Page, select template language as YAML, delete the existing code and copy and paste the following code:

```yaml
AWSTemplateFormatVersion: 2010-09-09

Description: >
  This Cloudformation Template creates Web Server on EC2 Instance. 
  This will run on Amazon Linux 2 (ami-026dea5602e368e96) EC2 Instance with
  custom security group allowing SSH connections from anywhere on port 22.
Parameters:
  KeyPairName:
    Description: Enter the name of your Key Pair for SSH connections.
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: Must one of the existing EC2 KeyPair

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH for WebServer
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-09d95fab7fff3776c
      InstanceType: t2.micro
      KeyName: !Ref KeyPairName
      SecurityGroupIds:
        - !GetAtt WebServerSecurityGroup.GroupId
      Tags:                
        -                        
          Key: Name
          Value: !Sub WebServer of ${AWS::StackName}   
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
                   
Outputs:
  WebsiteURL:
    Description: WebServer DNS Name
    Value: !Sub 
      - ${PublicAddress}
      - PublicAddress: !GetAtt WebServer.PublicDnsName
```

- Click create stack icon like a cloud at the top left.
- Enter key-pair name and then create stack.
- Check the instance being created in EC2.
- Check if the web server is active.
- Delete stack when you finished. This will destroy created resources.

