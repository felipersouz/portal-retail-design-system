# Prompt: Create Service

Use this prompt to generate a complete Angular service with full CRUD.

---

## Prompt Template

```
Create an Angular 20 service for [ENTITY_NAME] following the portal-retail patterns.

**Entity:** [ENTITY_NAME]
**Base API URL:** taken from environment.ts (use existing pattern from other services)
**File location:** src/app/core/services/[entity-name].service.ts

**Methods needed:**
- getAll(filters?: [FilterType]): Observable<[Entity][]>
- getById(id: number): Observable<[Entity]>
- create(dto: Create[Entity]Dto): Observable<[Entity]>
- update(id: number, dto: Update[Entity]Dto): Observable<[Entity]>
- delete(id: number): Observable<void>

**Entity interface fields:**
- id: number
- [field1]: [type]
- [field2]: [type]
- createdAt: string
- updatedAt: string

**Requirements:**
1. Use `inject(HttpClient)` 
2. `providedIn: 'root'`
3. Generate TypeScript interfaces: [Entity], Create[Entity]Dto, Update[Entity]Dto
4. Place interfaces in src/app/core/models/[entity-name].model.ts
5. Use `environment.apiUrl` as base URL (check existing services for the pattern)
```

---

## Example (filled)

```
Create an Angular 20 service for Store following the portal-retail patterns.

Entity: Store
File location: src/app/core/services/store.service.ts

Methods needed: getAll, getById, create, update, delete

Entity interface fields:
- id: number
- name: string
- cnpj: string
- address: string
- status: 'active' | 'inactive'
- clientId: number
- createdAt: string

Requirements: [same as above]
```
