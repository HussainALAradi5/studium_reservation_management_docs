# Regional use cases

### UC-REG-01 Admin creates a city
- **Actor:** Admin
- **Preconditions:** Signed in.
- **Main flow:** Opens Cities → "New city" dialog → name + country code → `POST /api/cities` → saved with generated code → table row added without a page reload.
- **Failure flows:** Duplicate name in the same country → 409 field error; invalid country code → 422.
- **Result:** City selectable in stadium addresses and search filters.

### UC-REG-02 Admin edits a city
- **Main flow:** Edit dialog → `PUT /api/cities/{code}` → name, country, active flag updated.
- **Result:** Deactivated cities disappear from filters and new address forms; existing addresses keep them.

### UC-REG-03 Admin deletes a city
- **Main flow:** Confirm dialog → `DELETE /api/cities/{code}` → soft-deleted.
- **Failure flows:** City used by an address → 409 `CITY_IN_USE`.
