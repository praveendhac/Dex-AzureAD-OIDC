# Dex-AzureAD-OIDC
Core OS Dex to Azure AD connection

Build dex binary and dex-authenticator app
```
$ git clone https://github.com/coreos/dex.git
$ cd dex
$ make
```
Binaries are present in 
```
$ ls dex/bin/
dex		example-app	grpc-client
```
Start Dex Server
/bin/dex serve config-microsoft.yaml
From dex repo, 
```
$ ./bin/dex serve examples/config-microsoft.yaml
time="2018-07-12T22:26:47Z" level=info msg="config issuer: http://127.0.0.1:5556/dex"
time="2018-07-12T22:26:47Z" level=info msg="config storage: sqlite3"
time="2018-07-12T22:26:47Z" level=info msg="config static client: example-app"
time="2018-07-12T22:26:47Z" level=info msg="config connector: microsoft"
time="2018-07-12T22:26:47Z" level=info msg="config connector: local passwords enabled"
time="2018-07-12T22:26:47Z" level=info msg="keys expired, rotating"
time="2018-07-12T22:26:47Z" level=info msg="keys rotated, next rotation: 2018-07-13 04:26:47.243512313 +0000 UTC"
time="2018-07-12T22:26:47Z" level=info msg="listening (http/telemetry) on 0.0.0.0:5558"
time="2018-07-12T22:26:47Z" level=info msg="listening (http) on 0.0.0.0:5556"
```
Start Dex Authenticator
./bin/example-app --debug

Browse through http://127.0.0.1:5555
Click on Login button (redirects to http://127.0.0.1:5556/dex/auth?client_id=example-app&redirect_uri=http%3A%2F%2F127.0.0.1%3A5555%2Fcallback&response_type=code&scope=openid+profile+email+offline_access&state=I+wish+to+wash+my+irish+wristwatch) 

Click on "Log in with Microsoft" button (redirects to https://login.microsoftonline.com/configured_tenant_id/oauth2/v2.0/authorize?client_id=configured_client_id&redirect_uri=http%3A%2F%2F127.0.0.1%3A5556%2Fdex%2Fcallback&response_type=code&scope=user.read&state=ewwkc32kwsedxuy7d22bcrs3t)

Enter your AzureAD credentials

If the credentials are correct we will be redirected to http://127.0.0.1:5556/dex/approval?req=v6zh7pncphriiuvet3syfvxtp

dex_grant_access

Click on "Grant Access" button (http://127.0.0.1:5555/callback?code=rh5lqa7pelndskimecqjviv7g&state=I+wish+to+wash+my+irish+wristwatch)
Response has below details confirming working of OIDC
```
Token:

eyJhbGciOiJSUzI1NiIaWeSoMeZCI6ImEyZGQ5NjgzNzJjZmJmYwYTAifQ.eyJpc3MiOiJodHRwOi8vMTI3LjAuMC4xOjU1.Q5NjgzNzJjZmJm1122ZDFjNDQ3ODUzMDk3OTNjMmYwYTAi-xx8rEYakY34qAgvvAqWvuzJR7Auz3LwWfHe9xGkf4EadxgjysQ6VvWTS3Z9yn3ddddtWMmGA2o-_4FdYfigB49D8h8Ee6vcVY2_t47iaBJ8KAdRmqlc8xXgHwVduQlr7praveenmMXlAnW-JMU6aRWYM8qhlMhQfymmW8NkJ5DIO_AnS86p9RdrCFB5M44nKrj917wcOCXO1-0U12tw0id-YaxxOWK4EOXiOUXq8uPaM6sqBw
Claims:

{
  "iss": "http://127.0.0.1:5556/dex",
  "sub": "RkNzktYmY0NS01M2U5NzdmOTdlNjASCW1pY3Jvc29mdA==",
  "aud": "example-app",
  "exp": 1531521656,
  "iat": 1531435256,
  "at_hash": "5lOe6h39xttB4mfC3T-Www",
  "email": "praveen@disects.onmicrosoft.com",
  "email_verified": true,
  "name": "Praveen D"
}
Refresh Token:

ChlkZHDjcXJuaTYPzNWZsdHRhaXYzA3RlYTRsEhV3b20zbzZ5EDNnZHlvaENqcHFxZmF6aNo1

```
##Errors
Error1:
time="2018-07-12T13:06:22Z" level=error msg="Connector \"microsoft\" returned error when creating callback: expected callback URL \"http://127.0.0.1:5556/dex/callback\" did not match the URL in the config \"https://dex.praveend.com/callback\""
FIX: Make sure redirectURL in config-microsoft.yaml and "Reply URLs" in AzureAD App registrations are same. Use HTTPS, http is not supported 

Error2:
AADSTS90130: Application 'configured_client_id_here' (configured_client_id_name_here) is not supported over the /common or /consumers endpoints. Please use the /organizations or tenant-specific endpoint.
FIX: "tenant:" configuration was missing

Error3:
AADSTS50011: The reply url specified in the request does not match the reply urls configured for the application: '5a3a2f89-0b36-454a-b4e9-16b5bff3c751'.
URL which threw above error
https://login.microsoftonline.com/configured_tenant_id/oauth2/v2.0/authorize?client_id=configured_client_id&redirect_uri=http%3A%2F%2F127.0.0.1%3A5556%2Fdex%2Fcallback&response_type=code&scope=user.read&state=x3cqx2k36zltpi5zjnkirldlf
FIX: Add http://127.0.0.1:5556/dex/callback to "Reply URLs" in AzureAD App registrations

##Debugging
If the dex configuration is correct, accessing below URL should return 200 containing key-value pairs which provide details about the OpenID Connect provider's configuration, including the URIs of the authorization, token, userinfo, and public-keys endpoints. URL is of the form issuer_url + "/.well-known/openid-configuration"
http://127.0.0.1:5556/dex/.well-known/openid-configuration

##Refer
https://github.com/coreos/dex/blob/master/Documentation/getting-started.md
https://github.com/coreos/dex/blob/master/Documentation/connectors/microsoft.md
http://openid.net/connect/
https://docs.microsoft.com/en-us/azure/aks/aad-integration
https://github.com/coreos/dex/blob/master/Documentation/kubernetes.md
https://github.com/mintel/dex-k8s-authenticator


