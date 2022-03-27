# OWASP ModSecurity Core Rule Set - DoS Protection Plugin

## Description of Mechanics

When a request hits a non-static resource (`TX:STATIC_EXTENSIONS`), then a counter for the IP address is being raised (`IP:DOS_COUNTER`). If the counter (`IP:DOS_COUNTER`) hits a limit (`TX:DOS_COUNTER_THRESHOLD`), then a burst is identified (`IP:DOS_BURST_COUNTER`) and the counter (`IP:DOS_COUNTER`) is reset. The burst counter expires within a timeout period (`TX:DOS_BURST_TIME_SLICE`).

If the burst counter (`IP:DOS_BURST_COUNTER`) is greater equal 2, then the blocking flag is being set (`IP:DOS_BLOCK`). The blocking flag (`IP:DOS_BLOCK`) expires within a timeout period (`TX:DOS_BLOCK_TIMEOUT`). All this counting happens in phase 5.

There is a stricter sibling to this rule (9514170) in paranoia level 2, where the burst counter check (`IP:DOS_BURST_COUNTER`) hits at greater equal 1.

### Blocking

The blocking is done in phase 1: When the blocking flag is encountered (`IP:DOS_BLOCK`), then the request is dropped without sending a response. If this happens, then a counter is raised (`IP:DOS_BLOCK_COUNTER`).

When an IP address is blocked for the first time, then the blocking is reported in a message and a flag (`IP:DOS_BLOCK_FLAG`) is set. This flag expires in 60 seconds.

When an IP address is blocked and the flag (`IP:DOS_BLOCK_FLAG`) is set, then the blocking is not being reported (to prevent a flood of alerts). When the flag (`IP:DOS_BLOCK_FLAG`) has expired and a new request is being blocked, then the counter (`IP:DOS_BLOCK_COUNTER`) is being reset to 0 and the block is being treated as the first block (-> alert).

In order to be able to display the counter (`IP:DOS_BLOCK_COUNTER`) and resetting it at the same time, we copy the counter (`IP:DOS_BLOCK_COUNTER`) into a different variable (`TX:DOS_BLOCK_COUNTER`), which is then displayed in turn.

### Variables

| Variable                   | Description                                                 |
| -------------------------- | ----------------------------------------------------------- |
| `IP:DOS_BLOCK`             | Flag if an IP address should be blocked                     |
| `IP:DOS_BLOCK_COUNTER`     | Counter of blocked requests                                 |
| `IP:DOS_BLOCK_FLAG`        | Flag keeping track of alert. Flag expires after 60 seconds. |
| `IP:DOS_BURST_COUNTER`     | Burst counter                                               |
| `IP:DOS_COUNTER`           | Request counter (static resources are ignored)              |
| `TX:DOS_BLOCK_COUNTER`     | Copy of `IP:DOS_BLOCK_COUNTER` (needed for display reasons) |
| `TX:DOS_BLOCK_TIMEOUT`     | Period in seconds a blocked IP will be blocked              |
| `TX:DOS_COUNTER_THRESHOLD` | Limit of requests, where a burst is identified              |
| `TX:DOS_BURST_TIME_SLICE`  | Period in seconds when we will forget a burst               |
| `TX:STATIC_EXTENSIONS`     | Paths which can be ignored with regards to DoS              |

As a precondition for these rules, please set the following three variables:

- `TX:DOS_BLOCK_TIMEOUT`
- `TX:DOS_COUNTER_THRESHOLD`
- `TX:DOS_BURST_TIME_SLICE`

And make sure that `TX:STATIC_EXTENSIONS` is also set.

## License

Copyright (c) 2022 OWASP ModSecurity Core Rule Set project. All rights reserved.

The OWASP ModSecurity Core Rule Set and its official plugins are distributed under Apache Software License (ASL) version 2. Please see the enclosed LICENSE file for full details.
