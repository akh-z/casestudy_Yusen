# Shipment Tracking — trade-offs, assumptions & data-quality handling

**Approach.** The notebook is a medallion pipeline — Bronze → Silver → Gold — with a
`verify_data_quality` gate after each layer that stops the run if a hard check fails, so bad
data never reaches the next stage. Warehouse arrives as a batch CSV read; carrier comes in
through Auto Loader as a stream. The two are unioned into one event stream (`silver_events`)
rather than joined on tracking number, and Gold is one row per shipment.

**Assumptions.** A shipment is identified by `(carrier, tracking_number)`, not the tracking
number alone, because a tracking string is only unique within a carrier (the data reuses the
same string across DHL/FedEx/UPS). Warehouse timestamps are treated as local site time and
converted to UTC via the `warehouse_tz` seed (country as fallback); carrier times already
carry their offset. A terminal status — delivered, cancelled, returned — is sticky and isn't
undone by a later in-transit scan, and on ties the carrier feed is trusted over the warehouse
for the physical leg. Origin is taken from the shipment as a whole, since carrier events carry
no origin. Sources are assumed append-only, with no hard-delete signal.

**Trade-offs.** Gold is a full recompute rather than an incremental merge, with MERGE as the documented next step. The
`(carrier, tracking_number)` grain correctly splits colliding tracking numbers but would
over-split a genuinely rebooked shipment, so that's the one rule to confirm with the business.
Throughout, the choice is to quarantine and flag rather than halt: bad rows go to `*_rejected`
tables and the run continues, and the pipeline only hard-stops when something fundamental
breaks (no valid data, or the grain isn't unique). The timezone seed is simple but fragile for
a new, unseeded country (it falls back to UTC).

**Data quality handling.** Bronze reads with an explicit schema in PERMISSIVE mode, so a
corrupt or wrong file lands its bad rows in `bronze_warehouse_rejected` instead of crashing,
and Auto Loader's schema rescue captures unexpected fields rather than dropping them. Silver
does the cleaning — dedup, carrier and tracking normalised (upper-cased so casing can't split
a shipment), statuses mapped to one vocabulary, local times to UTC — and quarantines anything
with an unmapped status or unparseable timestamp. Gold resolves the current status with the
terminal-sticky, latest-event logic and attaches honest flags — `ambiguous_tracking`,
`geo_mismatch`, `has_status_conflict`, `has_post_terminal_event`, `has_clock_anomaly`,
`origin_known` — nulling fields it can't trust instead of guessing. A closing DQ summary
reports everything quarantined by layer and reason, so a clean run shows zeros.
