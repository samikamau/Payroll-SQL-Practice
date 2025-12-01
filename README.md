DROP TABLE IF EXISTS jan_payroll;

-- Step 1: Create payroll table
CREATE TABLE jan_payroll
(
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(255), 
    gross_amount NUMERIC(12,2),
    taxable_income NUMERIC(10,2),
    nssf NUMERIC(10,2), 
    sha NUMERIC(10,2),
    ahl NUMERIC(10,2),
    paye NUMERIC(10,2),
    net_pay NUMERIC(10,2)
);

-- Step 2: Insert employees
INSERT INTO jan_payroll(full_name, gross_amount) 
VALUES
('James Musyoka', 15000),
('Onesmus Ochieng', 24000),
('Monicah Ngure', 50000),
('Eunice Mumbi', 0),
('James Kiarie', 1000000);

-- Step 3: Calculate deductions (NSSF, SHA, AHL)
UPDATE jan_payroll
SET
    nssf = ROUND(LEAST(gross_amount*0.06, 4320),2),
    sha = ROUND(gross_amount*0.0275,2),
    ahl = ROUND(gross_amount*0.015,2);

-- Step 4: Calculate taxable_income, PAYE, and net_pay using a CTE
WITH payroll_calc AS (
    SELECT
        id,
        gross_amount,
        nssf,
        sha,
        ahl,
        (gross_amount - (nssf + sha + ahl)) AS taxable_income,
        -- PAYE calculated once
        CASE
            WHEN (gross_amount - (nssf + sha + ahl)) <= 24000 THEN ROUND(GREATEST((gross_amount - (nssf + sha + ahl))*0.10 - 2400,0),2)
            WHEN (gross_amount - (nssf + sha + ahl)) <= 32333 THEN ROUND(GREATEST(24000*0.10 + ((gross_amount - (nssf + sha + ahl))-24000)*0.25 - 2400,0),2)
            WHEN (gross_amount - (nssf + sha + ahl)) <= 500000 THEN ROUND(GREATEST(24000*0.10 + (32333-24000)*0.25 + ((gross_amount - (nssf + sha + ahl))-32333)*0.30 - 2400,0),2)
            WHEN (gross_amount - (nssf + sha + ahl)) <= 800000 THEN ROUND(GREATEST(24000*0.10 + (32333-24000)*0.25 + (500000-32333)*0.30 + ((gross_amount - (nssf + sha + ahl))-500000)*0.325 - 2400,0),2)
            ELSE ROUND(GREATEST(24000*0.10 + (32333-24000)*0.25 + (500000-32333)*0.30 + (800000-500000)*0.325 + ((gross_amount - (nssf + sha + ahl))-800000)*0.35 - 2400,0),2)
        END AS paye
    FROM jan_payroll
)
UPDATE jan_payroll AS x
SET
    taxable_income = z.taxable_income,
    paye = z.paye,
    net_pay = x.gross_amount - (x.nssf + x.sha + x.ahl + z.paye)
FROM payroll_calc AS z
WHERE x.id = z.id;

-- Step 5: View final payroll
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
