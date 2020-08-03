# Creating a PKI with a Nitrokey HSM

## Hardware
The [Nitrokey HSM](https://shop.nitrokey.com/shop/product/nk-hsm-2-nitrokey-hsm-2-7)
is a [SmartCard-HSM](https://www.smartcard-hsm.com/)
integrated with a USB card reader.

Most of it's functionality can be accessed through a 
[PKCS#11](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/errata01/os/pkcs11-base-v2.40-errata01-os-complete.html)
interface. Note that in PKCS#11 terminology, the HSM is called the token which itself is in a slot (of a card reader).

See [https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM](https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM) for an overview.

## Software

### How To Install
On OpenSUSE Leap 15.2 Linux, all SW packages mentioned below can be installed from the standard repositories.

On Windows
- OpenSSL: is best compiled from the [sources](https://www.openssl.org/source/)
- OpenSC: an installer is offered at [OpenSC](https://github.com/OpenSC/OpenSC/wiki)


### [sc-hsm-tool](https://htmlpreview.github.io/?https://github.com/OpenSC/OpenSC/blob/master/doc/tools/tools.html#sc-hsm-tool)
The sc-hsm-tool is provided by the [OpenSC](https://github.com/OpenSC/OpenSC/wiki) project.
It is used to initialize the SmartCard-HSM.

### [pkcs11-tool](https://htmlpreview.github.io/?https://github.com/OpenSC/OpenSC/blob/master/doc/tools/tools.html#pkcs11-tool)
The pkcs11-tool is provided by the [OpenSC](https://github.com/OpenSC/OpenSC/wiki) project.
It allows to manage and use keys on the SmartCard-HSM.


### [opensc-pkcs11.so / opensc-pkcs11.dll](https://github.com/OpenSC/OpenSC)
This is the PKCS#11 library from the OpenSC.
The supported HW is listed at https://github.com/OpenSC/OpenSC/wiki/Supported-hardware-(smart-cards-and-USB-tokens).


### [libp11](https://github.com/OpenSC/libp11)
libp11 is a library offered by OpenSC, providing a providing a higher-level (compared to the PKCS#11 library) interface to access PKCS#11 objects
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

While the SO-PIN and the PIN are mandatory in the above commands, the label is optional. It may be useful to distinguish one token among others.

[Background on PIN vs SO-PIN](http://docs.oasis-open.org/pkcs11/pkcs11-ug/v2.40/cn02/pkcs11-ug-v2.40-cn02.html#_Toc406759984):
 PKCS#11 defines 2 types of users or roles:
* The Security Officer - authenticated by the SO-PIN - can only initialize the token and set the user PIN
* The User - authenticated by the (user) PIN - can use and manipulate all private objects on the token

Note that for ``pkcs11-tool``, the PIN can be either provided by command line (option ``--pin``) or by stdin/prompt (option ``-l`` or ``--login``).

### Create a private key
References:
* https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM#generate-key-pair
* https://github.com/OpenSC/OpenSC/wiki/Using-pkcs11-tool-and-OpenSSL

Create a RSA keypair (if the ``--usage-sign`` option is left out, the key usage will be less restricted)
* ``pkcs11-tool.exe -l --keypairgen --key-type rsa:2048 --id 111 --usage-sign --label "Key1"``

Export the public key (in ASN.1 DER SubjectPublicKeyInfo according [RFC5280](https://tools.ietf.org/html/rfc5280)) and check with openssl
* ``pkcs11-tool -l --id 111 --read-object --type pubkey --output-file pubkey_111.der``
* ``openssl rsa --pubin --inform der --in pubkey_111.der --noout --text``
* ``openssl asn1parse --inform der --in pubkey_111.der``

Sign something with the RSA key (check ``pkcs11-tool.exe -M`` for the available signature mechanisms)
* ``pkcs11-tool.exe -s --id 111 -m SHA256-RSA-PKCS --input-file to_be_signed.txt --output-file sig_SHA_256_RSA_PKCS.sig``
* ``pkcs11-tool.exe -s --id 111 -m SHA256-RSA-PKCS-PSS --input-file to_be_signed.txt --output-file sig_SHA_256_RSA_PKCS_PSS.sig``

Delete a key
* ``pkcs11-tool.exe -l --delete-object --type privkey --id 111``

### Check if OpenSSL finds the pkcs11 engine
Call [`openssl engine`](https://www.openssl.org/docs/man1.1.1/man1/engine.html) to find out whether OpenSSL finds the pkcs11 engine
* ``openssl engine pkcs11 -c -vvvv``

On my OpenSUSE Leap 15.2 installation, this works already - probably because the PKCS#11 engine and library are installed in some default directories.

If you need or want to have the configuration more explicit, then use something like this as this `hsm.conf`

    # PKCS11 engine config
    openssl_conf = openssl_def

    [openssl_def]
    engines = engine_section
   
    [req]
    distinguished_name = req_distinguished_name
   
    [req_distinguished_name]
    # empty
    
    [ca]
    default_ca = CA_default  # The default ca section

    [CA_default]
    new_certs_dir = .
    database = ./db
    default_md = sha256
    email_in_dn = no
    rand_serial = yes
    default_days = 365 
    policy = CA_policy # The policy defines where to take subject DN fields from

    [CA_policy]
    # minimalistic policy
    commonName = supplied

    [engine_section]
    pkcs11 = pkcs11_section

    [pkcs11_section]
    engine_id = pkcs11
    dynamic_path = /usr/lib64/engines-1.1/pkcs11.so
    MODULE_PATH = /usr/lib64/opensc-pkcs11.so
    init = 0
    PIN = YYYYYY

And then try again, with the hsm.conf specified by the OPENSSL_CONF environment variable
* ``OPENSSL_CONF=./hsm.conf openssl engine pkcs11 -c -vvvv``


### Sign something using OpenSSL and the PKCS#11 engine
Note that keys must be defined in form of a [RFC7512](https://tools.ietf.org/html/rfc7512) 
as used in the [libp11 example](https://github.com/OpenSC/libp11#using-p11tool-and-openssl-from-the-command-line)
(with some restrictions applied regarding the YubiKey/SmartCard HSM
[1](https://github.com/OpenSC/engine_pkcs11/issues/28)
[2](https://github.com/OpenSC/OpenSC/issues/1429)
[3](https://support.nitrokey.com/t/nitrokey-hsm-as-ca-for-signing-csrs/1107)
and maybe some slightly different flavor when using the Yubikey
[Y](https://developers.yubico.com/YubiHSM2/Usage_Guides/OpenSSL_with_libp11.html) )

With that given, we can call [`openssl rsautl`](https://www.openssl.org/docs/man1.1.1/man1/rsautl.html) like 
* ``openssl rsautl -engine pkcs11 -keyform engine -inkey "pkcs11:object=Key1;type=private;pin-value=YYYYYY" -sign -in aaa.txt -out aaa.sig``

or - if you want to be prompted for the PIN rather than providing it on the command line:
* ``openssl rsautl -engine pkcs11 -keyform engine -inkey "pkcs11:object=Key1;type=private" -sign -in aaa.txt -out aab.sig``

or - if you explicitly want to specify the OpenSSL config file:
* ``OPENSSL_CONF=./hsm.conf openssl rsautl -engine pkcs11 -keyform engine -inkey "pkcs11:object=Key1;type=private" -sign -in aaa.txt -out aac.sig``

or - if you want to identify the key by its ID insteead of its name (note the %XX%XX notation as key ID's are actually represented by UTF/HEX values):
* ``OPENSSL_CONF=./hsm.conf openssl rsautl -engine pkcs11 -keyform engine -inkey "pkcs11:id=%01%11;type=private" -sign -in aaa.txt -out aal.sig``

or - if you prefer openssl [`openssl dgst`](https://www.openssl.org/docs/man1.1.1/man1/dgst.html) to sign:
* ``OPENSSL_CONF=./hsm.conf openssl dgst -engine pkcs11 -keyform engine -sign "pkcs11:id=%01%11;type=private" -sha256 -out aam.sig aaa.txt``

or - same but with explicit sigopt:
* ``OPENSSL_CONF=./hsm.conf openssl dgst -engine pkcs11 -keyform engine -sign "pkcs11:id=%01%11;type=private" -sha256 -sigopt rsa_padding_mode:pkcs1 -out aan.sig aaa.txt``
* ``OPENSSL_CONF=./hsm.conf openssl dgst -engine pkcs11 -keyform engine -sign "pkcs11:id=%01%11;type=private" -sha256 -sigopt rsa_padding_mode:pss -out aao.sig aaa.txt``

or - if you prefer openssl [`openssl pkeyutl`](https://www.openssl.org/docs/man1.1.1/man1/pkeyutl.html) to sign:
* ``OPENSSL_CONF=./hsm.conf openssl pkeyutl -engine pkcs11 -keyform engine -inkey "pkcs11:id=%01%11;type=private" -sign -in aaa.txt -out aaq.sig``
* ``OPENSSL_CONF=./hsm.conf openssl pkeyutl -engine pkcs11 -keyform engine -inkey "pkcs11:id=%01%11;type=private" -sign -in aaa.txt -out aar.sig -pkeyopt rsa_padding_mode:pkcs1``

Further examples: see https://www.nitrokey.com/de/documentation/applications#p:nitrokey-hsm&os:windows&a:pki--certificate-authority-ca

### Create a root certificate

### Sign a CSR to create a client certificate



## Appendix 1: OpenSSL configuration
Details on the OpenSSL configuration file are provided in the following man pages:
* https://www.openssl.org/docs/man1.1.1/man1/openssl.html
* https://www.openssl.org/docs/man1.1.1/man5/config.html

The location of the OpenSSL configuration file is determined  as decribed in [`openssl conf`](https://www.openssl.org/docs/man1.1.1/man5/config.html) by 
* the compile time variable `OPENSSLDIR` pointing to a directory containing the file `openssl.cnf`
  * note that `openssl version -a` shows all compile time settings
* the environment variable `OPENSSL_CONF`
* the `-config` swithc which is present for some commands

Some commands support specific configuration options:
* https://www.openssl.org/docs/man1.1.1/man1/ca.html
* https://www.openssl.org/docs/man1.1.1/man1/req.html
* https://www.openssl.org/docs/man1.1.1/man1/x509.html

### OpenSSL PKCS#11 engine configuration basics
The engine configuration is described in the [openssl config](https://www.openssl.org/docs/man1.1.1/man5/config.html) man page.
Note that only the parameters `engine_id` and `dynamic_path` are common. 
All other parameters are engine-specific and can be queried by calling `openssl engine pkcs11 -c -vvvv`.


### OpenSSL req configuration basics
The [`openssl req`](https://www.openssl.org/docs/man1.1.1/man1/req.html) command can be used both to create CSR's (certificate sign requests) and self-signed certificates (if called with the `-x509` option).

The minimum requirement regarding the OpenSSL configuration file is the existence of the `[req]` section and containing a `distinguished_name` definition.
The `distinguished_name` itself must refer to another section which may be empty.

The absolute minimalistic way to create a self-signed root certificate is like this (with a rootkey already created by `openssl genrsa -out root_key.pem 2048`)
* ``openssl req -key root_key.pem -x509 -out "root_cert.pem" -subj "/CN=the root"``

To create a CSR (with a key already created by `openssl genrsa -out leaf_key.pem 2048`), you would correspondingly call
* ``openssl req -key leaf_key.pem -new -out "leaf_cert.csr" -subj "/CN=the leaf"``

In practice, generally all the certificate and CSR details a set via the OpenSSL configuration file.

### OpenSSL ca configuration basics
The [`openssl ca`](https://www.openssl.org/docs/man1.1.1/man1/ca.html) command can be used as a minimal CA (certificate authority) application.

The minimum requirement regarding the OpenSSL configuration file is the existence of the `[ca]`section and some parameters as in the example above.

A minimalistic way to create acertificate based on a CSR and a root would be like this:
* ``openssl ca -in leaf_cert.csr -out leaf_cert.pem -keyfile root_key.pem -cert root_cert.pem -batch``



### OpenSSL full configuration
See example above


## Appendix 2: building the OpenSSL PKCS#11 engine on Windows

## Appendix 3: (Further) References
* https://github.com/OpenSC/libp11
* https://github.com/OpenSC/OpenSC/wiki/SmartCardHSM
* https://github.com/OpenSC/OpenSC/wiki/Using-pkcs11-tool-and-OpenSSL
* https://framkant.org/2018/04/smartcard-hsm-backed-openssl-ca/
* https://jamielinux.com/docs/openssl-certificate-authority/index.html
* http://cedric.dufour.name/blah/IT/SmartCardsHowto.html
* https://raymii.org/s/articles/Get_Started_With_The_Nitrokey_HSM.html
* https://developers.yubico.com/yubico-piv-tool/YKCS11/Supported_applications/pkcs11tool.html
  * https://www.yubico.com/products/services-software/download/smart-card-drivers-tools/
 
