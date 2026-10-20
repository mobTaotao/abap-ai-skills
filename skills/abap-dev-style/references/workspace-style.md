# Workspace Style Notes

These notes are derived from the user's ABAP workspace, especially:

- `ZMMR4011_no_approval.abap`
- `ZPPR011_operation_type_modified.abap`

Use these observations as a starting point, then inspect the active target program and match its local style.

## Program Header

Reports commonly start with SAP banner comments, a `REPORT z... MESSAGE-ID ...` statement, and Chinese business description or edit-log comments.

```abap
*&---------------------------------------------------------------------*
*& Report ZPPR011
*&---------------------------------------------------------------------*
*  作者:D163432
*  描述:投入产出报表
*&---------------------------------------------------------------------*
REPORT zppr011 MESSAGE-ID zpp.
```

For new report snippets, include a compact header only when the user asks for a full report. For small modifications, do not add a new banner unless the surrounding code uses one for similar edits.

## Declaration Style

The workspace uses both `TYPES: BEGIN OF ... END OF ...` structures and global `DATA:` blocks. Field comments are usually Chinese and placed inline after the field.

```abap
TYPES: BEGIN OF gty_alv,
         werks TYPE aufk-werks,   "工厂
         aufnr TYPE aufk-aufnr,   "生产订单号
         matnr TYPE afpo-matnr,   "产品编码
       END OF gty_alv.

DATA: gt_alv TYPE TABLE OF gty_alv,
      gs_alv TYPE gty_alv.
```

Common prefixes:

- `gt_`: global internal table
- `gs_`: global work area or structure
- `lt_`: local internal table
- `ls_` / `lw_`: local work area
- `lv_`: local scalar
- `gty_` / `typ_`: type definitions
- `cns_`: constants
- `<fs_*>`: field symbols

## Procedural Flow

Reports are generally procedural and use event blocks with `PERFORM` calls.

```abap
START-OF-SELECTION.
  "权限检查
  PERFORM frm_authority_check.
  "获得数据
  PERFORM frm_get_data TABLES gt_alv.
  "ALV展示数据
  PERFORM frm_display_alv TABLES gt_alv.
```

Forms use `frm_*` names and usually have a banner comment above them:

```abap
*&---------------------------------------------------------------------*
*& Form frm_get_data
*&---------------------------------------------------------------------*
FORM frm_get_data TABLES t_alv STRUCTURE gs_alv.

  REFRESH: t_alv.

ENDFORM.
```

## SQL Style

The workspace contains both classic and newer Open SQL. Match the target file.

Classic style example:

```abap
SELECT a~aufnr,     "生产订单号
       a~objnr,     "对象号
       a~auart,     "订单类型
       b~matnr      "单品物料编码
  FROM aufk AS a
 INNER JOIN afpo AS b ON b~aufnr = a~aufnr
  INTO CORRESPONDING FIELDS OF TABLE @lt_aufk
 WHERE a~werks = @p_werks
   AND a~aufnr IN @s_aufnr.
```

Local expectations:

- Put field comments beside selected fields when the list is long.
- Keep aliases (`a~`, `b~`, `ekko~`, `ekpo~`) aligned with nearby code.
- Use SAP table-field types for structure fields.
- Check driver tables before secondary selects.
- Sort before binary-search reads.

## ALV Style

The workspace uses LVC/SLIS ALV patterns, including `REUSE_ALV_GRID_DISPLAY_LVC`, `cl_gui_alv_grid`, `lvc_t_fcat`, `slis_t_extab`, and callback forms.

For field additions, update the structure, retrieval/mapping, and field catalog together.

```abap
CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY_LVC'
  EXPORTING
    i_callback_program      = lv_repid
    i_callback_user_command = 'FRM_USER_COMMAND'
    is_layout_lvc           = ls_layout
    it_fieldcat_lvc         = gt_fieldcat_lvc
    i_save                  = 'X'
  TABLES
    t_outtab                = gt_tab.
```

When a field catalog macro is present, use it instead of creating a different pattern:

```abap
fcat 'NETWR' '' '' '' '净值' 'EKPO' 'NETWR' 0.
```

## BAPI, Function Module, And BDC Handling

Function module calls use aligned `EXPORTING`, `IMPORTING`, `TABLES`, and `EXCEPTIONS` sections. Always handle `sy-subrc` or returned messages in the same style as the file.

```abap
CALL FUNCTION 'MD_CONVERT_MATERIAL_UNIT'
  EXPORTING
    i_matnr              = <fs_data>-matnr
    i_in_me              = <fs_data>-meins
    i_out_me             = <fs_data>-meins_base
    i_menge              = <fs_data>-menge
  IMPORTING
    e_menge              = <fs_data>-menge
  EXCEPTIONS
    error_in_application = 1
    error                = 2
    OTHERS               = 3.
```

## Chat-First Output

For normal ABAP coding help, output code in chat rather than editing files.

Use separate code blocks for each insertion point:

```text
位置一：在 TYPES: BEGIN OF gty_alv 中加入
```

```abap
        zzfield TYPE mara-zzfield, "自定义字段
```

```text
位置二：在 frm_get_data 的 SELECT 字段清单中加入
```

```abap
       m~zzfield, "自定义字段
```

Only edit files directly when the user says to modify the file, apply the change, or similar.
