# Final Delivery Checklist

## 1. Functional Acceptance

- [ ] Users can create `P-Item` records.
- [ ] Users can create a `P-BOM` with at least 5 `P-BOM Item` child rows.
- [ ] `P-BOM Item` can reference `P-Item` through a Link field.
- [ ] Duplicate BOM child items are blocked.
- [ ] Child quantity less than or equal to 0 is blocked.
- [ ] Header `total_qty` is calculated from child row quantities.
- [ ] Users can create `P-ECO` and link it to `P-BOM`.
- [ ] When an ECO is released, the linked BOM version updates to the target version.
- [ ] When an ECO is released, the linked BOM status becomes Approved.

## 2. Permissions and Workflow

- [ ] Engineering users can create draft BOMs.
- [ ] Engineering users cannot submit BOMs.
- [ ] Engineering managers can submit BOMs.
- [ ] ECO Workflow includes Draft, Under Review, and Released.
- [ ] Released ECOs cannot be edited.
- [ ] Quality users can only view approved BOMs and released ECOs.

## 3. Reports and API

- [ ] Query Report shows approved BOM structures.
- [ ] Report can filter by finished item.
- [ ] Report can export to Excel.
- [ ] API token can read the `P-Item` list.
- [ ] API token can create one test item.

## 4. Engineering Delivery

- [ ] Custom DocTypes are in the `pdm_tutorial` Custom App.
- [ ] Key roles, permissions, workflow, and field configuration are exported through fixtures.
- [ ] `bench migrate` has been run to verify migration.
- [ ] `bench clear-cache` has been run to verify refresh behavior.
- [ ] At least one unit test covers BOM child item validation.
- [ ] Code has been committed to Git with a clear commit message.

## 5. Final Demo

- [ ] Demonstrate the full flow from item creation to ECO release.
- [ ] Demonstrate one failure case, such as duplicate child item validation.
- [ ] Demonstrate permission differences between engineering user and engineering manager.
- [ ] Demonstrate report query and API call results.
