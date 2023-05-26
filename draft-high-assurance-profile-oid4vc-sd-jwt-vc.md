%%%
title = "High Assurance Profile of OpenID4VC with SD-JWT-VC"
abbrev = "draft-high-assurance-profile-oid4vc-sd-jwt-vc"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid4vc", "sd-jwt"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-high-assurance-profile-oid4vc-sd-jwt-vc-latest"
status = "standard"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="Microsoft"
   [author.address]
   email = "kristina.yasuda@microsoft.com"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="yes.com"
   [author.address]
   email = "torsten@lodderstedt.net"

%%%

.# Abstract

This document defines a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. This profiled specifications include OpenID for Vefiriable Presentations, OpenID for Verifiable Credential Issuance, Self-Issued OpenID Provider v2, SD-JWT-VC.

{mainmatter}


# Introduction

This document defines a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. This document is an interoperability profile that can be used by implementations in various contexts, be it a certain industry or certain regulatory requirements.

This document is not a specification, but a profile. It outlines specifications required for implementations to interoperate among each other. It also clarifies the set of features to be mandatory, for the optionalities mentioned in the referenced specifications.

The profile uses OpenID for Verifiable Credential Issuance (OID4VCI) and OpenID for Verifiable Presentations (OID4VP) as the base protocols for issuance and presentation of Credentials. Credential formats used in the profile are W3C Verifiable Credentials (VC Data Model v2.0) secured using Selective Disclosure for JWTs (SD-JWT). A full list of the open standards used in this profile can be found in Overview of the Open Standards Requirements.

## Audience Target audience/Usage

The audience of the document is implementers that require a high level of security and privacy for their solutions. A non-exhaustive list of the interested parties includes eIDAS 2.0, California Department of Motor Vehicles, Open Wallet Foundation (OWF), IDunion, GAIN, and the Trusted Web project of the Japanese government, but is expected to grow to include other jurisdictions and private sector companies.

# Terminology

ToDo: Mostly reference other specs.

# Scope

The following aspects are in scope of this interoperability profile:
* Protocol for issuance of the Credentials online (can be both remote and in-person)
* Protocol for presentation of Credentials online (can be both remote and in-person)
* User Authentication
* Wallet Instance Attestation (during Credential issuance)
* Credential Format
* Status Management of the Credentials, including revocation
* Cryptographic Holder Binding
* Issuer key resolution
* Issuer identification (as prerequisite for trust management)
* Crypto Suites

Assumptions made are the following:
* The issuers and verifiers cannot pre-discover wallet’s capability
* The issuer is talking to the wallet supporting the features defined in this profile (via wallet invocation mechanism)
* There are mechanisms in place for the verifiers and issuers to discover each other’s capability

## Out of Scope

The following items are out of scope for the current version of this document, but are planned to be added in the future versions:
* Trust Management, i.e. authorization of an issuer to issue certain types of credentials, authorization of the Wallet to be issued certain types of credentials, authorization of  the Verifier to receive certain types of credentials.
* Protocol for presentation of Credentials over BLE for offline use-cases.

## Scenarios/Business Requirements

* Combined Issuance of SD-JWT and mdoc
* Both issuer-initiated and wallet-initiated issuance
* eIDAS credentials as defined in ARF 1.0

## Standards Requirements

Unless explicitly stated, all normative requirements apply to all participating entities: Issuers, Wallets and Verifiers.

| (as defined in this profile) | Issuer | Wallet | Verifier |
|:--- |:--- |:--- |:--- |
|OID4VP| N/A |MUST|MUST|
|OID4VCI| MUST|MUST|N/A|
|SIOPv2|N/A|MUST|SHOULD|
|SD-JWT|MUST|MUST|MUST|

# OpenID for Verifiable Credential Issuance

* MUST support both pre-auth code flow and authorization code flow.
* MUST support SD-JWT VC profile as defined in OID4VCI specification.
* MUST support sender-constrained Tokens using a mechanism as defined in [@!I-D.ietf-oauth-dpop].
* MUST support [@!RFC7636] with `S256` as the code challenge method.

## Credential Offer

* Both Grant Types `authorization_code` and `urn:ietf:params:oauth:grant-type:pre-authorized_code` MUST be supported as defined in Section 4.1.1 in [@!OIDF.OID4VCI]
* For Grant Type `authorization_code`, the Issuer MUST include a scope value. The wallet MUST use that value in the `scope` Authorization parameter. For `urn:ietf:params:oauth:grant-type:pre-authorized_code`, pre-authorized code is used by the issuer to identify the credential type.
* As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.

Note: Authorization Code flow does not require Credential Offer from the Issuer to the Wallet, and fits the scenario where the Wallet has preconfigured issuers interested in being promoted by wallet as entrance into a VC ecosystem will be motivated to implement it, might be easier to implement with existing libs and on top of existing implementations.

## Authorization Endpoint

   * MUST use Pushed Authorization Requests (PAR) [@!RFC9126] to send the Authorization Request.
   * Wallet MUST authenticate itself at the PAR endpoint using the same rules as defined in (#token-endpoint) for client authentication at the token endpoint.
   * MUST use `scope` parameter to communicate credential type(s) to be issued. The scope value MUST map to a specific Credential type. This mapping MUST be used when obtaining user consent.
   * `scope` parameter MUST be communicated from the Issuer to the Wallet either in the Credential Offer, or Credential Issuer metadata.
   * The `client_id` value in the PAR request MUST be a string that the Wallet has used as the `sub` value in the client key attestation JWT.

## Token Endpoint {#token-endpoint}

   * The Wallets MUST perform client authentication as defined in draft-looker-key-attestation-client-authentication
   * Refresh token MUST be supported for credential refresh
   * Deferred issuance MUST be supported by the means of deferred authorization. Token error response parameters `authorization_pending` and `slow_down`, and credential offer parameter `interval` MUST be supported.
   * Schema of the wallet attestation VC MUST be as follows:

Note: It is RECOMMENDED to use ephemeral client key attestation JWT for client authentication.

Note: Issuers should be mindful of how long the usage of the refresh token is allowed to refresh a credential, as opposed to starting the issuance flow from the beginning. For example, if the User is trying to refresh a credential more than a year after its original issuance, the usage of the refresh tokens is NOT RECOMMENDED.

### Wallet Instance Attestation Schema

Section 3.1 of wallet attestation draft would define the basics, and this profile will define the details.

## Credential Endpoint

   * `JWT` proof type MUST be supported.

# OpenID for Verifiable Presentations

   * As a way to invoke the Wallet, at least a custom URL scheme haip:// MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.
   * Response type MUST be `vp_token`.
   * Response mode MUST be `direct_post` with `redirect_uri` as defined in Section 6.2 of [@!OIDF.OID4VP].
   * Authorization Request MUST be sent using the `request_uri` parameter as defined in JWT-Secured Authorization Request (JAR) [@!RFC9101].
   * `client_id_scheme` parameter MUST be present in the Authorization Request.
   * `client_id_scheme` value MUST be either `x509_san` or `verifier_attestation`. Wallet MUST support both. Verifier MUST support at least one.
   * When the Client Identifier Scheme is `x509_san_uri`, the Client Identifier MUST be URI and match the `uniformResourceIdentifier` Subject Alternative Name (SAN) [@!RFC5280] entry in the leaf certificate passed with the request. The request MUST be signed with the private key corresponding to the public key in the leaf X.509 certificate of the certificate chain added to the request in the `x5c` JOSE header [@!RFC7515] of the signed request object. The Wallet MUST validate the signature and the trust chain of the X.509 certificate. All Verifier metadata other than the public key MUST be obtained from the `client_metadata` parameter.
   * When the Client Identifier Scheme is `verifier_attestation`, the Client Identifier MUST equal `sub` claim value in the Verifier attestation JWT. The request MUST be signed with the private key corresponding to the public key in the `cnf` claim in the Verifier attestation JWT. The Verifier attestation VC MUST be added to a newly defined `verifier_attestation` JOSE Header of a request object. The Wallet MUST validate the signature on the Verifier attestation JWT by a trusted third party. Verifier metadata MUST be obtained from the Verifier attestation JWT. Schema of the verifier attestation VC is defined in the Annex.
   * Presentation Definition JSON object MUST be sent using a `presentation_definition` parameter.
   * The following features from the DIF Presentation Exchange v2.0.0 MUST be supported. JSON schema for the supported features is in (#presentation-definition-schema):
      * In the `presentation_definition` object, `id`, `input_descriptors` and `submission_requirements` properties MUST be supported.
      * In the `input-descriptors` object, `id`, `name`, `purpose`, `group (for submission requirement)`, `format`, and `constraints` properties MUST be supported. In the `constraints` object, `limit_disclosure`, and `fields` properties MUST be supported. In the `fields` object, `path` and `filter` properties MUST be supported.
      * In the `submission_requirements` object, `name`, `rule (`pick` only)`, `count`, `from` properties MUST be supported.

# User Authentication

To authenticate the user, subject identifier in a Self-Issued ID Token MUST be used as defined in [@!OIDF.SIOPv2].

   * `subject_syntax_types_supported` value MUST be `urn:ietf:params:oauth:jwk-thumbprint`
   * As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.

# SD-JWT VCs {#sd-jwt-vc}

As a credential format, the SD-JWT VC as defined in this section MUST be supported.

All the requirements defined in [@!I-D.ietf-oauth-selective-disclosure-jwt] MUST be supported. In addition, this profile defines additional requirements outlined in this section.

* Both Compact serialization and JSON serialization MUST be supported as defined in [@!I-D.ietf-oauth-selective-disclosure-jwt].
* The following JWT Claims MUST be supported Content (differentiate issuance & presentation)

| | SD-JWT as issued by the Issuer | Normative Definition |
|:--- |:--- |:--- |
|iss |MUST |[@!RFC7519], Section 4.1.1 |
|iat |MUST |[@!RFC7519], Section 4.1.6 |
| exp | SHOULD (at the discretion of the issuer) | [@!RFC7519], Section 4.1.4 |
|cnf|	MUST|	[@!RFC7800]|
|type|	MUST| draft-terbu-sd-jwt-vc|
|status|SHOULD (at the discretion of the issuer)|	WIP|

* The Issuer MUST NOT make any of the JWT Claims in the table above to be selectively disclosable, so that they are always present in the SD-JWT-VC presented by the Holder.
* It is at the discretion of the Issuer whether to use `exp` claim or a `status` claim to express the validity period of an SD-JWT-VC. The wallet and the verifier  MUST support both mechanisms.
* `iss` claim MUST be used to obtain Issuer’s signing key as defined in (#issuer-key-resolution). `iss` claim MUST be either an HTTPS URL or a Subject Alternative Name (SAN) from an X.509 certificate. Wallets and the Verifiers MUST support both options.
* `type` JWT claim as defined in draft-terbu-sd-jwt-vc. For details on the `cnf` claim, see (#holder-key-resolution).
* `status` claim MUST be present as defined in draft-terbu-sd-jwt-vc/draft-looker-jwt-cwt-status-list
* For Cryptographic Holder Binding, HB-JWT MUST be supported as defined in [@!I-D.ietf-oauth-selective-disclosure-jwt].

Further claims, either registered or private JWT claims, can be added to the credential as required by the respective credential type, or determined by the Issuer or the Holder. All claims that are not understood by implementations MUST be ignored, as defined in section 4, [@!RFC7519].

Note: If there is a requirement to communicate information about the verification status and identity assurance data of the claims about the subject, the syntax defined by @!OIDF.ekyc-ida SHOULD be used. It is up to each jurisdiction and ecosystem, whether to require it to the implementers of this profile.

Note: If there is a requirement to provide Subject’s identifier assigned and maintained by the Issuer, `sub` claim MAY be used. There is no requirement for a binding to exist between `sub` and `cnf` claims. See section X in @!I-D.draft-terbu-sd-jwt-vc for implementation considerations.

Note: Currently this profile only supports presentation of credentials with cryptographic Holder Binding: holder signature is required to bind presentation to a particular transaction and audience. This is planned to be expanded, once OpenID for Verifiable Credentials adds support for other forms of Holder Binding. See https://bitbucket.org/openid/connect/issues/1537/presenting-vc-without-a-vp-using-openid4vp

Note: In some credential types, it is not desirable to include an expiration date (eg: diploma attestation). Therefore, this profile leaves its inclusion to the Issuer, or the body defining the respective credential type.

## Issuer identification and key resolution to validate an issued Credential {#issuer-key-resolution}

* When `iss` claim is an HTTPS URL, the key used to validate the Issuer’s signature on the SD-JWT VC MUST be hosted at the .well-known path as defined in draft-terbu-sd-jwt-vc. JOSE header `kid` MUST be used to obtain the key.
* When the `iss` identifier is a Subject Alternative Name (SAN), and the key used to validate the Issuer’s signature on the SD-JWT VC MUST be obtained from the `x5c` JOSE header.

Note: The issuer MAY decide to support both options. In which case, it is at the discretion of the Wallet and the Verifier which key to use for the issuer signature validation.

## Holder key resolution {#holder-key-resolution}

* The public key passed in the `cnf` claim [@!RFC7800] MUST be a JSON Web Key [@!RFC7517] contained in the `jwk` claim. `kid` MUST be included in the JWK in the `cnf` claim, since it will be used in the JOSE Header.

Note: this requirement might change when claim-based and biometrics-based holder binding become available.

Note: Re-using the same Credential across Verifiers, or re-using the same JWK value across multiple Credentials gives colluding Verifiers a mechanism to correlate the User. There are currently two known ways to address this with SD-JWT VCs. First is to issue multiple instances of the same credentials with different JWK values, so that if each instance of the credential is used at only one Verifier, it can be reused multiple times. Another is to use each credential only once (ephemeral credentials). It is RECOMMENDED to adopt one of these mechanisms.

### Cryptographic Holder Binding between VC and VP

* Holder Binding JWT MUST be supported as defined in [@!I-D.ietf-oauth-selective-disclosure-jwt].

# Crypto Suites

Issuers, holders and verifiers MUST support P-256 (secp256r1) as a key type with ES256 JWT algorithm for signing and signature validation whenever this profiles requires to do so:
* SD-JWT-VC
* Wallet Instance Attestation
* DPoP
* HB JWT
* Authorization request during presentation

SHA256 MUST be supported by all the entities as the hash algorithm to generate and validate the digests in the SD-JWT VC.

Note: When using this profile with other cryptosuites, it is recommended to be explicit about which entity is required to support which curve for signing and/or signature validation

# Implementations Considerations

## Validity Period of the Signature and the Claim Values

`iat` and `exp` JWT claims express both the validity period of both the signature and the claims about the subject, unless there is a separate claim used to express the validity of the claims.

{backmatter}

<reference anchor="OIDF.OID4VCI" target="https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html">
        <front>
          <title>OpenID for Verifiable Credential Issuance</title>
          <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
            <organization>yes.com</organization>
          </author>
          <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
            <organization>Microsoft</organization>
          </author>
          <author initials="T." surname="Looker" fullname="Tobias Looker">
            <organization>Mattr</organization>
          </author>
          <date day="20" month="June" year="2022"/>
        </front>
</reference>

<reference anchor="OIDF.OID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0.html">
      <front>
        <title>OpenID for Verifiable Presentations</title>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
         <organization>Spruce Systems, Inc.</organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>yes.com</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>Microsoft</organization>
        </author>
        <author initials="A." surname="Lemmon" fullname="Adam Lemmon">
          <organization>Convergence.tech</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="20" month="June" year="2022"/>
      </front>
</reference>

<reference anchor="OIDF.SIOPv2" target="https://openid.net/specs/openid-connect-self-issued-v2-1_0.html">
  <front>
    <title>Self-Issued OpenID Provider V2</title>
    <author ullname="Kristina Yasuda">
      <organization>Microsoft</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
   <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
      <organization>yes.com</organization>
    </author>
   <date day="18" month="December" year="2021"/>
  </front>
</reference>

# Combined Issuance of SD-JWT VC and mdocs

   * If combined issuance is required, Batch Credential Endpoint MUST be supported.

# JSON Schema for the supported Presentation Definition properties {#presentation-definition-schema}

<{{schemas/presentation_definition.json}}
