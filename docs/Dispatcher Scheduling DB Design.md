---
linktitle: "Dispatcher Schedule DB ER-Design"
categories: ["Design"]
status: "approved"
weight: 2
toc_hide: false
hide_summary: false
description: >
  ER Model for Database used to schedule tasks in Dispatcher service of Intel Inband Manageability
---

# Dispatcher Schedule Database Design

## Overview

The database to be used will be SQLite3.  The database will be used to keep track of scheduled tasks as either a single task at a specified time or a repeated task set up as a cron job.  

The goal of this design is to have a database design that adheres to the principles of Normal Form.

NOTE: If the Dispatcher receives a new scheduled manifest, the contents of the APSheduler and Database will be removed and replaced with the entire contents of the received manifest.  Any schedules that were in the previous manifest and not in the new one will no longer be scheduled.

## ER Model

```mermaid
---

title: Dispatcher Schedule ER Model

---

erDiagram
    SINGLE_SCHEDULE ||--|{ SINGLE_SCHEDULE_JOB : schedules
    SINGLE_SCHEDULE_JOB {
        INTEGER priority "Order the job manifests should run - Starting with 0"
        INTEGER schedule_id PK,FK "REFERENCES SINGLE_SCHEDULE(schedule_id)"
        INTEGER task_id PK,FK "REFERENCES job(task_id)"
        TEXT status "NULL or scheduled"
    }

    IMMEDIATE_SCHEDULE {
        INTEGER id PK "AUTOINCREMENT"
        TEXT request_id 
    }

    SINGLE_SCHEDULE {
        INTEGER id PK "AUTOINCREMENT"
        TEXT request_id 
        TEXT start_time "NOT NULL - Format -> 2024-01-01T00:00:00"
        TEXT end_time "NOT NULL - Format -> 2024-01-01T00:00:00"
    }
    JOB ||--o{ SINGLE_SCHEDULE_JOB: performs
    JOB ||--o{ REPEATED_SCHEDULE_JOB : performs
    JOB ||--o{ IMMEDIATE_SCHEDULE_JOB :performs

    JOB {
        INTEGER task_id PK "AUTOINCREMENT"
        TEXT job_id "FROM MJUNCT"
        TEXT manifest "NOT NULL"
    }  
    
    IMMEDIATE_SCHEDULE ||--|{ IMMEDIATE_SCHEDULE_JOB : schedules
    IMMEDIATE_SCHEDULE_JOB {
        INTEGER priority "Order the job manifests should run - Starting with 0"
        INTEGER schedule_id PK,FK "REFERENCES IMMEDIATE_SCHEDULE(schedule_id)"
        INTEGER task_id PK,FK "REFERENCES job(task_id)"
        TEXT status "NULL or scheduled"
    }

    REPEATED_SCHEDULE ||--|{ REPEATED_SCHEDULE_JOB : schedules
    REPEATED_SCHEDULE_JOB {
        INTEGER priority "Order the job manifests should run"
        INTEGER schedule_id PK,FK "REFERENCES REPEATED_SCHEDULE(schedule_id)"
        INTEGER task_id PK,FK "REFERENCES job(task_id)"
        TEXT status "NULL or scheduled"
    }

    REPEATED_SCHEDULE {
        INTEGER id PK "AUTOINCREMENT"
        TEXT request_id "NOT NULL"
        TEXT cron_duration "NOT NULL"
        TEXT cron_minutes "NOT NULL"
        TEXT cron_hours "NOT NULL"
        TEXT cron_day_month "NOT NULL"
        TEXT cron_month "NOT NULL"
        TEXT cron_day_week "NOT NULL"
    }
    
```

## DB Table Examples

### JOB Table

| task_id | job_id | manifest |
| :---- | :---- | :----- |
| 1 | swupd-fc276d74-014f-44c2-a57c-de1944a6e974 | valid Inband Manageability XML manifest - SOTA download only |
| 2 | swupd-09273593-ff7b-433f-820c-86440b6f8cf3 | valid Inband Manageability XML manifest - SOTA install only|
| 3 | setpwr-70ab8502-2c53-4417-ae57-90e4c2b7f0b6 | valid Inband Manageability XML manifest - Reboot system |
| 4 | fwupd-718814f3-b12a-432e-ac38-093e8dcb4bd1 | valid Inband Manageability XML manifest - FOTA |
| 5 | setpwr-d8be8ae4-7512-43c0-9bdd-9a066de17322 | valid Inband Manageability XML manifest - Reboot system |

### IMMEDIATE_SCHEDULE Table

| id | request_id |
| :---- | :---- |
| 1  | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 |
| 2  | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 |
| 3  | c9b74125-f3bb-440a-ad80-8d02090bd337 |

### SINGLE_SCHEDULE Table

| id | request_id | start_time | end_time |
| :---- | :---- | :---- | :---- |
| 1  | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 | 2024-04-01T08:00:00 | 2024-04-01T12:00:00 |
| 2  | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 | 2024-05-01T08:00:00 | 2024-05-01T12:00:00|
| 3  | c9b74125-f3bb-440a-ad80-8d02090bd337 | 2024-04-02T08:00:00 | 2024-04-02T14:00:00|

### REPEATED_SCHEDULE Table

NOTE: These values may not make sense in the real world.  Just for demonstration purposes.

| id | request_id | cron_duration | cron_minutes | cron_hours | cron_day_month | cron_month |cron_day_week |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 | P1D | 0 | */3 | * | * | * |
| 2 | 6bf587ac-1d70-4e21-9a15-097f6292b9c4 | P7D | 0 | */6 | * | * | * |
| 3 | c9b74125-f3bb-440a-ad80-8d02090bd337 | P2D | 0 | */8 | * | * | * |
| 4 | c9b74125-f3bb-440a-ad80-8d02090bd337 | P14D | 0 | * | * | * | * |

### IMMEDIATE_SCHEDULE_JOB Table

Example: To do a download, install, and reboot of SOTA at the time in schedule 1

| priority | schedule_id | task_id | status |
| :---- | :---- | :---- | :----- |
| 0   | 1 | 1 | scheduled |
| 1   | 1 | 2 | scheduled |
| 2   | 1 | 3 |   |

### SINGLE_SCHEDULE_JOB Table

Example: To do a download, install, and reboot of SOTA at the time in schedule 1

| priority | schedule_id | task_id | status |
| :---- | :---- | :---- | :----- |
| 0   | 1 | 1 | scheduled |
| 1   | 1 | 2 | scheduled |
| 2   | 1 | 3 |   |

### REPEATED_SCHEDULE_JOB Table

Example: To do a download, install, and reboot of SOTA at the repeated time in schedule 2

| priority | schedule_id | task_id | status |
| :---- | :---- | :---- | :----- |
| 0   | 2 | 4 | scheduled |
| 1   | 2 | 5 | scheduled |

## Field Descriptions

| Name  | Description | Required? |
| :--- | :--- | :---|
| cron_duration |  period between successive executions of a scheduled task defined by a cron job | Yes |
| cron_minutes | field in a cron job schedule that specifies the minute of the hour at which the task should run, ranging from 0 to 59  | Yes |
| cron_hours | field in a cron job schedule that specifies the hour of the day at which the task should execute, using a 24-hour format ranging from 0 to 23 | Yes |
| cron_day_month | field in a cron job schedule that specifies the day of the month on which the task should run, ranging from 1 to 31.   | Yes  |
| cron_month |  field in a cron job schedule that specifies the month during which the task should execute, ranging from 1 (January) to 12 (December).   | Yes |
| cron_day_week | field in a cron job schedule that specifies the day of the week on which the task should run, ranging from 0 (Sunday) to 6 (Saturday).     | Yes |
| end_time | ending date/time for a single scheduled request. | No |
| id    | Auto-generated by DB.  The request Ids will not be unique in the tables to use as a PK. | Yes |
| job_id | ID for each job assigned by MJunct.  It will have an abbreviation of the job type in front of a UUID.  All manifests in the same schedule will have the same jobId. | Yes |
| manifest | valid Inband Manageability manifest | Yes |
| request_id | Request ID generated by MJunct that is used to trace the request.  Every schedule and job in the manifest will have the same requestId. | Yes |
| start_time | starting date/time for a single scheduled request.| Yes |
| status | indicates if the request has been scheduled.  Not scheduled unless 'scheduled' is indicated in the field.  This is used to ensure the same manifest is not ran after a reboot | No |
| task_id | Autoincremented number in the JOB table to store jobs and their manifests.  Used to differentiate when a schedule has multiple manifests.  They would all have the same requestId and JobId, so that taskId ensures the DB conforms to Normal Form.   | Yes |
