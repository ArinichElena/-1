# Поликлиники города N

Объектом БД является деятельность поликлиник города N, оказывающих медицинскую помощь пациентам в части приемов врачей, направлении на исследования, получении результатов исследований от лабораторий.

**На основе БД можно будет:**

- посчитать количество пациентов, получаемых консультации врачей во временных срезах, в разбивке по диагнозам, специализациям врачей, для отслеживания сезонных пиков заболевания и выявления аномалий;
- отследить лаборатории нарушающие сроки выполнения исследований, а также выявлять исследования, которые чаще всего назначаются, строить прогнозы и на основе них закупать нужные препараты;
- узнать какие врачи пользуются популярностью и понять, какие из них перегружены и возможно необходимо увеличить/уменьшить количество персонала.

Схема БД [https://app.sqldbm.com/PostgreSQL/Edit/p252077/#][схема бд]

<image src="https://github.com/ArinichElena/Documentation/blob/main/СхемаБДV2.jpg.png">

**referral_doctor - факт направления к врачу**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор направления | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | specialization_id | Идентификатор специализации врача | int | FK (specialization.id) | + | Дополнительная информация по справочнику Специализация врача |
| 4 | load_date | Дата заведения | datetime | | + | |

create table referral_doctor (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
specialization_id INT not null references specialization,  
load_date TIMESTAMP default CURRENT_TIMESTAMP   
);  

**record_doctor - факт записи к врачу**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор записи | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | referal_id | Идентификатор направления | bigint | FK (referal.id) | - | Может быть не заполнено, т.к. пациент не обязательно записывается по направлению|
| 4 | doctor_id | Идентификатор врача | int | FK (doctor.id) | + | Дополнительная информация по справочнику Врач |
| 5 | clinic_id | Идентификатор поликлиники | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |
| 6 | load_date | Дата заведения | datetime | | + ||
| 7 | record_date | Дата записи к врачу | datetime | | + ||

create table record_doctor (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
referal_id BIGINT references referal,  
doctor_id INT not null references doctor,  
clinic_id INT not null references clinic,  
load_date TIMESTAMP default CURRENT_TIMESTAMP,  
record_date TIMESTAMP not null  
); 

**reception_doctor - факт состоявшегося приеме у врача**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор приема | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | record_id | Идентификатор записи к врачу | bigint | FK (record.id) | - | Может быть не заполнено, т.к. пациент не обязательно приходит на прем по записи (например живая очередь) |
| 4 | doctor_id | Идентификатор врача | int | FK (doctor.id) | + | Дополнительная информация по справочнику Врач |
| 5 | clinic_id | Идентификатор поликлиники | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |
| 6 | load_date | Дата заведения | datetime | | + | |
| 7 | reception_date | Дата приема | datetime | | + | |
| 8 | code_diag | Код диагноза | varchar | | - | По международной классификации болезней |
| 9 | comment_doctor | Комментарий | text | | - | Комментарий, заполняющийся врачом (рекомендации лечения, расшифровка диагноза и т.п.)|

create table reception_doctor (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
record_id BIGINT references record,  
doctor_id INT not null references doctor,  
clinic_id INT not null references clinic,  
load_date TIMESTAMP default CURRENT_TIMESTAMP,  
code_diag VARCHAR(32),  
comment_doctor TEXT  
);  

*простой индекс по полю коду диагноза. можно по нему группировать/отбирать данные по коду диагноза*  

create index idx_reception_diag  
on reception_doctor(code_diag);  

*простой индекс по дате приема. для составление графиков по дням, для отслеживания количества приемов, отбор только необходимого периода*  

create index idx_recep_doc_load_date  
on reception_doctor(load_date);  

**referral_research - факт направления на исследование**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор направления | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | research_id | Идентификатор исследования | int | FK (research.id) | + | Дополнительная информация по справочнику Исследование |
| 4 | load_date | Дата заведения | datetime | | + | |

create table referral_research (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
research_id INT not null references research,  
load_date TIMESTAMP default CURRENT_TIMESTAMP  
);  

**record_research - факт записи на исследование**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор записи | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | referal_research_id | Идентификатор направления на исследование | bigint | FK (referal_research.id) | - | Может быть не заполнено, т.к. пациент не обязательно записывается по направлению |
| 4 | research_id | Идентификатор исследования | int | FK (research.id) | + | Дополнительная информация по справочнику Исследование |
| 5 | clinic_id | Идентификатор поликлиники | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |
| 6 | load_date | Дата заведения | datetime | | + | |
| 7 | record_date | Дата записи к врачу | datetime | | + | |

create table record_research (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
referal_research_id BIGINT references referal_research,  
research_id INT not null references research,  
clinic_id INT not null references clinic,  
load_date TIMESTAMP default CURRENT_TIMESTAMP,  
record_date TIMESTAMP not null  
);  

**fact_research - факт состоявшегося исследования**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор прошедшего исследования | bigint | PKUnique | + | Автонумерация |
| 2 | patient_id | Идентификатор пациента | bigint | FK (patient.id) | + | Дополнительная информация по справочнику Пациент |
| 3 | record_research_id | Идентификатор записи на исследование | bigint | FK (record_research.id) | - | Может быть не заполнено, т.к. пациент не обязательно приходит на исследование по записи (например живая очередь) |
| 4 | doctor_id | Идентификатор врача, проводившего исследование | int | FK (doctor.id) | + | Дополнительная информация по справочнику Врач |
| 5 | clinic_id | Идентификатор поликлиники | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |
| 6 | laboratory_id | Идентификатор лаборатории | int | FK (laboratory.id) | - | Поле заполняется, если исследование было лабораторное, а не инструментальное |
| 7 | research_id | Идентификатор исследования | int | FK (research.id) | + | Дополнительная информация по справочнику Исследование |
| 8 | load_date | Дата заведения | datetime | | + | |
| 9 | result_date | Дата получения результата исследования | datetime | | + | |
| 10 | result_research | Описание результата | text | | - | |
| 11 | comment_research | Комментарий | text | | - | Комментарий, заполняющийся врачом (расшифровка результата и т.п.) |

create table fact_research (  
id BIGSERIAL primary key,  
patient_id BIGINT not null references patient,  
record_research_id BIGINT references record_research,  
doctor_id INT not null references doctor,  
clinic_id INT not null references clinic,  
laboratory_id INT references laboratory,  
research_id INT not null references research,  
load_date TIMESTAMP default CURRENT_TIMESTAMP,  
result_date TIMESTAMP not null,  
result_research TEXT,  
comment_research TEXT  
);  

*простой индекс по дате исследования. для составление графиков по дням, для отслеживания количества исследований, отбор только необходимого периода*  
  
create index idx_fact_research_load_date   
on fact_research(load_date);  

**patient - пациент**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор пациента | bigint | PKUnique | + | Автонумерация |
| 2 | surname | Фамилия пациента | varchar | | + | |
| 3 | name | Имя пациента | varchar | | + | |
| 4 | patronymic | Отчество пациента | varchar | | - | |
| 5 | birthday | Дата рождения | date | | - | |
| 6 | medical_policy | Медицинский полис | bigint | | - | |
| 7 | gender | Пол | varchar | | - | |

create table patient (  
id BIGSERIAL primary key,  
surname VARCHAR(100) not null,  
name VARCHAR(100) not null,  
patronymic VARCHAR(100),  
birthday DATE,  
medical_policy BIGINT UNIQUE,  
gender VARCHAR(32)  
);  

**clinic - поликлиника**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор поликлиники | int | PKUnique | + | Автонумерация |
| 2 | name | Полное наименование поликлиники | text | | + | |
| 3 | short_name | Краткое наименование поликлиники | text | | + | |
| 4 | address | Адрес | text | | + | |
| 5 | type_clinic_id | Тип поликлиник | int | FK (type_clinic.id) | + | Дополнительная информация по справочнику Тип поликлиники |

create table clinic (  
id SERIAL primary key,  
name TEXT not null,  
short_name TEXT not null,  
address TEXT not null,  
type_clinic_id INT not null references type_clinic  
);  

**type_clinic – тип поликлиники**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор типа | int | PKUnique | + | Автонумерация |
| 2 | name | Наименование типа поликлиники | varchar | | + | Детская, взрослая, смешанная |

create table type_clinic (  
id SERIAL primary key,  
name VARCHAR(32) not null  
); 

**laboratory - лаборатория**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор лаборатории | int | PKUnique | + | Автонумерация |
| 2 | name | Полное наименование лаборатории | text | | + | |
| 3 | short_name | Краткое наименование лаборатории | text | | + | |
| 4 | address | Адрес | text | | + | |
| 5 | clinic_id | Идентификатор поликлиники, к которой относится лаборатория | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |

create table laboratory (  
id SERIAL primary key,  
name TEXT not null,  
short_name TEXT not null,  
address TEXT not null,  
clinic_id INT not null references clinic  
);  

**doctor - врач**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор врача | int | PKUnique | + | Автонумерация |
| 2 | surname | Фамилия врача | varchar | | + | |
| 3 | name | Имя врача | varchar | | + | |
| 4 | patronymic | Отчество врача | varchar | | - | |
| 5 | birthday | Дата рождения | date | | - | |
| 6 | gender | Пол | varchar | | - | |
| 7 | specialization_id | Идентификатор специализации | int | FK (specialization.id) | + | Дополнительная информация по справочнику Специализация врача |
| 8 | clinic_id | Идентификатор поликлиники | int | FK (clinic.id) | + | Дополнительная информация по справочнику Поликлиника |

create table doctor (  
id SERIAL primary key,  
surname VARCHAR(100) not null,  
name VARCHAR(100) not null,  
patronymic VARCHAR(100),  
birthday DATE,  
gender VARCHAR(32),  
specialization_id INT not null references specialization,  
clinic_id INT not null  
);  

**specialization – специализация врача**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор специализации | int | PKUnique | + | Автонумерация |
| 2 | name | Наименование специализации | varchar | | + | |

create table specialization (  
id SERIAL primary key,  
name VARCHAR(200) not null  
);

**research - исследование**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор исследования | int | PKUnique | + | Автонумерация |
| 2 | name | Полное наименование исследования | text | | + | |
| 3 | short_name | Краткое исследования | text | | + | |
| 4 | type_research_id | Идентификатор типа исследования | int | FK (type_research.id) | + | Дополнительная информация по справочнику Тип исследования |

create table research (  
id SERIAL primary key,  
name TEXT not null,  
short_name TEXT not null,  
type_research_id INT not null references type_research  
); 

**type_research – тип исследования**
| № | Наименование | Описание | Тип данных | Ограничение | Обязательность заполнения | Примечание |
|:----|:----|:----|:----|:----|:----|:----|
| 1 | id | Идентификатор специализации | int | PKUnique | + | Автонумерация |
| 2 | name | Наименование типа исследования | varchar | | + | Лабораторное, инструментальное |

create table type_research (  
id SERIAL primary key,  
name VARCHAR(32) not null  
); 

[схема бд]: https://app.sqldbm.com/PostgreSQL/Edit/p252077/#
