- **Source**: Eric Rescorla, [RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3](https://www.rfc-editor.org/rfc/rfc8446.html), August 2018—especially Sections 4.1.2, 4.2.1, 5.2, and Appendix D.
- **One-liner**: TLS 1.3 negotiates its real version in an extension while keeping legacy fields and optional handshake signals looking like TLS 1.2, allowing modern peers to select TLS 1.3 and older peers to fall back normally.

- ## Why TLS 1.3 Does Not Simply Advertise “1.3”
	- Earlier TLS designs put the highest supported version directly in the `ClientHello` version field.
	- During TLS 1.3 deployment testing, many servers and network middleboxes rejected unfamiliar version values instead of ignoring or forwarding them.
	- TLS 1.3 therefore avoids putting `0x0304` directly in the legacy version field. This works around **protocol ossification**: infrastructure that assumes today’s protocol details can never change.

- ## How Version Negotiation Works
	- A TLS 1.3-capable client sends a normal-looking `ClientHello` with:
		- `legacy_version = 0x0303`, the code for TLS 1.2.
		- A `supported_versions` extension listing the versions it can use, such as TLS 1.3 (`0x0304`) followed by TLS 1.2 (`0x0303`).
		- Other extensions carrying TLS 1.3 parameters, such as supported key shares and signature algorithms.
	- **If the server supports TLS 1.3**:
		- It selects TLS 1.3 in its own `supported_versions` extension.
		- Its `ServerHello.legacy_version` still says TLS 1.2, so the extension—not the legacy field—is authoritative.
	- **If the server supports only TLS 1.2**:
		- A compliant server ignores the unfamiliar extension, reads the TLS 1.2 legacy version, and negotiates TLS 1.2 normally.
		- No second connection attempt is normally required; the same handshake continues using the mutually supported older version.

- ## Middlebox Compatibility Mode
	- Version negotiation and middlebox compatibility are related but distinct mechanisms.
	- To make a TLS 1.3 handshake resemble TLS 1.2 session resumption:
		- The client places a non-empty, usually fresh 32-byte value in `legacy_session_id`.
		- A TLS 1.3 server echoes that value in `legacy_session_id_echo`.
		- The peers send dummy `ChangeCipherSpec` records at specified points. TLS 1.3 endpoints ignore these records.
	- An older TLS 1.2 server may interpret the unknown session ID as a failed resumption attempt and proceed with a full TLS 1.2 handshake.
	- A TLS 1.3 server is **not actually resuming a TLS 1.2 session**; these legacy-looking fields are camouflage for brittle middleboxes.

- ## Why Encrypted TLS 1.3 Traffic Also Looks Familiar
	- After encryption begins, every TLS 1.3 ciphertext record has:
		- An outer content type of `application_data`.
		- A legacy record version of `0x0303` (TLS 1.2).
	- The real inner content type—handshake, alert, or application data—is encrypted inside the record.
	- Middleboxes therefore see an ordinary TLS 1.2-shaped encrypted stream, while the endpoints process it as TLS 1.3.

- ## Mental Model
	- TLS 1.3 uses a **TLS 1.2-shaped envelope** for compatibility, but negotiates and authenticates TLS 1.3 semantics inside the handshake.
	- The `supported_versions` extension answers “which protocol are we using?”; the legacy version, session ID, dummy `ChangeCipherSpec`, and record headers mainly prevent old infrastructure from breaking the connection.
