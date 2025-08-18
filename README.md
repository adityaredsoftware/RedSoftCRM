# Clinic WhatsApp Bot - BPMN Style Process Flow

## Process Overview
**Business Process**: Automated WhatsApp lead management and appointment booking for clinic services (Hair, Teeth, Skin)

**Participants**: 
- Customer (WhatsApp user)
- Clinic Executive Bot
- Google Sheets CRM
- Consultation System

---

## Main Process Flow

### 1. Message Reception & Initial Processing
```
[START] WhatsApp Message Received
    ↓
[Task] Send Typing Indicator
    ↓
[Task] Extract User ID from WhatsApp contact
    ↓
[Task] Format Phone Number (+XX-XXXXXXXXXX)
    ↓
[Gateway] Check if User Exists in CRM
```

### 2. User Existence Check
```
[Exclusive Gateway] User Exists in CRM?
    ├─ YES → [Subprocess] Handle Existing User
    └─ NO → [Subprocess] Handle New User
```

---

## Subprocess: Handle New User

### Stage 1: Initial Contact
```
[Task] Check if User Exists in Lead Tracking Sheet
    ↓
[Exclusive Gateway] User in Lead Tracking?
    ├─ YES → [Subprocess] Handle by Stage
    └─ NO → [Task] Create New Lead (Stage 1)
            ↓
            [Task] AI: Request Full Name
            ↓
            [Task] Send Welcome Message
            ↓
            [End Event]
```

### Stage 2: Name Collection
```
[Task] AI: Detect Valid Name
    ↓
[Exclusive Gateway] Valid Name Detected?
    ├─ YES → [Task] Update Lead (Stage 2)
           ↓
           [Task] AI: Request City
           ↓
           [End Event]
    └─ NO → [Task] AI: Re-request Name
           ↓
           [End Event]
```

### Stage 3: City Collection
```
[Task] AI: Detect Valid City
    ↓
[Exclusive Gateway] Valid City Detected?
    ├─ YES → [Task] Update Lead (Stage 3)
           ↓
           [Task] Transfer to Main CRM
           ↓
           [Task] AI: Present Service Options
           ↓
           [End Event]
    └─ NO → [Task] AI: Re-request City
           ↓
           [End Event]
```

---

## Subprocess: Handle Existing User

### User Stage Routing
```
[Task] Get User Data from Lead Tracking
    ↓
[Inclusive Gateway] Route by Stage
    ├─ Stage 3 → [Subprocess] Service Selection
    ├─ Stage 4 → [Subprocess] Date Selection  
    ├─ Stage 5 → [Subprocess] Time Selection
    └─ Stage 6 → [Subprocess] General Conversation
```

### Stage 4: Service Selection
```
[Task] AI: Detect Service (Hair/Teeth/Skin)
    ↓
[Exclusive Gateway] Valid Service Detected?
    ├─ YES → [Task] Update CRM with Service
           ↓
           [Task] Update Lead (Stage 4)
           ↓
           [Task] Create Consultation Record
           ↓
           [Task] AI: Request Appointment Date
           ↓
           [End Event]
    └─ NO → [Task] AI: Re-present Service Options
           ↓
           [End Event]
```

### Stage 5: Date Selection
```
[Task] AI: Detect Valid Date
    ↓
[Exclusive Gateway] Valid Date Detected?
    ├─ YES → [Task] Update Lead (Stage 5)
           ↓
           [Task] Update Consultation Date
           ↓
           [Task] AI: Request Appointment Time
           ↓
           [End Event]
    └─ NO → [Task] AI: Re-request Date
           ↓
           [End Event]
```

### Stage 6: Time Selection
```
[Task] AI: Detect Valid Time
    ↓
[Exclusive Gateway] Valid Time Detected?
    ├─ YES → [Task] Update Lead (Stage 6)
           ↓
           [Task] Complete Consultation Booking
           ↓
           [Task] AI: Confirm Appointment
           ↓
           [End Event]
    └─ NO → [Task] AI: Re-request Time
           ↓
           [End Event]
```

### Stage 7: General Conversation
```
[Task] AI: Analyze Message Content
    ↓
[Exclusive Gateway] Clinic-Related Query?
    ├─ YES → [Task] AI: Provide Helpful Response
           ↓
           [End Event]
    └─ NO → [Task] AI: Politely Decline & Redirect
           ↓
           [End Event]
```

---

## Data Integration Points

### Google Sheets Integration
1. **Lead Tracking Sheet**
   - Columns: Phone Number, Name, City, Stage
   - Purpose: Track lead progression through stages

2. **Main CRM Sheet (Clarin Lead)**
   - Comprehensive customer data
   - Service preferences
   - Contact information

3. **Consultation Sheet**
   - Appointment scheduling
   - Service assignments
   - Doctor assignments

### AI Processing Chain
```
[Input] User Message
    ↓
[Task] OpenAI GPT-4o-mini Processing
    ↓
[Task] JSON Response Parsing
    ↓
[Output] Structured Response with:
    - Detection Result (boolean)
    - Reply Message (string)  
    - Extracted Data (string/null)
```

---

## Business Rules

### Name Validation Rules
- 1-4 words representing a person's name
- Exclude business names, places, job titles
- Handle greetings with names

### City Validation Rules  
- 1-4 words representing a city/locality
- Examples: Mumbai, New Delhi, Greater Noida
- Exclude postal codes, business names

### Service Detection Rules
- Exact match: "Hair", "Teeth", or "Skin"
- Can appear alone or within longer messages
- Case-insensitive detection

### Date/Time Validation Rules
- **Dates**: DD/MM/YYYY, Month names, relative terms
- **Times**: 12/24-hour format, time ranges
- Preserve original user formatting

---

## Error Handling & Fallbacks

### Invalid Input Scenarios
```
[Error] Invalid/Unrelated Input
    ↓
[Task] AI: Polite Correction
    ↓
[Task] Re-prompt for Correct Information
    ↓
[End Event]
```

### System Error Scenarios
```
[Error] Sheet Update Failure
    ↓
[Task] Continue with Regular Output
    ↓
[Task] Log Error (if applicable)
    ↓
[End Event]
```

---

## Key Performance Indicators

### Conversion Metrics
- **Stage 1 → Stage 2**: Name collection rate
- **Stage 2 → Stage 3**: City collection rate  
- **Stage 3 → Stage 4**: Service selection rate
- **Stage 4 → Stage 6**: Appointment completion rate

### Response Quality
- AI detection accuracy per stage
- User satisfaction with responses
- Time to complete booking process
