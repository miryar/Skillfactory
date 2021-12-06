Задание 4.1

SELECT a.city,
       count(a.*)
FROM dst_project.airports AS a
GROUP BY city
ORDER BY 2 DESC
LIMIT 2

Задание 4.2
Вопрос 1. Таблица рейсов содержит всю информацию о прошлых, текущих и 
запланированных рейсах. Сколько всего статусов для рейсов определено в таблице?

SELECT COUNT(DISTINCT f.status)
FROM dst_project.flights AS f

Вопрос 2. Какое количество самолетов находятся в воздухе на момент среза в базе
(статус рейса «самолёт уже вылетел и находится в воздухе»).

SELECT COUNT(f.*)
FROM dst_project.flights AS f
WHERE f.status = 'Departed'

Вопрос 3. Места определяют схему салона каждой модели. Сколько мест имеет самолет
модели  (Boeing 777-300)?

SELECT COUNT(s.*)
FROM dst_project.seats AS s
WHERE s.aircraft_code = '773'

Вопрос 4. Сколько состоявшихся (фактических) рейсов было совершено между 1 апреля
2017 года и 1 сентября 2017 года?

SELECT COUNT(f.*)
FROM dst_project.flights AS f
WHERE f.scheduled_arrival BETWEEN '2017-04-01' AND '2017-09-01'
  AND f.status = 'Arrived'

Задание 4.3
Вопрос 1. Сколько всего рейсов было отменено по данным базы?

SELECT COUNT(f.*)
FROM dst_project.flights AS f
WHERE f.status = 'Cancelled'

Вопрос 2. Сколько самолетов моделей типа Boeing, Sukhoi Superjet, Airbus
 находится в базе авиаперевозок?

SELECT 'Boeing' AS model_name,
       COUNT(a.model)
FROM dst_project.aircrafts AS a
WHERE a.model LIKE '%Boeing%'
UNION
SELECT 'Sukhoi Superjet' AS model_name,
       COUNT(a.model)
FROM dst_project.aircrafts AS a
WHERE a. model LIKE '%Sukhoi Superjet%'
UNION
SELECT 'Airbus' AS model_name,
       COUNT(a.model)
FROM dst_project.aircrafts AS a
WHERE a. model LIKE '%Airbus%'

Вопрос 3. В какой части (частях) света находится больше аэропортов?

SELECT 'Asia',
       COUNT(a.*)
FROM dst_project.airports AS a
WHERE a.timezone LIKE '%Asia%'
UNION
SELECT 'Europe',
       COUNT(a.*)
FROM dst_project.airports AS a
WHERE a.timezone LIKE '%Europe%'
UNION
SELECT 'Australia',
       COUNT(a.*)
FROM dst_project.airports AS a
WHERE a.timezone LIKE '%Australia%'

Вопрос 4. У какого рейса была самая большая задержка прибытия за все время сбора данных?
Введите id рейса (flight_id).

SELECT f.flight_id,
       f.actual_arrival-f.scheduled_arrival
FROM dst_project.flights AS f
WHERE f.actual_arrival IS NOT NULL
ORDER BY 2 DESC
LIMIT 1

Задание 4.4
Вопрос 1. Когда был запланирован самый первый вылет, сохраненный в базе данных?

SELECT MIN(f.scheduled_departure)
FROM dst_project.flights AS f

Вопрос 2. Сколько минут составляет запланированное время полета в самом длительном рейсе?

SELECT MAX(DATE_PART('hour', f.scheduled_arrival-f.scheduled_departure)*60+
DATE_PART('minute', f.scheduled_arrival-f.scheduled_departure))
FROM dst_project.flights AS f

Вопрос 3. Между какими аэропортами пролегает самый длительный по времени запланированный рейс?

SELECT f.departure_airport,
       f.arrival_airport,
       f.scheduled_arrival-f.scheduled_departure
FROM dst_project.flights AS f
ORDER BY 3 DESC
LIMIT 1

Вопрос 4. Сколько составляет средняя дальность полета среди всех самолетов в минутах?
 Секунды округляются в меньшую сторону (отбрасываются до минут).

SELECT AVG(DATE_PART('hour', f.scheduled_arrival-f.scheduled_departure)*60+
DATE_PART('minute', f.scheduled_arrival-f.scheduled_departure))
FROM dst_project.flights AS f

Задание 4.5
Вопрос 1. Мест какого класса у SU9 больше всего?

SELECT s.fare_conditions,
       COUNT(s.*)
FROM dst_project.seats AS s
WHERE s.aircraft_code = 'SU9'
GROUP BY s.fare_conditions
ORDER BY 2 DESC
LIMIT 1

Вопрос 2. Какую самую минимальную стоимость составило бронирование за всю историю?

SELECT min(b.total_amount)
FROM dst_project.bookings AS b

Вопрос 3. Какой номер места был у пассажира с id = 4313 788533?

SELECT b.seat_no
FROM dst_project.tickets AS t
JOIN dst_project.boarding_passes AS b ON t.ticket_no=b.ticket_no
WHERE t.passenger_id = '4313 788533'

Задание 5.1
Вопрос 1. Анапа — курортный город на юге России. Сколько рейсов прибыло в Анапу за 2017 год?

SELECT count(*)
FROM dst_project.flights AS f
JOIN dst_project.airports AS a ON f.arrival_airport=a.airport_code
WHERE a.city = 'Anapa'
  AND DATE_PART('year', actual_arrival) = 2017

Вопрос 2. Сколько рейсов из Анапы вылетело зимой 2017 года?

SELECT COUNT(*)
FROM dst_project.flights AS f
JOIN dst_project.airports AS a ON f.departure_airport=a.airport_code
WHERE a.city = 'Anapa'
  AND DATE_PART('year', actual_arrival) = 2017
  AND DATE_PART('month', actual_arrival) IN (12,1,2)

Вопрос 3. Посчитайте количество отмененных рейсов из Анапы за все время.

SELECT COUNT(*)
FROM dst_project.flights AS f
JOIN dst_project.airports AS a ON f.departure_airport=a.airport_code
WHERE a.city = 'Anapa'
  AND f.status = 'Cancelled'

Вопрос 4. Сколько рейсов из Анапы не летают в Москву?

SELECT count(*)
FROM dst_project.flights AS f
JOIN dst_project.airports AS ad ON f.departure_airport=ad.airport_code
JOIN dst_project.airports AS aa ON f.arrival_airport=aa.airport_code
WHERE ad.city = 'Anapa'
  AND aa.city != 'Moscow'

Вопрос 5. Какая модель самолета летящего на рейсах из Анапы имеет больше всего мест?

Решил что более простым решением будет сделать в два селекта

1)SELECT DISTINCT ac.model
FROM dst_project.flights AS f
JOIN dst_project.airports AS ap ON f.departure_airport = ap.airport_code
JOIN dst_project.aircrafts AS ac ON f.aircraft_code = ac.aircraft_code
WHERE ap.city = 'Anapa'

2)SELECT ac.model,
       COUNT(s.*)
FROM dst_project.aircrafts AS ac
JOIN dst_project.seats AS s ON s.aircraft_code = ac.aircraft_code
WHERE ac.model in ('Boeing 737-300',
                   'Sukhoi Superjet-100')
GROUP BY ac.model
ORDER BY 2 DESC
LIMIT 1