# LearnLanka — User Story Set v0.1

## Story 1: Tutor Search and Filter
**As a** Student
**I want** to filter tutors by subject, grade level, language, and price range
**So that** I can easily find suitable tutors

### Acceptance Criteria
- **Given** I am on the tutor search page **when** I select a subject, grade level, language, and price range **then** only tutors matching all selected filters are displayed
- **Given** I apply a filter **when** no tutors match **then** a "no results found" message is displayed
- **Given** I clear all filters **when** finding for another tutor **then** all suitable tutors are displayed

### INVEST self-check
- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

## Story 2: Online Payment
**As a** Student
**I want** to pay for sessions online using card or eZ Cash
**So that** payments are convenient and secure

### Acceptance Criteria
- **Given** I have a confirmed booking **when** I choose to pay via card or eZ Cash and complete the transaction **then** the payment is processed and I receive a receipt
- **Given** that I don't have balance **when** I confirm the payment **then** I am notified and the booking is not confirmed
- **Given** a payment is completed **when** I view my booking **then** the status shows as paid

### INVEST self-check
- [x] Independent 
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

## Story 3:  Set Tutor Availability
**As a** Tutor
**I want** to set my availability
**So that** students can only book sessions when I am available

### Acceptance Criteria
- **Given** I am on my availability page **when** I mark a time slot as available and save **then** that slot becomes bookable by students
- **Given** I mark a slot as unavailable **when** a student views my profile **then** that slot is not shown as bookable
- **Given** I update my availability **when** a student has already booked that slot **then** the existing booking is not affected

### INVEST self-check
- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

## Story 4: Personal Data Protection
**As a** Tutor
**I want** my personal data to be stored and handled in compliance with the Sri Lanka Personal Data Protection Act 2022
**So that** I can trust the platform with my information

### Acceptance Criteria
- **Given** I register on the platform **when** I submit my personal details **then** I am shown a consent notice and must explicitly agree before my data is stored
- **Given** I request deletion of my account **when** the request is submitted **then** my personal data is deleted within the timeframe required by PDPA 2022
- **Given** my data is stored **when** a third party requests access **then** my data is not shared without my consent

### INVEST self-check
- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [ ] Small — PDPA compliance spans multiple flows (consent, deletion, data sharing)
- [x] Testable

## Story 5: Monitor Bookings 
**As an** Operations Admin
**I want** to monitor student bookings and payments
**So that** I can ensure the platform operates smoothly

### Acceptance Criteria
- **Given** I am logged in as an admin **when** I open the dashboard **then** I can see all active, completed, and cancelled bookings
- **Given** I view the payments section **when** I select a booking **then** I can see the payment status and commission deducted
- **Given** a payment fails **when** I view the dashboard **then** the failed transaction is flagged for review
- **Given** a booking is canceled by a teacher before 12 hours **when** I view the dashboard **then** the student is refunded

### INVEST self-check
- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

## Story 6: Book a Tutoring Session
**As a** Student
**I want** to book a 1-hour tutoring session with a tutor
**So that** I can schedule lessons in advance

### Acceptance Criteria
- **Given** I am viewing a tutor's profile **when** I select an available slot and confirm **then** a booking request is sent to the tutor
- **Given** I submit a booking request **when** the tutor accepts **then** I receive a confirmation notification
- **Given** I try to book a slot **when** it is already taken **then** the slot is not selectable

### INVEST self-check
- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

