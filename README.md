# Deploy-10-EC2-with-EIP-and-SSM-Agent

### 1、批量开机且绑定EIP的需求探讨

在某特殊场景下，批量创建EC2且需要绑定EIP。

如果使用ASG弹性扩展组+Spot实例，无法很好的管理EIP。这个过程需要使用Userdata编写AWSCLI脚本申请EIP，然后获取EIP的ID再去逐个绑定，而且Spot弹性实例一旦释放了重新申请，脚本需要重新匹配EIP映射。

对比这个复杂度，转而使用CloudFormation则可以几行配置就将EC2和EIP绑定。因此这个场景通过CloudFormation来实现。当然也可以使用CDK实现。

### 2、部署方案的探讨

在创建完成后，还需要批量部署软件。如果可以在AMI内打包应用是最好的，直接使用CloudFormation执行AMI ID即可。

此外，还可以使用EFS共享文件系统，在多个EC2上多重挂载，即可实现一次应用升级对所有EC2有效。

如果考虑到后续升级版本，也可以使用CodeDeploy做全面的CICD对接。

如果客户当前DevOps流程不完善或者存在能力问题，则可以使用最简单的shell脚本方式部署应用。此时，通过System Manager Fleet Manager，可以在EC2上批量执行shell。这个方案要求的前提是：EC2系统是Amazon Linux 2（兼容功能CentOS7），或者其他安装了SSM Agent的操作系统；同时，EC2挂载了IAM角色，此角色需要赋予名为AmazonSSMManagedInstanceCore的IAM Policy。由此即可通过System Manager Run Command功能批量执行脚本。这一部分工作，也可以集成在CloudFormation创建EC2脚本中。

### 3、参考模版和配置Demo视频

请参考[CloudFormation](https://github.com/aobao32/Deploy-10-EC2-with-EIP-and-SSM-Agent/blob/main/Deploy%2010%20EC2%20with%20EIP%20and%20SSM%20Agent.yml)模版。

请参考视频：

（1）使用Cloudfromation创建多个EC2并绑定EIP。此模版已经为EC2绑定了SSM需要的IAM Role。如下视频。

<video src="https://blogimg.bitipcman.com/video/Cloudformation-system-manager-01.mov" width="1280px" height="720px" controls="controls"></video>

（2）检查CloudFormation创建的EC2正常并通过Session Manager登录验证。如下视频。

<video src="https://blogimg.bitipcman.com/video/Cloudformation-system-manager-02.mov" width="1280px" height="720px" controls="controls"></video>

（3）使用System Manager批量执行命令。如下视频。

<video src="https://blogimg.bitipcman.com/video/Cloudformation-system-manager-03.mov" width="1280px" height="720px" controls="controls"></video>
