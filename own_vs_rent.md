# Own vs. Rent: GitHub issue triage in n8n and Zapier

**Scenario and assumptions.** A small product team gets 250 newly opened issues a month. I assume at most one route-log failure a month and one workflow change a quarter. These are assumptions, not measurements.

**What I tested.** The n8n side is built and tested end to end (see `test_evidence.md`; seven executions took 13 to 164 ms each). For Zapier I did not complete the GitHub trigger plus Formatter test, so the Zapier column rests on my Week 6 Zapier build (an RSS trigger, a Filter and a Tables step on a 14-day trial) and on Zapier's pricing page as I read it on 1 October 2026. Plan limits can change.

| Dimension | Self-hosted n8n (Own) | Zapier (Rent) | Your evidence or assumption |
| --- | --- | --- | --- |
| Initial setup time | Docker stack, two Data Tables, 17 nodes and the forward command | A two-step Zap is expected to be quick | I did not time either; Zapier GitHub trigger not tested |
| Public webhook and credential setup | No credential; GitHub CLI forwards events to localhost, which only works while my machine and the command run | Needs a GitHub connection through Zapier | Zapier connection not tested; localhost setup is for testing only |
| Ability to add branching | Switch node with four outputs, built and run | Branching needs more steps; the Free plan allowed two-step Zaps | Week 6 pricing page read, 1 Oct 2026 |
| Duplicate prevention and ledger | `issue_key` ledger with If row exists / does not exist; replay test passed | Tables Find Records with create if not found finds a row but does not update it | Zapier behavior from my Week 6 Zap, not from issues |
| Failure visibility and recovery | Failure path writes `failed` and a safe note; manual repair steps in `runbook.md` | Zap history shows failed runs (assumed) | Zapier failure handling not tested |
| Monthly volume assumption | 250 issues a month | 250 issues a month | About 3 action steps per issue gives about 750 tasks |
| Direct platform cost | No license fee for the self-hosted Community Edition per n8n's docs; my machine and time | Free plan allows 100 tasks a month, so a paid plan is needed | I did not check paid prices, so no number is given |
| Maintenance, security and uptime | Mine: updates, backups, keeping the machine and forward running | The vendor runs it | Localhost setup has no uptime at all when my laptop is off |
| Data location and access | Data Tables live in my Docker volume | Tables data is stored on Zapier's servers | From the Week 6 build |

**Setup friction I met.** On the n8n side the friction was in the plumbing around the tool, not the tool itself. My GitHub CLI was logged in to four accounts, so the forward would have targeted the wrong one until I switched to the repository owner. GitHub's Redeliver button returned 401 through the forwarder, so the duplicate test needed a replay of the same payload fields instead. The whole setup also depends on my machine being on, which is fine for a sprint and unacceptable for a team. On the Zapier side the friction I recorded in Week 6 was the plan ceiling: a Filter step made my Zap three steps, and the Free plan allowed only two. Both tools asked me to design duplicate handling, but only n8n made me own it completely.

**Conclusion.** Under these assumptions I would pick n8n for a small product team that has one engineer willing to own hosting. At 250 issues a month the Free Zapier plan runs out of tasks quickly, and the ledger, duplicate guard and visible failure state are things n8n let me build and test directly. The prototype is not production ready: it needs a real public webhook URL, hosting with backups, and a unique constraint on `issue_key`. If nobody on the team wants to run a server, a paid Zapier plan is the better fit, because the vendor carries uptime and security and the team pays with money instead of time. The deciding question is who owns the maintenance, not the feature list.
