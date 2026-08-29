# Product backlog: catalogue quality and prioritisation by business impact

The mission: qualify the anomalies in the product catalogue, measure their business impact, and prioritise the corrections accordingly. The notebook (`notebooks/06_catalog_quality.ipynb`) is the deliverable that answers this backlog.

## Product context

The product catalogue contains known anomalies: non-product codes mixed in with the articles, inconsistent labels, aberrant prices. Before correcting anything, we need to know what it costs to leave them uncorrected, otherwise prioritisation is arbitrary.

## Epics

### Epic 1: make the product catalogue reliable (master data)

| # | User story | Acceptance criteria | Priority |
|---|---|---|---|
| 1.1 | As a Data Product Owner, I want a documented and audited product catalogue so that the sales teams do not work on duplicated or inconsistent product entries | Every anomaly in `DATA_CATALOG.md` is quantified in lines affected, and the catalogue is updated as soon as a new anomaly is found | Must |
| 1.2 | As a Data Product Owner, I want non-product codes (fees, adjustments, tests) isolated from the product catalogue so that product performance KPIs are not polluted | The excluded codes are documented as an explicit list, not as an implicit filter buried in the code | Must |

### Epic 2: prioritise corrections by business impact and conversion

| # | User story | Acceptance criteria | KPI | Priority |
|---|---|---|---|---|
| 2.1 | As a Proxy Product Owner, I want to know the revenue exposed by catalogue anomalies so that corrections are prioritised by impact rather than at random | The revenue tied to products in anomaly (Q2, Q3) is computed and compared to total revenue | Exposed revenue / total revenue | Must |
| 2.2 | As a Proxy Product Owner, I want to track the cancellation rate per product so that I can tell which anomalies actually affect conversion, not just the data | Products are ranked by cancellation rate, with the order count shown alongside | Cancellation rate by `StockCode` | Should |
| 2.3 | As a Proxy Product Owner, I want a ranking of products by performance (best and worst sellers) so that correction effort goes to the products that matter most | The ranking crosses business performance with the presence of a catalogue anomaly | Top and bottom products by revenue, crossed with quality status | Should |

## Simplified roadmap

- **Sprint 1**: Epic 1, making the catalogue reliable. This is a blocking condition.
- **Sprint 2**: Epic 2, pricing the impact and prioritising. This is where the value of the backlog sits.

## Definition of done

A user story is done when the anomaly is quantified in both line count and business impact in revenue, not merely listed.
