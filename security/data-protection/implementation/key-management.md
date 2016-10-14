---
uid: security/data-protection/implementation/key-management
---
<a name=data-protection-implementation-key-management></a>

  # Key Management

The data protection system automatically manages the lifetime of master keys used to protect and unprotect payloads. Each key can exist in one of four stages.

* Created - the key exists in the key ring but has not yet been activated. The key shouldn't be used for new Protect operations until sufficient time has elapsed that the key has had a chance to propagate to all machines that are consuming this key ring.

* Active - the key exists in the key ring and should be used for all new Protect operations.

* Expired - the key has run its natural lifetime and should no longer be used for new Protect operations.

* Revoked - the key is compromised and must not be used for new Protect operations.

Created, active, and expired keys may all be used to unprotect incoming payloads. Revoked keys by default may not be used to unprotect payloads, but the application developer can [override this behavior](../consumer-apis/dangerous-unprotect.md#data-protection-consumer-apis-dangerous-unprotect.md) if necessary.

Warning: The developer might be tempted to delete a key from the key ring (e.g., by deleting the corresponding file from the file system). At that point, all data protected by the key is permanently undecipherable, and there is no emergency override like there is with revoked keys. Deleting a key is truly destructive behavior, and consequently the data protection system exposes no first-class API for performing this operation.

  ## Default key selection

When the data protection system reads the key ring from the backing repository, it will attempt to locate a "default" key from the key ring. The default key is used for new Protect operations.

The general heuristic is that the data protection system chooses the key with the most recent activation date as the default key. (There's a small fudge factor to allow for server-to-server clock skew.) If the key is expired or revoked, and if the application has not disabled automatic key generation, then a new key will be generated with immediate activation per the [key expiration and rolling](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management-expiration) policy below.

The reason the data protection system generates a new key immediately rather than falling back to a different key is that new key generation should be treated as an implicit expiration of all keys that were activated prior to the new key. The general idea is that new keys may have been configured with different algorithms or encryption-at-rest mechanisms than old keys, and the system should prefer the current configuration over falling back.

There is an exception. If the application developer has [disabled automatic key generation](../configuration/overview.md#data-protection-configuring-disable-automatic-key-generation.md), then the data protection system must choose something as the default key. In this fallback scenario, the system will choose the non-revoked key with the most recent activation date, with preference given to keys that have had time to propagate to other machines in the cluster. The fallback system may end up choosing an expired default key as a result. The fallback system will never choose a revoked key as the default key, and if the key ring is empty or every key has been revoked then the system will produce an error upon initialization.

<a name=data-protection-implementation-key-management-expiration></a>

  ## Key expiration and rolling

When a key is created, it is automatically given an activation date of { now + 2 days } and an expiration date of { now + 90 days }. The 2-day delay before activation gives the key time to propagate through the system. That is, it allows other applications pointing at the backing store to observe the key at their next auto-refresh period, thus maximizing the chances that when the key ring does become active it has propagated to all applications that might need to use it.

If the default key will expire within 2 days and if the key ring does not already have a key that will be active upon expiration of the default key, then the data protection system will automatically persist a new key to the key ring. This new key has an activation date of { default key's expiration date } and an expiration date of { now + 90 days }. This allows the system to automatically roll keys on a regular basis with no interruption of service.

There might be circumstances where a key will be created with immediate activation. One example would be when the application hasn't run for a time and all keys in the key ring are expired. When this happens, the key is given an activation date of { now } without the normal 2-day activation delay.

The default key lifetime is 90 days, though this is configurable as in the following example.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   services.AddDataProtection()
       // use 14-day lifetime instead of 90-day lifetime
       .SetDefaultKeyLifetime(TimeSpan.FromDays(14));
   ````

An administrator can also change the default system-wide, though an explicit call to SetDefaultKeyLifetime will override any system-wide policy. The default key lifetime cannot be shorter than 7 days.

  ## Automatic keyring refresh

When the data protection system initializes, it reads the key ring from the underlying repository and caches it in memory. This cache allows Protect and Unprotect operations to proceed without hitting the backing store. The system will automatically check the backing store for changes approximately every 24 hours or when the current default key expires, whichever comes first.

Warning: Developers should very rarely (if ever) need to use the key management APIs directly. The data protection system will perform automatic key management as described above.

The data protection system exposes an interface IKeyManager that can be used to inspect and make changes to the key ring. The DI system that provided the instance of IDataProtectionProvider can also provide an instance of IKeyManager for your consumption. Alternatively, you can pull the IKeyManager straight from the IServiceProvider as in the example below.

Any operation which modifies the key ring (creating a new key explicitly or performing a revocation) will invalidate the in-memory cache. The next call to Protect or Unprotect will cause the data protection system to reread the key ring and recreate the cache.

The sample below demonstrates using the IKeyManager interface to inspect and manipulate the key ring, including revoking existing keys and generating a new key manually.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": true, "classes": [], "xml:space": "preserve", "language": "none", "source": "/Users/shirhatti/src/Docs/aspnet/security/data-protection/implementation/key-management/samples/key-management.cs"} -->

````none

   using System;
   using System.IO;
   using System.Threading;
   using Microsoft.AspNetCore.DataProtection;
   using Microsoft.AspNetCore.DataProtection.KeyManagement;
   using Microsoft.Extensions.DependencyInjection;

   public class Program
   {
       public static void Main(string[] args)
       {
           var serviceCollection = new ServiceCollection();
           serviceCollection.AddDataProtection()
               // point at a specific folder and use DPAPI to encrypt keys
               .PersistKeysToFileSystem(new DirectoryInfo(@"c:\temp-keys"))
               .ProtectKeysWithDpapi();
           var services = serviceCollection.BuildServiceProvider();

           // perform a protect operation to force the system to put at least
           // one key in the key ring
           services.GetDataProtector("Sample.KeyManager.v1").Protect("payload");
           Console.WriteLine("Performed a protect operation.");
           Thread.Sleep(2000);

           // get a reference to the key manager
           var keyManager = services.GetService<IKeyManager>();

           // list all keys in the key ring
           var allKeys = keyManager.GetAllKeys();
           Console.WriteLine($"The key ring contains {allKeys.Count} key(s).");
           foreach (var key in allKeys)
           {
               Console.WriteLine($"Key {key.KeyId:B}: Created = {key.CreationDate:u}, IsRevoked = {key.IsRevoked}");
           }

           // revoke all keys in the key ring
           keyManager.RevokeAllKeys(DateTimeOffset.Now, reason: "Revocation reason here.");
           Console.WriteLine("Revoked all existing keys.");

           // add a new key to the key ring with immediate activation and a 1-month expiration
           keyManager.CreateNewKey(
               activationDate: DateTimeOffset.Now,
               expirationDate: DateTimeOffset.Now.AddMonths(1));
           Console.WriteLine("Added a new key.");

           // list all keys in the key ring
           allKeys = keyManager.GetAllKeys();
           Console.WriteLine($"The key ring contains {allKeys.Count} key(s).");
           foreach (var key in allKeys)
           {
               Console.WriteLine($"Key {key.KeyId:B}: Created = {key.CreationDate:u}, IsRevoked = {key.IsRevoked}");
           }
       }
   }

   /*
    * SAMPLE OUTPUT
    *
    * Performed a protect operation.
    * The key ring contains 1 key(s).
    * Key {1b948618-be1f-440b-b204-64ff5a152552}: Created = 2015-03-18 22:20:49Z, IsRevoked = False
    * Revoked all existing keys.
    * Added a new key.
    * The key ring contains 2 key(s).
    * Key {1b948618-be1f-440b-b204-64ff5a152552}: Created = 2015-03-18 22:20:49Z, IsRevoked = True
    * Key {2266fc40-e2fb-48c6-8ce2-5fde6b1493f7}: Created = 2015-03-18 22:20:51Z, IsRevoked = False
    */

   ````

  ## Key storage

The data protection system has a heuristic whereby it tries to deduce an appropriate key storage location and encryption at rest mechanism automatically. This is also configurable by the app developer. The following documents discuss the in-box implementations of these mechanisms:

* [In-box key storage providers](key-storage-providers.md#data-protection-implementation-key-storage-providers.md)

* [In-box key encryption at rest providers](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest-providers.md)
