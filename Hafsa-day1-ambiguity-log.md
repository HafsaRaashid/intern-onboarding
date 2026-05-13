# LearnLanka — Ambiguity Hunt Log

## Brief reference
LearnLanka connects Sri Lankan O/L and A/L students with **vetted tutors** for **one-to-one** online sessions. Students should be able to find tutors by subject, language, and price; **book** and pay**; **rate the tutor afterwards**. Tutors **set their availability** and **get paid weekly**. We want it to be **fast**, **secure**, and **ready** before **exam season**.

## Findings
| # | Quote | Why ambiguous | Clarification question | Priority |
|---|-------|---------------|----------------------|----------|
| 1 | "vetted tutors" | Doesn't define the criteria or process for vetting | What criteria must a tutor meet to be considered vetted? Is there an internal system required for vetting?  | H |
| 2 | "one-to-one" |  Doesn't address whether group sessions will  be supported | Are the sessions limited to one-to-one sessions or are group sessions possible ?If so what is the maximum group size? | M | 
| 3 | "book" | Doesn't specify cancellation policy, or refund rules |  What happens if a student or tutor cancels, and is a refund issued? | H |
| 4 | "pay" | Doesn't specify payment methods or who bears transaction fees | What payment methods are supported? Who bears the gateway transaction fee? | H |
| 5 | "rate the tutor afterwards" | Doesn't define the scale, the eligibility to rate, whether only tutors can rate students, if ratings are editable or whether written feedback is supported| What rating format should be used (e.g. stars, written feedback, or both)? Can tutors also rate students? How long after a session can a rating be submitted? | L |
| 6 | "set their availability" | Doesn't specify how far in advance, whether slots can be recurring, or what happens if availability changes after a booking | How far in advance can tutors set availability? Can they set recurring slots? Can they edit availability after a booking is made? | M |
| 7 | "get paid weekly" | Doesn't specify payout method or whether commission is deducted before payout | Is commission deducted before payout? What payment methods will be used? | H |
| 8 | "fast" | No measurable performance target defined | What is the maximum acceptable response time for tutor search and booking actions under normal load? | M |
| 9 | "secure" | No security standard specified | Which security standards must the platform comply with? | H |
| 10 | "exam season" | No specific deadline given | What is the exact launch deadline? Which exam season is being targeted? | H |

## Results Summary
| Metric | Target | Achieved |
|--------|--------|----------|
| Items found | 10+ | 10 |
| High-priority items | 3+ | 6 |
| Items convertible to test cases | 5+ | 9 |

## Top 3 questions to ask the founders
- 1.What criteria must a tutor meet to be considered vetted?
- 2.Which security standards must the platform comply with?
- 3.What is the exact launch deadline? Which exam season is being targeted?

## Reflection
- What kind of ambiguity tripped you up most?
The lack of details related to the security standards
- Which question is most likely to change the architecture if answered?
What criteria must a tutor meet to be considered vetted? Is there an internal system required for vetting?
If vetting requires document uploads, background checks, or admin approval workflows, that adds a whole verification system to the architecture (storage for documents and an admin review interface)