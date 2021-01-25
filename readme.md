1. Create a few projects for your demonstration. We will be building our app in Dev, and promoting it through Test and Prod.

$ export GUID=env

$ oc new-project ${GUID}-development --description="Development Environment" --display-name="Development Environment"

$ oc new-project ${GUID}-testing --description="Testing Environment" --display-name="Testing Environment"

$ oc new-project ${GUID}-production --description="Production Environment" --display-name="Production Environment"

2. Switch to the pipeline-${GUID}-dev project, which is where most of the work is to be done:

$ oc project ${GUID}-development

3. Deploy Jenkins to control your builds and deployment pipeline:

$ oc new-app jenkins-persistent -p ENABLE_OAUTH=false -e JENKINS_PASSWORD=abc123 -n ${GUID}-development

4. The above command created a Jenkins deployment and service accounts in the Dev project. Since we want to be able to use Jenkins to run images in the Test and Prod projects, we need to give the Jenkins pservice account authorization to manage resources in those projects. To do so, use oc policy to enable the Jenkins service account to manage resources in the pipeline-${GUID}-test and pipeline-${GUID}-prod projects:

$ oc policy add-role-to-user edit system:serviceaccount:${GUID}-development:jenkins -n ${GUID}-testing

$ oc policy add-role-to-user edit system:serviceaccount:${GUID}-development:jenkins -n ${GUID}-production

5. When an application is deployed in a project, the service accounts in the project do the work. Since the sample application’s image will be owned by the Dev project, it is not by default accessible to the Test and Prod projects. The service accounts in the Test and Prod projects need to be able to pull images in the Dev projects. Use oc policy to enable pulling of images from the pipeline-${GUID}-dev project to the pipeline-${GUID}-test and pipeline-${GUID}-prod projects:

$ oc policy add-role-to-group system:image-puller system:serviceaccounts:${GUID}-testing -n ${GUID}-development

$ oc policy add-role-to-group system:image-puller system:serviceaccounts:${GUID}-production -n ${GUID}-development

6. Deploy the sample application in the pipeline-${GUID}-dev project:

$ oc new-app httpd~https://github.com/amri-firdaus/html -n ${GUID}-development

7. Verify that the build completed and tag the image. First, we tag the latest build of cotd2 with the tag testready. Then we tag that same image with prodready:

$ oc tag webapp:latest webapp:testready -n ${GUID}-development

$ oc tag webapp:testready webapp:prodready -n ${GUID}-development

$ oc describe imagestream webapp -n ${GUID}-development

8. Imagine that we have successfully smoke tested the application in Dev, and that we will now promote it to Test. Imagine futher that the image is tested in the Test project, and is promoted to Prod. Deploy the cotd2 application in the pipeline-${GUID}-test and pipeline-${GUID}-prod projects:

$ oc new-app ${GUID}-development/webapp:testready --name=webapp -n ${GUID}-testing

$ oc new-app ${GUID}-development/webapp:prodready --name=webapp -n ${GUID}-production

9. Create routes for all three of the applications:

$ oc expose service webapp -n ${GUID}-development

$ oc expose service webapp -n ${GUID}-testing

$ oc expose service webapp -n ${GUID}-production

10. Disable automatic deployment for all of the deployment configurations in your demonstration:

$ oc get dc webapp -o yaml -n ${GUID}-development | sed 's/automatic: true/automatic: false/g' | oc replace -f -

$ oc get dc webapp -o yaml -n ${GUID}-testing | sed 's/automatic: true/automatic: false/g' | oc replace -f -

$ oc get dc webapp -o yaml -n ${GUID}-production | sed 's/automatic: true/automatic: false/g' | oc replace -f -

11. Use either of the following methods to create the initial BuildConfig pipeline:

Method 1: Use the command line from any host with the OpenShift client:

Create a file with the contents of the BuildConfig pipeline.

Use the oc create -f FILENAME.yaml command to create the BuildConfig pipeline.

Method 2: Use the OpenShift Container Platform web console.

Log in to the OPENTLC shared OpenShift Container Platform web console.

Select your pipeline-${GUID}-dev project.

On the left menu, select the Developer perspective from the dropdown list.

Select Add → Import YAML, paste the BuildConfig pipeline text in the text box, and then click Create.
