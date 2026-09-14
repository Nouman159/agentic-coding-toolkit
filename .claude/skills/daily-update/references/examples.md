# Style Samples (voice reference)

Read all three before writing. Match sentence rhythm, punctuation, and tone — bold lead phrase, colon, then a short before/after explanation. Do not copy their content, only their voice and structure. Each sample below is a generic, fictional example from a different kind of product — they exist to show the *pattern*, not to be reused verbatim.

---

## Style A — Concise, many small fixes scattered across areas

Use when the day was a lot of small independent fixes/tweaks spread across different areas (e.g. Web + Mobile, or multiple unrelated modules). Group under platform/area headings (only add headings if there's more than one area). Each bullet: **bold short lead phrase** + colon + one clear sentence describing what was broken and what now works, or what changed.

```
Today's Update
Hi! Here's what we worked on today across web and mobile

Web

Cart totals fixed: The order summary was excluding active discount codes from the displayed total; it now recalculates and shows the correct amount.
Wishlist sync fixed: Items added to the wishlist on one device weren't appearing on another; wishlist state now syncs across sessions.
Checkout address step: Fixed a validation bug that blocked checkout when a state/province field was left in its default value.
Search filters: Category filters on the search results page now persist when navigating back from a product page.

Mobile

Reduced the tap target size mismatch on the "Add to Cart" button for a more consistent feel.
Order tracking fixed: Tapping a past order no longer opens a blank tracking screen; it now loads the correct shipment status.
Pull-to-refresh fixed everywhere: Pull-to-refresh wasn't working on several tabs; it now reliably reloads data across the app.
Fixed the icon alignment on the Notifications tab.
Fixed the bottom padding on the Payment Methods screen.
The address edit sheet is now properly centered, resolving the earlier layout shift.
Consistent empty states: All list screens now share the same empty-state illustration and copy; previously a few screens looked inconsistent.
```

---

## Style B — Slightly more detail, grouped by module/feature (~1.2x the length of Style A per line)

Use when the day's work clusters cleanly into a handful of modules/features, each with a few related changes, but nothing rises to "whole feature shipped" scale. One line per module: **Module Name:** followed by a comma-separated list of what changed in that module, each item still clear enough to stand alone.

```
Task Board: Improved drag-and-drop reordering, added due-date badges, fixed column width on narrow screens, and cleaned up the card hover state
Notifications: Fixed duplicate alerts on assignment changes and corrected the unread-count badge
Project Settings: Corrected member-role labels, permission toggle behavior, and access-list sorting
Billing: Added a downloadable invoice PDF for past charges
Integrations: Updated the naming convention for connected-workspace entries
Team Dashboard: Removed the legacy activity feed and added a "Recent Projects" shortcut next to "Create New Project"
```

---

## Style C — Major feature/milestone completion, narrative roman-numeral list

Use when the day (or the period being summarized) completed a large feature, a migration, or a big chunk of new system — work with real scale (many files, many lines, multiple subsystems). Numbered with lowercase roman numerals (i, ii, iii, ...). Each point is a full sentence or two, narrative and descriptive, naming the actual subsystems/components built and citing scale (line counts, file counts, component counts) when the diff stats support it.

```
i. Migrated the legacy invoicing service from a standalone deployment into
the main product, fully integrating it as a native module accessible from
the primary navigation sidebar

ii. Scaffolded the complete billing architecture with 11 page components, 6
component subdirectories, and dedicated context/hooks/utilities — totaling
14,000+ lines across 42 files

iii. Developed a comprehensive plan editor (3,900+ lines) with tiered
pricing rules, a proration calculator, discount-code validation, and a
live invoice preview

iv. Built a customer-facing billing portal with payment-method management,
invoice history, a dispute flow (open/resolved), and downloadable receipts

v. Implemented usage metering with a background aggregation job and a
webhook-based sync to the payments provider

vi. Created a reconciliation dashboard with a transaction ledger, filter
tabs, and CSV export functionality

vii. Built the refund-review system with an approval queue, audit trail,
and reviewer-comment thread

viii. Developed an internal support-tools panel for manually adjusting
subscription state during escalations

ix. Implemented the dunning workflow for failed-payment retries and
customer notification scheduling

x. Created a subscription-health page with churn-risk indicators, stat
cards, and cohort sorting

xi. Built a plan-comparison page with feature-matrix selection and
upgrade/downgrade flows

xii. Established backend database infrastructure with 5 migrations
(subscription tiers, invoices, usage records, payment-method metadata, an
invoice-numbering race-condition fix)

xiii. Integrated backend API routes and storage-layer methods for billing
data persistence, replacing the earlier mock data

xiv. Added shared UI components including a payment-status badge, a
signature-free e-sign stub, a rich text editor for invoice notes,
breadcrumb navigation, and empty-state illustrations

xv. Standardized the billing module's visual design with a consistent
header component and alignment with the broader product design system
```
