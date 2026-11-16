# OpenCode Resources

This repository contains agents and skills for use with OpenCode. It contains no application, .NET manifest, or test suite.

## Contents

- **Agents** in [`.opencode/agents/`](.opencode/agents/):
  - [`coordinateur`](.opencode/agents/coordinateur.md), the primary agent (`primary`) and entry point that plans and coordinates specialists. It uses GitHub Copilot models, which must be available and configured in OpenCode.
  - [`architecte`](.opencode/agents/architecte.md), the architect, designs solutions and analyzes their contracts and risks;
  - [`developpeur`](.opencode/agents/developpeur.md), the developer, implements assigned changes;
  - [`testeur`](.opencode/agents/testeur.md), the tester, defines and writes tests;
  - [`reviewer`](.opencode/agents/reviewer.md), the reviewer, performs independent, read-only reviews;
  - [`documentaliste`](.opencode/agents/documentaliste.md), the documentarian, maintains documentation.

  The latter five are subagents (`subagent`). The agent files retain their V1 `permission` frontmatter, which is now deprecated in the OpenCode V2 documentation; they have not been modernized.
- **Skills** in [`.opencode/skills/`](.opencode/skills/): [`ub-mode`](.opencode/skills/ub-mode/SKILL.md), a step-by-step workflow, and [`solid`](.opencode/skills/solid/SKILL.md), software engineering principles and references. The `solid` references are in [`.opencode/skills/solid/references/`](.opencode/skills/solid/references/).

## Using the resources with OpenCode

OpenCode discovers project agents in `.opencode/agents/<id>.md` and project skills in `.opencode/skills/<id>/SKILL.md` ([agent documentation](https://opencode.ai/docs/agents/), [skill documentation](https://opencode.ai/docs/skills/)). Open this repository in OpenCode to use these resources.

To use them in another project, copy the `.opencode/` directory there. If files with the same names already exist, inspect them and merge the configurations rather than blindly overwriting them. Optional global installation is possible in `~/.config/opencode/agents/` and `~/.config/opencode/skills/`.

Explicitly invoke a skill in a request, for example `@ub-mode` or `@solid`. The presence of the `ub-mode` skill does not by itself create a `/ub-mode` command.

## `ub-mode` .NET tools

Installing the OpenCode resources above is separate from installing .NET CLI tools. Install .NET tools in the environment or .NET project that uses them, not in this repository. Check the available SDKs and runtimes with `dotnet --info`. Global tools also require the .NET global tools directory to be on `PATH` ([Microsoft documentation](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools)). OpenCode does not install these tools automatically.

### Global installation

After installation, run the tools from the relevant .NET project directory:

```sh
dotnet tool install --global Crap4DotNet
dotnet-crap analyze ./MyProject.sln --run-tests --threshold 10 --output crap-report.json

dotnet tool install --global dotnet-stryker
# From the test project directory:
dotnet stryker
```

- **CRAP / Crap4DotNet** requires .NET 8 or later. `ub-mode` calls for a CRAP score below 10, while Crap4DotNet defaults to 30: `--threshold 10` brings the analysis closer to that goal, but does not strictly enforce it. The tool reports only scores greater than the threshold, so a score exactly equal to 10 is still accepted. To enforce the `< 10` goal strictly, inspect the report (for example, with `--output crap-report.json`) and treat every score `≥ 10` as needing correction. The skill's standard invocation does not pass this threshold. The `--run-tests` option relies on `coverlet.collector` and VSTest. Microsoft.Testing.Platform (MTP) has an incompatible case: consult the tool documentation to configure coverage manually. [Crap4DotNet project](https://github.com/7Factor/crap4dotnet)
- **Stryker.NET** requires a .NET 10 or later runtime for the tool; the application's target framework (TFM) can be older. Run `dotnet stryker` from the test project. [Stryker.NET guide](https://stryker-mutator.io/docs/stryker-net/getting-started/)

### Project-pinned versions

To pin the versions used by a team, create a local manifest in the consuming .NET project, once:

```sh
dotnet new tool-manifest
dotnet tool install Crap4DotNet
dotnet tool install dotnet-stryker
```

Other collaborators can then restore the tools from the manifest with `dotnet tool restore`. Do not create this manifest in this resource repository. Local commands can be run with `dotnet tool run`:

```sh
dotnet tool run dotnet-crap -- analyze ./MyProject.sln --run-tests --threshold 10 --output crap-report.json
dotnet tool run dotnet-stryker
```

### DupFinder and precautions

The `ub-mode` skill mentions `jb dupfinder .`, but JetBrains removed DupFinder from the Command Line Tools starting with version 2021.3; version 2021.2 was the last to include it. Modern JetBrains tools therefore do not provide this command. This skill line is historical and is not guaranteed; no supported modern installation procedure is recommended here. [JetBrains announcement](https://blog.jetbrains.com/dotnet/2021/08/12/sunsetting-dupfinder-command-line-tool/)

Analysis, tests, and mutations can take a long time and execute project code. .NET tools run with full trust in the process: verify their provenance before installation and run them only on trusted projects.
