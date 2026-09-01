# nxcenum

Bash wrapper around [netexec](https://www.netexec.wiki/) to quickly scope how far a known/compromised
credential reaches across multiple services (SMB, LDAP, WinRM, RDP, MSSQL, SSH) in a single run.

Given one confirmed username/password (or a set of targets), `nxcenum` runs the auth check against
every requested protocol, and when SMB access succeeds it automatically lists the accessible shares.
Built for the "I have valid creds, how far do they go" step of an authorized penetration test or
Active Directory assessment.

## Features

- Checks one credential against multiple protocols (`smb`, `ldap`, `winrm`, `rdp`, `mssql`, `ssh`, or `all`) in one command.
- Accepts a single target or a file with one target per line.
- Automatically enumerates SMB shares when authentication succeeds.
- Highlights successes (`[+]`), failures (`[-]`) and admin-equivalent access (`(Pwn3d!)`) in color.
- Explicit `[!] No response` message when a protocol/port doesn't answer, instead of silent output.
- Filters out noisy Python `SyntaxWarning` lines from underlying dependencies (impacket/lsassy).
- Optional `-o <file>` flag to save clean, color-free output for reporting.

## Installation

Requires [netexec](https://github.com/Pennyw0rth/NetExec) to be installed and available in your `PATH`.

```bash
git clone https://github.com/<your-username>/nxcenum.git
cd nxcenum
chmod +x nxcenum
sudo mv nxcenum /usr/local/bin/
```

## Usage

```
nxcenum <protocols|all> <targets> -u <username> -p <password> [-o <output_file>]
```

- `<protocols>`: comma-separated list (`smb,ldap,winrm`) or `all`.
- `<targets>`: a single IP/hostname, or a path to a file with one target per line.
- `-u` / `-p`: the credential to test.
- `-o` (optional): save plain-text results to a file, in addition to the colored terminal output.

### Examples

Check every supported protocol against a single host:
```bash
nxcenum all 10.10.10.10 -u bob -p 'P@ssw0rd!'
```

Check only SMB and WinRM against a list of targets, saving the results for a report:
```bash
nxcenum smb,winrm targets.txt -u bob -p 'P@ssw0rd!' -o results.txt
```

## Sample output

```
══════ Protocolo: SMB ══════
  -> Target: 10.10.10.10
SMB   10.10.10.10   445   DC01   [+] corp.local\bob:P@ssw0rd! (Pwn3d!)
     [+] Acceso SMB confirmado, enumerando shares en 10.10.10.10
SMB   10.10.10.10   445   DC01   [*] Enumerated shares
SMB   10.10.10.10   445   DC01   Share   Permissions   Remark
...
```

## Notes

- Does not pre-check ports; netexec runs against every target regardless of port state, so output stays
  consistent across the whole target list.
- The `-o` output file is always plain text (ANSI color codes stripped), safe to paste directly into a report.

## Disclaimer

This tool is intended exclusively for authorized security testing (penetration tests, red team
engagements, CTFs, or your own lab environments) where you have explicit permission to test the target
systems. Using it against systems you do not own or lack authorization for is illegal. The author is
not responsible for misuse.

## Credits

This tool started as a rewrite of [NTHSec/nxcspray](https://github.com/NTHSec/nxcspray), which inspired
the original protocol-sweeping approach. `nxcenum` has since diverged significantly in scope and
functionality (credential scoping instead of spraying, automatic SMB enumeration, output filtering,
color highlighting, file export), but credit belongs to the original author for the idea.

## License

MIT, see [LICENSE](LICENSE).
