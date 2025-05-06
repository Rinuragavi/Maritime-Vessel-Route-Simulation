# Vessel Route Simulation Project

## Project Overview

This project simulates the movement of a vessel between two ports, generates AIS (Automatic Identification System) messages for the vessel's positions, and streams these messages over WebSocket. The system stores AIS messages in a database (e.g., SQLite) and provides analytics for the vessel's route, including distance traveled.

---

## Table of Contents

- [Installation](#installation)
- [Project Structure](#project-structure)
- [Usage](#usage)
  - [Route Generation](#route-generation)
  - [AIS Simulation](#ais-simulation)
  - [WebSocket Playback](#websocket-playback)
  - [Ingestion & Database](#ingestion--database)
  - [Analytics Queries](#analytics-queries)
- [How Simulation Works](#how-simulation-works)
- [How Ingestion Works](#how-ingestion-works)
- [Sample Queries and Outputs](#sample-queries-and-outputs)
- [Assumptions and Simplifications](#assumptions-and-simplifications).

---

## Installation

### Dependencies

To install the required dependencies, run the following command:

```bash
pip install searoute pyais websockets pandas geopy SQLAlchemy
```

## Project Structure

```
ais_simulation_project/
├── data/                          # Contains data files
│   └── ports.csv                  # Ports data file for route generation
├── src/                           # Source code
│   ├── route_generator.py         # Generates routes between ports
│   ├── ais_simulator.py           # Simulates AIS message encoding
│   ├── websocket_server.py        # WebSocket server for message streaming
│   ├── websocket_client.py        # WebSocket client for message reception
│   ├── db/                        # Database-related files
│   │   ├── models.py              # Database models
│   │   ├── ingest.py              # Data ingestion script
│   │   └── init_db.py             # Initializes the database
├── tests/                         # Test files for the project
├── README.md                      # Project documentation
└── requirements.txt               # Required Python packages
```

## Usage

### Route Generation

1. The route_generator.py module generates a route between two randomly selected ports.

2. Load ports from the data/ports.csv file.

Randomly select two ports and generate a route between them using the searoute-py library.

Example usage:

```bash
from src.route_generator import load_ports, select_ports, generate_route

ports_df = load_ports("data/ports.csv")
port1, port2 = select_ports(ports_df)
route = generate_route(port1, port2)
```

## AIS Simulation

The ais_simulator.py module simulates the vessel's movement between waypoints and generates AIS messages for each position.

1. Given a list of waypoints (latitude, longitude), the module simulates the vessel’s movement at a given speed.

2. It generates AIS position reports, encodes them using pyais, and returns them for playback or database storage.

Example usage:

```bash
from src.ais_simulator import simulate_positions, encode_ais

waypoints = [(longitude, latitude), ...]
positions = simulate_positions(waypoints, speed_knots=10)
encoded_ais = [encode_ais(pos) for pos in positions]
```

## WebSocket Playback

The websocket_server.py module streams AIS messages over WebSocket.

1. Messages are sent to clients with a specified playback speed.

2. The server listens on ws://localhost:8765 by default, but you can adjust the address and port.

Example usage:

```bash
import asyncio
from src.websocket_server import playback_server

asyncio.run(playback_server(simulated_positions, speed=2.0))
```

## Ingestion & Database

The db/init_db.py and db/ingest.py modules set up the database and ingest AIS messages.

1. The database is initialized with a schema to store AIS messages.

2. Messages are parsed, decoded, and inserted into the database.

Example usage:

```bash
from src.db.init_db import init_db
from src.db.ingest import ingest_ais_message

init_db()  # Initialize the database
# Ingest an AIS message into the database
ingest_ais_message(decoded_message)
```

## Analytics Queries

The analytics.py module contains functions to perform analytics on the stored AIS messages.

1. It can calculate the total distance traveled by a vessel, given a series of positions.

2. It uses geopy to calculate the distance between consecutive positions.

Example usage:

```bash
from src.analytics import calculate_distance

distance = calculate_distance(positions)
print(f"Total Distance Covered: {distance} nautical miles")
```

## How Simulation Works

1. Route Generation: The project loads port data, randomly selects two ports, and uses searoute-py to generate a route between them.

2. AIS Simulation: The vessel moves along the generated route, and AIS messages are generated at regular intervals, encoding the vessel's position.

3. Playback: The generated AIS messages are streamed over WebSocket, with the option to control playback speed.

4. Ingestion: Messages are received by a WebSocket client, decoded, and inserted into the database for storage and further analysis.

5. Analytics: The database can be queried to calculate the vessel's trajectory, speed, and distance traveled.

## How Ingestion Works

1. WebSocket Client: The websocket_client.py module connects to the WebSocket server and receives AIS messages.

2. AIS Message Decoding: Each received message is decoded using the pyais library, which extracts the vessel's position.

3. Database Insertion: The decoded AIS messages are inserted into the database (SQLite by default).

## Sample Queries and Outputs

1. Distance Calculation: You can query the database for the vessel's route and calculate the total distance covered.

Example query:

```bash
SELECT * FROM ais_messages WHERE mmsi = '123456789' ORDER BY timestamp;
```

2. Speed Calculation: You can query the database for speed-related data.

## Assumptions and Simplifications

1. Simplified Route Generation: Ports are selected randomly from the dataset, and the route is generated without considering factors like weather or ocean currents.

2. Single Vessel: The simulation assumes only one vessel is being simulated at a time.

3. Playback Speed: The WebSocket server assumes a fixed playback speed. Real-time simulations are not accounted for.

4. Database: The default database is SQLite. For larger-scale simulations, you may want to switch to PostgreSQL or another database.

