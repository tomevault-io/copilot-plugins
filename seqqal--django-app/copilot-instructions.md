## django-app

> You are tasked with completing and refining an IoT Temperature & Humidity Monitoring Application. The project is already partially coded but not fully functional.

PROJECT REQUIREMENTS

You are tasked with completing and refining an IoT Temperature & Humidity Monitoring Application. The project is already partially coded but not fully functional.
Technology stack:
• Hardware: ESP8266 + DHT11 sensor
• Backend: Django (with Django REST Framework), Python
• Frontend: React (JavaScript)
• Database: SQLite, PostgreSQL, or similar
• Hosting: PythonAnywhere (or other cloud platforms)
Overall Goal:
• Collect temperature and humidity data from the DHT11 sensor via the ESP8266 every 20 minutes(but for now lets just create a post endpoint that will post data from the backend to store temp and humidity data) ).
• Send these readings via WiFi to the Django backend endpoint.
• Store the data in a database.
• Display real-time data and historical graphs in the React frontend.
• Implement an alerting system for temperature anomalies.
• Provide administrative functionality (user management, incident tracking, etc.).


DATA ACQUISITION (DHT11 + ESP8266)
• The DHT11 sensor collects temperature (°C) and humidity (%) every 20 minutes.
• The ESP8266 posts this data to an endpoint in the Django backend.
• Ensure proper error handling for lost connections or invalid data.

run the commands in docker compose exec backend or front end

BACKEND APPLICATION (Django)
• Create a REST API using Django REST Framework.
• Endpoints for receiving sensor data, retrieving historical data, and managing alerts/incidents.
• Store temperature and humidity readings in the database.
• Implement endpoints for user management (create, modify, delete users) and permissions.
• Handle authentication (tokens or other suitable method).


FRONTEND APPLICATION (React)
• Display temperature and humidity data in real time (or near real time).
• Show graphs of historical data (daily, weekly, monthly views).
• Include a clear user interface for:
– Viewing active alerts.
– Managing incidents.
– Viewing and editing user profiles.


ALERTING SYSTEM
• Temperature thresholds:
– Normal: 2°C to 8°C
– Critical: 0°C to 2°C or 8°C to 10°C
– Severe: Outside these ranges
• Alert escalation rules:
– First alert: sent to Operator 1.
– After four consecutive alerts: add Operator 2.
– After seven consecutive alerts: add Operator 3.
• When temperature returns to normal, the alert count resets.


INCIDENT MANAGEMENT (TEMPERATURE ANOMALIES)
• Archive each incident (anomaly) in the database for future reference.
• Detailed tracking of incidents:
– Temperature at time of incident
– Start time of incident
– Alerts sent (history of escalation)
• Provide a dedicated incident page where users can:
– View details (timestamp, temperature data, alerts triggered)
– Add remarks or comments
– Acknowledge incidents
– Monitor actions taken by each user


ADMINISTRATIVE FEATURES
• User Management:
– Add, edit, delete users with the following info: name, surname, password, email, phone number, token, etc.
– Assign roles and permissions.
• Security & Permissions:
– Ensure security best practices (e.g., enforcement of role-based access).


PROJECT TIMELINE (approximate)
• Study & Requirements Review: 1 week
• Backend Development (Django): 2 weeks
• Frontend Development (React): 2 weeks
• Integration & Testing: 1 week
• Deployment & Finalization: 1 week


DELIVERABLES
• Complete source code for backend (Django) and frontend (React).
• Documentation (code architecture, setup instructions, user manual).
• Demonstration of a functional system with real sensor data being monitored.


EVALUATION CRITERIA
• Functional correctness: correct data flow from sensor to web app.
• Code quality & best practices (Python/Django, React).
• UI/UX design: intuitive, responsive, clear.
• Reliability & performance under normal network conditions.
• Timely completion within agreed milestones.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seqqal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
