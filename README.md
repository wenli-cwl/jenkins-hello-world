# Jenkins Hello World

This repository contains a simple `index.html` page and a Jenkins pipeline
for building and deploying the page to IIS. The deployment port and site
name are chosen based on the selected environment (`dev`, `qa`, `uat` or
`prod`). The values for each environment are read from Jenkins
environment variables (for example `DEV_DEPLOY_PORT`, `DEV_SITE_NAME`,
`QA_DEPLOY_PORT`, `QA_SITE_NAME`, and so on).

The pipeline demonstrates:

* Archiving build files.
* Using environment variables and Jenkins credentials.
* Deploying to IIS after manual approval using the environment-specific port
  and site name.
* Publishing a dummy package step to GitHub Packages.
* Running a sample JUnit test report.
* Sending an email notification after deployment.
