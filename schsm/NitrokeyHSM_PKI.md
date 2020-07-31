# Creating a PKI with a Nitrokey HSM

## Hardware
The [Nitrokey HSM](https://shop.nitrokey.com/shop/product/nk-hsm-2-nitrokey-hsm-2-7)
is a [SmartCard-HSM](https://www.smartcard-hsm.com/)
with a USB connector.

Most of it's functionality can be accessed through a 
[PKCS#11](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/errata01/os/pkcs11-base-v2.40-errata01-os-complete.html)
interface. Note that in PKCS#1 terminology, the HSM is called the token.

See [https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM](https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM) for an overview.

## Software

### How To Install
On OpenSUSE Linux, all SW packages mentioned below can be installed from the standard repositories.

On Windows
- OpenSSL is best compiled from the [sources](https://www.openssl.org/source/)
- OpenSC: an installer is offered at [OpenSC](https://github.com/OpenSC/OpenSC/wiki)


### [sc-hsm-tool](https://htmlpreview.github.io/?https://github.com/OpenSC/OpenSC/blob/master/doc/tools/tools.html#sc-hsm-tool)
The sc-hsm-tool is provided by the [OpenSC](https://github.com/OpenSC/OpenSC/wiki) project.
It is used to initialize the SmartCard-HSM.

### [pkcs11-tool](https://htmlpreview.github.io/?https://github.com/OpenSC/OpenSC/blob/master/doc/tools/tools.html#pkcs11-tool)
The pkcs11-tool is provided by the [OpenSC](https://github.com/OpenSC/OpenSC/wiki) project.
It allows to manage and use keys on the SmartCard-HSM.


### [libp11](https://github.com/OpenSC/libp11)
libp11 is the PKCS#11 library offered by OpenSC.
It offers access to SmartCard HSM's like the Nitrokey HSM

### [engine_pkcs11](https://github.com/OpenSC/engine_pkcs11)
The engine_pkcs11 is a PKCS#11 engine for OpenSSL


## Step-By-Step

### Some commands to test the Nitrokey
It may be a good idea to try these commands before and after initialization or creation or manipulation of an object
* ``pkcs11-tool.exe -I`` Display general token information
* ``pkcs11-tool.exe -T`` List slots with tokens
* ``pkcs11-tool.exe -L`` Display a list of available slots on the token.
* ``pkcs11-tool.exe -M`` Display a list of mechanisms supported by the token.
* ``pkcs11-tool.exe -O`` Display a list of objects. Private keys are only displayed when used with either --login or --pin.
* ``pkcs11-tool.exe -t`` Perform some tests on the token. This option is most useful when used with either --login or --pin.


### Initialize the Nitrokey
Main Reference: https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM#initialize-the-device


Either use sc-hsm-tool: 
* ``sc-hsm-tool --initialize --so-pin XXXXXXXXXXXXXXXXX --pin YYYYYY --label "LuckyCow1"``

Or use pkcs11-tool (don't ask me why the same PIN has to be provided in two arguments)
* ``pkcs11-tool --init-token --init-pin --so-pin=XXXXXXXXXXXXXXXXX --new-pin=YYYYYY --pin=YYYYYY --label="LuckyCow1"``

While the SO-PIN and the PIN are mandatory in the above commands, the label is optional. It may be useful one token among others.

[Background on PIN vs SO-PIN](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759984):
 PKCS#11 defines 2 types of users or roles:
* The Security Officer - authenticated by the SO-PIN - can only initialize the token and set the user PIN
* The User - authenticated by the (user) PIN - can use and manipulate all private objects on the token

Note that for ``pkcs11-tool``, the PIN can be either provided by command line (option ``--pin``) or by stdin/prompt (option ``-l`` or ``--login``).

### Create a private key
References:
* https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM#generate-key-pair
* https://github.com/OpenSC/OpenSC/wiki/Using-pkcs11-tool-and-OpenSSL

Create a RSA key (if the ``--usage-sign`` option is left out, the key usage will be less restricted)
* ``pkcs11-tool.exe -l --keypairgen --key-type rsa:2048 --id 111 --usage-sign --label "Key1"``

Export the public key

Sign something with the RSA key (check ``pkcs11-tool.exe -M`` for the available signature mechanisms)
* ``pkcs11-tool.exe -s --id 111 -m SHA256-RSA-PKCS --input-file to_be_signed.txt --output-file sig_SHA_256_RSA_PKCS.sig``
* ``pkcs11-tool.exe -s --id 111 -m SHA256-RSA-PKCS-PSS --input-file to_be_signed.txt --output-file sig_SHA_256_RSA_PKCS_PSS.sig``

Delete a key
* ``pkcs11-tool.exe -l --delete-object --type privkey --id 111``


### Sign something using OpenSSL and the PKCS#11 engine
Just to demnonstrate the basic usage of the engine and how to identify keys when using openssl

https://www.nitrokey.com/de/documentation/applications#p:nitrokey-hsm&os:windows&a:pki--certificate-authority-ca

### Create a root certificate

### Sign a CSR to create a client certificate



## Appendix 1: OpenSSL configuration

### OpenSSL PKCS#11 engine configuration basics

### OpenSSL CA configuration basics

??? https://jamielinux.com/docs/openssl-certificate-authority/introduction.html

### OpenSSL CSR configuration basics

### OpenSSL full configuration


## Appendix 2: building the OpenSSL PKCS#11 engine on Windows

## Appendix 3: (Further) References
* https://github.com/OpenSC/libp11
* https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM
* https://github.com/OpenSC/OpenSC/wiki/Using-pkcs11-tool-and-OpenSSL
* https://framkant.org/2018/04/smartcard-hsm-backed-openssl-ca/
* https://jamielinux.com/docs/openssl-certificate-authority/index.html
* http://cedric.dufour.name/blah/IT/SmartCardsHowto.html
