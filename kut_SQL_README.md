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
