# Reverse Engineering Assignment Week 12

**Ibrahim Irfanul Haq (H1A023003)**

This document explains the structural relationships between key entities in a healthcare system database.

## Overview

This assignment explores the relationships between core healthcare data models: Visit, Encounter, Provider, and Person. Understanding these relationships is essential for working with healthcare information systems.

## Database Schema

### Entity Relationship Diagram
```sql
-- Person table (base entity)
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(255),
    address TEXT,
    birth_date DATE,
    gender VARCHAR(10)
);

-- Provider table (extends Person)
CREATE TABLE provider (
    provider_id INT PRIMARY KEY,
    person_id INT NOT NULL,
    specialty VARCHAR(100),
    license_number VARCHAR(50),
    FOREIGN KEY (person_id) REFERENCES person(person_id)
);

-- Visit table (container)
CREATE TABLE visit (
    visit_id INT PRIMARY KEY,
    patient_id INT NOT NULL,
    location VARCHAR(100),
    start_date DATETIME,
    end_date DATETIME,
    FOREIGN KEY (patient_id) REFERENCES person(person_id)
);

-- Encounter table (discrete events)
CREATE TABLE encounter (
    encounter_id INT PRIMARY KEY,
    visit_id INT NOT NULL,
    provider_id INT,
    encounter_type VARCHAR(50),
    encounter_date DATETIME,
    diagnosis TEXT,
    FOREIGN KEY (visit_id) REFERENCES visit(visit_id),
    FOREIGN KEY (provider_id) REFERENCES provider(provider_id)
);
```

## Usage Example
```python
# Example: Query to get all encounters in a visit
def get_visit_encounters(visit_id):
    query = """
    SELECT e.encounter_id, e.encounter_type, e.encounter_date,
           p.name as provider_name
    FROM encounter e
    JOIN provider pr ON e.provider_id = pr.provider_id
    JOIN person p ON pr.person_id = p.person_id
    WHERE e.visit_id = ?
    ORDER BY e.encounter_date
    """
    return execute_query(query, [visit_id])

# Example: Get provider details
def get_provider_info(provider_id):
    query = """
    SELECT pr.provider_id, pr.specialty, pr.license_number,
           p.name, p.address
    FROM provider pr
    JOIN person p ON pr.person_id = p.person_id
    WHERE pr.provider_id = ?
    """
    return execute_query(query, [provider_id])
```

## Entity Relationships

### 1. Visit and Encounter Relationship

**Visit (Kunjungan)**: Represents a period of time when a patient actively interacts with the healthcare system at a specific location. A visit serves as a container that can span one day or multiple days.

**Encounter (Perjumpaan)**: Represents a discrete event or single clinical transaction, such as:
- Recording vital signs
- Making a diagnosis
- Completing a form

**Structural Relationship**: One Visit can contain multiple Encounters.
```
Visit (1) ──── (Many) Encounter
```

### 2. Provider and Person Relationship

**Person (Individu)**: The base entity that stores demographic data (name, address, attributes) for all individuals in the system, including patients, users, and providers.

**Provider (Penyedia Layanan)**: A specialized role assigned to a person who delivers healthcare services.

**Structural Relationship**: The provider table contains a foreign key (`person_id`) that points directly to the person table, making Provider an extension of the Person entity.
```
Person (1) ──── (1) Provider
             person_id (FK)
```

## Database Schema Summary

| Entity | Purpose | Key Relationships |
|--------|---------|-------------------|
| Person | Base demographic data for all individuals | Referenced by Provider |
| Provider | Healthcare service provider role | Extends Person via `person_id` FK |
| Visit | Container for patient interaction period | Contains multiple Encounters |
| Encounter | Discrete clinical transaction/event | Belongs to one Visit |

## Key Concepts

- **One-to-Many**: A single Visit encompasses multiple Encounters
- **One-to-One Extension**: Provider extends Person through foreign key relationship
- **Role-Based Design**: Provider represents a specialized role of the Person entity

## Author

Ibrahim Irfanul Haq (H1A023003)
