  
[![Docker Build Status](https://img.shields.io/docker/build/rajatjain1997/layered-3d-graphs.svg)](https://hub.docker.com/r/rajatjain1997/layered-3d-graphs)
[![GitHub license](https://img.shields.io/badge/license-AGPLv3-blue.svg)](https://raw.githubusercontent.com/rajatjain1997/Layered-3D-Graphs/master/LICENSE)

# Layered-3D-Graphs

Layered-3D-Graphs plots 3D force directed and layered graphs on the basis of a **pre-requisite of** relationship between nodes.

## Quick Reference

 - **Where to file issues:**
 	- https://github.com/rajatjain1997/Layered-3D-Graphs/issues

- **Maintained By:**
	- [Rajat Jain](https://github.com/rajatjain1997)
	- [Rohit Ghivdonde](https://github.com/RohitG28)

## Introduction

The graph is rendered using the following sequence of steps:

1. Read the data and initialize each node to **(x = 0, y = 0, layer = 1)**, where layer is plotted on the z-axis.

2. Calculate the layer of each node using the "pre-requisite of" relationship. Each node has a layer assigned to it such that, 
	* No two nodes related by an edge are on the same level.
	* For each node at layer n, there exists at least one node at layer n-1 which is related to the aforementioned node.

3. Apply electrostatic repulsive forces between each node in a particular layer and give each edge a spring force.

4. Store the graph in a database, so that if the same dataset is needed, calculations need not be done.

5. Render the graph with the coordinates (x, y, layer) calculated in step 3.

Layered-3D-Graphs can scale to huge datasets easily, with most of the time taken only in serializing the data to the database, and constructing indices.

## Installation

Layered-3D-Graphs is available as a docker image. Pull the image from docker hub first by:

	docker pull rajatjain1997/layered-3d-graphs

The following are the ports used by the image:

Container Port | Host Port | Task
---- | ---- | -------------------
8000 | ANY  | Dataset processing
4000 | 4000 | Graph Plotter

To run the image, use:

	docker run \
		-p 4000:4000 -p 8000:8000 \
		--volume=$HOME/Layered-3D-Graphs/data:/code/data \
		--name=layered-3d-graphs layered-3d-graphs

This binds the data volume of the container to the host machine to allow for easy input of data.

## Configuration

By default, Layered-3D-Graphs visualizations are only accessible through localhost. However, easy configuration options are provided to make it available on local or wide area networks.

Configuration File | Option | Function | Default
------------------ | ------ | -------- | -------
server.json        | host   | IP of the application, set it to wherever Layered-3D-Graphs is to be hosted | localhost
server.json        | plotter_mapping   | The port to which the plotter is mapped outside the container    | 4000

For persistent configuration, you need to copy the `\code\config` directory on your local machine and bind it to a new instance of the container.

**NOTE:** After configuration, the container needs to be restarted.

## Creating/Preparing Data

Layered-3D-Graphs accepts JSON format graphs, with the key **"nodes"** holding an array of all vertix objects and **"links"** holding an array denoting the relationships.

Each node in **nodes** should have the following keys:
- id = *Unique ID for the node*
- name = *Name of the node*

Each relationship in **links** should have the following keys:
- source = *The node from which the link orginates*
- destination = *The node on which the link ends*

Once prepared, the data file can be placed in the docker container's **data** volume (bound at *$HOME/Layered-3D-Graphs/data*) and can be plotted by visiting the web interface at

	localhost:8000/?data="FILE_NAME"

Layered-3D-Graphs has been tested with more than 30000 nodes and 70000 edges! It's easy to plot big graphs!

## UI

Once plotting is completed, Layered-3D-Graphs offers a simple web-interface which allows the user to search and plot the dependencies and parents of each node in the graph by either doing a **full text search** or by **clicking any node in the graph**. The results, once computed, are available in a sub-graph.

Both the graphs and the subgraphs are fully interactive and support the search functionality.

The graph is persistently stored in a database inside the container. Therefore, it can be generated again without recalculations by visiting

	localhost:4000

## Backend Design

The backend is written keeping big graphs in mind, and employs the use of nodejs, redis, neo4j and shiny. Each component achieves separate purposes, as described below.

- **Nodejs:** The main page is hosted on an express app, which does all the calculations related to forces on the graph on the client machine and sends the calculated information to the server. This information is sent through sockets as they maintain persistent connections and are thus able to consistently post messages to the server without the overhead of sending the headers and waiting for the response per request. 	

- **Redis:** Loading all the data in neo4j takes time as queries on databases are slower than the speed at which the server is receiving the database insertion requests. Kue is used to maintain an asynchronous queue on redis which acts as a buffer from the server where queries are stored until neo4j is ready to execute them.

- **Neo4j:** Neo4j is the database that is used to store persistent information about the last graph calculated by Layered-3D-Graphs. This helps in fast access of the last graph generated any number of times at the cost of the initial time taken in order to construct the database.

- **Shiny:** The shiny server uses the power of plotly to batch plot the entire graph present in the database. It also hosts the search engine and the entire UI of the graph rendering page, and if directly visited skips the recalculations.

## Use Cases



## License

All Layered-3D-Graphs source code is made available under the terms of the GNU Affero Public License (GNU AGPLv3).