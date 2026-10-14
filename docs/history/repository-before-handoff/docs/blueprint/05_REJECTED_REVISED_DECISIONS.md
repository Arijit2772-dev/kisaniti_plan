# Revised / Rejected Decision Register

This register indexes the important revised, refined, and corrected positions in §67 of the [consolidated architecture handoff](../architecture/KisanNiti_Consolidated_Architecture_Handoff_2026-08-10.md). It preserves the previous idea so it cannot be silently revived. The architecture remains authoritative.

## RR-001 — Full allocation assumption (`REVISED`)

**Previous idea**<br>
“Selected quantity must always be fully allocated.”

**→ Current architecture position**

- Both full and partial allocation policies exist.
- Partial remainder remains explicit inventory.

## RR-002 — Cancellation only before execution (`REVISED`)

**Previous idea**<br>
“Approved plan can only be cancelled before execution.”

**→ Current architecture position**

- Only remaining unexecuted commitment is released.
- Executed ActualSales remain reality.

## RR-003 — ActualSale ownership (`REFINED`)

**Previous idea**<br>
“ActualSale belongs to SellingAssignment.”

**→ Current architecture position**

- ActualSale belongs fundamentally to ProduceLot.
- SellingAssignment reference is optional.

## RR-004 — DHT22/spoilage claim (`REFINED`)

**Previous idea**<br>
“Rule-based spoilage prediction from DHT22.”

**→ Current architecture position**

- Use a conservative environmental StorageAssessment.
- Make no unsupported direct spoilage claim.

## RR-005 — Routing-provider coupling (`REFINED`)

**Previous idea**<br>
“OSRM is routing.”

**→ Current architecture position**

- RoutingService is provider-independent.
- The provider is replaceable.

## RR-006 — Market arrival as physical capacity (`REFINED`)

**Previous idea**<br>
“Historical arrival number is mandi physical capacity.”

**→ Current architecture position**

- It is an empirical, versioned MarketLoadPolicy/reference.
- `STRICT`, `ADVISORY`, and `DISABLED` semantics are configurable.

## RR-007 — PlanRevision cancellation semantics (`REFINED`)

**Previous idea**<br>
“PlanRevision `APPROVED → CANCELLED` is the universal cancellation model.”

**→ Current architecture position**

- Approval remains a historical fact.
- Post-approval cancellation/release is quantity-aware operational commitment state.
- Aggregate cancellation may be derived.

## RR-008 — Effective optimizer price (`REFINED`)

**Previous idea**<br>
“`p[m,d]` is permanently sufficient.”

**→ Current architecture position**

- Domain PriceForecast remains mandi/commodity/date.
- The optimizer accepts future-ready effective `p[l,m,d]`.
- V1 can map it directly to the mandi/day forecast.

## RR-009 — Production solver selection (`CORRECTED`)

**Previous idea**<br>
“MILP is already the frozen production solver.”

**→ Current architecture position**

- MILP is the leading formulation to prototype.
- The production solver must pass benchmark gates.

## Governance rule

Any proposal that resembles a previous idea above must explicitly cite this register and the current architecture position. A later approved change must preserve the old and new positions, supporting evidence, and supersession history; never overwrite this record.
