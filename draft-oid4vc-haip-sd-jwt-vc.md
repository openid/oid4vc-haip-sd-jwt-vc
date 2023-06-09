%%%
title = "OpenID4VC High Assurance Interoperability Profile with SD-JWT VC"
abbrev = "oid4vc-haip-sd-jwt-vc"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid4vc", "sd-jwt"]

[seriesInfo]
name = "Internet-Draft"
value = "draft-oid4vc-haip-sd-jwt-vc-latest"
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

This document defines a profile of OpenID for Verifiable Credentials in combination with the credential format SD-JWT VC. The aim is to select features and to define a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. The profiled specifications include OpenID for Verifiable Credential Issuance [@!OIDF.OID4VCI], OpenID for Verifiable Presentations [@!OIDF.OID4VP], Self-Issued OpenID Provider v2 [@!OIDF.SIOPv2], and SD-JWT VC [@!I-D.terbu-sd-jwt-vc].

{mainmatter}


# Introduction

This document defines a set of requirements for the existing specifications to enable interoperability among Issuers, Wallets and Verifiers of Credentials where a high level of security and privacy is required. This document is an interoperability profile that can be used by implementations in various contexts, be it a certain industry or a certain regulatory environment.

This document is not a specification, but a profile. It refers to the specifications required for implementations to interoperate among each other and for the optionalities mentioned in the referenced specifications, defines the set of features to be mandatory to implement.

The profile uses OpenID for Verifiable Credential Issuance [@!OIDF.OID4VCI] and OpenID for Verifiable Presentations [@!OIDF.OID4VP] as the base protocols for issuance and presentation of Credentials, respectively. The credential format used is SD-JWT VC as specified in [@!I-D.terbu-sd-jwt-vc]. Additionally, considerations are given on how deployments can perform a combined issuance of credentials in both SD-JWT VC and ISO mdoc [@ISO.18013-5] formats.

A full list of the open standards used in this profile can be found in Overview of the Open Standards Requirements (reference).

## Audience Target audience/Usage

The audience of the document is implementers that require a high level of security and privacy for their solutions. A non-exhaustive list of the interested parties includes eIDAS 2.0, California Department of Motor Vehicles, Open Wallet Foundation (OWF), IDunion, GAIN, and the Trusted Web project of the Japanese government, but is expected to grow to include other jurisdictions and private sector companies.

# Terminology

This specification uses the terms "Holder", "Issuer", "Verifier", and "Verifiable Credential" as defined in [@!I-D.terbu-sd-jwt-vc].

# Scope

The following aspects are in scope of this interoperability profile:

* Protocol for issuance of the Verifiable Credentials (can be both remote and in-person) (OID4VCI)
* Protocol for online presentation of Verifiable Credentials (can be both remote and in-person) (OID4VP)
* Protocol for User Authentication by the Wallet at a Verifier (SIOP v2)
* Wallet Attestation (during Credential issuance)
* Credential Format (SD-JWT VC)
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

The following items are out of scope for the current version of this document, but might be added in future versions:

* Trust Management, i.e. authorization of an issuer to issue certain types of credentials, authorization of the Wallet to be issued certain types of credentials, authorization of  the Verifier to receive certain types of credentials.
* Protocol for presentation of Verifiable Credentials for offline use-cases, e.g. over BLE.

## Scenarios/Business Requirements

* Combined Issuance of SD-JWT VC and mdoc
* Both issuer-initiated and wallet-initiated issuance
* eIDAS PID and (Q)EAA as defined in eIDAS ARF 1.0

## Standards Requirements

Unless explicitly stated, all normative requirements apply to all participating entities: Issuers, Wallets and Verifiers.

| (as defined in this profile) | Issuer | Wallet | Verifier |
|:--- |:--- |:--- |:--- |
|OID4VP| N/A |MUST|MUST|
|OID4VCI| MUST|MUST|N/A|
|SIOPv2|N/A|MUST|SHOULD|
|SD-JWT VC|MUST|MUST|MUST|

# OpenID for Verifiable Credential Issuance

Implementations of this profile:

* MUST support both pre-auth code flow and authorization code flow.
* MUST support SD-JWT VC profile as defined in OID4VCI specification.
* MUST support sender-constrained Tokens using a mechanism as defined in [@!I-D.ietf-oauth-dpop].
* MUST support [@!RFC7636] with `S256` as the code challenge method.

Both Wallet initiated and Issuer initiated issuance is supported.

## Credential Offer

* The Grant Types `authorization_code` and `urn:ietf:params:oauth:grant-type:pre-authorized_code` MUST be supported as defined in Section 4.1.1 in [@!OIDF.OID4VCI]
* For Grant Type `authorization_code`, the Issuer MUST include a scope value in order to allow the Wallet to identify the desired credential type. The wallet MUST use that value in the `scope` Authorization parameter. For Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`, the pre-authorized code is used by the issuer to identify the credential type(s). (pending OID4VCI PR#519)
* As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.

Note: The Authorization Code flow does not require a Credential Offer from the Issuer to the Wallet. However, it is included in the feature set of the Credential Offer because it might be easier to implement with existing libraries and on top of existing implementations than the pre-authorized code Grant Type.

Both sending Credential Offer same-device and cross-device is supported.

## Authorization Endpoint

   * MUST use Pushed Authorization Requests (PAR) [@!RFC9126] to send the Authorization Request.
   * Wallets MUST authenticate itself at the PAR endpoint using the same rules as defined in (#token-endpoint) for client authentication at the token endpoint.
   * MUST use `scope` parameter to communicate credential type(s) to be issued. The scope value MUST map to a specific Credential type. (pending OID4VCI PR#520)
   * The `client_id` value in the PAR request MUST be a string that the Wallet has used as the `sub` value in the client attestation JWT.

## Token Endpoint {#token-endpoint}

   * The Wallets MUST perform client authentication as defined in [!I-D.ietf-looker-key-attestation-client-authentication].
   * Refresh tokens MUST be supported for credential refresh.
   * Wallets MUST support deferred authorization by being able to process the Token error response parameters `authorization_pending` and `slow_down`, and the credential offer parameter `interval`.
   * The wallet attestation JWT scheme is defined in (#wallet-attestation-schema).

Note: It is RECOMMENDED to use ephemeral client attestation JWTs for client authentication in order to prevent linkability across Credential Issuers.

Note: Issuers should be mindful of how long the usage of the refresh token is allowed to refresh a credential, as opposed to starting the issuance flow from the beginning. For example, if the User is trying to refresh a credential more than a year after its original issuance, the usage of the refresh tokens is NOT RECOMMENDED.

### Wallet Attestation Schema {#wallet-attestation-schema}

[Section 3.1 of wallet attestation draft would define the basics, and this profile will define the details.]

## Credential Endpoint

   * The `JWT` proof type MUST be supported.

## Server Metadata

* The Credential Issuer MUST publish a mapping of every Credential Type it supports to a scope value.

# OpenID for Verifiable Presentations

   * As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.
   * Response type MUST be `vp_token`.
   * Response mode MUST be `direct_post` with `redirect_uri` as defined in Section 6.2 of [@!OIDF.OID4VP].
   * Authorization Request MUST be sent using the `request_uri` parameter as defined in JWT-Secured Authorization Request (JAR) [@!RFC9101].
   * `client_id_scheme` parameter MUST be present in the Authorization Request.
   * `client_id_scheme` value MUST be either `x509_san_dns` or `verifier_attestation`. Wallet MUST support both. Verifier MUST support at least one. (pending OID4VCI PR #524 for verifier_attestation)
   * Presentation Definition JSON object MUST be sent using a `presentation_definition` parameter.
   * The following features from the DIF Presentation Exchange v2.0.0 MUST be supported. A JSON schema for the supported features is in (#presentation-definition-schema):

    * In the `presentation_definition` object, `id`, `input_descriptors` and `submission_requirements` properties MUST be supported.
    * In the `input-descriptors` object, `id`, `name`, `purpose`, `group`, `format`, and `constraints` properties MUST be supported. In the `constraints` object, `limit_disclosure`, and `fields` properties MUST be supported. In the `fields` object, `path` and `filter` properties MUST be supported. A `path` MUST contain exactly one entry with a static path to a certain claim. A `filter` MUST only contain `type` elements of value `string` and `const` elements.
    * In the `submission_requirements` object, `name`, `rule (`pick` only)`, `count`, `from` properties MUST be supported.

# Self-Issued OP v2

To authenticate the user, subject identifier in a Self-Issued ID Token MUST be used as defined in [@!OIDF.SIOPv2].

   * As a way to invoke the Wallet, at least a custom URL scheme `haip://` MUST be supported. Implementations MAY support other ways to invoke the wallets as agreed by trust frameworks/ecosystems/jurisdictions, not limited to using other custom URL schemes.
   * `subject_syntax_types_supported` value MUST be `urn:ietf:params:oauth:jwk-thumbprint`

# SD-JWT VCs {#sd-jwt-vc}

As credential format, SD-JWT VCs as defined in [@!I-D.terbu-sd-jwt-vc] MUST be used.

In addition, this profile defines the following additional requirements.

* Compact serialization MUST be supported as defined in [@!I-D.ietf-oauth-selective-disclosure-jwt].
* The following JWT Claims MUST be supported Content (differentiate issuance & presentation)

JSON serialization MAY be supported if required by the jurisdiction.

| Claim | SD-JWT as issued by the Issuer | Normative Definition |
|:--- |:--- |:--- |
|iss |MUST |[@!RFC7519], Section 4.1.1 |
|iat |MUST |[@!RFC7519], Section 4.1.6 |
| exp | SHOULD (at the discretion of the issuer) | [@!RFC7519], Section 4.1.4 |
|cnf|	MUST|	[@!RFC7800]|
|type|	MUST| [@!I-D.terbu-sd-jwt-vc]|
|status|SHOULD (at the discretion of the issuer)|	WIP|

* The Issuer MUST NOT make any of the JWT Claims in the table above to be selectively disclosable, so that they are always present in the SD-JWT-VC presented by the Holder.
* It is at the discretion of the Issuer whether to use `exp` claim and/or a `status` claim to express the validity period of an SD-JWT-VC. The wallet and the verifier  MUST support both mechanisms.
* The `iss` claim MUST be an HTTPS URL. The `iss` value is used to obtain Issuer’s signing key as defined in (#issuer-key-resolution).
* The `type` JWT claim as defined in [@!I-D.terbu-sd-jwt-vc].
* The `cnf` claim [@!RFC7800] MUST conform to the definition given in [@!I-D.terbu-sd-jwt-vc]. Implementations conforming to this profile MUST include the JSON Web Key [@!RFC7517] in the `jwk` sub claim.

Note: Currently this profile only supports presentation of credentials with cryptographic Holder Binding: the holder's signature is required to proof the credential is presented by the holder it was issued to. This profile might support claim-based and biometrics-based holder binding once OpenID for Verifiable Credentials adds support for other forms of Holder Binding. See https://bitbucket.org/openid/connect/issues/1537/presenting-vc-without-a-vp-using-openid4vp

Note: Re-using the same Credential across Verifiers, or re-using the same JWK value across multiple Credentials gives colluding Verifiers a mechanism to correlate the User. There are currently two known ways to address this with SD-JWT VCs. First is to issue multiple instances of the same credentials with different JWK values, so that if each instance of the credential is used at only one Verifier, it can be reused multiple times. Another is to use each credential only once (ephemeral credentials). It is RECOMMENDED to adopt one of these mechanisms.

Note: If there is a requirement to communicate information about the verification status and identity assurance data of the claims about the subject, the syntax defined by [@!OIDF.ekyc-ida] SHOULD be used. It is up to each jurisdiction and ecosystem, whether to require it to the implementers of this profile.

Note: If there is a requirement to provide the Subject’s identifier assigned and maintained by the Issuer, `sub` claim MAY be used. There is no requirement for a binding to exist between `sub` and `cnf` claims. See section X in [@!I-D.terbu-sd-jwt-vc] for implementation considerations.

Note: In some credential types, it is not desirable to include an expiration date (eg: diploma attestation). Therefore, this profile leaves its inclusion to the Issuer, or the body defining the respective credential type.

## Issuer identification and key resolution to validate an issued Credential {#issuer-key-resolution}

This profile supports two ways to represent and resolves the key required to validate the issuer signature of a SD-JWT VC, web PKI-based key resolution and x.509 certificates.

* Web-based key resolution: The key used to validate the Issuer’s signature on the SD-JWT VC MUST be obtained from the SD-JWT VC issuer's metadata as defined in Section 5 of [@!I-D.terbu-sd-jwt-vc]. The JOSE header `kid` MUST be used to identify the respective key.
* x.509 certificates: the SD-JWT VC contains the issuer's certificate along with a trust chain in the `x5c` JOSE header. In this case, the `iss` value MUST be an URL with a FQDN matching a `dNSName` Subject Alternative Name (SAN) [@!RFC5280] entry in the leaf certificate.

Note: The issuer MAY decide to support both options. In which case, it is at the discretion of the Wallet and the Verifier which key to use for the issuer signature validation.

### Cryptographic Holder Binding between VC and VP

* For Cryptographic Holder Binding, an HB-JWT as defined in [@!I-D.terbu-sd-jwt-vc] MUST always be present when presenting a SD-JWT VC.

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

<reference anchor="OIDF.ekyc-ida" target="https://openid.net/specs/openid-connect-4-identity-assurance-1_0-ID4.html">
  <front>
    <title>OpenID Connect for Identity Assurance 1.0</title>
    <author ullname="Torsten Lodderstedt ">
      <organization>yes</organization>
    </author>
    <author fullname="Daniel Fett">
      <organization>yes</organization>
    </author>
 <author fullname="Mark Haine">
      <organization>Considrd.Consulting Ltd</organization>
    </author>
     <author fullname="Alberto Pulido">
      <organization>Santander</organization>
    </author>
     <author fullname="Kai Lehmann">
      <organization>1&amp;1 Mail &amp; Media Development &amp; Technology GmbH</organization>
    </author>
     <author fullname="Kosuke Koiwai">
      <organization>KDDI Corporation</organization>
    </author>
   <date day="19" month="August" year="2022"/>
  </front>
</reference>

<reference anchor="ISO.18013-5" target="https://www.iso.org/standard/69084.html">
        <front>
          <title>ISO/IEC 18013-5:2021 Personal identification — ISO-compliant driving license — Part 5: Mobile driving license (mDL)  application</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date year="2021"/>
        </front>
</reference>

<reference anchor="VC-DATA" target="https://www.w3.org/TR/vc-data-model-2.0/">
        <front>
        <title>Verifiable Credentials Data Model v2.0</title>
        <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Dave Longley">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="David Chadwick">
            <organization>Crossword Cybersecurity PLC</organization>
        </author>
        <date day="4" month="May" year="2023"/>
        </front>
</reference>

# Combined Issuance of SD-JWT VC and mdocs

   * If combined issuance is required, the Batch Credential Endpoint MUST be supported.

# JSON Schema for the supported Presentation Definition properties {#presentation-definition-schema}

<{{schemas/presentation_definition.json}}
