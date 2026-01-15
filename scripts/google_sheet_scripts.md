/**
 * KUT v2 ENTERPRISE PLATFORM - Master Controller
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
 * STEP 1: INITIALIZE WORKBOOK
 * Creates the empty tabs with headers for all 7 silos.
 */
function INITIALIZE_V2_WORKBOOK() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const ui = SpreadsheetApp.getUi();

  const sheetsNeeded = [
    { name: "kutv2_branches", headers: [["ID", "Branch Name", "Is Prod Center"]] },
    { name: "kutv2_measuring_units", headers: [["ID", "Unit Name", "Abbr"]] },
    { name: "kutv2_ingredients", headers: [["ID", "Ingredient Name", "Unit ID", "Reorder Level"]] },
    { name: "kutv2_stock_ledger", headers: [["ID", "Branch ID", "Ing ID", "Batch", "Qty", "Last Updated"]] },
    { name: "kutv2_pos_products", headers: [["ID", "Barcode", "Product Name", "Category", "Price"]] },
    { name: "kutv2_sales_log", headers: [["ID", "Branch ID", "Prod ID", "Qty", "Price", "Time"]] },
    { name: "kutv2_recipes", headers: [["ID", "Prod ID", "Ing ID", "Qty Needed"]] }
  ];

  sheetsNeeded.forEach(s => {
    let sheet = ss.getSheetByName(s.name);
    if (!sheet) {
      sheet = ss.insertSheet(s.name);
    }
    sheet.clear();
    sheet.getRange(1, 1, 1, s.headers[0].length).setValues(s.headers)
         .setFontWeight("bold")
         .setBackground("#cfe2f3"); // Light blue professional look
    sheet.setFrozenRows(1);
  });

  // Cleanup: Delete the default "Sheet1" if it exists
  const defaultSheet = ss.getSheetByName("Sheet1");
  if (defaultSheet) ss.deleteSheet(defaultSheet);

  ui.alert("🚀 v2 Framework Initialized. Click '📥 2. Pull Substantial Data' to populate.");
}

/**
 * STEP 2: PULL SUBSTANTIAL DATA
 * Connects to MySQL and populates the sheets with the v2 data.
 */
function pullV2Data() {
  const ui = SpreadsheetApp.getUi();
  try {
    const conn = Jdbc.getConnection(DB_URL, DB_CONFIG.user, DB_CONFIG.pwd);
    
    // Mapping our sheets to their SQL queries and column counts
    const tableMapping = [
      { name: "kutv2_branches", query: "SELECT * FROM kutv2_branches", cols: 3 },
      { name: "kutv2_measuring_units", query: "SELECT * FROM kutv2_measuring_units", cols: 3 },
      { name: "kutv2_ingredients", query: "SELECT * FROM kutv2_ingredients", cols: 4 },
      { name: "kutv2_stock_ledger", query: "SELECT * FROM kutv2_stock_ledger", cols: 6 },
      { name: "kutv2_pos_products", query: "SELECT * FROM kutv2_pos_products", cols: 5 },
      { name: "kutv2_sales_log", query: "SELECT * FROM kutv2_sales_log", cols: 6 },
      { name: "kutv2_recipes", query: "SELECT * FROM kutv2_recipes", cols: 4 }
    ];

    tableMapping.forEach(map => {
      const results = conn.createStatement().executeQuery(map.query);
      writeDataToSheet(map.name, results, map.cols);
    });

    conn.close();
    ui.alert("✅ Substantial v2 Data Pulled Successfully!");
  } catch (e) {
    ui.alert("❌ Error: " + e.message);
  }
}

/**
 * HELPER: Writes result sets to sheets
 */
function writeDataToSheet(sheetName, results, colCount) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);
  if (!sheet) return;

  // Clear existing rows (keep headers)
  if (sheet.getLastRow() > 1) {
    sheet.getRange(2, 1, sheet.getLastRow() - 1, colCount).clearContent();
  }

  let rows = [];
  while (results.next()) {
    let row = [];
    for (let i = 1; i <= colCount; i++) {
      row.push(results.getString(i));
    }
    rows.push(row);
  }

  if (rows.length > 0) {
    sheet.getRange(2, 1, rows.length, colCount).setValues(rows);
  }
}

/**
 * MENU CREATOR
 */
function onOpen() {
  SpreadsheetApp.getUi().createMenu('🏢 KUT Enterprise v2')
      .addItem('⚠️ 1. Initialize Workbook', 'INITIALIZE_V2_WORKBOOK')
      .addItem('📥 2. Pull Substantial Data', 'pullV2Data')
      .addToUi();
}
