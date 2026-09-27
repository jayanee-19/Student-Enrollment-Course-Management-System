# Student Enrollment & Course Management System (SECMS)

## 1. Project Title

**Student Enrollment & Course Management System (SECMS)** is a Salesforce Lightning application for managing student, course, instructor, and enrollment records.

## 2. Project Overview

SECMS brings the core enrollment workflow into a Salesforce app. Users can maintain student, course, and instructor information, create new or re-enrollment records, assign instructors through record-triggered automation, create follow-up tasks for requested enrollments, and route eligible enrollment requests for approval. Reports and a dashboard summarize student status, course enrollments, paid fees, and pending approvals.

The project is implemented as Salesforce metadata in a Salesforce DX source project. This README describes the configuration checked into this repository; org-specific behavior depends on deploying and configuring that metadata in Salesforce.

## 3. Objectives

- Organize student, course, instructor, and enrollment information in related Salesforce records.
- Support new enrollment and re-enrollment record entry.
- Automate enrollment dates and category-based instructor assignment.
- Provide an approval path for eligible, fee-paid enrollment requests.
- Present enrollment and fee information through Salesforce reports and a dashboard.
- Demonstrate Salesforce configuration, declarative automation, access controls, and source-based deployment.

## 4. Key Features

- Lightning app with navigation tabs for Home, Students, Courses, Instructors, Enrollments, Reports, Dashboards, and Tasks.
- Custom objects for Student, Course, Instructor, and Enrollment.
- Two enrollment record types with separate layouts.
- Active validation rules for student email format and fee status when an enrollment is marked approved.
- Four active record-triggered flows: setting the enrollment date, assigning instructors, handling enrollment outcomes, and creating the Requested-enrollment follow-up Task.
- `Create_Followup_Task` creates a follow-up Task for Requested Enrollments, assigned to the SECMS Admin user and due two days later.
- Enrollment related lists on Student, Course, and Instructor layouts with enrollment names, linked Student/Course/Instructor names, date, status, fee state, and amount.
- An active Enrollment approval process assigned to the Training Manager queue.
- Four reports and a student management dashboard.
- Permission sets for enrollment officers, instructors, and dashboard/report editors.

## 5. Salesforce Objects

All four business records are custom Salesforce objects.

### Student

`Student__c` stores student information. Configured fields include registration number, serial number, date of birth, address, email, phone, student status, enrollment status, and fees paid.

### Course

`course__c` stores course information, including description, department, category, duration, fees, and an optional Student lookup. Course categories configured in the metadata are Technical, Language, and Non-Technical.

### Instructor

`Instructor__c` stores instructor information, including instructor code, email, phone, and expertise.

### Enrollment

`Enrollment__c` records a student's course enrollment. Fields include Student, Course, Instructor, enrollment date, enrollment status, fees paid, total amount, comments, previous enrollment, and re-enrollment reason. Read-only formula fields `Student_Name__c`, `Course_Name__c`, and `Instructor_Name__c` expose linked record names for related-list columns. Enrollment status values are Requested, Approved, and Rejected.

## 6. Object Relationships

The configured relationships are lookups:

- An Enrollment can reference one Student through `Enrollment__c.Student__c`; a Student can be referenced by multiple Enrollment records.
- An Enrollment can reference one Course through `Enrollment__c.Course__c`; a Course can be referenced by multiple Enrollment records.
- An Enrollment can reference one Instructor through `Enrollment__c.Instructor__c`; an Instructor can be referenced by multiple Enrollment records.
- An Enrollment can reference a prior Enrollment through `Enrollment__c.Previous_Enrollment__c`, supporting a link to an earlier record.
- A Course can optionally reference a Student through `course__c.Student__c`.
- Student, Course, and Instructor layouts display an Enrollment related list with Enrollment Name, Student, Course, Instructor, Enrollment Date, Enrollment Status, Fees Paid, and Total Amount. The three linked record name columns use read-only Enrollment formula fields.

These fields are lookup relationships in the metadata, not master-detail relationships.

## 7. Record Types

Both active record types are defined on `Enrollment__c`. Each uses Requested as the default Enrollment Status value and has its own page layout.

- **New Enrollment** (`New_Enrollment`) — for a new enrollment request; uses the New Enrollment layout.
- **Re-Enrollment** (`Re_Enrollment`) — for a returning enrollment; uses the Re-Enrollment layout and includes previous-enrollment and re-enrollment-reason fields in the object configuration.

## 8. Validation Rules

The repository defines these active validation rules:

- **Student email domain:** `Student__c.e_mail__c` must contain `@student.edu`. The error is displayed on the email field.
- **Student fees before approval:** prevents a Student record from having Enrollment Status `Approved` while `Fees_paid__c` is false.
- **Enrollment fees before approval:** prevents an Enrollment record from having Enrollment Status `Approved` while `Fees_Paid__c` is false.

The Student-level fee rule checks the Student status field; the Enrollment-level rule checks the Enrollment status field.

## 9. Flows & Automation

The following active flows are included in the source metadata:

### Set Enrollment Date

A before-save flow on Enrollment runs when a record is created and sets `Enrollment_Date__c` to the current date.

### Auto Assign Instructor

An after-save flow on Enrollment runs on creation and update. It reads the related Course category and looks up an Instructor by configured instructor code: `INSTR_A` for Technical, `INSTR_B` for Language, and `INSTR_C` for Non-Technical. When a matching instructor is found, the flow updates the Enrollment's Instructor lookup. The routing uses these codes; it does not select by capacity or availability.

### Create Follow-up Task

The active flow API name is `Create_Followup_Task`. It runs after an Enrollment is created or updated when `Enrollment_Status__c` is `Requested`. The Task subject is `Follow up on enrollment request.`, the owner is Jayanee R (the active SECMS System Administrator), `WhatId` is the Enrollment Id, and `ActivityDate` is `TODAY() + 2`. Enrollment has Activities enabled so a Task can be related to it. The Task owner Id is specific to the connected SECMS org and must be updated when deploying this Flow to another org.

### Enrollment Approved Action

An after-save flow on Enrollment runs on update when Enrollment Status changes to Approved or Rejected. It finds the related Student; for an approved enrollment, it sets the Student's `Status__c` to Active. If the Student has an email address, it sends an outcome email based on the enrollment status. The flow metadata defines the email actions; successful delivery depends on Salesforce org email configuration.

## 10. Approval Process

The active approval process is **Approval Enrollment Request** on Enrollment. Its entry criteria require Enrollment Status `Requested` and `Fees_Paid__c = TRUE`. The record owner may submit the request, which is assigned to the **Training Manager** queue. Approval sets Enrollment Status to Approved and locks the record; rejection sets the status to Rejected. Approval history is enabled, and recall is allowed. The process allows editing by administrators while pending.

## 11. Reports

The `SECMS_Reports` folder contains these reports:

- **Students by Status** — groups student records by student status and includes email and phone.
- **Enrollments by Course** — groups enrollment records by course and includes student and enrollment status.
- **Revenue Report Fees Paid** — filters to enrollments marked Fees Paid and summarizes Total Amount by course.
- **Pending Enrollment Approvals** — filters to Requested enrollments and includes student, course, total amount, owner, and created date.

## 12. Dashboard

The **Student Management Dashboard** is configured as a logged-in-user dashboard. It contains four components based on the reports above:

- Donut chart: Students by Status.
- Bar chart: Enrollments by Course.
- Gauge: Revenue Report - Fees Paid.
- Table: Pending Enrollment Approvals.

## 13. Security

The project includes an Admin profile and three permission sets. Student and Course sharing defaults are Public Read/Write, and Enrollment sharing is Private; a live metadata retrieve confirmed these settings before and after deployment.

- **Enrollment Officer Access** grants create/read/edit access to Enrollment records and read access to Students and Courses, with field-level permissions and visibility to both Enrollment record types.
- **Instructor Enrollment Read** grants read-only object and field access to Enrollment records, including the linked-name formula fields.
- **Dashboard Editor** grants report and dashboard creation/customization and report-running permissions.

Enrollment is configured with Private sharing. The other custom objects have their own sharing settings in metadata. Assign profiles and permission sets according to the target org's access policy; permission-set definitions alone do not assign them to users.

## 14. Testing & Verification

This repository contains Salesforce metadata and project tooling configuration. It does not contain Apex classes, Apex triggers, Lightning Web Components, or corresponding Apex/LWC test files. The package scripts define linting and LWC Jest commands, but no LWC source or Jest tests are included in the tracked project files.

The live Flow deployment succeeded as deployment `0AfgK00000UdfmnSAB`. Test Enrollment `CODEX_FLOW_FINAL_REQUESTED_20260927` (`a06gK00000P1KpVQAV`) created Task `00TgK00000CYFCDUA5`; the live Task query confirmed the exact subject, owner Jayanee R (System Administrator), matching `WhatId`, and ActivityDate `2026-09-29` (Salesforce `TODAY() + 2`). Test Enrollment `CODEX_FLOW_FINAL_NEGATIVE_20260927` (`a06gK00000P1KvxQAF`) had zero Tasks while Rejected; changing it to Requested created the follow-up Task, verifying both trigger paths. Raw results are under `codex-output/deployments/followup-task-flow-deployment.txt` and `codex-output/tests/final-flow-requested-enrollment-create-response.txt`, `final-flow-requested-task-verification.txt`, `final-flow-nonrequested-verification.txt`, and `final-flow-update-to-requested-verification.txt`. The three parent layouts were retrieved and their configured related-list columns and formula values were verified in the earlier live evidence under `codex-output/audit/` and `codex-output/tests/`.

The screenshots below are the original 15 project screenshots and document existing app screens and workflow states. No new UI screenshot was captured for the Task and expanded parent related lists in this environment. Email delivery and effective access under a separate instructor user were not directly tested.

The full project PDF was reviewed. Its core objects, app, layouts, validations, flows, approval process, reports, dashboard, sharing defaults, and screenshots are represented in this project. The PDF also names duplicate-course prevention, seat-availability checks, Apex-trigger confirmation/record updates, and instructor access to assigned records. Those items remain incomplete because the PDF does not define duplicate matching across re-enrollments, course capacity and seat reservation rules, trigger events/acceptance behavior, or an Instructor-to-User mapping. Instructor-only ownership is shown as an optional setup in the PDF. The complete requirement-by-requirement audit and exact validation result are in [the final project report](docs/SECMS_Final_Report.md).

## 15. Screenshots

The following screenshots are stored in [`screenshots/`](screenshots/):

1. [SECMS Home](screenshots/01-secsm-home.png)
2. [Student Record](screenshots/02-student-record.png)
3. [Student Validation](screenshots/03-student-validation.png)
4. [Course Record](screenshots/04-course-record.png)
5. [Instructor Record](screenshots/05-instructor-record.png)
6. [New Enrollment](screenshots/06-new-enrollment.png)
7. [Enrollment Record](screenshots/07-enrollment-record.png)
8. [Enrollment Date Flow](screenshots/08-enrollment-date-flow.png)
9. [Instructor Assignment](screenshots/09-instructor-assignment.png)
10. [Submit for Approval](screenshots/10-submit-approval.png)
11. [Approval History](screenshots/11-approval-history.png)
12. [Approved Enrollment](screenshots/12-approved-enrollment.png)
13. [Rejected Enrollment](screenshots/13-rejected-enrollment.png)
14. [Reports](screenshots/14-reports.png)
15. [Dashboard](screenshots/15-dashboard.png)

## 16. Project Structure

```text
.
├── config/
│   └── project-scratch-def.json
├── force-app/main/default/
│   ├── applications/       # SECMS Lightning app
│   ├── approvalProcesses/  # Enrollment approval process
│   ├── dashboards/         # Student management dashboard
│   ├── flows/              # Record-triggered flows
│   ├── layouts/            # Object and record-type layouts
│   ├── objects/             # Custom objects, fields, rules, record types
│   ├── permissionsets/     # Role-focused permission sets
│   ├── profiles/           # Admin profile
│   ├── queues/             # Training Manager queue
│   ├── reports/             # SECMS reports
│   ├── tabs/                # Custom object tabs
│   └── workflows/           # Enrollment status field updates
├── screenshots/             # Project screenshots
├── docs/                    # PDF requirement audit and final report
├── package.json             # JavaScript development tooling scripts
└── sfdx-project.json        # Salesforce DX project configuration
```

## 17. Technologies Used

- Salesforce Platform and Lightning Experience.
- Salesforce DX source format and Salesforce CLI (`sf`).
- Salesforce custom objects, fields, validation rules, record types, layouts, reports, dashboard, approval process, queue, profile, permission sets, and record-triggered flows.
- Salesforce Tasks related to Enrollment records and formula fields used to display linked Student, Course, and Instructor names.
- Node.js/npm project tooling configured for ESLint, Prettier, and Salesforce LWC Jest. The repository currently has no LWC source or associated Jest tests.

## 18. Setup / Deployment

### Prerequisites

- Salesforce CLI installed.
- Access to a Salesforce org that can receive the included metadata.
- Node.js/npm if using the package scripts.

### Deploy to an authorized org

From the project root, authorize the target org and deploy the default package directory:

```bash
sf org login web --alias secms-org
sf project deploy start --source-dir force-app --target-org secms-org
```

The scratch-org definition in `config/project-scratch-def.json` specifies a Developer Edition org named `Demo company` with Lightning Experience enabled. For a scratch org, authenticate to a Dev Hub and run:

```bash
sf org create scratch --definition-file config/project-scratch-def.json --alias secms-scratch --set-default --duration-days 7
sf project deploy start --source-dir force-app --target-org secms-scratch
```

After deployment, confirm the flows and approval process are active, configure or verify user access and email settings in the org, assign the needed permission sets, and verify the app and its reports using representative records. The repository's package scripts can be installed with `npm install`; `npm test` invokes the configured LWC Jest command, but no LWC tests are currently included.

## 19. GitHub Repository

[Student Enrollment & Course Management System on GitHub](https://github.com/jayanee-19/Student-Enrollment-Course-Management-System)
