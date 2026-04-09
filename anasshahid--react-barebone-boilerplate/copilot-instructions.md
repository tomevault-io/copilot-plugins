## react-barebone-boilerplate

> This application follows a modular structure where everything is separated as a feature along with some common feature/configurations defined in different places.


This application follows a modular structure where everything is separated as a feature along with some common feature/configurations defined in different places.

The core project structure is given below: 

- src
  - assets // contains images and other static assets that needs to be compiled
  - components  // contains commmon componets used in the project
    - layout // layout related components that are used throughout the application and in different layouts
    - text-editor // text editor related components (we are using TipTap for creating the WSIWYG editor)
    - ui // core UI components that are powered by ShadCN/UI
    - ...other-components // some more components (customer pllus ShadCN/UI)
  - config // application configuration for different features is defined in individual files inside this directory
  - constants // permissions, sidebar etc. Constants hold the information that needs to be presented in a static format and mostly doesn't change regularly
  - features // this is one of the most important directory. Each feature of the project is housed inside the features directory
  - hooks // common hooks are defined here (including hooks required/added by shadcn/ui)
  - layouts // application layouts for different scenarios are housed in this directory
  - lib // lib houses common libraries/custom libraries for error handling and other featuers
  - locales // application current has two languages and both are configured in this directory. Each feature has its own locale files which are then imported inside the common language specific file in this directory. 
  - pages // pages houses common 404 and other error related pages which are not feature specific
  - providers // currently we only have theme-provider for switching theme in the application 
  - router // all routs for all features are exported from the index.jsx within this directory
  - services // services houses api-slice.js for RTK Query. Each feature has it's own services inside the feature directory but they all converge and export in here.
  - store // store has root-reducer.js and index.js -- index provides the main store and root-reducer provides the common place for merging all the reducers/api slices from different features into one root reducer
  - styles // we have global.css and theme.config.js. both of these files are rarely changed and provide the basis of common styles config
  - utils // currrently we have auth.js which is a utility to fetch auth token from the Clerk authentication.
  - App.css
  - App.jsx
  - il8n.js
  - index.css
  - main.jsx
- index.html
- components.json
- package.json
- README.md


## Sturcure for Features
- src
  - features
    - index.js // this file re-exports all common exports for this feature (e.g., re-exporting components, re-exporting services etc.)
    - components // this directory houses common components that are related to the specific feature itself. Any dialogs, custom input options, any other supported custom components required to finish the UI of this feature goes here. 
        - index.js // index.js inside the components directory, re-exports components from their individual component file
    - locales // locales has two files en.json and es.json. These two files have all the static text strings for this specific feature defined inside for both languages 
    - pages // pages is the containers from the old era boilerplates. A page is created using various components from the feature and is the composition of the feature's page on the UI. 
    - services // services holds index.js which creates RTK Query APIs inside the index.js file. 
    - store // store has slice of the feature for redux store. We use RTK for creating the slices and managing it. There's also selectors.js file in the store directory which creates basic selectors that can be used elsewhere in the application or within the feature itself as well. 
    - routes.jsx // routes.jsx file in the feature directory is where the routes for this feature are housed. We use react-router-dom for routing. Pages correspond directly to the routes and the route only renders a page (never a component from components directory)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnasShahid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AnasShahid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
