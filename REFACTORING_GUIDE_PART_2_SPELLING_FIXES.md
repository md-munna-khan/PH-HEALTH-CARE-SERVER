# Part 2: Spelling Mistakes and Typo Fixes - Complete Guide

## 📋 Overview

This guide covers **ALL spelling mistakes, typos, and naming inconsistencies** found across the entire backend codebase.

---

## 🔍 Identified Issues

### Critical Spelling Errors:

1. **`apointmentFee`** → **`appointmentFee`** (Missing 'p')
2. **`quilification`** → **`qualification`** (Wrong spelling)
3. **`andCondions`** → **`andConditions`** (Missing 't')
4. **`whereConditons`** → **`whereConditions`** (Missing 'i')
5. **`inserIntoDB`** → **`insertIntoDB`** (Missing 't')
6. **`interverlTime`** → **`intervalTime`** (Wrong spelling)
7. **`sevice`** → **`service`** (Missing 'r')
8. **`madical_reports`** → **`medical_reports`** (Wrong spelling - **DATABASE MAPPING**)

---

## 🛠️ Fix-by-Fix Implementation

### Fix 1: User Validation - `quilification` → `qualification`

**File: `src/app/modules/User/user.validation.ts`**

**Find and Replace:**

```typescript
// FIND (around line 44-45):
        qualification: z.string({
            error: "quilification is required"
        }),

// REPLACE WITH:
        qualification: z.string({
            error: "Qualification is required"
        }),
```

---

### Fix 2: Doctor Validation - `apointmentFee` → `appointmentFee`

**File: `src/app/modules/Doctor/doctor.validation.ts`**

**Find and Replace 1:**

```typescript
// FIND (around line 26-28):
        apointmentFee: z.number({
            error: 'Blood group is required',
        }),

// REPLACE WITH:
        appointmentFee: z.number({
            error: 'Appointment fee is required',
        }),
```

**Find and Replace 2:**

```typescript
// FIND (around line 49):
        apointmentFee: z.number().optional(),

// REPLACE WITH:
        appointmentFee: z.number().optional(),
```

---

### Fix 3: Doctor Interface - `apointmentFee` → `appointmentFee`

**File: `src/app/modules/Doctor/doctor.interface.ts`**

**Find and Replace:**

```typescript
// FIND:
export type IDoctorUpdate = {
  name: string;
  profilePhoto: string;
  contactNumber: string;
  address: string;
  registrationNumber: string;
  experience: number;
  gender: "MALE" | "FEMALE";
  apointmentFee: number;
  qualification: string;
  currentWorkingPlace: string;
  designation: string;
  specialties: ISpecialties[];
};

// REPLACE WITH:
export type IDoctorUpdate = {
  name?: string;
  profilePhoto?: string;
  contactNumber?: string;
  address?: string;
  registrationNumber?: string;
  experience?: number;
  gender?: "MALE" | "FEMALE";
  appointmentFee?: number;
  qualification?: string;
  currentWorkingPlace?: string;
  designation?: string;
  specialties?: string[];
  removeSpecialties?: string[];
};
```

**Note:** This also fixes the interface to match Part 1 changes.

---

### Fix 4: Doctor Constants - `apointmentFee` → `appointmentFee`

**File: `src/app/modules/Doctor/doctor.constants.ts`**

**Find and Replace:**

```typescript
// FIND:
export const doctorFilterableFields: string[] = [
  "searchTerm",
  "email",
  "contactNumber",
  "gender",
  "apointmentFee",
  "specialties",
];

// REPLACE WITH:
export const doctorFilterableFields: string[] = [
  "searchTerm",
  "email",
  "contactNumber",
  "gender",
  "appointmentFee",
  "specialties",
];
```

---

### Fix 5: User Service - Multiple Fixes

**File: `src/app/modules/User/user.sevice.ts`**

**Step 5.1: Rename File**

```bash
# In terminal:
mv src/app/modules/User/user.sevice.ts src/app/modules/User/user.service.ts
```

**Step 5.2: Fix `andCondions` → `andConditions`**

**Find and Replace 1:**

```typescript
// FIND (around line 110):
const andCondions: Prisma.UserWhereInput[] = [];

// REPLACE WITH:
const andConditions: Prisma.UserWhereInput[] = [];
```

**Find and Replace 2:**

```typescript
// FIND (around line 113):
        andCondions.push({

// REPLACE WITH:
        andConditions.push({
```

**Find and Replace 3:**

```typescript
// FIND (around line 124):
        andCondions.push({

// REPLACE WITH:
        andConditions.push({
```

**Step 5.3: Fix `whereConditons` → `whereConditions`**

**Find and Replace 1:**

```typescript
// FIND (around line 133):
const whereConditons: Prisma.UserWhereInput =
  andCondions.length > 0 ? { AND: andCondions } : {};

// REPLACE WITH:
const whereConditions: Prisma.UserWhereInput =
  andConditions.length > 0 ? { AND: andConditions } : {};
```

**Find and Replace 2:**

```typescript
// FIND (around line 136):
        where: whereConditons,

// REPLACE WITH:
        where: whereConditions,
```

**Find and Replace 3:**

```typescript
// FIND (around line 159):
where: whereConditons;

// REPLACE WITH:
where: whereConditions;
```

---

### Fix 6: User Controller - Import Path Fix

**File: `src/app/modules/User/user.controller.ts`**

**Find and Replace:**

```typescript
// FIND (line 2):
import { userService } from "./user.sevice";

// REPLACE WITH:
import { userService } from "./user.service";
```

---

### Fix 7: Admin Service - Multiple Fixes

**File: `src/app/modules/Admin/admin.service.ts`**

**Find and Replace 1:**

```typescript
// FIND (around line 16):
const andCondions: Prisma.AdminWhereInput[] = [];

// REPLACE WITH:
const andConditions: Prisma.AdminWhereInput[] = [];
```

**Find and Replace 2:**

```typescript
// FIND (around line 19):
        andCondions.push({

// REPLACE WITH:
        andConditions.push({
```

**Find and Replace 3:**

```typescript
// FIND (around line 30):
        andCondions.push({

// REPLACE WITH:
        andConditions.push({
```

**Find and Replace 4:**

```typescript
// FIND (around line 38):
    andCondions.push({

// REPLACE WITH:
    andConditions.push({
```

**Find and Replace 5:**

```typescript
// FIND (around line 43):
const whereConditons: Prisma.AdminWhereInput = { AND: andCondions };

// REPLACE WITH:
const whereConditions: Prisma.AdminWhereInput = { AND: andConditions };
```

**Find and Replace 6:**

```typescript
// FIND (around line 46):
        where: whereConditons,

// REPLACE WITH:
        where: whereConditions,
```

**Find and Replace 7:**

```typescript
// FIND (around line 57):
where: whereConditons;

// REPLACE WITH:
where: whereConditions;
```

---

### Fix 8: Schedule Service - Multiple Fixes

**File: `src/app/modules/Schedule/schedule.sevice.ts`**

**Step 8.1: Rename File**

```bash
# In terminal:
mv src/app/modules/Schedule/schedule.sevice.ts src/app/modules/Schedule/schedule.service.ts
```

**Step 8.2: Fix `inserIntoDB` → `insertIntoDB`**

**Find and Replace 1:**

```typescript
// FIND (around line 14):
const inserIntoDB = async (payload: ISchedule): Promise<Schedule[]> => {

// REPLACE WITH:
const insertIntoDB = async (payload: ISchedule): Promise<Schedule[]> => {
```

**Find and Replace 2:**

```typescript
// FIND (around line 192):
export const ScheduleService = {
  inserIntoDB,
  getAllFromDB,
  getByIdFromDB,
  deleteFromDB,
};

// REPLACE WITH:
export const ScheduleService = {
  insertIntoDB,
  getAllFromDB,
  getByIdFromDB,
  deleteFromDB,
};
```

**Step 8.3: Fix `interverlTime` → `intervalTime`**

**Find and Replace 1:**

```typescript
// FIND (around line 17):
const interverlTime = 30;

// REPLACE WITH:
const intervalTime = 30;
```

**Find and Replace 2:**

```typescript
// FIND (around line 49):
//     endDateTime: addMinutes(startDateTime, interverlTime)

// REPLACE WITH:
//     endDateTime: addMinutes(startDateTime, intervalTime)
```

**Find and Replace 3:**

```typescript
// FIND (around line 53):
const e = await convertDateTime(addMinutes(startDateTime, interverlTime));

// REPLACE WITH:
const e = await convertDateTime(addMinutes(startDateTime, intervalTime));
```

**Find and Replace 4:**

```typescript
// FIND (around line 74):
startDateTime.setMinutes(startDateTime.getMinutes() + interverlTime);

// REPLACE WITH:
startDateTime.setMinutes(startDateTime.getMinutes() + intervalTime);
```

---

### Fix 9: Schedule Controller - Import Path Fix

**File: `src/app/modules/Schedule/schedule.controller.ts`**

**Find and Replace 1:**

```typescript
// FIND (line 5):
import { ScheduleService } from "./schedule.sevice";

// REPLACE WITH:
import { ScheduleService } from "./schedule.service";
```

**Find and Replace 2:**

```typescript
// FIND (around line 9):
const inserIntoDB = catchAsync(async (req: Request, res: Response) => {
    const result = await ScheduleService.inserIntoDB(req.body);

// REPLACE WITH:
const insertIntoDB = catchAsync(async (req: Request, res: Response) => {
    const result = await ScheduleService.insertIntoDB(req.body);
```

**Find and Replace 3:**

```typescript
// FIND (around line 59):
export const ScheduleController = {
  inserIntoDB,
  getAllFromDB,
  getByIdFromDB,
  deleteFromDB,
};

// REPLACE WITH:
export const ScheduleController = {
  insertIntoDB,
  getAllFromDB,
  getByIdFromDB,
  deleteFromDB,
};
```

---

### Fix 10: Schedule Routes - Method Name Fix

**File: `src/app/modules/Schedule/schedule.routes.ts`**

**Find and Replace:**

```typescript
// FIND (around line 28):
ScheduleController.inserIntoDB;

// REPLACE WITH:
ScheduleController.insertIntoDB;
```

---

### Fix 11: Specialties Service - `inserIntoDB` → `insertIntoDB`

**File: `src/app/modules/Specialties/specialties.service.ts`**

**Find and Replace 1:**

```typescript
// FIND (around line 6):
const inserIntoDB = async (req: Request) => {

// REPLACE WITH:
const insertIntoDB = async (req: Request) => {
```

**Find and Replace 2:**

```typescript
// FIND (around line 36):
export const SpecialtiesService = {
  inserIntoDB,
  getAllFromDB,
  deleteFromDB,
};

// REPLACE WITH:
export const SpecialtiesService = {
  insertIntoDB,
  getAllFromDB,
  deleteFromDB,
};
```

---

### Fix 12: Specialties Controller - Method Name Fix

**File: `src/app/modules/Specialties/specialties.controller.ts`**

**Find and Replace 1:**

```typescript
// FIND (around line 7):
const inserIntoDB = catchAsync(async (req: Request, res: Response) => {
    const result = await SpecialtiesService.inserIntoDB(req);

// REPLACE WITH:
const insertIntoDB = catchAsync(async (req: Request, res: Response) => {
    const result = await SpecialtiesService.insertIntoDB(req);
```

**Find and Replace 2:**

```typescript
// FIND (around line 40):
export const SpecialtiesController = {
  inserIntoDB,
  getAllFromDB,
  deleteFromDB,
};

// REPLACE WITH:
export const SpecialtiesController = {
  insertIntoDB,
  getAllFromDB,
  deleteFromDB,
};
```

---

### Fix 13: Specialties Routes - Method Name Fix

**File: `src/app/modules/Specialties/specialties.routes.ts`**

**Find and Replace:**

```typescript
// FIND (around line 29):
return SpecialtiesController.inserIntoDB(req, res, next);

// REPLACE WITH:
return SpecialtiesController.insertIntoDB(req, res, next);
```

---

### Fix 14: Database Schema - `madical_reports` → `medical_reports`

**⚠️ CRITICAL: This requires a database migration!**

**File: `prisma/schema/patientData.prisma`**

**Find and Replace:**

```prisma
// FIND (around line 36):
  @@map("madical_reports")

// REPLACE WITH:
  @@map("medical_reports")
```

**Migration Steps:**

```bash
# 1. Create migration
npx prisma migrate dev --name fix_medical_reports_spelling --create-only

# 2. Edit the generated migration file to:
#    - Rename the table from "madical_reports" to "medical_reports"
#    - Update all foreign key constraints

# 3. Apply migration
npx prisma migrate dev

# 4. Verify
npx prisma studio
```

**Generated Migration SQL (manual edit):**

```sql
-- DropForeignKey
ALTER TABLE "madical_reports" DROP CONSTRAINT "madical_reports_patientId_fkey";

-- Rename table
ALTER TABLE "madical_reports" RENAME TO "medical_reports";

-- AddForeignKey
ALTER TABLE "medical_reports" ADD CONSTRAINT "medical_reports_patientId_fkey"
  FOREIGN KEY ("patientId") REFERENCES "patients"("id") ON DELETE RESTRICT ON UPDATE CASCADE;
```

---

## 📊 Summary of All Spelling Fixes

| #   | Error                | Correct               | Files Affected | Impact   |
| --- | -------------------- | --------------------- | -------------- | -------- |
| 1   | `apointmentFee`      | `appointmentFee`      | 3 files        | Medium   |
| 2   | `quilification`      | `qualification`       | 1 file         | Low      |
| 3   | `andCondions`        | `andConditions`       | 2 files        | Low      |
| 4   | `whereConditons`     | `whereConditions`     | 2 files        | Low      |
| 5   | `inserIntoDB`        | `insertIntoDB`        | 6 files        | Medium   |
| 6   | `interverlTime`      | `intervalTime`        | 1 file         | Low      |
| 7   | `user.sevice.ts`     | `user.service.ts`     | 1 file         | Low      |
| 8   | `schedule.sevice.ts` | `schedule.service.ts` | 1 file         | Low      |
| 9   | `madical_reports`    | `medical_reports`     | 1 schema + DB  | **HIGH** |

---

## 🗂️ Files to Modify (Complete List)

### TypeScript Files (18 files):

1. `src/app/modules/User/user.validation.ts`
2. `src/app/modules/User/user.sevice.ts` → `user.service.ts` (rename)
3. `src/app/modules/User/user.controller.ts`
4. `src/app/modules/Doctor/doctor.validation.ts`
5. `src/app/modules/Doctor/doctor.interface.ts`
6. `src/app/modules/Doctor/doctor.constants.ts`
7. `src/app/modules/Admin/admin.service.ts`
8. `src/app/modules/Schedule/schedule.sevice.ts` → `schedule.service.ts` (rename)
9. `src/app/modules/Schedule/schedule.controller.ts`
10. `src/app/modules/Schedule/schedule.routes.ts`
11. `src/app/modules/Specialties/specialties.service.ts`
12. `src/app/modules/Specialties/specialties.controller.ts`
13. `src/app/modules/Specialties/specialties.routes.ts`

### Schema Files (1 file):

14. `prisma/schema/patientData.prisma`

---

## ✅ Testing After Fixes

### 1. Type Checking

```bash
npx tsc --noEmit
```

### 2. Build Test

```bash
npm run build
```

### 3. Start Server

```bash
npm run dev
```

### 4. API Tests

- Test all doctor-related endpoints
- Test all schedule-related endpoints
- Test all specialties-related endpoints
- Test medical reports retrieval

### 5. Database Tests

```bash
# Verify table name
npx prisma studio
# Should show "medical_reports" not "madical_reports"
```

---

## 🚀 Execution Order

**Execute in this exact order to avoid errors:**

1. **Phase 1: File Renames**

   - Rename `user.sevice.ts` → `user.service.ts`
   - Rename `schedule.sevice.ts` → `schedule.service.ts`

2. **Phase 2: Import Fixes**

   - Fix imports in `user.controller.ts`
   - Fix imports in `schedule.controller.ts`

3. **Phase 3: Variable Name Fixes**

   - Fix all `andCondions` → `andConditions`
   - Fix all `whereConditons` → `whereConditions`
   - Fix all `apointmentFee` → `appointmentFee`
   - Fix all `quilification` → `qualification`
   - Fix all `inserIntoDB` → `insertIntoDB`
   - Fix all `interverlTime` → `intervalTime`

4. **Phase 4: Database Schema Fix**

   - Update Prisma schema
   - Create and run migration
   - Regenerate Prisma Client: `npx prisma generate`

5. **Phase 5: Testing**
   - Run TypeScript checks
   - Build project
   - Start server
   - Test all affected endpoints

---

## 📝 Migration Checklist

- [ ] Backup database before migration
- [ ] Update Prisma schema file
- [ ] Create migration with `--create-only` flag
- [ ] Manually edit migration SQL
- [ ] Review migration SQL carefully
- [ ] Test migration on dev database first
- [ ] Run migration: `npx prisma migrate dev`
- [ ] Regenerate client: `npx prisma generate`
- [ ] Verify table name in Prisma Studio
- [ ] Test medical report APIs
- [ ] Update all imports if needed
- [ ] Commit changes to version control

---

**Next:** See `REFACTORING_GUIDE_PART_3_METADATA_SANITIZATION.md`
