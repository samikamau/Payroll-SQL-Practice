DROP TABLE IF EXISTS jan_payroll;

-- Step 1: Create payroll table
CREATE TABLE jan_payroll
(
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(255), 
    gross_amount NUMERIC(12,2),
    nssf NUMERIC(10,2), 
    sha NUMERIC(10,2),
    ahl NUMERIC(10,2),
    taxable_income NUMERIC(12,2),
    paye NUMERIC(12,2),
    net_pay NUMERIC(12,2)
);

-- Step 2: Insert employees
INSERT INTO jan_payroll(full_name, gross_amount) 
VALUES
('James Musyoka', 15000),
('Onesmus Ochieng', 24000),
('Monicah Ngure', 50000),
('Eunice Mumbi', 72000),
('James Kiarie', 800000);

-- Step 3: Calculate deductions (NSSF, SHA, AHL)
UPDATE jan_payroll
SET
nssf = ROUND(LEAST(gross_amount*0.06,4320),2),
sha = ROUND(gross_amount*0.0275,2),
ahl= ROUND(gross_amount*0.015,2);

-- Step 4: Calculate taxable income

UPDATE jan_payroll
SET
taxable_income = gross_amount - (nssf+sha+ahl);

WITH paye_calc AS (
SELECT 
id,
taxable_income,

CASE
	WHEN taxable_income <= 24000 
		THEN ROUND(GREATEST(taxable_income * 0.10 - 2400,0),2)
    WHEN taxable_income <= 32333 
		THEN ROUND(GREATEST(24000*0.10 + 
		(taxable_income-24000)*0.25 - 2400,0),2)
    WHEN taxable_income <= 500000 
		THEN ROUND(GREATEST(24000*0.10 + 
		(32333-24000)*0.25 + 
		(taxable_income-32333)*0.30 - 2400,0),2)
    WHEN taxable_income <= 800000 
		THEN ROUND(GREATEST(24000*0.10 + 
		(32333-24000)*0.25 + 
		(500000-32333)*0.30 + 
		(taxable_income-500000)*0.325 - 2400,0),2)
    ELSE ROUND(GREATEST(24000*0.10 + 
		(32333-24000)*0.25 + 
		(500000-32333)*0.30 +
		(800000-500000)*0.325 +
		(taxable_income-800000)*0.35 - 2400,0),2)
		
	END AS paye			
	FROM jan_payroll
)

UPDATE jan_payroll AS x

SET paye = z.paye
 FROM 
 paye_calc AS z
 WHERE
 x.id = z.id;

-- Step 6: Calculate net pay
UPDATE jan_payroll
SET net_pay = gross_amount - (nssf + sha + ahl + paye);

-- Step 7: View final payroll
SELECT 
    full_name,
    gross_amount,
    nssf,
    sha,
    ahl,
    taxable_income,
    paye,
    net_pay
FROM jan_payroll;
