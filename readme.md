1. Create a few projects for your demonstration. We will be building our app in Dev, and promoting it through Test and Prod.

$ export GUID=techdata


$ oc new-project ${GUID}-development --description="Development Environment" --display-name="Development Environment"

$ oc new-project ${GUID}-testing --description="Testing Environment" --display-name="Testing Environment"

$ oc new-project ${GUID}-production --description="Production Environment" --display-name="Production Environment"


$ oc project ${GUID}-development


$ oc new-app jenkins-persistent -p ENABLE_OAUTH=false -e JENKINS_PASSWORD=abc123 -n ${GUID}-development


$ oc policy add-role-to-user edit system:serviceaccount:${GUID}-development:jenkins -n ${GUID}-testing

$ oc policy add-role-to-user edit system:serviceaccount:${GUID}-development:jenkins -n ${GUID}-production


$ oc policy add-role-to-group system:image-puller system:serviceaccounts:${GUID}-testing -n ${GUID}-development

$ oc policy add-role-to-group system:image-puller system:serviceaccounts:${GUID}-production -n ${GUID}-development


$ oc new-app httpd~https://github.com/amri-firdaus/html -n ${GUID}-development


$ oc tag webapp:latest webapp:testready -n ${GUID}-development

$ oc tag webapp:testready webapp:prodready -n ${GUID}-development


$ oc describe imagestream webapp -n ${GUID}-development


$ oc new-app ${GUID}-development/webapp:testready --name=webapp -n ${GUID}-testing

$ oc new-app ${GUID}-development/webapp:prodready --name=webapp -n ${GUID}-production


$ oc expose service webapp -n ${GUID}-development

$ oc expose service webapp -n ${GUID}-testing

$ oc expose service webapp -n ${GUID}-production


$ oc get dc webapp -o yaml -n ${GUID}-development | sed 's/automatic: true/automatic: false/g' | oc replace -f -

$ oc get dc webapp -o yaml -n ${GUID}-testing | sed 's/automatic: true/automatic: false/g' | oc replace -f -

$ oc get dc webapp -o yaml -n ${GUID}-production | sed 's/automatic: true/automatic: false/g' | oc replace -f -

