# Unreal Game Sync (UGS)

## Overview

Unreal Game Sync (UGS) is Epic's official tool for synchronizing a team's Unreal Engine project from Perforce (P4). It extends basic P4 sync with pre-built editor binaries, automated CI status display, build scheduling, and team annotations—eliminating the need for every team member to compile the engine from source.

## Key Features

- **One-click sync and build** — fetches source and pre-compiled editor binaries from a P4 depot.
- **CI Status Display** — shows build/test status for each CL (changelist) so teams know which CLs are "green".
- **Badges & Annotations** — team members can mark CLs with custom badges (e.g., "QA Passed").
- **Build scheduling** — automatically trigger local or CI builds after sync.
- **Filter profiles** — control which files/paths are synced per user type (artist vs. programmer).
- **Selective workspace** — sync only the paths relevant to your discipline.

## Installation

1. Download UGS from the [Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-game-sync-ugs-for-unreal-engine) or build from source in the `Engine/Source/Programs/UnrealGameSync` directory.
2. Install and configure the Perforce P4V client.
3. Launch UGS and point it to your `.uproject` file path in the P4 depot.

## Setup

### Configuring the P4 Connection

1. Open UGS.
2. Go to **File > Open Project**.
3. Enter your **P4 Server**, **Username**, and **Workspace** (client spec).
4. Browse to the `.uproject` file path in the depot.
5. Click OK — UGS will display the available CLs.

### Setting Up a UGS Metadata Server (Optional)

The metadata server stores CI build results and badge data, making it visible to all team members in UGS.

- Requires a small .NET web service (`UnrealGameSync.Metadata.Server.dll`).
- Store the server URL in `Build/UnrealGameSync.ini` in the depot:

```ini
[Default]
ApiUrl=https://your-metadata-server/api
```

### Configuring Perforce Streams

UGS works with both classic depots and Perforce Streams. Configure stream mappings in your workspace client spec.

## Common Workflows

### Daily Sync (Artist / Designer)

1. Launch UGS.
2. The CL list shows green (passed CI), yellow (building), red (failed), or grey (unverified) CLs.
3. Double-click a green CL to sync to it.
4. UGS downloads pre-built binaries (if configured) so artists skip compilation.
5. UGS launches the Unreal Editor automatically after sync completes.

### Sync with Build (Programmer)

1. Open UGS **Options > Sync > After Syncing: Build Editor**.
2. Sync to any CL; UGS will automatically compile the Editor after.
3. For incremental builds, UGS uses the existing Build configuration.

### Submitting Changes (P4V Integration)

UGS is a sync/build tool — actual P4 submits are done through P4V or the command line. UGS displays pending changes and links to P4V.

### Build CI Badges

- CI systems (Jenkins, TeamCity, Buildkite) can POST build results to the metadata server.
- Results appear as colored badges on each CL in the UGS list.
- Team members can see which CL broke the build without needing to check CI dashboards directly.

## Filter Profiles

Create named profiles to control which P4 paths are synced:

```
// Example: Artist profile skips Source/ directory
-//depot/project/Source/...

// Example: Programmer profile syncs everything
//depot/project/...
```

Configure in **Options > Sync Filters**.

## Build Configuration File

Place `Build/UnrealGameSync.ini` in the P4 depot to configure project-wide UGS settings:

```ini
[Default]
ApiUrl=https://your-ugs-metadata-server/api
ArchiveType=Editor

[Build]
Solution=MyProject.sln
Configuration=Development
Platform=Win64
```

## Git Support (UGS + Git)

UGS was originally designed for Perforce but has limited Git support via community plugins. For Git-based projects, consider using GitHub Actions or similar CI tools to display build status — UGS's Git integration is not as mature as its P4 support.

## AI / MCP Integration

UGS itself does not currently have an official MCP server, but its REST API (metadata server) can be queried programmatically:

- `GET /api/build` — list build results
- `POST /api/build` — report a build result from CI
- `GET /api/badge` — get badge annotations

An AI agent can use these endpoints to monitor CI health, trigger syncs, or annotate CLs. See [docs/mcp-servers/overview.md](../mcp-servers/overview.md).

## References

- [UGS Documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-game-sync-ugs-for-unreal-engine)
- [UGS Source Code](https://github.com/EpicGames/UnrealEngine/tree/release/Engine/Source/Programs/UnrealGameSync) (requires UE source access)
