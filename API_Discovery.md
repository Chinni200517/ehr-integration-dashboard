# API Discovery — DrChrono (selected endpoints)

This document lists the endpoints used for the CRUD dashboard demo along with sample requests/responses and notes.

## Patient Management
- `GET https://drchrono.com/api/patients`  
  - Query params: `first_name`, `last_name`, `page`, `limit`, `search`  
  - Sample response (trimmed):
  ```json
  {
    "meta": { "next": null },
    "results": [
      {
        "id": 12345,
        "first_name": "John",
        "last_name": "Doe",
        "date_of_birth": "1980-02-14",
        "phone_number": "555-1234",
        "email": "john@example.com",
        "medical_record_number": "MR-001"
      }
    ]
  }
  ```

- `GET https://drchrono.com/api/patients/{id}` — retrieve patient details
- `PUT https://drchrono.com/api/patients/{id}` — update demographics/contact (payload: only updatable fields)
- `POST https://drchrono.com/api/patients` — create new patient (depends on account permissions)

## Appointments
- `GET https://drchrono.com/api/appointments`  
  - Params: `date`, `doctor`, `patient`, `page`
- `POST https://drchrono.com/api/appointments` — create (requires provider id, patient id, start time)
- `PUT https://drchrono.com/api/appointments/{id}` — reschedule or update
- `DELETE https://drchrono.com/api/appointments/{id}` — cancel

## Clinical
- `GET https://drchrono.com/api/notes?patient={patient_id}` — fetch notes
- `POST https://drchrono.com/api/notes` — add clinical note
- `GET https://drchrono.com/api/lab_results?patient={id}` — lab results (read)

## Billing & Admin
- `GET https://drchrono.com/api/transactions?patient={id}` — payments / balances
- `GET https://drchrono.com/api/eligibility?patient={id}` — insurance eligibility (if available)

## Common patterns & headers
- Authorization: `Bearer <access_token>`
- Content-Type: `application/json`

## Limitations & Notes
- Pagination: many endpoints use paginated results with `next` links.
- Rate limits: handle 429 with backoff.
- Some write operations (labs, billing) may require additional account privileges.