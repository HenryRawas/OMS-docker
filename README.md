# Trying the container solution pack for Microsoft Operations Management Suite

## What is the Microsoft Operations Management Suite?

The Microsoft Operations Management Suite (OMS) is a software-as-a-service offering from Microsoft that allows Enterprise IT to manage any hybrid cloud. It offers log analytics, automation, backup and recovery, and security and compliance.  Sign up for a free account at [http://mms.microsoft.com](http://mms.microsoft.com) or read more about here: [https://www.microsoft.com/en-us/server-cloud/operations-management-suite/overview.aspx](https://www.microsoft.com/en-us/server-cloud/operations-management-suite/overview.aspx).

## What can you do with the container solution pack?

With this feature, you&#39;ll be able to:

- See information about all container hosts in a single location
- Know which containers are running, what image they're running, and where they're running
- See an audit trail for actions on containers
- Troubleshoot by viewing and searching centralized logs without remoting to the Docker hosts
- Find containers that may be &quot;noisy neighbors&quot; and consuming excess resources on a host
- View centralized CPU, memory, storage, and network usage and performance information for containers

## Joining the private preview

You must be a member of the private preview to use this feature. To join, drop us a line at OMSContainers@microsoft.com.

## Setting up

Your container hosts must be running before setup:

- Docker 1.8 and above
- An x64 version of Ubuntu, CoreOS, Amazon Linux, SUSE 13.2, CentOS 7, or SLES 12

You have two choices for how to capture your container information. You can use OMS for all containers on a container host, or designate specific containers to send information to OMS.

### To use OMS for all containers on a container host

1. Edit `/etc/default/docker` and add this line:
``` 
DOCKER_OPTS="--log-driver=fluentd --log-opt fluentd-address=localhost:25225"
```
2. Save the file and then restart the docker service:
```
service docker restart
```
3. Start the OMS container:
```
$>sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -e WSID="your workspace id" -e KEY="your key" -h=`hostname` -p 127.0.0.1:25224:25224/udp -p 127.0.0.1:25225:25225 --name="omsagent" --log-driver=none microsoft/oms
```
Then start containers you&#39;d like to be monitored.

### To use OMS for specific containers on a host

1. Start the OMS container:
```
$>sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -e WSID="your workspace id" -e KEY="your key" -h=`hostname` -p 127.0.0.1:25224:25224/udp -p 127.0.0.1:25225:25225 --name="omsagent" --log-driver=fluentd --log-opt fluentd-address=localhost:25225 microsoft/oms
```
### If you are switching from the installed agent to the container

If you previously used the directly installed agent and want to switch to using the container, you must remove the omsagent first by running the installer with the –purge option.

## What now?
Once you’re set up, we’d like you to try the following scenarios and play around with the system. What works? What is missing? What else do you need for this to be useful for you? Let us know at OMSContainers@microsoft.com.

### Overview
Look at the Container top tile – it’s intended to show you a quick overview of the system. Does it contain the information you need to see first? If not, tell us what you expect to see instead.

![DockerOverviewTile](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerOverviewTile.png)

The top tile shows hosts that are overwhelmed with CPU or Memory usage (>90%), as well as an overview of how many containers you have in the environment and whether they’re failed, running, or stopped. 

### Dashboard view 
Click the solution pack tile. From there you'll see views organized by: 
* Containers by image
* 	Host
*	Errors
*	Audit Trail
The container solutions pack works by collecting various performance metrics and log data and sending it to the Operations Management Suite service. Each pane you see on the UI is a visual representation of a search that is run on this data.

**Try it:** Click on the top tile of this pane.
 ![DockerHostsPicture](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerContainerHosts.png)
 
You should see something like this:

![DockerHostsSearchView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerContainerInventorySearch.png?raw=true)

From here you can edit the search query to modify it to something specific.  For a tutorial on the basics of OMS search, check out the [OMS log search tutorial](https://technet.microsoft.com/library/mt484120.aspx).

**Try it:** Modify the search query so that it shows you all the stopped containers instead of the running containers by changing Running to Stopped in the search box. 

### Finding a failed container
OMS will mark a container as Failed if it has exited with a non-zero exit code. You can see an overview of the errors and failures in the environment in this tile: 

![DockerFailedContainerView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerFailedContainerView.png?raw=true)

**Try it:** Get specifics of a failed container by clicking on the tile. You'll see something like this: 
 
![DockerFailedContainerSearchView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerFailedContainerSearchView.png?raw=true)
 
From here, click on one of the image names to get additional information such as image size and number of stopped and failed images. Expand the “show more” to get the image ID: 
 
![DockerContainerExpandedView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerContainerExpandedView.png?raw=true)

**Try it:** Find the container that is running this image. Type the following into the search box:  
```
Type=ContainerInventory <ImageID>
```
This will show you the logs and you can scroll to see the failed container: 

![DockerContainerFailedSearchView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerContainerFailedSearchView.png?raw=true)
 
### Search Logs
When you're troubleshooting a specific error, it can help to see where it is occurring in your environment. Become familiar with the types of logs so you can construct queries to get the information you want:

* **ContainerInventory** – Use this type when you’re want information about where containers are located, what their names are, and what images they’re running. 
* **ContainerImageInventory** – Use this type when you’re trying to find information organized by image and to get image information such as image IDs or sizes. 
* **ContainerLog** – Use this type when you want to find specific error log information and entries.
* **ContainerServiceLog** – Use this type when you’re trying to find audit trail information for the Docker daemon, such as start, stop, delete or pull commands. 

**Try it:** Pick an image that you know has failed recently and find the error logs for it. Start by finding a container name that is running that image with a ContainerInventory search: 

```
Type=ContainerInventory drupal Failed
```

![DockerDrupalFailedSearchView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerDrupalFailedSearchView.png?raw=true)

Note the name of the container under “Name”, and do a search for those logs. In our case, it would be `Type=ContainerLog prickly_varahamihira.`

### View performance information
When you're beginning to construct queries, it can help to see what's possible first. For example, to see all performance data, try a broad query by typing the following into the search box: 
```
Type=Perf *
```

![DockerPerfSearchView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerPerfSearchView.png?raw=true)

You can see this in a more graphical form if you click the word "Metrics" on the upper right:

![DockerPerfMetricsView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerPerfMetricsView.png?raw=true)

**Try it:** Scope the performance data you're seeing to a specific container by adding typing the name of it to the right of your query:
```
Type=Perf <containerName> 
```

![DockerPerfContainerMetricsView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerPerfMetricsContainerView.png)

Scroll around to see the list of which performance metrics are collected for an individual container. 

## Example queries
Finally, sometimes it can help to build queries by beginning with an example or two and adjusting to fit your environment. Play around with the links on the Notable Queries page (on the far right) to help you build more advanced queries: 

![DockerNotableQueries](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerNotableQueries.png?raw=true)

## Saving queries
Saving queries is a standard feature in OMS and can help you keep queries you've found useful.  

**Try it:** After you construct a query you find useful, save it by clicking the "Save" at the top. This will let you easily access it later from the My Dashboard page.

![DockerDashboardView](https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/docs/pictures/DockerPics/DockerDashboardView.png?raw=true)

If you've made it this far, thanks a bunch. Drop us a line at OMSContainers@microsoft.com and let us know you made it through - tell us what works, what doesn't, and what we need to build next. 
