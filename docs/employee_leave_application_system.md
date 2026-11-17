# Employee Leave Application System

## A. Project Description

The Employee Leave Application System is a web-based application designed to help an administrative office manage employee leave requests electronically.

Instead of submitting paper-based leave forms, employees can log in to the system, submit leave applications, select the type of leave, provide the dates and reason for their absence, and monitor the status of their applications.

The administrator or authorized office personnel can review submitted applications, approve or reject requests, manage employee records, and generate leave reports.

The system aims to make leave processing more organized, efficient, and secure while protecting confidential employee information.

---

## B. Objectives

The project aims to:

1. Develop a centralized system for managing employee leave applications.
2. Allow employees to submit leave requests online.
3. Enable administrators to review, approve, or reject applications.
4. Maintain accurate employee leave records.
5. Generate reports about employee leave transactions.
6. Protect employee information through authentication and access control.

---

## C. Main System Features

### Employee Module

- Employee registration or account creation by the administrator.
- Secure login and logout.
- View personal profile.
- Submit leave applications:
  - Select leave type.
  - Specify start date and end date.
  - Provide reason for leave.
  - Upload supporting documents when required.
- View application status (Pending, Approved, or Rejected).
- View leave application history.

### Administrator Module

- Admin login.
- Dashboard showing the number of employees and leave applications.
- Add, edit, and manage employee accounts.
- View all submitted leave applications.
- Review leave details and supporting documents.
- Approve or reject leave requests.
- Provide remarks or reasons for rejected applications.
- Manage leave types.
- View employee leave history.
- Generate and print leave reports.

---

## D. User Roles

| Employee                         | Administrator               |
| :------------------------------- | :-------------------------- |
| Log in and log out               | Log in and log out          |
| Submit leave application         | Manage employee accounts    |
| View personal leave records      | View all leave applications |
| Upload supporting documents      | Approve or reject requests  |
| Track application status         | Generate leave reports      |
| Update permitted profile details | Manage leave types          |

---

## E. Security Requirements

Since this is an Information Assurance and Security project, students must implement:

1. **Password Hashing** - Passwords must not be stored as plain text.
2. **Authentication** - Only registered users can access the system.
3. **Role-Based Access Control** - Employees cannot access administrator pages.
4. **Data Privacy** - Employees can only view their own leave applications.
5. **SQL Injection Prevention** - Use prepared statements for database queries.
6. **Input Validation** - Validate dates, required fields, and uploaded documents.
7. **Activity Logs** - Record important actions, such as leave approval and rejection.
8. **Session Security** - Prevent unauthorized access to logged-in accounts.

---

## G. Example System Workflow

1. Employee logs in.
2. Employee selects **Apply for Leave**.
3. Employee fills out the leave application form.
4. System saves the application with a **Pending** status.
5. Administrator logs in and reviews the application.
6. Administrator approves or rejects the request.
7. Employee logs in again and checks the application status.
