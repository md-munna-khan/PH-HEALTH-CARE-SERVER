# Part 1: Doctor Specialties Transaction Fix - Complete Guide

## 📋 Overview

This guide addresses the **Doctor Specialties Management** issue where:

- **CREATE**: Doctors cannot add specialties during creation
- **UPDATE**: Doctors need ability to add new specialties and remove existing ones

## 🔍 Problem Analysis

### Current Issues:

1. **User Service (`user.sevice.ts`)** - `createDoctor` function doesn't handle `specialties` array
2. **Doctor Update** - Current implementation uses `isDeleted` flag which is confusing
3. **No proper validation** for specialties array in create/update operations
4. **Missing transaction** in doctor creation with specialties

### Database Schema (Already Correct):

```prisma
model Specialties {
  id                String              @id @default(uuid())
  title             String
  icon              String
  doctorSpecialties DoctorSpecialties[]
  @@map("specialties")
}

model DoctorSpecialties {
  specialitiesId String
  specialities   Specialties @relation(fields: [specialitiesId], references: [id])
  doctorId String
  doctor   Doctor @relation(fields: [doctorId], references: [id])
  @@id([specialitiesId, doctorId])
  @@map("doctor_specialties")
}
```

---

## 🛠️ Solution Implementation

### Step 1: Update Validation Schema

**File: `src/app/modules/User/user.validation.ts`**

```typescript
import { Gender, UserRole, UserStatus } from "@prisma/client";
import { z } from "zod";

const createAdmin = z.object({
  password: z.string({
    error: "Password is required",
  }),
  admin: z.object({
    name: z.string({
      error: "Name is required!",
    }),
    email: z.string({
      error: "Email is required!",
    }),
    contactNumber: z.string({
      error: "Contact Number is required!",
    }),
  }),
});

const createDoctor = z.object({
  password: z.string({
    error: "Password is required",
  }),
  doctor: z.object({
    name: z.string({
      error: "Name is required!",
    }),
    email: z.string({
      error: "Email is required!",
    }),
    contactNumber: z.string({
      error: "Contact Number is required!",
    }),
    address: z.string().optional(),
    registrationNumber: z.string({
      error: "Reg number is required",
    }),
    experience: z.number().optional(),
    gender: z.enum([Gender.MALE, Gender.FEMALE]),
    appointmentFee: z.number({
      error: "Appointment fee is required",
    }),
    qualification: z.string({
      error: "Qualification is required",
    }),
    currentWorkingPlace: z.string({
      error: "Current working place is required!",
    }),
    designation: z.string({
      error: "Designation is required!",
    }),
    // NEW: Add specialties array for doctor creation
    specialties: z
      .array(
        z.string().uuid({
          message: "Each specialty must be a valid UUID",
        })
      )
      .min(1, {
        message: "At least one specialty is required",
      })
      .optional(),
  }),
});

const createPatient = z.object({
  password: z.string(),
  patient: z.object({
    email: z
      .string({
        error: "Email is required!",
      })
      .email(),
    name: z.string({
      error: "Name is required!",
    }),
    contactNumber: z.string({
      error: "Contact number is required!",
    }),
    address: z.string({
      error: "Address is required",
    }),
  }),
});

const updateStatus = z.object({
  body: z.object({
    status: z.enum([UserStatus.ACTIVE, UserStatus.BLOCKED, UserStatus.DELETED]),
  }),
});

export const userValidation = {
  createAdmin,
  createDoctor,
  createPatient,
  updateStatus,
};
```

**Key Changes:**

- ✅ Fixed: `quilification` → `qualification`
- ✅ Fixed: `appointment fee` capitalization
- ✅ Added: `specialties` array validation with UUID check and minimum 1 requirement (optional)

---

### Step 2: Update User Service for Doctor Creation

**File: `src/app/modules/User/user.sevice.ts`**

Replace the entire `createDoctor` function:

```typescript
const createDoctor = async (req: Request): Promise<Doctor> => {
  const file = req.file;

  if (file) {
    const uploadToCloudinary = await fileUploader.uploadToCloudinary(file);
    req.body.doctor.profilePhoto = uploadToCloudinary?.secure_url;
  }

  const hashedPassword: string = await bcrypt.hash(
    req.body.password,
    Number(config.salt_round)
  );

  const userData = {
    email: req.body.doctor.email,
    password: hashedPassword,
    role: UserRole.DOCTOR,
  };

  // Extract specialties from doctor data
  const { specialties, ...doctorData } = req.body.doctor;

  const result = await prisma.$transaction(async (transactionClient) => {
    // Step 1: Create user
    await transactionClient.user.create({
      data: userData,
    });

    // Step 2: Create doctor
    const createdDoctorData = await transactionClient.doctor.create({
      data: doctorData,
    });

    // Step 3: Create doctor specialties if provided
    if (specialties && Array.isArray(specialties) && specialties.length > 0) {
      // Verify all specialties exist
      const existingSpecialties = await transactionClient.specialties.findMany({
        where: {
          id: {
            in: specialties,
          },
        },
        select: {
          id: true,
        },
      });

      const existingSpecialtyIds = existingSpecialties.map((s) => s.id);
      const invalidSpecialties = specialties.filter(
        (id) => !existingSpecialtyIds.includes(id)
      );

      if (invalidSpecialties.length > 0) {
        throw new Error(
          `Invalid specialty IDs: ${invalidSpecialties.join(", ")}`
        );
      }

      // Create doctor specialties relations
      const doctorSpecialtiesData = specialties.map((specialtyId) => ({
        doctorId: createdDoctorData.id,
        specialitiesId: specialtyId,
      }));

      await transactionClient.doctorSpecialties.createMany({
        data: doctorSpecialtiesData,
      });
    }

    // Step 4: Return doctor with specialties
    const doctorWithSpecialties = await transactionClient.doctor.findUnique({
      where: {
        id: createdDoctorData.id,
      },
      include: {
        doctorSpecialties: {
          include: {
            specialities: true,
          },
        },
      },
    });

    return doctorWithSpecialties!;
  });

  return result;
};
```

**Key Changes:**

- ✅ Added transaction-based creation
- ✅ Extract `specialties` array from doctor data
- ✅ Validate specialty IDs exist before creating relations
- ✅ Create `DoctorSpecialties` junction records
- ✅ Return doctor with populated specialties
- ✅ Proper error handling for invalid specialty IDs

---

### Step 3: Update Doctor Validation Schema

**File: `src/app/modules/Doctor/doctor.validation.ts`**

```typescript
import { z } from "zod";

const create = z.object({
  body: z.object({
    email: z.string({
      error: "Email is required",
    }),
    name: z.string({
      error: "Name is required",
    }),
    profilePhoto: z.string({
      error: "Profile Photo is required",
    }),
    contactNumber: z.string({
      error: "Contact Number is required",
    }),
    registrationNumber: z.string({
      error: "Registration Number is required",
    }),
    experience: z.number({
      error: "Experience is required",
    }),
    gender: z.string({
      error: "Gender is required",
    }),
    appointmentFee: z.number({
      error: "Appointment Fee is required",
    }),
    qualification: z.string({
      error: "Qualification is required",
    }),
    currentWorkingPlace: z.string({
      error: "Current Working Place is required",
    }),
    designation: z.string({
      error: "Designation is required",
    }),
  }),
});

const update = z.object({
  body: z.object({
    name: z.string().optional(),
    profilePhoto: z.string().optional(),
    contactNumber: z.string().optional(),
    registrationNumber: z.string().optional(),
    experience: z.number().optional(),
    gender: z.string().optional(),
    appointmentFee: z.number().optional(),
    qualification: z.string().optional(),
    currentWorkingPlace: z.string().optional(),
    designation: z.string().optional(),
    // NEW: Add specialties management
    specialties: z.array(z.string().uuid()).optional(),
    removeSpecialties: z.array(z.string().uuid()).optional(),
  }),
});

export const DoctorValidation = {
  create,
  update,
};
```

**Key Changes:**

- ✅ Fixed: `apointmentFee` → `appointmentFee` (twice)
- ✅ Fixed: `Apointment Fee` → `Appointment Fee`
- ✅ Added: `specialties` array for adding new specialties
- ✅ Added: `removeSpecialties` array for removing existing specialties

---

### Step 4: Update Doctor Interface

**File: `src/app/modules/Doctor/doctor.interface.ts`**

```typescript
export type IDoctorFilterRequest = {
  searchTerm?: string | undefined;
  email?: string | undefined;
  contactNumber?: string | undefined;
  gender?: string | undefined;
  specialties?: string | undefined;
};

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
  // NEW: Simplified specialty management
  specialties?: string[]; // Array of specialty IDs to add
  removeSpecialties?: string[]; // Array of specialty IDs to remove
};
```

**Key Changes:**

- ✅ Fixed: `apointmentFee` → `appointmentFee`
- ✅ Removed: Complex `ISpecialties` type with `isDeleted` flag
- ✅ Simplified: Use plain string arrays for specialty IDs
- ✅ Made all fields optional (proper for update operation)

---

### Step 5: Update Doctor Service

**File: `src/app/modules/Doctor/doctor.service.ts`**

Replace the `updateIntoDB` function:

```typescript
const updateIntoDB = async (id: string, payload: IDoctorUpdate) => {
  const { specialties, removeSpecialties, ...doctorData } = payload;

  const doctorInfo = await prisma.doctor.findUniqueOrThrow({
    where: {
      id,
      isDeleted: false,
    },
  });

  await prisma.$transaction(async (transactionClient) => {
    // Step 1: Update doctor basic data
    if (Object.keys(doctorData).length > 0) {
      await transactionClient.doctor.update({
        where: {
          id,
        },
        data: doctorData,
      });
    }

    // Step 2: Remove specialties if provided
    if (
      removeSpecialties &&
      Array.isArray(removeSpecialties) &&
      removeSpecialties.length > 0
    ) {
      // Validate that specialties to remove exist for this doctor
      const existingDoctorSpecialties =
        await transactionClient.doctorSpecialties.findMany({
          where: {
            doctorId: doctorInfo.id,
            specialitiesId: {
              in: removeSpecialties,
            },
          },
        });

      if (existingDoctorSpecialties.length !== removeSpecialties.length) {
        const foundIds = existingDoctorSpecialties.map(
          (ds) => ds.specialitiesId
        );
        const notFound = removeSpecialties.filter(
          (id) => !foundIds.includes(id)
        );
        throw new Error(
          `Cannot remove non-existent specialties: ${notFound.join(", ")}`
        );
      }

      // Delete the specialties
      await transactionClient.doctorSpecialties.deleteMany({
        where: {
          doctorId: doctorInfo.id,
          specialitiesId: {
            in: removeSpecialties,
          },
        },
      });
    }

    // Step 3: Add new specialties if provided
    if (specialties && Array.isArray(specialties) && specialties.length > 0) {
      // Verify all specialties exist in Specialties table
      const existingSpecialties = await transactionClient.specialties.findMany({
        where: {
          id: {
            in: specialties,
          },
        },
        select: {
          id: true,
        },
      });

      const existingSpecialtyIds = existingSpecialties.map((s) => s.id);
      const invalidSpecialties = specialties.filter(
        (id) => !existingSpecialtyIds.includes(id)
      );

      if (invalidSpecialties.length > 0) {
        throw new Error(
          `Invalid specialty IDs: ${invalidSpecialties.join(", ")}`
        );
      }

      // Check for duplicates - don't add specialties that already exist
      const currentDoctorSpecialties =
        await transactionClient.doctorSpecialties.findMany({
          where: {
            doctorId: doctorInfo.id,
            specialitiesId: {
              in: specialties,
            },
          },
          select: {
            specialitiesId: true,
          },
        });

      const currentSpecialtyIds = currentDoctorSpecialties.map(
        (ds) => ds.specialitiesId
      );
      const newSpecialties = specialties.filter(
        (id) => !currentSpecialtyIds.includes(id)
      );

      // Only create new specialties that don't already exist
      if (newSpecialties.length > 0) {
        const doctorSpecialtiesData = newSpecialties.map((specialtyId) => ({
          doctorId: doctorInfo.id,
          specialitiesId: specialtyId,
        }));

        await transactionClient.doctorSpecialties.createMany({
          data: doctorSpecialtiesData,
        });
      }
    }
  });

  // Step 4: Return updated doctor with specialties
  const result = await prisma.doctor.findUnique({
    where: {
      id: doctorInfo.id,
    },
    include: {
      doctorSpecialties: {
        include: {
          specialities: true,
        },
      },
    },
  });

  return result;
};
```

**Key Changes:**

- ✅ Removed confusing `isDeleted` flag approach
- ✅ Separate arrays: `specialties` (to add) and `removeSpecialties` (to remove)
- ✅ Proper validation before add/remove operations
- ✅ Prevent duplicate specialties
- ✅ Clear error messages
- ✅ Transaction ensures atomicity

---

### Step 6: Update Doctor Constants

**File: `src/app/modules/Doctor/doctor.constants.ts`**

```typescript
export const doctorSearchableFields: string[] = [
  "name",
  "email",
  "contactNumber",
  "address",
  "qualification",
  "designation",
];

export const doctorFilterableFields: string[] = [
  "searchTerm",
  "email",
  "contactNumber",
  "gender",
  "appointmentFee",
  "specialties",
];
```

**Key Changes:**

- ✅ Fixed: `apointmentFee` → `appointmentFee`

---

## 📝 API Request/Response Examples

### Create Doctor with Specialties

**Endpoint:** `POST /user/create-doctor`

**Request Body (form-data):**

```json
{
  "data": {
    "password": "doctor123",
    "doctor": {
      "name": "Dr. John Smith",
      "email": "john.smith@hospital.com",
      "contactNumber": "+1234567890",
      "address": "123 Medical Plaza",
      "registrationNumber": "MD12345",
      "experience": 10,
      "gender": "MALE",
      "appointmentFee": 500,
      "qualification": "MBBS, MD",
      "currentWorkingPlace": "City Hospital",
      "designation": "Senior Cardiologist",
      "specialties": [
        "uuid-of-cardiology-specialty",
        "uuid-of-internal-medicine-specialty"
      ]
    }
  },
  "file": <profile-image-file>
}
```

**Response:**

```json
{
  "success": true,
  "message": "Doctor Created successfully!",
  "data": {
    "id": "doctor-uuid",
    "name": "Dr. John Smith",
    "email": "john.smith@hospital.com",
    "profilePhoto": "cloudinary-url",
    "contactNumber": "+1234567890",
    "address": "123 Medical Plaza",
    "registrationNumber": "MD12345",
    "experience": 10,
    "gender": "MALE",
    "appointmentFee": 500,
    "qualification": "MBBS, MD",
    "currentWorkingPlace": "City Hospital",
    "designation": "Senior Cardiologist",
    "isDeleted": false,
    "averageRating": 0.0,
    "createdAt": "2024-11-16T10:00:00.000Z",
    "updatedAt": "2024-11-16T10:00:00.000Z",
    "doctorSpecialties": [
      {
        "specialitiesId": "uuid-of-cardiology-specialty",
        "doctorId": "doctor-uuid",
        "specialities": {
          "id": "uuid-of-cardiology-specialty",
          "title": "Cardiology",
          "icon": "heart-icon-url"
        }
      },
      {
        "specialitiesId": "uuid-of-internal-medicine-specialty",
        "doctorId": "doctor-uuid",
        "specialities": {
          "id": "uuid-of-internal-medicine-specialty",
          "title": "Internal Medicine",
          "icon": "stethoscope-icon-url"
        }
      }
    ]
  }
}
```

---

### Update Doctor - Add New Specialties

**Endpoint:** `PATCH /doctor/:doctorId`

**Request Body:**

```json
{
  "name": "Dr. John Smith Jr.",
  "appointmentFee": 600,
  "specialties": ["uuid-of-neurology-specialty", "uuid-of-pediatrics-specialty"]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Doctor data updated!",
  "data": {
    "id": "doctor-uuid",
    "name": "Dr. John Smith Jr.",
    "appointmentFee": 600,
    "doctorSpecialties": [
      {
        "specialitiesId": "uuid-of-cardiology-specialty",
        "specialities": {
          "id": "uuid-of-cardiology-specialty",
          "title": "Cardiology",
          "icon": "heart-icon-url"
        }
      },
      {
        "specialitiesId": "uuid-of-internal-medicine-specialty",
        "specialities": {
          "id": "uuid-of-internal-medicine-specialty",
          "title": "Internal Medicine",
          "icon": "stethoscope-icon-url"
        }
      },
      {
        "specialitiesId": "uuid-of-neurology-specialty",
        "specialities": {
          "id": "uuid-of-neurology-specialty",
          "title": "Neurology",
          "icon": "brain-icon-url"
        }
      },
      {
        "specialitiesId": "uuid-of-pediatrics-specialty",
        "specialities": {
          "id": "uuid-of-pediatrics-specialty",
          "title": "Pediatrics",
          "icon": "child-icon-url"
        }
      }
    ]
  }
}
```

---

### Update Doctor - Remove Specialties

**Endpoint:** `PATCH /doctor/:doctorId`

**Request Body:**

```json
{
  "removeSpecialties": [
    "uuid-of-internal-medicine-specialty",
    "uuid-of-pediatrics-specialty"
  ]
}
```

**Response:**

```json
{
  "success": true,
  "message": "Doctor data updated!",
  "data": {
    "id": "doctor-uuid",
    "doctorSpecialties": [
      {
        "specialitiesId": "uuid-of-cardiology-specialty",
        "specialities": {
          "id": "uuid-of-cardiology-specialty",
          "title": "Cardiology",
          "icon": "heart-icon-url"
        }
      },
      {
        "specialitiesId": "uuid-of-neurology-specialty",
        "specialities": {
          "id": "uuid-of-neurology-specialty",
          "title": "Neurology",
          "icon": "brain-icon-url"
        }
      }
    ]
  }
}
```

---

### Update Doctor - Add and Remove Simultaneously

**Endpoint:** `PATCH /doctor/:doctorId`

**Request Body:**

```json
{
  "appointmentFee": 700,
  "specialties": ["uuid-of-orthopedics-specialty"],
  "removeSpecialties": ["uuid-of-neurology-specialty"]
}
```

---

## ✅ Testing Checklist

- [ ] Create doctor without specialties (should work)
- [ ] Create doctor with 1 specialty
- [ ] Create doctor with multiple specialties
- [ ] Create doctor with invalid specialty ID (should fail)
- [ ] Update doctor - add new specialties
- [ ] Update doctor - remove existing specialties
- [ ] Update doctor - add and remove in same request
- [ ] Update doctor - try to remove non-existent specialty (should fail)
- [ ] Update doctor - try to add duplicate specialty (should not create duplicate)
- [ ] Verify transaction rollback on error

---

## 🚀 Migration Steps

1. **Backup database**
2. **Update validation files** (user.validation.ts, doctor.validation.ts)
3. **Update interface files** (doctor.interface.ts)
4. **Update constant files** (doctor.constants.ts)
5. **Update service files** (user.sevice.ts, doctor.service.ts)
6. **Test all scenarios** using the checklist above
7. **Deploy to staging**
8. **Run integration tests**
9. **Deploy to production**

---

## 📚 Related Files Modified

1. `src/app/modules/User/user.validation.ts`
2. `src/app/modules/User/user.sevice.ts`
3. `src/app/modules/Doctor/doctor.validation.ts`
4. `src/app/modules/Doctor/doctor.interface.ts`
5. `src/app/modules/Doctor/doctor.constants.ts`
6. `src/app/modules/Doctor/doctor.service.ts`

---

**Next:** See `REFACTORING_GUIDE_PART_2_SPELLING_FIXES.md`
