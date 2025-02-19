# goit-rdb-fp


## 1  ----------------------------------------------------------

create schema pandemic;

use pandemic;

SELECT * FROM pandemic.infectious_cases;


## 2  ----------------------------------------------------------

CREATE TABLE entities (
    id INT PRIMARY KEY auto_increment,
    entity VARCHAR(45) UNIQUE NOT NULL,
    code VARCHAR(8) UNIQUE NOT NULL
);

CREATE TABLE infectious (
    id INT PRIMARY KEY auto_increment,
    entity_id INT REFERENCES entities(id) ON DELETE CASCADE,
    year YEAR NOT NULL,
    number_yaws INT,
    polio_cases INT,
    cases_guinea_worm INT,
    number_rabies FLOAT,
    number_malaria FLOAT,
    number_hiv FLOAT,
    number_tuberculosis FLOAT,
    number_smallpox FLOAT,
    number_cholera_cases INT,
    UNIQUE (entity_id, year)
);

INSERT INTO entities (entity, code)
SELECT DISTINCT Entity, Code FROM infectious_cases;

INSERT INTO infectious (
    entity_id, year, number_yaws, polio_cases, cases_guinea_worm, number_rabies, 
    number_malaria, number_hiv, number_tuberculosis, number_smallpox, number_cholera_cases
)
SELECT e.id, ic.Year, NULLIF(ic.Number_yaws, ''), 
       NULLIF(ic.polio_cases, ''), 
       NULLIF(ic.cases_guinea_worm, ''), 
       NULLIF(ic.Number_rabies, ''), 
       NULLIF(ic.Number_malaria, ''), 
       NULLIF(ic.Number_hiv, ''), 
       NULLIF(ic.Number_tuberculosis, ''), 
       NULLIF(ic.Number_smallpox, ''), 
       NULLIF(ic.Number_cholera_cases, '')
FROM infectious_cases ic
JOIN entities e ON ic.Entity = e.entity AND ic.Code = e.code;


## 3  ------------------------------------------------------------

SELECT e.entity, 
	e.code, 
    MIN(i.number_rabies) as min_number_rabies, 
    MAX(i.number_rabies) as max_number_rabies, 
    AVG(i.number_rabies) as average_number_rabies, 
    SUM(i.number_rabies) as total_number_rabies
FROM infectious i
JOIN entities e ON i.entity_id = e.id
WHERE i.number_rabies IS NOT NULL AND i.number_rabies <> ''
GROUP BY e.entity, e.code
order by average_number_rabies DESC
limit 10;


## 4 --------------------------------------------------------------

SELECT id, year, 
       DATE_FORMAT(CONCAT(year, '-01-01'), '%Y-%m-%d') AS start_date,
       date(now()) AS now,
       year(now()) - year AS years_pass  
FROM infectious;


## 5  -------------------------------------------------------------

DROP FUNCTION IF EXISTS YearsPass;

DELIMITER //
CREATE FUNCTION YearsPass(start_year YEAR)
RETURNS INT
NO SQL
BEGIN
    DECLARE result INT;
    SET result = (year(now()) - start_year);
    RETURN result;
END //
DELIMITER ;

SELECT id, year, YearsPass(year) as years_pass
FROM infectious;


## or  ------------------------------------------------------------

DROP FUNCTION IF EXISTS YearsPass;

DELIMITER //
CREATE FUNCTION YearsPass(start_date CHAR(10))
RETURNS INT
NO SQL
BEGIN
    DECLARE result INT;
    SET result = year(now()) - YEAR(DATE(start_date));
    RETURN result;
END //
DELIMITER ;

WITH TEMPTABLE AS (SELECT id, year, DATE_FORMAT(CONCAT(year, '-01-01'), '%Y-%m-%d') AS start_date FROM infectious)
SELECT id, start_date, YearsPass(start_date) as years_pass FROM TEMPTABLE;
