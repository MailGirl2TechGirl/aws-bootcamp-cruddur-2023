# Week 1 — App Containerization


#Implemented a healthcheck for all services within the docker-compose.yml file

[View commit on GitHub](https://github.com/MailGirl2TechGirl/aws-bootcamp-cruddur-2023/commit/019ab2d58914)


![image](https://user-images.githubusercontent.com/124912958/220475368-f8ada7cc-d530-4b3a-8a8f-94b46c56b9e4.png)

#Ran the dockerfile CMD as an external script by creating a new script updating the Dockerfile

This one gave me a lot of issues. After 3 hours of trying different things it finally worked when I restarted my Gitpod Workspace. I really need to get better at documenting as I go because by the time I have success I've forgotten everything I have tried. This was my first experience using another branch as I did not want to interfere with main as it was working. Or so I thought. When I finally got fed up and almost to the point of giving up, I went back to main and ran docker compose up and I was getting the same error. I inspected the raw code from the webpage and found a CORS error. After searching "CORS" in discord someone had mentioned restarting their workspace. This did the trick and while I could kick myself for not first trying the age old question, "Did you reboot?" My first instinct as a System Administrator is to reboot, so why did I not consider that? Either way, I mostly learn from my mistakes so it was beneficial in the long run. 

[View commit on GitHub](https://github.com/MailGirl2TechGirl/aws-bootcamp-cruddur-2023/commit/c504612694ec1cfd9c90ad82064ba0d7fa21f13c)

Installed Docker on my local machine

![image](https://user-images.githubusercontent.com/124912958/220666405-0c69ed51-2c15-4888-9d7d-05757508e757.png)

I cloned my repo from github to my local machine and when I tried docker compose up I was getting a react error but once I installed npm onto my local machine the front end was working.
I then changed my enviromnet varisables to:
```environment:
      FRONTEND_URL: "http://localhost:3000"
      BACKEND_URL: "http://localhost:4567"
```
Im still unable to see the backend and still troubleshooting this: 


![iFM80tl](https://user-images.githubusercontent.com/124912958/221369245-677f5a50-c0ca-4339-9f9e-4876e44e2399.png)

After 2 days of troubleshooting I finally got it working on my local machine. I had issues with ```npm i``` and it was giving me react errors. I had to change ownership of the directory as it was created with root. I also had to delete the package-lock.json and once I was able to run ```npm i``` without sudo, it worked. 

![ubuntu-cruddur](https://i.imgur.com/04jtjs8.png)


Pulled an image from Dockerhub: 

![docker1](https://user-images.githubusercontent.com/124912958/221369290-0c978ab6-6eca-42bd-8bbf-e85f85333aa1.png)

Pushed an image Dockerhub: 

![docker2](https://user-images.githubusercontent.com/124912958/221369293-7b952258-3e60-437b-9a6d-b769cc6abf38.png)




