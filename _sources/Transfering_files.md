(Transfering)=
# Transfering Files to/from cluster
================================================================================================================================

## via command line
**On a Mac**, you can transfer small files via the command line using `rsync`. You can [learn more about rsync here](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories), or [here to see all command options](https://linux.die.net/man/1/rsync)
:::{admonition}These rsync commands are run from a separate terminal 
(you do not need to close your open terminal, but open a second one from your computer to run these)
:::
#To Download a file from ERI to local:
```
rsync -raz --progress <username>@ssh.grit.ucsb.edu:<ERI path/fileName.ext> <local path/fileName.ext>
```
#To Upload a file from local to ERI
```
rsync <local path/fileName.ext> -raz --progress <username>@ssh.grit.ucsb.edu:<ERI path/fileName.ext>
```
**On Windows**, rsync or a similar file sync tool can be set up on a bash command interface such as git bash or cygwin, but the setup is a bit dodgy and unstable. A more robust option for Windows is to use an all-in-one SSH/FTP client like MobaXterm, or to add an FTP client to the mix.

## via FTP client
You can also transfer files using a File Transfer Protocol (FTP). If you are already using MobaXterm, you have an FTP included. Just double-click files from the file tree on the left to open them, or drag them onto your computer to copy them. You can also use a dedicated FTP such as WinSCP. ([Get WinSCP here](https://winscp.net/eng/index.php)). There are also FTP options for Macs, such as [FileZilla(]https://filezilla-project.org/).

As with SSH, you must be logged into a UCSB VPN client first.
| Example with WinSCP2:                               |                               |
| :------------------------------------: | :------------------------------------: |
|   ![](/Images/WinSCP2.png)             |  ![](/Images/WinSCP3.png)              |
| 1 - In "New Site", enter username and address for bellows. Then click "Advanced" | 2 - In "Tunnel", enter unsername and address for ERI. Click "OK" and save config. |
You will be prompted to enter your password twice.
Now you have a visual directory map from which you can transfer files between your PC and the Cluster

```{figure} /Images/WinSCP4.png
:scale: 20%
:name: WinSCP4.png
```

## via Globus online
For large file transfers, globus-online is recommended, as it will pick up where it left off if your connection gets interrupted. CSC has a good description of globus-online and getting set up [on their website here](http://csc.cnsi.ucsb.edu/docs/globus-online)


#  Copy processing scripts
Copy the four primary processing scripts to the folder from which you will be submitting commands(home/\<username\>/code/bash).
    Replace \<country\> and \<cntry\> with text that corresponds to the country you are working in:
    * For Paraguay: \<\country\> = paraguay and \<cntry\> = py
    * For Uruguay: \<country\> = uruguay and \<cntry\> = ury
    * For Chile: \<country\> = chile and \<cntry\> = chile
```
#if it doesn't already exist, Make code/bash directory:
mkdir ~/code/bash
#Copy scripts over:
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eostac_dl_<cntry>.sh ~/code/bash/eostac_dl_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eostac_brdf_<cntry>.sh ~/code/bash/eostac_brdf_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eostac_pipe_<cntry>.sh ~/code/bash/eostac_pipe_<cntry>.sh
cp /raid-cel/sandbox/sandbox-cel/<country>_lc/templates/eostac_pipe_ts_<cntry>.sh ~/code/bash/eostac_pipe_ts_<cntry>.sh
```   
:::{note} If you get an error that 'raid-cel/sandbox/sandbox_cel....' does not exist:
    try `/home/sandbox-cel` in the place of `/raid-cel/sandbox/sandbox-cel`
:::

