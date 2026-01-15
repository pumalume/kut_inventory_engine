SET FOREIGN_KEY_CHECKS = 0;
TRUNCATE TABLE kutv2_sales_log;
TRUNCATE TABLE kutv2_stock_ledger;
TRUNCATE TABLE kutv2_recipes;
TRUNCATE TABLE kutv2_pos_products;
TRUNCATE TABLE kutv2_ingredients;
TRUNCATE TABLE kutv2_measuring_units;
TRUNCATE TABLE kutv2_branches;
SET FOREIGN_KEY_CHECKS = 1;

-- 1. BRANCHES (The Network)
INSERT INTO kutv2_branches (branch_id, branch_name, is_production_center) VALUES 
(1, 'Ana Mutfak (Main Kitchen)', TRUE), 
(2, 'Çarşı Kafe (Downtown)', FALSE),
(3, 'Sahil Büfe (Beach Front)', FALSE);

-- 2. MEASURING UNITS
INSERT INTO kutv2_measuring_units (unit_id, unit_name, unit_abbr) VALUES 
(1, 'Kilogram', 'kg'), (2, 'Litre', 'lt'), (3, 'Adet', 'adt'), (4, 'Gram', 'g');

-- 3. INVENTORY SILO (Substantial Ingredients list)
INSERT INTO kutv2_ingredients (ingredient_id, ingredient_name, unit_id, reorder_level) VALUES 
(1, 'Antep Fıstığı (Boz)', 1, 40.00),
(2, 'Tam Yağlı Süt', 2, 500.00),
(3, 'Kristal Şeker', 1, 200.00),
(4, 'Un (Baklavalık)', 1, 300.00),
(5, 'Sade Yağ', 1, 50.00),
(6, 'Çikolata Pul', 1, 20.00),
(7, 'Salep (Saf)', 1, 5.00);

-- 4. STOCK LEDGER (Simulating actual warehouse levels)
INSERT INTO kutv2_stock_ledger (branch_id, ingredient_id, batch_number, quantity_on_hand) VALUES 
(1, 1, 'BATCH-PIS-001', 55.00),  -- 55kg Pistachio (Safe)
(1, 2, 'MILK-JAN-15', 120.00),   -- 120L Milk (CRITICAL: Below 500L)
(1, 3, 'SUGAR-T-1', 450.00),     -- 450kg Sugar (Safe)
(1, 4, 'FLOUR-BRD', 25.00),      -- 25kg Flour (CRITICAL: Below 300kg)
(1, 7, 'SALEP-09', 1.50);        -- 1.5kg Salep (LOW STOCK)

-- 5. POS SILO (Diverse Product Catalog)
INSERT INTO kutv2_pos_products (product_id, barcode, product_name, category, default_price) VALUES 
(1, '869001', 'Fıstıklı Baklava (Lüks)', 'Baklava', 650.00),
(2, '869002', 'Dondurma (Külah)', 'Dondurma', 45.00),
(3, '869003', 'Çikolatalı Pasta (No.1)', 'Pasta', 550.00),
(4, '869004', 'Sade Tulumba (Koli)', 'Tatlı', 180.00),
(5, '869005', 'Fıstıklı Sarma', 'Baklava', 850.00),
(6, '869006', 'Dondurma (500g Paket)', 'Dondurma', 220.00);

-- 6. SALES LOG (Substantial mock sales across branches)
INSERT INTO kutv2_sales_log (branch_id, product_id, qty_sold, final_price) VALUES 
(2, 1, 12, 650.00), (2, 3, 4, 550.00), (2, 4, 25, 180.00), -- Downtown Sales
(3, 2, 145, 45.00), (3, 6, 30, 220.00), (3, 5, 8, 850.00); -- Beach Front (High Ice Cream Sales)

-- 7. THE FUTURE BRIDGE (Recipes for later reference)
INSERT INTO kutv2_recipes (product_id, ingredient_id, required_qty) VALUES 
(1, 1, 0.300), (1, 3, 0.400), (1, 4, 0.500), -- Baklava Recipe
(2, 2, 0.150), (2, 7, 0.005);                -- Ice Cream Scoop Recipe
