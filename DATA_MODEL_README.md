# Onco Global Patient Scheduling System - Data Model Documentation

## Overview
This document describes the custom Salesforce data model created for the AI-Powered Patient Scheduling Agent for Onco Global healthcare network.

## Custom Objects

### 1. Hospital__c
**Purpose:** Represents Onco Global hospital locations and facilities.

**Fields:**
- `Name` (Auto Number) - Hospital Name
- `Address__c` (Text Area) - Complete hospital address
- `City__c` (Text) - City location
- `Phone__c` (Phone) - Main contact phone
- `Email__c` (Email) - Main contact email

**Sharing Model:** Read/Write

---

### 2. Department__c
**Purpose:** Medical departments within hospitals (e.g., Oncology, Gynecology, Pediatrics).

**Fields:**
- `Name` (Text) - Department Name
- `Description__c` (Long Text Area) - Detailed description of services
- `Is_Active__c` (Checkbox) - Active status (default: true)

**Sharing Model:** Read/Write

---

### 3. Doctor__c
**Purpose:** Healthcare providers and doctors in the Onco Global network.

**Fields:**
- `Name` (Text) - Doctor Name
- `Specialty__c` (Picklist - Required) - Medical specialty
  - Oncology
  - Gynecology
  - Pediatrics
  - Cardiology
  - Neurology
  - Orthopedics
  - Radiology
  - Surgery
- `Department__c` (Lookup to Department__c) - Primary department
- `Hospital__c` (Lookup to Hospital__c - Required) - Primary hospital
- `Email__c` (Email) - Contact email
- `Phone__c` (Phone) - Contact phone
- `Bio__c` (Long Text Area) - Professional biography
- `Is_Available__c` (Checkbox) - Accepting new appointments (default: true)

**Relationships:**
- Belongs to Hospital (Master-Detail equivalent via required lookup)
- Belongs to Department (Optional)

**Sharing Model:** Read/Write

---

### 4. Patient__c
**Purpose:** Patient records for the healthcare network.

**Fields:**
- `Name` (Text) - Patient Name
- `Patient_ID__c` (Text - Unique, External ID) - Unique patient identifier
- `Email__c` (Email) - Contact email
- `Phone__c` (Phone) - Contact phone
- `Date_of_Birth__c` (Date) - Date of birth

**Sharing Model:** Private

**Note:** In production with Health Cloud, use Person Accounts instead of this custom object.

---

### 5. Appointment__c
**Purpose:** Patient appointments with doctors at hospital facilities.

**Fields:**
- `Name` (Auto Number: APT-{0000}) - Appointment Number
- `Patient__c` (Lookup to Patient__c - Required) - Patient
- `Doctor__c` (Lookup to Doctor__c - Required) - Assigned doctor
- `Hospital__c` (Lookup to Hospital__c - Required) - Hospital location
- `Appointment_Date_Time__c` (DateTime - Required) - Scheduled date/time
- `Status__c` (Picklist - Required) - Appointment status
  - Scheduled (default)
  - Confirmed
  - In Progress
  - Completed
  - Cancelled
  - No Show
  - Rescheduled
- `Reason__c` (Text Area) - Reason for visit

**Relationships:**
- Belongs to Patient
- Belongs to Doctor
- Belongs to Hospital

**Sharing Model:** Private

---

### 6. FAQ_Content__c
**Purpose:** Knowledge base content for Agentforce agent responses.

**Fields:**
- `Name` (Text) - FAQ Title
- `Question__c` (Long Text Area) - The FAQ question
- `Answer__c` (Long Text Area) - The FAQ answer
- `Category__c` (Picklist) - FAQ category
  - Oncology Services
  - Treatments
  - Hospital Facilities
  - Appointment Booking
  - Insurance & Billing
  - Patient Care
  - General Information

**Sharing Model:** Read/Write

---

### 7. Agent_Conversation__c
**Purpose:** Logs Agentforce agent conversations for analytics and improvement.

**Fields:**
- `Name` (Auto Number: CONV-{0000}) - Conversation Number
- `Patient__c` (Lookup to Patient__c) - Patient who had the conversation
- `Session_ID__c` (Text - External ID) - Unique session identifier
- `Intent__c` (Picklist) - Primary conversation intent
  - Book Appointment
  - Reschedule Appointment
  - Cancel Appointment
  - Find Doctor
  - General Inquiry
  - Department Information
  - Hospital Location
  - Other
- `Outcome__c` (Picklist) - Final outcome
  - Resolved
  - Escalated to Human
  - Abandoned
  - In Progress

**Sharing Model:** Private

---

## Entity Relationship Diagram

```
Hospital__c
    ├── Doctors (1:many)
    │   └── Doctor__c
    │       ├── Department__c (lookup)
    │       └── Appointments (1:many)
    │           └── Appointment__c
    └── Appointments (1:many)
        └── Appointment__c

Department__c
    └── Doctors (1:many)
        └── Doctor__c

Patient__c
    ├── Appointments (1:many)
    │   └── Appointment__c
    └── Agent Conversations (1:many)
        └── Agent_Conversation__c

FAQ_Content__c (Standalone knowledge base)

Agent_Conversation__c
    └── Patient__c (lookup)
```

---

## Data Model Features

### 1. **Relationships**
- Lookup relationships enable flexible data navigation
- Required lookups ensure data integrity
- External IDs support data integration and upserts

### 2. **Picklists**
- Standardized values for reporting and consistency
- Support for Agentforce intent recognition
- Easy to extend with new values

### 3. **Security**
- Private sharing for sensitive patient data
- Read/Write for operational data
- Field-level security ready

### 4. **Audit & Reporting**
- Auto-number fields for tracking
- Timestamp fields (CreatedDate, LastModifiedDate)
- Report-enabled for analytics

---

## Deployment Instructions

### Deploy to Salesforce Org

```bash
# Authenticate to your org
sf org login web --alias MyOrg

# Deploy the metadata
sf project deploy start --source-dir force-app/main/default

# Verify deployment
sf project deploy report
```

### Verify Objects Created

```bash
# List custom objects
sf sobject list --sobject-type custom
```

---

## Next Steps

After deploying the data model:

1. **Load Sample Data**
   - Create 3-5 hospitals
   - Create 5-10 departments
   - Create 10-20 doctors
   - Create sample patients
   - Create sample FAQ content

2. **Configure Security**
   - Create permission sets
   - Assign field-level security
   - Set up sharing rules

3. **Build Apex Controllers**
   - AppointmentBookingController
   - DoctorSearchController
   - NotificationService
   - FAQSearchController

4. **Create Agentforce Agent**
   - Design agent topics
   - Build custom actions
   - Configure intent recognition

5. **Build UI Components (LWC)**
   - Doctor search
   - Appointment booking flow
   - Patient dashboard
   - Agent chat widget

---

## Health Cloud Integration

When Health Cloud is available, migrate to:
- **Person Accounts** → Replace Patient__c
- **Care Program Providers** → Enhance Doctor__c
- **Healthcare Facilities** → Replace Hospital__c
- **Service Appointments** → Replace Appointment__c

---

## Support & Documentation

For questions or issues:
1. Review Salesforce metadata API documentation
2. Check deployment logs: `sf project deploy report`
3. Validate relationships in Schema Builder
4. Test with sample data before production use

---

**Version:** 1.0  
**Last Updated:** April 26, 2026  
**Maintainer:** Onco Global Development Team