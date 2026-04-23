# paket-strategy-min

## Probe metadata

| Field | Value |
|---|---|
| Pattern | strategy-min-vs-max |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | none |
| Resolution strategy | min (resolves to lowest matching version) |
| Dependency groups | Main only (no named groups) |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises Paket's `strategy: min` global resolution setting. With
`strategy: min`, Paket selects the lowest version of each package that satisfies
all constraints, rather than the latest compatible version (which is `strategy: max`,
Paket's default). The lockfile therefore contains older, minimum-matching versions
for all three direct dependencies and their transitives. Mend must read and report
exactly those locked versions — not re-resolve to latest — demonstrating correct
lockfile-driven detection behaviour.

## Expected dependency tree

### Direct dependencies (Main group)

| Package | Constraint | Resolved version (min) | Has transitives |
|---|---|---|---|
| Microsoft.Extensions.Logging | >= 6.0.0 | 6.0.0 | Yes |
| Newtonsoft.Json | >= 12.0.0 | 12.0.1 | No |
| Serilog | >= 2.10.0 | 2.10.0 | No |

With `strategy: max` (default) these same constraints would resolve to the
latest available versions (8.x, 13.x, and 4.x respectively as of generation
date). The minimum-version lockfile is the distinguishing signal of this probe.

### Transitive dependencies (resolved by lockfile)

| Package | Resolved version | Required by |
|---|---|---|
| Microsoft.Extensions.DependencyInjection.Abstractions | 6.0.0 | Microsoft.Extensions.Logging, Microsoft.Extensions.Options |
| Microsoft.Extensions.Logging.Abstractions | 6.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Options | 6.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Primitives | 6.0.0 | Microsoft.Extensions.Options |

### Detection expectations

- Mend must detect **7 unique packages** in total from `paket.lock`.
- `Microsoft.Extensions.Logging` must be reported as **6.0.0**, not any higher version.
- `Newtonsoft.Json` must be reported as **12.0.1**, not 13.x.
- `Serilog` must be reported as **2.10.0**, not 3.x or 4.x.
- All transitives must also carry their 6.0.0 resolved versions.
- Any version mismatch indicates Mend is re-resolving instead of reading `paket.lock`.
- `storage: none` must not affect detection — Mend reads the lockfile, not the
  local packages cache.

## File structure

```
paket-strategy-min/
├── paket.dependencies       # strategy: min, 3 direct deps with >= lower-bound constraints
├── paket.lock               # Lockfile with minimum resolved versions (7 packages)
├── expected-tree.json       # Expected Mend dependency tree (probe format)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # 3 direct package references
```
