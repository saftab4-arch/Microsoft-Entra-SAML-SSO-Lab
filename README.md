Microsoft Entra ID -- Enterprise SAML SSO & Identity Access Lab

Hands-on Microsoft Entra ID lab implementing and troubleshooting SAML
2.0 SSO with Microsoft Entra ID as the Identity Provider (IdP) and
Microsoft Entra SAML Toolkit as the test Service Provider (SP).

Objectives

Configure cloud users and security groups

Configure an Entra Enterprise Application

Implement group-based application access

Configure SAML 2.0 SSO

Understand IdP vs SP

Configure Entity ID, Sign-on URL, and ACS/Reply URL

Configure NameID and SAML claims

Configure SAML signing certificate trust

Test SP-initiated SSO

Review MFA / Conditional Access and sign-in logs

Troubleshoot authorization, provisioning, NameID, and ACS failures

Understand SCIM and JIT provisioning concepts

Architecture

User
  |
  v
Service Provider
  |
  | Redirect
  v
Microsoft Entra ID
  |
  | Authentication + MFA
  | App authorization
  v
Signed SAML Assertion
  |
  | POST
  v
ACS / Reply URL
  |
  v
Application Session

1. Identity and Group Foundation

Cloud-only users and security groups were created in Microsoft Entra ID.
Groups included department and application-access groups such as
SG-Sales, SG-IT, SG-HR, SG-Finance, SG-SaaS-App-Users, and
SG-VDI-Users.

The scalable access model was:

User -> Security Group -> Enterprise Application -> Access

2. Enterprise Application

The Microsoft Entra SAML Toolkit was added as an Enterprise Application.



Application access was assigned through security groups.



Assignment required = Yes was used so authentication alone did not
automatically authorize application access.

3. Basic SAML Configuration



Important values:

Entity ID / Identifier -- uniquely identifies the Service
Provider.

Sign-on URL -- where the SP-initiated login begins.

Reply URL / ACS -- where Entra sends the SAML response after
authentication.

The Service Provider Entity ID used was:

https://samltoolkit.azurewebsites.net

The ACS endpoint used in this lab was:

https://samltoolkit.azurewebsites.net/SAML/Consume/22412

4. Attributes, Claims, and NameID



The important NameID configuration was:

Name identifier format: Email address
Source: Attribute
Source attribute: user.userprincipalname

Example:

NameID = hamdan@basitcloudlab.com

The Service Provider uses NameID to map the federated Entra identity to
its application-side identity.

5. SAML Signing Certificate



Entra creates and digitally signs the SAML assertion. The Service
Provider uses the trusted Entra certificate/public key to verify the
signature.

Entra -> Signed Assertion -> Service Provider -> Signature Validation

6. Entra SAML Endpoints



Login URL = where the application sends the user for Entra
authentication.

Entra Identifier = identity of the Entra IdP/issuer.

Logout URL = endpoint used as part of SAML logout.

7. Service Provider Configuration



The Service Provider was configured with the Entra Login URL, Entra
Identifier, Logout URL, and signing certificate. Entra was configured
with the SP Entity ID and ACS/Reply URL.

Entra ID (IdP) <------ SAML Trust ------> SAML Toolkit (SP)

8. Successful SP-Initiated SSO



End-to-end flow:

Hamdan
  |
  v
SP Initiated Login
  |
  v
Redirect to Entra
  |
  v
Authentication + MFA
  |
  v
Enterprise App Authorization
  |
  v
Signed SAML Assertion
  |
  v
ACS / Reply URL
  |
  v
Validate Signature + Issuer + NameID
  |
  v
Application Session
  |
  v
ACCESS GRANTED

The user's Entra password is not sent to the Service Provider. The
application trusts the signed assertion issued by Entra.

9. Group-Based Authorization



Example:

Hamdan -> SG-Sales -> Enterprise Application -> Authorized

This demonstrated the difference between:

Authentication: Is this really Hamdan?

Authorization: Is Hamdan allowed to use this application?

10. Sign-In Logs





Entra sign-in logs were used to inspect user, application, status,
authentication requirement, Conditional Access, failure reason, error
code, Request ID, and Correlation ID.

A major lesson was that Entra Success does not always mean the SaaS
application successfully established a session. If Entra succeeds but
the application fails afterward, troubleshooting continues on the
Service Provider side.

Authentication vs Authorization vs Provisioning

Authentication

User -> Entra ID -> Password/MFA -> Identity Verified

Authorization

User -> Security Group -> Enterprise App Assignment -> Access Allowed

Provisioning

Provisioning answers whether the user actually exists inside the SaaS
application.

During this lab, the SAML Toolkit required a matching local application
identity.

Entra identity: iqra@basitcloudlab.com
                     |
                     v
Application identity: iqra@basitcloudlab.com

This demonstrated that SAML authentication and SaaS user provisioning
are separate processes.

Real-World SCIM / JIT

A production SaaS platform may use SCIM:

Microsoft Entra ID
    |          |
   SCIM       SAML
    |          |
Provision   Authenticate
    |          |
    +---- SaaS App

SCIM can create, update, and deprovision supported SaaS identities
automatically.

Some applications instead support Just-In-Time (JIT) provisioning,
where the first successful SAML login creates the application account
automatically.

Troubleshooting Performed

Ticket 1 -- AADSTS50105: User Not Assigned

Symptom: Iqra authenticated to Entra but was denied application
access.

Finding: The account was valid, but her security group was not
assigned to the Enterprise Application.

Authentication = Successful
Authorization  = Failed

Error:

AADSTS50105

Resolution: Assigned SG-IT to the Enterprise Application and
retested.

Lesson: A valid Entra identity does not automatically provide
authorization to every Enterprise Application.

Ticket 2 -- SaaS User Was Not Provisioned

After fixing Entra authorization, Iqra's Entra sign-in succeeded and the
SAML response reached the application, but the test Service Provider
still failed.

Entra Authentication       = SUCCESS
Enterprise App Authorization = SUCCESS
SAML Assertion             = SUCCESS
Application User Mapping   = FAILURE

Root cause: The test SAML Toolkit required a matching local account
for iqra@basitcloudlab.com.

Resolution: Created the matching application-side identity and
successfully tested SSO.

Lesson: SAML SSO does not automatically provision SaaS accounts.
Production applications may use SCIM, JIT, or another provisioning
workflow.

Ticket 3 -- Incorrect NameID Mapping

The working NameID was deliberately changed from:

user.userprincipalname

to:

user.displayname

Instead of receiving an email-style identity such as:

hamdan@basitcloudlab.com

the Service Provider received an identity similar to:

Hamdan Malik

Result: Entra authentication could succeed while the Service
Provider could not correctly map the user.

Resolution: Restored:

Name identifier format: Email address
Source attribute: user.userprincipalname

Lesson: The claims Entra sends must match the identity format
expected by the Service Provider.

Ticket 4 -- AADSTS50011: Reply URL / ACS Mismatch

The registered Reply URL was deliberately changed.

The Service Provider requested its legitimate ACS URL, but the URL no
longer matched the Reply URL registered in Entra.

Error:

AADSTS50011

Entra stopped the flow rather than sending a SAML response to an
unregistered destination.

Resolution: Restored:

https://samltoolkit.azurewebsites.net/SAML/Consume/22412

SSO was successfully retested.

Lesson: Reply URL validation is an important SAML security control.

Troubleshooting Workflow

1. Identify affected user
   |
2. Verify account status
   |
3. Verify group membership
   |
4. Verify Enterprise App assignment
   |
5. Check Entra sign-in logs
   |
6. Review error code / failure reason
   |
7. Review Conditional Access
   |
8. Check SAML claims / NameID
   |
9. Check Entity ID / ACS / Reply URL
   |
10. Determine failure layer:
    Authentication / Authorization /
    Provisioning / Service Provider

Key Lessons

SAML allows a Service Provider to delegate authentication to
Microsoft Entra ID.

Entra credentials are not handed directly to the SaaS application.

Entra digitally signs SAML assertions.

The Service Provider validates the signature using the trusted Entra
certificate.

NameID and claims must match what the Service Provider expects.

Authentication and authorization are separate.

Group-based assignments scale better than individual assignments.

A successful Entra sign-in does not guarantee application-side
success.

SaaS provisioning is separate from SAML authentication.

SCIM/JIT can automate SaaS account creation when supported.

Sign-in logs and error codes are essential for SSO troubleshooting.

ACS/Reply URL validation protects the SAML response destination.

Technologies / Concepts

Microsoft Entra ID

Enterprise Applications

SAML 2.0

Microsoft Entra SAML Toolkit

Entra Security Groups

Group-Based Application Assignment

MFA

Conditional Access

SAML Claims

NameID

SAML Signing Certificates

SP-Initiated SSO

Entra Sign-In Logs

Authentication / Authorization / Provisioning

SCIM concepts

JIT provisioning concepts

Screenshot Files

Place the screenshots in /screenshots/:

01-enterprise-application-overview.png

02-users-groups-assignment.png

03-basic-saml-configuration.png

04-saml-attributes-claims.png

05-saml-signing-certificate.png

06-entra-saml-endpoints.png

07-service-provider-saml-configuration.png

08-successful-sso-hamdan.png

09-group-based-sso-access.png

10-successful-sso-signin-log.png

11-sso-signin-activity-details.png

Future Extension

The next identity-focused extension can use a SCIM-capable application
to demonstrate:

Create Entra User
      |
Assign Security Group
      |
SCIM Provisions SaaS User
      |
SAML SSO
      |
Remove Assignment / Disable User
      |
SCIM Deprovisions SaaS User

Author

Syed Aftab

Hands-on cloud and infrastructure learning project focused on Microsoft
Azure, Microsoft Entra ID, enterprise identity, and cloud
administration.
