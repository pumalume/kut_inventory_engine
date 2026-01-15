-- -----------------------------------------------------
-- 1. MEASURING UNITS (The foundation for all quantities)
-- -----------------------------------------------------
CREATE TABLE kut_measuring_units (
    unit_id INT AUTO_INCREMENT PRIMARY KEY,
    unit_name VARCHAR(50) UNIQUE NOT NULL, 
    unit_abbr VARCHAR(10)
) ENGINE=InnoDB;

-- Initial Turkish Seed Data
INSERT INTO kut_measuring_units (unit_name, unit_abbr) VALUES 
('Adet', 'adt'),    -- Piece
('Koli', 'kl'),     -- Case/Box
('Paket', 'pkt'),   -- Pack
('Kilogram', 'kg'), -- Kilogram
('Gram', 'g'),      -- Gram
('Litre', 'lt'),    -- Liter
('Mililitre', 'ml'),-- Milliliter
('Bağ', 'bağ');     -- Bunch

-- -----------------------------------------------------
-- 2. INGREDIENTS (Master Inventory List)
-- -----------------------------------------------------
CREATE TABLE kut_ingredients (
    ingredient_id INT AUTO_INCREMENT PRIMARY KEY,
    ingredient_name VARCHAR(255) UNIQUE NOT NULL,
    current_stock_qty DECIMAL(15,4) DEFAULT 0.0000,
    unit_id INT,
    FOREIGN KEY (unit_id) REFERENCES kut_measuring_units(unit_id) ON UPDATE CASCADE
) ENGINE=InnoDB;

-- -----------------------------------------------------
-- 3. RECIPES (Links your kut_products to ingredients)
-- -----------------------------------------------------
CREATE TABLE kut_recipes (
    recipe_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(255) NOT NULL, -- Ties to your existing kut_products table
    ingredient_id INT,
    required_qty DECIMAL(15,4) NOT NULL,
    unit_id INT,
    FOREIGN KEY (ingredient_id) REFERENCES kut_ingredients(ingredient_id) ON DELETE CASCADE,
    FOREIGN KEY (unit_id) REFERENCES kut_measuring_units(unit_id) ON UPDATE CASCADE
) ENGINE=InnoDB;

-- -----------------------------------------------------
-- 4. INVENTORY ORDERS (The "Red Flag" Header)
-- -----------------------------------------------------
CREATE TABLE kut_inventory_orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(100) UNIQUE NOT NULL,
    transaction_type ENUM('GIRIS', 'CIKIS', 'DUZELTME') NOT NULL, -- Entry, Exit, Adjustment
    order_status ENUM('BEKLIYOR', 'ONAYLANDI') DEFAULT 'BEKLIYOR', -- Pending vs Acknowledged
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    confirmed_at TIMESTAMP NULL
) ENGINE=InnoDB;

-- -----------------------------------------------------
-- 5. ORDER ITEMS (The specific changes per order)
-- -----------------------------------------------------
CREATE TABLE kut_order_items (
    item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    ingredient_id INT,
    quantity DECIMAL(15,4) NOT NULL,
    unit_id INT,
    FOREIGN KEY (order_id) REFERENCES kut_inventory_orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (ingredient_id) REFERENCES kut_ingredients(ingredient_id) ON UPDATE CASCADE,
    FOREIGN KEY (unit_id) REFERENCES kut_measuring_units(unit_id) ON UPDATE CASCADE
) ENGINE=InnoDB;


-- Replace your old kut_products with this updated version
DROP TABLE IF EXISTS kut_products;
CREATE TABLE kut_products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(255) UNIQUE NOT NULL,
    category VARCHAR(100), -- e.g., 'Dondurma', 'Pasta'
    unit_price DECIMAL(10,2) DEFAULT 0.00,
    is_active BOOLEAN DEFAULT TRUE
) ENGINE=InnoDB;


-- The "Identity" (Master Data)
CREATE TABLE kut_ingredients (
    ingredient_id INT AUTO_INCREMENT PRIMARY KEY,
    ingredient_name VARCHAR(255) UNIQUE NOT NULL,
    unit_id INT,
    reorder_level DECIMAL(15,4), -- When to trigger a warning
    FOREIGN KEY (unit_id) REFERENCES kut_measuring_units(unit_id)
);

-- The "State" (Inventory Ledger)
CREATE TABLE kut_stock_ledger (
    stock_id INT AUTO_INCREMENT PRIMARY KEY,
    ingredient_id INT,
    batch_number VARCHAR(50),
    expiry_date DATE,
    quantity_on_hand DECIMAL(15,4),
    warehouse_location VARCHAR(100),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (ingredient_id) REFERENCES kut_ingredients(ingredient_id)
);


-- 1. CLEAN UP (Optional: removes previous test data to avoid ID conflicts)
SET FOREIGN_KEY_CHECKS = 0;
TRUNCATE TABLE kut_stock_ledger;
TRUNCATE TABLE kut_order_items;
TRUNCATE TABLE kut_inventory_orders;
TRUNCATE TABLE kut_ingredients;
TRUNCATE TABLE kut_measuring_units;
SET FOREIGN_KEY_CHECKS = 1;

-- 2. MEASURING UNITS
INSERT INTO kut_measuring_units (unit_id, unit_name, unit_abbr) VALUES 
(1, 'Kilogram', 'kg'),
(2, 'Litre', 'L'),
(3, 'Adet', 'pcs'),
(4, 'Gram', 'g');

-- 3. INGREDIENTS (The Master List)
-- Setting reorder levels to see 'LOW STOCK' logic in action
INSERT INTO kut_ingredients (ingredient_id, ingredient_name, unit_id, reorder_level) VALUES 
(1, 'Antep Fıstığı', 1, 20.00),
(2, 'Tam Yağlı Süt', 2, 100.00),
(3, 'Sahlep', 1, 5.00),
(4, 'Kristal Şeker', 1, 50.00),
(5, 'Vanilya Çubuğu', 3, 10.00);

-- 4. INVENTORY ORDERS (Transactional Headers)
-- One is pending (Red Flag), one is already confirmed (Green)
INSERT INTO kut_inventory_orders (order_id, order_number, transaction_type, order_status) VALUES 
(101, 'ORD-2026-001', 'GIRIS', 'ONAYLANDI'),
(102, 'ORD-2026-002', 'GIRIS', 'BEKLIYOR');

-- 5. ORDER ITEMS (The specific things we ordered)
INSERT INTO kut_order_items (order_id, ingredient_id, quantity) VALUES 
(101, 1, 25.00), -- 25kg Pistachio (Confirmed)
(101, 2, 120.00),-- 120L Milk (Confirmed)
(102, 3, 2.00),  -- 2kg Salep (Pending - Not in stock yet!)
(102, 4, 10.00); -- 10kg Sugar (Pending)

-- 6. STOCK LEDGER (The Physical Reality)
-- We only populate this with the "Confirmed" items from Order 101
INSERT INTO kut_stock_ledger (ingredient_id, batch_number, quantity_on_hand, warehouse_location, expiry_date) VALUES 
(1, 'BATCH-PIS-01', 25.00, 'Kuru Depo', '2026-12-01'),
(2, 'BATCH-MILK-A', 60.00, 'Soğuk Hava', '2026-02-15'),
(2, 'BATCH-MILK-B', 60.00, 'Soğuk Hava', '2026-02-20'),
(4, 'OLD-SUGAR', 5.00, 'Kuru Depo', '2027-01-01');
