## What is OSRM ?  

OSRM (Open Source Routing Machine) is a high-performance routing engine designed to calculate the shortest or fastest paths in road networks. It supports various routing profiles like car, bicycle, and foot, and allows for customization to suit different needs. OSRM is known for its speed and efficiency, leveraging advanced algorithms such as contraction hierarchies for fast routing calculations. As an open-source project, it offers flexibility for developers to modify and integrate it into their applications. 
Common applications include navigation systems, logistics optimization, and mapping services.

### Preferrence of OSRM over Google API?
The cost is the main factor that will be considered for such a comparison. OSRM is a powerful open-source routing engine to solve the shortest paths in road networks. It's quite fast too!

##### OSRM API Documentation: http://project-osrm.org/docs/v5.22.0/api/#general-options

This repository focuses on deploying OSRM within a Docker container. Follow these steps to successfully deploy the OSRM.PBF file on a Docker container.

### Step 1: Download OSM.PBF file for your required region

OSRM (Open Source Routing Machine) uses PBF (Protocolbuffer Binary Format) files, which are highly efficient for storing and transferring large datasets, such as map data from OpenStreetMap (OSM). PBF files contain all the necessary geographic information required for routing and are essential for initializing the OSRM backend. By deploying these files in a Docker container, you can leverage OSRM's high-performance routing capabilities.
[Geofabrik]https://download.geofabrik.de/

Download the required PBF files from [Geofabrik](https://download.geofabrik.de/). For this example, we'll use the Denmark region file.

To download the PBF file using the Command Prompt on Windows, you can use `curl` or `wget`.
Here Denmark region OSRM.pbf file is used.


```
  wget https://download.geofabrik.de/europe/denmark-latest.osm.pbf 
```

### Step 2: Pre-process with the Vehicle profile (Car, Bike)

```
docker run -t -v "${PWD}:/data" osrm/osrm -backend osrm-extract -p /opt/car.lua /data/denmark-latest.osm.pbf
```

##### How It Works ?
The command is used to run the OSRM (Open Source Routing Machine) extraction process within a Docker container.
It initiates a Docker container using the osrm/osrm-backend image, which contains the OSRM backend software. The -t flag allocates a pseudo-terminal, making the container interactive. The -v "${PWD}:/data" option mounts the current working directory from the host machine (denoted by ${PWD}) to the /data directory inside the container, providing the container access to the host's files. Inside the container, the osrm-extract command processes the OSM (OpenStreetMap) data file denmark-latest.osm.pbf using the car routing profile specified by the -p /opt/car.lua option. This step is crucial for preparing the routing data needed by OSRM to provide efficient routing services.


### Step 3: Partitioning And Customization  

We use osrm-partition to partition the graph recursively into cells and osrm-customize customizes these cells by calculating routing weights for all cells.

```
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/Denmark-latest.osrm
```

```
docker run -t -v "${PWD}:/data" osrm/osrm-backend osrm-partition /data/denmark-latest.osrm
```

##### How It Works ?
The command is used to partition the OSRM (Open Source Routing Machine) data within a Docker container. 
It starts a Docker container using the osrm/osrm-backend image, which contains the necessary tools for OSRM. 
The -t flag allocates a pseudo-terminal to the container. The -v "${PWD}:/data" option mounts the current working directory on the host machine (${PWD}) 
to the /data directory inside the container, ensuring that the container has access to the host's files. 
Inside the container, the osrm-partition command is run on the file /data/denmark-latest.osrm. This command partitions the routing data, which is an essential step for preparing the dataset for fast and efficient route queries, 
enabling the OSRM server to handle complex routing requests quickly.


### Step 4: Run Image 

```
docker run -d -p 5000:5000 --rm --name osrm -v "${PWD}:/data" osrm/osrm-backend osrm-routed --algorithm mld /data/north-eastern-zone-latest.osrm
```

Using osrm-routed we are spawning the http server. Algorithm mld or known as Multi-Level Dijkstra is used to run to specify the algorithm.

The docker container should be up and running on the URL and test this on these positions.

```
http://localhost:5000/route/v1/driving/10.319532450675782,55.43598147121253;10.28794675779819,55.454286184745506?steps=true
```



