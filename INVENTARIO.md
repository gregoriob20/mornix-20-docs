# Inventario de modulos del cliente

Generado desde los repos clonados en `/opt/odoo-client/`. `origen` clasifica por el campo `author` del manifest.

- **cliente**: codigo propio (Nimetrix / Oasis / Mornix / ingenieros) -> se migra a mano.
- **tercero**: modulo de un vendor externo -> hay que conseguir su version v20, no migrarlo.
- **OCA**: comunidad -> se toma del upstream cuando exista rama 20.0.
- `dup` marca modulos que existen en v16 **y** v18: partir de la version v18, ya trae 2 saltos hechos.

Total: **226 modulos**, **282,491 lineas** (py+xml+js).


## v16 — 140 modulos, 155,307 lineas

| repo | modulo | origen | lineas | deps | dup | autor |
|---|---|---|---|---|---|---|
| l10n_ve_odoo_16 | `l10n_ve_full` | cliente | 21,580 | 13 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `theme_clarico_vega` | tercero | 12,116 | 2 |  | Emipro Technologies Pvt. Ltd. |
| l10n_ve_odoo_16 | `l10n_ve_dpt` | tercero | 9,500 | 1 | si | INM & LDR Soluciones Tecnológicas y |
| l10n_ve_odoo_16 | `os_sale_reports` | cliente | 6,985 | 11 |  | Oasis Consultora  C.A |
| l10n_ve_odoo_16 | `account_dual_currency` | cliente | 6,647 | 13 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `base_marketplace` | tercero | 6,633 | 3 |  | Teqstars |
| l10n_ve_odoo_16 | `emipro_theme_base` | tercero | 4,761 | 4 |  | Emipro Technologies Pvt. Ltd. |
| l10n_ve_odoo_16 | `tvs_module` | cliente | 4,671 | 6 |  | Consultores G3C  C.A |
| l10n_ve_odoo_16 | `os_l10n_ve_fiscal` | cliente | 4,050 | 6 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `dhv_module_extend` | cliente | 3,786 | 10 |  | Oasis Consultora  C.A |
| l10n_ve_odoo_16 | `os_generate_reports` | cliente | 3,135 | 4 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `os_logistic_consignment` | cliente | 2,808 | 6 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `account_loan` | OCA | 2,634 | 1 |  | Creu Blanca,Odoo Community Associat |
| l10n_ve_sucursales | `branch` | tercero | 2,543 | 9 | si | BrowseInfo |
| l10n_ve_odoo_16 | `l10n_ve_pos` | cliente | 2,455 | 4 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `os_fiscal_print` | cliente | 2,287 | 5 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `l10n_ve_stock_account` | tercero | 2,108 | 3 | si | binaural-dev |
| l10n_ve_odoo_16 | `bi_import_pricelist` | tercero | 2,063 | 4 |  | BrowseInfo |
| l10n_ve_odoo_16 | `nimetrix_detailed_sale_report` | cliente | 1,914 | 1 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_tags_reports` | tercero | 1,806 | 4 |  | ErpMstar Solutions |
| l10n_ve_odoo_16 | `os_reports_guzzo` | cliente | 1,796 | 4 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report_v2` | cliente | 1,612 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_pos_dual_currency` | cliente | 1,525 | 11 |  | OasisConsultora C.A. |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report` | cliente | 1,514 | 1 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `nx_fiscal_print` | cliente | 1,328 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `stock_picking_batch_extended` | OCA | 1,310 | 2 |  | Camptocamp, Tecnativa, Odoo Communi |
| l10n_ve_odoo_16 | `os_anticipos` | cliente | 1,248 | 2 |  | Consultores G3C  C.A |
| l10n_ve_sucursales | `branch_accounting_report` | tercero | 1,239 | 5 |  | BrowseInfo |
| l10n_ve_odoo_16 | `invoice_bs_usd_diera` | cliente | 1,153 | 5 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `l10n_ve_kardex` | cliente | 988 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `l10n_ve_fiscal_book` | cliente | 965 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `pos_upgrade_nimetrix_autanashop` | cliente | 956 | 5 |  | Nimetrix, Inc |
| l10n_ve_odoo_16 | `pos_vpos` | cliente | 937 | 1 |  | Luis Pinzón, Nimetrix |
| l10n_ve_odoo_16 | `nimetrix_stock_invoiced_qty` | cliente | 914 | 6 |  | Nimetrix C.A |
| l10n_ve_sucursales | `filter_agend_partner_balance_branch` | cliente | 870 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_comission_multimax` | cliente | 824 | 9 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `nimetrix_report_arc` | cliente | 815 | 3 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `purchase_discount` | OCA | 791 | 1 |  | Tiny, Acysos S.L., Tecnativa, ACSON |
| l10n_ve_odoo_16 | `nimetrix_report_complementsv2` | cliente | 783 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `bi_pos_restrict_stock` | tercero | 764 | 2 |  | BrowseInfo |
| l10n_ve_odoo_16 | `invoice_bs_usd` | cliente | 739 | 5 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `product_logistics_uom` | OCA | 705 | 1 |  | Akretion, ACSONE SA/NV, Odoo Commun |
| l10n_ve_odoo_16 | `stock_limitation` | tercero | 696 | 2 |  | faOtools |
| l10n_ve_odoo_16 | `stock_available` | OCA | 665 | 1 |  | Numérigraphe, Sodexis, Odoo Communi |
| l10n_ve_sucursales | `bi_branch_budget_ent` | tercero | 653 | 2 | si | BrowseInfo |
| l10n_ve_odoo_16 | `nimetrix_pos_resume_report` | cliente | 651 | 3 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `solariums` | cliente | 648 | 8 |  | ING Jose Blanco |
| l10n_ve_odoo_16 | `l10n_ve_stock` | cliente | 643 | 5 | si | Oasis Consultora  C.A |
| l10n_ve_sucursales | `bi_odoo_multi_branch_hr` | tercero | 636 | 7 | si | BrowseInfo |
| l10n_ve_odoo_16 | `nimetrix_report_complements` | cliente | 591 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_product_replacement_cost` | cliente | 590 | 7 |  | Oasis Consultora  C.A |
| l10n_ve_odoo_16 | `os_generate_pricelist` | cliente | 581 | 4 |  | Oasis Consultora C.A. |
| l10n_ve_sucursales | `l10n_ve_branch_reports` | cliente | 576 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_municipal_taxes` | cliente | 545 | 4 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `sale_account_manager_customer_credit_limit_approval` | tercero | 540 | 3 |  | TechUltra Solutions Private Limited |
| l10n_ve_sucursales | `bi_branch_pos` | tercero | 532 | 3 | si | BrowseInfo |
| l10n_ve_odoo_16 | `os_igtf_account` | cliente | 530 | 6 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `gandocam_module_extend` | cliente | 502 | 2 |  | Oasis Consultora  C.A |
| l10n_ve_sucursales | `branch_l10n_ve_kardex` | cliente | 482 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `import_pickings_aV16` | tercero | 469 | 3 |  | Caret IT Solutions Pvt. Ltd. |
| l10n_ve_odoo_16 | `zero_part_no` | tercero | 439 | 4 |  | Zero Systems |
| l10n_ve_odoo_16 | `nimetrix_cost_history` | cliente | 421 | 4 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `pos_stock_available_online` | OCA | 417 | 3 |  | Cetmix, Odoo Community Association  |
| l10n_ve_sucursales | `bi_odoo_mrp_multi_branch` | tercero | 406 | 3 | si | BrowseInfo |
| l10n_ve_odoo_16 | `nimetrix_report_igtf` | cliente | 386 | 3 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_fix_wh_iva` | cliente | 382 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_pos_igtf` | cliente | 374 | 2 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `stock_no_negative` | OCA | 362 | 1 | si | Akretion,Odoo Community Association |
| l10n_ve_sucursales | `nimetrix_branch_bank_statement` | cliente | 358 | 4 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `update_cost_price` | cliente | 351 | 4 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `nimetrix_daily_book_report` | cliente | 351 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_pos_payment_reference` | ? | 348 | 3 |  |  |
| l10n_ve_odoo_16 | `product_packaging_dimension` | OCA | 337 | 2 |  | Camptocamp, Akretion, Odoo Communit |
| l10n_ve_odoo_16 | `nimetrix_municipal_taxes_report` | cliente | 334 | 5 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `import_internal_transfer_app` | tercero | 334 | 1 |  | Edge Technologies |
| l10n_ve_odoo_16 | `nimetrix_detailed_sale_report_consignment` | cliente | 315 | 3 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_generate_doc_origin` | cliente | 309 | 4 |  | Consultores G3C  C.A |
| l10n_ve_odoo_16 | `mai_product_pricelist_dynamic_listview` | tercero | 308 | 1 |  | MAISOLUTIONSLLC |
| l10n_ve_odoo_16 | `nimetrix_standard_report_invoice` | cliente | 299 | 2 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_invoice_delivery_relationship` | cliente | 296 | 1 |  | Nimetrix C,A |
| l10n_ve_odoo_16 | `unidades_presentacion` | cliente | 283 | 3 | si | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `product_dimension` | OCA | 277 | 1 |  | brain-tec AG, ADHOC SA, Camptocamp  |
| l10n_ve_sucursales | `bi_odoo_multi_branch_project` | tercero | 276 | 3 |  | BrowseInfo |
| l10n_ve_odoo_16 | `nimetrix_commission_report` | cliente | 272 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `ob_invoice_line_view` | tercero | 269 | 1 |  | Odoo Bin |
| l10n_ve_sucursales | `branch_analytic_account` | cliente | 265 | 2 | si | Nimetrix C.A. |
| l10n_ve_odoo_16 | `os_autana_shop_module_extend` | cliente | 261 | 2 |  | OasisConsultora C.A. |
| l10n_ve_odoo_16 | `stock_picking_group_by_max_weight` | OCA | 260 | 2 |  | ACSONE SA/NV,Odoo Community Associa |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report_consignment_v2` | cliente | 259 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `ip_import_picking` | tercero | 248 | 1 |  | iPredict IT Solutions Pvt. Ltd. |
| l10n_ve_odoo_16 | `nimetrix_mail_withholding_taxes` | cliente | 244 | 3 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_internal_tranfers` | cliente | 243 | 2 |  | Oasisconsultora C.A. |
| l10n_ve_sucursales | `bi_odoo_crm_multi_branch` | tercero | 238 | 3 |  | BrowseInfo |
| l10n_ve_sucursales | `bi_multi_branch_helpdesk` | tercero | 231 | 4 |  | BrowseInfo |
| l10n_ve_odoo_16 | `os_filter_financial_reports` | cliente | 226 | 3 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report_consignment` | cliente | 224 | 3 |  | Nimetrix C.A |
| l10n_ve_sucursales | `os_sale_reports_branch_filter` | cliente | 224 | 4 |  | Oasis Consultora  C.A |
| l10n_ve_odoo_16 | `pways_restrict_quantity` | tercero | 213 | 5 |  | Preciseways |
| l10n_ve_odoo_16 | `nimetrix_detailed_sale_report_comission` | cliente | 209 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_pricelist` | cliente | 203 | 2 |  | OasisConsultora C.A. |
| l10n_ve_odoo_16 | `nimetrix_change_tasa_ret_iva` | cliente | 198 | 1 |  | Nimetrix C.A |
| l10n_ve_sucursales | `bi_odoo_multi_branch_tendor` | tercero | 188 | 4 |  | BrowseInfo |
| l10n_ve_odoo_16 | `os_fields_required` | cliente | 185 | 3 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `fecha_vencimiento` | cliente | 177 | 1 |  | Nimetrix C.A |
| l10n_ve_sucursales | `bi_multi_branch_subscriptions` | tercero | 177 | 5 |  | BrowseInfo |
| l10n_ve_odoo_16 | `l10n_ve_fiscal_book_wh_line` | cliente | 175 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `pos_product_limit_odoo` | tercero | 170 | 3 |  | Cybrosys Techno Solutions |
| l10n_ve_odoo_16 | `bi_print_journal_entries` | tercero | 169 | 2 |  | BrowseInfo |
| l10n_ve_odoo_16 | `fiscal_lock_days` | cliente | 162 | 2 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `unidades_permitidas` | cliente | 157 | 3 | si | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `nimetrix_change_price_purchase` | cliente | 148 | 4 |  | Nimetrix C.A |
| l10n_ve_sucursales | `bi_branch_assets_ent` | tercero | 147 | 2 |  | BrowseInfo |
| l10n_ve_odoo_16 | `os_fields_required_autanashop` | cliente | 142 | 3 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `cancel_invoice_auto` | cliente | 140 | 3 |  | ING. Miguel Alzuru |
| l10n_ve_sucursales | `bi_branch_scrap_order` | tercero | 137 | 3 | si | BrowseInfo |
| l10n_ve_odoo_16 | `l10n_ve_bank_reconciliation` | cliente | 135 | 1 |  | Nimetrix |
| l10n_ve_odoo_16 | `nimetrix_restrictions` | cliente | 134 | 1 | si | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_fix_partner_pay_rec` | cliente | 129 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `permission_purchase_price` | cliente | 117 | 4 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `nimetrix_pos_refund_password` | tercero | 112 | 1 |  | Cybrosys Techno Solutions |
| l10n_ve_odoo_16 | `stock_picking_group_by_base` | OCA | 104 | 1 |  | ACSONE SA/NV,Odoo Community Associa |
| l10n_ve_odoo_16 | `os_plazo_cobranza` | cliente | 92 | 4 |  | Oasis Consultora C.A. |
| l10n_ve_odoo_16 | `ssq_purchase_multi_product_selection` | tercero | 89 | 1 |  | Sanesquare Technologies |
| l10n_ve_odoo_16 | `nimetrix_change_price_sale` | cliente | 83 | 4 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_change_price_invoice` | cliente | 82 | 3 |  | Nimetrix C.A |
| l10n_ve_sucursales | `nimetrix_general_sale_report_branch` | cliente | 80 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `os_cost_standard_price_stock` | cliente | 78 | 4 |  | Oasis Consultora C.A |
| l10n_ve_odoo_16 | `b1_pos_company_logo` | tercero | 74 | 1 |  | Brent137 |
| l10n_ve_odoo_16 | `os_fields_remove_sale` | cliente | 74 | 3 |  | Nimetrix |
| l10n_ve_odoo_16 | `status_despacho` | cliente | 73 | 2 |  | ING Miguel Alzuru |
| l10n_ve_odoo_16 | `sale_invoice_limit` | cliente | 68 | 1 |  | Nimetrix C.A. |
| l10n_ve_odoo_16 | `os_pos_sale_filter` | ? | 60 | 2 |  |  |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report_comission` | cliente | 58 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_general_sale_report_comission_v2` | cliente | 58 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `hide_button_refund` | cliente | 57 | 2 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_fix_arc_proveedores` | cliente | 57 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `nimetrix_vat_withholding_statements` | cliente | 55 | 1 |  | Nimetrix C.A |
| l10n_ve_odoo_16 | `auto_check_availability` | cliente | 51 | 2 |  | ING. Luis Marcano |
| l10n_ve_odoo_16 | `nimetrix_ve_admin_note` | cliente | 42 | 1 |  | Nimetrix C.A. |
| l10n_ve_sucursales | `bi_all_in_one_branch_ent` | tercero | 33 | 14 |  | BrowseInfo |

## v18 — 86 modulos, 127,184 lineas

| repo | modulo | origen | lineas | deps | dup | autor |
|---|---|---|---|---|---|---|
| nx_localizacion | `l10n_ve_nimetrix` | cliente | 14,788 | 7 |  | Nimetrix C.A |
| nx_localizacion | `l10n_ve_dpt` | tercero | 9,498 | 1 | si | INM & LDR Soluciones Tecnológicas y |
| nx_tools | `advanced_web_domain_widget` | tercero | 6,199 | 1 |  | Terabits Technolab |
| nx_tools | `nimetrix_klk_integration` | cliente | 5,413 | 7 |  | Nimetrix |
| nx_tools | `nimetrix_klk_integration_standard` | cliente | 5,193 | 6 |  | Nimetrix |
| nx_tools | `payment_nx_megasoft` | cliente | 4,829 | 1 |  | Mornix C.A |
| nx_dual_currency | `nimetrix_dual_currency` | cliente | 4,789 | 5 |  | Nimetrix C.A |
| nx_localizacion | `tk_security_master` | tercero | 3,614 | 5 |  | TechKhedut Inc. |
| nx_localizacion | `l10n_ve_stock_account` | tercero | 3,244 | 3 | si | binaural-dev |
| nx_tools | `nimetrix_product_report` | cliente | 3,174 | 4 |  | Nimetrix C.A |
| nx_tools | `simplify_access_management` | tercero | 3,168 | 2 |  | Terabits Technolab |
| nx_sucursales | `branch` | tercero | 2,701 | 9 | si | BrowseInfo |
| nx_tools | `payment_nx_mercantil` | ? | 2,596 | 1 |  |  |
| nx_dual_currency | `nimetrix_account_reports_dual` | cliente | 2,490 | 2 |  | Nimetrix C.A |
| nx_point_of_sale | `nimetrix_fiscal_print` | cliente | 2,461 | 6 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_pos_dual_currency` | cliente | 2,251 | 6 |  | Nimetrix C.A |
| nx_tools | `nimetrix_tpv_despacho` | cliente | 2,097 | 4 |  | Nimetrix C.A |
| nx_tools | `nimetrix_novus` | cliente | 2,063 | 4 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_sitef` | cliente | 2,003 | 2 |  | Luis Pinzón, Nimetrix |
| nx_tools | `nimetrix_detailed_sale_report` | cliente | 1,896 | 2 | si | Nimetrix C.A |
| nx_dual_currency | `nimetrix_stock_cost_usd` | cliente | 1,812 | 6 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_currency_rate` | cliente | 1,751 | 5 |  | Nimetrix C.A |
| nx_dual_currency | `nimetrix_mrp_dual_currency` | ? | 1,695 | 3 |  |  |
| nx_tools | `payment_pagoapago` | cliente | 1,675 | 2 |  | Morna Tech |
| nx_tools | `nimetrix_iva_resumen_report` | cliente | 1,658 | 2 |  | Nimetrix C.A |
| nx_tools | `replic_poin_of_sale` | tercero | 1,611 | 1 |  | Your Company |
| nx_tools | `nimetrix_general_sale_report` | cliente | 1,586 | 2 | si | Nimetrix C.A |
| nx_localizacion | `nimetrix_retencion_municipal` | cliente | 1,514 | 3 |  | Nimetrix C.A |
| nx_tools | `nimetrix_thefactoryhka` | cliente | 1,505 | 5 |  | Nimetrix |
| nx_tools | `nimetrix_unidigital` | cliente | 1,491 | 3 |  | Nimetrix C.A |
| nx_tools | `payment_nx_mercantil_panama` | ? | 1,410 | 2 |  |  |
| nx_tools | `l10n_ve_stock` | cliente | 1,241 | 4 | si | Nimetrix C.A |
| nx_localizacion | `nimetrix_third_party_sales` | cliente | 1,221 | 2 |  | Nimetrix C.A |
| nx_tools | `employee_purchase_requisition` | tercero | 1,155 | 4 |  | Cybrosys Techno Solutions |
| nx_tools | `nimetrix_internal_cash_transfer` | cliente | 1,099 | 2 |  | Mornix C.A |
| nx_tools | `nimetrix_pos_qz_dispatch` | cliente | 1,040 | 2 |  | Nimetrix C.A |
| nx_tools | `nimetrix_unidigital_retenciones` | cliente | 967 | 1 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_report_arc` | cliente | 837 | 3 | si | Nimetrix C.A |
| nx_localizacion | `nimetrix_igtf` | cliente | 830 | 2 |  | Nimetrix C.A |
| nx_sucursales | `bi_branch_pos` | tercero | 809 | 3 | si | BrowseInfo |
| nx_tools | `nimetrix_standard_report_invoice_simple` | cliente | 803 | 4 |  | Nimetrix C.A |
| nx_dual_currency | `nimetrix_customer_statement` | cliente | 782 | 3 |  | Nimetrix C.A |
| nx_point_of_sale | `nimetrix_report_z_fiscal` | cliente | 777 | 3 |  | Nimetrix C.A |
| nx_tools | `nimetrix_optimized_reconcile` | cliente | 769 | 1 |  | Nimetrix C.A |
| nx_tools | `nimetrix_report_base` | cliente | 754 | 5 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_vpos` | cliente | 674 | 1 |  | Luis Pinzón, Nimetrix |
| nx_sucursales | `bi_branch_budget_ent` | tercero | 643 | 2 | si | BrowseInfo |
| nx_sucursales | `bi_odoo_multi_branch_hr` | tercero | 632 | 7 | si | BrowseInfo |
| nx_localizacion | `nimetrix_municipal_taxes` | cliente | 556 | 4 |  | Nimetrix C.A |
| nx_dual_currency | `nimetrix_list_price_property` | cliente | 504 | 1 |  | Nimetrix C.A |
| nx_sucursales | `mornix_branch_analytic_line_sync` | cliente | 500 | 1 |  | Mornix C.A |
| nx_tools | `cathequim_employee_purchase_requisition` | cliente | 497 | 1 |  | Nimetrix |
| nx_localizacion | `nimetrix_stock_account_report` | cliente | 471 | 2 |  | Nimetrix C.A. rosselyn.barroyeta@ni |
| nx_localizacion | `nimetrix_trial_balance_report` | cliente | 459 | 3 |  | Nimetrix C.A. |
| nx_localizacion | `nimetrix_standard_report_invoice` | cliente | 431 | 3 | si | Nimetrix C.A |
| nx_tools | `nimetrix_pos_reprint_distpatch` | cliente | 408 | 2 |  | Nimetrix C.A |
| nx_sucursales | `bi_odoo_mrp_multi_branch` | tercero | 406 | 3 | si | BrowseInfo |
| nx_tools | `stock_no_negative` | OCA | 396 | 1 | si | Akretion,Odoo Community Association |
| nx_point_of_sale | `nimetrix_standard_report_tpv` | cliente | 386 | 1 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_account_journal_report` | cliente | 372 | 3 |  | Nimetrix C.A |
| nx_tools | `nx_invoice_mass_delete` | cliente | 345 | 3 |  | Nimetrix C.A. |
| nx_sucursales | `branch_analytic_account` | cliente | 299 | 2 | si | Nimetrix C.A. |
| nx_tools | `unidades_presentacion` | cliente | 282 | 2 | si | Oasis Consultora C.A |
| nx_tools | `nimetrix_sypago` | ? | 245 | 2 |  |  |
| nx_dual_currency | `nimetrix_cost_usd_property` | cliente | 243 | 1 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_sale_fiscal_print` | cliente | 234 | 3 |  | Luis Pinzón, Nimetrix |
| nx_point_of_sale | `nx_vpos_print` | cliente | 233 | 1 |  | Luis Pinzón, Nimetrix |
| nx_tools | `nimetrix_presentation_unit_quantities` | cliente | 231 | 1 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_fb_wh_line` | cliente | 221 | 1 |  | Nimetrix C.A |
| nx_tools | `nimetrix_purchase_requisition_extended` | cliente | 215 | 1 |  | Nimetrix C.A |
| nx_tools | `nimetrix_product_brands` | cliente | 203 | 3 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_sitef_print` | cliente | 197 | 1 |  | Luis Pinzón, Nimetrix |
| nx_localizacion | `fiscal_lock_days` | cliente | 194 | 2 | si | Nimetrix C.A |
| nx_tools | `os_stock_picking_seq` | cliente | 187 | 2 |  | Nimetrix, Inc |
| nx_tools | `unidades_permitidas` | cliente | 159 | 3 | si | Oasis Consultora C.A |
| nx_dual_currency | `nimetrix_dual_pricelist` | cliente | 156 | 1 |  | Nimetrix C.A |
| nx_sucursales | `bi_branch_scrap_order` | tercero | 137 | 3 | si | BrowseInfo |
| nx_point_of_sale | `nimetrix_pos_res_partner` | cliente | 136 | 2 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_restrictions` | cliente | 134 | 1 | si | Nimetrix C.A |
| nx_point_of_sale | `nimetrix_pos_global_discount` | cliente | 115 | 2 |  | Nimetrix C.A |
| nx_tools | `nx_sale_uom` | ? | 88 | 1 |  |  |
| nx_tools | `nimetrix_product_force_taxes` | cliente | 81 | 1 |  | Nimetrix C.A |
| nx_tools | `pos_admin_server` | tercero | 71 | 3 |  | Odoo S.A |
| nx_dual_currency | `nimetrix_igtf_dual` | cliente | 63 | 2 |  | Nimetrix C.A |
| nx_localizacion | `nimetrix_default_invoice_date` | cliente | 63 | 2 |  | Nimetrix C.A |
| nx_point_of_sale | `nx_tpv_stock_validate` | cliente | 58 | 2 |  | Nimetrix C.A. |
