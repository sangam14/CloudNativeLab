---
layout: default
title: Allows Seamless Upgrades
parent: Docker For Developer
nav_order: 4
---

# Allows Seamless Upgrades

Even in scaled-up scenarios, a container-based approach makes tricky concepts seem trivial. Without containers, 
your favorite admin will not be happy with you if he has to update every server, including the dependencies.

![Every server needs to be updated](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/everyserver.png)

Of course, in such a case, the update process depends on the application and its dependencies. Don’t even try to tell your admins about DevOps if you want 
to remain alive. By using containers, it’s a simple matter of telling the orchestrator that you want to run a new image version,
and it gradually replaces every container with another one running the new version. Whatever technology stack is running inside the containers,
telling the orchestrator that you want to run a new image version is a simple command

The illustration below shows the process as it goes on: the orchestrator replaces one container, and then moves on to the other ones. 
While the new container is not ready, traffic is being routed to the old version containers so that there is no interruption of service.

![Rerouting traffic to the updated container](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/WordpressupdatedContainer.png)



