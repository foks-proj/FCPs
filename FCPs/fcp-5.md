| FCP | 5 |
| :--- | :--- |
| **Title** | Public Teams and Users |
| **Author(s)** | Max Krohn [@maxtaco](https://github.com/maxtaco) /  [max@ne43.com](mailto:max@ne43.com) |
| **Status** | Proposed |
| **Track** | Standards |

## Abstract

Allow unauthenticated internet users to view the sigchains of certain users and teams. These public users and teams can make public, notarized signatures; these signatures can be verified by
any user. And finally, users and teams can advertise these public identities via [KeyOxide](https://keyoxide.org/).

## Motivation

Some users and teams might want to make public signatures and attestations. For example, they
might want to sign a document, a tweet, or a software package. Signers or groups of signers
have all the same problems FOKS users have today: they have multiple devices, and can form
mutable teams, or teams of mutable teams, etc. As such they must rotate keys, and consumers
of their signatures should have a way to verify acceptable keys without fear of server cheating.

FOKS currently does not support this use case, since privacy restrictions prevent it.
Bob can only see Alice's sigchain if: (1) Alice has explicitly allowed Bob to do so;
or (2) Bob and Alice are both on an "open view" server, and Bob has authenticated himself
to that server. There is good reason for this default configuration: Alice might not 
want all users on the internet to know when her sigchain gets updated, since those
updates typically imply addition or deletion of keys. And similarly, a team is by
default private, hiding from the public its membership, and when its membership
changes. 

This FCP covers changes to FOKS primitives to allow these public teams and users, and
furthermore, simple applications of these changes. Once public teams and users are
supported, now it makes sense for these public parties to issue public signatures.
And similarly, it makes sense for the public to be able to verify these signatures.
Similarly, it also makes sense for public parties to be able to _advertise_ their public
identities, so they can be cryptographically triangulated, as we see with Keybase and
[KeyOxide](https://keyoxide.org/). 

## Specification

### Changes to Sigchain Protocol

In [proto-src/lib/chains.snowp](https://github.com/foks-proj/go-foks/blob/main/proto-src/lib/chains.snowp):

```diff

+ enum ReadAccessType {
+     case None    @0; // to be used if no value is specified
+     case Private @1; // private read access (default if not specified)
+     case Public  @2; // public read access, this new feature
+ };

+ // Though this variant functions as an Enum now, in the future it
+ // may become more interesting for more interesting types of read access.
+ variant ReadAccess switch (t : ReadAccessType) {
+     default: void;
+ }

variant ChangeMetadata switch (t : ChangeType) {
    case DeviceName, Username, Teamname @0 : Commitment;
    case Eldest @1 : EldestMetadata;
    case TeamIndexRange @2 : RationalRange;
    case MemberLoadFloor @3 : Role;
+   case ReadAccess @4 : ReadAccess;
}
```

### Database Schema Changes

In `foks_users/p4.sql`:

```sql

CREATE TYPE read_access AS ENUM('private', 'public');

ALTER TABLE users
    ADD COLUMN read_access read_access NOT NULL DEFAULT 'private';
ALTER TABLE teams
    ADD COLUMN read_access read_access NOT NULL DEFAULT 'private';
```

### Changes Authentication and Access Controls and Server Configuration:

Both [server/shared.TeamLoader](https://github.com/foks-proj/go-foks/blob/main/server/shared/team_loader.go) and 
[server/shared.UserLoader](https://github.com/foks-proj/go-foks/blob/main/server/shared/user_loader.go) 
have `checkPerms` methods. In both cases, this method should be 
changed to allow tokenless public viewing if the underlying party
is set to `read_access = 'public'`.

Currently, `RegClientConn.ResolveUsername` always fails (with a permission
error) and `UserClientConn.ResolveUsername` needs an authenticated user
to work. Both need to be extended to allow the resolution of a username
to UID if the underlying user is set to `read_access = 'public'`.

### Rules for Composition of Public Teams

Public teams can be composed of public users and public teams, recursively.
Public teams and users can be members of private teams. Accordingly, once a team
or user becomes public, it is unallowable to change it back to private.


### Support For ASPE (KeyOxide)

The [ASPE](https://ariadne.id/related/ariadne-signature-profile-0/) protocol is
a way to advertise public identities and to therefore conform to the
[KeyOxide](https://docs.keyoxide.org/developers/write-aspe-server/) set of
standards. FOKS will support this protocol as part of its `web` server.  We
should take the opportunity to make some of the existing features of the web
server optional, especially for servers where its current features are not
needed.

### Public Signatures

Public users and teams can use their public keys to make public signatures.
As we will see below, there are various forms these signatures can take.
One issue to consider is the following: Alice is a public user, member of 
a public team Bills, which is in turn member of a public team Colts. Alice
wishes to sign a public statement on behalf of the Colts. Which signing key
does she use? The best options seem to be: (a) the Colts's latest PTK; or (b)
Alice's current device key. If several signatures are allowed, maybe all
three relevant keys should be signed, demonstrating the chain of authority.

Another issue to consider is revocation. Say Bob signs a statement at time
$t_1$, and later revokes the signing key at time $t_2$. Verifiers ought to be
able to verify that the signing key was valid at the time of the signature, even
though it was later revoked. Here, we can use the same Merkle Tree mechanism
that FOKS uses to order signatures.  In other words, it makes sense to layer
"notarization" on top of normal cryptographic signing. Notarized signatures
are buried in a server's Merkle Tree, allowing revocation statements to
transitively include hashes of everything the key signed while it was valid.

#### FOKS Notarized Signature Service

Public teams and users can make notarized signatures. These are signatures
that are reflected in the server's Merkle Tree. Once in the Merkle Tree,
the system can provide a cryptographic proof that the data predated
other events, without fear of backdating attacks. 

The system offers two varieties of notarized signatures: (1) linear; and (2)
random access. Linear signatures happen in a provable order, wherein it's
impossible to hide a signature after its introduction.  Random access
signatures, on the other hand, lack linearizability, and there is no way for an
outside observer to infer their number.  However, they do retain the property
that they can be proven to predate other events (like linear signatures).

##### Linear Signatures

Linear signatures use the same mechanism as other secondary sigchain 
chains in FOKS, like the `UserSettings` and `TeamMembership` chains.

We introduce two new `ChainType`s:

```diff
enum ChainType {
    User @0;
    Name @1;
    UserSettings @2;
    Team @3;
    TeamMembership @4;
+   NotarizedDataLinear @5; 
+   NotarizedDataRandomAccess @6;
}
```

And also we introduce a new `GenericLinkPayload` variant case:

```diff

+ struct StackedSigners {
+   entity @0 : FQEntity;
+   signer @1 : FQEntityInHostScope;
+ }

+ struct NotarizedDataLink {
+   prefixedHash @0 : StdHash;
+   innerSigners @1 : List(StackedSigners); // empty if only one signer
+}

variant GenericLinkPayload switch (t : ChainType) @0xfc82ed34909b2f4c {
    case UserSettings @0 : UserSettingsLink;
    case TeamMembership @1 : TeamMembershipLink;
+   case NotarizedDataLinear @2 : NotarizedDataLink;
    default: void;
}
```

Everything else follows preexisting patterns. Note that hash inside of the
`NotarizedDataLink` ought to be the output of a `PrefixedHash` operation: one in
which the `rpc.TypeUniqueID` is prepended to the actual data being hashed, to
achieve domain separation. The Snowpack specification does not enforce this, but
the FOKS implementation ought to.

##### Random Access Signatures

Random access signatures use the same `MerkleTreeRFInput` keys as normal signature chains,
but with `seqno=0` and `location` random and non-empty for all entries. Recall:

```
struct MerkleTreeRFInput @0xb0e268f388acc97a {
   ct @0 : ChainType;
   entity @1 : EntityID;
   seqno @2 : Seqno;
   location @3 : Option(TreeLocation); 
}
```

For the value, we introduce a new type of link, which doesn't have the seqno/prev 
structure of all other links we have seen so far:

```diff
enum LinkType {
    GROUP_CHANGE @1;
    GENERIC @2;
+   RANDOM_ACCESS @3;
}

variant LinkInner switch (t : LinkType) @0xacf9066572a9e7de {
    case GROUP_CHANGE @0 : GroupChange;
    case GENERIC @1 : GenericLink;
+   case RANDOM_ACCESS @2 : RandomAccessEntry;
}

+ struct RandomAccessHeader {
+    root @0 : TreeRoot;
+    time @1 : Time;
+    location @2 : TreeLocation; // matches *this* entry's location, not the next one's
+};

+ struct RandomAccessEntry {
+    header @0 : RandomAccessHeader;
+    entity @1 : FQEntity;
+    signer @2 : FQEntityInHostScope;
+    payload @3 : RandomAccessEntryPayload;
+ }

+ variant RandomAccessEntryPayload switch (t : ChainType) @0xc3f6cb96e183b68d {
+    case NotarizedDataRandomAccess @0 : NotarizedDataLink;
+    default: void;
+ }
```

Note that the Merkle construction pipeline will have to be amended to
accommodate the new `RandomAccessEntry` type. Previously, leaves are laid down
in linear order, as determined by incrementing `seqno` values. Now, the leaves
will be laid down in random order, but we need to ensure that they all make it
into the tree.

#### Production of Minisign-Style Signatures

Users and teams can produce
[minisign](https://jedisct1.github.io/minisign/)-style signatures using their
PTKs, PUKs, or device keys. For YubiKey devices, the delegated EdDSA subkey can
be used to produce the minisign-style signature.

Minisign-style signatures can be be notarized or not, and if notarized, can use
either the linear or random access signature mechanism described above. 

Whether notarized or not, the "untrusted comment" field of the minisign
signature contains the HostID and mode signature-specific key _id_ in the form
`foks:<id>@<hostID>`. The goal here is a hint to retrieve the relevant keys and/or notarization on
verification. If not notarized, the _id_ field is the `EntityID` of the signer.
If notarized, the _id_ field is the signature ID of the particular signature
(which in turn can be used to lookup the `EntityID` of the signer). In the
notarized case, the signature ID computed as the hash of the
`MinisignSignatureIDPayload` with the prefix `0x13`, base62-encoded:

```
variant MinisignSignatureLocation switch (t : ChainType) {
    case NotarizedDataLinear @0 : Seqno;
    case NotarizedDataRandomAccess @1 : TreeLocation;
}

struct MinisignSignatureIDPayload @0x8ea254111a138818 {
    entity @0 : FQEntity;
    device @1 : FQEntityInHostScope;
    loc @2 : MinisignSignatureLocation;
}

typedef MinisignSignatureID = EntityID;         // Leading byte 0x13 -- 33 Bytes (hash of MinisignSignatureIDPayload)
```

To notarize a minisign-style signature, put the entire text of the signature (including the untrusted comment)
into the following structure and hash; then use the output in a `NotarizedDataLink`:

```
typedef MinisignSignature @0xa128f36f46a9db01 = Text;
```

Note that to produce one notarized minisign signature, two actual signature are required. One
to make the inner minisign signature, and another to introduce it into the Merkle Tree chain
for notarization.

#### Production of FOKS-Style "Stacked" Team Signatures

FOKS-style signatures only come in the notarized variety (for now). To make such a signature,
first produce a prefixed hash from:

```
typedef GenericSignaturePayload @0xc1812d4b794e2f5c = Blob;
```

Then feed the hash into the `NotarizedDataLink` as the `prefixedHash` field. If using only one signer, 
the `innerSigners` field is empty. The multiple signers case is when Alice is acting on behalf of 
a path through the team DAG. The "outermost" signature is implied, performed with her device key.
Then PTKs along the path are listed in the `innerSigners` field, with the ultimate team
being listed first, and then traversing the path down to her user. Signatures are performed
in this order, with the target team signing first, and her user device signing last
(and over all other signatures). The signature is the entirety of the `LinkOuter` structure,
either in binary or base62-encoded.

#### Validation of Notarized Signatures 

To validate one of these signatures, the verifier must first fetch and playback the relevant
sigchains. We consider the following three cases:

* For non-notarized minisigs, fetch the `<entityID>@<hostID>` from the untrusted comment.
  Contact the server at _hostID_, fetch and replay the sigchain for _entityID_, 
  and then verify the signature.
* For notarized minisigs, fetch the `<signatureID>@<hostID>` from the untrusted comment.
  Contact the server at `hostID`, and fetch the `MinisignSignatureIDPayload` that hashes
  down to the `signatureID`. Verify the hash. Then, fetch and play the signature link
  of the signer advertised in the `MinisignSignatureIDPayload` as above. Finally,
  verify that the signature was produced after the signing key was introduced into
  the Merkle tree and before the signing key was revoked (if indeed it was revoked).
  This proof uses the same "bookending" approach currently used in the Team Membership
  and User Settings Chains.
* For FOKS-style signatures, the signature should contain the necessary information
  to determine the host and signer to fetch the necessary sigchains. This case
  follows much like the notarized minisig case above.

## Backwards Compatibility

There are no major backwards compatibility concerns. Users and teams are considered
to be private by default. Legacy clients can still load public users and teams if
using legacy credentialling methods. They will not be able to load public users and 
teams using "anonymous" credentials.

## Reference Implementation

In progress.

## Security Considerations

We should carefully consider the fields we are signing over, and various attacks
against the identity system (social or otherwise) that would weaken the signature system.
This work is left open for now, but has to be finalized before the FCP is activated.

## Needed Tests

* Signup as a public user; viewed by an unauthenticated anonymous user
* Public team, can be loaded by an unauthenticated anonymous user
* Public team DAG, can be loaded by an unauthenticated anonymous user
* Check that public team CLKRs work as regular CLKRs do.
* Check that private teams and users can become public, but not vice versa.

## CLI Changes

* CLI tools for creating and verifying signatures, whether minisign-style or FOKS-style
* Tools for converting users and teams to public visibility.
* Ability to list public teams
* Ability to load and examine public users
* Tools for introducing KeyOxide proofs for public users and teams

## Open Questions

* Should we consider a trimmed down version of FOKS-style signatures?
* Should there be the a non-notarized version of FOKS-style signatures?
* My hunch is that we'll want to jigger with the signature formats as we build
  this FCP.
* Is KeyOxide an active project? I made a profile but couldn't get it to load. I'm
  not sure if it worked.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
