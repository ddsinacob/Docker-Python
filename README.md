# Docker-Python
Python CGI with Docker


A simple python script which can be served as CGI script using Apache2 in Ubuntu Docker Container. Demo app is deployed at GCP.

Create a Dockerbuild file
-------------------------

      
        FROM ubuntu:17.10
        RUN apt-get update -y; \
            apt-get install -y apache2; \
            a2enmod cgi; \
            apt-get install python2.7 -y; \
            apt-get install python-pip -y; \
            apt-get install python2.7-dev -y; \
            pip install requests;
        ENTRYPOINT service apache2 start && /bin/bash
        EXPOSE 80
        
        
Build the Image
---------------

        docker build -t ubuntu17 .
        
   
        [root@instance-2 testing]# docker images
        REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
        ubuntu17            latest              b6c34fda4057        4 minutes ago       477MB
        
        
        
Prepare the CGI script
----------------------
    Script name : myapp , permission : chmod +x myapp

     
        #!/usr/bin/env python2.7
        import os
        import requests
        import json

        class Python_CGI(object):
                """docstring for Python_CGI"""
                def __init__(self, url):
                        self.url = url

                def __call__(self):
                        res = requests.get(self.url+"?api_key=DEMO_KEY")
                        print "Content-Type: text/html"
                        print
                        print """
                        <TITLE>CGI script ! Python Weather API call</TITLE>
                        <H1>Neo - Lookup</H1>
                        <H2>MyAPP at Compute Instance in GCE</H2>
                        """+"\n Request URL : "+self.url+" \n ,status code: "+str(res.status_code)+"\n<pre>"+json.dumps(res.json(), indent=3, sort_keys=True)+"</pre>"
                        return
        Python_CGI(os.environ.get("QUERY_STRING").split("=")[1])()
        


Create and Run a Docker Container to serve CGI Script
-----------------------------------------------------
    Placingt the python script under apache2 default CGI folder . Exposing container port 80 to 80 in compute instance and added a firewall ingress rule in GCP. I am also mounting local directory cgi-bin which contins python script to /usr/lib/cgi-bin in docker container.
    

    
        docker run -d -p 80:80 -v "$(pwd)"/cgi-bin/:/usr/lib/cgi-bin ubuntu17
        

    
        [root@instance-2 testing]# docker ps
        CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                  NAMES
        c606244ddfe4        ubuntu17            "/bin/sh -c 'servi..."   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   pensive_brahmagupta
        
Test the CGI script
-------------------


  
      http://XX.XXX.XXX.XX/cgi-bin/myapp?url=https://api.nasa.gov/EPIC/api/natural/date/2015-10-31
