# AD Learning Path 44 — Validate and Troubleshoot AD Replication

## Objective
Assess inbound and outbound AD replication between `DC01` and `DC02`, identify failures by naming context and partner, remediate the underlying cause, and prove convergence with a controlled test object.

## Prerequisites
- Two writable domain controllers
- Healthy DNS and time synchronization
- Domain administration rights
- Baseline replication output captured before fault testing

## Workflow
1. Run a forest-wide replication summary and inspect each DC's inbound partners.
2. Export connection objects and queued operations.
3. Run the `dcdiag` replication test and review Directory Service, DFS Replication, DNS, and System events.
4. Classify failures by DNS, time, RPC/firewall, authentication, topology, or lingering-object risk.
5. Correct the root cause before forcing synchronization.
6. Create a disposable OU on one DC, synchronize, and query it through both DCs.
7. Compare object metadata and timestamps to prove convergence.

```powershell
repadmin.exe /replsummary
repadmin.exe /showrepl * /csv | Out-File .\evidence\showrepl.csv
repadmin.exe /showconn *
repadmin.exe /queue *
dcdiag.exe /test:replications /v

New-ADOrganizationalUnit -Name 'Replication-Test' `
    -Path 'DC=corp,DC=lab' -ProtectedFromAccidentalDeletion $false
repadmin.exe /syncall /AdeP
```

## Validation
```powershell
Get-ADOrganizationalUnit -Identity 'OU=Replication-Test,DC=corp,DC=lab' -Server DC01
Get-ADOrganizationalUnit -Identity 'OU=Replication-Test,DC=corp,DC=lab' -Server DC02
repadmin.exe /showobjmeta DC01 'OU=Replication-Test,DC=corp,DC=lab'
repadmin.exe /showobjmeta DC02 'OU=Replication-Test,DC=corp,DC=lab'
repadmin.exe /replsummary
```

## Evidence
Store baseline and final replication summaries, `showrepl` CSV, connection/queue output, relevant events, test-object metadata, root-cause timeline, remediation, and final pass/fail result under `evidence/`.

## Troubleshooting
- 1722/RPC errors: verify DNS, routing, firewall, and RPC endpoint reachability.
- Access or Kerberos errors: validate time, SPNs, secure channels, and DC account state.
- Repeated KCC/topology errors: inspect Sites and Services rather than creating arbitrary manual connections.

## Security notes
Do not use forced replication as a permanent workaround. Preserve evidence before deleting metadata or changing topology.

## Cleanup
Delete the `Replication-Test` OU after successful convergence and evidence capture.

## References
- Microsoft Learn: `repadmin`
- Microsoft Learn: Troubleshoot AD replication

## Next activity
`AD-Learning-Path-45-Configure-AD-Sites-and-Subnets`
