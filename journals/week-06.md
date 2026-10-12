# Weekly Build Activity Journal

_Figures 1 and 2 mentioned below are in the Brightspace copy of this journal._

## CS 590 Low Code No Code for Rapid Prototyping

**Student Name:** Joel Adu-Kwarteng  
**Week:** 6  
**Date Range:** 09/28/2026 to 10/04/2026  
**List Course Work and/or Project:** Week 6 In-Class Activity: Tech Signal news aggregator in n8n, rebuilt in Zapier for the Own vs. Rent report. Sprint 2: GitHub Issue Triage Automation in n8n, Parts 1 and 2 (Part 3 not started).

Use this journal to give an accurate weekly account of your work. Specific evidence and an honest diagnosis matter more than a smooth success story. A failed build, a blocked stack, or unfinished work can earn full credit when you show what happened, what you tried, and what you will do next.

## 1. Weekly Focus and Goals [1 point]

State the current sprint, lab, or project objective. Then list up to three specific goals you planned to complete this week.

**Objective:** Build the Tech Signal pipeline (trigger, retrieve, filter, transform) in self-hosted n8n, rebuild the same idea in free Zapier to collect first-hand evidence for the Own vs. Rent report, and start Sprint 2 by preparing the public repository, the n8n Data Tables and the webhook intake for the GitHub issue triage workflow.

  - **Goal 1:** Build Tech Signal in n8n: Manual Trigger, RSS Read on the TechCrunch AI feed, Limit, Filter, Edit Fields and a Data Table upsert, then run it twice to check for duplicates.
  - **Goal 2:** Rebuild the same project in Zapier (Part 9): RSS trigger, topic filter, and a duplicate-safe save to a Zapier Table, with notes on the differences.
  - **Goal 3:** Sprint 2 Parts 1 and 2: create the public repo n8n-sprint2-issue-triage, the workflow shell, both Data Tables, and the webhook intake with the opened-action gate and payload normalization.

## 2. Completed Work and Evidence [3 points]

For each completed item, state what you changed or built and attach or link the evidence. Evidence may be a screenshot, repository commit, Figma file, Baserow base, n8n workflow export, app link, test output, or another inspectable artifact.

| Completed work                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Evidence                                                                                                                                                                             | Why it matters to this week’s objective                                                                                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tech Signal in n8n, complete. Manual Trigger, RSS Read (TechCrunch AI feed), Limit (10), Filter (OR conditions, Ignore Case), Edit Fields and a Data Table Upsert on item_url. Edit Fields maps the publisher fields to my own names (item_url, title, published_at, source, topic, summary, collected_at). Two full runs on 10/04: 20 items read, 10 kept by Limit, 8 kept by Filter, 8 upserted; the second run updated the same rows and the table stayed at 17 rows. | Figure 1 (successful execution), Figure 2 (tech_signal_items table), and news_aggregator.json in the submission folder.                                                           | This is the trigger, retrieve, filter, transform part of the loop. Mapping feed fields to my own column names is what makes the later Data Table step possible.            |
| Same project built in Zapier (Part 9). Zap draft: RSS by Zapier (New Item in Feed, same TechCrunch URL), Filter by Zapier (Title contains AI OR Description contains AI), Zapier Tables Find Records on table tech_signal_items (item_url exactly equals Link, create if not found, seven columns mapped). First test created 1 row; a retest returned the same record ID with Zap Data Was Found = true and no new row. The Zap is an unpublished draft.                 | Zapier Test step Data in and Data out for each step; table tech_signal_items; comparison notes in Week06_Part9_n8n_vs_Zapier_notes.docx (course folder).                      | Direct evidence for the Own vs. Rent report: the same task built twice, with the step count, plan limits, timestamp handling and duplicate handling compared side by side. |
| Sprint 2 Parts 1 and 2. Public repo JoelisKing/n8n-sprint2-issue-triage (Issues tab on). In n8n: workflow "Sprint 2 - GitHub Issue Triage" with a POST Webhook at sprint2-issue-triage, an IF "Opened action?" gate (true output only), and an Edit Fields "Normalize issue" node. Data Tables github_issue_ledger (10 columns) and github_issue_route_log (4 columns) created with the exact column names.                                                             | https://github.com/JoelisKing/n8n-sprint2-issue-triage (public). n8n workflow and both Data Tables on my local instance; the workflow is inactive and nothing has been executed yet. | The stable issue_key (repository + "\#" + issue number) and the two tables are the foundation for the duplicate guard and the route log in Part 3.                        |

**Data and security:** The feed is a public TechCrunch RSS feed and the repository holds only course test content, so there is no personal or client data. The Sprint 2 repo is public: it has no tokens, no .env file and no webhook secret. Zapier was used on a 14-day Professional trial, which is a cost and privacy implication: a Zap on the free plan could not include the Filter step, and Zapier Tables stores the feed rows on Zapier's servers, not mine.

## 3. Work in Progress and Next Step [1 point]

Name the work that is not complete yet. For each item, state the smallest next action you will take.

| Work in progress                                                                              | Current state                                                                                                                       | Next concrete action                                                                                                                                              |
| --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tech Signal submission (in-class activity, due 10/04).                                        | Workflow run and exported (news_aggregator.json), both screenshots taken, README.md with the reflection written. Not yet uploaded. | Upload the submission folder to Brightspace.                                                                                                                      |
| Sprint 2 Part 3 (Steps 8 to 10): classification, duplicate guard, route log and failure path. | Not started. Parts 1 and 2 are done. The repository has no README yet, and a stray empty Data Table named "j" exists in n8n.        | Build the Classify title Switch with the four route-setting nodes, then the two Data Table existence checks. Delete the stray table and add the README in Part 5. |

## 4. What You Learned [2 points]

Describe the result of this week’s work. Include one decision, observation, or result that changed your understanding of the problem, user flow, data model, workflow, or platform.

Then name the prototype dimension you tested most directly this week, **role**, **look and feel**, **implementation**, **data shape**, or **process**, and one important question your work did not answer.

The result is a working Tech Signal pipeline in n8n (through the Data Table upsert), the same pipeline as a three-step Zap with a table write that finds or creates rows, and a Sprint 2 foundation (public repo, two Data Tables, webhook intake with an opened-only gate). Extensibility note for Sprint 2: because issue_key combines the repository name and the issue number, the same ledger could take issues from several repositories without a schema change.

The observation that changed my understanding was how differently the two tools handle duplicates. n8n makes me design idempotency myself: an Upsert on item_url updates a matching row, and my second full run on 10/04 updated the same rows without adding any (the table stayed at 17 rows). Zapier gives some of it for free at the trigger (the "What Triggers a New Feed Item?" setting defaults to Different Guid/URL), but its Tables action had no upsert, so I used Find Records with create if not found. That finds the row without updating it, so an edited article would never refresh its row. The second observation is about cost: the Filter made my Zap three steps, and the Free plan allows two-step Zaps only, so it built only because of the trial. A simple "filter then save" flow already crosses the free line.

**Extensibility observation.** The next capability I would add without replacing the core design is a human approval step for bug issues: a Wait node after Set bug route that holds the record until I approve it. The tutorial's design makes this easier in three ways. Classify title already hands every item a stable `category`, and each route has its own Set node, so a new step attaches to one branch without touching the others. `issue_key` is a stable identifier to look the record up and resume against. And `route_log_status` is a text column, so a new value such as `awaiting_approval` needs no schema change. Two things make it harder. The Insert pending ledger row node maps `route_log_status` to the fixed text `pending`, so the approval state would need a new write, and the later update nodes in the design refer to earlier nodes by name, so moving or renaming nodes could break them.

**Prototype dimension tested:** implementation (how the same flow is built in an owned tool and in a rented one).

**Question not answered:** I did not publish the Zap or watch a scheduled run, so I have no real numbers for task usage or polling delay (the Free plan polls every 15 minutes and allows 100 tasks a month). I also did not test whether the Zapier Contains filter is case sensitive.


## 5. Challenge and Diagnosis [2 points]

Describe one obstacle honestly and specifically.

  - **What happened?** Include the exact error, unexpected behavior, unclear requirement, or missed plan.

> In the Zapier Filter step, a single rule "Title contains AI" did not pass the sample record from the TechCrunch AI feed, so the Zap could not continue to the Tables step.

  - **What did you try or rule out?**

> I added a second rule joined with OR (Description contains AI), and the sample record then passed. I have not confirmed why the title rule failed: the sample headline may simply not contain "AI", or the match may be case sensitive. I did not test either explanation, and I did not see an Ignore Case option in the Zapier filter, so I cannot yet say which it was.

  - **What will you try next?**

> Test the filter with two controlled values, a title containing "AI" and one containing "ai", to find out whether the match is case sensitive, and write the result into the Own vs. Rent report as a difference from n8n's Filter node, which has an Ignore Case option.

## 6. Response Plan [1 point]

State what you anticipate to address next week. If you need instructor help, please send an email!

Next week I will build Sprint 2 Part 3 (Switch classification, duplicate guard, route log and failure path) and activate the webhook with gh webhook forward for the Part 4 tests (Sprint 2 is due 10/11). I will also start drafting the Own vs. Rent report (due 10/18) from the Week 5 and Week 6 notes.

**Question for the instructor:** For Part 9, is a three-step Zap on the 14-day Professional trial acceptable for the comparison, or should the Own vs. Rent evidence reflect only what the strict Free plan (two-step Zaps) allows?


## Course Evidence Check

  - I included evidence that a reviewer can inspect.
  - I used only synthetic or course-approved data and did not include secrets, tokens, or personal information.
  - I named a real outcome and a real challenge rather than writing a generic status update.
  - If this work has a cost, security, privacy, or extensibility implication, I named it in the relevant section above.
