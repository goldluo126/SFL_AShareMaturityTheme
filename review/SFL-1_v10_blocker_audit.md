# SFL-1 v10 blocker audit (vs v9-review A/B list)

> Non-normative. Canonical rules: `SFL-1_v10.md`.
> Verdicts: CLOSED / STILL OPEN / PARTIAL.

## Blockers

| ID | Verdict | One-line evidence |
|---|---|---|
| A-01 | **CLOSED** | §3.5.1 `FrozenMetricBackcast` at \(\tau_{confirm}\): backcast GCCSlope/RMI/IDS history on frozen cohort/null; bans temp+frozen splice |
| A-02 | **CLOSED** | §18 Gate7: match cutoff ≤S−1; A/B/C/D signals = S; exec = S+1; `NoCapacityStartThroughS` = never entered capacity thru S |
| A-03 | **PARTIAL** | §19 `P10_NULL_EPISODE` + `AnnualFalseEpisodes_y` unique-episode set; but same § still says “按上式对有效日求和” (day-sum leftover) |
| A-04 | **CLOSED** | §3.3/§5.4: ledger merge OK; forbid union B/L/K + single null; origins keep EPISODE_NULL/state; exits by `origin_episode_id` |
| A-05 | **CLOSED** | §15.0 over-cap: ban new buys / higher reserve; no forced delever; event `CLUSTER_OVER_CAP_NO_NEW_BUY` |
| A-06 | **CLOSED** | §18 Gate6 `R^{arm}_{net}`: §14.1 buy + 20d hold + §14.3 sell/T+1 + §14.4 costs; gross = `DIAGNOSTIC_GROSS_ARM` only |
| A-07 | **CLOSED** | §20.2.4: bootstrap draws ID1 persistent component as whole block; forbids per-episode redraw |
| A-08/A-09 | **CLOSED** | §14.1 RMB mother order: `ParentOrderTargetValue` / `RemainingTargetValue`; cross-day inherits CNY, not frozen qty |
| A-10 | **CLOSED** | §14.3 `SellableQty_t = SettledShares_{t-1} − PendingSellReserved_t`; unsettled buys settle next open |
| A-11 | **CLOSED** | §15.5 PIT corporate-action ledger + §4 DG16 → `LEDGER_INVALID` |
| A-12 | **CLOSED** | §0.3.2 control basket: same D+1/D+2/D+3 attempt days, RemainingTargetValue inheritance, no solo fills on non-attempt days |
| A-13 | **CLOSED** | §18 Gate8: per-signal paired `Outcome_no_gap − Outcome_actual`; bans rejected∩later-filled double-count |
| A-14 | **CLOSED** | §18 Gate9 tape fields include `parent_order_id`; `InitialCapital` = Σ once per parent `original_target_value` |
| A-15 | **CLOSED** | §20.5 items 2–8 formulas; §20.5/§22: Gate9 PASS + dynamic (2–8) FAIL → `PASS_ENTRY_ONLY` (`DYNAMIC_GUARDRAIL_FAIL`) |
| B-04 | **CLOSED** | §2 H-CLOCK + §17.4: IUT = \(\beta_3\) level interaction; `GCCSlope>0` = Gate5 guardrail only |

## NEW leftovers / contradictions

- **P10 annual text:** unique-episode cardinality formula vs leftover “对有效日求和” / day-coverage language in same paragraph (§19).
- **Field rename drift:** §14.3/§15.3 `UnsettledBuyShares` vs §15.5 `UnsettledShares`.
- **Gate6 title leftover:** heading still “机制口径假想账本（v9）” while formal estimand is net `R^{arm}_{net}`.
- **Temporary Cohort / InitialCapital=Σtape rows / rejected vs filled groups:** no active rules remain (only explicit bans).
