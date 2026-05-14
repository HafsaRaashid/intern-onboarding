LearnLanka — Requirements Document

## 1\. Problem Statement



Many Sri Lankan O/L and A/L students find it difficult to find qualified tutors who match their subject needs, preferred language, and budget. When searching for tuitions it is often difficult for students to compare tutors, schedule sessions, and make secure payments online.

At the same time, tutors face challenges in managing their availability, reaching students efficiently, and receiving payments. During exam seasons, these issues become more significant due to the increased demand for tutoring services.

In conclusion , there is a need for a fast, secure, and user-friendly online platform that connects students with vetted tutors for one-to-one learning sessions. The system should allow students to search tutors by subject, language, and price, book and pay for sessions online, and provide ratings after classes. Tutors should be able to manage their schedules and receive payments conveniently through the platform.



## 2\. Personas

1. O/L Student Persona - “Manula”
2. 

Goals:

Find qualified tutors who teach O/L subjects
Find tutors who teach in English medium
Find tutors who conduct one-to-one online sessions
Find tutors within an affordable budget
Book and pay online
Rate teacher based on experience

Frustrations:

Difficulty in finding qualified teachers
Difficulty in finding classes within the budget
Difficulty in meeting the language requirement



2. Math Tutor Persona - “Yasmin”

Goals:

Find O/L students who need Math tutoring
Receive payments on time securely
Schedule classes effectively

Frustrations:

Difficulty in finding students
Communication issues with students
Delays or issues in payments



3. Operations Admin Persona - "Yahya"

Goals:
Handle feedback and complaints effectively
Monitor bookings and payments

Frustrations:
Difficulty in tracking payments efficiently
Difficulty in handling issues between students and teachers



## 3\. Functional Requirements

Student
Students shall be able to filter tutors by subject, grade level, language, and price range.
Students shall be able to rate tutors (1–5 stars) and leave a one-line comment after a completed session.
Students shall be able to book a 1-hour tutoring session with a tutor.
Students shall be able to pay for sessions online via card or eZ Cash.

Tutor
Tutors shall be able to publish availability slots for students to book.
Tutors shall be able to accept or decline booking requests.
Tutors shall be able to cancel sessions with at least 12 hours notice.
Tutors shall receive weekly payments via bank transfer.

Operations Admin
Admins shall have a dashboard to monitor bookings and payments.
Admins shall be able to receive and manage complaints and feedback from both parties.

System
The platform shall automatically deduct a 15% commission from each completed session before paying out to the tutor.



## 4\. Non-Functional Requirements



| Category     | Metric                                 | Target                                                                        | How we'll measure it                          |

|--------------|----------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------|

| Speed        | Search results return time             | Results returned in under 800 ms at the 95th percentile from a Sri Lankan ISP | Monitor response time for Tutor search results|

| Availability | Uptime                                 | 99.5% monthly uptime                                                          | Monitor booking endpoint                      |

| Concurrency  | Simultaneous video sessions            | Support 200 simultaneous video sessions in the first 6 months                 | Load testing with simulated concurrent users  |

| Privacy      | Sri Lanka Personal Data Protection Act | Comply with Sri Lanka Personal Data Protection Act 2022                       | Compliance testing                            |

| Security     | Payment data                           | Not stored on LearnLanka servers, use a PCI-DSS compliant gateway             | Security audits                               |



## 5\. Assumptions

1. Users register with email and password; email verification is required before booking.
2. The 12-hour cancellation notice window applies to both tutors and students.
3. Cancelled sessions are refunded to the student in full.
4. Booking requests not accepted/declined by the tutor within 24 hours auto-expire.
5. The 15% commission is deducted before the tutor payout, not charged separately to the student.
6. Video sessions are conducted entirely through the third-party provider, LearnLanka does not record or store session content.
7. If a video session fails due to a third-party outage, the session is rescheduled, not auto-refunded.
8. The Operations Admin requires a dashboard to monitor bookings, payments, and both party profiles, and a management interface.

## 6\. Out of Scope



1. Video conferencing infrastructure - video sessions are supported via a third-party provider (Daily.co or 100ms); LearnLanka does not build or host its own video service.
2. In-app messaging/chat - no direct messaging feature between student and tutor outside of the session
3. Native mobile app — LearnLanka is a web application optimised for mobile browsers; a dedicated Android or iOS app distributed via the Play Store or App Store is not being built.
4. Bank payout system — Sampath Vishwa handles payouts; LearnLanka is not building a custom payout system

