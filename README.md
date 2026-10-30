# Tallgrass

**Your collection, region by region.**

*A BayMatt Labs project.*

Tallgrass is a personal Pokémon National Dex tracker. It helps collectors see what they own, find what is missing, and follow their progress across regions.

The experience starts with an overall completion summary and regional progress charts. Selecting a region opens its monsters in a searchable list, where users can filter by collected or missing status, update ownership, and add personal notes.

Tallgrass tracks monsters and their variants. It does not require identifying individual cards, sets, printings, or quantities.

## Project Status

Tallgrass is in early development. The database foundation and initial workbook import are complete, and desktop and mobile design concepts have been approved in both light and dark mode. The application itself has not been implemented in this repository.

### Completed Foundation

- PostgreSQL schema `pokemon_tcg` created in Neon.
- Neon Auth enabled, with collections linked to their owners.
- Shared catalog imported with 1,025 standard monsters, 58 regional variants, 96 Megas, and two Primals.
- Existing ownership imported into the initial private collection.
- Database constraints and collection access rules tested.
- Desktop and mobile mockups established using the BayMatt Labs brand guide.
- Monster artwork removed from the approved designs.

Database migrations, reusable seed data, and design assets still need to be brought into this repository. Personal account information and collection records must remain separate from reusable catalog seeds.

## Product Experience

### See Your Progress

The hero area shows overall completion alongside regional charts with explicit collected and total counts. Standard National Dex completion is the default, with separate views for regional, Mega, and Primal variants.

Selecting a region filters the list below. An All Regions view supports browsing and searching across the full catalog.

### Manage Your Collection

Each list entry shows its National Dex number, name, applicable region, and ownership status. Users can mark an entry collected or missing directly from the row. An options menu provides secondary actions such as editing notes.

Changes save automatically. Feedback, retry behavior, and undo make updates clear, including when a changed entry disappears from the current filter.

### Make It Yours

Users sign in to access their own collections. The planned experience includes collection selection, remembered browsing preferences, recently collected entries, regional milestones, and optional read-only progress sharing.

Both themes follow the BayMatt Labs design system, with responsive layouts and controls that work with touch, a keyboard, and assistive technology.

## Tracking Rules

- Each species has one standard variant. Regional, Mega, and Primal variants are separate entries.
- Collecting one variant does not automatically collect another.
- Standard National Dex completion uses the 1,025 standard catalog entries as its denominator.
- Region and variant scope determine the progress population. Collected/missing filters affect the list, not the completion calculation.
- Standard entries use their species' region. Variants may override that region.
- The current catalog preserves the workbook's 11 region labels, including Hisui and Kitakami. Any regrouping should be an explicit product decision.
- An ownership row means collected. No ownership row means unchecked.
- The workbook did not record Mega or Primal ownership; those entries were imported unchecked.
- Imported timestamps describe the import, not when a monster was originally collected.
- Personal notes and account details remain private, including on shared progress pages.

## Data Model

| Table | Responsibility |
| --- | --- |
| `pokemon_tcg.monster` | Shared species catalog with a unique National Dex number, name, and region. |
| `pokemon_tcg.monster_variant` | Standard, regional, Mega, and Primal entries belonging to a species. |
| `pokemon_tcg.collection` | Named collection with a required owner in Neon Auth. |
| `pokemon_tcg.collection_monster` | Owned variants and optional notes, unique within each collection. |

The `pokemon_tcg.collection_dex` view combines the catalog with ownership to produce a complete checklist for an accessible collection.

Inserting a species creates its standard variant automatically. Constraints protect standard variants, prevent duplicate variant codes and ownership entries, and require a region override for regional variants.

### Application Access

The existing database uses the restricted `pokemon_tcg_app` role and transaction-local `app.user_id` context for collection access.

The backend must verify the user's session, switch to that role, and bind the verified user ID within the same database transaction before executing collection queries. Neon Auth does not set this application context automatically. The administrative database role bypasses row security and must not be used directly for user-scoped queries.

Catalog changes remain an administrative responsibility. Browser clients must never receive administrative database credentials.

## Development Roadmap

These phases describe the implementation sequence. Features listed below are planned unless explicitly identified as completed above.

### Phase 1 — Repository and Application Foundation

- [ ] Select the application framework, package manager, and hosting approach using BayMatt Labs conventions.
- [ ] Scaffold the application and establish the source structure.
- [ ] Add versioned database migrations and a catalog-only seed/import process.
- [ ] Add environment-variable documentation and an example configuration without credentials.
- [ ] Bring the approved text-only mockups and permitted brand assets into the repository.
- [ ] Define shared design tokens for typography, colors, spacing, and both themes.
- [ ] Establish automated formatting, code checks, and an initial CI workflow.
- [ ] Document actual local setup and development commands.

**Ready when:** a new contributor can set up the application and a development database from repository instructions without using production credentials or personal collection data.

### Phase 2 — Authentication and Collections

- [ ] Implement the chosen sign-in methods using Neon Auth.
- [ ] Add sign-in, sign-out, session restoration, and protected application routes.
- [ ] Define and implement the first-collection experience.
- [ ] Support collection selection and agree on collection creation, renaming, and deletion behavior.
- [ ] Integrate the restricted database role and verified user context.
- [ ] Add loading, empty, and authentication-error states.
- [ ] Test access with multiple users and collections.

**Ready when:** users can sign in, reach their own collection, and cannot read or change another user's private data.

### Phase 3 — National Dex Dashboard and Collection Tracking

- [ ] Build the overall completion summary and clickable regional charts.
- [ ] Implement All Regions and Standard / Regional / Mega / Primal scopes.
- [ ] Build the responsive monster list without character artwork.
- [ ] Add name and Dex-number search, sorting, and All / Collected / Missing filters.
- [ ] Add collection toggles and row options for notes.
- [ ] Implement automatic saving, error recovery, and undo.
- [ ] Remember region, scope, filters, and sorting.
- [ ] Implement system-aware theme selection with a remembered manual override.
- [ ] Cover progress calculations and ownership updates with meaningful tests.

**Ready when:** users can browse, find, and update monsters on desktop and mobile, and every progress total stays consistent with the selected scope.

### Phase 4 — Activity and Milestones

- [ ] Add a recently collected view using ownership timestamps.
- [ ] Distinguish the imported collection baseline from subsequent additions.
- [ ] Decide whether activity is limited to recent additions or includes a permanent change history.
- [ ] Add an activity-event migration if permanent history is selected.
- [ ] Define and display regional and scope-specific completion milestones.
- [ ] Make milestone feedback accessible and avoid repeating celebrations after every refresh.

**Ready when:** users can understand recent changes and completed milestones without treating imported ownership as newly collected activity.

### Phase 5 — Optional Progress Sharing

- [ ] Add opt-in sharing settings and public link identifiers through a migration.
- [ ] Define whether shared pages expose summaries, the monster checklist, or both.
- [ ] Build a responsive, read-only public progress view.
- [ ] Allow collection owners to disable or revoke sharing.
- [ ] Keep notes, email addresses, and authentication details out of public responses.
- [ ] Test shared-page access separately from private collection access.

**Ready when:** an owner can deliberately share progress, visitors cannot edit the collection, and disabling sharing removes public access.

### Phase 6 — Release Readiness

- [ ] Verify the complete experience in both themes across desktop and mobile sizes.
- [ ] Check keyboard navigation, screen-reader labels, contrast, and touch targets.
- [ ] Verify empty collections, completed regions, filtered-out updates, and failed saves.
- [ ] Run end-to-end tests for authentication, collection updates, notes, undo, and sharing.
- [ ] Confirm production environment configuration and database migrations.
- [ ] Configure error reporting and operational checks.
- [ ] Document the deployed URL, release process, and support path.
- [ ] Update this README to distinguish shipped features from remaining plans.

**Ready when:** the deployed application meets the agreed tracking, privacy, accessibility, and reliability requirements.

## Decisions Still to Confirm

- Frontend framework, hosting provider, and production address.
- Sign-in methods and the new-user collection workflow.
- Collection management controls and any collection-count limits.
- Whether to keep the current 11 region groupings in the final interface.
- Recent additions versus a permanent activity history.
- The exact information included in shared progress pages.
- Offline requirements and whether they belong in a later release.
- Repository licensing and the process for maintaining the shared catalog.

## Outside the Initial Scope

- Individual card sets, printings, finishes, grades, quantities, and prices.
- Monster thumbnails, sprites, or character artwork.
- Importing the Forms worksheet or Alcremie's combination matrix.
- Gigantamax tracking.

The initial import also omitted Shaymin Sky because it falls outside the current standard, regional, Mega, and Primal categories. Galarian Linoone's duplicated workbook identifier was resolved to National Dex number 264.

## Local Development

The application has not been scaffolded yet. Install, development-server, test, and build commands will be added during Phase 1 after the stack is selected.

Secrets and personal collection exports must stay out of version control. Development and test work should use an isolated database branch.

---

**Tallgrass**

*Made with care by BayMatt Labs.*
