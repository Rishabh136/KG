# KG Shop — Employee Attendance & Payroll App

A complete Android app for managing employee attendance and payroll for KG Shop.  
Built with **Kotlin · MVVM · Room Database · Material Design 3**

---

## 🗂️ Project Structure

```
KGShop/
├── app/
│   ├── build.gradle                        # App-level Gradle config
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/kgshop/
│       │   ├── data/
│       │   │   ├── dao/                    # Room DAOs
│       │   │   │   ├── EmployeeDao.kt
│       │   │   │   ├── AttendanceDao.kt
│       │   │   │   ├── AdvanceDao.kt
│       │   │   │   └── PayrollDao.kt
│       │   │   ├── database/
│       │   │   │   └── KGShopDatabase.kt   # Room DB instance
│       │   │   ├── entities/               # Data models
│       │   │   │   ├── Employee.kt
│       │   │   │   ├── Attendance.kt
│       │   │   │   ├── Advance.kt
│       │   │   │   └── Payroll.kt
│       │   │   └── repository/
│       │   │       └── KGShopRepository.kt # Single data access point
│       │   ├── ui/
│       │   │   ├── auth/
│       │   │   │   └── LoginActivity.kt
│       │   │   ├── admin/
│       │   │   │   ├── AdminMainActivity.kt
│       │   │   │   ├── AdminDashboardFragment.kt
│       │   │   │   ├── EmployeeListFragment.kt
│       │   │   │   ├── AddEditEmployeeDialog.kt
│       │   │   │   ├── AdminAttendanceFragment.kt
│       │   │   │   ├── EditAttendanceDialog.kt
│       │   │   │   ├── AdminPayrollFragment.kt
│       │   │   │   ├── AdvanceManagementFragment.kt
│       │   │   │   ├── ReportsFragment.kt
│       │   │   │   └── adapters/
│       │   │   │       ├── EmployeeAdapter.kt
│       │   │   │       ├── AttendanceAdapter.kt
│       │   │   │       ├── PayrollAdapter.kt
│       │   │   │       └── AdvanceAdapter.kt
│       │   │   └── employee/
│       │   │       ├── EmployeeMainActivity.kt
│       │   │       ├── EmployeeDashboardFragment.kt
│       │   │       ├── EmployeeAttendanceFragment.kt
│       │   │       └── EmployeePayrollFragment.kt
│       │   ├── utils/
│       │   │   ├── Converters.kt           # Room type converters
│       │   │   ├── DateUtils.kt            # Date/time helpers
│       │   │   ├── Extensions.kt           # Kotlin extension functions
│       │   │   ├── HashUtils.kt            # SHA-256 password hashing
│       │   │   └── SessionManager.kt       # Login session storage
│       │   └── viewmodel/
│       │       ├── AuthViewModel.kt
│       │       ├── EmployeeViewModel.kt
│       │       ├── AttendanceViewModel.kt
│       │       └── PayrollViewModel.kt
│       └── res/
│           ├── layout/                     # All XML layouts
│           ├── menu/                       # Navigation menus
│           ├── drawable/                   # Backgrounds, icons
│           ├── values/                     # strings, colors, themes, dimens
│           └── xml/
│               └── file_paths.xml          # FileProvider config for PDF
├── build.gradle                            # Root Gradle config
├── settings.gradle
└── gradle.properties
```

---

## 🗃️ Database Schema

### `employees`
| Column         | Type    | Notes                          |
|----------------|---------|--------------------------------|
| id             | INTEGER | PRIMARY KEY AUTOINCREMENT      |
| employeeCode   | TEXT    | Unique login code (e.g. EMP01) |
| name           | TEXT    |                                |
| phone          | TEXT    |                                |
| password       | TEXT    | SHA-256 hashed                 |
| salaryType     | TEXT    | MONTHLY or DAILY               |
| salaryAmount   | REAL    | Monthly salary or daily rate   |
| joiningDate    | TEXT    | yyyy-MM-dd                     |
| isAdmin        | INTEGER | 0 = employee, 1 = admin        |
| isActive       | INTEGER | Soft delete flag               |

### `attendance`
| Column      | Type    | Notes                                    |
|-------------|---------|------------------------------------------|
| id          | INTEGER | PRIMARY KEY                              |
| employeeId  | INTEGER | FK → employees.id                        |
| date        | TEXT    | yyyy-MM-dd (UNIQUE per employee per day) |
| timeIn      | TEXT    | HH:mm:ss                                 |
| timeOut     | TEXT    | HH:mm:ss (nullable)                      |
| totalHours  | REAL    | Auto-calculated                          |
| status      | TEXT    | PRESENT / ABSENT / HALF_DAY              |

### `advances`
| Column     | Type    | Notes                      |
|------------|---------|----------------------------|
| id         | INTEGER | PRIMARY KEY                |
| employeeId | INTEGER | FK → employees.id          |
| amount     | REAL    |                            |
| date       | TEXT    | yyyy-MM-dd                 |
| month      | TEXT    | yyyy-MM (payroll month)    |
| note       | TEXT    |                            |
| isDeducted | INTEGER | 0 = pending, 1 = deducted  |

### `payroll`
| Column          | Type    | Notes                                     |
|-----------------|---------|-------------------------------------------|
| id              | INTEGER | PRIMARY KEY                               |
| employeeId      | INTEGER | FK → employees.id                         |
| month           | TEXT    | yyyy-MM (UNIQUE per employee per month)   |
| totalWorkingDays| INTEGER | Mon–Sat in the month                      |
| presentDays     | INTEGER |                                           |
| absentDays      | INTEGER |                                           |
| grossSalary     | REAL    | Before deductions                         |
| deductions      | REAL    | Absence deductions                        |
| advanceAmount   | REAL    | Sum of advances for the month             |
| netSalary       | REAL    | grossSalary − deductions − advanceAmount  |
| isPaid          | INTEGER | 0 = pending, 1 = paid                     |
| generatedAt     | TEXT    | Date of last generation                   |

---

## 💡 Payroll Formula

**Monthly Salary Employee:**
```
Per Day = Monthly Salary ÷ Working Days in Month
Deduction = Absent Days × Per Day
Net Salary = Monthly Salary − Deduction − Total Advance
```

**Daily Wage Employee:**
```
Gross = Present Days × Daily Rate
Net Salary = Gross − Total Advance
```

---

## 🚀 How to Run in Android Studio

### Prerequisites
- Android Studio Hedgehog (2023.1) or later
- JDK 17
- Android SDK 34
- An Android device or emulator (API 24+)

### Steps

1. **Open Project**
   - Launch Android Studio
   - Click **File → Open**
   - Select the `KGShop/` folder
   - Click **OK**

2. **Sync Gradle**
   - Android Studio will automatically prompt to sync
   - Or go to **File → Sync Project with Gradle Files**
   - Wait for all dependencies to download (~2–3 min first time)

3. **Run the App**
   - Connect a device (enable USB debugging) or start an emulator
   - Click the green ▶️ **Run** button
   - Select your device and click OK

4. **First Login**
   ```
   Employee ID : ADMIN
   Password    : admin123
   ```
   The admin account is seeded automatically when the database is first created.

---

## 🔐 Adding Employees (Admin)

1. Log in as Admin
2. Navigate to **Employees** from the drawer
3. Tap the **+** FAB button
4. Fill in:
   - Employee Code (e.g. `EMP01`)
   - Full Name
   - Phone
   - Password (employee uses this to log in)
   - Salary Type (Monthly / Daily Wage)
   - Salary Amount
   - Joining Date

---

## 📅 Attendance Flow

**Employee:**
1. Opens app → Dashboard shows today's status
2. Taps **CLOCK IN** → records Time In
3. Taps **CLOCK OUT** → records Time Out + calculates hours
4. Cannot clock in twice on the same day

**Admin:**
1. Can view all attendance by month
2. Can tap any record to edit it
3. Can manually add attendance via the + button

---

## 💰 Payroll Flow

1. Admin navigates to **Payroll**
2. Selects the month using arrow buttons
3. Taps **Generate / Recalculate Payroll**
4. The system calculates for all active employees
5. Admin can mark individual employees as **Paid**

---

## 📥 PDF Export

1. Go to **Reports**
2. Select month
3. Tap **Export Payroll as PDF**
4. The PDF is saved to external storage and opened automatically
5. Share or print from the PDF viewer

---

## 🛠️ Dependencies Used

| Library              | Version | Purpose                  |
|----------------------|---------|--------------------------|
| Room                 | 2.6.1   | Local SQLite database    |
| ViewModel/LiveData   | 2.7.0   | MVVM architecture        |
| Material Components  | 1.11.0  | UI design                |
| Navigation Component | 2.7.6   | Fragment navigation      |
| Kotlin Coroutines    | 1.7.3   | Async operations         |
| KSP                  | 1.9.0   | Annotation processing    |

> **Note:** PDF generation uses Android's built-in `android.graphics.pdf.PdfDocument` — no external library needed!

---

## ⚠️ Troubleshooting

| Issue | Fix |
|-------|-----|
| Build fails on KSP | Ensure KSP version matches Kotlin version in `build.gradle` |
| Database not initialized | Uninstall app and reinstall — admin is seeded on first launch |
| PDF doesn't open | Install a PDF viewer app on the device |
| `minSdkVersion` error | Ensure device/emulator is API 24 or higher |

---

## 📸 Features Summary

| Feature               | Admin | Employee |
|-----------------------|-------|----------|
| Login                 | ✅    | ✅       |
| Add/Edit/Delete Emp   | ✅    | ❌       |
| View All Employees    | ✅    | ❌       |
| Clock In/Out          | ✅    | ✅       |
| Edit Attendance       | ✅    | ❌       |
| View Own Attendance   | ✅    | ✅       |
| Generate Payroll      | ✅    | ❌       |
| View Own Payroll      | ✅    | ✅       |
| Record Advances       | ✅    | ❌       |
| Export PDF Reports    | ✅    | ❌       |
| Dashboard Stats       | ✅    | ✅ (own) |
