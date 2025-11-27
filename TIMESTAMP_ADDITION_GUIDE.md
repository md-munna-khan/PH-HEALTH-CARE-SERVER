# Adding Timestamps to Prisma Models - Complete Guide

## 📋 Overview

This guide adds `createdAt` and `updatedAt` timestamps to all models that are currently missing them.

---

## 🔍 Current State Analysis

### Models WITH Timestamps ✅ (Already Complete)

These models already have proper timestamps and need NO changes:

1. **User** - Has `createdAt` and `updatedAt` ✅
2. **Admin** - Has `createdAt` and `updatedAt` ✅
3. **Doctor** - Has `createdAt` and `updatedAt` ✅
4. **Patient** - Has `createdAt` and `updatedAt` ✅
5. **Appointment** - Has `createdAt` and `updatedAt` ✅
6. **Payment** - Has `createdAt` and `updatedAt` ✅
7. **Prescription** - Has `createdAt` and `updatedAt` ✅
8. **Review** - Has `createdAt` and `updatedAt` ✅
9. **Schedule** - Has `createdAt` and `updatedAt` ✅
10. **PatientHealthData** - Has `createdAt` and `updatedAt` ✅
11. **MedicalReport** - Has `createdAt` and `updatedAt` ✅

### Models WITHOUT Timestamps ❌ (Need Updates)

These **2 junction/relation models** are missing timestamps:

1. **Specialties** - Missing both timestamps ❌
2. **DoctorSpecialties** - Missing both timestamps ❌
3. **DoctorSchedules** - Missing both timestamps ❌

---

## 🎯 Why Add Timestamps to Junction Tables?

### Benefits:

1. **Audit Trail** - Know when relationships were created/modified
2. **Data Analytics** - Track when doctors added specialties, when schedules were created
3. **Debugging** - Easier to troubleshoot issues with temporal context
4. **Best Practice** - Industry standard for database design
5. **Historical Data** - Understand system evolution over time

### Use Cases:

- "When did this doctor add the Cardiology specialty?"
- "When was this schedule slot created?"
- "Show me all doctor-schedule assignments from last month"
- "Track specialty additions over time for analytics"

---

## 🛠️ Implementation Guide

### Step 1: Update Specialties Model

**File:** `prisma/schema/specialty.prisma`

**CURRENT CODE:**

```prisma
model Specialties {
  id                String              @id @default(uuid())
  title             String
  icon              String
  doctorSpecialties DoctorSpecialties[]

  @@map("specialties")
}
```

**REPLACE WITH:**

```prisma
model Specialties {
  id                String              @id @default(uuid())
  title             String
  icon              String
  createdAt         DateTime            @default(now())
  updatedAt         DateTime            @updatedAt
  doctorSpecialties DoctorSpecialties[]

  @@map("specialties")
}
```

**Changes Made:**

- ✅ Added `createdAt DateTime @default(now())` - Auto-set on creation
- ✅ Added `updatedAt DateTime @updatedAt` - Auto-update on modification

---

### Step 2: Update DoctorSpecialties Model

**File:** `prisma/schema/specialty.prisma`

**CURRENT CODE:**

```prisma
model DoctorSpecialties {
  specialitiesId String
  specialities   Specialties @relation(fields: [specialitiesId], references: [id])

  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  @@id([specialitiesId, doctorId])
  @@map("doctor_specialties")
}
```

**REPLACE WITH:**

```prisma
model DoctorSpecialties {
  specialitiesId String
  specialities   Specialties @relation(fields: [specialitiesId], references: [id])

  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@id([specialitiesId, doctorId])
  @@map("doctor_specialties")
}
```

**Changes Made:**

- ✅ Added `createdAt DateTime @default(now())` - Track when doctor-specialty relationship was created
- ✅ Added `updatedAt DateTime @updatedAt` - Track if relationship metadata changes (future-proof)

---

### Step 3: Update DoctorSchedules Model

**File:** `prisma/schema/schedule.prisma`

**CURRENT CODE:**

```prisma
model DoctorSchedules {
  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  scheduleId String
  schedule   Schedule @relation(fields: [scheduleId], references: [id])

  isBooked Boolean @default(false)

  appointmentId String?      @unique
  appointment   Appointment? @relation(fields: [appointmentId], references: [id])

  @@id([doctorId, scheduleId])
  @@map("doctor_schedules")
}
```

**REPLACE WITH:**

```prisma
model DoctorSchedules {
  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  scheduleId String
  schedule   Schedule @relation(fields: [scheduleId], references: [id])

  isBooked Boolean @default(false)

  appointmentId String?      @unique
  appointment   Appointment? @relation(fields: [appointmentId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@id([doctorId, scheduleId])
  @@map("doctor_schedules")
}
```

**Changes Made:**

- ✅ Added `createdAt DateTime @default(now())` - Track when schedule was assigned to doctor
- ✅ Added `updatedAt DateTime @updatedAt` - Track when `isBooked` changes or appointment is assigned

---

## 📝 Complete Updated Files

### File 1: `prisma/schema/specialty.prisma`

```prisma
model Specialties {
  id                String              @id @default(uuid())
  title             String
  icon              String
  createdAt         DateTime            @default(now())
  updatedAt         DateTime            @updatedAt
  doctorSpecialties DoctorSpecialties[]

  @@map("specialties")
}

model DoctorSpecialties {
  specialitiesId String
  specialities   Specialties @relation(fields: [specialitiesId], references: [id])

  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@id([specialitiesId, doctorId])
  @@map("doctor_specialties")
}
```

---

### File 2: `prisma/schema/schedule.prisma`

```prisma
model Schedule {
  id              String            @id @default(uuid())
  startDateTime   DateTime
  endDateTime     DateTime
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  doctorSchedules DoctorSchedules[]
  appointment     Appointment?

  @@map("schedules")
}

model DoctorSchedules {
  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])

  scheduleId String
  schedule   Schedule @relation(fields: [scheduleId], references: [id])

  isBooked Boolean @default(false)

  appointmentId String?      @unique
  appointment   Appointment? @relation(fields: [appointmentId], references: [id])

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@id([doctorId, scheduleId])
  @@map("doctor_schedules")
}
```

---

## 🗄️ Database Migration

### Step 1: Generate Migration

```bash
# Navigate to project root
cd c:/Projects/next-level/batches/batch-5/PH-HealthCare/My-PH-HealthCare/Backend-My-PH-HealthCare

# Generate migration
npx prisma migrate dev --name add_timestamps_to_junction_tables
```

### Step 2: Migration SQL Preview

The migration will generate SQL similar to this:

```sql
-- AlterTable: Add timestamps to specialties
ALTER TABLE "specialties"
  ADD COLUMN "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  ADD COLUMN "updatedAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP;

-- AlterTable: Add timestamps to doctor_specialties
ALTER TABLE "doctor_specialties"
  ADD COLUMN "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  ADD COLUMN "updatedAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP;

-- AlterTable: Add timestamps to doctor_schedules
ALTER TABLE "doctor_schedules"
  ADD COLUMN "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  ADD COLUMN "updatedAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP;
```

### Step 3: Apply Migration

```bash
# The migration is automatically applied by the command above
# If you created with --create-only, run:
npx prisma migrate dev
```

### Step 4: Regenerate Prisma Client

```bash
# Regenerate Prisma Client with new fields
npx prisma generate
```

### Step 5: Verify in Prisma Studio

```bash
# Open Prisma Studio to verify
npx prisma studio
```

Check that:

- ✅ `specialties` table has `createdAt` and `updatedAt` columns
- ✅ `doctor_specialties` table has `createdAt` and `updatedAt` columns
- ✅ `doctor_schedules` table has `createdAt` and `updatedAt` columns
- ✅ Existing records have timestamps set to current time
- ✅ New records will auto-populate timestamps

---

## 🔧 Handling Existing Data

### Default Values for Existing Records

When you run the migration, PostgreSQL will:

1. **For `createdAt`**: Set to `CURRENT_TIMESTAMP` (migration execution time)
2. **For `updatedAt`**: Set to `CURRENT_TIMESTAMP` (migration execution time)

**Note:** Existing records won't have their true creation date, but all **future** records will track accurately.

### Optional: Set More Accurate Timestamps

If you want existing records to have more accurate timestamps, you can run post-migration SQL:

```sql
-- For DoctorSpecialties: Use doctor creation date as proxy
UPDATE doctor_specialties ds
SET
  "createdAt" = d."createdAt",
  "updatedAt" = d."createdAt"
FROM doctors d
WHERE ds."doctorId" = d.id;

-- For DoctorSchedules: Use schedule creation date as proxy
UPDATE doctor_schedules ds
SET
  "createdAt" = s."createdAt",
  "updatedAt" = s."createdAt"
FROM schedules s
WHERE ds."scheduleId" = s.id;

-- For Specialties: Keep default (no better proxy available)
-- They will show migration timestamp
```

---

## 📊 Impact Analysis

### Backend Code Changes Needed: **NONE** ✅

Good news! Since we're only **adding** fields (not modifying/removing), existing code will continue to work:

- ✅ Existing queries won't break
- ✅ Existing inserts will auto-populate timestamps
- ✅ Existing updates will auto-update `updatedAt`
- ✅ No service/controller changes needed
- ✅ No validation schema changes needed

### Optional Enhancements

You MAY want to enhance responses to include timestamps:

#### Example 1: Specialties Service

**File:** `src/app/modules/Specialties/specialties.service.ts`

Current response already includes all fields (including timestamps after migration):

```typescript
const getAllFromDB = async (): Promise<Specialties[]> => {
  return await prisma.specialties.findMany();
  // Will now automatically include createdAt and updatedAt
};
```

No changes needed! ✅

#### Example 2: Doctor Service (Doctor with Specialties)

**File:** `src/app/modules/Doctor/doctor.service.ts`

When returning doctor with specialties:

```typescript
const result = await prisma.doctor.findUnique({
  where: { id },
  include: {
    doctorSpecialties: {
      include: {
        specialities: true,
      },
    },
  },
});
```

Response will now include:

```json
{
  "doctorSpecialties": [
    {
      "specialitiesId": "uuid",
      "doctorId": "uuid",
      "createdAt": "2025-11-16T10:00:00.000Z", // NEW
      "updatedAt": "2025-11-16T10:00:00.000Z", // NEW
      "specialities": {
        "id": "uuid",
        "title": "Cardiology",
        "icon": "icon-url",
        "createdAt": "2025-11-01T10:00:00.000Z", // NEW
        "updatedAt": "2025-11-01T10:00:00.000Z" // NEW
      }
    }
  ]
}
```

No code changes needed - it's automatic! ✅

---

## 🧪 Testing Guide

### Test 1: Create New Specialty

```bash
POST http://localhost:5000/api/v1/specialties
Content-Type: multipart/form-data

{
  "title": "Neurology",
  "icon": "brain-icon-url"
}

# Expected Response (includes new timestamps):
{
  "success": true,
  "message": "Specialty created successfully",
  "data": {
    "id": "uuid",
    "title": "Neurology",
    "icon": "brain-icon-url",
    "createdAt": "2025-11-16T10:00:00.000Z",  // Auto-populated
    "updatedAt": "2025-11-16T10:00:00.000Z"   // Auto-populated
  }
}
```

### Test 2: Update Specialty

```bash
PATCH http://localhost:5000/api/v1/specialties/:id
Content-Type: application/json

{
  "title": "Neurology - Updated"
}

# Expected Response:
{
  "success": true,
  "message": "Specialty updated successfully",
  "data": {
    "id": "uuid",
    "title": "Neurology - Updated",
    "icon": "brain-icon-url",
    "createdAt": "2025-11-16T10:00:00.000Z",  // Unchanged
    "updatedAt": "2025-11-16T10:05:00.000Z"   // Updated! ✅
  }
}
```

### Test 3: Create Doctor with Specialties

```bash
POST http://localhost:5000/api/v1/user/create-doctor

# DoctorSpecialties records created with timestamps
# createdAt and updatedAt will be auto-populated
```

### Test 4: Doctor Schedule Creation

```bash
POST http://localhost:5000/api/v1/doctor-schedule

# DoctorSchedules records created with timestamps
# createdAt and updatedAt will be auto-populated
```

### Test 5: Verify Existing Records

```bash
GET http://localhost:5000/api/v1/specialties

# All existing specialties now have timestamps
# Set to migration execution time
```

---

## ✅ Testing Checklist

After migration:

- [ ] Run `npx prisma generate` successfully
- [ ] Server starts without errors
- [ ] GET /specialties returns records with timestamps
- [ ] POST /specialties creates record with timestamps
- [ ] PATCH /specialties updates `updatedAt` only
- [ ] Doctor creation with specialties populates DoctorSpecialties timestamps
- [ ] Schedule assignment populates DoctorSchedules timestamps
- [ ] Existing records have timestamps (migration time)
- [ ] New records have accurate timestamps
- [ ] `updatedAt` changes only on updates, not on reads

---

## 📋 Summary of Changes

### Models Updated: 3

| Model             | File             | Fields Added             | Purpose                                       |
| ----------------- | ---------------- | ------------------------ | --------------------------------------------- |
| Specialties       | specialty.prisma | `createdAt`, `updatedAt` | Track specialty creation/modification         |
| DoctorSpecialties | specialty.prisma | `createdAt`, `updatedAt` | Track when doctor added specialty             |
| DoctorSchedules   | schedule.prisma  | `createdAt`, `updatedAt` | Track schedule assignment and booking changes |

### Database Tables Updated: 3

| Table                | Columns Added            |
| -------------------- | ------------------------ |
| `specialties`        | `createdAt`, `updatedAt` |
| `doctor_specialties` | `createdAt`, `updatedAt` |
| `doctor_schedules`   | `createdAt`, `updatedAt` |

### Code Changes Required: **0** ✅

- No service changes needed
- No controller changes needed
- No validation changes needed
- Timestamps auto-populate on insert
- Timestamps auto-update on update
- All existing APIs work as-is

---

## 🚀 Implementation Steps

### Step-by-Step Execution

1. **Update Schema Files** (5 minutes)

   - Update `prisma/schema/specialty.prisma`
   - Update `prisma/schema/schedule.prisma`

2. **Generate Migration** (2 minutes)

   ```bash
   npx prisma migrate dev --name add_timestamps_to_junction_tables
   ```

3. **Verify Migration** (2 minutes)

   ```bash
   npx prisma studio
   # Check all 3 tables have new columns
   ```

4. **Regenerate Client** (1 minute)

   ```bash
   npx prisma generate
   ```

5. **Test Server** (5 minutes)

   ```bash
   npm run dev
   # Test all affected endpoints
   ```

6. **Optional: Adjust Existing Data** (10 minutes)

   ```sql
   -- Run post-migration SQL if needed
   ```

7. **Commit Changes** (2 minutes)
   ```bash
   git add .
   git commit -m "feat: add timestamps to junction tables (Specialties, DoctorSpecialties, DoctorSchedules)"
   git push
   ```

**Total Time: ~30 minutes**

---

## ⚠️ Important Notes

### What `@updatedAt` Does

The `@updatedAt` directive tells Prisma to:

1. ✅ Auto-set the field to current timestamp on **creation**
2. ✅ Auto-update the field to current timestamp on **updates**
3. ✅ **NOT** update the field on reads/queries
4. ✅ Handle this at the Prisma Client level (not database triggers)

### When `updatedAt` Changes

```typescript
// ✅ updatedAt WILL change:
await prisma.specialties.update({
  where: { id },
  data: { title: "New Title" },
});

await prisma.doctorSchedules.update({
  where: { doctorId_scheduleId: { doctorId, scheduleId } },
  data: { isBooked: true },
});

// ❌ updatedAt will NOT change:
await prisma.specialties.findMany(); // Just reading
await prisma.specialties.findUnique({ where: { id } }); // Just reading
```

### Migration Safety

- ✅ **Safe for production** - Only adds columns, doesn't modify/remove
- ✅ **Non-breaking** - All existing queries work
- ✅ **Backward compatible** - Old code continues to work
- ✅ **Forward compatible** - New code can use timestamps
- ✅ **No downtime** required - Can apply during operation

---

## 🎯 Success Criteria

Your implementation is successful when:

- [ ] All 3 schema files updated
- [ ] Migration created and applied successfully
- [ ] Prisma Client regenerated
- [ ] Server starts without errors
- [ ] All existing APIs work as before
- [ ] New records have accurate timestamps
- [ ] Updates properly modify `updatedAt`
- [ ] `createdAt` never changes after creation
- [ ] Prisma Studio shows timestamps in all 3 tables

---

## 📚 Related Documentation

- Prisma `@default(now())`: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#default
- Prisma `@updatedAt`: https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#updatedat

---

## 🔄 Rollback Plan

If you need to rollback:

```bash
# Undo last migration
npx prisma migrate resolve --rolled-back <migration-name>

# Or revert to previous migration
npx prisma migrate resolve --applied <previous-migration-name>

# Regenerate client
npx prisma generate
```

---

**End of Timestamp Addition Guide**

**Implementation Ready! ✅**
