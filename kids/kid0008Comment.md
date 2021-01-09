# KID0008 - Key-Event State Machine - Commentary

## Navigation

[Back to table of contents](readme.md)
|Link|Commentary|Section
|---|---|---|
|[0000](kid0000.md)|[X](kid0000Comment.md)|Glossary, overview, how to use|
|[0001](kid0001.md)|[X](kid0001Comment.md)|Prefixes, Derivation and derivation reference tables|
|[0002](kid0002.md)|[X](kid0002Comment.md)|Data model (field & event concepts and semantics)|
|[0003](kid0003.md)|[X](kid0003Comment.md)|Serialization|
|[0004](kid0004.md)|[X](kid0004Comment.md)|Key Configuration (Signing threshold & key set)|
|[0005](kid0005.md)|[X](kid0005Comment.md)|Next Key Commitment (Pre-Rotation)|
|[0006](kid0006.md)|[X](kid0006Comment.md)|Seals|
|[0007](kid0007.md)|[X](kid0007Comment.md)|Delegation (pending PR by Sam)|
|[0008](kid0008.md)|X|Key-Event State Machine|
|[0009](kid0009.md)|[X](kid0009Comment.md)|Indirect Mode & Witnesses|
|0010||Recovery/consensus Algorithm (KAACE)|
|0011||Database & Storage Considerations|
|0097|n/a|**Non-Normative** Implementation Guidance|
|0098|n/a|Use Cases|
|0099|n/a|Test Vectors and Normative Statement Index|

### Rationale for Direct-mode Replay (from Whitepaper, section 10.1)

With direct replay mode, the first party is a controller of a given identifier and wishes to interact with a second party, the validator, using that given identifier. The second party must first validate that the given identifier is under the control of the first party i.e. establish control authority. Because the identifier (self-certifying) is bound to one or more key-pairs via the corresponding public keys, the controller may establish control by sending a verifiable inception operation event message signed with the corresponding private keys. From the perspective of the validator, this inception event establishes the controller’s initial authority over the identifier at issuance. This control authority is subsequently maintained via successive verifiable rotation operation events signed with unexposed pre-rotated keys (recall the description of pre-rotation above).

The controller also establishes the order of events. Because only the controller may create verifiable key events, the controller alone is sufficient to establish an authoritative order of events, i.e. it is the sole source of truth for events. No other source of truth with respect to ordering is necessary. The controller primarily establishes an event sequence by including in each event, except the inception event, a backward cryptographic commitment (digest) to the contents of the previous event. Secondarily each event also includes a monotonically increasing sequence number (non-wrapping whole number counter). The counter simplifies application programmer interfaces (APIs) for managing and reasoning about events. The inclusion of a commitment (digest) to the content of the previous event effectively backward chains the events together in an immutable sequence. 

The validator needs access to a record or log of the key event messages in order to initially verify the provenance of the current controlling key-pair for the identifier. This is a key event log (KEL). Therefore, as long as the validator maintains a copy of the original event sequence as it first receives it, the validator will be able to detect any later exploit that attempts to change any of the events in that sequence. Moreover, with a backward chained key event sequence, possession of only the latest digest from all the events allows the detection of tampering of any earlier event in any other copy of the key event sequence that is later presented to the validator i.e. duplicity detection. This means a validator does not need to maintain strict custody over its full copy of the KEL but merely the final event in order to detect tampering in any other full copy. To clarify, the validator still needs access initially to a copy of all the events in order to verify and establish control authority but should the validator lose custody of the full event log after validation and only maintain thereafter custody of the last event it has seen it may re-establish the validity of some other copy by backwards validation via the digest from its preserved event. In this case the sequence number allows a validator to more easily establish if a different event log has been tampered with prior to its last event by examining the event with the same sequence number in the alternate sequence to see if its digest is the same and validates against its previous event and so forth.

To elaborate, as long as the validator maintains a copy the KEL, an exploiter may not establish control of the identifier due to compromise from an exposure exploit of some earlier event in the KEL. For example, a later compromise of the original key-pair could be used to forge a different inception event. As long as the validator has a copy of the original inception event it could detect the forged inception event and ignore it. Likewise later compromise of any of the exposed keys could be used to forge different rotation events. As long as the validator at one time had access to a copy of the original chained key rotation event sequence starting with the original inception event it could detect the exploited rotation events and ignore them. Absent any other infrastructure, in order that the validator obtain a complete event log, the controller must ensure that the validator has received each and every rotation event in sequence. This requires an acknowledged transfer of each new rotation event. In order to ensure that this occurs the controller and validator must both be communicating directly to each other, thus online, at the time of the transfer. If either party becomes unavailable the interaction pauses and the other party must wait until both parties are online to resume the interaction. Consequently this case is only useful for interactions where pausing and resuming (i.e. intermittent availability) is acceptable behavior.

Upon reception of an event the validator sends a receipt message as an acknowledgment. The receipt message includes a signature by the validator of the associated key event message, in other words, an event receipt. The controller now has a signed receipt that attests that the validator received and verified the key event message. The validator is thereby in this narrow sense also acts like a witness of the event. The controller can keep the receipt in a key event receipt log (KERL). This provides a trust basis for transactions with the validator. The validator may not later repudiate its signed receipts nor refer to a different key event history in interactions with the controller without detection by the controller. By virtue of choosing to directly communicate to a validator, the controller is implicitly designating that validator as a witness of the key event stream. 

Each party could establish its own identifier for use with the other in this pair-wise interaction. Each party would thereby in turn be the controller for its own identifier and the validator and witness for the other’s identifier. Each could maintain a log of the key events for the other’s identifier and key event receipts from the other for its own identifier thereby allowing each to be invulnerable to subsequent exploit of the associated keys with respect to the pair-wise interaction. A log that includes both the signed key events (signed by the controller) and signed key event receipts (signed by the validator) is a log of doubly signed receipts of key events. Each is implicitly designating the other as a witness of its own key events. Any transactions conducted with the associated keys within the time-frame maintained by the logged key event histories may thereby be verified with respect to the keys without the need for other infrastructure such as a distributed consensus ledger. A discussion of how to verify associated transaction events is provided later. Furthermore the validator could keep a log of duplicitous events sent to it directly from the controller e.g. a duplicitous event log (DEL). With this the validator could prove that the controller has been exploited.

Of particular concern with this approach is the original exchange of the inception event. In a direct interaction, however, the controller may create a unique identifier for use with the associated validator and thereby a unique inception event for that identifier. This inception event is therefore a one time, place, and use event. Consequently, as long as the validator retains a copy of the original inception event, (or a copy of any later event), the inception event itself is not subject to later exploit due to exposure and compromise from subsequent usage of the originating key-pair. Another way of looking at this approach is that each pair-wise relationship gets a unique set of identifiers and associated key-pairs for each party.

The exchange of the inception event message must also be made invulnerable to man-in-themiddle attacks (for example by using multi-factor authentication) otherwise an imposter (manin-the-middle) could create a different identifier under its control and confuse the validator about the correct identifier to use in interactions with the genuine controller. A diagram of the direct replay mode infrastructure is shown below.

![](https://i.imgur.com/j7QikUW.png)