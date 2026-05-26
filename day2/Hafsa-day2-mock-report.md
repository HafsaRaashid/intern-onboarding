# BookSwap — Mock Smoke Test Report

## Setup
- Prism command used (`npx @stoplight/prism-cli mock Hafsa-day2-bookswap-openapi.yaml`)
- Postman / Bruno collection link: https://hafsamohammedraashid-9864631.postman.co/workspace/Hafsa-Mohammed-Raashid's-Worksp~10ff78f6-a8b6-4b5d-88d3-51ea72eff888/collection/52995759-ef9ceb96-a3a8-4776-a768-ac5c82cfd5fa?action=share&source=copy-link&creator=52995759

## Results Summary
| Metric | Target | Achieved |
|--------|--------|----------|
| Tests run | 5 | 5 |
| Tests passing against the mock | 5 | 3 |
| Endpoints with explicit error responses | 4+ | 5 |

## Findings

- What did the mock reveal that the OpenAPI on its own did not?
401 was listed as a response but Prism returned 200 with no auth header revealed that no security scheme was actually applied
400 was listed as a response but Prism accepted any body revealing that no required fields were defined

- Which endpoints feel awkward to call?
/books/999/borrow-requests feels awkward to call in isolation because 999 is a dummy ID. In a real flow, the caller would first call GET /books, select a book from the results, and the bookId would be extracted automatically from the response to construct the URL

## Spec changes you would make
- 1. Add security schema to enforce auth when listing books
- 2. Make title required when posting a book