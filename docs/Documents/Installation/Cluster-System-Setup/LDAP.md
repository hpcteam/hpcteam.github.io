# OpenLDAP Installation and Configuration Guide 

---

# 1. Introduction to LDAP

## What is LDAP?

LDAP (Lightweight Directory Access Protocol) is a protocol used to access and manage directory information services over a network.

A directory is a specialized database optimized for:

* Read-heavy operations
* Hierarchical data structure
* Fast lookups

---

## What LDAP Can Do

LDAP is mainly used for:

* Centralized authentication (users login across multiple systems)
* Storing user and group information
* Access control
* Integration with services (SSSD, NFS, HPC schedulers)

In HPC environments:

* Single user identity across login, compute, GPU nodes
* Consistent UID/GID mapping
* Centralized user management

---

# 2. LDAP Architecture

LDAP follows a tree structure called:

👉 DIT (Directory Information Tree)

Example:

```
dc=mycluster,dc=local
 ├── ou=People
 │    ├── uid=user1
 │    └── uid=user2
 └── ou=Group
      └── cn=hpcusers
```

---

## Key Terms

### DN (Distinguished Name)

Full path to an object:

```
uid=user1,ou=People,dc=mycluster,dc=local
```

---

### DC (Domain Component)

Represents domain name:

```
dc=mycluster,dc=local
```

---

### OU (Organizational Unit)

Logical grouping:

```
ou=People
ou=Group
```

---

### CN (Common Name)

Name of object (group or general name):

```
cn=hpcusers
```

---

### UID

User login name:

```
uid=user1
```

---

# 3. Installation (RHEL 8)

## Required Packages

```
yum install -y openldap openldap-servers openldap-clients
```

---

## Start Service

```
systemctl enable --now slapd
systemctl status slapd
```


---

# 4. Initial Server Configuration

## Set Root Password

```
slappasswd
```

Copy generated hash.

---

## Configure Root DN

Create file:

```
vi root.ldif
```

```
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=mycluster,dc=local


dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=mycluster,dc=local


dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}HASH
```

Apply:

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f  root.ldif
```

---

# 5. Schema Configuration

## What is Schema?

Schema defines:

* Object classes
* Attributes

Without schema:

* LDAP cannot understand user/group structure

---

## Important Schemas

| Schema        | Purpose          |
| ------------- | ---------------- |
| core          | Base schema      |
| cosine        | Extra attributes |
| inetorgperson | User objects     |

---

## Load Required Schemas

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

---

# 6. Create Base Directory (DIT)

```
vi base.ldif
```

```
dn: dc=mycluster,dc=local
objectClass: top
objectClass: dcObject
objectClass: organization
o: mycluster
dc: mycluster


dn: ou=People,dc=mycluster,dc=local
objectClass: organizationalUnit
ou: People


dn: ou=Group,dc=mycluster,dc=local
objectClass: organizationalUnit
ou: Group
```

Apply:

```
ldapadd -x -D "cn=admin,dc=mycluster,dc=local" -W -f base.ldif
```

---

# 7. Create Group

```
dn: cn=hpcusers,ou=Group,dc=mycluster,dc=local
objectClass: top
objectClass: posixGroup
cn: hpcusers
gidNumber: 10001
```


---

# 8. Create User

```
dn: uid=user1,ou=People,dc=mycluster,dc=local
objectClass: top
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: user1
sn: user1
uid: user1
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/user1
loginShell: /bin/bash
userPassword: password123
```

---

# 9. Verification

## Check entries

```
ldapsearch -x -b "dc=mycluster,dc=local"
```

---

# 10. Client Configuration (SSSD)

## Install

```
yum install -y sssd sssd-ldap oddjob oddjob-mkhomedir
```

---

## Configure

```
vi /etc/sssd/sssd.conf
```

```
[sssd]
services = nss, pam
config_file_version = 2
domains = mycluster.local

[domain/mycluster.local]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap.mycluster.local
ldap_search_base = dc=mycluster,dc=local
ldap_id_use_start_tls = false
ldap_tls_reqcert = never
```

---

## Enable

```
chmod 600 /etc/sssd/sssd.conf
systemctl enable --now sssd
```

---

# 11. Testing

```
id user1
getent passwd user1
su - user1
```

---

# 12. HPC Integration Notes

* Same LDAP used across all nodes
* UID/GID must be consistent
* Use shared home (NFS/Lustre)
* Integrate with scheduler (PBS/Slurm)

---

# 13. Common Issues

* LDIF formatting errors
* Missing schema
* TLS misconfiguration
* SSSD cache issues

---

# 14. TLS (SSL) Configuration

## What is TLS in LDAP?

TLS (Transport Layer Security) is used to secure LDAP communication by encrypting data such as usernames and passwords.

---

## Why TLS is Required

* Prevents password exposure over network
* Secures authentication
* Required for production environments

---

## Types of Secure LDAP

| Type     | Port | Description                    |
| -------- | ---- | ------------------------------ |
| STARTTLS | 389  | Upgrades normal LDAP to secure |
| LDAPS    | 636  | Direct secure LDAP connection  |

---

## TLS Configuration Steps (Server)

### 1. Generate Certificate

```
mkdir -p /etc/openldap/certs
cd /etc/openldap/certs
openssl genrsa -out ldap.key 2048
openssl req -new -x509 -key ldap.key -out ldap.crt -days 365
```

> Common Name (CN) must match server hostname (e.g., ldap.mycluster.local)

---

### 2. Set Permissions

```
chown ldap:ldap ldap.*
chmod 600 ldap.key
```

---

### 3. Configure LDAP for TLS

```
vi tls.ldif
```

```
dn: cn=config
changetype: modify
add: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/ldap.crt
-
add: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/ldap.key
```

Apply:

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f tls.ldif
```

---

### 4. Restart Service

```
systemctl restart slapd
```

---

## TLS Configuration (Client - SSSD)

```
vi /etc/sssd/sssd.conf
```

```
ldap_uri = ldap://ldap.mycluster.local
ldap_id_use_start_tls = true
ldap_tls_reqcert = demand
ldap_tls_cacert = /etc/openldap/certs/ldap.crt
```

---

## Testing TLS

```
ldapsearch -x -ZZ -H ldap://ldap.mycluster.local -b "dc=mycluster,dc=local"
```

---

## Temporary Disable TLS (Lab Only)

```
ldap_id_use_start_tls = false
ldap_tls_reqcert = never
ldap_auth_disable_tls_never_use_in_production = true
```

> ⚠️ Not recommended for production

---

# 15. Important LDAP Commands 

## 1. Check LDAP Service Status

```
systemctl status slapd
```

## 2. Start/Restart LDAP Service

```
systemctl restart slapd
```

## 3. Search Entire LDAP Directory

```
ldapsearch -x -b "dc=mycluster,dc=local"
```

## 4. Add Entry (User/Group/Base)

```
ldapadd -x -D "cn=admin,dc=mycluster,dc=local" -W -f file.ldif
```

## 5. Delete Entry

```
ldapdelete -x -D "cn=admin,dc=mycluster,dc=local" -W "dn"
```

## 6. Modify Entry

```
ldapmodify -x -D "cn=admin,dc=mycluster,dc=local" -W -f file.ldif
```

## 7. Create User (via LDIF)

```
dn: uid=user1,ou=People,dc=mycluster,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: user1
sn: user1
uid: user1
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/user1
loginShell: /bin/bash
userPassword: password123
```

## 8. Create Group (via LDIF)

```
dn: cn=hpcusers,ou=Group,dc=mycluster,dc=local
objectClass: posixGroup
cn: hpcusers
gidNumber: 10001
memberUid: user1
```

## 9. Check Specific User

```
ldapsearch -x -b "dc=mycluster,dc=local" uid=user1
```

## 10. Check Specific Group

```
ldapsearch -x -b "dc=mycluster,dc=local" cn=hpcusers
```

## 11. Set/Reset User Password (Admin)

```
ldappasswd -x -D "cn=admin,dc=mycluster,dc=local" -W -S "uid=user1,ou=People,dc=mycluster,dc=local"
```

## 12. Generate Password Hash

```
slappasswd
```

## 13. Test LDAP Connectivity

```
ldapsearch -x -H ldap://ldap.mycluster.local -b "dc=mycluster,dc=local"
```

## 14. Check User from Client (SSSD)

```
id user1
getent passwd user1
```

## 15. Clear SSSD Cache

```
sss_cache -E
systemctl restart sssd
```

---


LDAP provides centralized identity management for HPC clusters, ensuring consistency, scalability, and simplified administration.

