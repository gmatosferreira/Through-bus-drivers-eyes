# Through the Bus Driver’s Eye: Linking Operational Data and Driver Perspectives to Bus Services Monitoring and Planning

By [Gonçalo Matos](https://ushift.tecnico.ulisboa.pt/team-goncalo-matos/)

Thesis to obtain the Master of Science Degree in Transportation Systems

Supervised by [Professor Filipe Moura](https://ushift.tecnico.ulisboa.pt/team-filipe-moura/) and [Doctor Rosa Félix](https://ushift.tecnico.ulisboa.pt/team-rosa-felix/)

October 2025



## Abstract

Bus transit services are essential to urban mobility, yet in many cities they are perceived as unreliable and inefficient. This thesis focuses on this problem, trying to understand it from a perspective that has received little attention by the literature: through the bus drivers' eyes. The research explores how drivers' tacit insights can complement explicit operational data to enhance network monitoring and planning. Semi-structured interviews with controllers from different cities provided an overview of the state-of-the-practice, followed by a case study of Lisbon's bus operator, Carris. A focus group with eight bus drivers revealed their perceptions on performance, factors impacting it and improvement opportunities. Thematic and ranking analyses were conducted to identify their priorities and awareness across operational, tactical, and strategic planning dimensions. In parallel, more than 270,000 operational messages and 32 million events from Carris' operating assistance system were analysed using text mining and spatial aggregation, and their interrelations examined to contextualize drivers' observations and identify recurrent operational patterns. Results demonstrate that this methodology can improve communication efficiency in operational monitoring. Also, it shows that drivers hold valuable knowledge about systemic inefficiencies (such as unrealistic schedules, inadequate infrastructure or ineffective communication systems) that align with operational data patterns. The study concludes that integrating drivers' feedback and field observations can strengthen and inform monitoring and planning decisions. It advocates for a participatory framework where human and technological resources complement each other, bridging the gap between quantitative monitoring and qualitative operational understanding.


## Keywords

Transit monitoring; transit policy; transit network planning; bus drivers; stakeholder engagement; Carris



## Repository structure

This repository aims to disseminate the code used in the exploratory data analysis (EDA) of the operational data provided by the case study 
bus operator, Carris. It covers two data sets, one for messages (a log of all the messages exchanged between drivers and controllers during operations)
and another for events (a log of vehicle positions and operational events, such as stop detection, door opening/closing, etc.).

Messages were processed using text mining techniques to identify the main topics ([Categorizing Free Text Messages.md](./Categorizing%20Free%20Text%20Messages.md)),
and their geographical distribution analysed to identify the spatial dynamics associated with which message category ([Messages: Identifying Spatial Dynamics.md](./Messages:%20Identifying%20Spatial%20Dynamics.md)).

Events were first aggregated to identify trips and characterize the main operational occurrences ([Trip Identification Algorithm.md](./Trip%20Identification%20Algorithm.md)),
over which several analysis were performed, namely the determination of operational performance indicators ([Events: Reliability Buffer Index.md](./Events:%20Reliability%20Buffer%20Index.md); [Events: Bus Bunching.md](./Events:%20Bus%20Bunching.md); [Events: Short Turnings.md](./Events:%20Short%20Turnings.md))
and also the correlation between message categories and trips characteristics ([Associating Messages to Events.md](./Associating%20Messages%20to%20Events.md)).


