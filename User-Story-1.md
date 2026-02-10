# Story ET-101: Create Building for Client

## User Story
**As a** Metergy energy consultant  
**I want to** add a new building to a client's portfolio  
**So that** I can start tracking energy consumption for that location

## Acceptance Criteria
✅ Given I am authenticated as a consultant
   When I POST to /api/clients/{clientId}/buildings with valid data
   Then a new building is created and returned with a 201 status

✅ Given I am authenticated as a consultant
   When I POST with missing required fields
   Then I receive a 400 Bad Request with validation errors

✅ Given I am authenticated as a client user
   When I try to create a building
   Then I receive a 403 Forbidden (only consultants can create)

✅ Given the client doesn't exist
   When I try to create a building for that client
   Then I receive a 404 Not Found

✅ Given a building name already exists for that client
   When I try to create another building with the same name
   Then I receive a 409 Conflict

## Technical Notes
- Building name must be unique per client (can duplicate across clients)
- Square footage must be > 0
- Year built must be between 1800 and current year + 1

## Definition of Done
- Code written and committed
- Unit tests passing (>80% coverage)
- Integration tests passing
- Code reviewed and approved
- Merged to develop branch
- Manual testing in DEV environment completed

```
** Story Points:** 5

** Sprint:** 1

---

## Tasks Breakdown
```

- Task 1: Set up project structure (Clean Architecture)
- Task 2: Create domain entities (Client, Building)
- Task 3: Create DTOs and validation
- Task 4: Create repository interfaces
- Task 5: Implement repositories
- Task 6: Create service layer
- Task 7: Create controller endpoint
- Task 8: Add authorization
- Task 9: Write unit tests
- Task 10: Write integration tests
- Task 11: Update Swagger documentation