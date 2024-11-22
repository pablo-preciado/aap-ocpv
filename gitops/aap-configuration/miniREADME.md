Once it's deployed get the route to your AAP instance:

    oc get route my-aap-instance -n my-aap

Also get the admin password from the secret created by the operator:

    oc get secret my-aap-instance-admin-password -n my-aap -o go-template --template="{{.data.password|base64decode}}"

Login to the AAP instance with the admin user and the password you just retrieved. In AAP, fill the data so it can use a subscription:

![AAP Subscrition](images/aap-subscription.png)

In our case we use username/password with our Red Hat account, then accept EULA and finish.

You should now see something like this:

![AAP first login](images/aap-first-login.png)

We're going to create a couple of new Credential Types for connecting with Red Hat's CDN. go to Automation Execution > Infrastructure > Credential types > Create credential type. Give it the folowing values:
 - Name: Red Hat CDN User/Pass
 - Input configuration:
 ```
 fields:
  - id: rhn_username
    type: string
    label: "RHN Username"
  - id: rhn_password
    type: string
    label: "RHN Password"
    secret: true
```
 - Injector configuration:
 ```
 extra_vars:
  rhsm_username: "{{ rhn_username }}"
  rhsm_password: "{{ rhn_password }}"
 ```
![AAP Credential Type RHN User/Pass](images/aap-credtype-rhnuser.png)

Create another credential type for the Red Hat portal Service Account. Go to Automation Execution > Infrastructure > Credential types > Create credential type. Fill with the folowing values:
 - Name: Red Hat CDN Service Account
 - Input configuration:
 ```
 fields:
  - id: rhn_client_id
    type: string
    secret: true
    label: "RHN Client ID"
  - id: rhn_client_secret
    type: string
    label: "RHN Client Secret"
    secret: true
```
 - Injector configuration:
 ```
extra_vars:
  rhn_password: '{{ rhn_client_secret }}'
  rhn_username: '{{ rhn_client_id }}'
 ```
![AAP Credential Type RHN Service Account](images/aap-credtype-rhnsa.png)

Lastly, create a token for your user in Ansible Automation Platform. Go to Access Management > Users > click on admin > go to Tokens tab > Create Token. Give it a Write scope and leave the rest empty. Create the token and copy paste it to a safe place, you wont be able to retrieve it afterwards.

Now, go back to the OpenShift cluster and create a secret in the aap namespace looking like this:
```
apiVersion: v1
kind: Secret
metadata:
  name: controller-access
  namespace: aap
  type: Opaque
stringData:
  token: <generated-token>
  host: <ansible-automation-platform-url>
```

You can retrieve the AAP url by running this command:
```
oc get route my-aap -n aap -o jsonpath='{.spec.host}'
```
