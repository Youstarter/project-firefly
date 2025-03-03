---
keywords:
  - Adobe I/O
  - Extensibility
  - API Documentation
  - Developer Tooling
---

# App Builder and Adobe I/O Runtime
 
App Builder greatly simplifies the process of building Cloud Native Applications that leverage Adobe I/O Runtime. 

If you have existing actions deployed on Adobe I/O Runtime and you want to migrate them to an App Builder application (maybe a headless application) these are the main steps:
* In the [Developer Console](/console) create a new project and choose App Builder as the template
* Configure the Adobe CLI and then use it to initialize a new application for the project you’ve created in the Developer Console; check [Setting Up Your Environment](../getting_started/index.md)
* Create copies of your existing actions in this project
* Deploy and test the new actions
* Update your applications to point to the new actions
 
## Developer Preview and Production SLA
 
If you are a customer who uses Adobe I/O Runtime in production (with production SLAs) and plan on using App Builder, you might wonder what the App Builder Developer Preview and I/O Runtime production SLA mean. Any I/O Runtime action you deploy as part of your App Builder projects is also covered by the overall production SLA you have for I/O Runtime.

