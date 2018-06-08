# Node.RPG
Node.RPG is a self contained web application server for the ILE environment on IBMi. 

It is a service program that provides a simple programmable HTTP server for your 
application so you easy can plug your RPG code into a services infrastructure 
or make simple web applications without the need of any third party webserver products.

Node.RPG is a HTTP application server you can bind into your own ILE RPG projects, 
to give you a easy deploy mechanism, that fits into DevOps and microservices alike environments.

The self contained web application server makes it so much easier to develop web application. 

Simply compile and submit. Yes - You don't need GCI, Apache, nginx or IceBreak - simply compile and submit.

The design paradigm is the same as found in Node.JS hence the name. Where Node.JS uses 
JavaScript, Node.RPG aims for any ILE language where RPG are the most popular.

Except for initialization, It only requires two lines of code:
```
 node_listen ( config : servletCallback); 
 node_write ( request : response);
```

The `node_liste` are listening on the TCP/IP port and interface you define in the 
config structure. For each http request it will call your "servlet" which is a 
callback procedure that takes a request and a response parameter
   
![](image.png)


The idea is that you deploy your (open source of cause) RPG packages at NPM so the RPG community can benefit from each others work. The NPM echosystem is the same for Node.JS and Node.RPG    


Example: 
```
        // -----------------------------------------------------------------------------
        // This example runs a simple servlet using Node.RPG
        // Note: It requires your RPG code to be reentrant and compiled
        // for multithreading. Each client request is handled by a seperate thread.
        // Start it:
        // SBMJOB CMD(CALL PGM(DEMO01)) JOB(NODERPG) JOBQ(QSYSNOMAX) ALWMLTTHD(*YES)        
        // -----------------------------------------------------------------------------     
        ctl-opt copyright('Sitemule.com  (C), 2018');
        ctl-opt decEdit('0,') datEdit(*YMD.) main(main);
        ctl-opt debug(*yes) bndDir('NODERPG');
        ctl-opt thread(*CONCURRENT);
        /include noderpg.inc
        // -----------------------------------------------------------------------------
        // Main
        // -----------------------------------------------------------------------------     
        dcl-proc main;

            dcl-ds config likeds(configDS);

            config.port = 44999;
            config.host = '*ANY';

            node_listen (config : %paddr(myservlet));

        end-proc;
        // -----------------------------------------------------------------------------
        // Servlet call back implementation
        // -----------------------------------------------------------------------------     
        dcl-proc myservlet;

            dcl-pi *n;
                request  likeds(REQUESTDS);
                response likeds(RESPONSEDS);
            end-pi;
  
            node_Write(response:'Hello world');

        end-proc;
```

 
# Installation
What you need before you start:

* IBMI 7.3 TR3 ( obove or alike)
* 5733OPS git and gmake
* ILE C 
* ILE RPG compiler.


From a IBMi menu prompt start the SSH deamon:`===> STRTCPSVR *SSHD`
Or start ssh from win/mac

```
mkdir /prj
cd /prj 
git -c http.sslVerify=false clone https://github.com/NielsLiisberg/Node.RPG.git
cd Node.RPG
gmake 
cd test 
gmake
```

# Test it:
Log on to your IBMi.
from a IBMi menu prompt 
````
CALL QCMD
ADDLIBLE NODERPG
SBMJOB CMD(CALL PGM(DEMO01)) ALWMLTTHD(*YES) JOB(NODE_DEMO1) JOBQ(QSYSNOMAX) 
SBMJOB CMD(CALL PGM(DEMO02)) ALWMLTTHD(*YES) JOB(NODE_DEMO2) JOBQ(QSYSNOMAX) 
````
Now test it in a browser:

* http://myibmi:44998
* http://myibmi:44999

A simple hello and list with a counter. Please note that the job requires `ALWMLTTHD(*YES)`


# Develop:
You compile the project with gmake, and I have also included a 
setup folder for vsCode so you can compile any changes 
with `Ctrl-shift-B` You need however to 
change .vsCode/task.json file to point 
to your IBMi address. The compile feature requires that you have SSH stated: `STRTCPSVR *SSHD` 

# Moving on
In this first commit we have only implemented the `node_listen` and `node_write`, so there is not much use for real world application, however - all the plumbing arround with git / compile / deploy are working. We at Sitemule.com are striving to move the core of the IceBreak server into the Node.RPG project over the next couple of months. So stay tuned.

Happy Node.RPG coding

