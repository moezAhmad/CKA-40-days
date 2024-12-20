# TLS certificates in Kubernetes  

https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

1. To begin, generate a private key with the filename `learner.key` using OpenSSL:

`openssl genrsa -out learner.key 2048`

2. Next, create a CSR using the private key `learner.key`. The CSR will be named `learner.csr` and will include the common name (CN) learner:


` openssl req -new -key learner.key -out learner.csr -subj "/CN=learner"`

3. To use the CSR in Kubernetes, encode the `.csr` file in Base64 format and remove any line breaks:

`cat myuser.csr | base64 | tr -d "\n"`

4. Create a Kubernetes `CertificateSigningRequest` resource by defining it in a YAML file (e.g., csr.yml). Below is an example configuration:

```yml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: learner
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Z6Q0NBVDhDQVFBd0VqRVFNQTRHQTFVRUF3d0hiR1ZoY201bGNqQ0NBU0l3RFFZSktvWklodmNOQVFFQgpCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNa3RGZGQvZ2wvL1hOcE5XNUhzSmVDQzZNUWtETElsTHBiZHBYeVZLS3g1Ckd1NlAwZ2NmZVVkaUFGNGNueTZycnhQOUhDYzNwRERTbE5XWWl4MDNuVTZGMzk0YWNTd2VHajZORWgrdjErczAKZ0xLcnVCOWRUNmJTN3duWllnOTRFM0ZLdjVENnkzSzUxNFIvM1BveDZOdlJ3VlloVmo3ZWhMYnZDdkYzYUZTVgpaMUp3OXMyTWhYakdLZEY2dkpXSVJndlR2OVQyUk45dzdhWTNPNmZucWI1YjgwVmRvb05pa0FoalpXRGE3a1I3CllEeFcyVFZaS0dCbXZhdUMxdVZMd1ZtMjdadnlycDlOdXBjbjk5aWlsRkxPcENjRHFGVmtGVU5VM1FnUVNOLzIKTjkxOGNodFBJZ2kxeTFlNXE3cnJKc0x3RHdPNlNtb2Qzdmtxc0xiRExjc0NBd0VBQWFBQU1BMEdDU3FHU0liMwpEUUVCQ3dVQUE0SUJBUUFXeWlpYSs0cHJUbmlZUXRCVW9MckxKdmhxUUNmTmVVSXpKSW5leGhLeVBxdjBNMXB1Cld2OVNDUE1pZW84UDdxTVcxcjBZRUFJeDVGV3FaSWNFRnZRbGNTWDJmYmx2czhGelZYanRHcUlVVTlKTWlTbGIKTWFBMVh4LzhRM2dqOFJweG43TjBqcGhVN0FobEtLRXFzbjhRUVVpQkEvTTlhUTB0aWd5NFA4dm1BNklUNkdMMwpqclE1UkJjYTdlN1J0VWNJVTFONXVkZjhKWFBnMWRzMHlvRlRnajczQWk5MlNIS0htT2NMUlRVdlhONHZ3b3dYCitnUnljT0J5Vy9na2l3enpydWk4RkZHRWM3QUZpNE9jV2taUm9rNUpiTHhFK0t4ZmhrZVdDOVdvTlovblllRHEKdjg3MkdBeTN2Mm9pT2lCbncxYzRFeHg4QmVONitqbFp6c2VQCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 604800  # one week
  usages:
  - client auth
```


5. Create the CSR by applying the YAML definition:

`kubectl apply -f csr.yml`

6. You can verify the status of the CSR. Initially, it will be in a `Pending` state:

`kubectl get csr`

7. Approve the certificate once it is in the `Pending` state:

`kubectl certificate approve learner`

8. To fetch the issued certificate, use the following command to retrieve the certificate in Base64 format:

`kubectl get csr/myuser -o yaml`

9. Finally, extract the certificate from the CSR and save it as `learner.crt`:

`kubectl get csr learner -o jsonpath='{.status.certificate}'| base64 -d > learner.crt`