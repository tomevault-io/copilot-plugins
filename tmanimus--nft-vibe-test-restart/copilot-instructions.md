## matthew-berman-rules

> Coding with multiple files

# Coding pattern preferences


– Always prefer simple solutions  

– Avoid duplication of code whenever possible, which means checking for other areas of the codebase that might already have similar code and functionality  

– You are careful to only make changes that are requested or you are confident are well understood and related to the change being requested  

– When fixing an issue or bug, do not introduce a new pattern or technology without first exhausting all options for the existing implementation. And if you finally do this, make sure to remove the old implementation afterwards so we don’t have duplicate logic.  

– Keep the codebase very clean and organized  

– Avoid writing scripts in files if possible, especially if the script is likely only to be run once  

– Avoid having files over 200–300 lines of code. Refactor at that point.  

– Never overwrite my .env file without first asking and confirming

if the project uses dev, test and prod environments:
    – Mocking data is only needed for tests, never mock data for dev or prod  
    – Write code that takes into account the different environments: dev, test, and prod  
    – Never add stubbing or fake data patterns to code that affects the dev or prod environments  




# Coding pattern preferences


– Always prefer simple solutions  

– Avoid duplication of code whenever possible, which means checking for other areas of the codebase that might already have similar code and functionality  

– You are careful to only make changes that are requested or you are confident are well understood and related to the change being requested  

– When fixing an issue or bug, do not introduce a new pattern or technology without first exhausting all options for the existing implementation. And if you finally do this, make sure to remove the old implementation afterwards so we don’t have duplicate logic.  

– Keep the codebase very clean and organized  

– Avoid writing scripts in files if possible, especially if the script is likely only to be run once  

– Avoid having files over 200–300 lines of code. Refactor at that point.  

– Never overwrite my .env file without first asking and confirming

if the project uses dev, test and prod environments:
    – Mocking data is only needed for tests, never mock data for dev or prod  
    – Write code that takes into account the different environments: dev, test, and prod  
    – Never add stubbing or fake data patterns to code that affects the dev or prod environments  

---
> Source: [TMAnimus/NFT-Vibe-test-restart](https://github.com/TMAnimus/NFT-Vibe-test-restart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
