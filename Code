import heapq
import webbrowser

class Node:
    def __init__(self, name, is_hospital=False, capacity=0):
        self.name = name
        self.is_hospital = is_hospital
        self.capacity = capacity
        self.neighbors = {}

    def add_neighbor(self, neighbor, distance, traffic_level):
        self.neighbors[neighbor] = (distance, traffic_level)

class Patient:
    def __init__(self, id, location, priority):
        self.id = id
        self.location = location
        self.priority = priority

class RoutePlanner:
    def __init__(self):
        self.nodes = {}
        self.patients = []
        self.traffic_data = {}

    def add_node(self, name, is_hospital=False, capacity=0):
        node = Node(name, is_hospital, capacity)
        self.nodes[name] = node
        return node

    def add_road(self, start, end, distance, traffic_level):
        if start in self.nodes and end in self.nodes:
            self.nodes[start].add_neighbor(end, distance, traffic_level)
            self.nodes[end].add_neighbor(start, distance, traffic_level)

    def a_star_search(self, start, goal):
        open_set = [(0, start)]
        g_score = {node: float('inf') for node in self.nodes}
        g_score[start] = 0
        f_score = {node: float('inf') for node in self.nodes}
        f_score[start] = g_score[start] + self.heuristic(start, goal)

        came_from = {}
        while open_set:
            _, current = heapq.heappop(open_set)
            if current == goal:
                path = []
                while current in came_from:
                    path.append(current)
                    current = came_from[current]
                path.append(start)
                return path[::-1], f_score[goal]

            for neighbor, (distance, traffic_level) in self.nodes[current].neighbors.items():
                traffic_multiplier = self.traffic_data.get((current, neighbor), traffic_level)
                tentative_g_score = g_score[current] + distance * traffic_multiplier
                if tentative_g_score < g_score[neighbor]:
                    came_from[neighbor] = current
                    g_score[neighbor] = tentative_g_score
                    f_score[neighbor] = g_score[neighbor] + self.heuristic(neighbor, goal)
                    heapq.heappush(open_set, (f_score[neighbor], neighbor))
        return None, float('inf')

    def heuristic(self, node1, node2):
        return 1  # Constant heuristic for simplicity

    def find_best_route(self, patient_location):
        min_time = float('inf')
        best_route = None
        best_hospital = None

        # Check each hospital to calculate adjusted travel time considering traffic
        for hospital_name, hospital in self.nodes.items():
            if hospital.is_hospital and hospital.capacity > 0:
                # Perform A* search for the current hospital to get route and travel time
                route, travel_time = self.a_star_search(patient_location, hospital_name)

                # Ensure a valid route was found
                if route is not None:
                    # Adjust travel time based on traffic intensity
                    adjusted_travel_time = self.calculate_adjusted_travel_time(route, travel_time)

                    # Compare and select the hospital with the least adjusted travel time
                    if adjusted_travel_time < min_time:
                        min_time = adjusted_travel_time
                        best_route = route
                        best_hospital = hospital
                    # If the traffic intensity is high, check for the next closest hospital
                    if adjusted_travel_time > 20:  # Example threshold for high traffic
                        continue  # Skip to the next hospital

        # Reserve capacity in the selected hospital if one was found
        if best_hospital:
            best_hospital.capacity -= 1
        return best_route, min_time

    def calculate_adjusted_travel_time(self, route, travel_time):
        total_adjusted_time = 0
        for i in range(len(route) - 1):
            start = route[i]
            end = route[i + 1]
            distance, default_traffic = self.nodes[start].neighbors[end]

            # Use updated traffic data if available, otherwise default
            traffic_multiplier = self.traffic_data.get((start, end), default_traffic)

            # Adjust travel time with traffic
            total_adjusted_time += distance * traffic_multiplier
        #     print(f"Segment {start} to {end}: distance={distance}, traffic_multiplier={traffic_multiplier}, segment_time={distance * traffic_multiplier}")

        # print(f"Total adjusted travel time for route {route} is {total_adjusted_time}")
        return total_adjusted_time

    def dispatch_ems(self):
        self.patients.sort(key=lambda x: x.priority)
        for patient in self.patients:
            best_route, travel_time = self.find_best_route(patient.location)
            if best_route:
                print(f"Dispatching EMS to {patient.id} at {patient.location} with route {best_route} in {travel_time:.1f} time units.")
            else:
                print(f"No viable route found for Patient {patient.id} at {patient.location} within traffic threshold.")

    def add_patient(self):
        id = input("Enter patient ID: ")
        location = input("Enter patient location: ")
        priority = int(input("Enter patient priority (1 for high, 3 for low): "))
        patient = Patient(id, location, priority)
        self.patients.append(patient)
        print("Patient added successfully.")
    
    def update_traffic_data(self):
        start = input("Enter start location: ")
        end = input("Enter end location: ")
        if (start, end) in self.traffic_data or (end, start) in self.traffic_data:
            new_traffic_level = float(input("Enter new traffic level multiplier: "))
            self.traffic_data[(start, end)] = new_traffic_level
            self.traffic_data[(end, start)] = new_traffic_level
            print("Traffic data updated successfully.")
        else:
            print("Road does not exist between these locations.")

    def open_map_to_nearest_hospital(self):

        patient_location = input("Enter patient location: ")
        best_route, travel_time = self.find_best_route(patient_location)

        if best_route:
            origin = patient_location.replace(" ", "+")
            destination = best_route[-1].replace(" ", "+")
            url = f"https://www.google.com/maps/dir/{origin}/{destination}"

            # Output detailed route information in the terminal
            print(f"\n--- Route to Nearest Hospital ---")
            print(f"Patient Location: {patient_location}")
            print(f"Nearest Hospital: {best_route[-1]}")
            print(f"Route: {' -> '.join(best_route)}")
            print(f"Total Travel Time: {travel_time:.1f} time units")
            print("Opening route in Google Maps...")
            webbrowser.open(url)
        else:
            print("No route found to a hospital.")


    def show_database(self):
        print("\n--- EMS Route Planner Database ---")
        
        print("\nLocations and Hospitals:")
        for name, node in self.nodes.items():
            if node.is_hospital:
                print(f"Hospital: {name}, Capacity: {node.capacity}")
            else:
                print(f"Location: {name}")
                
        print("\nRoads:")
        for start_node, node in self.nodes.items():
            for end_node, (distance, traffic_level) in node.neighbors.items():
                print(f"Road from {start_node} to {end_node}: Distance = {distance}, Traffic Level = {traffic_level}")
        
        print("\nPatients:")
        for patient in self.patients:
            print(f"Patient ID: {patient.id}, Location: {patient.location}, Priority: {patient.priority}")
        
        print("\nTraffic Data:")
        for (start, end), level in self.traffic_data.items():
            print(f"Traffic level between {start} and {end}: {level}")
    
    def update_road_data(self):
        start = input("Enter the start location: ")
        end = input("Enter the end location: ")
        
        # Check if both locations exist and a road exists between them
        if (start in self.nodes and end in self.nodes and end in self.nodes[start].neighbors):
            # Prompt for traffic intensity update
            new_traffic_level = input("Enter new traffic intensity multiplier (or press Enter to skip): ")
            # Update traffic intensity if a new value is provided
            if new_traffic_level:
                new_traffic_level = float(new_traffic_level)
                self.traffic_data[(start, end)] = new_traffic_level
                self.traffic_data[(end, start)] = new_traffic_level
                # Update traffic intensity for both directions in neighbors
                self.nodes[start].neighbors[end] = (self.nodes[start].neighbors[end][0], new_traffic_level)
                self.nodes[end].neighbors[start] = (self.nodes[end].neighbors[start][0], new_traffic_level)
                print("Traffic intensity updated successfully.")

            # Prompt for distance update
            new_distance = input("Enter new distance (or press Enter to skip): ")
            # Update distance if a new value is provided
            if new_distance:
                new_distance = float(new_distance)
                # Update distance for both directions in neighbors
                self.nodes[start].neighbors[end] = (new_distance, self.nodes[start].neighbors[end][1])
                self.nodes[end].neighbors[start] = (new_distance, self.nodes[end].neighbors[start][1])
                print("Distance updated successfully.")
        else:
            print("No existing road between these locations.")


            
    def display_menu(self):
        print("\n--- EMS Route Planner Menu ---")
        print("1. Add New Location")
        print("2. Add New Road")
        print("3. Add New Patient")
        print("4. Update Traffic Data")
        print("5. Update Road Data")  
        print("6. Dispatch EMS")
        print("7. Show Entire Database")
        print("8. Find Route to Nearest Hospital in Google Maps")
        print("9. Exit")

def main():
    planner = RoutePlanner()

        # Coimbatore Hospitals
    planner.add_node("Ganga Hospital", is_hospital=True, capacity=10)
    planner.add_node("Coimbatore Medical College Hospital", is_hospital=True, capacity=12)
    planner.add_node("PSG Hospitals", is_hospital=True, capacity=8)
    planner.add_node("KMCH (Kovai Medical Center and Hospital)", is_hospital=True, capacity=9)
    planner.add_node("Royal Care Hospital", is_hospital=True, capacity=6)
    planner.add_node("Lotus Eye Hospital", is_hospital=True, capacity=5)
    planner.add_node("K.G. Hospital", is_hospital=True, capacity=9)
    planner.add_node("Sankara Eye Hospital", is_hospital=True, capacity=5)
    planner.add_node("Sri Ramakrishna Hospital", is_hospital=True, capacity=7)

    # Tiruppur Hospitals
    planner.add_node("Revathi Medical Center", is_hospital=True, capacity=7)
    planner.add_node("Aravind Eye Hospital Tiruppur", is_hospital=True, capacity=5)
    planner.add_node("Tiruppur Government Hospital", is_hospital=True, capacity=10)
    planner.add_node("Velan Hospital", is_hospital=True, capacity=6)
    planner.add_node("Mahalakshmi Hospital", is_hospital=True, capacity=8)

    # Erode Hospitals
    planner.add_node("Erode Trust Hospital", is_hospital=True, capacity=8)
    planner.add_node("KMC Hospital Erode", is_hospital=True, capacity=6)
    planner.add_node("Shree Hospital", is_hospital=True, capacity=5)
    planner.add_node("Erode Government Hospital", is_hospital=True, capacity=12)
    planner.add_node("Arokya Hospital", is_hospital=True, capacity=7)

    # Salem Hospitals
    planner.add_node("Vinayaka Missions Hospital", is_hospital=True, capacity=11)
    planner.add_node("SKS Hospital", is_hospital=True, capacity=9)
    planner.add_node("Salem Government Hospital", is_hospital=True, capacity=15)
    planner.add_node("Manipal Hospital Salem", is_hospital=True, capacity=8)
    planner.add_node("Sudar Hospital", is_hospital=True, capacity=5)

    # Coimbatore Locations
    planner.add_node("Rathinapuri")
    planner.add_node("Gandhipuram")
    planner.add_node("Singanallur")
    planner.add_node("Saravanampatti")
    planner.add_node("Ukkadam")

    # Tiruppur Locations
    planner.add_node("Anupparpalayam")
    planner.add_node("Palladam")
    planner.add_node("Avinashi")
    planner.add_node("Veerapandi")

    # Erode Locations
    planner.add_node("Perundurai")
    planner.add_node("Kavindapadi")
    planner.add_node("Bhavani")
    planner.add_node("Gobichettipalayam")

    # Salem Locations
    planner.add_node("Ammapet")
    planner.add_node("Fairlands")
    planner.add_node("Hasthampatti")
    planner.add_node("Suramangalam")

    # Adding Roads within each city (including location-to-hospital connections)

    # Coimbatore Roads
    planner.add_road("Rathinapuri", "Ganga Hospital", distance=4, traffic_level=1.1)
    planner.add_road("Gandhipuram", "Royal Care Hospital", distance=5, traffic_level=1.0)
    planner.add_road("Singanallur", "Coimbatore Medical College Hospital", distance=6, traffic_level=1.0)
    planner.add_road("Saravanampatti", "KMCH (Kovai Medical Center and Hospital)", distance=7, traffic_level=1.3)
    planner.add_road("Ukkadam", "K.G. Hospital", distance=6, traffic_level=1.1)
    planner.add_road("Royal Care Hospital", "KMCH (Kovai Medical Center and Hospital)", distance=11, traffic_level=1.4)
    planner.add_road("Ganga Hospital", "K.G. Hospital", distance=6, traffic_level=1.1)
    planner.add_road("PSG Hospitals", "Sri Ramakrishna Hospital", distance=5, traffic_level=1.2)

    # Additional Coimbatore Location-to-Hospital Roads
    planner.add_road("Rathinapuri", "Sri Ramakrishna Hospital", distance=4, traffic_level=1.2)
    planner.add_road("Gandhipuram", "PSG Hospitals", distance=5, traffic_level=1.1)
    planner.add_road("Singanallur", "Lotus Eye Hospital", distance=8, traffic_level=1.2)
    planner.add_road("Saravanampatti", "Sankara Eye Hospital", distance=9, traffic_level=1.3)
    planner.add_road("Ukkadam", "Ganga Hospital", distance=6, traffic_level=1.2)

    # Tiruppur Roads
    planner.add_road("Revathi Medical Center", "Aravind Eye Hospital Tiruppur", distance=5, traffic_level=1.2)
    planner.add_road("Tiruppur Government Hospital", "Velan Hospital", distance=3, traffic_level=1.1)
    planner.add_road("Revathi Medical Center", "Mahalakshmi Hospital", distance=4, traffic_level=1.3)
    planner.add_road("Tiruppur Government Hospital", "Palladam", distance=6, traffic_level=1.2)
    planner.add_road("Aravind Eye Hospital Tiruppur", "Velan Hospital", distance=7, traffic_level=1.1)

    # Additional Tiruppur Location-to-Hospital Roads
    planner.add_road("Anupparpalayam", "Revathi Medical Center", distance=5, traffic_level=1.1)
    planner.add_road("Palladam", "Tiruppur Government Hospital", distance=4, traffic_level=1.2)
    planner.add_road("Avinashi", "Aravind Eye Hospital Tiruppur", distance=6, traffic_level=1.3)
    planner.add_road("Veerapandi", "Velan Hospital", distance=5, traffic_level=1.1)
    planner.add_road("Palladam", "Mahalakshmi Hospital", distance=7, traffic_level=1.0)

    # Erode Roads
    planner.add_road("Erode Trust Hospital", "KMC Hospital Erode", distance=7, traffic_level=1.1)
    planner.add_road("Shree Hospital", "Arokya Hospital", distance=3, traffic_level=1.4)
    planner.add_road("Erode Government Hospital", "Perundurai", distance=5, traffic_level=1.1)
    planner.add_road("KMC Hospital Erode", "Erode Government Hospital", distance=4, traffic_level=1.2)

    # Additional Erode Location-to-Hospital Roads
    planner.add_road("Perundurai", "Erode Trust Hospital", distance=6, traffic_level=1.0)
    planner.add_road("Kavindapadi", "KMC Hospital Erode", distance=4, traffic_level=1.2)
    planner.add_road("Bhavani", "Shree Hospital", distance=3, traffic_level=1.3)
    planner.add_road("Gobichettipalayam", "Erode Government Hospital", distance=7, traffic_level=1.2)

    # Salem Roads
    planner.add_road("Vinayaka Missions Hospital", "SKS Hospital", distance=6, traffic_level=1.1)
    planner.add_road("Salem Government Hospital", "Sudar Hospital", distance=5, traffic_level=1.0)
    planner.add_road("Manipal Hospital Salem", "Fairlands", distance=7, traffic_level=1.2)
    planner.add_road("SKS Hospital", "Sudar Hospital", distance=4, traffic_level=1.1)

    # Additional Salem Location-to-Hospital Roads
    planner.add_road("Ammapet", "Vinayaka Missions Hospital", distance=8, traffic_level=1.2)
    planner.add_road("Fairlands", "SKS Hospital", distance=5, traffic_level=1.1)
    planner.add_road("Hasthampatti", "Salem Government Hospital", distance=6, traffic_level=1.0)
    planner.add_road("Suramangalam", "Manipal Hospital Salem", distance=6, traffic_level=1.3)

    # Inter-City Roads for Cross-City EMS Support
    planner.add_road("Sri Ramakrishna Hospital", "Revathi Medical Center", distance=40, traffic_level=1.6)
    planner.add_road("Sri Ramakrishna Hospital", "Erode Trust Hospital", distance=52, traffic_level=1.4)
    planner.add_road("Revathi Medical Center", "Vinayaka Missions Hospital", distance=45, traffic_level=1.5)
    planner.add_road("Erode Trust Hospital", "SKS Hospital", distance=48, traffic_level=1.3)
    planner.add_road("Royal Care Hospital", "Revathi Medical Center", distance=42, traffic_level=1.5)
    planner.add_road("KMCH (Kovai Medical Center and Hospital)", "Tiruppur Government Hospital", distance=30, traffic_level=1.4)
    planner.add_road("PSG Hospitals", "Aravind Eye Hospital Tiruppur", distance=38, traffic_level=1.3)
    planner.add_road("K.G. Hospital", "Vinayaka Missions Hospital", distance=50, traffic_level=1.6)
    planner.add_road("Coimbatore Medical College Hospital", "Shree Hospital", distance=47, traffic_level=1.2)

   
    

    # Main menu loop
    while True:
        planner.display_menu()
        choice = input("Select an option (1-8): ")
        if choice == '1':
            planner.add_node(input("Enter location name: "))
        elif choice == '2':
            start = input("Enter start location: ")
            end = input("Enter end location: ")
            distance = float(input("Enter distance: "))
            traffic_level = float(input("Enter traffic level multiplier: "))
            planner.add_road(start, end, distance, traffic_level)
        elif choice == '3':
            planner.add_patient()
        elif choice == '4':
            planner.update_traffic_data()
        elif choice == '5':  
            planner.update_road_data()
        elif choice == '6':
            planner.dispatch_ems()
        elif choice == '7':
            planner.show_database()
        elif choice == '8':
            planner.open_map_to_nearest_hospital()
        elif choice == '9':
            print("Exiting EMS Route Planner. Goodbye!")
            break
        else:
            print("Invalid option. Please choose again.")

if __name__ == "__main__":
    main()
