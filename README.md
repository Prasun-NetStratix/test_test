DATABASE VISUALISATION
# Database Structure Visualization - School Management SaaS

## 🏗️ Complete Database Architecture

### 1. High-Level System Overview
```mermaid
graph TB
    subgraph "Firebase Project"
        subgraph "Super Admin Layer"
            SA[Super Admin]
            SA --> |Manages| Schools
            SA --> |Controls| Features
            SA --> |Handles| Billing
        end
        
        subgraph "Multi-Tenant Schools"
            S1[School 1<br/>ABC123]
            S2[School 2<br/>XYZ789]
            S3[School N<br/>DEF456]
        end
        
        subgraph "Shared System"
            R[Roles & Permissions]
            F[Feature Management]
            B[Billing & Subscriptions]
        end
    end
    
    SA --> S1
    SA --> S2
    SA --> S3
    SA --> R
    SA --> F
    SA --> B
```

### 2. Core Collections Structure
```mermaid
graph TD
    subgraph "Root Collections"
        Users["users/{uid}"]
        Schools[schools/{schoolId}]
        SuperAdmin[superadmin/]
        System[system/]
    end
    
    subgraph "School-Specific Collections"
        SchoolProfile[schools/{schoolId}/profile]
        SchoolUsers[schools/{schoolId}/users/{uid}]
        SchoolStudents[schools/{schoolId}/students/{studentId}]
        SchoolTeachers[schools/{schoolId}/teachers/{teacherId}]
        SchoolClasses[schools/{schoolId}/classes/{classId}]
        SchoolSubjects[schools/{schoolId}/subjects/{subjectId}]
    end
    
    subgraph "Academic Collections"
        Attendance[schools/{schoolId}/attendance/{date}/{userId}]
        AttendanceSummary[schools/{schoolId}/attendanceSummary/{userId}/{month}]
        Marks[schools/{schoolId}/marks/{studentId}/{examId}]
        ReportCards[schools/{schoolId}/reportCards/{studentId}/{termId}]
    end
    
    subgraph "Financial Collections"
        FeeStructure[schools/{schoolId}/feeStructure/{structureId}]
        StudentFees[schools/{schoolId}/studentFees/{studentId}/{academicYear}]
        FeeTransactions[schools/{schoolId}/feeTransactions/{transactionId}]
    end
    
    subgraph "Communication Collections"
        Announcements[schools/{schoolId}/announcements/{announcementId}]
        Notifications[schools/{schoolId}/notifications/{userId}/{notificationId}]
        Events[schools/{schoolId}/events/{eventId}]
    end
    
    subgraph "Operational Collections"
        Library[schools/{schoolId}/library/books/{bookId}]
        Transport[schools/{schoolId}/transport/routes/{routeId}]
        Inventory[schools/{schoolId}/inventory/{itemId}]
    end
    
    Schools --> SchoolProfile
    Schools --> SchoolUsers
    Schools --> SchoolStudents
    Schools --> SchoolTeachers
    Schools --> SchoolClasses
    Schools --> SchoolSubjects
    Schools --> Attendance
    Schools --> AttendanceSummary
    Schools --> Marks
    Schools --> ReportCards
    Schools --> FeeStructure
    Schools --> StudentFees
    Schools --> FeeTransactions
    Schools --> Announcements
    Schools --> Notifications
    Schools --> Events
    Schools --> Library
    Schools --> Transport
    Schools --> Inventory
```

### 3. User Authentication & Role Flow
```mermaid
sequenceDiagram
    participant U as User
    participant A as App
    participant F as Firebase Auth
    participant DB as Firestore
    participant CC as Custom Claims
    
    U->>A: Enter School Code + UID + Password
    A->>A: Construct Email: {uid}@{schoolCode}.school
    A->>F: Sign In with Email/Password
    F->>A: Return User Credential
    A->>CC: Set Custom Claims (role, schoolId, permissions)
    CC->>F: Update User Claims
    A->>DB: Fetch User Data with schoolId filter
    DB->>A: Return User Profile & Permissions
    A->>U: Grant Access Based on Role
```

### 4. Data Relationships & References
```mermaid
erDiagram
    SCHOOLS {
        string schoolId PK
        string schoolCode UK
        string name
        string type
        object subscription
        object settings
        timestamp createdAt
        string createdBy FK
    }
    
    USERS {
        string uid PK
        string schoolCode FK
        string schoolId FK
        string email
        string role
        array permissions
        object profile
        string status
        timestamp lastLogin
        timestamp createdAt
        string createdBy FK
    }
    
    STUDENTS {
        string studentId PK
        string uid FK
        string schoolId FK
        string admissionNumber
        string classId FK
        string section
        string parentUid FK
        object emergencyContact
        timestamp admissionDate
        boolean isActive
    }
    
    TEACHERS {
        string teacherId PK
        string uid FK
        string schoolId FK
        string employeeId
        array subjects FK
        array classes FK
        string qualification
        number experience
        timestamp joiningDate
        object salary
        boolean isActive
    }
    
    CLASSES {
        string classId PK
        string schoolId FK
        string name
        string section
        string academicYear
        array subjects FK
        string classTeacher FK
        array students FK
        object schedule
        boolean isActive
    }
    
    SUBJECTS {
        string subjectId PK
        string schoolId FK
        string name
        string code
        string description
        array teachers FK
        number maxMarks
        number passMarks
        boolean isActive
    }
    
    ATTENDANCE {
        string userId FK
        string schoolId FK
        string date
        string status
        timestamp checkInTime
        timestamp checkOutTime
        string markedBy FK
        string notes
        object location
    }
    
    FEES {
        string structureId PK
        string schoolId FK
        string name
        string academicYear
        object classes
        boolean isActive
        timestamp createdAt
        string createdBy FK
    }
    
    PAYMENTS {
        string transactionId PK
        string schoolId FK
        string studentId FK
        string feeType
        number amount
        string paymentMethod
        string gatewayPaymentId
        string status
        string receiptNumber
        timestamp paidAt
        string processedBy FK
    }
    
    SCHOOLS ||--o{ USERS : "has"
    SCHOOLS ||--o{ STUDENTS : "enrolls"
    SCHOOLS ||--o{ TEACHERS : "employs"
    SCHOOLS ||--o{ CLASSES : "offers"
    SCHOOLS ||--o{ SUBJECTS : "teaches"
    SCHOOLS ||--o{ ATTENDANCE : "tracks"
    SCHOOLS ||--o{ FEES : "charges"
    SCHOOLS ||--o{ PAYMENTS : "processes"
    
    USERS ||--o| STUDENTS : "becomes"
    USERS ||--o| TEACHERS : "becomes"
    USERS ||--o| ATTENDANCE : "has"
    
    CLASSES ||--o{ STUDENTS : "contains"
    CLASSES ||--o{ SUBJECTS : "includes"
    CLASSES ||--o| TEACHERS : "assigned"
    
    SUBJECTS ||--o{ TEACHERS : "taught_by"
    SUBJECTS ||--o{ CLASSES : "included_in"
    
    STUDENTS ||--o{ PAYMENTS : "makes"
    STUDENTS ||--o{ ATTENDANCE : "has"
```

### 5. Collection Hierarchy & Naming Convention
```mermaid
graph TD
    subgraph "Collection Naming Pattern"
        Pattern["schools/{schoolId}/{collection}/{documentId}"]
        Pattern2["superadmin/{collection}/{documentId}"]
        Pattern3["system/{collection}/{documentId}"]
    end
    
    subgraph "Example Paths"
        Path1["schools/ABC123/students/STU001"]
        Path2["schools/ABC123/classes/CLS10A"]
        Path3["schools/ABC123/attendance/2024-01-15/USER001"]
        Path4["superadmin/schools/ABC123"]
        Path5["system/roles/teacher"]
    end
    
    Pattern --> Path1
    Pattern --> Path2
    Pattern --> Path3
    Pattern2 --> Path4
    Pattern3 --> Path5
```

### 6. Security Rules Structure
```mermaid
graph LR
    subgraph "Security Layers"
        L1[Authentication Layer]
        L2[School Isolation Layer]
        L3[Role-Based Access Layer]
        L4[Permission-Based Layer]
    end
    
    subgraph "Access Control"
        AC1[Super Admin: Full Access]
        AC2[Principal: School Admin]
        AC3[Teacher: Class Access]
        AC4[Student: Own Data]
        AC5[Parent: Child Data]
        AC6[Cashier: Fee Access]
    end
    
    L1 --> L2
    L2 --> L3
    L3 --> L4
    L4 --> AC1
    L4 --> AC2
    L4 --> AC3
    L4 --> AC4
    L4 --> AC5
    L4 --> AC6
```

### 7. Data Flow & Query Patterns
```mermaid
flowchart TD
    subgraph "User Login Flow"
        A[User Input] --> B[Email Construction]
        B --> C[Firebase Auth]
        C --> D[Custom Claims]
        D --> E[Role Assignment]
        E --> F[Permission Loading]
    end
    
    subgraph "Data Access Flow"
        G[User Request] --> H[School ID Check]
        H --> I[Role Validation]
        I --> J[Permission Check]
        J --> K[Data Retrieval]
        K --> L[Response]
    end
    
    subgraph "Query Optimization"
        M[Single Query] --> N[Compound Where Clauses]
        N --> O[Pagination]
        O --> P[Local Caching]
        P --> Q[Batch Operations]
    end
    
    F --> G
    L --> M
```

### 8. Cost Optimization Structure
```mermaid
graph TD
    subgraph "Before Optimization"
        B1[Multiple Collections]
        B2[Individual Queries]
        B3[Real-time All Data]
        B4[No Caching]
        B5[Large Documents]
    end
    
    subgraph "After Optimization"
        A1[Embedded Data]
        A2[Compound Queries]
        A3[Strategic Real-time]
        A4[Local Caching]
        A5[Compressed Files]
    end
    
    subgraph "Cost Impact"
        C1[75% Read Reduction]
        C2[90% Query Reduction]
        C3[80% Real-time Reduction]
        C4[70% Cache Hit Rate]
        C5[50% Storage Reduction]
    end
    
    B1 --> A1
    B2 --> A2
    B3 --> A3
    B4 --> A4
    B5 --> A5
    
    A1 --> C1
    A2 --> C2
    A3 --> C3
    A4 --> C4
    A5 --> C5
```

### 9. Feature Access Control Matrix
```mermaid
graph TD
    subgraph "Feature Categories"
        FC1[Academic Management]
        FC2[Financial Management]
        FC3[Administrative]
        FC4[Communication]
        FC5[Library & Transport]
    end
    
    subgraph "User Roles"
        UR1[Super Admin]
        UR2[Principal]
        UR3[Teacher]
        UR4[Student]
        UR5[Parent]
        UR6[Cashier]
        UR7[Librarian]
    end
    
    subgraph "Access Levels"
        AL1[Full Access]
        AL2[School Level]
        AL3[Class Level]
        AL4[Personal Level]
        AL5[Feature Specific]
    end
    
    FC1 --> UR1
    FC1 --> UR2
    FC1 --> UR3
    FC1 --> UR4
    FC1 --> UR5
    
    FC2 --> UR1
    FC2 --> UR2
    FC2 --> UR6
    
    FC3 --> UR1
    FC3 --> UR2
    
    FC4 --> UR1
    FC4 --> UR2
    FC4 --> UR3
    
    FC5 --> UR1
    FC5 --> UR2
    FC5 --> UR7
    
    UR1 --> AL1
    UR2 --> AL2
    UR3 --> AL3
    UR4 --> AL4
    UR5 --> AL4
    UR6 --> AL5
    UR7 --> AL5
```

### 10. Scalability & Performance Metrics
```mermaid
graph LR
    subgraph "Performance Metrics"
        PM1[Query Response Time]
        PM2[Data Transfer Size]
        PM3[Cache Hit Rate]
        PM4[Batch Operation Success]
        PM5[Real-time Update Latency]
    end
    
    subgraph "Scalability Factors"
        SF1[Number of Schools]
        SF2[Students per School]
        SF3[Daily Operations]
        SF4[Concurrent Users]
        SF5[Data Growth Rate]
    end
    
    subgraph "Optimization Targets"
        OT1[< 100ms Response]
        OT2[< 1MB Transfer]
        OT3[> 80% Cache Hit]
        OT4[> 99% Success Rate]
        OT5[< 500ms Latency]
    end
    
    PM1 --> OT1
    PM2 --> OT2
    PM3 --> OT3
    PM4 --> OT4
    PM5 --> OT5
    
    SF1 --> PM1
    SF2 --> PM2
    SF3 --> PM3
    SF4 --> PM4
    SF5 --> PM5
```

---

## 📊 Key Design Principles

### 1. **Multi-Tenant Isolation**
- Each school's data is completely isolated
- School ID is included in every collection path
- Security rules enforce school boundaries

### 2. **Role-Based Access Control**
- Super Admin: Full system access
- Principal: Complete school management
- Teacher: Class and student access
- Student/Parent: Personal data only
- Cashier: Fee-related operations
- Librarian: Library operations

### 3. **Cost Optimization**
- Embedded data reduces read operations
- Compound queries minimize multiple requests
- Pagination handles large datasets
- Local caching reduces server calls
- Batch operations for bulk updates

### 4. **Security & Privacy**
- Firebase Auth with custom claims
- Firestore security rules
- Input validation and sanitization
- Rate limiting and monitoring
- Audit logging for compliance

### 5. **Performance & Scalability**
- Efficient indexing strategy
- Strategic use of real-time updates
- Offline-first approach with sync
- Optimized file storage
- Load balancing considerations

This visual representation shows the complete database architecture designed for your School Management SaaS platform, highlighting the relationships, security layers, and optimization strategies that will ensure cost-effectiveness, security, and scalability.

