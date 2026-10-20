---
name: abap-dev-style
description: Use when Codex writes, reviews, explains, or modifies SAP ABAP code for this user, especially custom SAP report programs in the user's local workspace style with FORM/PERFORM flow, Open SQL, ALV field catalogs, MM/PP tables, BAPI/BDC/function modules, and incremental field or behavior changes. Prefer chat-first, paste-ready ABAP code blocks with insertion guidance over editing files unless the user explicitly asks Codex to modify files.
---

# ABAP Dev Style

## Operating Mode

Default to a chat-first workflow for ABAP changes.

- Provide paste-ready `abap` code blocks in the conversation unless the user explicitly asks to edit files.
- Include the target insertion point: program name, section, nearby `FORM`, event block, type, `SELECT`, mapping loop, or ALV field catalog.
- Split snippets by insertion location when a change touches multiple places.
- Include enough surrounding context to paste safely, but avoid dumping whole reports unless requested.
- If the user asks to modify files directly, make the smallest complete edit and still summarize the changed areas with key snippets.
- Inspect the target report before giving file-specific guidance. Match the style of that file over generic ABAP preferences.

## Local Reference

Before producing substantial ABAP, consult `references/workspace-style.md` for the user's observed local conventions, then verify against the current target file.

## Report Shape

Preserve classic custom report structure when the file already uses it:

```abap
INITIALIZATION.
  PERFORM frm_init.

AT SELECTION-SCREEN.
  PERFORM frm_check_data.

START-OF-SELECTION.
  PERFORM frm_get_data.
  PERFORM frm_edit_data.
  PERFORM frm_display_alv.
```

Use focused `FORM frm_* ... ENDFORM.` routines and call them with `PERFORM`. Avoid converting procedural reports to classes unless the user asks for a redesign.

## Naming

Follow the target file's prefixes and casing:

- Global internal tables: `gt_*`; global work areas: `gs_*`.
- Local internal tables: `lt_*`; local work areas: `ls_*` or `lw_*`.
- Local scalar variables: `lv_*`.
- Constants: `cns_*` or the existing local constant prefix.
- Types: `gty_*`, `typ_*`, `ty_*`, or the file's existing pattern.
- Field symbols: `<fs_*>`.
- Selection-screen fields: `p_*`, `s_*`, and radio buttons matching existing names.
- Forms: `frm_*`.

Keep ABAP keyword and identifier casing consistent with nearby code. Mixed legacy casing is acceptable when the surrounding file uses it.

## Data And SQL

Use SAP Dictionary types for fields from standard or custom tables. Prefer explicit fields over `SELECT *`.

- Match the file's Open SQL style. Use host variables (`@`) only when the report already uses modern syntax or the requested snippet requires it.
- Use joins for primary result sets when the report already does so.
- Use `INTO CORRESPONDING FIELDS OF TABLE` when target field names match.
- Guard `FOR ALL ENTRIES` and follow the local pattern for initial table checks.
- `SORT ... BY ...` before `READ TABLE ... BINARY SEARCH`.
- Keep authority checks, `sy-subrc` handling, and message behavior consistent with the current program.

## ALV

Preserve the existing ALV implementation, whether it uses `REUSE_ALV_GRID_DISPLAY_LVC`, `CL_GUI_ALV_GRID`, SALV, or a local field catalog macro.

- Add new ALV fields near related business fields.
- Use Chinese column labels consistent with existing business wording.
- Prefer `ref_table` and `ref_field` for standard SAP fields when the local field catalog supports them.
- Respect existing editable-cell helpers and do not make display-only fields editable unless requested.

## Complete Change Path

For an ABAP field or behavior change, trace and update every required place:

1. Output type or ALV structure.
2. Data retrieval: `SELECT`, function module result, BAPI return, BDC result, or derived table.
3. Mapping or calculation: `LOOP AT`, `READ TABLE`, `MOVE-CORRESPONDING`, `MODIFY`, or `APPEND`.
4. ALV field catalog, layout, editable style, or screen/table-control binding.
5. Mode-specific branches such as summary/detail radio buttons.

State which areas are changed. If a field belongs only to summary or detail mode, explicitly preserve the other mode.

## Comments And Change Marks

Keep comments short and business-focused. Use Chinese comments when explaining business fields or SAP process behavior.

Match existing change markers when adding a marked block:

```abap
"ADD BY NAME YYYYMMDD
**--ADD by NAME YYYY/MM/DD Start-----
**--ADD by NAME YYYY/MM/DD End-------
```

For small snippets, prefer concise inline comments only where the reason is not obvious.

## Response Pattern

When answering an ABAP coding request, use this shape:

1. Briefly state the assumed target section or file.
2. Provide one or more paste-ready `abap` code blocks.
3. Mention any related updates the user must make, such as text symbols, GUI status, message class entries, or selection-screen texts.
4. Keep explanations compact and focused on SAP behavior.
