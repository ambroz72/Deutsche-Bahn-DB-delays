# Deutsche-Bahn-DB-delays
End-to-end BI project analyzing Deutsche Bahn punctuality across ~2,000 German stations. Includes data cleaning, a star-schema SQL model, and an interactive Power BI dashboard identifying worst-performing stations, lines, and delay time patterns. Built with Python, PostgreSQL, and Tableau

Question - Which stations and train lines have the worst punctuality, and when do delays spike?

-- =====================================================
-- Deutsche Bahn Punctuality Analysis
-- Star Schema DDL (SQLite)
-- =====================================================
-- Fact table: fact_delays (one row per train stop event)
-- Dimensions: dim_station, dim_line, dim_date
-- =====================================================

PRAGMA foreign_keys = ON;

-- ---------------------------------------------------
-- Dimension: Station
-- One row per unique station (~2000 stations)
-- ---------------------------------------------------
CREATE TABLE dim_station (
    station_id   INTEGER PRIMARY KEY,
    eva_nr       TEXT,
    station      TEXT NOT NULL,
    state        TEXT,
    city         TEXT,
    zip          TEXT,
    latitude     REAL,
    longitude    REAL,
    category     INTEGER   -- 1 (major hub) to 5 (small station)
);

-- ---------------------------------------------------
-- Dimension: Line
-- One row per unique train line
-- ---------------------------------------------------
CREATE TABLE dim_line (
    line_id      INTEGER PRIMARY KEY,
    line         TEXT NOT NULL,
    train_type   TEXT      -- e.g. ICE, IC, RE, RB (if derivable)
);

-- ---------------------------------------------------
-- Dimension: Date
-- One row per calendar date in the dataset (7 days)
-- ---------------------------------------------------
CREATE TABLE dim_date (
    date_id        INTEGER PRIMARY KEY,
    date           TEXT NOT NULL,     -- ISO format YYYY-MM-DD
    weekday_name   TEXT,
    is_weekend     INTEGER            -- 0 = false, 1 = true
);

-- ---------------------------------------------------
-- Fact: Delays
-- One row per train stop event (arrival/departure)
-- ---------------------------------------------------
CREATE TABLE fact_delays (
    fact_id                 INTEGER PRIMARY KEY AUTOINCREMENT,
    stop_id                 TEXT,              -- original snapshot/stop identifier
    station_id              INTEGER NOT NULL,
    line_id                 INTEGER NOT NULL,
    date_id                 INTEGER NOT NULL,
    hour                    INTEGER,
    arrival_plan            TEXT,              -- ISO datetime
    arrival_change          TEXT,
    departure_plan          TEXT,
    departure_change        TEXT,
    arrival_delay_m         INTEGER,
    departure_delay_m       INTEGER,
    arrival_delay_check     TEXT,
    departure_delay_check   TEXT,
    info                    TEXT,

    FOREIGN KEY (station_id) REFERENCES dim_station(station_id),
    FOREIGN KEY (line_id)    REFERENCES dim_line(line_id),
    FOREIGN KEY (date_id)    REFERENCES dim_date(date_id)
);

-- ---------------------------------------------------
-- Indexes for common analysis queries
-- ---------------------------------------------------
CREATE INDEX idx_fact_station ON fact_delays(station_id);
CREATE INDEX idx_fact_line ON fact_delays(line_id);
CREATE INDEX idx_fact_date ON fact_delays(date_id);
CREATE INDEX idx_fact_hour ON fact_delays(hour);
