# OpenAPI: управление заявлениями

# Описание значимости артефакта

## Процесс и контекст использования

Артефакт используется в рамках процесса проектирования компонента "Управление заявлениями на социальный налоговый вычет" на этапе технического проектирования API-контрактов.

## Цель создания

Зафиксировать единый контракт для сервиса управления заявлениями, который определяет, как внешние клиенты взаимодействуют с сервисом, какие данные он ожидает и как их хранит.

## Что становится определено

OpenAPI - описано синхронное REST-взаимодействие между клиентским приложением и сервисом

JSON-Schema - описаны отдельные схемы для валидации запросов и ответов

## Пользователи артефакта

Backend-разработчики - для описания кода серверной части строго по контракту: валидируют запросы по JSON-Schema, формируют ответы, обрабатывают коды ошибок

Frontend-разработчики - для интеграции с REST API

Тестировщики - для написания тестов на основе контрактов

Архитекторы - для проверки согласованности API с общей архитектурой и принципами REST/HATEOAS

## Использование в дальнейшем

В дальнейшем артефакт используют за:

- Основу для генерации кода
- Основу для написания интеграционных тестов
- Основу для согласования с командами
- Валидацию данных
- Документацию

## Последствия отсутствия

При отсутствии артефакта могут возникнуть следующие последствия:

- Разработчики не имеют единого понимания контрактов
- Нет возможности автоматической генерации кода
- Сложность согласования с командами без чётких спецификаций
- Усложняется тестирование и валидация

# Спецификация

## OpenAPI
```yaml
openapi: '3.0.3'
info:
  title: Управление заявлениями на социальный налоговый вычет
  version: '1.0'
servers:
  - url: https://api.server.test
security:
  - bearerAuth: []
paths:
  /api/v1/requests:
    get:
      summary: Получить список заявлений
      description: Возвращает список всех заявлений клиента
      operationId: listRequests
      parameters:
        - name: category
          in: query
          schema:
            $ref: '#/components/schemas/RequestCategory'
          description: Фильтр по категории расходов
        - name: status
          in: query
          schema:
            $ref: '#/components/schemas/RequestStatus'
          description: Фильтр по статусу заявления
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 10
            default: 5
          description: Количество записей на странице
        - name: offset
          in: query
          schema:
            type: integer
            minimum: 0
            default: 0
          description: Смещение для пагинации
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ListRequest'
              example:
                items:
                  - requestId: 550e8400-e29b-41d4-a716-446655440001
                    status: verification
                    dateCreated: 2025-01-15T10:00:00Z
                    amount: 15000.00
                    _links:
                      self:
                        href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                        method: GET
                      edit:
                          href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                          method: PATCH
                          title: Изменить заявление
                  - requestId: 550e8400-e29b-41d4-a716-446655440002
                    status: completed
                    dateCreated: 2024-12-01T09:00:00Z
                    amount: 22000.00
                    _links:
                      self:
                        href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440002
                        method: GET
                      edit:
                          href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                          method: PATCH
                          title: Изменить заявление
                totalCount: 2
                _links:
                  self:
                    href: /api/v1/requests?limit=5&offset=0
                    method: GET
                  create:
                    href: /api/v1/requests
                    method: POST
                    title: Создать новое заявление    
        '400':      
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '429':
          $ref: '#/components/responses/TooManyRequests'

    post:
      summary: Создать заявление на вычет
      description: Создает заявление на получение социального налогового вычета
      operationId: createRequest
      parameters:
        - name: Idempotency-Key
          in: header
          required: true
          schema:
            type: string
            format: uuid
          description: Уникальный ключ идемпотентности
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateRequest'
            examples:
              selfRequest:
                summary: Вычет за себя
                value:
                  transactionId: 550e8400-e29b-41d4-a716-446655440000
                  category: education
                  recipient:
                    type: self
                    surname: Иванов
                    name: Петр
                    patronymic: Иванович
                    inn: 1234567890
                    passport:
                      series: 4512
                      number: 345678
                      issuedBy: Отделом УФМС России по г. Москве
                      issuedDate: 2010-05-15
                  accountDetails:
                    bankName: Банк России
                    bik: 044525225
                    accountNumber: 40802810123456789012
              childRequest:
                  summary: Вычет за ребенка
                  value:
                    transactionId: 550e8400-e29b-41d4-a716-446655440000
                    category: education
                    recipient:
                      type: child
                      surname: Иванов
                      name: Петр
                      patronymic: Иванович
                      birthCertificate:
                        series: 1234
                        number: 567890
                        issuedDate: 2020-03-20
                        issuedBy: Управлением ЗАГС г. Москвы
                    accountDetails:
                      bankName: Банк России
                      bik: 044525225
                      accountNumber: 40802810123456789012
      responses:
        '202':
          description: Заявление создано
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Response'
              example:
                requestId: 550e8400-e29b-41d4-a716-446655440001
                status: created
                _links:
                  self:
                    href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                    method: GET
                    title: Получить информацию о заявлении
                  edit:
                    href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                    method: PATCH
                    title: Изменить заявление
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '409':
          $ref: '#/components/responses/Conflict'
        '429':
          $ref: '#/components/responses/TooManyRequests'

  /api/v1/requests/{requestId}:
    get:
      summary: Получить информацию о заявлении
      description: Возвращает информацию о заявлении
      operationId: getRequest
      parameters:
        - name: requestId
          in: path
          required: true
          schema:
            type: string
            format: uuid
          description: Идентификатор заявления
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/GetRequest'
              example:
                request:
                  requestId: 550e8400-e29b-41d4-a716-446655440001
                  transactionId: 550e8400-e29b-41d4-a716-446655440000
                  category: education
                  status: sent_to_fns
                  isDraft: 1
                  isBlocked: 0
                  recipient:
                    type: self
                    surname: Иванов
                    name: Петр
                    patronymic: Иванович
                    inn: 1234567890
                  accountDetails:
                    bankName: Банк России
                    bik: 044525225
                    accountNumber: 40802810123456789012
                  amount: 15000.00
                  fnsId: FNS-2025-001
                  dateCreated: 2025-01-15T10:00:00Z
                  dateUpdated: 2025-01-20T15:30:00Z
                _links:
                  list:
                    href: /api/v1/requests
                    method: GET
                    title: Получить список заявлений
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '429':
          $ref: '#/components/responses/TooManyRequests'

    patch:
      summary: Изменить заявление
      description: Обновляет заявление в статусе "created"
      operationId: editRequest
      parameters:
        - name: requestId
          in: path
          required: true
          schema:
            type: string
            format: uuid
          description: Идентификатор заявления
        - name: Idempotency-Key
          in: header
          required: true
          schema:
            type: string
            format: uuid
          description: Уникальный ключ идемпотентности
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/EditRequest'
            examples:
              updateRecipient:
                summary: Изменить получателя
                value:
                  recipient:
                    type: spouse
                    surname: Иванова
                    name: Мария
                    patronymic: Петровна
                    inn: 1234567891
                    passport:
                      series: 4513
                      number: 345679
                      issuedBy: Отделом УФМС России по г. Москве
                      issuedDate: 2012-06-20
              updateAccount:
                summary: Изменить реквизиты счета
                value:
                  accountDetails:
                    bankName: Новый банк
                    bik: 044525226
                    accountNumber: 40802810234567890123
              submit:
                summary: Подать заявление
                value:
                  isDraft: 1
              cancel:
                summary: Отменить заявление
                value:
                  isBlocked: 1
      responses:
        '200':
          description: Заявление изменено
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/EditResponse'
              example:
                updated:
                    summary: После редактирования
                    value:
                      request:
                        requestId: 550e8400-e29b-41d4-a716-446655440001
                        transactionId: 550e8400-e29b-41d4-a716-446655440000
                        category: education
                        status: created
                        isDraft: 1
                        isBlocked: 0
                        recipient:
                          type: spouse
                          surname: Иванова
                          name: Мария
                          patronymic: Петровна
                          inn: 1234567891
                        accountDetails:
                          bankName: Банк России
                          bik: 044525225
                          accountNumber: 40802810123456789012
                        amount: 15000.00
                        dateCreated: 2025-01-15T10:00:00Z
                        dateUpdated: 2025-01-20T16:00:00Z
                      _links:
                        self:
                          href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                          method: GET
                          title: Получить информацию о заявлении
                        list:
                          href: /api/v1/requests
                          method: GET
                          title: Получить список заявлений
                submitted:
                  summary: После подачи
                  value:
                    request:
                      requestId: 550e8400-e29b-41d4-a716-446655440001
                      transactionId: 550e8400-e29b-41d4-a716-446655440000
                      category: education
                      status: sent_to_partner
                      isDraft: 1
                      isBlocked: 0
                      recipient:
                        type: self
                        surname: Иванов
                        name: Иван
                        patronymic: Иванович
                        inn: 1234567890
                      accountDetails:
                        bankName: Банк России
                        bik: 044525225
                        accountNumber: 40802810123456789012
                      amount: 15000.00
                      dateCreated: 2025-01-15T10:00:00Z
                      dateUpdated: 2025-01-20T16:00:00Z
                    _links:
                      self:
                        href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                        method: GET
                        title: Получить заявление
                      list:
                        href: /api/v1/requests
                        method: GET
                        title: Получить список заявлений
                cancelled:
                  summary: После отмены
                  value:
                    request:
                      requestId: 550e8400-e29b-41d4-a716-446655440001
                      transactionId: 550e8400-e29b-41d4-a716-446655440000
                      category: education
                      status: created
                      isDraft: 0
                      isBlocked: 1
                      recipient:
                        type: self
                        surname: Иванов
                        name: Иван
                        patronymic: Иванович
                        inn: 1234567890
                      accountDetails:
                        bankName: Банк России
                        bik: 044525225
                        accountNumber: 40802810123456789012
                      amount: 15000.00
                      dateCreated: 2025-01-15T10:00:00Z
                      dateUpdated: 2025-01-20T16:00:00Z
                    _links:
                      self:
                        href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                        method: GET
                        title: Получить заявление
                      list:
                        href: /api/v1/requests
                        method: GET
                        title: Получить список заявлений
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
        '409':
          $ref: '#/components/responses/Conflict'
        '429':
          $ref: '#/components/responses/TooManyRequests'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    RequestCategory:
      type: string
      enum: [education, medical, sport, insurance, pension, charity]
      description: Категории для социального налогового вычета

    RequestStatus:
      type: string
      enum: [created, sent_to_partner, sent_to_fns, prefilled, verification, refund, completed, rejected]
      description: Статусы заявления

    RecipientInfo:
      type: object
      additionalProperties: false
      required:
        - type
        - surname
        - name
      properties:
        type:
          type: string
          enum: [self, spouse, parent, child]
          description: Тип получателя вычета (себя, супруг, родитель, ребенок)
        surname:
          type: string
          minLength: 1
          maxLength: 50
          description: Фамилия получателя
        name:
          type: string
          minLength: 1
          maxLength: 50
          description: Имя получателя
        patronymic:
          type: string
          minLength: 1
          maxLength: 50
          description: Отчество получателя
        inn:
          type: string
          minLength: 10
          maxLength: 12
          description: ИНН получателя
        passport:
          $ref: '#/components/schemas/PassportInfo'
        birthCertificate:
          $ref: '#/components/schemas/BirthCertificateInfo'

    PassportInfo:
      type: object
      additionalProperties: false
      required:
        - series
        - number
        - unitCode
        - issuedBy
        - issuedDate
      properties:
        series:
          type: string
          minLength: 4
          maxLength: 4
          description: Серия паспорта
        number:
          type: string
          minLength: 6
          maxLength: 6
          description: Номер паспорта
        unitCode:
          type: string
          minLength: 7
          maxLength: 7
          description: Код подразделения
        issuedBy:
          type: string
          minLength: 1
          maxLength: 255
          description: Кем выдан
        issuedDate:
          type: string
          format: date
          description: Дата выдачи

    BirthCertificateInfo:
      type: object
      additionalProperties: false
      required:
        - series
        - number
        - issuedDate
        - issuedBy
      properties:
        series:
          type: string
          minLength: 4
          maxLength: 4
          description: Серия свидетельства о рождении
        number:
          type: string
          minLength: 6
          maxLength: 6
          description: Номер свидетельства о рождении
        issuedDate:
          type: string
          format: date
          description: Дата выдачи
        issuedBy:
          type: string
          minLength: 1
          maxLength: 255
          description: Кем выдано

    AccountDetails:
      type: object
      additionalProperties: false
      required:
        - bankName
        - bik
        - accountNumber
      properties:
        bankName:
          type: string
          minLength: 1
          maxLength: 255
          description: Наименование банка
        bik:
          type: string
          minLength: 9
          maxLength: 9
          description: БИК банка
        accountNumber:
          type: string
          minLength: 20
          maxLength: 20
          description: Номер счета в банке

    CreateRequest:
      type: object
      additionalProperties: false
      required:
        - transactionId
        - category
        - recipient
      properties:
        transactionId:
          type: string
          format: uuid
          description: Идентификатор транзакции
        category:
          $ref: '#/components/schemas/RequestCategory'
        recipient:
          $ref: '#/components/schemas/RecipientInfo'
        accountDetails:
          $ref: '#/components/schemas/AccountDetails'

    EditRequest:
      type: object
      additionalProperties: false
      properties:
        recipient:
          $ref: '#/components/schemas/RecipientInfo'
        accountDetails:
          $ref: '#/components/schemas/AccountDetails'
    
    EditResponse:
      type: object
      additionalProperties: false
      properties:
        requestId:
          type: string
          format: uuid
          description: Идентификатор заявления
        status:
          $ref: '#/components/schemas/RequestStatus'
          description: Статус заявления
        _links:
          type: object
          additionalProperties: false
          properties:
            self:
              $ref: '#/components/schemas/Link'
            list:
              $ref: '#/components/schemas/Link'

    Link:
      type: object
      additionalProperties: false
      required:
        - href
        - method
      properties:
        href:
          type: string
          format: uri
          description: URL ресурса
        method:
          type: string
          enum: [GET, POST, PUT, PATCH, DELETE]
          description: HTTP-метод
        title:
          type: string
          description: Описание

    Response:
      type: object
      additionalProperties: false
      properties:
        requestId:
          type: string
          format: uuid
          description: Идентификатор заявления
        status:
          $ref: '#/components/schemas/RequestStatus'
          description: Статус заявления
        _links:
          type: object
          additionalProperties: false
          properties:
            self:
              $ref: '#/components/schemas/Link'
            edit:
              $ref: '#/components/schemas/Link'

    Request:
      type: object
      additionalProperties: false
      required:
        - requestId
        - transactionId
        - category
        - status
        - isDraft
        - isBlocked
        - dateCreated
        - dateUpdated
      properties:
        requestId:
          type: string
          format: uuid
          description: Идентификатор заявления
        transactionId:
          type: string
          format: uuid
          description: Идентификатор транзакции
        category:
          $ref: '#/components/schemas/RequestCategory'
        status:
          $ref: '#/components/schemas/RequestStatus'
        isDraft:
          type: integer
          enum: [0, 1]
          default: 0
          description: Утверждение заявления
        isBlocked:
          type: integer
          enum: [0, 1]
          default: 0
          description: Блокировка
        recipient:
          $ref: '#/components/schemas/RecipientInfo'
        accountDetails:
          $ref: '#/components/schemas/AccountDetails'
        amount:
          type: number
          format: double
          minimum: 0
          description: Сумма возврата по заявлению
        fnsId:
          type: string
          description: Идентификатор заявления в ФНС
        dateCreated:
          type: string
          format: date-time
          description: Дата и время создания заявления
        dateUpdated:
          type: string
          format: date-time
          description: Дата и время последнего обновления заявления

    GetRequest:
      type: object
      additionalProperties: false
      properties:
        request:
          $ref: '#/components/schemas/Request'
        _links:
          type: object
          additionalProperties: false
          properties:
            list:
              $ref: '#/components/schemas/Link'

    RequestSummary:
      type: object
      additionalProperties: false
      properties:
        requestId:
          type: string
          format: uuid
          description: Идентификатор заявления
        status:
          $ref: '#/components/schemas/RequestStatus'
        dateCreated:
          type: string
          format: date-time
          description: Дата и время создания заявления
        amount:
          type: number
          format: double
          minimum: 0
          description: Сумма возврата по заявлению
        _links:
          type: object
          additionalProperties: false
          properties:
            self:
              $ref: '#/components/schemas/Link'

    ListRequest:
      type: object
      additionalProperties: false
      properties:
        items:
          type: array
          items:
            $ref: '#/components/schemas/RequestSummary'
        totalCount:
          type: integer
          description: Количество заявлений
        _links:
          type: object
          additionalProperties: false
          properties:
            self:
              $ref: '#/components/schemas/Link'
            create:
              $ref: '#/components/schemas/Link'

    ErrorResponse:
      type: object
      additionalProperties: false
      properties:
        status:
          type: integer
          description: Код ошибки
        error:
          type: string
          description: Описание ошибки
        message:
          type: string
          description: Информация об ошибке

  responses:
    BadRequest:
      description: Невалидный запрос
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Unauthorized:
      description: Пользователь не аутентифицирован
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Forbidden:
      description: Доступ запрещен
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    NotFound:
      description: Ресурс не найден
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Conflict:
      description: Конфликт идемпотентности
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    TooManyRequests:
      description: Превышен лимит запросов
      headers:
        Retry-After:
          schema:
            type: integer
        X-RateLimit-Limit:
          schema:
            type: integer
        X-RateLimit-Remaining:
          schema:
            type: integer
        X-RateLimit-Reset:
          schema:
            type: string
            format: date-time
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
```
## JSON-Schema
**CreateRequest: Запрос на создание заявления**
```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "additionalProperties": false,
  "required": ["transactionId", "category", "recipient"],
  "properties": {
    "transactionId": {
      "type": "string",
      "minLength": 36,
      "maxLength": 36
    },
    "category": {
      "type": "string",
      "enum": ["education", "medical", "sport", "insurance", "pension", "charity"]
    },
    "recipient": {
      "type": "object",
      "additionalProperties": false,
      "required": ["type", "surname", "name"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["self", "spouse", "parent", "child"]
        },
        "surname": {
          "type": "string",
          "minLength": 1,
          "maxLength": 50
        },
        "name": {
          "type": "string",
          "minLength": 1,
          "maxLength": 50
        },
        "patronymic": {
          "type": "string",
          "minLength": 1,
          "maxLength": 50
        },
        "inn": {
          "type": "string",
          "minLength": 10,
          "maxLength": 12
        },
        "passport": {
          "type": "object",
          "additionalProperties": false,
          "required": ["series", "number", "unitCode", "issuedBy", "issuedDate"],
          "properties": {
            "series": {
              "type": "string",
              "minLength": 4,
              "maxLength": 4
            },
            "number": {
              "type": "string",
              "minLength": 6,
              "maxLength": 6
            },
            "unitCode": {
              "type": "string",
              "minLength": 7,
              "maxLength": 7
            },
            "issuedBy": {
              "type": "string",
              "minLength": 1,
              "maxLength": 255
            },
            "issuedDate": {
              "type": "string",
              "format": "date"
            }
          }
        },
        "birthCertificate": {
          "type": "object",
          "additionalProperties": false,
          "required": ["series", "number", "issuedDate", "issuedBy"],
          "properties": {
            "series": {
              "type": "string",
              "minLength": 4,
              "maxLength": 4
            },
            "number": {
              "type": "string",
              "minLength": 6,
              "maxLength": 6
            },
            "issuedDate": {
              "type": "string",
              "format": "date"
            },
            "issuedBy": {
              "type": "string",
              "minLength": 1,
              "maxLength": 255
            }
          }
        }
      }
    },
    "accountDetails": {
      "type": "object",
      "additionalProperties": false,
      "required": ["bankName", "bik", "accountNumber"],
      "properties": {
        "bankName": {
          "type": "string",
          "minLength": 1,
          "maxLength": 255
        },
        "bik": {
          "type": "string",
          "minLength": 9,
          "maxLength": 9
        },
        "accountNumber": {
          "type": "string",
          "minLength": 20,
          "maxLength": 20
        }
      }
    }
  }
}
```
**Response: Ответ на создание заявления**
```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "additionalProperties": false,
  "properties": {
    "requestId": {
      "type": "string",
      "minLength": 36,
      "maxLength": 36
    },
    "status": {
      "type": "string",
      "enum": ["created", "sent_to_partner", "sent_to_fns", "prefilled", "verification", "refund", "completed", "rejected"]
    },
    "_links": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "self": {
          "$ref": "#/definitions/link"
        },
        "edit": {
          "$ref": "#/definitions/link"
        }
      }
    }
  },
  "definitions": {
    "link": {
      "type": "object",
      "additionalProperties": false,
      "required": ["href", "method"],
      "properties": {
        "href": {
          "type": "string",
          "format": "url",
          "minLength": 1
        },
        "method": {
          "type": "string",
          "enum": ["GET", "POST", "PUT", "PATCH", "DELETE"]
        },
        "title": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200
        }
      }
    }
  }
}
```