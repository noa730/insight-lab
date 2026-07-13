# Insight Lab / 洞察实验室

AI-powered research & presentation plugin for [Claude Code](https://claude.com/claude-code).

## Skills

### /research "topic"

Launch a full research project. Produces data-backed reports with visualizations.

- 5 output types: research report, feasibility study, proposal, decision memo, project plan
- 3 depth levels: quick (3-5 questions), standard (5-8), deep (8-12)
- Dependency chain: research → feasibility → proposal → decision → plan
- MECE issue decomposition, parallel data collection, cross-validation, auto-charting
- Human-in-the-loop checkpoints at every phase

| Output Type | Description | Output File |
|-------------|-------------|-------------|
| research | Industry research report | `report.md` |
| feasibility | Feasibility study | `feasibility-report.md` |
| proposal | Project proposal | `proposal.md` |
| decision | Decision memo | `decision.md` |
| plan | Project plan | `project-plan.md` |

### /present "topic"

Create interactive HTML presentations from research outputs or raw materials.

- 4-phase workflow: Outline → Design → Chart → Build
- Multi-file HTML with keyboard navigation, drill-down pages, presenter notes
- Auto style recommendation based on audience type
- Chart generation via infographic/vega-lite/ECharts skills
- 1920x1080 adaptive scaling for projectors

## Install

Add to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "insight-lab": {
      "source": {
        "source": "github",
        "repo": "noagao/insight-lab"
      }
    }
  },
  "enabledPlugins": {
    "insight-lab@insight-lab": true
  }
}
```

Then restart Claude Code.

## Usage

```
/research "EDA industry analysis"
/research resume

/present "quarterly business review"
/present resume
```

## Output Directories

Running `/research` or `/present` creates output in:

```
research-output/<project-slug>/    # research outputs
present-output/<project-slug>/     # presentation outputs
```

These are created in whatever directory Claude Code is running in.

## License

MIT
