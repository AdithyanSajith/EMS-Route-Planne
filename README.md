EMS Route Planner
The EMS Route Planner is a smart route optimization tool designed to assist emergency medical services (EMS) in identifying the most efficient routes to hospitals. This tool factors in real-time traffic data, patient priority levels, and hospital capacity to calculate optimal paths and reduce EMS response times in critical scenarios.

Features
Dynamic Traffic Updates: Adjusts routes based on current traffic conditions to avoid delays.
Patient Priority-Based Routing: Prioritizes patients based on urgency, ensuring the most critical cases receive prompt attention.
Hospital Capacity Tracking: Monitors available capacity at nearby hospitals to find the best-suited facility.
A Search Algorithm:* Utilizes the A* algorithm to find the shortest paths, balancing speed and route accuracy.
Constraint Satisfaction Problem (CSP) Techniques: Handles complex constraints, such as patient priorities and hospital availability.
Google Maps Integration: Provides a visual representation of routes for real-time navigation assistance.
Project Structure
Classes

Node: Represents locations (hospitals, other areas) with properties like neighbors and traffic level.
Patient: Contains details about each patient, including priority and location.
RoutePlanner: Main class handling nodes, patients, traffic updates, route calculation, and EMS dispatch.
Key Functions

a_star_search: Implements the A* algorithm to compute optimal paths.
find_best_route: Finds the best route to a hospital based on traffic, capacity, and patient priority.
dispatch_ems: Sorts patients by priority and dispatches EMS units along the calculated best routes.
