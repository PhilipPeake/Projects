#SSH Tunneling

>**NOTE**: that the examples below use port 8080 as the remote and local ports. In reality you will want to use an appropriate local port and the appropriate remote port. For Tomcat hosting web consoles, 8080 would be appropriate, for the Metrics Engine you probably want to make both ports 8443.
(images/putty-1.png)

Expand the SSH item in the left-hand panel, and select Tunnels:

![Image]
(images/putty-2.png)

Source port the port you want to tunnel on the remote system. Destination is where you want to connect this to:

![Image]
(images/putty-3.png)

Click the “Add” button, then scroll back up the left-hand panel and click on “Session”.