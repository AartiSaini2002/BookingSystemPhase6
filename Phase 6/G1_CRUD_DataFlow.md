# 1️⃣ CREATE – RResource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (form.js and resources.js)
    participant B as Backend (Express Route)
    participant V as express-validator
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Submit form
    F->>F: Client-side validation
    F->>B: POST /api/resources (JSON)

    B->>V: Validate request
    V-->>B: Validation result

    alt Validation fails
        B-->>F: 400 Bad Request + errors[]
        F-->>U: Show validation message
    else Validation OK
        B->>S: create Resource(data)
        S->>DB: INSERT INTO resources
        DB-->>S: Result / Duplicate error

        alt Duplicate
            S-->>B: Duplicate detected
            B-->>F: 409 Conflict
            F-->>U: Show duplicate message
        else Success
            S-->>B: Created resource
            B-->>F: 201 Created
            F-->>U: Show success message
        end
    end
```

# 2️⃣ READ — Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (resources.js)
    participant B as Backend (Express Route)
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Load resources page
    F->>B: GET /api/resources
    
    B->>S: getAllResources()
    S->>DB: SELECT * FROM resources
    DB-->>S: Array of resources
    S-->>B: Resources data
    
    alt Success
        B-->>F: 200 OK (JSON array)
        F-->>U: Display resource list
    else Empty list
        B-->>F: 200 OK (empty array)
        F-->>U: Show "No resources found"
    else Server error
        B-->>F: 500 Internal Server Error
        F-->>U: Show error message
    end
```

# 3️⃣ UPDATE — Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (form.js)
    participant B as Backend (Express Route)
    participant V as express-validator
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Submit updated form
    F->>F: Client-side validation
    F->>B: PUT /api/resources/:id (JSON)

    B->>V: Validate request
    V-->>B: Validation result

    alt Validation fails
        B-->>F: 400 Bad Request + errors[]
        F-->>U: Show validation message
    else Validation OK
        B->>S: updateResource(id, data)
        S->>DB: UPDATE resources WHERE id = :id

        alt Resource not found
            DB-->>S: affectedRows = 0
            S-->>B: Not found
            B-->>F: 404 Not Found
            F-->>U: Show "Resource not found"
        else Success
            DB-->>S: Success
            S-->>B: Updated resource
            B-->>F: 200 OK
            F-->>U: Show "Update successful"
        end
    end
```

# 4️⃣ DELETE — Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (resources.js)
    participant B as Backend (Express Route)
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Click Delete button
    F->>F: Show confirmation dialog
    U->>F: Confirm deletion
    
    F->>B: DELETE /api/resources/:id
    B->>S: deleteResource(id)
    S->>DB: DELETE FROM resources WHERE id = :id
    
    alt Resource not found
        DB-->>S: affectedRows = 0
        S-->>B: null
        B-->>F: 404 Not Found
        F-->>U: Show "Resource not found"
    else Foreign key constraint violation
        DB-->>S: SQL Error (foreign key)
        S-->>B: Conflict error
        B-->>F: 409 Conflict
        F-->>U: Show "Cannot delete: has existing bookings"
    else Deletion successful
        DB-->>S: affectedRows > 0
        S-->>B: Success
        B-->>F: 204 No Content
        F->>F: Remove from UI
        F-->>U: Show "Resource deleted" message
    end
```