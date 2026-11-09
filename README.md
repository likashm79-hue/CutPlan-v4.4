# CutPlan v0.4.4

CutPlan converts cutting lists into monochrome, portrait sheet layouts using a multi-stage guillotine packing heuristic.

## v0.4.4 changes

- **PDF pagination:** one board per A4 portrait PDF page. Boards are never arranged two-up in print output.
- **Drawing labels:** dimension values include a space before `mm` (for example, `780 mm`); `Length` / `Width` prefixes are removed. Dimension values are enlarged where the part has room and moved into a compact external reference schedule when a part is too small.
- **No edge-banding on drawings:** drawings contain no edge-banding colors, patterns, edge highlights, symbols, labels or legends. Edge banding remains available in the material totals.
- **Manual edge-banding controls:** every row has independent editable edge counts for 0.8 mm and 2 mm material on the length and width dimensions. Counts are 0, 1 or 2. Dashboard metre totals update immediately after an edit.
- **Safer extracted-row parsing:** data-like lines that cannot be confidently parsed are kept in an "Items needing review" editor. Correct the reference, dimensions, quantity and rotation, then add the row. Flattened PDF/OCR lines containing multiple part records are split into separate rows where their structure is recognizable. The parser no longer silently discards these source lines.
- **Existing guillotine engine preserved:** the nesting and cut-splitting code is unchanged from v0.4.3.

## Three cutting modes

- **Cut along width** — fixes the primary cut direction across the sheet width.
- **Cut along length** — fixes the primary cut direction along the sheet length.
- **Optimal** — evaluates both primary cut directions and chooses the best validated result found.

All three modes use the same guillotine placement and splitting rules. The optimizer is heuristic and does not prove global optimality.

## Cutting-list input and edge-banding notation

The preferred row order is:

`Reference | Length | Width | Quantity`

Rotation may be included as a fifth value in CSV or recognized text.

- References may be numeric or alphanumeric, for example `1A`, `2KB`, `CAB-A1`.
- Append one underscore `_` per edge requiring **0.8 mm** edge banding.
- Append one tilde `~` per edge requiring **2 mm** edge banding.
- The markers are interpreted independently. For example, `780__` means two 780 mm edges with 0.8 mm banding; `500_` means one 500 mm edge with 0.8 mm banding; `600~~` means two 600 mm edges with 2 mm banding.
- No marker means no banding for that dimension.
- The row table also has manual counts: `0.8 L`, `0.8 W`, `2 L`, `2 W`. `L` refers to the length dimension and `W` to the width dimension. Each count represents 0, 1 or 2 edges.
- Automatic markers remain editable. Manual edits flow into CSV export and dashboard material totals.
- PDF/image extraction should always be reviewed. Unresolved data-like lines are shown separately and can be corrected; OCR can still misread small underscore and tilde marks.

The dashboard calculates total linear metres for 0.8 mm and 2 mm banding, including row quantity. Banding is a material estimate only; it is not drawn or used to change cutting geometry.

## Drawing and PDF output

- Board drawings are portrait and monochrome.
- Dimension values appear without length/width labels and include a space before `mm`.
- Original cutting-list references are shown inside parts where space allows.
- Small-part dimensions are repeated in an external reference schedule when they cannot fit legibly inside the part.
- Every board is printed on its own A4 portrait page, including the final board without an extra trailing page.
- Edge banding is not represented visually in drawings.

## Files

- `index.html` — browser interface, PDF/image/CSV/manual import, dashboard, drawings and print/PDF export.
- `cutting-list-parser.js` — alphanumeric reference, dimension suffix and OCR text parsing with unresolved-line recovery data.
- `nesting-engine.js` — multi-stage guillotine engine and geometry validation; unchanged from v0.4.3.
- `test-engine.js` — Node.js regression checks, including the 27-row / 51-piece benchmark, banding totals and parser recovery.

## Run locally

Keep all four application files in one folder, then run:

```bash
python3 -m http.server 8000
```

Open `http://localhost:8000` in your browser.

## Run engine and parser tests

```bash
node test-engine.js
```

## Important production notes

- Set the actual sheet size, measured blade kerf and edge margin before generating a production plan.
- A blank kerf is treated as 0 mm and is explicitly provisional; it is not production-ready.
- The engine places parts only in rectangular free regions and splits those regions with straight guillotine cuts. It validates piece count, bounds, overlaps and recorded cut extents.
- The optimizer is heuristic. It searches 12, 64 or 240 deterministic strategies depending on the selected effort; it does not prove global optimality.
- Verify the resulting cut order, grain direction, clamping, trim allowance and banding requirements against the actual saw and job before cutting.
