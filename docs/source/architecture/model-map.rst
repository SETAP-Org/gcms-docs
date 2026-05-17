Module Map
==========

The diagram below shows every significant module in the GCMS codebase
and how they connect. It complements the high-level overview on the
:doc:`index` page by mapping each file you'll find in the repository
to its position in the layered architecture.

Colours indicate the architectural layer each module belongs to:

- **Blue** — client-side (browser)
- **Orange** — entry points (boot files at repo root)
- **Pink** — real-time layer (Socket.io)
- **Purple** — routes
- **Green** — controllers
- **Yellow** — models
- **Teal** — utilities
- **Lavender** — data stores
- **Red** — external services

.. mermaid::

   flowchart TB
       subgraph Client["Client (Browser)"]
           UI["EJS Views<br/>(views/)"]
           CJS["Client JS<br/>(public/scripts/)"]
           CSS["Stylesheets<br/>(public/styles/)"]
           SocketClient["Socket.io Client"]
       end

       subgraph Boot["Entry Points"]
           Listen["listen.js<br/>(starts HTTP server)"]
           ServerJS["server.js<br/>(HTTP + Socket.io)"]
           AppJS["app.js<br/>(Express app)"]
       end

       subgraph RT["Real-time Layer"]
           SocketServer["Socket.io Server<br/>(utils/socket.js)"]
       end

       subgraph Routes["Routes (routes/)"]
           PR["pageRoutes"]
           AR["authRoutes"]
           UR["userRoutes"]
           PJR["projectRoutes"]
           TR["taskRoutes"]
           CR["calendarRoutes"]
           NR["notificationsRoutes"]
           CTR["contributionsRoutes"]
           NoR["noteRoutes"]
           FR["fileRoutes"]
           AIR["aiRoutes"]
       end

       subgraph Controllers["Controllers (controllers/)"]
           SC["serveControllers"]
           AC["authControllers"]
           UC["userControllers"]
           PC["projectControllers"]
           UPC["userProjectControllers"]
           TC["taskControllers"]
           CC["calendarControllers"]
           NC["notificationControllers"]
           CTC["contributionControllers"]
           KC["konvaControllers"]
           FC["fileControllers"]
           AICC["aiChatControllers"]
       end

       subgraph Models["Models (models/)"]
           UM["userModels"]
           PM["projectModels"]
           UPM["userProjectModels"]
           TM["taskModels"]
           ChM["chatModels"]
           CalM["calendarModels"]
           MeetM["meetingModels"]
           NoM["notificationModels"]
           CtM["contributionModels"]
           KM["konvaModels"]
           FM["fileModels"]
           AIM["aiChatModels"]
       end

       subgraph Utils["Utilities (utils/)"]
           AuthU["auth.js<br/>(Passport)"]
           SessU["session.js"]
           SupU["supabase.js<br/>(pg pool + client)"]
           GemU["gemini.js"]
           FileU["fileFetcher.js"]
           EmailU["emailSender.js"]
           EConfU["emailConfig.js"]
       end

       subgraph DB["Data Layer"]
           PGPool[("PostgreSQL<br/>via pg pool")]
           Bucket[("Supabase<br/>Storage Bucket")]
       end

       subgraph External["External Services"]
           MS["Microsoft<br/>Graph API"]
           Gem["Google Gemini"]
           Mail["Mailtrap SMTP"]
       end

       UI -->|HTTP| Listen
       CJS -->|fetch| Listen
       SocketClient <-->|WebSocket| Listen

       Listen --> ServerJS
       ServerJS --> AppJS
       ServerJS --> SocketServer

       AppJS --> Routes
       Routes --> Controllers

       SocketServer --> Controllers

       Controllers --> Models
       Controllers --> Utils

       Models --> SupU
       SupU --> PGPool

       FC --> SupU
       SupU --> Bucket

       AuthU --> MS
       CC --> MS
       FileU --> MS

       AICC --> GemU
       GemU --> Gem
       AICC --> FileU
       FileU --> SupU

       EmailU --> EConfU
       EConfU --> Mail
       NC --> EmailU

       AppJS -.uses.-> AuthU
       AppJS -.uses.-> SessU

       classDef clientNode fill:#e8f0fe,stroke:#1a73e8,color:#000
       classDef bootNode fill:#fff3e0,stroke:#f57c00,color:#000
       classDef rtNode fill:#fce4ec,stroke:#c2185b,color:#000
       classDef routeNode fill:#f3e5f5,stroke:#7b1fa2,color:#000
       classDef ctrlNode fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef modelNode fill:#fff9c4,stroke:#f9a825,color:#000
       classDef utilNode fill:#e0f2f1,stroke:#00796b,color:#000
       classDef dbNode fill:#ede7f6,stroke:#512da8,color:#000
       classDef extNode fill:#ffebee,stroke:#c62828,color:#000

       class UI,CJS,CSS,SocketClient clientNode
       class Listen,ServerJS,AppJS bootNode
       class SocketServer rtNode
       class PR,AR,UR,PJR,TR,CR,NR,CTR,NoR,FR,AIR routeNode
       class SC,AC,UC,PC,UPC,TC,CC,NC,CTC,KC,FC,AICC ctrlNode
       class UM,PM,UPM,TM,ChM,CalM,MeetM,NoM,CtM,KM,FM,AIM modelNode
       class AuthU,SessU,SupU,GemU,FileU,EmailU,EConfU utilNode
       class PGPool,Bucket dbNode
       class MS,Gem,Mail extNode

How to read this diagram
------------------------

- **Solid arrows** indicate a direct function call or HTTP/socket
  transport
- **Dashed arrows** indicate "uses" relationships where one module
  configures another (e.g. ``app.js`` uses ``auth.js`` to set up
  Passport, but doesn't call it on every request)
- **Cylinders** represent persistent data stores
- The diagram shows the layered structure of the codebase, not the
  request flow. For request flow, see the sequence on the
  :doc:`index` page

Notable patterns
----------------

A few things to notice in the diagram:

- **Two paths into controllers** — Express routes for HTTP requests,
  and the Socket.io server for real-time events. Both end up calling
  the same controller layer.
- **Microsoft Graph appears in three places** — authentication
  (``auth.js``), calendar sync (``calendarControllers``), and file
  fetching for AI context (``fileFetcher.js``).
- **Two data stores** — the PostgreSQL pool for relational data and
  the Supabase Storage bucket for file uploads. Both are reached
  through ``utils/supabase.js``.
- **Notification fan-out** — when a notification is created, the
  controller writes to the database via the model AND emits an email
  via the email utility. Both are essential to the user-facing flow.