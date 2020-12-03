3. Validating the Lab Orchestrator Installation

After installing multipass and restarting your workstation, the installation can be validated by running the following commands:
```
multipass launch -n test1
```
Once the command is completed, the output should be the following:
```
Launched: test1
```
If your virtual machine did not launch, multipass allows you to inspect information about the instance.
```
multipass info test1
```
The info command for multipass can give information about the test1 instance.

If the instance is still starting, on some platforms multipass has issues starting the VM itself as part of the launch. We can force-start the instance by using the start command.
```
multipass start --all
```
If the VM does not show as having an IP address then there may be an incompatibility between multipass and your host network. If your network is slow then sometimes the multipass launch command can timeout and report failure while downloading the VM image, even though the VM creation is still in progress and may yet succeed.

Once the virtual machine has launched, try to execute a shell for test1.
```
multipass shell test1
```
A “Message of the Day” should display, with a command prompt at the very bottom:
```

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
ubuntu@test1:~$
```
At this prompt, you can type exit to log-out.
```
exit
``
After the launch is successful of the test1 instance, delete and purge this instance.
```
multipass delete test1
multipass purge
```
