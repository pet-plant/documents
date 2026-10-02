# APIs & Interfaces

| Component | Description |
|---|---|
| [core](#core) | Authentication and device pairing |
| [registry](#registry) | Plant registration, and which owner, species and device each plant has |
| [knowledge](#knowledge) | Probe generation from research documents, and its review |
| [orchestrator](#orchestrator) | Runs the scheduled pipeline |
| [capture](#capture) | TODO |
| [assessment](#assessment) | TODO |
| [advice](#advice) | TODO |
| [companion](#companion) | TODO |


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

TODO

## API

### `GET /companion/devices/me/state`

The character's current state, open issues and idle state for the plant this device shows (device)

**Response**

```json
{
  "name": "string",
  "timeLabel": "string",
  "level": "integer",
  "xpRatio": "number (0-1)",
  "dayCount": "integer",
  "lastWateredLabel": "string",
  "issues": [
    {
      "id": "string (unique, unchanged until the issue is closed)",
      "status": "thirsty | needs_light | other_problem",
      "comment": {
        "text": "string",
        "dateLabel": "string"
      },
      "statusCard": {
        "statusLabel": "string",
        "detail": "string",
        "action": "string"
      },
      "action": {
        "type": "water | move | inspect",
        "label": "string"
      }
    }
  ],
  "idle": {
    "status": "happy",
    "comment": {
      "text": "string",
      "dateLabel": "string"
    },
    "statusCard": {
      "statusLabel": "string",
      "detail": "string",
      "action": "string"
    },
    "action": {
      "type": "acknowledge",
      "label": "string"
    }
  }
}
```


## Interfaces

TODO
