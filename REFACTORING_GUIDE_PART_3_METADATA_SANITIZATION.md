# Part 3: Metadata and Data Sanitization - Complete Guide

## 📋 Overview

This guide ensures:

1. **Consistent metadata** is returned from all paginated APIs
2. **Proper data sanitization** is implemented across all endpoints
3. **Security best practices** for sensitive data handling

---

## 🔍 Current State Analysis

### APIs Returning Metadata ✅

The following services **correctly** return metadata:

1. **AdminService.getAllFromDB** ✅
2. **UserService.getAllFromDB** ✅
3. **DoctorService.getAllFromDB** ✅
4. **PatientService.getAllFromDB** ✅
5. **AppointmentService.getMyAppointment** ✅
6. **AppointmentService.getAllFromDB** ✅
7. **DoctorScheduleService.getMySchedule** ✅
8. **DoctorScheduleService.getAllFromDB** ✅
9. **ScheduleService.getAllFromDB** ✅
10. **PrescriptionService.patientPrescription** ✅
11. **PrescriptionService.getAllFromDB** ✅
12. **ReviewService.getAllFromDB** ✅

### APIs NOT Returning Metadata ⚠️

These need attention:

1. **SpecialtiesService.getAllFromDB** ❌ (No pagination)
2. **DoctorService.getByIdFromDB** ❌ (Single item - no pagination needed)
3. **AdminService.getByIdFromDB** ❌ (Single item - no pagination needed)
4. **PatientService.getByIdFromDB** ❌ (Single item - no pagination needed)
5. **ScheduleService.getByIdFromDB** ❌ (Single item - no pagination needed)

---

## 🛠️ Fix 1: Add Pagination to Specialties

**Problem:** `SpecialtiesService.getAllFromDB` returns raw array without metadata.

**File: `src/app/modules/Specialties/specialties.service.ts`**

**CURRENT CODE:**

```typescript
const getAllFromDB = async (): Promise<Specialties[]> => {
  return await prisma.specialties.findMany();
};
```

**REPLACE WITH:**

```typescript
import { IPaginationOptions } from "../../interfaces/pagination";
import { paginationHelper } from "../../../helpers/paginationHelper";

const getAllFromDB = async (options: IPaginationOptions) => {
  const { limit, page, skip } = paginationHelper.calculatePagination(options);

  const result = await prisma.specialties.findMany({
    skip,
    take: limit,
    orderBy:
      options.sortBy && options.sortOrder
        ? { [options.sortBy]: options.sortOrder }
        : { createdAt: "desc" },
  });

  const total = await prisma.specialties.count();

  return {
    meta: {
      total,
      page,
      limit,
    },
    data: result,
  };
};
```

**File: `src/app/modules/Specialties/specialties.controller.ts`**

**CURRENT CODE:**

```typescript
const getAllFromDB = catchAsync(async (req: Request, res: Response) => {
  const result = await SpecialtiesService.getAllFromDB();
  sendResponse(res, {
    statusCode: httpStatus.OK,
    success: true,
    message: "Specialties data fetched successfully",
    data: result,
  });
});
```

**REPLACE WITH:**

```typescript
import pick from "../../../shared/pick";

const getAllFromDB = catchAsync(async (req: Request, res: Response) => {
  const options = pick(req.query, ["limit", "page", "sortBy", "sortOrder"]);
  const result = await SpecialtiesService.getAllFromDB(options);
  sendResponse(res, {
    statusCode: httpStatus.OK,
    success: true,
    message: "Specialties data fetched successfully",
    meta: result.meta,
    data: result.data,
  });
});
```

---

## 🔒 Data Sanitization Audit

### Password Handling ✅

**File: `src/app/modules/User/user.service.ts`**

**CURRENT CODE (getAllFromDB):**

```typescript
select: {
    id: true,
    email: true,
    role: true,
    needPasswordChange: true,
    status: true,
    createdAt: true,
    updatedAt: true,
    admin: true,
    patient: true,
    doctor: true
}
```

**✅ GOOD:** Password is explicitly excluded. No changes needed.

---

### Sensitive Data in Profile APIs ⚠️

**File: `src/app/modules/User/user.service.ts`**

**ISSUE:** `getMyProfile` returns sensitive user data including `needPasswordChange`.

**FIND (getMyProfile function):**

```typescript
const getMyProfile = async (user: IAuthUser) => {
  const userInfo = await prisma.user.findUniqueOrThrow({
    where: {
      email: user?.email,
      status: UserStatus.ACTIVE,
    },
    select: {
      id: true,
      email: true,
      needPasswordChange: true,
      role: true,
      status: true,
    },
  });

  let profileInfo;

  if (userInfo.role === UserRole.SUPER_ADMIN) {
    profileInfo = await prisma.admin.findUnique({
      where: {
        email: userInfo.email,
      },
    });
  } else if (userInfo.role === UserRole.ADMIN) {
    profileInfo = await prisma.admin.findUnique({
      where: {
        email: userInfo.email,
      },
    });
  } else if (userInfo.role === UserRole.DOCTOR) {
    profileInfo = await prisma.doctor.findUnique({
      where: {
        email: userInfo.email,
      },
    });
  } else if (userInfo.role === UserRole.PATIENT) {
    profileInfo = await prisma.patient.findUnique({
      where: {
        email: userInfo.email,
      },
    });
  }

  return { ...userInfo, ...profileInfo };
};
```

**REPLACE WITH (Better Sanitization):**

```typescript
const getMyProfile = async (user: IAuthUser) => {
  const userInfo = await prisma.user.findUniqueOrThrow({
    where: {
      email: user?.email,
      status: UserStatus.ACTIVE,
    },
    select: {
      id: true,
      email: true,
      needPasswordChange: true,
      role: true,
      status: true,
    },
  });

  let profileInfo;

  if (userInfo.role === UserRole.SUPER_ADMIN) {
    profileInfo = await prisma.admin.findUnique({
      where: {
        email: userInfo.email,
      },
      select: {
        id: true,
        name: true,
        email: true,
        profilePhoto: true,
        contactNumber: true,
        isDeleted: true,
        createdAt: true,
        updatedAt: true,
      },
    });
  } else if (userInfo.role === UserRole.ADMIN) {
    profileInfo = await prisma.admin.findUnique({
      where: {
        email: userInfo.email,
      },
      select: {
        id: true,
        name: true,
        email: true,
        profilePhoto: true,
        contactNumber: true,
        isDeleted: true,
        createdAt: true,
        updatedAt: true,
      },
    });
  } else if (userInfo.role === UserRole.DOCTOR) {
    profileInfo = await prisma.doctor.findUnique({
      where: {
        email: userInfo.email,
      },
      select: {
        id: true,
        name: true,
        email: true,
        profilePhoto: true,
        contactNumber: true,
        address: true,
        registrationNumber: true,
        experience: true,
        gender: true,
        appointmentFee: true,
        qualification: true,
        currentWorkingPlace: true,
        designation: true,
        averageRating: true,
        isDeleted: true,
        createdAt: true,
        updatedAt: true,
        doctorSpecialties: {
          include: {
            specialities: true,
          },
        },
      },
    });
  } else if (userInfo.role === UserRole.PATIENT) {
    profileInfo = await prisma.patient.findUnique({
      where: {
        email: userInfo.email,
      },
      select: {
        id: true,
        name: true,
        email: true,
        profilePhoto: true,
        contactNumber: true,
        address: true,
        isDeleted: true,
        createdAt: true,
        updatedAt: true,
        patientHealthData: true,
        medicalReport: {
          select: {
            id: true,
            patientId: true,
            reportName: true,
            reportLink: true,
            createdAt: true,
            updatedAt: true,
          },
        },
      },
    });
  }

  return { ...userInfo, ...profileInfo };
};
```

**✅ IMPROVEMENT:** Explicit field selection prevents accidental data leaks.

---

### Transaction ID Exposure ⚠️

**File: `src/app/modules/Payment/payment.service.ts`**

**ISSUE:** Transaction IDs might be exposed in responses.

**RECOMMENDATION:** Ensure payment responses sanitize sensitive payment gateway data.

**FIND (validatePayment function):**

```typescript
await prisma.$transaction(async (tx) => {
  const updatedPaymentData = await tx.payment.update({
    where: {
      transactionId: response.tran_id,
    },
    data: {
      status: PaymentStatus.PAID,
      paymentGatewayData: response,
    },
  });

  await tx.appointment.update({
    where: {
      id: updatedPaymentData.appointmentId,
    },
    data: {
      paymentStatus: PaymentStatus.PAID,
    },
  });
});

return {
  message: "Payment success!",
};
```

**✅ GOOD:** Already sanitized - only returns success message, not sensitive payment data.

---

### Email Exposure in Public APIs ⚠️

**File: `src/app/modules/Doctor/doctor.service.ts`**

**CURRENT:** Doctor email is exposed in `getAllFromDB`.

**RECOMMENDATION:** Create a separate public API without email exposure for non-authenticated users.

**ADD NEW FUNCTION:**

```typescript
const getAllPublic = async (
  filters: IDoctorFilterRequest,
  options: IPaginationOptions
) => {
  const { limit, page, skip } = paginationHelper.calculatePagination(options);
  const { searchTerm, specialties, ...filterData } = filters;

  const andConditions: Prisma.DoctorWhereInput[] = [];

  if (searchTerm) {
    andConditions.push({
      OR: doctorSearchableFields.map((field) => ({
        [field]: {
          contains: searchTerm,
          mode: "insensitive",
        },
      })),
    });
  }

  if (specialties && specialties.length > 0) {
    andConditions.push({
      doctorSpecialties: {
        some: {
          specialities: {
            title: {
              contains: specialties,
              mode: "insensitive",
            },
          },
        },
      },
    });
  }

  if (Object.keys(filterData).length > 0) {
    const filterConditions = Object.keys(filterData).map((key) => ({
      [key]: {
        equals: (filterData as any)[key],
      },
    }));
    andConditions.push(...filterConditions);
  }

  andConditions.push({
    isDeleted: false,
  });

  const whereConditions: Prisma.DoctorWhereInput =
    andConditions.length > 0 ? { AND: andConditions } : {};

  const result = await prisma.doctor.findMany({
    where: whereConditions,
    skip,
    take: limit,
    orderBy:
      options.sortBy && options.sortOrder
        ? { [options.sortBy]: options.sortOrder }
        : { averageRating: "desc" },
    select: {
      id: true,
      name: true,
      // email: false, // Hide email in public API
      profilePhoto: true,
      contactNumber: true,
      address: true,
      registrationNumber: true,
      experience: true,
      gender: true,
      appointmentFee: true,
      qualification: true,
      currentWorkingPlace: true,
      designation: true,
      averageRating: true,
      createdAt: true,
      updatedAt: true,
      doctorSpecialties: {
        include: {
          specialities: true,
        },
      },
      review: {
        select: {
          rating: true,
          comment: true,
          createdAt: true,
          patient: {
            select: {
              name: true,
              profilePhoto: true,
            },
          },
        },
      },
    },
  });

  const total = await prisma.doctor.count({
    where: whereConditions,
  });

  return {
    meta: {
      total,
      page,
      limit,
    },
    data: result,
  };
};
```

**Export:**

```typescript
export const DoctorService = {
  updateIntoDB,
  getAllFromDB,
  getAllPublic, // NEW
  getByIdFromDB,
  deleteFromDB,
  softDelete,
  getAISuggestion,
};
```

---

### Patient Health Data Privacy ⚠️

**File: `src/app/modules/Patient/patient.services.ts`**

**CURRENT:** Patient health data is included in `getAllFromDB` - might be too much for admin views.

**RECOMMENDATION:** Add role-based data filtering.

**ADD NEW PARAMETER TO getAllFromDB:**

```typescript
const getAllFromDB = async (
  filters: IPatientFilterRequest,
  options: IPaginationOptions,
  includeHealthData: boolean = false // NEW PARAMETER
) => {
  const { limit, page, skip } = paginationHelper.calculatePagination(options);
  const { searchTerm, ...filterData } = filters;

  const andConditions = [];

  if (searchTerm) {
    andConditions.push({
      OR: patientSearchableFields.map((field) => ({
        [field]: {
          contains: searchTerm,
          mode: "insensitive",
        },
      })),
    });
  }

  if (Object.keys(filterData).length > 0) {
    andConditions.push({
      AND: Object.keys(filterData).map((key) => {
        return {
          [key]: {
            equals: (filterData as any)[key],
          },
        };
      }),
    });
  }

  andConditions.push({
    isDeleted: false,
  });

  const whereConditions: Prisma.PatientWhereInput =
    andConditions.length > 0 ? { AND: andConditions } : {};

  // Conditional include based on parameter
  const includeClause = includeHealthData
    ? {
        medicalReport: true,
        patientHealthData: true,
      }
    : {
        medicalReport: {
          select: {
            id: true,
            reportName: true,
            createdAt: true,
          },
        },
      };

  const result = await prisma.patient.findMany({
    where: whereConditions,
    skip,
    take: limit,
    orderBy:
      options.sortBy && options.sortOrder
        ? { [options.sortBy]: options.sortOrder }
        : {
            createdAt: "desc",
          },
    include: includeClause,
  });

  const total = await prisma.patient.count({
    where: whereConditions,
  });

  return {
    meta: {
      total,
      page,
      limit,
    },
    data: result,
  };
};
```

---

## 🔐 Input Validation & Sanitization

### Add Input Trimming Middleware

**NEW FILE: `src/app/middlewares/sanitizeInput.ts`**

```typescript
import { NextFunction, Request, Response } from "express";

/**
 * Middleware to trim string inputs and remove potentially harmful characters
 */
export const sanitizeInput = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  // Sanitize body
  if (req.body) {
    req.body = sanitizeObject(req.body);
  }

  // Sanitize query
  if (req.query) {
    req.query = sanitizeObject(req.query);
  }

  // Sanitize params
  if (req.params) {
    req.params = sanitizeObject(req.params);
  }

  next();
};

const sanitizeObject = (obj: any): any => {
  if (typeof obj !== "object" || obj === null) {
    return sanitizeValue(obj);
  }

  if (Array.isArray(obj)) {
    return obj.map(sanitizeObject);
  }

  const sanitized: any = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      sanitized[key] = sanitizeObject(obj[key]);
    }
  }
  return sanitized;
};

const sanitizeValue = (value: any): any => {
  if (typeof value === "string") {
    // Trim whitespace
    value = value.trim();

    // Remove null bytes
    value = value.replace(/\0/g, "");

    // HTML encode dangerous characters (optional - depends on use case)
    // value = value.replace(/[<>]/g, '');
  }
  return value;
};
```

**File: `src/app.ts`**

**ADD AFTER cors() middleware:**

```typescript
import { sanitizeInput } from "./app/middlewares/sanitizeInput";

// ... existing code ...

app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(sanitizeInput); // ADD THIS LINE
```

---

## 📊 Metadata Consistency Checklist

### Standard Metadata Structure

All paginated responses should follow this structure:

```typescript
{
  success: true,
  message: string,
  meta: {
    total: number,
    page: number,
    limit: number
  },
  data: Array<any>
}
```

### Services Already Compliant ✅

- [x] AdminService.getAllFromDB
- [x] UserService.getAllFromDB
- [x] DoctorService.getAllFromDB
- [x] PatientService.getAllFromDB
- [x] AppointmentService.getMyAppointment
- [x] AppointmentService.getAllFromDB
- [x] DoctorScheduleService.getMySchedule
- [x] DoctorScheduleService.getAllFromDB
- [x] ScheduleService.getAllFromDB
- [x] PrescriptionService.patientPrescription
- [x] PrescriptionService.getAllFromDB
- [x] ReviewService.getAllFromDB

### Services Needing Update ⚠️

- [ ] SpecialtiesService.getAllFromDB (Updated in this guide)

---

## 🛡️ Security Best Practices Summary

### 1. Password Handling ✅

- Never return password field
- Always hash passwords with bcrypt
- Use strong salt rounds (currently using config.salt_round)

### 2. Sensitive Data ✅

- Explicit field selection in queries
- Role-based data access
- Sanitize payment gateway responses

### 3. Input Validation ✅

- Use Zod schemas for validation
- Trim string inputs
- Remove null bytes
- Validate UUIDs for IDs

### 4. Error Messages ⚠️

**RECOMMENDATION:** Don't expose internal errors to clients.

**FILE: `src/app/middlewares/globalErrorHandler.ts`**

Ensure error handler doesn't leak sensitive info:

```typescript
// Add this check
const sanitizeError = (error: any) => {
  // Don't expose Prisma errors in production
  if (process.env.NODE_ENV === "production" && error.code?.startsWith("P")) {
    return {
      message: "Database operation failed",
      errorDetails: null,
    };
  }
  return error;
};
```

### 5. Rate Limiting ⚠️

**RECOMMENDATION:** Add rate limiting for public APIs.

**NEW FILE: `src/app/middlewares/rateLimiter.ts`**

```typescript
import rateLimit from "express-rate-limit";

export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: "Too many requests from this IP, please try again later.",
  standardHeaders: true,
  legacyHeaders: false,
});

export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Limit login attempts
  message: "Too many login attempts, please try again later.",
  skipSuccessfulRequests: true,
});
```

**File: `src/app/routes/index.ts`**

```typescript
import { apiLimiter } from "../middlewares/rateLimiter";

router.use(apiLimiter); // Apply to all routes
```

**File: `src/app/modules/Auth/auth.routes.ts`**

```typescript
import { authLimiter } from "../../middlewares/rateLimiter";

router.post("/login", authLimiter, AuthController.loginUser);
```

---

## 📋 Implementation Checklist

### Metadata Fixes:

- [ ] Update SpecialtiesService.getAllFromDB
- [ ] Update SpecialtiesController.getAllFromDB
- [ ] Test all paginated endpoints

### Data Sanitization:

- [ ] Add explicit select fields to User.getMyProfile
- [ ] Create Doctor.getAllPublic for non-authenticated users
- [ ] Add includeHealthData parameter to Patient.getAllFromDB
- [ ] Create sanitizeInput middleware
- [ ] Add rate limiting middleware
- [ ] Update error handler for production

### Testing:

- [ ] Test all GET endpoints return proper metadata
- [ ] Test sensitive data is not exposed
- [ ] Test input sanitization works
- [ ] Test rate limiting
- [ ] Test error responses don't leak info

---

## 📦 Required Dependencies

```bash
npm install express-rate-limit
```

---

## 🚀 Deployment Notes

1. **Environment Variables:**

   ```env
   NODE_ENV=production
   RATE_LIMIT_WINDOW_MS=900000
   RATE_LIMIT_MAX_REQUESTS=100
   ```

2. **Prisma Client:**

   ```bash
   npx prisma generate
   ```

3. **Build:**

   ```bash
   npm run build
   ```

4. **Test:**
   - Run all API tests
   - Verify metadata structure
   - Check sensitive data is hidden
   - Test rate limiting

---

**Next:** See `REFACTORING_GUIDE_PART_4_SUMMARY.md`
