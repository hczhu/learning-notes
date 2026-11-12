file-created-at:: 2026-10-29

- **Source**: The Hack, “Passkeys are a trap. Do not use them.”, *Non-Functional*, October 10, 2026; supplied article text, including its corrections and addenda. Technical claims checked against the primary references linked below on October 29, 2026.
- **Takeaway**: Passkeys improve phishing resistance, but “supports passkeys” does not tell you who stores your keys, whether you can move them, or how you recover access. Evaluate those details rather than treating every implementation as identical.
- **Reading stance**: The article's useful warning is about user control and ecosystem dependence. Its predictions about surveillance and universal software restrictions are the author's concerns—not established consequences of ordinary passkey use.

- ## What happens when you use a passkey

	- **Registration** creates a public/private key pair for an account. The website stores the public key; the authenticator or passkey provider protects the private key. A breach of the public-key database alone does not give attackers a reusable login secret. [Google overview](https://developers.google.com/identity/passkeys)
	- **Login**: the website sends a fresh challenge; the authenticator signs data covering that challenge and the login context; the website verifies the signature with the registered public key. It does not receive the private key. Registration attestation and login assertions are different operations. [MDN protocol explanation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Authentication_API/Attestation_and_Assertion)
	- **Phishing resistance** comes from binding credentials to the relying party—the service—and checking the origin. A look-alike site cannot simply collect a typed passkey and relay it like a password or OTP. Domain binding does **not** mean binding to one physical device: synced credentials can work across devices. [Google overview](https://developers.google.com/identity/passkeys)
	- **Fingerprint, face, or PIN** unlocks local use of the credential; it is not the credential itself. Biometrics remain on the device, not with the website. A PIN can provide user verification without biometrics. [FIDO overview](https://fidoalliance.org/passkeys/)

- ## Properties that matter to users

	- | Property | Practical meaning | What to check |
	  | --- | --- | --- |
	  | Synced vs device-bound | Synced credentials can appear on other devices through a provider. Device-bound credentials stay with one authenticator, including a hardware security key. | If this device dies, where is my next working login? |
	  | Sync vs export/migration | Sync within a provider is not the same as moving to a competing provider. | Does my exact source/destination combination support passkey transfer—not just password export? |
	  | Hardware vs software protection | Hardware can resist extracting a key; software storage can offer different backup/export options. These are implementation choices, not a simple good/bad ranking. | Is this particular credential exportable? Do not infer that from a device having a secure chip. |
	  | Presence vs verification | A touch establishes interaction; a PIN or biometric check establishes local authorization. | Can anyone holding my unlocked device use the account? Who knows its PIN or has enrolled biometrics? |
	  | Authenticator attestation | Can establish credential provenance, allowing a service to accept or reject authenticator types. | Does this service permit my chosen provider/security key and offer a recovery path? |
	- These are **not independent user-selectable switches**. Provider capabilities, browser support, and service policy constrain the available combinations. The article's “non-copyable but transferable” category is not a universal migration promise.
	- **Hardware security keys are also passkey authenticators**, not a competing protocol. Carrying the key between computers moves the authenticator; it need not export the secret. Losing the sole key remains an availability risk. [FIDO overview](https://fidoalliance.org/passkeys/)
	- FIDO's **Credential Exchange Format/Protocol (CXF/CXP)** addresses provider-to-provider transfer. A standard's existence does not establish support in your installed apps; verify and test the migration. Do not assume a password CSV includes passkeys. [Credential exchange specifications](https://fidoalliance.org/specifications-credential-exchange-specifications/)

- ## What websites can see: authenticator flags

	- | Flag | Meaning |
	  | --- | --- |
	  | `UP` | User presence for the operation; not proof of a specific person's identity. |
	  | `UV` | Local user verification succeeded, such as a PIN or biometric check. |
	  | `BE` | Backup eligible; fixed when the credential is created. Not a promise of export to every provider. |
	  | `BS` | Currently backed up; can change. Eligibility alone does not mean a backup exists. |
	  | `AT` | Attested credential data is included—not proof of trusted remote attestation. |
	  | `ED` | Extension data is included. |
	- Ordinary WebAuthn login verification checks `UP`; a zero-valued flag is not general permission for unattended logins. Services can require `UV` and use backup state in policy. These flags do not enumerate installed apps or provide independent hardware certification. [W3C WebAuthn, §§6.1, 7.2](https://www.w3.org/TR/webauthn-3/)

- ## Attestation: real restrictions, overstated conclusions

	- **Real capability**: at registration, trusted attestation can provide evidence about the authenticator's provenance/model and associated security properties. A service can restrict acceptable authenticators; overly narrow allowlists can exclude users and future devices. WebAuthn's default attestation preference is `none`. [Yubico guidance](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/Attestation.html)
	- **Important distinction**: ordinary authenticator attestation is not a live audit of the entire operating system, browser extensions, or every installed application. Broader device-integrity policies require additional mechanisms; using a passkey does not by itself grant that visibility. This distinction follows from attestation's registration-time scope. [Yubico guidance](https://developers.yubico.com/WebAuthn/WebAuthn_Developer_Guide/Attestation.html)
	- A service may later change its acceptance policy or require re-enrollment, but cannot retroactively manufacture attestation evidence that an existing credential never supplied. The article itself corrects its earlier suggestion that websites can directly query everyone's future attestation compatibility.
	- Hardware protection is not absolute immunity: resisting extraction does not prevent a compromised device from abusing an authorized session or potentially invoking a key. Nor does attestation guarantee that a device will never be compromised.

- ## TOTP comparison: short-lived code, long-lived seed

	- TOTP derives a code from a **shared secret seed and time**, normally using 30-second steps. Code generation works offline; compatible apps can use the same seed. The service also needs the shared secret or equivalent protected access to verify codes. [RFC 6238](https://www.rfc-editor.org/info/rfc6238/)
	- Photographing a displayed code exposes a short-lived value; photographing the setup QR code or stealing the seed enables generating future codes until the secret is replaced. A verifier may accept adjacent time steps for clock drift. Never confuse code expiry with seed expiry. [RFC 6238](https://www.rfc-editor.org/info/rfc6238/)
	- Password plus TOTP improves on a password alone, but an attacker can relay both through a fake login page in real time. Passkeys address this phishing weakness; the article acknowledges this in its addendum. Copying TOTP seeds for convenience also copies durable account access.

- ## Before relying on a passkey: user checklist

	- **Locate it**: record which provider or hardware key holds each important account's credential. Check the save-location prompt rather than accepting an unexpected default.
	- **Test recovery before removing alternatives**: register a second independent authenticator where supported; protect recovery codes separately. Two synced devices under one provider are not fully independent of that provider account's recovery process.
	- **Protect the provider account and device lock**: syncing shifts some risk to vault access, device enrollment, and account recovery. Secure recovery email and avoid weak fallback routes that bypass the stronger login.
	- **Test migration before wiping or selling a device**: confirm access from the replacement, using export/import or a separately registered credential. If export is unavailable, add a new passkey while the old login still works.
	- **Distinguish cross-device login from migration**: a nearby phone can approve a laptop login without copying its passkey to the laptop. QR/Bluetooth proximity flows are authentication, not backup. [Google security explanation](https://security.googleblog.com/2023/05/so-long-passwords-thanks-for-all-phish.html)
	- **Revoke at the service when access is compromised**: remove the affected credential and review active sessions/recovery methods. Do not assume deleting a local copy revokes other copies or existing sessions. Prefer separate accounts/delegation over sharing a private credential.
	- **Keep endpoint defenses**: passkeys do not eliminate malware, stolen sessions, or scams that persuade you to perform actions after a legitimate login.

- **Related**: [[2026-07-passing-on-passwords-passkeys]]. Clarification to that earlier note: origin binding concerns the service, not necessarily the physical device.
