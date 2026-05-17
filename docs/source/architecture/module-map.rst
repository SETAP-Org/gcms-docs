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
           PUNav["userNav"]
           PPNav["projectNav"]
           PLNav["landingNav"]
           PAI["aiAssistant"]
           PFoot["footer"]
           PCook["cookieBanner"]
           PLoad["loading"]
       end

       subgraph LocalScripts["Local Client Scripts"]
           SLD["LD-mode.js"]
           SWel["welcome.js"]
           SUDash["userDash.js"]
           SProf["profile.js"]
           SProj["projects.js"]
           SKonva["konva-dash.js"]
           SLoadCal["loadCalandar.js"]
           SPInfo["projectInfo.js"]
           STask["projectTasks.js"]
           SCal["projectCalendar.js"]
           SChat["projectChat.js"]
           SCont["projectContributions.js"]
           SFiles["projectFiles.js"]
           SErr["error.js"]
           SAI["aiAssistant.js"]
           SCook["cookieBanner.js"]
           SUNav["userNav.js"]
           SPNav["projectNav.js"]
           SUtil["utils.js"]
       end

       subgraph External["External JS"]
           SIO["socket.io.js"]
           ChartJS["chart.js"]
           Konva["konva.js"]
           FullCal["fullcalendar.js"]
       end

       VLand --> PLNav
       VLand --> PCook
       VLand --> PFoot
       VLand --> PLoad
       VLand --> SLD

       VWel --> SWel

       VUDash --> PUNav
       VUDash --> PFoot
       VUDash --> PLoad
       VUDash --> SLD
       VUDash --> SUDash

       VProj --> PUNav
       VProj --> PFoot
       VProj --> PLoad
       VProj --> SLD
       VProj --> SProj

       VProf --> PUNav
       VProf --> PFoot
       VProf --> PLoad
       VProf --> SLD
       VProf --> SProf

       VPDash --> PUNav
       VPDash --> PPNav
       VPDash --> PAI
       VPDash --> PFoot
       VPDash --> PLoad
       VPDash --> SLD
       VPDash --> SKonva
       VPDash --> SLoadCal
       VPDash --> Konva
       VPDash --> FullCal

       VPInfo --> PUNav
       VPInfo --> PPNav
       VPInfo --> PAI
       VPInfo --> PFoot
       VPInfo --> PLoad
       VPInfo --> SLD
       VPInfo --> SPInfo
       VPInfo --> SIO

       VPTask --> PUNav
       VPTask --> PPNav
       VPTask --> PAI
       VPTask --> PFoot
       VPTask --> PLoad
       VPTask --> SLD
       VPTask --> STask
       VPTask --> SIO

       VPCal --> PUNav
       VPCal --> PPNav
       VPCal --> PAI
       VPCal --> PFoot
       VPCal --> PLoad
       VPCal --> SLD
       VPCal --> SCal
       VPCal --> SIO

       VPChat --> PUNav
       VPChat --> PPNav
       VPChat --> PAI
       VPChat --> PFoot
       VPChat --> PLoad
       VPChat --> SLD
       VPChat --> SChat
       VPChat --> SIO

       VPCont --> PUNav
       VPCont --> PPNav
       VPCont --> PAI
       VPCont --> PFoot
       VPCont --> PLoad
       VPCont --> SLD
       VPCont --> SCont
       VPCont --> ChartJS

       VPFiles --> PUNav
       VPFiles --> PPNav
       VPFiles --> PAI
       VPFiles --> PLoad
       VPFiles --> SLD
       VPFiles --> SFiles

       VErr --> PLoad
       VErr --> SLD
       VErr --> SErr

       PAI --> SAI
       PCook --> SCook
       PUNav --> SUNav
       PPNav --> SPNav

       SChat --> SUtil
       STask --> SUtil
       SUDash --> SUtil
       SProj --> SUtil
       SPInfo --> SUtil
       SCal --> SUtil
       SCont --> SUtil
       SFiles --> SUtil
       SAI --> SUtil
       SKonva --> SUtil

       classDef view fill:#e8f0fe,stroke:#1a73e8,color:#000
       classDef partial fill:#fef7e0,stroke:#f9a825,color:#000
       classDef script fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef ext fill:#ffebee,stroke:#c62828,color:#000

       class VLand,VWel,VUDash,VProf,VProj,VPDash,VPInfo,VPTask,VPCal,VPChat,VPCont,VPFiles,VErr view
       class PUNav,PPNav,PLNav,PAI,PFoot,PCook,PLoad partial
       class SLD,SWel,SUDash,SProf,SProj,SKonva,SLoadCal,SPInfo,STask,SCal,SChat,SCont,SFiles,SErr,SAI,SCook,SUNav,SPNav,SUtil script
       class SIO,ChartJS,Konva,FullCal ext