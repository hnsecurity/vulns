# HN Security Advisory - https://hnsecurity.it/

* Title: Multiple race conditions in Keycloak's anti-brute force mechanism
* Products:
  * Keycloak <= 22.0.11  
  * Keycloak <= 24.0.6  
  * Keycloak <= 25.0.2  
* Author: Ema Srdoc <ema.srdoc at hnsecurity.it> and Maurizio Agazzini <maurizio.agazzini at hnsecurity.it>
* Date: 2024-10-30
* CVE Names and Vendor CVSS Scores:  
  CVE-2024-4629: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:N
* Advisory URLs:  
  https://github.com/hnsecurity/vulns/blob/main/HNS-2024-09-Keycloak.md

# Description

Race conditions in the context of web applications, particularly in authentication processes involving user credentials and one-time passwords (OTPs), pose significant security risks.

In a typical authentication scenario, when a user submits their credentials (username and password) through a form, the server validates these credentials before granting access. However, in the presence of race conditions, multiple requests from the same user or different users may arrive at the server simultaneously or in rapid succession.

Keycloak implements an anti-brute force mechanism (which is not enabled by default) which should limit the attack attempts by locking out user accounts after a configured number of incorrect login attempts:

- https://www.keycloak.org/docs/latest/server_admin/#password-guess-brute-force-attacks

Based on our analysis, the software does not implement controls to block attacks of this kind; therefore, it's possible to send multiple requests that are processed simultaneously, completely bypassing the anti-brute-force mechanism. Furthermore, when the account is locked, active sessions are not invalidated.

The issue has been verified both on the username and password-based login and on the OTP request. No checks have been performed on the "Quick Login Check Milliseconds" functionality, but we expect it to be affected by the same issue.

The configuration used for testing was set to lockout user accounts after 3 incorrect login attempts.

Further details on the technique used are available here:
 - https://portswigger.net/research/smashing-the-state-machine
 - https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local
 - https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack

# Affected platforms

All Keycloak versions up to and including version 22.0.11, 24.0.6, 25.0.2 are affected.

# Disclosure timeline

18/04/2024 - First communication with details.  
18/04/2024 - Response: we can't see the details.  
18/04/2024 - Send details again as an attachment.  
02/05/2024 - Request for a follow-up.  
09/05/2024 - Response: CVE-2024-4629 assigned, fix in backlog.  
24/05/2024 - Request for a roadmap of the fix.  
22/08/2024 - Request for a follow-up.  
26/08/2024 - Response: already fixed and published as a [public issue](https://github.com/keycloak/keycloak/issues/31726)  
03/09/2024 - [CVE-2024-4629](https://access.redhat.com/security/cve/CVE-2024-4629) is published.  
