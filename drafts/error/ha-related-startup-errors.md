# HA related startup errors

## Failure scenarios related to decryption issues for passwords when starting up in HA mode.  

### What This Article Covers

- Intro
- Error Case 1: Failed to Decrypt deployment configuration
- Error Case 2: Failed to Decrypt secrets in deployment fragment

# Understanding Decryption Failures in HighByte Intelligence Hub Startup

When the Intelligence Hub fails to start, it often indicates an issue with decrypting the deployment configuration or secrets within deployment fragments. These failures typically stem from problems with the `deployment-settings.json` file or the associated PKCS#12 certificate used for decryption.

## Error Case 1: Failed to Decrypt Deployment Configuration

**Error Message:**
```
java.lang.Exception: Failed to decrypt deployment configuration.
    at com.highbyte.intelligencehub.runtime.deployment.c.a(Unknown Source)
    at com.highbyte.intelligencehub.runtime.deployment.c.a(Unknown Source)
    at com.highbyte.intelligencehub.runtime.a.c.e(Unknown Source)
    at com.highbyte.intelligencehub.runtime.Main.b(Unknown Source)
    at com.highbyte.intelligencehub.runtime.main.main(Unknown Source)
```

### Description
This error occurs when **Intelligence Hub** is unable to decrypt the main `deployment-settings.json` file. This configuration file stores critical deployment metadata, including encrypted references to fragments and other runtime parameters.

### Common Causes
- The PKCS#12 (`.p12`) certificate file used for decryption does not match the one used to encrypt the `deployment-settings.json`.  
- The certificate password has changed or is incorrect.  
- The certificate file is missing, corrupted, or inaccessible due to permissions.  

### Recommended Actions
1. Verify that the `.p12` file exists in the correct path and matches the original encryption key used during the deployment export.  
2. Confirm that the certificate password specified in the environment or startup configuration is correct.  
3. Check file permissions to ensure the Intelligence Hub runtime has read access to the `.p12` file.  
4. If decryption still fails, re-export the deployment package from a system with a valid configuration and matching certificate.

***

## Error Case 2: Failed to Decrypt Secrets in Deployment Fragment

**Error Message:**
```
Intelligence Hub failed to start. Contact HighByte Support and provide the following error information:
java.lang.Exception: Failed to decrypt secrets in fragment
    at com.highbyte.intelligencehub.runtime.deployment.c.a(Unknown Source)
    at com.highbyte.intelligencehub.runtime.deployment.c.b(Unknown Source)
    at com.highbyte.intelligencehub.runtime.a.c.e(Unknown Source)
    at com.highbyte.intelligencehub.runtime.Main.b(Unknown Source)
    at com.highbyte.intelligencehub.runtime.main.main(Unknown Source)
C:\HB\testdeploy431\HighByte-Intelligence-Hub-4.3
```

### Description
This error indicates that the **deployment configuration** (`deployment-settings.json`) was successfully decrypted, allowing the Intelligence Hub to locate and load deployment fragments. However, during the fragment processing phase, the system was unable to decrypt one or more **secrets** contained within those fragments.

### Common Causes
- The fragment files contain encrypted secrets created with a different or mismatched PKCS#12 certificate.  
- The deployment fragments were manually modified or imported from another environment using a different encryption key.  
- Corruption or truncation of the fragment files during transport or versioning.  

### Recommended Actions
1. Confirm that the same PKCS#12 certificate used to encrypt the deployment fragments is available and valid.  
2. Re-export the affected deployment fragments from a known working environment using the same certificate.  
3. Review the fragment paths and ensure no manual edits or file mismatches exist.  
4. If necessary, perform a clean deployment import with verified valid credentials and secrets.

***

## Additional Notes
Both errors are encryption-related startup failures tied to certificate management. HighByte Intelligence Hub relies on consistent and valid cryptographic credentials to securely handle deployment data. When moving deployments between systems, always transfer both the deployment configuration and its corresponding `.p12` key together to maintain compatibility.

***

