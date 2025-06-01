**why we use github instead of jenkins ?**

In jenkins we will install jenkins in master and attached slaves servers to run the jobs

But in Github we have runners(similar to slave)

**shared runners:** Shared runners we can run the jobs it is completey free, but we cant have access to the backend.
           
           Ex: if we want to run any commands we need to write ubuntu_latest

**private runners:**  Here we can bring our own virtual machine and add this as a runner, it is complete isolated

          
**Setup**

Step-1: create a repository, push the code to the github 

![image](https://github.com/user-attachments/assets/7d588f7e-7053-4b5d-a099-5fc81d3676b1)

Step-2: For shared runners click on Actions and select the template
