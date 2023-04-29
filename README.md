# Set up Prow locally with Minikube
### Webhook and HMAC token
1. Register here and get API key and webhook URL - https://www.ultrahook.com/register.
    You will get something like this:
    * _API key_ - nYKv6q6X9lJUbDCGXIRQfJ6yNuip1234
    * _URL_ - https://gznznzjsn-prow.ultrahook.com

2. Run in terminal:  
   `sudo gem install ultrahook`  
   `echo “api_key: nYKv6q6X9lJUbDCGXIRQfJ6yNuip1234” > ~/.ultrahook` (set your _API key_)
3. Run in terminal:  
   `openssl rand -hex 20`  
   You will get _HMAC token_, save it somewhere.


### GitHub App
1. Go to your GitHub account _Settings>Developer settings>GitHub Apps_. Click _New GitHub App_.
2. Set name for your App.
3. Set any dummy Homepage url.
4. Set Webhook URL like this: https://gznznzjsn-prow.ultrahook.com/hook (don't forget _/hook_). Use your HMAC token as a Webhook secret.
5. Make sure your app has this minimum set of permissions:
   1. Repository permissions:
      * Actions: Read-Only (Only needed when using the merge automation tide)
      * Administration: Read-Only (Required to fetch teams and collaborateurs)
      * Checks: Read-Only (Only needed when using the merge automation tide)
      * Commit statuses: Read & write
      * Contents: Read & write
      * Issues: Read & write
      * Metadata: Read-Only
      * Pull Requests: Read & write
   2. Organization permissions:
      * Members: Read-Only (Read & write when using peribolos) 
6. Subscribe to all events.
7. Click _Create GitHub App_.
8. Save your _App ID_ somewhere.
9. Scroll down and click _Generate a private key_. It will start downloading of file with this key. Read this file: `cat prow-app-gznznzjsn.2023-04-27.private-key.pem` (paste your file name). Save retrieved key somewhere, it will look like:
   * -----BEGIN RSA PRIVATE KEY-----  
     MIIEpQIBAAKCAQEAypPwLZtD/tkFLDXmhOL0w3kbvTfzTP0mEy1wjCpKllUiG9yQ  
     ...  
     lt7flFUEBXrg9GMLX8aGiKKXRnwHaTmj+sW48EmTx+SojMrTW5EnFho=  
     -----END RSA PRIVATE KEY-----
10. Go to _Install App_, choose _Only select repositories_ and select repository in which you will test bot. Then _Install_.

### Deployment in Minikube 
1. Copy this file https://github.com/kubernetes/test-infra/blob/master/config/prow/cluster/prowjob-crd/prowjob_customresourcedefinition.yaml (or use stored in this repository _prowjob_customresourcedefinition.yaml_) and apply it:  
   `kubectl apply --server-side=true -f prowjob_customresourcedefinition.yaml`
2. Copy one more file https://github.com/kubernetes/test-infra/blob/master/config/prow/cluster/starter/starter-s3-kind.yaml (or use stored in this repository _starter-s3-kind.yaml_).
   **On the moment of writing I have a problem with official starter in prow-controller-manager** ([PR on this bug](https://github.com/kubernetes/test-infra/pull/29370)), so I recommend you to use my version of it.
3. Fill in missing values in this starter, they are surrounded with << ... >> (example _starter-s3-kind-example.yaml_ is in this repository).
4. Apply starter: `kubectl apply -f starter-s3-kind.yaml`.
5. Run in terminal `minikube service hook -n prow` to expose hook and copy it's port.
6. Run in terminal `ultrahook prow 50902` (paste port of your hook).

### Test Prow
1. Create Pull Request in repository with installed App.
2. Comment `/meow`. Bot will answer you with cute picture if everything done correctly.
3. Run in terminal `minikube service deck -n prow`. Checkout status of jobs. If job presubmit-echo-test is stuck pending, most likely it is the problem from previous section, 2nd step.