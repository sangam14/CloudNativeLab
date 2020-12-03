Stopping and Resuming your Lab

he multipass utility offers stop and start functionality for safely freezing and resuming instances. You can use this if desired to free up resources from your lab host machine while you aren’t working on the course. 

If you haven’t already done so, you can exit the host1 instance by using the exit command.

exit
Stop

Stopping instances is the recommended way to manage your lab. This allows you to save your progress, and later continue from where you left off.  To do this stop all instances with the following command:
```
multipass stop --all
```
On some platforms, the stop action will crash and not proceed with shutting the instances down. This is an issue with Multipass that we expect to be resolved.

List

To confirm the stop has successfully completed, you can view the state of the instances managed by multipass using the list command:
```
multipass list
Example output:

Name                    State             IPv4             Image
control                 Stopped           --               Ubuntu 20.04 LTS
host1                   Stopped           --               Ubuntu 20.04 LTS
node1                   Stopped           --               Ubuntu 20.04 LTS
node2                   Stopped           --               Ubuntu 20.04 LTS
Start
```
When you are ready to resume the course, you can simply start the instances from a suspended, or stopped state. The commands to start all instances can be found below. We recommend starting the instances in this order to cause minimum disruption to the Kubernetes control plane.
```
multipass start control
multipass start node1
multipass start node2
multipass start host1
```
Special Note Regarding Windows VMware Interactions

On Windows, if you're running an old version of VMware Workstation/Player you may find that VMware can not start your VMs after using multipass due to its use of Hyper-V. This has been fixed in the latest version of VMware Workstation and Windows 10. To workaround this issue, after stopping your multipass VMs, you can switch the Hyper-V feature off using the following command:

bcdedit /set hypervisorlaunchtype off
After rebooting your computer, VMware should now be able to launch your Virtual Machines once again. To toggle the feature back on again to resume your certification lab you can use the following command:

bcdedit /set hypervisorlaunchtype auto
After rebooting your computer once again, you can start the instances in multipass as normal and continue the certification program labs.
