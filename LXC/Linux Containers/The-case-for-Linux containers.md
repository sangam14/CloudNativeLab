# The case for Linux containers

- The hypervisor as part of the operating system is responsible for managing the life cycle of virtual machines, and has been around since the early days of mainframe machines in the late 1960s. Most modern virtualization implementations, such as Xen and KVM, can trace their origins back to that era. The main reason for the wide adoption of these virtualization technologies around 2005 was the need to better control and utilize the ever-growing clusters of compute resources. The inherited security of having an extra layer between the virtual machine and the host OS was a good selling point for the security minded, though as with any other newly adopted technology there were security incidents.

- Nevertheless, the adoption of full virtualization and paravirtulization significantly improved the way servers are utilized and applications provisioned. In fact, virtualization such as KVM and Xen is still widely used today, especially in multitenant clouds and cloud technologies such as OpenStack.

- Hypervisors provide the following benefits, in the context of the problems outlined earlier:

   - Ability to run different operating systems on the same physical server

   - More granular control over resource allocation

   - Process isolation – a kernel panic on the virtual machine will not effect the host OS

   - Separate network stack and the ability to control traffic per virtual machine

   - Reduce capital and operating cost, by simplification of data center management and better utilization of available server resources
   
- Arguably the main reason against using any sort of virtualization technology today is the inherited overhead of using multiple kernels in the same OS. It would be much better, in terms of complexity, if the host OS can provide this level of isolation, without the need for hardware extensions in the CPU, or the use of emulation software such as QEMU, or even kernel modules such as KVM. Running an entire operating system on a virtual machine, just to achieve a level of confinement for a single web server, is not the most efficient allocation of resources.

- Over the last decade, various improvements to the Linux kernel were made to allow for similar functionality, but with less overhead – most notably the kernel namespaces and cgroups. One of the first notable technologies to leverage those changes was LXC, since kernel 2.6.24 and around the 2008 time frame. Even though LXC is not the oldest container technology, it helped fuel the container revolution we see today.

- The main benefits of using LXC include:

    - Lesser overheads and complexity than running a hypervisor

    - Smaller footprint per container

    -  Start times in the millisecond range

   - Native kernel support
