/**
 * KTD ENTERPRISE PLATFORM - Master Controller
 * Namespace: ktd_
 */

const DB_CONFIG = {
  host: "srv990.hstgr.io",
  name: "u214463888_iqIB3",
  user: "u214463888_GOVnK",
  pwd: "Zv3UcW5koC", 
  port: 3306
};

const DB_URL = `jdbc:mysql://${DB_CONFIG.host}:${DB_CONFIG.port}/${DB_CONFIG.name}`;

/**
 * 1. INITIALIZE KTD WORKBOOK
 * Creates the tabs with headers for all 11 enterprise tables.
 */
function INITIALIZE_KTD_WORKBOOK() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const ui = SpreadsheetApp.getUi();

  const sheetsNeeded = [
    { name: "ktd_branches", headers: [["branch_id", "branch_name", "is_production_center"]] },
    { name: "ktd_measuring_units", headers: [["unit_id", "unit_name", "unit_abbr"]] },
    { name: "ktd_ingredients", headers: [["ingredient_id", "ingredient_name", "unit_id", "reorder_level", "current_cost"]] },
    { name: "ktd_stock_ledger", headers: [["stock_id", "branch_id", "ingredient_id", "quantity_change", "reason", "ai_context", "timestamp"]] },
    { name: "ktd_pos_products", headers: [["product_id", "barcode", "product_name", "category", "unit_price"]] },
    { name: "ktd_recipes", headers: [["recipe_id", "product_id", "ingredient_id", "qty_required"]] },
    { name: "ktd_production_log", headers: [["production_id", "branch_id", "product_id", "batch_quantity", "status", "timestamp"]] },
    { name: "ktd_customers", headers: [["customer_id", "customer_name", "phone", "balance_limit"]] },
    { name: "ktd_sales_log", headers: [["sale_id", "branch_id", "product_id", "quantity_sold", "total_price", "customer_id", "timestamp"]] },
    { name: "ktd_invoices", headers: [["invoice_id", "customer_id", "sale_id", "amount", "invoice_date", "is_paid"]] },
    { name: "ktd_payments", headers: [["payment_id", "customer_id", "amount_paid", "payment_date", "payment_method"]] }
  ];

  sheetsNeeded.forEach(s => {
    let sheet = ss.getSheetByName(s.name);
    if (!sheet) { sheet = ss.insertSheet(s.name); }
    sheet.clear();
    sheet.getRange(1, 1, 1, s.headers[0].length).setValues(s.headers)
         .setFontWeight("bold")
         .setBackground("#d9ead3"); // Professional green for KTD
    sheet.setFrozenRows(1);
  });

  const defaultSheet = ss.getSheetByName("Sheet1");
  if (defaultSheet) ss.deleteSheet(defaultSheet);

  ui.alert("🚀 KTD Enterprise Framework Initialized!");
}

/**
 * 2. PULL KTD DATA
 * Automatically detects column counts and pulls data from MySQL.
 */
function pullKtdData() {
  const ui = SpreadsheetApp.getUi();
  try {
    const conn = Jdbc.getConnection(DB_URL, DB_CONFIG.user, DB_CONFIG.pwd);
    
    const tables = [
      "ktd_branches", "ktd_measuring_units", "ktd_ingredients", 
      "ktd_stock_ledger", "ktd_pos_products", "ktd_recipes", 
      "ktd_production_log", "ktd_customers", "ktd_sales_log", 
      "ktd_invoices", "ktd_payments"
    ];

    tables.forEach(tableName => {
      const results = conn.createStatement().executeQuery(`SELECT * FROM ${tableName}`);
      const metaData = results.getMetaData();
      const colCount = metaData.getColumnCount(); // Dynamic column counting 🤖
      
      writeDataToSheet(tableName, results, colCount);
    });

    conn.close();
    ui.alert("✅ KTD Data Sync Complete!");
  } catch (e) {
    ui.alert("❌ Sync Error: " + e.message);
  }
}

/**
 * HELPER: Writes data rows
 */
function writeDataToSheet(sheetName, results, colCount) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);
  if (!sheet) return;

  if (sheet.getLastRow() > 1) {
    sheet.getRange(2, 1, sheet.getLastRow() - 1, colCount).clearContent();
  }

  let rows = [];
  while (results.next()) {
    let row = [];
    for (let i = 1; i <= colCount; i++) { row.push(results.getString(i)); }
    rows.push(row);
  }

  if (rows.length > 0) {
    sheet.getRange(2, 1, rows.length, colCount).setValues(rows);
  }
}

/**
 * MENU
 */
function onOpen() {
  SpreadsheetApp.getUi().createMenu('🏢 KTD Enterprise Platform')
      .addItem('⚠️ 1. Initialize Workbook', 'INITIALIZE_KTD_WORKBOOK')
      .addItem('📥 2. Pull Data from Server', 'pullKtdData')
      .addToUi();
}
