### Cloudfront certificate mgmt
Your Cloudfront distribution must exist before associating a certificate with it

1. Create the CSR through your cert vendor (using DigiCert as example): http://www.digicert.com/easy-csr/keytool.htm
2. Provide requested values; `Common name` is the domain name and key size should be `RSA 2048`
3. The form will produce a keytool script that you'll copy/paste/run in your terminal; this will create the `.jks` and `.csr` files that will be used later
    - The script will prompt for a password; you can use any value you want but you MUST REMEMBER IT and use it for ALL passwords in this process or you'll need to start over
        - Will use `UrPass` for this value in all examples following
    - YOU MUST RETAIN THESE FILES FOR LATER USE.  IF YOU LOSE/MODIFY THEM, YOU MUST START OVER!
4. Request a new SSL cert from your provider
5. You have a cert!  Download your cert in a **single `.pem` file containing all the certs** -- this should contain three certificates:
    - Certificate
    - Intermediate certificste
    - Root certificate
6. Open the `.pem` file (use non-formattng text editor) and **cut the top certificate** (be sure to include the `-----BEGIN CERTIFICATE-----` AND `-----END CERTIFICATE----`) and **paste** it into a new file named `<domain_com>-cert.pem` (replace the `.` with underscores)
    - This file is what is used for pinned certs and can be distributed to other systems
7. Remove any blank lines and save the original files as `<domain_com>-chain.pem` (replace the `.` with underscores)
    - This file creates a chain of authenticity between the cert (above) and the root certificate
8. At this point we have almost everything we need to import the cert to AWS but need one last piece -- an RSA-encrypted private key
    - In terminal, go to the directory where you've stored the `.jks` file from step #3
    - Create a `.p12` file from the `.jks` file:
        - `$ keytool -importkeystore -srckeystore <domain_com>.jks -srcstorepass UrPass -srckeypass UrPass -srcalias server -destalias server -destkeystore <domain_com>.p12 -deststoretype PKCS12 -deststorepass UrPass -destkeypass UrPass`
    - Create private key from `.p12`:
        - `$ openssl pkcs12 -in <domain_com>-private-key.p12 -nodes -nocerts -out <domain_com>-private-key.pem`
    - Create RSA pem key (private):
        - `$ openssl rsa -in <domain_com>-private-key.pem -out <domain_com>-rsa-private-key.pem`
9. You have a lot of files; AWS needs three of them:
    - `<domain_com>-cert.pem` (step 6)
    - `<domain_com>-chain.pem` (step 7)
    - `<domain_com>-rsa-private-key.pem` (step 8)
10. Deploying to AWS IAM
    - Run the below ensuring that the date provided is the **expiration date**:
        - `aws iam upload-server-certificate --server-certificate-name <domain>_Mmm_YYYY --certificate-body file://<domain_com>-cert.pem --certificate _chain file://<domain_com>-chain.pem --private-key file://<domain_com>-rsa-private-key.pem --path /cloudfront/<distributionID>/`
11. Update Cloudfront distriution to use the new certificate; validate E2E: https://www.digicert.com/help