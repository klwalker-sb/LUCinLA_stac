# Connecting to the ERI cluster
================================================================================================================================

The Earth Research Institute hosts a series of "fat-node" linux systems to facilitate large-scale computing.
To learn more about the computing resources at ERI, see the [ERI IT Page](https://www.eri.ucsb.edu/it-resources).
For a more comprehensive overview of cluster computing, see these [hpc carpentry lessons](http://www.hpc-carpentry.org/hpc-shell).

## Account setup

To join this computing environment, you will first need to **request an ERI account** This can be done through Kendra Walker to make sure you are added to the appropriate spaces. Once you receive your credentials from ERI, make sure to store your password in a secure manner where you will remember it (you do not get to choose or change your password).

You will also need a **github account** so that you can be invited to the necessary code repositories. Send your github username to Jordan Graesser so that he can invite you to the repositories (make sure to accept the invitations). If you do not already have a github account, you can create one here: [GitHub home](https://github.com/). If you use your UCSB email, you will then qualify for a free student account, which means you can invite more than three people to collaborate in any repositories you create. You can request [student benefits here](https://education.github.com/). Click on "Get Benefits" in the upper right corner, then click the blue "get student benefits" box. Fill out the form and press "continue". 

## Connect to VPN

If you are not on campus, you will need to connect to the UCSB network through a VPN (Pulse Secure). 
If you are not already set up with a VPN, see [ucsb's it page for vpn setup](https://www.it.ucsb.edu/pulse-secure-campus-vpn/get-connected-vpn).

## ssh into the bellows server on the ERI cluster

The CEL project space resides on the bellows server, which is first reached through a general ERI gateway.  
This means you will need to ssh twice into the system to reach our space:

```
ssh [username]@ssh.eri.ucsb.edu
#enter password at prompt
ssh [username]@bellows.eri.ucsb.edu
#enter password at prompt
```

**If using windows,** it is easiest to connect to the cluster via an ssh (Secure SHell) client such as [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/), [Xming](http://sourceforge.net/projects/xming/), or [MobaXterm](https://mobaxterm.mobatek.net/). A Putty conneciton simulates the remote terminal view, and thus requires fully command-line based interfacing. MobaXterm, on the hand, offers ssh/ftp hybrid that allows for both command-line and graphical interfacing; this may be preferable to those who are more confortable manipulating files visually. Note, [X2go](https://wiki.x2go.org/doku.php/download:start) is also an option that provides a graphics viewer. See the CSC instructions [here](http://csc.cnsi.ucsb.edu/docs/using-x2go-gui-login-knot-or-pod) (you will need to modify these for the ERI login).

| **To configure Putty:**  |                       |                                          |
| :------------------------------------: | :------------------------------------: | :-------------------------------------: |
|   ![](/Images/Putty1.jpg)              |  ![](/Images/Putty2.jpg)               |   ![](/Images/Putty3.jpg)               |
|**1)** Host name=ssh1. Give configuration a name and save  |**2)** Click "SSH" on left. Remote Command=ssh2 |**3)** Click "Session" on left, "Save", then "Open" |         
You will be prompted for your password twice.
Next time you log in via Putty, just load your configuration by its name and "Open".

|**To configure MobaXterm:** |                                 |
| :---------------------------------------: | :---------------------: |
|   ![](/Images/Moba1.png)                  | ![](/Images/Moba2.png) |
| Click new session icon in top left corner |  **1)** Click "SSH" in top left corner. **2)** In "Remote Host" enter [username]@ssh.eri.ucsb.edu **3)** Open "Advanced SSH Settings" **4)** In "Execute Command" enter ssh [username]@bellows.eri.ucsb.edu **5)** Click Ok. |
You will be prompted to enter your password twice.
Next time you log in via MobaXterm, just double click your session name.
