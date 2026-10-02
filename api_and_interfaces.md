# APIs & Interfaces

| Component                    | Description                                                            |
| ---------------------------- | ---------------------------------------------------------------------- |
| [Base Response Envelope](#base-response-envelope) | Standardized API response format for all services      |
| [core](#core)                 | Authentication and device pairing                                      |
| [registry](#registry)         | Plant registration, and which owner, species and device each plant has |
| [knowledge](#knowledge)       | Probe generation from research documents, and its review               |
| [orchestrator](#orchestrator) | Runs the scheduled pipeline                                            |
| [capture](#capture)           | TODO                                                                   |
| [assessment](#assessment)     | TODO                                                                   |
| [advice](#advice)             | TODO                                                                   |
| [companion](#companion)       | Character dialogue, plant voice, and care status presented to the web client / UI |

<br>

---

# Base Response Envelope

All API responses across all components follow a standardized top-level envelope (`BaseResponse<T>`). The actual payload model is nested inside the `data` field:

## Success Response (`2xx`)

```json
{
  "success": true,
  "data": { ... },
  "message": null,
  "error": null
}
```

## Failure Response (`4xx` / `5xx`)

```json
{
  "success": false,
  "data": null,
  "message": "Human-readable explanation of the error or status.",
  "error": {
    "code": "ERROR_CODE",
    "details": { ... }
  }
}
```

## Envelope Fields

| Field | Type | Requirement | Description |
| :--- | :--- | :---: | :--- |
| `success` | `boolean` | **Mandatory** | `true` if the request succeeded (`2xx`), `false` on failure (`4xx`/`5xx`). |
| `data` | `object` \| `array` \| `null` | **Nullable** | The response payload on success; `null` on failure. |
| `message` | `string` \| `null` | **Optional (Nullable)** | Human-readable status or diagnostic explanation; `null` if none. |
| `error` | `object` \| `null` | **Optional (Nullable)** | Structured error details when `success: false`; `null` on success. |
| `error.code` | `string` | **Mandatory on error** | Machine-readable error code (e.g. `"RESOURCE_NOT_FOUND"`, `"UNAUTHORIZED"`, `"INTERNAL_SERVER_ERROR"`). |
| `error.details` | `object` \| `null` | **Optional (Nullable)** | Additional contextual validation or debugging information. |

<br>

---

# core

Authentication and device pairing

## API

### `GET /health`

Liveness check (public)

**Response**

```json
{
  "status": "ok"
}
```

---

### `POST /auth/register`

Register a user (public)

**Request**

```json
{
  "email": "string",
  "name": "string",
  "password": "string"
}
```

**Response**

```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "is_active": "boolean",
  "is_superuser": "boolean",
  "created_at": "datetime"
}
```

---

### `POST /auth/token`

Log in and get a JWT. Sent as a form (public)

**Request**

```json
{
  "username": "string",
  "password": "string"
}
```

**Response**

```json
{
  "access_token": "string",
  "token_type": "bearer"
}
```

---

### `GET /auth/me`

The signed-in user (user)

**Response**

```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "is_active": "boolean",
  "is_superuser": "boolean",
  "created_at": "datetime"
}
```

---

### `POST /devices/pair`

The device starts pairing and gets a code to show on its display (public)

**Request**

```json
{
  "physical_id": "string"
}
```

**Response**

```json
{
  "device_code": "string",
  "user_code": "string",
  "expires_in": "integer",
  "interval": "integer"
}
```

---

### `POST /devices/pair/approve`

The owner enters the code and approves the device (user)

**Request**

```json
{
  "user_code": "string",
  "name": "string | null"
}
```

**Response**

```json
{
  "id": "uuid",
  "physical_id": "string",
  "owner_id": "uuid",
  "name": "string | null",
  "status": "active | revoked",
  "paired_at": "datetime",
  "revoked_at": "datetime | null",
  "last_seen_at": "datetime | null",
  "created_at": "datetime"
}
```

---

### `POST /devices/pair/token`

The device collects its token once approved (public)

**Request**

```json
{
  "device_code": "string"
}
```

**Response**

```json
{
  "access_token": "string",
  "token_type": "bearer",
  "device_id": "uuid",
  "physical_id": "string"
}
```

---

### `GET /devices/me`

The device itself (device)

**Response**

```json
{
  "id": "uuid",
  "physical_id": "string",
  "owner_id": "uuid",
  "name": "string | null",
  "status": "active | revoked",
  "paired_at": "datetime",
  "revoked_at": "datetime | null",
  "last_seen_at": "datetime | null",
  "created_at": "datetime"
}
```

---

### `GET /devices`

My devices (user)

**Response**

```json
[
  {
    "id": "uuid",
    "physical_id": "string",
    "owner_id": "uuid",
    "name": "string | null",
    "status": "active | revoked",
    "paired_at": "datetime",
    "revoked_at": "datetime | null",
    "last_seen_at": "datetime | null",
    "created_at": "datetime"
  }
]
```

---

### `GET /devices/{device_id}`

One device (user)

**Response**

```json
{
  "id": "uuid",
  "physical_id": "string",
  "owner_id": "uuid",
  "name": "string | null",
  "status": "active | revoked",
  "paired_at": "datetime",
  "revoked_at": "datetime | null",
  "last_seen_at": "datetime | null",
  "created_at": "datetime"
}
```

---

### `POST /devices/{device_id}/revoke`

Revoke a device (user)

**Response**

```json
{
  "id": "uuid",
  "physical_id": "string",
  "owner_id": "uuid",
  "name": "string | null",
  "status": "active | revoked",
  "paired_at": "datetime",
  "revoked_at": "datetime | null",
  "last_seen_at": "datetime | null",
  "created_at": "datetime"
}
```

## Interfaces

### `core.devices.get_device_by_physical_id(session, physical_id)`

Returns a device → registry

**Returns**

```json
{
  "id": "uuid",
  "physical_id": "string",
  "owner_id": "uuid",
  "name": "string | null",
  "status": "active | revoked",
  "paired_at": "datetime",
  "revoked_at": "datetime | null",
  "last_seen_at": "datetime | null",
  "created_at": "datetime"
}
```

<br>

---

# registry

Plant registration, and which owner, species and device each plant has

## API

### `POST /registry/plants`

Register a plant (user)

**Request**

```json
{
  "name": "string",
  "species_code": "string | null",
  "species_confirmed": "boolean",
  "device_id": "string | null",
  "note": "string | null",
  "owner_id": "uuid | null"
}
```

**Response**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `GET /registry/plants`

My plants (user)

**Query**: `owner_id`, `species_code`, `include_archived`

**Response**

```json
[
  {
    "id": "uuid",
    "owner_id": "uuid",
    "name": "string",
    "species_code": "string | null",
    "species_confirmed_at": "datetime | null",
    "device_id": "string | null",
    "note": "string | null",
    "created_at": "datetime",
    "updated_at": "datetime",
    "archived_at": "datetime | null"
  }
]
```

---

### `GET /registry/plants/{plant_id}`

One plant (user)

**Response**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `PATCH /registry/plants/{plant_id}`

Update a plant. Only the fields sent are changed (user)

**Request**

```json
{
  "name": "string",
  "species_code": "string | null",
  "species_confirmed": "boolean",
  "device_id": "string | null",
  "note": "string | null"
}
```

**Response**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `DELETE /registry/plants/{plant_id}`

Archive a plant (user)

**Response**

204 No Content

---

### `GET /registry/devices/me/plant`

The plant this device photographs (device)

**Response**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `GET /registry/devices/{device_id}/plant`

The plant bound to the given device (user)

**Response**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

## Interfaces

### `registry.list_plant_ids(session, owner_id=None, species_code=None, include_archived=False)`

Returns plant ids → orchestrator

**Returns**

```json
[
  "uuid"
]
```

---

### `registry.get_plant(session, plant_id)`

Returns one plant → capture, assessment, advice, companion

**Returns**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `registry.get_plants(session, plant_ids)`

Returns several plants → any component

**Returns**

```json
{
  "<plant_id>": {
    "id": "uuid",
    "owner_id": "uuid",
    "name": "string",
    "species_code": "string | null",
    "species_confirmed_at": "datetime | null",
    "device_id": "string | null",
    "note": "string | null",
    "created_at": "datetime",
    "updated_at": "datetime",
    "archived_at": "datetime | null"
  }
}
```

---

### `registry.get_plant_by_device(session, device_id)`

Returns the plant a device photographs → capture

**Returns**

```json
{
  "id": "uuid",
  "owner_id": "uuid",
  "name": "string",
  "species_code": "string | null",
  "species_confirmed_at": "datetime | null",
  "device_id": "string | null",
  "note": "string | null",
  "created_at": "datetime",
  "updated_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `registry.is_plant_owned_by(session, plant_id, user_id)`

Returns whether a user owns a plant → any component

**Returns**

```json
"boolean"
```

<br>

---

# knowledge

Probe generation from research documents, and its review

## API

### `POST /knowledge/species`

Register a species (admin)

**Request**

```json
{
  "species_code": "string",
  "scientific_name": "string",
  "common_name": "string | null"
}
```

**Response**

```json
{
  "species_code": "string",
  "scientific_name": "string",
  "common_name": "string | null"
}
```

---

### `GET /knowledge/species`

Species list (admin)

**Response**

```json
[
  {
    "species_code": "string",
    "scientific_name": "string",
    "common_name": "string | null"
  }
]
```

---

### `POST /knowledge/documents`

Add a research document (admin)

**Request**

```json
{
  "species_code": "string",
  "title": "string",
  "body": "string",
  "author": "string",
  "source_url": "string | null",
  "source_note": "string | null"
}
```

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "title": "string",
  "body": "string",
  "content_hash": "string",
  "author": "string",
  "source_url": "string | null",
  "source_note": "string | null",
  "status": "active | archived",
  "created_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `GET /knowledge/documents`

Research documents (admin)

**Query**: `species_code`, `status`

**Response**

```json
[
  {
    "id": "uuid",
    "species_code": "string",
    "title": "string",
    "body": "string",
    "content_hash": "string",
    "author": "string",
    "source_url": "string | null",
    "source_note": "string | null",
    "status": "active | archived",
    "created_at": "datetime",
    "archived_at": "datetime | null"
  }
]
```

---

### `GET /knowledge/documents/pending`

Research documents with no probe set yet (admin)

**Query**: `species_code`

**Response**

```json
[
  {
    "id": "uuid",
    "species_code": "string",
    "title": "string",
    "body": "string",
    "content_hash": "string",
    "author": "string",
    "source_url": "string | null",
    "source_note": "string | null",
    "status": "active | archived",
    "created_at": "datetime",
    "archived_at": "datetime | null"
  }
]
```

---

### `GET /knowledge/documents/{document_id}`

One research document (admin)

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "title": "string",
  "body": "string",
  "content_hash": "string",
  "author": "string",
  "source_url": "string | null",
  "source_note": "string | null",
  "status": "active | archived",
  "created_at": "datetime",
  "archived_at": "datetime | null"
}
```

---

### `POST /knowledge/probe-sets/generate`

Generate probe sets for documents that have none (admin)

**Query**: `species_code`, `limit`

**Response**

```json
{
  "considered": "integer",
  "generated": [
    {
      "document_id": "uuid",
      "species_code": "string",
      "probe_set_id": "uuid | null",
      "probe_count": "integer",
      "error": "string | null"
    }
  ],
  "failed": [
    {
      "document_id": "uuid",
      "species_code": "string",
      "probe_set_id": "uuid | null",
      "probe_count": "integer",
      "error": "string | null"
    }
  ]
}
```

---

### `GET /knowledge/species/{species_code}/probe-sets`

A species' probe sets (admin)

**Response**

```json
[
  {
    "id": "uuid",
    "species_code": "string",
    "research_document_id": "uuid",
    "llm_model": "string",
    "prompt_version": "string | null",
    "status": "draft | approved | archived",
    "generated_at": "datetime",
    "approved_by": "string | null",
    "approved_at": "datetime | null",
    "rejected_by": "string | null",
    "rejected_at": "datetime | null",
    "archived_at": "datetime | null",
    "note": "string | null",
    "is_stale": "boolean",
    "probe_count": "integer"
  }
]
```

---

### `GET /knowledge/probe-sets/{probe_set_id}`

One probe set (admin)

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "research_document_id": "uuid",
  "llm_model": "string",
  "prompt_version": "string | null",
  "status": "draft | approved | archived",
  "generated_at": "datetime",
  "approved_by": "string | null",
  "approved_at": "datetime | null",
  "rejected_by": "string | null",
  "rejected_at": "datetime | null",
  "archived_at": "datetime | null",
  "note": "string | null",
  "is_stale": "boolean",
  "probe_count": "integer",
  "probes": [
    {
      "id": "uuid",
      "slug": "string",
      "care_need": "string",
      "priority": "integer",
      "is_screening": "boolean",
      "question": "string",
      "worse_looks_like": "string",
      "better_looks_like": "string",
      "not_this": "string",
      "evidence_quote": "string | null",
      "actions": [
        {
          "id": "uuid",
          "ordering": "integer",
          "instruction": "string",
          "urgency": "today | this_week | this_month",
          "expect_typical_hours": "integer",
          "expect_max_hours": "integer",
          "expected_signal": "string | null"
        }
      ],
      "exemplars": [
        {
          "id": "uuid",
          "role": "string",
          "storage_key": "string",
          "content_type": "string | null",
          "label": "object | null",
          "origin": "own_capture | web",
          "license": "string | null",
          "caption": "string | null"
        }
      ]
    }
  ],
  "trace_url": "string | null"
}
```

---

### `POST /knowledge/probe-sets/{probe_set_id}/approve`

Approve a probe set (admin)

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "research_document_id": "uuid",
  "llm_model": "string",
  "prompt_version": "string | null",
  "status": "draft | approved | archived",
  "generated_at": "datetime",
  "approved_by": "string | null",
  "approved_at": "datetime | null",
  "rejected_by": "string | null",
  "rejected_at": "datetime | null",
  "archived_at": "datetime | null",
  "note": "string | null",
  "is_stale": "boolean",
  "probe_count": "integer"
}
```

---

### `POST /knowledge/probe-sets/{probe_set_id}/reject`

Reject a probe set (admin)

**Request**

```json
{
  "comment": "string"
}
```

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "research_document_id": "uuid",
  "llm_model": "string",
  "prompt_version": "string | null",
  "status": "draft | approved | archived",
  "generated_at": "datetime",
  "approved_by": "string | null",
  "approved_at": "datetime | null",
  "rejected_by": "string | null",
  "rejected_at": "datetime | null",
  "archived_at": "datetime | null",
  "note": "string | null",
  "is_stale": "boolean",
  "probe_count": "integer"
}
```

---

### `POST /knowledge/probe-sets/{probe_set_id}/archive`

Archive the approved probe set (admin)

**Response**

```json
{
  "id": "uuid",
  "species_code": "string",
  "research_document_id": "uuid",
  "llm_model": "string",
  "prompt_version": "string | null",
  "status": "draft | approved | archived",
  "generated_at": "datetime",
  "approved_by": "string | null",
  "approved_at": "datetime | null",
  "rejected_by": "string | null",
  "rejected_at": "datetime | null",
  "archived_at": "datetime | null",
  "note": "string | null",
  "is_stale": "boolean",
  "probe_count": "integer"
}
```

---

### `GET /knowledge/species/{species_code}/probes`

A species' approved probes and actions (admin)

**Response**

```json
{
  "species_code": "string",
  "probe_set_id": "uuid",
  "status": "approved",
  "is_stale": "boolean",
  "generated_at": "datetime",
  "probes": [
    {
      "id": "uuid",
      "slug": "string",
      "care_need": "string",
      "priority": "integer",
      "is_screening": "boolean",
      "question": "string",
      "worse_looks_like": "string",
      "better_looks_like": "string",
      "not_this": "string",
      "evidence_quote": "string | null",
      "actions": [
        {
          "id": "uuid",
          "ordering": "integer",
          "instruction": "string",
          "urgency": "today | this_week | this_month",
          "expect_typical_hours": "integer",
          "expect_max_hours": "integer",
          "expected_signal": "string | null"
        }
      ],
      "exemplars": [
        {
          "id": "uuid",
          "role": "string",
          "storage_key": "string",
          "content_type": "string | null",
          "label": "object | null",
          "origin": "own_capture | web",
          "license": "string | null",
          "caption": "string | null"
        }
      ]
    }
  ]
}
```

## Interfaces

### `knowledge.get_species_probes(session, species_code)`

Returns a species' approved probes and actions → assessment, advice

**Returns**

```json
{
  "species_code": "string",
  "probe_set_id": "uuid",
  "status": "approved",
  "is_stale": "boolean",
  "generated_at": "datetime",
  "probes": [
    {
      "id": "uuid",
      "slug": "string",
      "care_need": "string",
      "priority": "integer",
      "is_screening": "boolean",
      "question": "string",
      "worse_looks_like": "string",
      "better_looks_like": "string",
      "not_this": "string",
      "evidence_quote": "string | null",
      "actions": [
        {
          "id": "uuid",
          "ordering": "integer",
          "instruction": "string",
          "urgency": "today | this_week | this_month",
          "expect_typical_hours": "integer",
          "expect_max_hours": "integer",
          "expected_signal": "string | null"
        }
      ],
      "exemplars": [
        {
          "id": "uuid",
          "role": "string",
          "storage_key": "string",
          "content_type": "string | null",
          "label": "object | null",
          "origin": "own_capture | web",
          "license": "string | null",
          "caption": "string | null"
        }
      ]
    }
  ]
}
```

---

### `knowledge.get_species(session, species_code)`

Returns a species → registry

**Returns**

```json
{
  "species_code": "string",
  "scientific_name": "string",
  "common_name": "string | null"
}
```

<br>

---

# orchestrator

Runs the scheduled pipeline

## API

### `GET /orchestrator/runs`

Pipeline runs (admin)

**Query**: `status`, `plant_id`, `limit`

**Response**

```json
[
  {
    "id": "uuid",
    "plant_id": "uuid",
    "scheduled_for": "datetime",
    "trigger": "schedule | manual",
    "status": "queued | running | succeeded | failed | skipped",
    "current_stage": "string",
    "detail": "string | null",
    "created_at": "datetime",
    "started_at": "datetime | null",
    "finished_at": "datetime | null"
  }
]
```

---

### `GET /orchestrator/runs/{run_id}`

One pipeline run (admin)

**Response**

```json
{
  "id": "uuid",
  "plant_id": "uuid",
  "scheduled_for": "datetime",
  "trigger": "schedule | manual",
  "status": "queued | running | succeeded | failed | skipped",
  "current_stage": "string",
  "detail": "string | null",
  "created_at": "datetime",
  "started_at": "datetime | null",
  "finished_at": "datetime | null"
}
```

---

### `POST /orchestrator/runs`

Run the pipeline for a plant now (admin)

**Request**

```json
{
  "plant_id": "uuid"
}
```

**Response**

```json
{
  "id": "uuid",
  "plant_id": "uuid",
  "scheduled_for": "datetime",
  "trigger": "schedule | manual",
  "status": "queued | running | succeeded | failed | skipped",
  "current_stage": "string",
  "detail": "string | null",
  "created_at": "datetime",
  "started_at": "datetime | null",
  "finished_at": "datetime | null"
}
```

---

### `GET /orchestrator/schedule`

The schedule in effect (admin)

**Response**

```json
{
  "enabled": "boolean",
  "timezone": "string",
  "schedule": [
    "string"
  ],
  "next_slot": "datetime | null",
  "catch_up_minutes": "integer",
  "stages": [
    "string"
  ]
}
```

## Interfaces

None

<br>

---

# capture

TODO

## API

TODO

## Interfaces

TODO

<br>

---

# assessment

TODO

## API

None

## Interfaces

TODO

<br>

---

# advice

TODO

## API

None

## Interfaces

TODO

<br>

---

# companion

Character dialogue, plant voice, and care status presented to the web client / UI

## API

### `GET /companion/devices/me/state`

The character's current state, care plan, and 1st-person plant voice message for the plant this device shows (device / client)

**Query**: `plant_id`

The post-VLM pipeline processes daily plant observations and persists state directly to the database. The backend server (MVCS) queries this state via repository/service and exposes `GET /companion/devices/me/state` wrapped in the standard `BaseResponse` envelope (where `data` contains the `CompanionState` object). The web client uses this payload to update the UI across three delivery modes:

1. **Steady / Healthy (`NO_ACTION`)**: Silent timeline update. Green indicator in UI, cheerful companion check-in card, zero alert popup, `care_plan: null`.
2. **Action Needed (`CARE_ADVICE_REQUIRED`)**: High-priority alert banner, red/amber indicator, interactive care card with prioritized action checklist, 2–3 word button labels, and 1st-person plant voice.
3. **Fallback Photo Retake (`REQUEST_MORE_INFORMATION`)**: Friendly photo retake prompt in UI when image quality or consensus falls below threshold (`< 0.50`), with `care_plan: null`.

**Response Schema**

```json
{
  "success": "boolean",
  "data": {
    "plant_id": "string",
    "name": "string",
    "species": "string",
    "dayCount": "integer",
    "timestamp": "datetime (ISO-8601 UTC)",
    "wateredTimestamp": "datetime (ISO-8601 UTC) | null",
    "level": "integer",
    "xpRatio": "float (0.0 - 1.0)",
    "health_status": "healthy | possibly_unhealthy | unhealthy",
    "decision": "NO_ACTION | CARE_ADVICE_REQUIRED | REQUEST_MORE_INFORMATION",
    "companion_message": "string",
    "care_plan": {
      "id": "string",
      "status_label": "string",
      "assessment": "string",
      "actions": [
        {
          "id": "string",
          "priority": "integer",
          "action": "string",
          "label": "string (max 30 chars)",
          "type": "water | move | inspect | other"
        }
      ]
    }
  },
  "message": "string | null",
  "error": {
    "code": "string",
    "details": "object | null"
  } | null
}
```

#### `data` Payload Data Dictionary (Companion State)

| Field | Type | Requirement | Allowed Values | Description & Frontend UI Mapping |
| :--- | :--- | :---: | :--- | :--- |
| `plant_id` | `string` | **Mandatory** | String identifier | Target plant ID to route to the correct UI screen. |
| `name` | `string` | **Mandatory** | Non-empty string (e.g. `"Monty"`) | Plant nickname displayed at the top of the screen. |
| `species` | `string` | **Mandatory** | Botanical name (e.g. `"Monstera deliciosa"`) | Botanical species reference. |
| `dayCount` | `integer` | **Mandatory** | $\\ge 1$ | 1-indexed sequential observation counter for this plant. |
| `timestamp` | `string` | **Mandatory** | ISO-8601 UTC string | Timestamp of the processed scan. |
| `wateredTimestamp` | `string` \| `null` | **Optional (Nullable)** | ISO-8601 UTC string or `null` | Timestamp of last watering event recorded in `care_events`; `null` if unwatered. |
| `level` | `integer` | **Mandatory** | $\\ge 1$ (default `1`) | Gamification character level. |
| `xpRatio` | `float` | **Mandatory** | `0.0` to `1.0` (default `0.0`) | Progress ratio towards next level for UI progress bar. |
| `health_status` | `string` | **Mandatory** | `"healthy"` \| `"possibly_unhealthy"` \| `"unhealthy"` | Status badge color (Green / Amber / Red). |
| `decision` | `string` | **Mandatory** | `"NO_ACTION"` \| `"CARE_ADVICE_REQUIRED"` \| `"REQUEST_MORE_INFORMATION"` | Controls whether to render care card, silent timeline, or photo retake prompt. |
| `companion_message` | `string` | **Mandatory** | Non-empty string | **1st-person speech bubble** spoken by the plant. |
| `care_plan` | `object` \| `null` | **Optional (Nullable)** | `null` on `NO_ACTION` / `REQUEST_MORE_INFORMATION`, Object on `CARE_ADVICE_REQUIRED` | Detailed botanical action plan (See `care_plan` schema below); `null` when healthy/retake. |

#### `care_plan` Object Schema

| Field | Type | Requirement | Description |
| :--- | :--- | :---: | :--- |
| `id` | `string` | **Mandatory** | Unique identifier for this care plan (e.g. `"cp_a7b8c9d0e1f2"`). |
| `status_label` | `string` | **Mandatory** | Short summary headline for the care card (e.g. `"Overwatering stress"`). |
| `assessment` | `string` | **Mandatory** | Botanical reasoning explaining root cause (e.g. overwatering, fungal). |
| `actions` | `list[object]` | **Mandatory** | Ordered list of action items (`[{"id": "...", "priority": 1, ...}]`, see below). |


#### `care_plan.actions[]` Object Schema

| Field | Type | Requirement | Allowed Values | Description |
| :--- | :--- | :---: | :--- | :--- |
| `id` | `string` | **Mandatory** | e.g. `"act_8e4b1a2c"` | Unique action identifier for tracking user completions. |
| `priority` | `integer` | **Mandatory** | $\\ge 1$ (1 is highest priority) | Relative priority of the action item. |
| `action` | `string` | **Mandatory** | Full botanical instruction string | Detailed explanation shown in action card or modal. |
| `label` | `string` | **Mandatory** | 2–3 words (max 30 chars) | **Short button text** for Web UI (e.g. `"Pause water"`). |
| `type` | `string` | **Mandatory** | `"water"` \| `"move"` \| `"inspect"` \| `"other"` | Categorical action type for UI iconography and navigation. |

---

**Example: Success Response (`CARE_ADVICE_REQUIRED` - Push Alert & Action Card)**

```json
{
  "success": true,
  "data": {
    "plant_id": "plant-monstera-1",
    "name": "Monty",
    "species": "Monstera deliciosa",
    "dayCount": 2,
    "timestamp": "2026-09-22T08:30:00Z",
    "wateredTimestamp": "2026-09-19T14:20:00Z",
    "level": 1,
    "xpRatio": 0.0,
    "health_status": "unhealthy",
    "decision": "CARE_ADVICE_REQUIRED",
    "companion_message": "Hey there! My lower leaves are turning yellow and drooping, just like back when we overwatered in March. Could you pause watering for 5 days so my roots can get some oxygen? 🌿",
    "care_plan": {
      "id": "cp_a7b8c9d0e1f2",
      "status_label": "Overwatering stress",
      "assessment": "Severe chlorosis and drooping indicates soil moisture saturation leading to root hypoxia. Immediate water restriction is essential.",
      "actions": [
        {
          "id": "act_8e4b1a2c",
          "priority": 1,
          "action": "Hold watering for 5 days until top 2 inches of soil are dry to the touch.",
          "label": "Pause water",
          "type": "water"
        },
        {
          "id": "act_9f5c2b3d",
          "priority": 2,
          "action": "Check drainage holes at the bottom of the pot are unblocked.",
          "label": "Drain tray",
          "type": "inspect"
        },
        {
          "id": "act_0a6d3c4e",
          "priority": 3,
          "action": "Move to a bright location with indirect sunlight to assist transpiration.",
          "label": "Move plant",
          "type": "move"
        }
      ]
    }
  },
  "message": null,
  "error": null
}
```

**Example: Success Response (`NO_ACTION` - Silent Green Timeline Update)**

```json
{
  "success": true,
  "data": {
    "plant_id": "plant-pothos-2",
    "name": "Perky",
    "species": "Epipremnum aureum",
    "dayCount": 5,
    "timestamp": "2026-09-22T09:00:00Z",
    "wateredTimestamp": "2026-09-20T10:00:00Z",
    "level": 1,
    "xpRatio": 0.0,
    "health_status": "healthy",
    "decision": "NO_ACTION",
    "companion_message": "I'm feeling great today! My leaves are perky and getting plenty of light. 🌱✨",
    "care_plan": null
  },
  "message": null,
  "error": null
}
```

**Example: Error Response (`LLM_UNAVAILABLE`)**

```json
{
  "success": false,
  "data": null,
  "message": "AI reasoning service (LLM) is currently unavailable. Please verify provider connectivity.",
  "error": {
    "code": "LLM_UNAVAILABLE",
    "details": {
      "provider": "ollama",
      "model": "llama3.2:latest",
      "reason": "Connection refused at http://localhost:11434"
    }
  }
}
```

**Example: Error Response (`INTERNAL_SERVER_ERROR`)**

```json
{
  "success": false,
  "data": null,
  "message": "An unexpected internal server error occurred while processing the plant companion state.",
  "error": {
    "code": "INTERNAL_SERVER_ERROR",
    "details": {
      "request_id": "req_8f1b2c3d4e5f",
      "reason": "Database connection timeout while querying plant state"
    }
  }
}
```


## Interfaces

TODO
