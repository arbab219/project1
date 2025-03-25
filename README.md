# project1

-- Toll Management System Project
-- This project includes tables, procedures, triggers, and sample queries.

-- Create Tables
CREATE TABLE Vehicle (
    VehicleID INT PRIMARY KEY,
    VehicleNumber VARCHAR2(20),
    VehicleType VARCHAR2(20),
    OwnerName VARCHAR2(50)
);

CREATE TABLE TollBooth (
    BoothID INT PRIMARY KEY,
    BoothLocation VARCHAR2(100)
);

CREATE TABLE TollTransaction (
    TransactionID INT PRIMARY KEY,
    VehicleID INT,
    BoothID INT,
    TransactionDate TIMESTAMP DEFAULT SYSTIMESTAMP,
    Amount NUMBER(10,2),
    PaymentStatus VARCHAR2(20),
    FOREIGN KEY (VehicleID) REFERENCES Vehicle(VehicleID),
    FOREIGN KEY (BoothID) REFERENCES TollBooth(BoothID)
);

-- Create Procedure to Register Vehicle
CREATE OR REPLACE PROCEDURE RegisterVehicle (
    p_VehicleID IN INT,
    p_VehicleNumber IN VARCHAR2,
    p_VehicleType IN VARCHAR2,
    p_OwnerName IN VARCHAR2
) AS
BEGIN
    INSERT INTO Vehicle (VehicleID, VehicleNumber, VehicleType, OwnerName)
    VALUES (p_VehicleID, p_VehicleNumber, p_VehicleType, p_OwnerName);
    DBMS_OUTPUT.PUT_LINE('Vehicle Registered Successfully');
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/

-- Create Procedure for Toll Payment
CREATE OR REPLACE PROCEDURE MakeTollPayment (
    p_TransactionID IN INT,
    p_VehicleID IN INT,
    p_BoothID IN INT,
    p_Amount IN NUMBER,
    p_PaymentStatus IN VARCHAR2
) AS
BEGIN
    INSERT INTO TollTransaction (TransactionID, VehicleID, BoothID, Amount, PaymentStatus)
    VALUES (p_TransactionID, p_VehicleID, p_BoothID, p_Amount, p_PaymentStatus);
    DBMS_OUTPUT.PUT_LINE('Toll Payment Recorded Successfully');
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/

-- Create Trigger to Validate Payment Status
CREATE OR REPLACE TRIGGER ValidatePaymentStatus
BEFORE INSERT OR UPDATE ON TollTransaction
FOR EACH ROW
BEGIN
    IF :NEW.PaymentStatus NOT IN ('Paid', 'Pending', 'Failed') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Invalid Payment Status');
    END IF;
END;
/

-- Sample Query to Generate Report
SELECT V.VehicleNumber, V.VehicleType, T.BoothLocation, TT.TransactionDate, TT.Amount, TT.PaymentStatus
FROM TollTransaction TT
JOIN Vehicle V ON TT.VehicleID = V.VehicleID
JOIN TollBooth T ON TT.BoothID = T.BoothID;

-- Example Test Data
BEGIN
    RegisterVehicle(1, 'KA01AB1234', 'Car', 'John Doe');
    RegisterVehicle(2, 'MH05XY5678', 'Truck', 'Jane Smith');
    
    INSERT INTO TollBooth (BoothID, BoothLocation) VALUES (101, 'Delhi Expressway');
    INSERT INTO TollBooth (BoothID, BoothLocation) VALUES (102, 'Mumbai Highway');
    
    MakeTollPayment(1001, 1, 101, 50, 'Paid');
    MakeTollPayment(1002, 2, 102, 150, 'Pending');
END;
/

-- View Transactions
SELECT * FROM TollTransaction;
