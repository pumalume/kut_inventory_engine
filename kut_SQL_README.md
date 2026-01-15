SET FOREIGN_KEY_CHECKS = 0;

-- 1. CLEAN SLATE V3
DROP TABLE IF EXISTS kutv3_payments;
DROP TABLE IF EXISTS kutv3_invoices;
DROP TABLE IF EXISTS kutv3_customers;
DROP TABLE IF EXISTS kutv3_recipes;
DROP TABLE IF EXISTS kutv3_sales_log;
DROP TABLE IF EXISTS kutv3_stock_ledger;
DROP TABLE IF EXISTS kutv3_pos_products;
DROP TABLE IF EXISTS kutv3_ingredients;
DROP TABLE IF EXISTS kutv3_measuring_units;
DROP TABLE IF EXISTS kutv3_branches;

-- 2. THE FOUNDATION
CREATE TABLE kutv3_branches (
    branch_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_name VARCHAR(100),
    is_production_center BOOLEAN
) ENGINE=InnoDB;

CREATE TABLE kutv3_measuring_units (
    unit_id INT AUTO_INCREMENT PRIMARY KEY,
    unit_name VARCHAR(50), 
    unit_abbr VARCHAR(10)
) ENGINE=InnoDB;

-- 3. THE INVENTORY SILO
CREATE TABLE kutv3_ingredients (
    ingredient_id INT AUTO_INCREMENT PRIMARY KEY,
    ingredient_name VARCHAR(255),
    unit_id INT,
    reorder_level DECIMAL(15,4),
    FOREIGN KEY (unit_id) REFERENCES kutv3_measuring_units(unit_id)
) ENGINE=InnoDB;

CREATE TABLE kutv3_stock_ledger (
    stock_id INT AUTO_INCREMENT PRIMARY KEY,
    branch_id INT,
    ingredient_id INT,
    quantity_change DECIMAL(15,4), -- Positive for arrival, Negative for production/waste
    reason ENUM('PURCHASE', 'PRODUCTION', 'WASTE', 'TRANSFER'),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (branch_id) REFERENCES kutv3_branches(branch_id),
    FOREIGN KEY (ingredient_id) REFERENCES kutv3_ingredients(ingredient_id)
) ENGINE=InnoDB;

-- 4. THE POS & RECIPE SILO (The "Brain")
CREATE TABLE kutv3_pos_products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    barcode VARCHAR(100),
    product_name VARCHAR(255),
    category VARCHAR(100),
    unit_price DECIMAL(10,2)
) ENGINE=InnoDB;

CREATE TABLE kutv3_recipes (
    recipe_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT,
    ingredient_id INT,
    qty_required DECIMAL(15,4), -- e.g., 0.250kg of Pistachio for 1 unit of Baklava
    FOREIGN KEY (product_id) REFERENCES kutv3_pos_products(product_id),
    FOREIGN KEY (ingredient_id) REFERENCES kutv3_ingredients(ingredient_id)
) ENGINE=InnoDB;

-- 5. THE BILLING SILO (New for v3)
CREATE TABLE kutv3_customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(255),
    phone VARCHAR(50),
    balance_limit DECIMAL(15,2) DEFAULT 5000.00
) ENGINE=InnoDB;

CREATE TABLE kutv3_invoices (
    invoice_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(15,2),
    invoice_date DATE,
    is_paid BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES kutv3_customers(customer_id)
) ENGINE=InnoDB;

CREATE TABLE kutv3_payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    amount_paid DECIMAL(15,2),
    payment_date DATE,
    FOREIGN KEY (customer_id) REFERENCES kutv3_customers(customer_id)
) ENGINE=InnoDB;

SET FOREIGN_KEY_CHECKS = 1;



SET FOREIGN_KEY_CHECKS = 0;

-- 1. BRANCHES & UNITS
INSERT INTO kutv3_branches (branch_id, branch_name, is_production_center) VALUES 
(1, 'Ana Mutfak (HQ)', TRUE), (2, 'Çarşı Şube', FALSE), (3, 'Sahil Büfe', FALSE);

INSERT INTO kutv3_measuring_units (unit_id, unit_name, unit_abbr) VALUES 
(1, 'Kilogram', 'kg'), (2, 'Litre', 'lt'), (3, 'Adet', 'adt');

-- 2. INGREDIENTS (The Raw Silo)
INSERT INTO kutv3_ingredients (ingredient_id, ingredient_name, unit_id, reorder_level) VALUES 
(1, 'Antep Fıstığı', 1, 50.0), (2, 'Tam Yağlı Süt', 2, 200.0), 
(3, 'Kristal Şeker', 1, 100.0), (4, 'Un', 1, 200.0);

-- 3. PRODUCTS & RECIPE BRIDGE (The "Brain" Silo)
INSERT INTO kutv3_pos_products (product_id, barcode, product_name, category, unit_price) VALUES 
(1, 'BRC-001', 'Fıstıklı Baklava (1kg)', 'Baklava', 750.00),
(2, 'BRC-002', 'Maraş Dondurma (1kg)', 'Dondurma', 400.00);

-- RECIPES: 1kg Baklava uses 0.3kg Pistachio, 0.4kg Sugar, 0.5kg Flour
INSERT INTO kutv3_recipes (product_id, ingredient_id, qty_required) VALUES 
(1, 1, 0.300), (1, 3, 0.400), (1, 4, 0.500),
-- 1kg Ice Cream uses 0.8L Milk, 0.2kg Sugar
(2, 2, 0.800), (2, 3, 0.200);

-- 4. STOCK MOVEMENTS (The Ledger Silo)
INSERT INTO kutv3_stock_ledger (branch_id, ingredient_id, quantity_change, reason) VALUES 
(1, 1, 200.0, 'PURCHASE'), -- HQ buys 200kg Pistachio
(1, 2, 500.0, 'PURCHASE'), -- HQ buys 500L Milk
(1, 1, -20.0, 'TRANSFER'), -- HQ sends 20kg to Branch 2
(2, 1, 20.0, 'TRANSFER'),  -- Branch 2 receives 20kg
(1, 4, 15.0, 'WASTE');      -- HQ loses 15kg Flour to spoilage

-- 5. CUSTOMERS & BILLING (The Money Silo)
INSERT INTO kutv3_customers (customer_id, customer_name, phone, balance_limit) VALUES 
(1, 'Hilton Otel', '555-001', 10000.00),   -- Large B2B
(2, 'Kordon Kafe', '555-002', 3000.00),    -- Medium B2B
(3, 'Yeni Pastane', '555-003', 1000.00);   -- New Trial

-- INVOICES (The Charges)
INSERT INTO kutv3_invoices (customer_id, amount, invoice_date, is_paid) VALUES 
(1, 4500.00, '2026-01-10', 0), (1, 3200.00, '2026-01-12', 0), -- Hilton owes 7700
(2, 1500.00, '2026-01-11', 0); -- Kordon owes 1500

-- PAYMENTS (The Cash)
INSERT INTO kutv3_payments (customer_id, amount_paid, payment_date) VALUES 
(1, 5000.00, '2026-01-14'); -- Hilton paid 5000 (Balance remaining: 2700)

SET FOREIGN_KEY_CHECKS = 1;
