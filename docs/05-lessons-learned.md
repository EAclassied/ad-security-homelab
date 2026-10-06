# Lessons Learned

A short reflection on what this project actually reinforced, beyond the
technical steps documented elsewhere.

## Troubleshooting is mostly about isolating variables

Nearly every issue in this lab (the dual-NIC mixup, the GPO not applying,
the DNS fallback masking a real failure, the SMB service "stopped" but
still listening, the Kerberoasting clock skew/encryption type/hash format
chain) was solved the same way: narrow down what's actually true by testing
one layer at a time — connectivity, then name resolution, then the
application layer — rather than guessing at the fix first. That pattern
is the same whether the "ticket" is a staged lab scenario or a tool
genuinely failing for reasons that weren't obvious at first glance.

## Defaults matter

Several of the most instructive moments came from Windows/AD defaults
behaving in ways that weren't obvious:

- `New-ADUser` silently defaults to the built-in `Users` container unless
  `-Path` is specified, which breaks GPO application in a way that gives
  no direct error message.
- NetBIOS/LLMNR fallback can mask a real DNS failure on a flat network,
  making a broken configuration look like it's working.
- A stopped Windows service doesn't always mean its port closes
  immediately — the SMB kernel driver held the listening socket past what
  the service status reported.

None of these are exotic edge cases — they're the kind of thing that shows
up in real help-desk tickets, and the lab surfaced all of them without any
of it being staged.

## Security controls are layered, and it's worth testing each layer

The file share exercise made this concrete: a GPO drive map is a
convenience, not a security boundary. Testing access three ways (as an
authorized user, as an unauthorized user, and via direct UNC path instead
of the mapped drive) confirmed the actual enforcement was happening at the
share/NTFS permission layer, not the GPO. Assuming a control works because
it *looks* like it's working, without testing the failure case, is a real
gap worth being deliberate about avoiding.

## Offline attacks bypass online protections entirely

The domain password policy (minimum length, complexity, lockout threshold)
provided zero protection against Kerberoasting, because that policy only
governs interactive authentication attempts. Once a ticket is extracted,
cracking happens entirely offline with no further contact with the domain
controller. This reframed, for me, what "strong password policy" actually
covers versus what it doesn't — service accounts need a fundamentally
different mitigation (long random passwords, or Managed Service Accounts)
rather than relying on the same policy that protects regular user logins.

## Next steps

Possible future additions to this lab: a second domain controller for
replication practice, AD Certificate Services (to properly enable LDAPS,
which this environment currently has listening but non-functional),
Windows Event Forwarding for centralized logging, and BloodHound for
visualizing attack paths across a larger, more realistic AD structure.
