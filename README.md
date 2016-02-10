
## Philosophy

The ChRIS-portable VM is a fully instantiated Ubuntu Virtual Machine, complete with a ChRIS server, Orthancs PACS server, and supporting software infrastructure that all fits onto an 8GB flash drive.

In order to fit into the 8GB space, certain critical components, notable the ChRIS <tt>data</tt> and <tt>users</tt> directories are off-loaded to the host.

## Prerequisites on Host

The host must provide a generic un*x type filesystem and services -- in particular unix type UIDs and GIDs, as well as a filesystem that supports symbolic links. In practice this means the host must be either Linux or Mac for the host operating system

## Initial setup

### Unpack the <tt>extra.tgz</tt> file to the host

The <tt>extra.tgz</tt> archive contains the ChRIS <tt>data</tt> and <tt>users</tt> directories. Unpack these directories where ever convenient on the host file system, e.g.

```bash
cd ~/
mkdir chris
cd chris
tar xvfz ~/Downloads/extra.tgz
```

where we assume the <tt>extra.tgz</tt> is in the <tt>~/Downloads</tt> directory.

### Create a Virtual Box "container"

#### Basic VirtualBox settings

For the most part, the VM is a standard VirtualBox <tt>vdi</tt> file configured with 2GB RAM and 2 CPUs. More detailed settings include: 

* Type: Linux Ubuntu 64-bit
* 2048 MB RAM
* 2 CPUs
* 128 MB Video Memory
* HiDPI Support: Enabled
* Storage SATA Controller: ChRIS_portable.vdi file

**The most important setting is the storage SATA Controller. Add the ChRIS-portable.vdi file as the setting to the controller.**

#### Network Port Forwarding:

Set the following Port Forwards in the Network Tab of the VirtualBox Configuration (note: all are TCP protocol rules):

| Host Port | Guest Port |
|-----------|------------|
|   2222    |    22      |
|   8001    |    80      |
|   8042    |   8042     |
|   4242    |   4242     |
|   8888    |   8888     |

## Start the VM

Once configured, start the Virtual Box Machine. Once Ubuntu boots, you will see a standard lightdm login screen. Select the "Chris System" login and use <tt>chris321</tt> as the password.

## Mount the relevant dirs from the host

In the ChRIS VM, you will note that the <tt>data</tt> and <tt>users</tt> directories are dead symbolic links:

```bash
# In the ChRIS VM!
cd ~/
ls -ld data users
```

will result in

```bash
[chris@chris-portable:x86_64-Linux]~$>ls -ld data users
lrwxrwxrwx 1 chris chris 42 Dec 11 17:32 data -> /mnt/kyon/Users/rudolph.pienaar/chris/data
lrwxrwxrwx 1 chris chris 43 Dec 11 17:32 users -> /mnt/kyon/Users/rudolph.pienaar/chris/users
[chris@chris-portable:x86_64-Linux]~$>
```

Note that these directories (<tt>data</tt> and <tt>users</tt>) are linked to corresponding directories on the host, located at <tt>/mnt/kyon/Users/rudolph.pienaar/chris/</tt> which reflects the file tree in my specific case.

In the VM, I link to these directories using:

```bash
sudo sshfs -o uid=6244,gid=1102,allow_other rudolph.pienaar@10.0.2.2:/ /mnt/kyon
```

The <tt>10.0.2.2</tt> address is invariant. However, update the login <tt>rudolph.pienaar</tt> and paths according to your particular setup. Note that my paths reflect a Mac OS X naming convention (the users' home directory on my host is <tt>/Users</tt> while on a Linux host this would be <tt>/home</tt>.

## Startup the Orthanc PACS server

To startup the Orthancs PACS server, open a terminal on the VM -- either from the VirtualBox session directly, or log into the VM from the host using

```
ssh -p 2222 chris@localhost
```

and start the Orthancs PACS server:

```
Orthanc ~/arch/Linux64/share/Orthanc/Resources/Configuration.json
```

## Use ChRIS on the VM

To connect to ChRIS on the VM, from the **host** in a browser (preferably Chrome), connect to

```
http://localhost:8001
```

You can login with <tt>chris</tt> and <tt>chris321</tt>.

To connect to the Orthanc server from the host, do

```
http://localhost:8042
```


