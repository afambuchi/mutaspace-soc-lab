# Mini PC Prototype Role

## Purpose

The mini PC is being kept as the prototype environment for the MutaSpace SOC Lab.

The previous lab was intentionally destroyed because it had served its original purpose as an early learning and testing environment. The mini PC will continue to be useful, but it will no longer be treated as the official lab platform.

## Why Keep the Mini PC?

The mini PC gives this project a safe place to:

1. Test Proxmox networking changes.
2. Practice VM creation.
3. Break and rebuild configurations.
4. Validate documentation before applying it to the official host.
5. Learn concepts before making the clean production-style version.

## Role Separation

| System | Role | Standard |
|---|---|---|
| Mini PC | Prototype lab | Flexible, experimental, allowed to break |
| New dedicated PC | Official MutaSpace SOC Lab | Clean, documented, reproducible |

## Documentation Rule

Any major step should be tested or reasoned through on the mini PC first when practical.

After the step is understood, it should be documented clearly and repeated on the official dedicated PC.

## Learning Note

A prototype environment is not a failure. It is where learning happens before a system becomes reliable enough to teach from.