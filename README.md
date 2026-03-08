Tor protects your privacy on the internet by hiding the connection between
your Internet address and the services you use. We believe Tor is reasonably
secure, but please ensure you read the instructions and configure it properly.

## Build

To build Tor from source:

```
./configure
make
make install
```

To build Tor from a just-cloned git repository:

```
./autogen.sh
./configure
make
make install
```

## Releases

The tarballs, checksums and signatures can be found here: https://dist.torproject.org

- Checksum: `<tarball-name>.sha256sum`
- Signatures: `<tarball-name>.sha256sum.asc`

### Schedule

You can find our release schedule here:

- https://gitlab.torproject.org/tpo/core/team/-/wikis/NetworkTeam/CoreTorReleases

### Keys that CAN sign a release

The following keys are the maintainers of this repository. One or many of
these keys can sign the releases, do NOT expect them all:

- Alexander Færøy:
  [514102454D0A87DB0767A1EBBE6A0531C18A9179](https://keys.openpgp.org/vks/v1/by-fingerprint/1C1BC007A9F607AA8152C040BEA7B180B1491921)
- David Goulet:
  [B74417EDDF22AC9F9E90F49142E86A2A11F48D36](https://keys.openpgp.org/vks/v1/by-fingerprint/B74417EDDF22AC9F9E90F49142E86A2A11F48D36)
- Nick Mathewson:
  [2133BC600AB133E1D826D173FE43009C4607B1FB](https://keys.openpgp.org/vks/v1/by-fingerprint/2133BC600AB133E1D826D173FE43009C4607B1FB)

## Development

See our hacking documentation in [doc/HACKING/](./doc/HACKING).

---

## Research Fork - Traffic Correlation Instrumentation

This fork adds lightweight instrumentation for traffic correlation research
in Shadow-based simulations. **These modifications are not intended for
production use.** They emit additional control-port events that expose
internal circuit state and should never be deployed on the live Tor network.

### Overview

Each client circuit is assigned a globally unique 64-bit
`research_id` at creation time. This identifier is propagated to every relay
hop via a custom relay cell, allowing per-circuit bandwidth observations from
multiple vantage points to be joined without ambiguity.

### New relay cell: `RELAY_COMMAND_UPDATE_RESEARCH_ID` (command 67)

Sent by the origin client to each extended hop of a circuit.
The payload is the 8-byte big-endian
`research_id`. Upon receipt, a relay stores the value in `circuit_t.research_id`
and emits a `CIRC RESEARCH_ID_UPDATED` control-port event.

### New control-port events

**On origin clients**:

```
650 CIRC RESEARCH_ID_CHOSEN LocalCircID=<id> ResearchID=<16-hex-digits>
```

Emitted when a new circuit is created and its `research_id`
is assigned.

**On relay nodes**:

```
650 CIRC RESEARCH_ID_UPDATED LocalOrCircID=<id> ResearchID=<16-hex-digits>
```

Emitted once per relay when the `RELAY_COMMAND_UPDATE_RESEARCH_ID` cell
arrives, confirming the relay has received and stored the research ID.

**Bandwidth events on relay nodes** (`CIRCUIT_PURPOSE_OR`, `research_id != 0` only):

```
650 CIRC_BW OR_STAT OR_CIRC_ID=<id> READ=<bytes> WRITTEN=<bytes> ResearchID=<16-hex-digits> TIME=<iso-timestamp>
```

Mirrors the origin-side `CIRC_BW` event. Emitted periodically for every OR
circuit that has transferred bytes since the last emission. `READ` counts 
bytes arriving from the client-ward side; `WRITTEN` counts bytes arriving 
from the exit-ward side. Counters reset to zero after each event.

`OR_CIRC_ID` is a per-process monotonically increasing identifier stored in
a new field `or_circuit_t.or_circuit_id`. It behaves identically to 
`origin_circuit_t.global_identifier`

## Resources

Home page:

- https://www.torproject.org/

Download new versions:

- https://www.torproject.org/download/tor

How to verify Tor source:

- https://support.torproject.org/little-t-tor/

Documentation and Frequently Asked Questions:

- https://support.torproject.org/

How to run a Tor relay:

- https://community.torproject.org/relay/