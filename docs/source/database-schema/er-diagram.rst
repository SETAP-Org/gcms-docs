Entity Relationship Diagram
===========================

The diagram below shows all GCMS database tables and the relationships
between them. Each box represents a table, and each line represents
a foreign key relationship.

.. mermaid::

   erDiagram
       users ||--o{ user_projects : "joins"
       projects ||--o{ user_projects : "has members"

       users ||--o{ projects : "leads"
       users ||--o{ projects : "creates"

       projects ||--o{ tasks : "contains"
       users ||--o{ tasks : "assigned to"

       projects ||--o{ messages : "has"
       users ||--o{ messages : "sends"

       projects ||--o{ meetings : "schedules"
       users ||--o{ meetings : "leads"

       meetings ||--o{ meeting_attendances : "tracks"
       users ||--o{ meeting_attendances : "attends"

       projects ||--o{ widgets : "displays"

       users ||--o{ notifications : "receives"
       projects ||--o{ notifications : "triggers"

       users {
           uuid user_id PK
           varchar user_first_name
           varchar user_last_name
           varchar user_email UK
           text microsoft_id UK
           varchar username UK
           timestamptz date_created
           timestamptz last_login
       }

       projects {
           uuid project_id PK
           uuid team_leader_id FK
           uuid created_by FK
           varchar project_name
           date project_deadline
           timestamptz p_date_created
           timestamptz p_time_updated
       }

       user_projects {
           uuid user_id PK_FK
           uuid project_id PK_FK
       }

       tasks {
           uuid task_id PK
           uuid project_id FK
           uuid assignee_id FK
           varchar task_title
           text task_description
           numeric task_weight
           task_status task_status
           date task_deadline
           timestamptz t_date_created
           timestamptz t_time_updated
       }

       messages {
           uuid message_id PK
           uuid sender_id FK
           uuid project_id FK
           text message_content
           timestamptz m_date_sent
       }

       meetings {
           uuid meeting_id PK
           uuid team_leader_id FK
           uuid project_id FK
           timestamptz scheduled_time
           integer meeting_duration
           meeting_location meeting_location
           text meeting_notes
       }

       meeting_attendances {
           uuid attendance_id PK
           uuid user_id FK
           uuid meeting_id FK
           attendance_status attendance_status
           timestamptz check_in_time
       }

       notifications {
           uuid notification_id PK
           uuid user_id FK
           uuid project_id FK
           notification_type notification_type
           text notification_message
           timestamptz n_date_created
           text target_username
           text project_name
       }

       widgets {
           uuid widget_id PK
           uuid project_id FK
           numeric widget_x
           numeric widget_y
           varchar widget_text
       }

Reading the diagram
-------------------

- **PK** — primary key
- **FK** — foreign key
- **UK** — unique constraint
- **PK_FK** — primary key that is also a foreign key (composite key in junction tables)
- ``||--o{`` — one-to-many relationship (one entity on the left, many on the right)