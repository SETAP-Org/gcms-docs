Module Map
==========

This page contains three diagrams that together provide a complete
view of the GCMS codebase. They are intentionally split because
showing everything in a single diagram becomes unreadable.

- **Front-end architecture** — how EJS views, client-side JS, and
  stylesheets are organised
- **Backend module map** — every route, controller, model, and
  utility, with key function names on the connections
- **External integration flows** — how external services connect
  to the codebase

Front-end architecture
----------------------

GCMS renders pages server-side with EJS, then progressively enhances
each page with a dedicated client-side script. Every page has its
own EJS view, JS file, and CSS file, plus shared partials and
utilities.

.. mermaid::

   flowchart LR
       subgraph Pages["EJS Pages"]
           VLand["landing.ejs"]
           VWel["welcome.ejs"]
           VUDash["userDash.ejs"]
           VProf["profile.ejs"]
           VProj["projects.ejs"]
           VPDash["projectDash.ejs"]
           VPInfo["projectInfo.ejs"]
           VPTask["projectTasks.ejs"]
           VPCal["projectCalendar.ejs"]
           VPChat["projectChat.ejs"]
           VPCont["projectContributions.ejs"]
           VPFiles["projectFiles.ejs"]
           VErr["error.ejs"]
       end

       subgraph Partials["Shared Partials"]
           PNav["userNav, projectNav, landingNav"]
           PNot["notifications"]
           PAI["aiAssistant"]
           PFoot["footer"]
           PCook["cookieBanner"]
           PLoad["loading"]
           PFiles["projectFiles"]
       end

       subgraph Scripts["Client Scripts"]
           SUtil["utils.js"]
           SLD["LD-mode.js"]
           SCook["cookieBanner.js"]
           SNav["userNav, projectNav"]
           SWel["welcome.js"]
           SUDash["userDash.js"]
           SProf["profile.js"]
           SProj["projects.js"]
           SKonva["konva-dash.js"]
           SPInfo["projectInfo.js"]
           STask["projectTasks.js"]
           SCal["projectCalendar.js, loadCalandar.js"]
           SChat["projectChat.js"]
           SCont["projectContributions.js"]
           SFiles["projectFiles.js"]
           SAI["aiAssistant.js"]
           SErr["error.js"]
       end

       subgraph Styles["Stylesheets"]
           CSSRoot["root-light.css, root-dark.css"]
           CSSBase["styles.css, header_style.css, footer.css"]
           CSSPage["Per-page CSS files"]
       end

       Pages -.embeds.-> Partials
       Pages -->|loads| Scripts
       Pages -->|links| Styles

       VPDash --> SKonva
       VPCal --> SCal
       VPChat --> SChat
       VPFiles --> SFiles
       PAI --> SAI

       Scripts -.uses.-> SUtil

       CSSBase -.uses tokens.-> CSSRoot
       CSSPage -.uses tokens.-> CSSRoot

       classDef view fill:#e8f0fe,stroke:#1a73e8,color:#000
       classDef partial fill:#fef7e0,stroke:#f9a825,color:#000
       classDef script fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef cssfile fill:#fce4ec,stroke:#c2185b,color:#000

       class VLand,VWel,VUDash,VProf,VProj,VPDash,VPInfo,VPTask,VPCal,VPChat,VPCont,VPFiles,VErr view
       class PNav,PNot,PAI,PFoot,PCook,PLoad,PFiles partial
       class SUtil,SLD,SCook,SNav,SWel,SUDash,SProf,SProj,SKonva,SPInfo,STask,SCal,SChat,SCont,SFiles,SAI,SErr script
       class CSSRoot,CSSBase,CSSPage cssfile

Backend module map
------------------

The backend follows a strict layered pattern. Each route file maps
to a controller, which calls one or more models. Arrows on the
diagram are labelled with the most representative function names
on each connection.

.. mermaid::

   flowchart TB
       subgraph Entry["Entry Points"]
           Listen["listen.js"]
           ServerJS["server.js"]
           AppJS["app.js"]
           Socket["utils/socket.js"]
       end

       subgraph Routes["Routes"]
           Rpage["pageRoutes"]
           Rauth["authRoutes"]
           Ruser["userRoutes"]
           Rproj["projectRoutes"]
           Rtask["taskRoutes"]
           Rcal["calendarRoutes"]
           Rnotif["notificationsRoutes"]
           Rcontrib["contributionsRoutes"]
           Rnote["noteRoutes"]
           Rfile["fileRoutes"]
           Rai["aiRoutes"]
       end

       subgraph Controllers["Controllers"]
           Cserve["serveControllers<br/>serveLanding, serveProjectDash,<br/>serveProjectTasks, ..."]
           Cauth["authControllers<br/>checkIfLoggedIn, signOut,<br/>authenticatePassport"]
           Cuser["userControllers<br/>addUser, updateUsername,<br/>getCurrentUserPhoto"]
           Cproj["projectControllers<br/>addProject, getUserProjects,<br/>updateTeamLeader, checkMembership"]
           Cuproj["userProjectControllers<br/>addUserToProject,<br/>removeUserFromProject"]
           Ctask["taskControllers<br/>addTask, getProjectTasks,<br/>updateTaskStatus, deleteTask"]
           Ccal["calendarControllers<br/>getEvent, addEvent, removeEvent"]
           Cnotif["notificationControllers<br/>fetchNotificationsByUserId,<br/>removeNotification"]
           Ccontrib["contributionControllers<br/>getProjectContributions"]
           Ckonva["konvaControllers<br/>addNote, updateNote,<br/>removeNote, getNotes"]
           Cfile["fileControllers<br/>initFileUpload, addFileMetadata,<br/>getDownloadUrl, deleteFile"]
           Cai["aiChatControllers<br/>postAiChatMessage,<br/>getAiChatMessages, clearAiChatHistory"]
       end

       subgraph Models["Models"]
           Muser["userModels"]
           Mproj["projectModels"]
           Muproj["userProjectModels"]
           Mtask["taskModels"]
           Mchat["chatModels"]
           Mcal["calendarModels"]
           Mmeet["meetingModels"]
           Mnotif["notificationModels"]
           Mcontrib["contributionModels"]
           Mkonva["konvaModels"]
           Mfile["fileModels"]
           Mai["aiChatModels"]
       end

       Listen --> ServerJS
       ServerJS --> AppJS
       ServerJS --> Socket

       AppJS --> Rpage
       AppJS --> Rauth
       AppJS --> Ruser
       AppJS --> Rproj
       AppJS --> Rtask
       AppJS --> Rcal
       AppJS --> Rnotif
       AppJS --> Rcontrib
       AppJS --> Rnote
       AppJS --> Rfile
       AppJS --> Rai

       Rpage --> Cserve
       Rauth --> Cauth
       Ruser --> Cuser
       Rproj --> Cproj
       Rproj --> Cuproj
       Rtask --> Ctask
       Rcal --> Ccal
       Rnotif --> Cnotif
       Rcontrib --> Ccontrib
       Rnote --> Ckonva
       Rfile --> Cfile
       Rai --> Cai

       Socket -->|chat send| Cnotif
       Socket -->|task update| Ctask
       Socket -->|widget update| Ckonva

       Cserve --> Mproj
       Cserve --> Muser
       Cauth --> Muser
       Cuser --> Muser
       Cproj --> Mproj
       Cproj --> Muproj
       Cuproj --> Muproj
       Ctask --> Mtask
       Ccal --> Mcal
       Ccal --> Mmeet
       Cnotif --> Mnotif
       Cnotif --> Mchat
       Ccontrib --> Mcontrib
       Ckonva --> Mkonva
       Cfile --> Mfile
       Cai --> Mai

       classDef entry fill:#fff3e0,stroke:#f57c00,color:#000
       classDef route fill:#f3e5f5,stroke:#7b1fa2,color:#000
       classDef ctrl fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef model fill:#fff9c4,stroke:#f9a825,color:#000

       class Listen,ServerJS,AppJS,Socket entry
       class Rpage,Rauth,Ruser,Rproj,Rtask,Rcal,Rnotif,Rcontrib,Rnote,Rfile,Rai route
       class Cserve,Cauth,Cuser,Cproj,Cuproj,Ctask,Ccal,Cnotif,Ccontrib,Ckonva,Cfile,Cai ctrl
       class Muser,Mproj,Muproj,Mtask,Mchat,Mcal,Mmeet,Mnotif,Mcontrib,Mkonva,Mfile,Mai model

External integration flows
--------------------------

GCMS connects to four external services. Each integration is
accessed through a dedicated utility module, with specific
controllers calling them.

.. mermaid::

   flowchart LR
       subgraph Controllers["Controllers"]
           Cauth["authControllers"]
           Ccal["calendarControllers"]
           Cuser["userControllers"]
           Cai["aiChatControllers"]
           Cfile["fileControllers"]
           Cnotif["notificationControllers"]
           Cother["other controllers"]
       end

       subgraph Utils["Utilities"]
           Uauth["auth.js (Passport strategy)"]
           Usup["supabase.js (pg pool + storage client)"]
           Ugem["gemini.js (AI client)"]
           Ufile["fileFetcher.js (download + extract text)"]
           Uemail["emailSender.js (Nodemailer)"]
           Uconf["emailConfig.js (SMTP config)"]
       end

       subgraph DB["Data Stores"]
           PG[("PostgreSQL via pg pool")]
           Bucket[("Supabase Storage Bucket")]
       end

       subgraph External["External Services"]
           MS["Microsoft Graph API"]
           Gem["Google Gemini API"]
           Mail["Mailtrap SMTP"]
       end

       Cauth -->|authenticatePassport| Uauth
       Uauth -->|OAuth + me| MS

       Ccal -->|getEvent, addEvent, removeEvent| MS
       Cuser -->|getCurrentUserPhoto| MS

       Cother -.via models.-> Usup
       Usup -->|SQL queries| PG

       Cfile -->|initFileUpload, getDownloadUrl, deleteFile| Usup
       Usup -->|signed URLs, storage ops| Bucket

       Cai -->|postAiChatMessage| Ugem
       Cai -->|fetch attached files| Ufile
       Ufile --> Usup
       Ugem -->|generateContent| Gem

       Cnotif -->|send notification| Uemail
       Uemail --> Uconf
       Uconf -->|SMTP| Mail

       classDef ctrl fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef util fill:#e0f2f1,stroke:#00796b,color:#000
       classDef db fill:#ede7f6,stroke:#512da8,color:#000
       classDef ext fill:#ffebee,stroke:#c62828,color:#000

       class Cauth,Ccal,Cuser,Cai,Cfile,Cnotif,Cother ctrl
       class Uauth,Usup,Ugem,Ufile,Uemail,Uconf util
       class PG,Bucket db
       class MS,Gem,Mail ext

How to read these diagrams
--------------------------

- **Function names in node labels** indicate which exported
  functions live in that module — not always an exhaustive list,
  but the most representative ones
- **Solid arrows** are direct function calls or HTTP/socket transport
- **Dashed arrows** are "uses" or "embeds" relationships rather than
  direct invocation
- **Cylinders** are persistent data stores
- The grouping into subgraphs reflects the actual folder structure
  of the repository