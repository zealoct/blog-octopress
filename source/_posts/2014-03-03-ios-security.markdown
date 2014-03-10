---
layout: post
title: "Notes on iOS Security Whitepaper"
date: 2014-03-03 19:56:09 +0800
comments: true
published: true
categories: 
- Mobile
- Security
- iOS
---

This notes is based on *iOS Security - February 2014*, origin link can be found [here](http://images.apple.com/iphone/business/docs/iOS_Security_Feb14.pdf).

This paper gives a brief description of iOS security, including hardware security features and how iOS leverages these features.

System Security
---------------

### Secure Boot Chain

ROM is sealed with public key of Apple Root CA, and will verify the Low-Level Bootloader (LLB) before load it.

For devices with an A7 processor, the *Secure Enclave* coprocessor also utilizes a secure boot process that ensures its separate software is verified and signed by Apple.

Verification failure will enter **recovery mode**, if Boot ROM is not even able to load or verify LLB, it enters **DFU (Device Firmware Update) mode**.



### System Software Authorization

GOAL: prevent the devices from being downgraded.

Need iTunes to upgrade, when upgrading, iTunes (or the device) would send 

1. a list cryptographic measurements of each installation bundle to be installed
2. nonce
3. device's unique ID (ECID)

to Apple installation authorization server.

If upgrade request is permitted, server would add ECID to the measurement and signs the result.

Device would check each item loaded from disk at boot time.


<!-- more -->


### Secure Enclave

There are two kinds of processors, *application processor* and *Secure Enclave*.

**What is Secure Enclave?** 
A coprocessor fabricated in the Apple A7 chip, has ITS OWN *secure boot* and *personalized software update*, *encrypted memory* and *hardware random number generator*.

**What does Secure Enclave do?**
It provides all cryptographic operations for Data Protection key management and maintains the integrity of Data Protection, it is also responsible for processing fingerprint data, determining if there is a match, and enabling access or purchase on behalf of the user. 

**How Secure Enclave communicate with app processor?** 
Through interrupt-driven mailbox and shared memory data buffers.


Each Secure Enclave is provisioned during fabrication with its own *UID (Unique ID)*, not known to Apple, not accessible to other parts of the system. Note that this UID is NOT SAME with that fused into application processor.

Create an ephemeral key tangled with UID to encrypt Secure Enclave's portion of the device's memory space.

Data saved to file system by Secure Enclave is encrypted with a key tangled with UID and an anti-replay counter.


> The session key exchange uses AES key wrapping with both sides providing a random key that establishes the session key and uses AES-CCM transport encryption

Utilizes System Software Authorization to ensure the integrity of its software and prevent downgrade.

#### Some Questions

1. How secure boot of Secure Enclave is achieved?

1. Where the ephemeral key is stored? How about the key and anti-replay counter used to encrypt data written to file system by Secure Enclave?




### Touch ID

80*80 pixel, 500 ppi raster scan, temporarily stored in Secure Enclave, data out from Touch ID sensor is encrypted, A7 can only forward it to Secure Enclave but never read its content.

User's fingerprint map never leaves ip5s

#### Process of unlocking an iPhone

On regular A7 processor, Data Protection *class keys* are discarded, and regenerated when user unlock the device with passcode.

With Touch ID, the keys are wrapped with a key given to Touch ID subsystem, Touch ID will provide the key for unwrapping if it recognizes the user's fingerprint (details refer to section *File Data Protection*)




### Simple Conclusion

The following features are most critical for iOS system security

- UID in Secure Enclave
- dedicated secure CPU
- unbreakable ROM


Encryption and Data Protection
-----

Focus on the protection of data stored on the device.

### Hardware Security Features

Apple's devices involves some hardware support for security, these includes:

- *Dedicated hardware AES 256 crypto engine* built into DMA path between Flash and Main Memory

- *Hardware SHA-1*

- *Fused UID*, in application processor, unique to each device, software and firmware cannot read UID, can only get the results of encryption or decryption operations

- *Fused GID*, common to a class of devices, used as an additional level of protection when delivering system software during installation and restore

> Integrating these keys into the silicon helps prevent them from being tampered with or bypassed, or accessed outside the AES engine. 

- A hardware *random number generator (RNG)* to create all cryptographic keys (except those used in Secure Enclave)

- *Effaceable Storage* for securely erasing saved keys




### File Data Protection

GOAL: Protect data stored in flash memory.

> Data Protection allows the device to respond to common events such as incoming phone calls, but also enables a high level of encryption for sensitive data

Managing a **hierarchy of keys**; built on hardware encryption; encrypted every file stored into the flash.

> Data Protection is controlled on a per-file basis by assigning each file to a class; accessibility is determined by whether the class keys have been unlocked

Create a new 260-bit key (per-file key) for EACH file on the data partition, *hardware AES engine* uses these keys to encrypt files when written to flash memory using AES CBC mode.

Per-file key is wrapped (by Enclave) with one class key (performed using NIST AES key wrapping, per RFC 3394). The wrapped per-file key is stored in the file’s *metadata*.

To open a file: **1.** decrypt metadata with *File System Key* **2.** unwrapped with class key **3.** supply the per-file key to hardware AES engine.

Use a random *File System Key* to encrypt the metadata of all files in the file system. This file system key is created when iOS first installed or when the device is wiped by a user. The key is stored in Effaceable Storage to be quickly erased. 

Once the File System Key is wiped, there should be no way to get the content of all the files.

The work of key management is done by Secure Enclave, as mentioned in section Secure Enclave.



### Passcodes

Supports *four-digit* and *arbitrary-length alphanumeric* passcodes.

> In addition to unlocking the device, a passcode provides the entropy for encryption keys, which are not stored on the device. This means an attacker in possession of a device can’t get access to data in certain protection classes without the passcode. 

Passcode is tangled with UID.

Takes longer and longer for brute-force hack.

**Where is this Passcodes stored?**
Passcodes should be managed by Secure Enclave, and stored in file system after being encrypted by Secure Enclave.



### Data Protection Classes

Basic classes:

- Complete Protection
- Protected Unless Open
- Protected Until First User Authentication
- No Protection

*NSFileProtectionComplete* : class key protected with a key derived from Passcode and device UID, auto discard the decrypted class key after the screen is lock. File becomes inaccessible until unlock (either by Passcode or Touch ID).

*NSFileProtectionCompleteUnlessOpen* : for files need to be written while locking. Besides per-file key, Data Protection: **1.** creates another *public/private key pair* for the file **2.** a shared secret is computed using file's private key and this class's public key **3.** wrap the per-file key with the hash of shared secret **4.** wrapped per-file key and file's public key are stored in the file's metadata **5.** wipe the file's private key from memory **6.** to open the file, the shared secret is re-generated using file's public key and this class's private key, to unwrap per-file key.

*NSFileProtectionCompleteUntilFirstUserAuthentication* : behaves in the same way as Complete Protection, only that decrypted class key is not wiped after lock. This is DEFAULT CLASS for all third-party app data.

*NSFileProtectionNone* : class key protected only with UID (no Passcode), kept in Effaceable Storage. All the keys needed to decrypt files of this class are stored on the device.

So in a short **conslusion**, all the files in iOS devices are encrypted, as there always be a hardware AES between memory and flash, only that stronger protection involves encrypting class key with Passcode, and auto wiping the decrypted key after the device is locked.



### Keychain Data Protection

GOAL: protect short but sensitive bits of data in apps, such as keys and login tokens.

Implemented as SQLite database, and there is only one database in the system. The *securityd* deamon determines which keychain items each process or app can access.

The deamon would check app's "keychain-access-groups" and the "application-identifier" entitlement. Apps from the same author (have the same access groups prefix allocated to them through the iOS Developer Program) can share Keychain items.

Similar protect class as file Data Protection.

> Each keychain class has a “This device only” counterpart, which is always protected with the UID when being copied from the device during a backup, rendering it useless if restored to a different device


### Keybags

Manage keys for file and Keychain Data Protection classes, four keybags: *System*, *Backup*, *Escrow*, and *iCloud*.

**System keybag** where wrapped class keys are stored, is No Protection class itself. Contents of system keybag are encrypted with a key held in Effaceable Storage. This key is wiped and regenerated each time user change Passcode. System keybag is the ONLY keybag stored on the device.

**Backup keybag** created when an encrypted backup is made by iTunes and stored 
on the computer where the device is backed up. The backed-up data is **re-encrypted** to a new set of keys (a new keybag).

**Escrow keybag** is used for iTunes syncing and mobile device management (MDM). Allows iTunes to sync without requiring the user to enter a passcode and allows an MDM server to remotely clear a user's passcode. Stored on computer. Contains exactly the SAME class keys used on device, protected by a newly created key, which is stored on the device in Protected Until First User Authentication class.

**iCloud Backup keybag** similar to Backup keybag, all class keys in this keybag are asymmetric.

> For all Data Protection classes except No Protection, the encrypted data is read from the device and sent to iCloud. The corresponding class keys are protected by iCloud keys. The keychain class keys are wrapped with a UID-derived key in the same way as an unencrypted iTunes backup. 



App Security
-----

iOS provides protection to ensure that apps are signed and verified, cannot execute malicious code, and are sandboxed to protect user data at all times.


### App Code Signing

Mandatory code signing esing Apple-issued certificate. Developers must join iOS Developer Program and provide their real-world indentity for verification. 



### Runtime Process Security

Sandbox. Randomly assigned home directory, can only communicate with APIs.

Majority of iOS and all third-party apps run as the non-privileged user "mobile".

Address space layout randomization (ASLR)

ARM's Execute Never (XN) feature, which marks memory pages as non-executable. 

> Memory pages marked as both writable and executable can be used only by apps under tightly controlled conditions: The kernel checks for the presence of the Apple-only dynamic code-signing entitlement.



### Data Protection in Apps

Data Protection is available for file and database APIs, Protected Until First User Authentication by default.



### Accessories

The following process is entirely handled by a integrated circuit that Apple provides to approved accessory manufacturers and is transparent to the accessory.

- Check accessory's Apple-provided certificate. 
- Send a challenge, which the accessory must answer with a signed response.



Network Security 
-----

Uses standard networking protocols for authenticated, authorized, and encrypted communications. Integrates proven technologies and the latest standards for both Wi-Fi and cellular data network connections. 

(Refer to the paper for details)

- SSL (v3), TLS (v1.0, v1.1, v1.2)
- VPN, supports multiple protocols and authentication methods
- Wi-Fi, industry-standard Wi-Fi protocols
- Bluetooth, Encryption Mode 3/4, Service Level 1 connections, multiple Blutooth profiles
- Single Sign-on (I do not know what this SSO is...)

### AirDrop Security

Use Bluetooth Low-Energy (BTLE) and Apple-created peer-to-peer Wi-Fi technology.

If enabled, a 2048-bit RSA identity is stored on the device, and an AirDrop identity hash is created based on email address and phone number.

Use TLS connection.


Internet Security
-----
Explain the security control of iMessage, FaceTime, Siri, iCloud, iCloudKeychain in detail.

### iMessage

The contents of messages of iMessage are protected by end-to-end encryption, so no one but the sender and receiver can access them, even Apple cannot.

Device generates two pairs of keys for iMessage: an RSA 1280-bit key for encryption and an ECDSA 256-bit key for signing. The private keys are stored in device's keychain while the public keys are sent to Apple's directory service (IDS), where they are associated with user's phone number or email address and Apple Push Notification Service (APNs) address.

To send a message, iMessage first fetches receiver's public keys and APNs addresses from the IDS, then encrypts the content using receiver's public keys, and signs the encrypted messages with the sender's private key, finally, iMessage dispatches each encrypted message to APNs for delivary. Metadata is not encrypted while communication with APNs is encrypted using TLS.

If the message contains attachments, the attachments are uploaded to iCloud after encryption, the keys to decrypt attackments along with URI to the encrypted attachments are included in the encrypted message.

For the receiver, each device receives its copy of the message from APNs, and decrypts the message with its own private key. The message can be verified using sender's public key.



Device Control
-----
Policies for Passcode Protection, Configuration Enforcement, Mobile Device Management, Apple Configurator, Device Restrictions, Supervised Only Restrictions and Remote Wipe.


Conclusion
-----

- Hardware support: UID, Secure Enclave, ROM, Hardware AES, Random Number Generator
- Dedicated Secure Processor, with encrypted memory
- Full Storage Encryption
- Hierarchy of Key Management

From the document we can see, Apple really takes great efforts in security, and as Apple's hardware and software are tightly combined, they possess the most enviable hardware security features. But to achieve security, Apple sacrifices the ability of third party apps by setting a lot of constrains and providing only a limited APIs.

Oh! One more thing, all of these protections are useless if your device is rooted.