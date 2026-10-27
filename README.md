# Cisco VPN Home Network Stability Check

A read-only PowerShell tool that tests whether a home network is stable enough for Cisco Secure Client VPN, then explains the result in plain English.

VPN drops are often blamed on the VPN when the real cause is weak Wi-Fi, a flaky router, or an unstable internet connection. This script tells you which one it is.

## What it checks

| Test | Target | What it measures |
| --- | --- | --- |
| Local router | Default gateway | Packet loss, latency, jitter |
| Internet | 1.1.1.1 | Packet loss, latency, jitter |
| VPN gateway DNS | Gateway from the Cisco profile | Whether the name resolves |
| VPN gateway TCP 443 | Gateway from the Cisco profile | Connection success rate, latency, jitter |
| Wi-Fi signal | Active wireless adapter | Signal strength (%) |

The VPN gateway is detected automatically from the Cisco Secure Client (or AnyConnect) profile on the machine. You can also pass it in manually.

## Requirements

- Windows 10 or 11
- PowerShell 5.1 or PowerShell 7
- Cisco Secure Client installed (only needed for automatic gateway detection)

## How to run

1. Download `Collect-CiscoVpnDiagnostics.ps1`.
2. If Windows blocked the download, right-click the file, choose **Properties**, check **Unblock**, and click OK. (Or run `Unblock-File .\Collect-CiscoVpnDiagnostics.ps1`.)
3. Right-click the file and choose **Run with PowerShell**, or run it from a PowerShell window:

```powershell
.\Collect-CiscoVpnDiagnostics.ps1
```

With options:

```powershell
.\Collect-CiscoVpnDiagnostics.ps1 -VpnGateway vpn.company.com -SampleCount 20
```

| Parameter | Default | Description |
| --- | --- | --- |
| `-VpnGateway` | Detected from Cisco profile | VPN gateway hostname to test |
| `-SampleCount` | 20 (range 10 to 60) | Number of samples per test |
| `-OutputDirectory` | Desktop | Where the report is saved |

A run takes roughly a minute at the default sample count.

## How the verdict works

The script prints a verdict and saves a text report. Each result maps to a plain-English finding:

| Condition | Finding |
| --- | --- |
| No default gateway found | Device is not properly connected to the home network |
| Router loss of 5% or more, or jitter over 30 ms | Computer-to-router link is unstable (try Ethernet or move closer) |
| Wi-Fi signal under 65% | Weak signal that can cause VPN drops |
| Internet loss of 3% or more, jitter over 50 ms, or average latency over 150 ms | Internet connection is unstable (test Ethernet, contact the ISP) |
| VPN gateway name does not resolve | DNS, profile, or network issue |
| VPN gateway TCP 443 loss of 10% or more | Gateway intermittently unreachable (escalate as gateway, proxy, firewall, or routing issue) |
| None of the above | Network looks stable; investigate the Secure Client profile, certificate, MFA, endpoint security, or gateway logs |

If the router does not answer ping, the script notes that the router link can't be measured directly rather than treating it as a failure.

## Example output

*Illustrative example of the report format.*

```
===== VPN NETWORK VERDICT =====
ISSUE: Wi-Fi signal is weak (52%). This can cause VPN drops; test closer to the router or on Ethernet.

Report saved to: C:\Users\<you>\Desktop\VPN-Network-Stability-<COMPUTERNAME>-20261026-141500.txt
```

## Privacy and safety

- Read-only: it does not change network settings or Cisco configuration.
- It does not collect passwords, MFA codes, tokens, or browsing history.
- The report includes the computer name and the VPN gateway hostname, so review it before sharing outside your organization.

## Limitations

- Windows only (uses `netsh` and `Resolve-DnsName`).
- TCP tests confirm the network path to the gateway on port 443; they do not require or test VPN login.
- Results reflect a short test window. Intermittent problems may need a second run at a different time.

## License

MIT. See [LICENSE](LICENSE).
