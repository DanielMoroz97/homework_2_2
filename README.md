# homework_2_2

CREATE MATERIALIZED VIEW book.bus_occupancy AS
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    GROUP BY s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    GROUP BY t.fkride
)
SELECT  
    r.id as ride_id, 
    r.startdate as depart_date, 
    bs.city || ', ' || bs.name as busstation,  
    st.all_place - coalesce(t.order_place, 0) as svobodno,  
    coalesce(t.order_place, 0) as busy_places            
FROM book.ride r
JOIN book.schedule s ON r.fkschedule = s.id
JOIN book.busroute br ON s.fkroute = br.id
JOIN book.busstation bs ON br.fkbusstationfrom = bs.id
LEFT JOIN order_place t ON t.fkride = r.id
JOIN all_place st ON r.fkbus = st.fkbus
;


CREATE OR REPLACE FUNCTION book.refresh_bus_occupancy()
RETURNS trigger AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY book.bus_occupancy;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_refresh_bus_occupancy
AFTER INSERT OR UPDATE OR DELETE ON book.tickets
FOR EACH STATEMENT
EXECUTE FUNCTION book.refresh_bus_occupancy();

    
ALTER TABLE book.tickets DISABLE TRIGGER trg_refresh_bus_occupancy;

INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40000, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40001, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40002, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40003, 1,'Moroz Danila', '{"phone": "+7905111111"}');

ALTER TABLE book.tickets ENABLE TRIGGER trg_refresh_bus_occupancy;

INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40004, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40005, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40006, 1,'Moroz Danila', '{"phone": "+7905111111"}');
INSERT INTO book.tickets (fkride, fkseat, fio,contact) VALUES (40007, 1,'Moroz Danila', '{"phone": "+7905111111"}');

Вставка без тригера 
![image](https://github.com/user-attachments/assets/a79e361f-6c82-44a8-9401-b539762a068e)
Вставка с стригером
![image](https://github.com/user-attachments/assets/857188ee-8674-4193-99db-40ec489e6bb8)

