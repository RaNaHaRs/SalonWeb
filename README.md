# Salon AI Web Application — Spring Boot Build Architecture

A enterprise-grade Salon Management and Computer Vision AI Platform. This system is designed around a **Spring Boot core backend** with an isolated, event-driven **Python Computer Vision Worker**.

---

## 🎯 Core Architectural Philosophy

1. **Spring Boot is the Source of Truth**: All business logic, customer profiles, staff management, permissions, security, service sessions, and audit logging reside strictly inside Spring Boot and PostgreSQL.
2. **AI Is a Later Event Integration**: Every feature triggered by AI (e.g., employee attendance, customer arrival, zone tracking) **must first work end-to-end via standard manual UI actions and REST APIs**.
3. **Decoupled Python Worker**: The Python AI service operates purely as an external event producer via internal REST endpoints (`/api/vision/events`) and WebSocket updates. It **never** writes directly to PostgreSQL or evaluates business permissions.
4. **Human-in-the-Loop Safeguards**: Uncertain AI recognition events trigger confirmation queues for receptionists rather than executing automatic database mutations.

---

## 🛠️ Technology Stack

| Component | Technology |
| :--- | :--- |
| **Backend Framework** | Java 17, Spring Boot 4.x (Spring WebMVC, Spring Data JPA, Spring Security, WebSocket, Validation) |
| **Database** | PostgreSQL |
| **Frontend** | React + Vite (Preferred) or Spring MVC / Thymeleaf |
| **AI Worker (Phase 10+)** | Python 3.10+, OpenCV, Face Recognition / Embedding Models |
| **Build System** | Apache Maven |
| **Utilities** | Lombok, DevTools |

---

## 🏗️ Spring Boot Package & Module Structure

```text
org.harsh.salonweb
├── auth          # Authentication, Spring Security, Roles & Permissions
├── employee      # Staff CRUD, Shifts, Station Assignment & Manual Attendance
├── customer      # Customer Profiles, Consent Status, Stylist Preferences & History
├── service       # Service Catalogue, Pricing & Service Sessions
├── visit         # Appointment Booking, Visit Status & History
├── reception     # Live Event Feed, Arrivals, Customer Cards
├── zone          # Salon Zone Definitions & Zone Time Calculations
├── event         # Central Application & Vision Event Engine
├── consultation  # Hair, Skin & Style Photo Analysis & Recommendations
├── report        # Dashboard Aggregation, Metrics & PDF/CSV Exports
├── correction    # Human Review Queue & Event Corrections
├── audit         # Decision Audit Logs & Access Records
└── vision        # REST / DTO Contracts for Computer Vision Integration (No Python code)
```

---

## 📊 Database Schema Plan

* **`employees`**: Staff metadata, role, station, recognition opt-in status.
* **`employee_attendance`**: Clock-in / clock-out timestamps, source (manual vs vision), duration.
* **`customers`**: Customer profiles, contact details, consent flags, notes.
* **`services`**: Service names, duration, prices, target zone.
* **`appointments`**: Scheduled customer visits, assigned employee, status.
* **`visits`**: Actual customer salon visits, arrival timestamp, check-out time.
* **`service_sessions`**: Active services per customer, assigned staff member, zone.
* **`salon_zones`**: Physical salon zones (e.g., Reception, Hair Station 1, Washing Area).
* **`presence_events`**: Immutable log of all movement and recognition events.
* **`analysis_results`**: Consultation photo upload metadata and raw analysis outputs.
* **`recommendations`**: Suggested products/treatments linked to customer visits.
* **`corrections`**: Log of manual reception overrides for AI or staff events.
* **`decision_audit`**: Audit trails for system and staff decisions.
* **`access_log`**: Security login and permission audit records.

---

## ⚡ AI-Agnostic Event Architecture

Both UI actions (manual mode) and the future Python AI Worker emit standardized event payloads processed by the same `EventService`:

### Event Payload Specification

```json
{
  "eventType": "CUSTOMER_ARRIVAL",
  "personType": "CUSTOMER",
  "personId": 102,
  "name": "Jane Doe",
  "confidence": 0.94,
  "cameraId": "cam-reception-01",
  "zone": "RECEPTION",
  "timestamp": "2026-09-29T15:30:00Z",
  "matchState": "CONFIRMED",
  "source": "VISION"
}
```

### Core Event Types

* **Manual Events**: `MANUAL_CHECK_IN`, `MANUAL_CHECK_OUT`, `CUSTOMER_ARRIVAL`, `ZONE_CHANGE`, `SERVICE_STARTED`, `SERVICE_ENDED`
* **Vision Events**: `VISION_PERSON_RECOGNIZED`, `VISION_CUSTOMER_RECOGNIZED`, `VISION_UNKNOWN_PERSON`, `VISION_NEEDS_CONFIRMATION`

---

## 🗺️ 15-Phase Development Roadmap

### Phase 1: Core Non-AI Application (Milestones 1 – 10)
* **Phase 0 — Foundation**: Maven setup, PostgreSQL connection, global exception handling & standardized DTO responses.
* **Phase 1 — Auth & Security**: Spring Security setup with `ADMIN`, `MANAGER`, `RECEPTION` roles.
* **Phase 2 — Employees**: Staff CRUD, station management, manual check-in/check-out UI.
* **Phase 3 — Customers**: Profile management, consent tracking, visit history.
* **Phase 4 — Services & Visits**: Service catalogue, appointment booking, visit tracking.
* **Phase 5 — Reception Workflow**: Live manual event feed, customer arrival cards, manual service startup.
* **Phase 6 — Zones & Event Engine**: Salon zone definitions, manual zone transfers, time tracking.
* **Phase 7 — Reports & Audit**: Attendance reports, service session stats, audit logging, manual correction panel.
* **Phase 8 — Consultation Module**: Photo upload interface, consultation records, mock/manual analysis results.
* **Phase 9 — E2E Non-AI Freeze**: Full system testing without Python; complete end-to-end demo execution.

### Phase 2: Computer Vision & AI Worker (Milestones 11 – 14)
* **Phase 10 — Python AI Service (Video)**: Isolated Python worker reading recorded video, detecting faces, POSTing recognition events to Spring Boot.
* **Phase 11 — Live Webcam Mode**: Real-time webcam feed recognition with confidence scoring and reception confirmation states.
* **Phase 12 — Quality & Liveness**: Quality filters, anti-spoofing checks, uncertain match queueing.
* **Phase 13 — Zones & CCTV**: Camera calibration, RTSP stream handling, automated zone detection.
* **Phase 14 — Skin, Hair & Style AI**: Guided photo analysis powered by AI models with receptionist review.

---

## 🚀 Getting Started

### Prerequisites
* **Java Development Kit (JDK 17+)**
* **Apache Maven 3.8+**
* **PostgreSQL 14+**

### Database Setup
1. Create a PostgreSQL database named `salonweb`:
   ```sql
   CREATE DATABASE salonweb;
   ```
2. Configure credentials in `src/main/resources/application.properties`:
   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/salonweb
   spring.datasource.username=postgres
   spring.datasource.password=your_password
   spring.jpa.hibernate.ddl-auto=update
   ```

### Building & Running
To compile and start the Spring Boot application:

```bash
# Run using Maven wrapper (Windows)
.\mvnw.cmd spring-boot:run

# Run using standard Maven
mvn clean spring-boot:run
```

The application will launch on `http://localhost:8080`.

---

## ✅ Definition of Done

* **Release 1 (Non-AI Core)**: Users can log in, manage employees/customers/services, record attendance, run reception workflows, start service sessions, view reports, and perform mock consultations — **with zero Python processes running**.
* **Release 2 (AI Integrated)**: The Python Vision service seamlessly emits vision events to Spring Boot to automate check-ins and arrivals, without breaking any existing Spring Boot business logic or manual overrides.
