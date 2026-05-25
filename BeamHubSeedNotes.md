# Beam Hub Seed Notes (Manual)

These are non-executable notes you can run manually in Firestore Console or script later.

## 1) Create a Unit
Collection: `units`
Document ID: `unit_demo_orchestra`

Fields:
- `name`: "Demo Orchestra"
- `shortCode`: "DO"
- `unitType`: "orchestra"
- `timezoneId`: "America/New_York"
- `isActive`: true
- `createdAt`: server timestamp
- `updatedAt`: server timestamp

## 2) Create an Event
Collection: `events`
Document ID: `event_demo_rehearsal_2026_02_20`

Fields:
- `unitId`: "unit_demo_orchestra"
- `title`: "Sectional Rehearsal"
- `startAt`: Timestamp("2026-02-20T19:00:00-05:00")
- `endAt`: Timestamp("2026-02-20T21:30:00-05:00")
- `location`: "Main Hall"
- `eventType`: "rehearsal"
- `weekOf`: Timestamp("2026-02-16T00:00:00-05:00")
- `status`: "scheduled"
- `visibility`: "unit"
- `mealPlanId`: null
- `createdAt`: server timestamp
- `updatedAt`: server timestamp

## 3) (Optional) Add Membership for visibility in app
Collection: `memberships`
Document ID: auto or `membership_<userId>_unit_demo_orchestra`

Fields:
- `userId`: "<auth uid>"
- `unitId`: "unit_demo_orchestra"
- `roleIds`: []
- `status`: "active"
- `createdAt`: server timestamp
- `updatedAt`: server timestamp
