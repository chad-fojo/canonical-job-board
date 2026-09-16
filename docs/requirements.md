# Software Requirements Specification (SRS)
**Project Name:** Canonical Job Board

## 1. Introduction
### 1.1 Purpose
The purpose of this project is to build a scalable, automated job board that aggregates open requisitions from disparate Applicant Tracking System (ATS) APIs (e.g., Greenhouse, Lever). The system normalizes varying API payloads into a canonical relational schema, allowing users to search and filter jobs across multiple companies through a unified interface.

### 1.2 Target Audience
This project demonstrates full-stack software engineering capabilities, specifically highlighting:
*   Data pipeline engineering and third-party API integration.
*   Relational database administration and schema normalization for complex, disparate data structures.
*   Cloud-native deployment and infrastructure management.

## 2. System Architecture
The application follows a decoupled, three-tier architecture:
1.  **Data Ingestion Service:** A scheduled pipeline that periodically polls ATS endpoints, sanitizes HTML descriptions, deduplicates entries, and normalizes the JSON payloads into the canonical database schema.
2.  **RESTful API Core:** A backend service that exposes endpoints for the frontend to query the normalized job data, utilizing pagination and indexed search filters.
3.  **Frontend Client:** A responsive web application displaying the aggregated job feed, routing users to the original ATS URL for the application process.

## 3. Proposed Technology Stack
*   **Backend / API:** Python - for robust data-fetching pipelines and REST API development.
*   **Database:** PostgreSQL - utilizing standard relational tables alongside JSONB columns for raw payload archiving.
*   **Frontend:** HTML5, CSS3, JavaScript.
*   **Cloud Infrastructure:** AWS
    *   *Compute:* EC2 or AWS Lambda - for the scheduled ingestion workers.
    *   *Database:* Amazon RDS.
*   **Version Control & CI/CD:** Git, GitHub Actions.

## 4. Functional Requirements
### 4.1 Data Ingestion & Synchronization
*   **[REQ-1.1]** The system shall fetch data from at least two distinct ATS public APIs (e.g., Greenhouse and Lever).
*   **[REQ-1.2]** The system shall map disparate ATS fields (title, location, department) into a unified canonical schema.
*   **[REQ-1.3]** The system shall execute a synchronization job on a predefined schedule (e.g., every 12 hours) to capture new postings and detect closed requisitions.
*   **[REQ-1.4]** The system shall store the raw, unmodified JSON payload from the ATS for audit and re-parsing purposes.

### 4.2 API layer
*   **[REQ-2.1]** The API shall provide a `GET /jobs` endpoint that returns a paginated list of active job postings.
*   **[REQ-2.2]** The API shall support query parameters for filtering by `company`, `department`, and `is_remote`.
*   **[REQ-2.3]** The API shall provide a `GET /jobs/{id}` endpoint to retrieve full details for a specific posting.

### 4.3 User Interface
*   **[REQ-3.1]** The frontend shall display a feed of active job postings, showing the public title, company logo, and location.
*   **[REQ-3.2]** The frontend shall allow users to filter the feed using the criteria supported by the API.
*   **[REQ-3.3]** The frontend shall render job descriptions securely, sanitizing any malicious scripts from the original ATS HTML payload.
*   **[REQ-3.4]** The frontend shall provide an "Apply Now" button that redirects the user to the source ATS application URL.

## 5. Non-Functional Requirements
### 5.1 Performance & Scalability
*   **Database Optimization:** Queries on the `jobs` and `job_postings` tables must utilize appropriate indexing on heavily filtered columns (e.g., `company_id`, `department_id`) to ensure API response times under 200ms.
*   **Resilience:** The data ingestion service must handle rate-limiting and intermittent failures gracefully, implementing retry logic with exponential backoff.

### 5.2 Security & Data Integrity
*   **HTML Sanitization:** All rich-text job descriptions ingested from third parties must be stripped of `<script>` tags and potentially dangerous attributes before being rendered on the client.
*   **Data Integrity:** Foreign key constraints must be strictly enforced between Companies, Departments, Locations, and Jobs to prevent orphaned records during sync operations.

## 6. Database Schema Design (Canonical Model)
The database normalizes the 1:N relationships between internal requisitions and public postings to third normal form (3NF).

| Table Name | Primary Key | Foreign Keys | Key Attributes |
| :--- | :--- | :--- | :--- |
| **Companies** | `id` | None | `name`, `ats_provider`, `tenant_id` |
| **Departments** | `id` | `company_id` | `name` |
| **Locations** | `id` | `company_id` | `name`, `city`, `state`, `is_remote` |
| **Jobs** (Internal Req) | `id` | `company_id`, `dept_id`, `loc_id` | `internal_title`, `status` |
| **Job_Postings** (Public) | `id` | `job_id` | `public_title`, `description_html`, `url`, `raw_payload` |
