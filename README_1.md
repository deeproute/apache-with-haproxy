# Describe one or more weaknesses of your Dockerfile. Is it ready to be used in production?

## Apache
- The [official apache docker image](https://hub.docker.com/_/httpd) should be used since it follows most of the best practices.
- All containers should start as non-root whenever possible. This container starts as root:
```sh
docker exec -it my-running-app sh
# whoami
root
```
- Resource requests & limits should be used
- Probes should be defined for health and readiness checks

## HAProxy
- Log verbosity should be increased to be able to see if, at least, the HTTP requests are arriving to HAProxy
- There is some warnings that need to be addressed. No Warnings or Errors should be left unattended.
- SSL Certificates must be mounted and encrypted:
  - Shouldn't be in a git repo
  - Any sensitive information like certs should be stored in other safer solutions (Hashicorp Vault, Azure Key Vault..)
  - Should be mounted as K8s Secrets
- should be volume mounted as Secrets (K8s Secrets). Since the certificates are being copied in the build process then it means the image can't be reused accross different environments with other certificates.
- Resource requests & limits should be used
- Probes should be defined for health and readiness checks