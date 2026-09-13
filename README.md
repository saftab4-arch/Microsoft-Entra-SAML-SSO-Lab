


Microsoft Entra ID – Enterprise SAML SSO & Identity Access Lab
Hands-on Microsoft Entra ID lab implementing and troubleshooting SAML 2.0 Single Sign-On (SSO) with Microsoft Entra ID as the Identity Provider (IdP) and Microsoft Entra SAML Toolkit as the Service Provider (SP).

The goal was not only to make SSO work, but to understand the complete authentication and authorization flow and troubleshoot realistic identity-access failures.

Objectives
Configure cloud users and security groups in Microsoft Entra ID

Configure an Entra Enterprise Application

Implement group-based application access

Configure SAML 2.0 SSO

Understand the Identity Provider (IdP) and Service Provider (SP) relationship

Configure Entity ID, Sign-on URL, and ACS/Reply URL

Configure NameID and SAML claims

Configure SAML signing certificate trust

Test SP-initiated SSO

Validate MFA / Conditional Access behavior

Review Microsoft Entra sign-in logs

Troubleshoot authorization, user-matching, NameID, and Reply URL failures

Understand how SAML authentication differs from application provisioning

Understand SCIM and Just-In-Time (JIT) provisioning concepts

Architecture
User
  |
  v
Service Provider (Microsoft Entra SAML Toolkit)
  |
  | SP-initiated SAML authentication request
  v
Microsoft Entra ID (Identity Provider)
  |
  | Authentication + MFA / Conditional Access
  |
  | Authorization check (Enterprise App assignment)
  |
  | Signed SAML assertion
  v
ACS / Reply URL on Service Provider
  |
  | Validate assertion + identify user
  v
Application Session
A successful Microsoft Entra authentication does not automatically mean the user receives application access.

The flow involves multiple layers:

Authentication – Entra verifies who the user is.

Conditional Access / MFA – Entra evaluates authentication requirements.

Enterprise Application authorization – Entra verifies whether the user or their group is allowed to use the application.

SAML assertion – Entra creates and signs an assertion containing identity claims.

Service Provider validation – The application validates the assertion.

Application account matching/provisioning – The application must know how to map the federated identity to an application account.

Application session – The application finally grants access.

1. Identity and Group Foundation
Cloud users and security groups were created in Microsoft Entra ID.

Example application-access groups included:

SG-Sales

SG-IT

SG-HR

SG-Finance

SG-SaaS-App-Users

SG-VDI-Users

Instead of assigning large numbers of individual users directly to applications, access can be managed through groups.

User
  ↓
Security Group
  ↓
Enterprise Application Assignment
  ↓
Application Access
This creates a more scalable access-management model.

2. Enterprise Application
The Microsoft Entra SAML Toolkit Enterprise Application was added to the tenant.

Microsoft Entra ID acted as the:

Identity Provider (IdP)

The SAML Toolkit acted as the:

Service Provider (SP)

The Enterprise Application became the Entra-side representation of the SaaS application.

3. SAML 2.0 Configuration
SAML-based Single Sign-On was configured between Microsoft Entra ID and the SAML Toolkit.

Important SAML endpoints and identifiers included:

Entity ID
The Entity ID identifies the Service Provider to Microsoft Entra ID.

https://samltoolkit.azurewebsites.net
Sign-on URL
The Sign-on URL is used to begin the Service Provider initiated authentication flow.

Assertion Consumer Service (ACS) / Reply URL
The ACS endpoint is where Microsoft Entra ID sends the SAML response after authentication.

Example structure:

https://samltoolkit.azurewebsites.net/SAML/Consume/<ID>
This endpoint must match the Reply URL configured in the Enterprise Application.

4. Identity Provider Configuration
The Service Provider was configured with Microsoft Entra identity-provider information, including:

Microsoft Entra Login URL

Microsoft Entra Identifier

Logout URL

SAML signing certificate

This establishes trust between the application and Microsoft Entra ID.

Conceptually:

Service Provider trusts
        ↓
Microsoft Entra ID
        ↓
because the SAML assertion is digitally signed
5. SAML Signing Certificate
Microsoft Entra ID signs the SAML assertion using its SAML signing certificate.

The Service Provider uses the corresponding certificate information to verify that:

The assertion was issued by the trusted Identity Provider

The assertion was not modified after being issued

This prevents an attacker from simply creating a fake SAML assertion claiming to be another user.

6. NameID and Claims
The required NameID claim was configured using:

user.userprincipalname
with the Name identifier format:

Email address
The SAML assertion also included identity attributes such as:

Email address

Given name

Surname

User principal name

Conceptually:

Microsoft Entra User
        ↓
Claims generated
        ↓
Signed SAML Assertion
        ↓
Service Provider
        ↓
Application maps NameID to a user
The NameID is particularly important because the Service Provider can use it to determine which application account the authenticated identity belongs to.

7. Group-Based Application Access
The Enterprise Application was configured so access could be granted through security-group membership.

Instead of:

User → Application
User → Application
User → Application
User → Application
the scalable model is:

Users
  ↓
Security Group
  ↓
Enterprise Application
For example:

Sara
  ↓
SG-Sales
  ↓
Microsoft Entra SAML Toolkit
Adding Sara to the authorized group allowed Entra to authorize her for the Enterprise Application.

This demonstrated the distinction between:

Authentication

Is this really Sara?

and:

Authorization

Is Sara allowed to access this application?

8. SP-Initiated SSO Flow
The lab tested Service Provider initiated SSO.

The complete flow was:

1. User opens the Service Provider login URL
                     ↓
2. Service Provider redirects the browser to Microsoft Entra ID
                     ↓
3. Entra identifies the user
                     ↓
4. User authenticates
                     ↓
5. MFA / Conditional Access is evaluated
                     ↓
6. Entra checks Enterprise Application assignment
                     ↓
7. Entra creates a signed SAML assertion
                     ↓
8. Browser POSTs the SAML response to the ACS / Reply URL
                     ↓
9. Service Provider validates the assertion
                     ↓
10. Service Provider maps the NameID to an application user
                     ↓
11. Application session is created
9. Successful SSO Validation
Successful SSO was tested with authorized users.

The browser was redirected through Microsoft Entra authentication and returned to the SAML Toolkit.

Microsoft Entra sign-in logs showed successful authentication events for the Enterprise Application.

This verified that:

SP → Entra → Authentication → Authorization
   → Signed Assertion → ACS → Application
was functioning correctly.

Troubleshooting Scenarios
A major part of the lab involved intentionally breaking the SAML environment and troubleshooting the resulting failures.

Ticket 1 – User Not Assigned to Enterprise Application
Problem
A user attempted to access the SAML application but received:

AADSTS50105
Microsoft Entra reported that the administrator had configured the application to block users unless they were explicitly granted access.

Investigation
The Enterprise Application sign-in logs were reviewed.

The user successfully reached Microsoft Entra ID, but Entra rejected application access because the user was not directly assigned and was not a member of an authorized group.

Root Cause
The user was authenticated but not authorized for the Enterprise Application.

Resolution
The appropriate security group was assigned to the Enterprise Application and the user was added to that group.

Lesson
Successful authentication ≠ application authorization
A valid username, password, and MFA response do not automatically grant access to an Enterprise Application.

Ticket 2 – User Added to Authorized Group
Problem
A user who needed access was missing from the application-access group.

Investigation
The following relationship was checked:

User
  ↓
Group Membership
  ↓
Enterprise Application Assignment
The Enterprise Application already allowed the Sales security group, but the user was not a member.

Resolution
The user was added to the appropriate security group.

After the membership change, Entra allowed the SAML authentication flow to continue.

Lesson
Group-based application assignment allows administrators to manage access without individually assigning every employee to every application.

Ticket 3 – Entra Authentication Succeeds but Application Still Fails
Problem
The user successfully authenticated with Microsoft Entra ID and was authorized for the Enterprise Application, but the SAML Toolkit still failed to create an application session.

Investigation
The SAML assertion was successfully generated and sent to the Service Provider.

This proved that Entra authentication and authorization were working.

The issue occurred after the assertion reached the application.

Root Cause
The SAML Toolkit is a test application that requires a matching local user account.

The Service Provider could not map the incoming NameID to an existing local application user.

Resolution
A matching local user was created in the SAML Toolkit.

SSO then succeeded.

Real-World Lesson
This highlighted the difference between:

Federated Authentication
        vs
User Provisioning
Real SaaS applications commonly solve this with:

SCIM provisioning

Just-In-Time (JIT) provisioning

Automated account creation

Identity lifecycle integrations

With SCIM, a typical flow might be:

HR / Identity Source
        ↓
Microsoft Entra ID
        ↓
SCIM Provisioning
        ↓
SaaS Application User Created
        ↓
SAML SSO
Therefore, organizations with thousands of users normally do not manually create every SaaS account.

Ticket 4 – NameID / Claim Mismatch
Problem
Authentication reached the Service Provider, but the application could not correctly process the SAML identity.

Investigation
The Attributes & Claims configuration was reviewed.

The Service Provider expected the NameID in email-address format.

The required claim was configured as:

Name identifier format: Email address
Source attribute: user.userprincipalname
Root Cause
A mismatch between what the Service Provider expected and what Microsoft Entra ID sent prevented proper user mapping.

Resolution
The NameID configuration was restored to the expected format and source attribute.

Lesson
The Identity Provider and Service Provider must agree on the identity format.

Entra sends NameID
        ↓
Service Provider reads NameID
        ↓
SP maps NameID to application identity
If those values do not align, authentication can succeed at the IdP while application login still fails.

Ticket 5 – Incorrect ACS / Reply URL
Problem
The authentication attempt failed with:

AADSTS50011
Microsoft Entra reported that the Reply URL specified in the request did not match the Reply URLs configured for the application.

Investigation
The ACS URL generated by the Service Provider was compared with the Reply URL configured in Microsoft Entra ID.

The values did not match.

Root Cause
The SAML response was configured to return to an incorrect endpoint.

Conceptually:

Entra
  |
  | SAML Response
  v
Wrong Reply URL ✗
The Service Provider expects the SAML response at its specific ACS endpoint.

Resolution
The correct ACS URL was restored in the Enterprise Application's SAML configuration.

Entra
  |
  | Signed SAML Response
  v
Correct ACS / Reply URL
  |
  v
Service Provider ✓
SSO worked again after correcting the URL.

Lesson
The Reply URL is not simply the application's homepage.

It is the specific endpoint where the Service Provider is waiting to receive and process the SAML response.

Sign-In Log Investigation
Microsoft Entra sign-in logs were used throughout the troubleshooting process.

The logs helped identify:

Successful authentication

Failed authentication

Enterprise Application authorization failures

Conditional Access results

User principal name

Application involved

Error codes

Failure reasons

Request IDs

Correlation IDs

This demonstrated why Entra sign-in logs are one of the first places an administrator should investigate when troubleshooting SSO.

Important Concepts Learned
Identity Provider vs Service Provider
Microsoft Entra ID = Identity Provider (IdP)

SaaS Application = Service Provider (SP)
The IdP authenticates the identity.

The SP provides the application or service.

Authentication vs Authorization
Authentication
"Who are you?"

        ↓

Authorization
"Are you allowed to use this application?"
A user can successfully authenticate and still be denied access.

Authorization vs Provisioning
Even authorization does not guarantee that an application has an account for the user.

Authentication
      ↓
Authorization
      ↓
Provisioning / Account Mapping
      ↓
Application Access
These are separate identity-management functions.

SAML vs SCIM
SAML handles federated authentication.

User → Entra ID → SAML Assertion → SaaS Application
SCIM handles user lifecycle provisioning.

Entra ID → Create / Update / Disable User → SaaS Application
In a production environment, the two technologies can work together:

SCIM = make sure the account exists
SAML = authenticate the account
Troubleshooting Methodology
The lab reinforced a useful troubleshooting sequence:

1. Can the user authenticate?
        ↓
2. Did MFA / Conditional Access succeed?
        ↓
3. Is the user/group assigned to the Enterprise Application?
        ↓
4. Is the SAML configuration correct?
        ↓
5. Is the NameID / claim format correct?
        ↓
6. Is the Reply URL / ACS endpoint correct?
        ↓
7. Did Entra issue the assertion?
        ↓
8. Did the Service Provider accept it?
        ↓
9. Does the application have/match the user account?
This avoids treating every SSO problem as a password problem.

Screenshots
Screenshots collected during the lab can be stored in:

screenshots/
Recommended repository structure:

.
├── README.md
└── screenshots/
    ├── 01-entra-users-and-groups.png
    ├── 02-enterprise-application.png
    ├── 03-users-and-groups-assignment.png
    ├── 04-saml-basic-configuration.png
    ├── 05-saml-idp-configuration.png
    ├── 06-attributes-and-claims.png
    ├── 07-nameid-configuration.png
    ├── 08-saml-signing-certificate.png
    ├── 09-saml-toolkit-configuration.png
    ├── 10-successful-sso.png
    ├── 11-entra-signin-logs.png
    └── 12-signin-log-details.png
Example Markdown for embedding screenshots:

![SAML Configuration](screenshots/04-saml-basic-configuration.png)
Skills Practiced
Microsoft Entra ID

Enterprise Applications

SAML 2.0

Single Sign-On (SSO)

Identity Provider / Service Provider architecture

Security groups

Group-based application assignment

MFA

Conditional Access validation

SAML claims

NameID

SAML signing certificates

ACS / Reply URLs

Microsoft Entra sign-in logs

Identity troubleshooting

SaaS application authorization

SAML vs SCIM provisioning concepts

Final Takeaway
The biggest lesson from this lab was that enterprise SSO is more than entering a username and password.

A working application login can depend on the entire chain:

Identity
   ↓
Authentication
   ↓
MFA / Conditional Access
   ↓
Enterprise Application Authorization
   ↓
SAML Claims
   ↓
Signed Assertion
   ↓
Correct ACS Endpoint
   ↓
Application User Mapping / Provisioning
   ↓
Application Session
Breaking individual parts of this chain and troubleshooting the resulting errors made the SAML authentication process much clearer than simply configuring a working SSO connection.

Technologies
Microsoft Entra ID | Enterprise Applications | SAML 2.0 | SSO | MFA | Conditional Access | Security Groups | Identity & Access Management | SCIM Concepts
