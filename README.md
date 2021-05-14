**Web SSH Terminals Architecture** <br/> 
![webssh (3)](https://user-images.githubusercontent.com/49838106/118291923-c8d4df00-b4e0-11eb-844c-d7fbbdf9bd57.png)


**Services Roles**
* **Session Proxy** <br/> Receiving all the sessions commands messages and return the outputs.<br/> Responsible for initialize each session environment. <br/>
* **Queues** <br/> __STDOUT Queue__ Receiving STDOUT of the SSH Shell process and pass it back to Session Proxy service (and to the client). <br/> 
__Commands Queue__ Receiving commands from clients and pass them into SSH Runner consumer. <br/> 
* **SSH Runner** <br/>Runs each SSH Session as a sub-process. <br/>Fetching (Sampeling) the STDOUT of each process and pass it into the Queue. <br/> Rececing the commands and send them into the STDIN of each process.  <br/> 
<br/>

**Notes**
* **Real Remote STDOUT** The ssh session shell is opened in another process (inside docker container). <br/>  We are pulling the STDOUT from this '/bin/shell; exit' process. <br/> It's possible also using Paramiko and skip this sub-process thing but it won't give us the PS1 and the REAL shell expreience.
* **Inner Container** of each SSH Runner provide shell session isolation from the host (backend) environment. <br/> The sub-process we opened is actually a shell which run SSH client, so if we EXIT (and somehow skip the 'exit' commnad) the SSH session we have shell on the backend SSH Runner service.<br/> Think about a smart user that will exit the SSH session and run 'reboot' command. It will reset the whole Backend!
* **Shell Output Styling** using ANSI escape code.  <br/> We use Ansi esace codes parser in the client for styling the output (colors, fonts, etc). 
* **Queues** will be implemented with RabbitMQ. They pass through a Exchange that routing the messages by their types to the matching queue.
