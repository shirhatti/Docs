---
uid: security/data-protection/implementation/key-encryption-at-rest
---
<a name=data-protection-implementation-key-encryption-at-rest></a>

  # Key Encryption At Rest

By default the data protection system [employs a heuristic](../configuration/default-settings.md#data-protection-default-settings.md) to determine how cryptographic key material should be encrypted at rest. The developer can override the heuristic and manually specify how keys should be encrypted at rest.

Note: If you specify an explicit key encryption at rest mechanism, the data protection system will deregister the default key storage mechanism that the heuristic provided. You must [specify an explicit key storage mechanism](key-storage-providers.md#data-protection-implementation-key-storage-providers.md), otherwise the data protection system will fail to start.

<a name=data-protection-implementation-key-encryption-at-rest-providers></a>

The data protection system ships with three in-box key encryption mechanisms.

  ## Windows DPAPI

*This mechanism is available only on Windows.*

When Windows DPAPI is used, key material will be encrypted via [CryptProtectData](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380261(v=vs.85).aspx) before being persisted to storage. DPAPI is an appropriate encryption mechanism for data that will never be read outside of the current machine (though it is possible to back these keys up to Active Directory; see [DPAPI and Roaming Profiles](https://support.microsoft.com/en-us/kb/309408/#6)). For example to configure DPAPI key-at-rest encryption.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
       // only the local user account can decrypt the keys
       .ProtectKeysWithDpapi();
   ````

If ProtectKeysWithDpapi is called with no parameters, only the current Windows user account can decipher the persisted key material. You can optionally specify that any user account on the machine (not just the current user account) should be able to decipher the key material, as shown in the below example.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
       // all user accounts on the machine can decrypt the keys
       .ProtectKeysWithDpapi(protectToLocalMachine: true);
   ````

  ## X.509 certificate

*This mechanism is not yet available on `.NET Core`.*

If your application is spread across multiple machines, it may be convenient to distribute a shared X.509 certificate across the machines and to configure applications to use this certificate for encryption of keys at rest. See below for an example.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
       // searches the cert store for the cert with this thumbprint
       .ProtectKeysWithCertificate("3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0");
   ````

Because this mechanism uses [X509Certificate2](https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509certificate2(v=vs.110).aspx) and [EncryptedXml](https://msdn.microsoft.com/en-us/library/system.security.cryptography.xml.encryptedxml(v=vs.110).aspx) under the covers, this feature is currently only available on Desktop CLR. Additionally, due to .NET Framework limitations only certificates with CAPI private keys are supported. See [Certificate-based encryption with Windows DPAPI-NG](xref:security/data-protection/implementation/key-encryption-at-rest#data-protection-implementation-key-encryption-at-rest-dpapi-ng) below for possible workarounds to these limitations.

<a name=data-protection-implementation-key-encryption-at-rest-dpapi-ng></a>

  ## Windows DPAPI-NG

*This mechanism is available only on Windows 8 / Windows Server 2012 and later.*

Beginning with Windows 8, the operating system supports DPAPI-NG (also called CNG DPAPI). Microsoft lays out its usage scenario as follows.

   Cloud computing, however, often requires that content encrypted on one computer be decrypted on another. Therefore, beginning with Windows 8, Microsoft extended the idea of using a relatively straightforward API to encompass cloud scenarios. This new API, called DPAPI-NG, enables you to securely share secrets (keys, passwords, key material) and messages by protecting them to a set of principals that can be used to unprotect them on different computers after proper authentication and authorization.

   From [https://msdn.microsoft.com/en-us/library/windows/desktop/hh706794(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/desktop/hh706794(v=vs.85).aspx)

The principal is encoded as a protection descriptor rule. Consider the below example, which encrypts key material such that only the domain-joined user with the specified SID can decrypt the key material.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
     // uses the descriptor rule "SID=S-1-5-21-..."
     .ProtectKeysWithDpapiNG("SID=S-1-5-21-...",
       flags: DpapiNGProtectionDescriptorFlags.None);
   ````

There is also a parameterless overload of ProtectKeysWithDpapiNG. This is a convenience method for specifying the rule "SID=mine", where mine is the SID of the current Windows user account.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
     // uses the descriptor rule "SID={current account SID}"
     .ProtectKeysWithDpapiNG();
   ````

In this scenario, the AD domain controller is responsible for distributing the encryption keys used by the DPAPI-NG operations. The target user will be able to decipher the encrypted payload from any domain-joined machine (provided that the process is running under their identity).

  ## Certificate-based encryption with Windows DPAPI-NG

If you're running on Windows 8.1 / Windows Server 2012 R2 or later, you can use Windows DPAPI-NG to perform certificate-based encryption, even if the application is running on [.NET Core](https://microsoft.com/net/core). To take advantage of this, use the rule descriptor string "CERTIFICATE=HashId:thumbprint", where thumbprint is the hex-encoded SHA1 thumbprint of the certificate to use. See below for an example.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   sc.AddDataProtection()
       // searches the cert store for the cert with this thumbprint
       .ProtectKeysWithDpapiNG("CERTIFICATE=HashId:3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0",
           flags: DpapiNGProtectionDescriptorFlags.None);
   ````

Any application which is pointed at this repository must be running on Windows 8.1 / Windows Server 2012 R2 or later to be able to decipher this key.

  ## Custom key encryption

If the in-box mechanisms are not appropriate, the developer can specify their own key encryption mechanism by providing a custom IXmlEncryptor.
