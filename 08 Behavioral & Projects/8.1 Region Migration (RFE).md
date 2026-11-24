# Region Migration (RFE)

Dublin → Spain migration of Appstore / FireTV services.

## The "not a developer" problem
- Oracle and Visa heard RFE as DevOps / migration work
- **Fix:** lead with code you wrote
	- Proxy routing / fallback
	- Data reconciliation
	- Traffic-shift + capacity algorithm
	- Rollback automation
- Map each to a pattern: Strangler Fig, CAP trade-off, canary, fail-fast (see the project mapping in [[07 System Design/Design Patterns/README|Design Patterns]])

## Questions to prepare
- What were the hardest problems, and how do you explain RFE as a deep engineering project rather than just a migration?
