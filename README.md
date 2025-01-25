# StageNet

StageNet is a protocol for simplifying reliable, real-time communication to multiple data sources using one unified system. To read the specification, see [PROTOCOL.md](PROTOCOL.md).

## About

At its core, you have a central console broadcasting messages to gateway boxes. These gateway boxes can operate multiple StageNet addresses and respond to messages directed at those addresses.

A common example in a theater setting would be a console broadcasting values to a DMX StageNet gateway, which communicates those signals over a DMX universe to connected fixtures. The gateway would operate 512 StageNet addresses, one for each DMX address in the universe.

You could also have StageNet-enabled power strip that hosts 6 StageNet addresses for switching the power to 6 different sockets.

## License
This repository is licensed under the [Creative Commons Attribution 4.0 International](LICENSE) license.
