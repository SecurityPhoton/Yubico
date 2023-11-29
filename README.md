# How to use Yubikey

![Key!](/data/yubikey.jpg "Key")
<img src="/data/yubikey.jpg" width="100" height="100">

## General concepts

There is 4 main usage examples of yubikey 5 NFC:
- OTP
- FIDO2
- PIV (Personal Identity Verification) and in the context of YubiKey, it refers to a smart card-based specification used for secure authentication and digital signatures.
- GPG (GNU Privacy Guard) refers to the capability of YubiKey devices to support GPG for secure encryption, digital signatures, and managing cryptographic keys.

## PINs and PUK & Key

YubiKey 5 series devices support several types of PINs (Personal Identification Numbers) and it is important to understand the concept of it before you use the key. Here are the main types of PINs supported by YubiKey 5:

- **User PIN** (Personal Identification Number):
            The User PIN is a mandatory PIN required to unlock the YubiKey and access its features. It protects the private keys stored on the device.
            This PIN is typically required when using the YubiKey for authentication, digital signatures, or encryption.
        It's essential to remember that the User PIN should be strong and kept confidential to prevent unauthorized access.

        Default: 123456

- **Admin PIN**:
            The Admin PIN provides administrative control over the YubiKey's settings and configurations.
            It's used to change certain settings, reset the User PIN, and perform administrative tasks on the device.
            The Admin PIN provides higher-level access than the User PIN and should be kept confidential as well.

        Default: 12345678

- **PUK** (PIN Unblocking Key):
            The PUK is used to unblock or reset the User PIN in case it's forgotten or entered incorrectly multiple times, leading to PIN lockout.
            It's a backup mechanism to regain access to the YubiKey by resetting the User PIN.
            The PUK is not intended for regular use and should be stored securely, as it allows resetting the User PIN and gaining access to the YubiKey.

        Default: Serial number of your Key - XXXXXXXX

- **Management Key** (MGM Key):
            The Management Key (also referred to as the MGM Key) is used for managing some specific features and settings on the YubiKey.
            It's used for certain YubiKey configurations and is less commonly used than the User and Admin PINs.

        Default: 010203040506070801020304050607080102030405060708

<mark>You must set your own PINs and PUK before use! Change the default!</mark>

## Yubico software overview 

**YubiKey minidriver**

The YubiKey Minidriver is a component designed to enable smart card functionality on Windows systems while using a YubiKey. It extends the capabilities of the YubiKey smart card functionality to work seamlessly within the Windows ecosystem, providing enhanced security features and allowing YubiKeys to be used as smart cards for authentication and cryptographic operations.
You need to install it on Windows systems for using your key. You can use this [guide](https://support.yubico.com/hc/en-us/articles/360015654560#Manual-Install)

**NB!** When deploying the Minidriver to remote servers <mark>where the YubiKey cannot be physically inserted</mark>, a legacy node must be created to load the minidriver. To do so, install the minidriver with the INSTALL_LEGACY_NODE=1 option set:

`msiexec /i YubiKey-Minidriver-4.1.1.210-x64.msi INSTALL_LEGACY_NODE=1 /quiet`

Installing the MSI with the Legacy Node option enabled on servers will prevent the Smart Card Logon Over RDP Fails with "Requested Key Container is not Available" error.

---

**YubiKey Manager:**

YubiKey Manager is a desktop application available for Windows, macOS, and Linux. It provides a graphical interface for managing and configuring YubiKeys, including settings for PINs, applications, and more. Users can perform tasks like changing PINs, enabling or disabling YubiKey functionalities, and configuring slot configurations with GUI.

---

**Yubico Authenticator:**

The Yubico Authenticator app is available for desktop and mobile platforms (Windows, macOS, iOS, Android). It functions as a TOTP (Time-Based One-Time Password) generator and supports storing OTP (One-Time Password) credentials on the YubiKey.

---

**YubiKey Personalization Tool** (This project is supported but no longer under active development):

The YubiKey Personalization Tool is a utility primarily used for advanced configuration and customization of YubiKeys. It allows users to configure various settings, such as setting static passwords, enabling or disabling OTP/HOTP, and changing YubiKey modes.

---

**YubiKey Manager CLI (Command-Line Interface):**

YubiKey Manager CLI aka ykman is a command-line tool available for managing YubiKeys. It provides a command-line interface for advanced users to perform tasks related to YubiKey configuration and management.
https://developers.yubico.com/yubikey-manager/

---

**Yubico Login for Windows: (Not to be confused with Yubico Authenticator)**

This software enables users to log in to Windows computers using a YubiKey for two-factor authentication.

## Usage exaples 
### Example 1: Using Yubikey for auth on remote SSH servers on Windows
1. First step is to generate the key and a certificate. This is where you have 2 options:
- generate on your PC and later import to your Yubikey (if you need your privatekey elsewhere)
- generate key&cert on Yubikey directly with a secure option not to share private key as it`s cannot be exported. For this task you can use gui or cli YubiKey Manager.

2. Second you have to export your public key and do some crypto magic to make it suitable for SSH server systems file .ssh\authorized_keys.
General steps are (use [this](https://gist.github.com/shanewholloway/15a0f5dda96b5d328d121f255f012ebf) GIST and make shell script):

`ykman piv export-certificate 9a my-public-certificate.pem`

Using ykman, the YubiKey Manager Command Line Interface (CLI), to export a certificate from slot 9a on the YubiKey with PIV (Personal Identity Verification) functionality. It exports the certificate and saves it to a file named my-public-certificate.pem.

`openssl x509 -in my-public-certificate.pem -noout -pubkey > my-public-key.pem`

This command uses openssl to extract the public key from the X.509 certificate stored in the file *my-public-certificate.pem*. The *-noout* flag indicates that no certificate output should be printed, and *-pubkey* specifies to output the public key only. The extracted public key is then redirected *(>)* and saved to a file named *my-public-key.pem*.

`ssh-keygen -i -m pkcs8 -f ./my-public-key.pem > id_for_your_yubikey_cert.pub`

Next ssh-keygen is used to convert the extracted public key (my-public-key.pem) into an OpenSSH public key format *(id_for_your_yubikey_cert.pub)* that can be used with SSH. The *-i* flag specifies that the input file is in SSH2-compatible format. *-m pkcs8* specifies the format for the key to import/export as PKCS#8. 
> PKCS#8 stands for Public-Key Cryptography Standards #8. It's a standard syntax used for storing and exchanging private key information ?in a secure manner, defined in the PKCS series by RSA Laboratories

The output of this conversion is redirected *(>)* and saved to a file named *id_for_your_yubikey_cert.pub*.

3. The id_for_your_yubikey_cert.pub file will contain all you need to paste in .ssh\authorized_keys file on server side. Do this to all systems you plan to access with Yubikey.
4. To work correctly on Windows OpenSSH you must follow [guide](https://support.yubico.com/hc/en-us/articles/360021606180-Using-YubiKey-PIV-with-Windows-native-SSH-client) and:
- instal the [Yubiko PIV Tool](https://developers.yubico.com/yubico-piv-tool/) - provides dll lib *libykcs11.dll* needed 
- check the verison of OpenSSH by running ssh -V in PowerShell. Your output should resemble OpenSSH_for_Windows_8.1p1, LibreSSL 3.0.2 or higher.
- create or modify *config* file with the line 
`'PKCS11Provider "C:\Program Files\Yubico\Yubico PIV Tool\bin\libykcs11.dll"'`
to allow OpenSSH use the physical keys like Yubikey is.
5. Just open Terminal and do ssh to server providing your pin.

Unfortunaly not all SSH windows programs work with PKCS11 so you cannot use them with key - so it may be unconvienent in some cases.
