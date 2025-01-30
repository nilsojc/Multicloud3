<p align="center">
  <img src="assets/diagram.png" 
</p>
  
## ☁️ MultiCloud, DevOps & AI Challenge — Day 3 —  ☁️

This is part of the third project of the Multicloud, Devops and AI Challenge!

In this project we will be 


<h2>Environments and Technologies Used</h2>

  - Docker
  - Amazon Web Services
  - Github Codespaces
  - Python
  - RapidAPI
  
  
<h2>Key Features</h2>  

✅
✅
✅


<h2>Step by Step Instructions</h2>

***1. Repo configuration***


NOTE: Keep in mind this is for a Linux environment, check the AWS documentation to install it in your supported OS.


   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install


We then do `AWS configure` and enter our access and secret key along with the region. Output format set to JSON. With this command we will double check that our credentials are put in place for CLI:

```
aws sts get-caller-identity
```

We will then roceed with installing the Docker CLI and Docker in Docker (Github Codespaces Setup)

```
curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-20.10.9.tgz -o docker.tgz \
tar -xzf docker.tgz \
sudo mv docker/docker /usr/local/bin/ \
rm -rf docker docker.tgz
```

`Ctrl + p` on Github Codespace > `Add Dev Container Conf files` > modify your active configuration > click on Docker (Docker-in-Docker)

![image](/assets/image1.png)



***2.  API and Requirements Setup***



***3. Docker Test and Debugging***




***Cleanup***

When you're done, you can clean up the Docker resources with these commands:



<h2>Conclusion</h2>

In this project, I learned how to 
