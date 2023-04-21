# Prow setup
1. [Source](https://docs.prow.k8s.io/docs/getting-started-deploy/).
First, you need to create a GitHub app. GitHub itself documents this. Initially, it is sufficient to set a dummy url for the Webhook. Below is a minimum set of permissions needed. 
Repository permissions:
* Actions: Read-Only (Only needed when using the merge automation tide)
* Administration: Read-Only (Required to fetch teams and collaborateurs)
* Checks: Read-Only (Only needed when using the merge automation tide)
* Contents: Read (Read & write needed when using the merge automation tide)
* Issues: Read & write
* Metadata: Read-Only
* Pull Requests: Read & write
* Commit statuses: Read & write
Organization permissions:
* Members: Read-Only (Read & write when using peribolos)
In Subscribe to events select all events.
After you saved the app, click “Generate Private Key” on the bottom and save the private key together with the App ID in the top of the page.

2. [Source](https://medium.com/@Devopscontinens/deploying-prow-in-minikube-626a03c43b15)
We also need to generate the hmac-token to give to GitHub for validating webhooks.
`$ openssl rand -hex 20 > hmac-token`
View private key to copy it to manifests further:
`$ cat << private key file >>`

Clone the Prow repository to local
`$ git clone https://github.com/kubernetes/test-infra.git`

`$ kubectl create --server-side=true -f config/prow/cluster/prowjob-crd/prowjob_customresourcedefinition.yaml`

Fulfill << parameters >> in config/prow/cluster/starter/starter-s3.yaml
(you will need hmac-token, app_id, private app key, github repository)
`$ kubectl apply -f config/prow/cluster/starter/starter-s3.yaml`


Register here and copy api key: https://www.ultrahook.com/register
Then run:
`$ echo “api_key: << api key >>” > ~/.ultrahook`
`$ minikube service hook -n prow`
Copy port:
`$ bash-3.2$ ultrahook prow << port >>`
Add webhook to github app like this:
https://gznznzjsn-prow.ultrahook.com/hook (example of url, do not forget ‘/hook’)
Add previously used hmac-token as a secret for webhook.
Install app into target repository.


3.Create personal access token and select scope ‘repo’.
`$ echo “<< token >>” > oauth-token`
`$ kubectl create secret generic oauth-token --from-file=oauth=./oauth-token -n prow`
`$ kubectl create secret generic oauth-token --from-file=oauth=./oauth-token -n test-pods`

4.
something with ouath…






