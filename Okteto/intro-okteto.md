---
layout: default
title: Introduction to okteto   
parent: Okteto
nav_orde : 1
---

# Introduction to okteto 


- okteto is tool for all kubernetes Developer to build , run , debug there application without depending on heavy load on local machine . still you can develope application quick and faster . 

- you don't need anything installed on machine rather then okteto cli . remember when we use docker on your machine its take to much memory and other resources . and its very painful cycle to build docker containers again and again locally . but okteto will provide remote developement using power of okteto cloud . 

- Okteto provide Clean and Easy CLi to without much understanding complexity of docker or kubernetes clustering and swarm .

# How Okteto Works Under the hood 

- okteto use concept of syncthing . its continuous file synchronization program . which workes in real time to synchronizes file between two or more computers .
   
- how this will help, well we have discussed that its don't need any kind of addition installation to run your kubernetes application because of syncthiing its will become much easier to run application .

- but in case of okteto its just not only synchronize file . but thing this way you have used docker build or docker container run kind of command to run or rebuild docker images thats where okteto is doing things more smarter . okteto will detect yml file 

Syncthing is a continuous file synchronization program. It synchronizes files between two or more computers in real time, safely protected from prying eyes. Your data is your data alone and you deserve to choose where it is stored, whether it is shared with some third party, and how it's transmitted over the internet.



Author: Sangam Biradar , Okteto Comunity Bangalore 
