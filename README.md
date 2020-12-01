# adf-summit-sample-glassfish
Summit Sample Applications for Oracle ADF deployed in Glassfish 4.1 and Oracle DB Dockers 

#Create Network
Create docker network for your ADF summit application server and Database.

```docker network create -d bridge ADFSummitNET```

#Build DB Docker Image
```docker build -t ancapit/summitdb .```
#Start DB Docker Container
```docker run -d -p 1521:1521 --network=InfraNET -p 5500:5500 --name summit-db ancapit/summitdb```

#Get your application ear 
Follow our blog post to generate Summit APP ear file, that is ready to deploy in Glassfish Server.
https://ancapit.com/deploying-oracle-adf-summit-core-sample-to-glassfish/

After that copy the generated .ear file to this projects **summit-server/webapps** folder
#Build Glassfish Docker Image
```docker build -t ancapit/summit-server .```

#Start Glassfish Docker Container
```docker run -d -p 8080:8080 --network=InfraNET --name summit-server ancapit/summit-server```

Now you should be able to access http://localhost:8080/SummitADF-ViewController-context-root/faces/index.jsf and see something like this in your browser.