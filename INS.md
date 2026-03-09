sequenceDiagram
autonumber
actor User
participant Frontend
participant AuthService
participant CatalogService
participant SecurityService
participant OrderService
participant PaymentService
participant NotificationService

User ->> Frontend: Войти в приложение
activate Frontend
Frontend->> AuthService: GET api/v1/auth_data
activate AuthService
AuthService-->> Frontend: auth_data
deactivate AuthService
Frontend -->> User: показать окно ввода логина и пароля
note over User: Ввести данные
User ->> Frontend: отправить данные
Frontend ->> AuthService: передать авторизационные данные клиента POST www./api/v1/auth_data
activate AuthService
note right of AuthService: валидация пользователя
AuthService -->> Frontend: 200 OK
deactivate AuthService
Frontend -->> User: личный кабинет

User->> Frontend: Оформить страховку на частный дом
Frontend->> CatalogService: GET: api/v1/fill_address
activate CatalogService
CatalogService-->>Frontend: форма заполнения адреса
deactivate CatalogService
Frontend-->>User: показать форму заполнения адреса
note over User: заполнить адрес
User->>Frontend: отправить данные
Frontend->>CatalogService: GET api/v1/tariffs? address = "client_address"
activate CatalogService
alt Адрес подходит для страхования
    CatalogService-->>Frontend: передать тарифы для адреса клиента
else нет тарифов для адреса клиента
    CatalogService->> Frontend: нет подходящих тарифов
    deactivate CatalogService
    Frontend->>User: по вашему адресу нет подходящих тарифов
    note over User: закрыть приложение
end

loop Документы [unlimited]
   User->>Frontend: загрузить документы для оформления полиса
   Frontend->>SecurityService: POST api/v1/docs/verify_docs
   activate SecurityService
   note over SecurityService: проверить документы(3-5 минут)
   alt документы валидны
       SecurityService->>OrderService: GET api/v1/tariffs/{client_id}
       OrderService->> SecurityService: tariffs{client_id}
       SecurityService->> Frontend: [тарифы для клиента]
       deactivate SecurityService
       Frontend-->> User: список полисов
   else Документы не валидны
       Frontend->>User: исправьте [документы]
   end
   note over User: выбрать подходящий полис
   User->>Frontend: тариф выбран (кнопка "Выбрать")
end

Frontend->>OrderService: POST api/v1/tariff? sum=500&term=1 year...
activate OrderService
OrderService->>OrderService: сформировать пакет документов (полис, общие положения, пр.)
OrderService-->>Frontend: отправить пакет документов
Frontend->> User: документы + чек-бокс согласия на страхование
User->>Frontend: подписать документы (нажать чек-бокс согласия на страхование)
Frontend->>OrderService: документы подписаны
note over OrderService: принять документы
OrderService-->> Frontend: документы приняты
Frontend->>PaymentService: запрос формы оплаты
activate PaymentService
PaymentService-->>Frontend: передать форму оплаты
Frontend->> User: показать форму оплаты
note over User: заполнить форму
User->> Frontend: нажать кнопку "Оплатить"
Frontend->>PaymentService: передать платежные данные
PaymentService-->>Frontend: 200 OK
deactivate Frontend
par
    PaymentService->>OrderService: полис "321" оплачен
    deactivate PaymentService
and
    OrderService->> NotificationService: отправить пакет документов на email
    activate NotificationService
    NotificationService-->> OrderService: запрос принят
    deactivate OrderService
    NotificationService->> User: пакет документов на email
    deactivate NotificationService
end
