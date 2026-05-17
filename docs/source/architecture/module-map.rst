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

       VLand --> SWel
       VUDash --> SUDash
       VProf --> SProf
       VProj --> SProj
       VPDash --> SKonva
       VPInfo --> SPInfo
       VPTask --> STask
       VPCal --> SCal
       VPChat --> SChat
       VPCont --> SCont
       VPFiles --> SFiles
       VErr --> SErr

       VLand --> CSSBase
       VPDash --> CSSPage
       PAI --> SAI

       SChat --> SUtil
       STask --> SUtil
       SUDash --> SUtil

       CSSBase --> CSSRoot
       CSSPage --> CSSRoot

       classDef view fill:#e8f0fe,stroke:#1a73e8,color:#000
       classDef partial fill:#fef7e0,stroke:#f9a825,color:#000
       classDef script fill:#e8f5e9,stroke:#2e7d32,color:#000
       classDef cssfile fill:#fce4ec,stroke:#c2185b,color:#000

       class VLand,VWel,VUDash,VProf,VProj,VPDash,VPInfo,VPTask,VPCal,VPChat,VPCont,VPFiles,VErr view
       class PNav,PNot,PAI,PFoot,PCook,PLoad,PFiles partial
       class SUtil,SLD,SCook,SNav,SWel,SUDash,SProf,SProj,SKonva,SPInfo,STask,SCal,SChat,SCont,SFiles,SAI,SErr script
       class CSSRoot,CSSBase,CSSPage cssfile