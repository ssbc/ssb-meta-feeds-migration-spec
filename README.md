# SSB metafeed migration path

In this document we will describe how to migrate classic SSB, with one
feed per device, to a world with multiple feeds using metafeeds. We
will also define what that means in terms of ID. Note, this document
does not define behavior for applications that do not have to interact
with classic SSB.

## Classic SSB

Classic SSB refers to feeds as defined in the [protocol guide], and
the key used to sign these feeds. In most cases people only have one
feed per device, so let us assume that model in this document. The key
used to sign messages on a feed is the identity of the device, the id
when connecting to other peers using muxrpc. Finally it is also used
for mentions and contacts messages to define the friend relationship.

## Metafeeds

A metafeed is a tree of feeds per device. There is a single feed at
the root of the tree and from this any number of nested subtrees or
subfeeds can exist. These subfeeds can either be new feeds or existing
feeds as defined in the [metafeeds spec].

In order to signal that someone has started using metafeeds, a classic
SSB feed is linked to a metafeed as described in the [existing SSB
identity part of the metafeeds spec]. After migrating, the identity of
the device ought to be the feed id of the root metafeed, but in order
to ease the transition, the classic ID is used until such a time as
the main feed is no longer the classic feed. The identity section in
this specification further refines the ID concept into multiple areas.

### Identity

As described earlier the ID of a device can now be seen as:

- The *signing key*, because we have many feeds, there are now multiple
  signing keys one for each feed.
- The *DM key* used to encrypt and decrypt private messages.
- The *external identity* used to describe yourself to people wanting
  to connect to you, is still the classic ID of the main feed.
- The *network identity* will for now be the classic ID. When the
  [network identity spec] is implemented there can be multiple
  identities all tying back to the same metafeed. Note that this might
  cause connection problems with older clients, so should only be
  implemented at such a time when a majority of the network supports
  metafeeds.
- Mentions should for now also still use the classic ID.
- The friend relationship as defined by contacts messages should
  similar to mentions use the classic SSB id for now. What happens
  after the main feed is changed has not yet been defined.

For multiple devices to act as a single identity the [fusion identity
spec] should be used. This spec defines a new fusion identity that can
be used for: private messages, external identity, mentions and friend
relationship.

### Tree structure v1

The tree should be structured as defined in the [v1 part] of the
metafeeds spec. Furthermore it should use the [group spec] for how to
structure the feeds used for private groups.

FIXME: Consider moving the whole v1 over here from the metafeeds
spec. And use a better example with index feeds, the main feed and
groups.

### Main feed

There should only be 1 (non-tombstoned) main feed in the tree and it
should use the name 'main feed' to define its position in the
tree. See v1 section above.

FIXME: example of such a message

### Indexes

Applications should define one v1 [index feed], an index of their
classic SSB main feeds about messages. The name 'index feed about'
should be used to define its position in the tree. See v1 section
above.

FIXME: example of such a message

### Network replication

In classic SSB you replicate feeds in full as defined by your hops
settings. Metafeeds allows one to do partial replication. This could
be both a part of the tree for feeds within your hops distance, such
as not replicating private group feeds you are not a part of, or it
could be only replicating the index feed and common private groups for
feeds outside of your hops range. Clients are expected to replicate as
much of the tree that they can within their hops range to increase the
likelihood that content will be available when replicating with peers.

Lets consider the scenarios:

 - full replication: you start out by replicating the main feed as you
   know this id from the friend relationship. Once you discover a
   message with type 'metafeed/announce' on the main feed, you start
   replicating this metafeed and in turn all subfeeds including index
   feeds using normal EBT replication.
 - partial replication of a device with private groups in common: the
   root metafeed id should be given from a group/add-member
   message. From this the tree structure of the metafeed can be
   replicated including index feeds and the feeds of private groups in
   common, but not the main feed. To replicate the main feed you
   follow them.
 - partial replication of a device outside your hops range: a client
   can choose to replicate feeds at max hops + 1 using partial
   replication. To discover the metafeed id we use the RPC `getSubset`
   from [subset replication] on any connected peer, asking for a
   message of type 'metafeed/announce' on the main feed. If connected
   peers do not support this RPC, then we can fallback to
   createHistoryStream to fetch the full feed in memory (i.e. without
   persisting it to the log). From this we persist the
   'metafeed/announce' message and from there we can replicate the
   metafeeds tree and the about index feed.

[protocol guide]: https://ssbc.github.io/scuttlebutt-protocol-guide/#feeds
[metafeeds spec]: https://github.com/ssbc/ssb-meta-feeds-spec
[existing SSB identity part of the metafeeds spec]: https://github.com/ssbc/ssb-meta-feeds-spec/#existing-ssb-identity
[network identity spec]: https://github.com/ssbc/ssb-network-identity-spec
[fusion identity spec]: https://github.com/ssbc/fusion-identity-spec
[v1 part]: https://github.com/ssbc/ssb-meta-feeds-spec/#v1
[group spec]: https://github.com/ssbc/ssb-meta-feed-group-spec
[index feed]: https://github.com/ssbc/ssb-secure-partial-replication-spec#indexes
[subset replication]: https://github.com/ssbc/ssb-subset-replication-spec
