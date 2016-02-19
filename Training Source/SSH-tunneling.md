#SSH TunnelingThis document is not intended to be a full description of ssh tunneling, simply enough to allow forwarding of ports on training VMs inaccessible via the Internet, to your local workstation/laptop.

>**NOTE**: that the examples below use port 8080 as the remote and local ports. In reality you will want to use an appropriate local port and the appropriate remote port. For Tomcat hosting web consoles, 8080 would be appropriate, for the Metrics Engine you probably want to make both ports 8443.##OpenSSH (Unix/Linux/Mac OSX)Setting up a forward tunnel with OpenSSH is as simple as adding command-line options. For example, we need to access a webserver running on port 8080 on a remote system with port 8080 closed to the Internet. We want to create a tunnel from port 8080 on our local machine to port 8080 on the remote machine:	$ ssh -L 8080:localhost:8080 ubid@pong1.unboundid.netThe –L option is parsed as follows: The first component is the port to open on the local machine (8080). The second component is where to “connect” this tunnel to, interpreted with respect to the machine we are connecting to. In this case, port 8080 on localhost (localhost:8080).As long as the ssh session is running, it will ferry encrypted data between port 8080 on the local system and port 8080 on the remote system.If you wanted to use a different port locally, adjust as necessary. E.g. to use port 9090 locally the command would be:	$ ssh -L 9090:localhost:8080 ubid@pong1.unboundid.netIt is possible to supply multiple tunnel arguments. For example to do the above but also make port 1389 on the remote machine available on the local machine:	$ ssh -L 8080:localhost:8080 \	-L 1389:localhost:1389 ubid@pong1.unboundid.net##PUTTYPUTTY is the most common ssh client used on Windows systems, it has a graphical interface which some may find somewhat inconsistent and confusing.Setting up: Open PUTTY and the window it displays looks like this:![Image]
(images/putty-1.png)

Expand the SSH item in the left-hand panel, and select Tunnels:

![Image]
(images/putty-2.png)

Source port the port you want to tunnel on the remote system. Destination is where you want to connect this to:

![Image]
(images/putty-3.png)

Click the “Add” button, then scroll back up the left-hand panel and click on “Session”.Add more tunnels and add as required.From there, enter the name/IP of the remote system and login. The tunnel will be established at that point.