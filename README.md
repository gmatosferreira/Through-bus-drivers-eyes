# Through the Bus Driver’s Eye: Linking Operational Data and Driver Perspectives to Bus Services Monitoring and Planning

By [Gonçalo Matos](https://ushift.tecnico.ulisboa.pt/team-goncalo-matos/)

Thesis to obtain the Master of Science Degree in Transportation Systems

Supervised by [Professor Filipe Moura](https://ushift.tecnico.ulisboa.pt/team-filipe-moura/) and [Doctor Rosa Félix](https://ushift.tecnico.ulisboa.pt/team-rosa-felix/)

October 2025



## Abstract

...


## Keywords

Transit monitoring; transit policy; transit network planning; bus drivers; stakeholder engagement; Carris



## Repository structure

This repository aims to disseminate the code used in the exploratory data analysis (EDA) of the operational data provided by the case study 
bus operator, Carris. It covers two data sets, one for messages (a log of all the messages exchanged between drivers and controllers during operations)
and another for events (a log of vehicle positions and operational events, such as stop detection, door opening/closing, etc.).

Messages were processed using text mining techniques to identify the main topics ([Categorizing Free Text Messages.md](./Categorizing Free Text Messages.md)),
and their geographical distribution analysed to identify the spatial dynamics associated with which message category ([Messages: Identifying Spatial Dynamics.md](./Messages: Identifying Spatial Dynamics.md)).

Events were first aggregated to identify trips and characterize the main operational occurrences ([Trip Identification Algorithm.md](./Trip Identification Algorithm.md)),
over which several analysis were performed, namely the determination of operational performance indicators ([Events: Reliability Buffer Index.md](./Events: Reliability Buffer Index.md); [Events: Bus Bunching.md](./Events: Bus Bunching.md); [Events: Short Turnings.md](./Events: Short Turnings.md))
and also the correlation between message categories and trips characteristics ([Associating Messages to Events.md](./Associating Messages to Events.md)).


