/** * RE-ENGINEERED: Pulls Ingredients and Current Stock from DB 
 * Useful for the "Current Inventory" tab in Google Sheets
 */
function pullInventoryFromDB() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName("kut_inventory_status") || ss.insertSheet("kut_inventory_status");
  
  try {
    var conn = Jdbc.getConnection(url, user, pwd);
    var sql = "SELECT i.ingredient_name, i.current_stock_qty, u.unit_name " +
              "FROM kut_ingredients i JOIN kut_measuring_units u ON i.unit_id = u.unit_id";
    var results = conn.createStatement().executeQuery(sql);
    
    sheet.clear();
    sheet.appendRow(["Ingredient Name", "Stock Quantity", "Unit"]);
    
    var dataRows = [];
    while (results.next()) {
      dataRows.push([results.getString(1), results.getDouble(2), results.getString(3)]);
    }
    if (dataRows.length > 0) sheet.getRange(2, 1, dataRows.length, 3).setValues(dataRows);
    conn.close();
  } catch (e) {
    Logger.log("Inventory Pull Error: " + e.toString());
  }
}

/** * RE-ENGINEERED: Process "Acknowledged" Orders 
 * This moves items from 'BEKLIYOR' (Red) to 'ONAYLANDI' (Green) and updates stock.
 */
function acknowledgePendingOrders() {
  try {
    var conn = Jdbc.getConnection(url, user, pwd);
    conn.setAutoCommit(false);

    // 1. Get all pending orders
    var stmt = conn.createStatement();
    var pendingOrders = stmt.executeQuery("SELECT order_id FROM kut_inventory_orders WHERE order_status = 'BEKLIYOR'");
    
    var orderIds = [];
    while (pendingOrders.next()) { orderIds.push(pendingOrders.getInt(1)); }

    for (var i = 0; i < orderIds.length; i++) {
      var id = orderIds[i];
      
      // Update Stock (Addition or Subtraction based on your logic)
      // For 'GIRIS' we add, for 'CIKIS' we subtract
      var updateStockSql = "UPDATE kut_ingredients i " +
                           "JOIN kut_order_items oi ON i.ingredient_id = oi.ingredient_id " +
                           "JOIN kut_inventory_orders o ON oi.order_id = o.order_id " +
                           "SET i.current_stock_qty = CASE " +
                           "  WHEN o.transaction_type = 'GIRIS' THEN i.current_stock_qty + oi.quantity " +
                           "  WHEN o.transaction_type = 'CIKIS' THEN i.current_stock_qty - oi.quantity " +
                           "  ELSE i.current_stock_qty END " +
                           "WHERE o.order_id = ?";
      
      var stockStmt = conn.prepareStatement(updateStockSql);
      stockStmt.setInt(1, id);
      stockStmt.executeUpdate();

      // Set Status to ONAYLANDI
      var statusStmt = conn.prepareStatement("UPDATE kut_inventory_orders SET order_status = 'ONAYLANDI', confirmed_at = NOW() WHERE order_id = ?");
      statusStmt.setInt(1, id);
      statusStmt.executeUpdate();
    }

    conn.commit();
    conn.close();
    SpreadsheetApp.getUi().alert("Processed " + orderIds.length + " pending orders into stock.");
  } catch (e) {
    SpreadsheetApp.getUi().alert("Acknowledgment Error: " + e.toString());
  }
}
