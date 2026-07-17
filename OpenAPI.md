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
                  - requestId: 550e8400-e29b-41d4-a716-446655440002
                    status: completed
                    dateCreated: 2024-12-01T09:00:00Z
                    amount: 22000.00
                    _links:
                      self:
                        href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440002
                        method: GET
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
                    fullName: Иванов Иван Иванович
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
                      fullName: Иванов Петр Иванович
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
                $ref: '#/components/schemas/CreateResponse'
              example:
                requestId: 550e8400-e29b-41d4-a716-446655440001
                status: created
                _links:
                  self:
                    href: /api/v1/requests/550e8400-e29b-41d4-a716-446655440001
                    method: GET
                    title: Получить информацию о заявлении
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
                  id: 550e8400-e29b-41d4-a716-446655440001
                  transactionId: 550e8400-e29b-41d4-a716-446655440000
                  category: education
                  status: sent_to_fns
                  recipient:
                    type: self
                    fullName: Иванов Иван Иванович
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
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'
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

    RequestStatus:
      type: string
      enum: [created, sent_to_partner, sent_to_fns, prefilled, verification, refund, completed, rejected]

    RecipientInfo:
      type: object
      additionalProperties: false
      required:
        - type
        - fullName
      properties:
        type:
          type: string
          enum: [self, spouse, parent, child]
        fullName:
          type: string
          minLength: 1
          maxLength: 200
          pattern: ^[А-ЯЁ][а-яё]+\s[А-ЯЁ][а-яё]+\s[А-ЯЁ][а-яё]+$
        inn:
          type: string
          minLength: 10
          maxLength: 12
          pattern: ^\d{10,12}$
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
        - issuedBy
        - issuedDate
      properties:
        series:
          type: string
          minLength: 4
          maxLength: 4
          pattern: ^\d{4}$
        number:
          type: string
          minLength: 6
          maxLength: 6
          pattern: ^\d{6}$
        issuedBy:
          type: string
          minLength: 1
          maxLength: 200
        issuedDate:
          type: string
          format: date
          pattern: ^\d{4}-\d{2}-\d{2}$

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
          pattern: ^\d{4}$
        number:
          type: string
          minLength: 6
          maxLength: 6
          pattern: ^\d{6}$
        issuedDate:
          type: string
          format: date
          pattern: ^\d{4}-\d{2}-\d{2}$
        issuedBy:
          type: string
          minLength: 1
          maxLength: 200

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
          maxLength: 200
        bik:
          type: string
          minLength: 9
          maxLength: 9
          pattern: ^\d{9}$
        accountNumber:
          type: string
          minLength: 20
          maxLength: 20
          pattern: ^\d{20}$

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
        category:
          $ref: '#/components/schemas/RequestCategory'
        recipient:
          $ref: '#/components/schemas/RecipientInfo'
        accountDetails:
          $ref: '#/components/schemas/AccountDetails'

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
        method:
          type: string
          enum: [GET, POST, PUT, PATCH, DELETE]
        title:
          type: string

    CreateResponse:
      type: object
      additionalProperties: false
      properties:
        requestId:
          type: string
          format: uuid
        status:
          $ref: '#/components/schemas/RequestStatus'
        _links:
          type: object
          additionalProperties: false
          properties:
            self:
              $ref: '#/components/schemas/Link'

    Request:
      type: object
      additionalProperties: false
      required:
        - id
        - transactionId
        - category
        - status
        - dateCreated
        - dateUpdated
      properties:
        id:
          type: string
          format: uuid
        transactionId:
          type: string
          format: uuid
        category:
          $ref: '#/components/schemas/RequestCategory'
        status:
          $ref: '#/components/schemas/RequestStatus'
        recipient:
          $ref: '#/components/schemas/RecipientInfo'
        accountDetails:
          $ref: '#/components/schemas/AccountDetails'
        amount:
          type: number
          format: double
          minimum: 0
        fnsId:
          type: string
        dateCreated:
          type: string
          format: date-time
        dateUpdated:
          type: string
          format: date-time

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
        status:
          $ref: '#/components/schemas/RequestStatus'
        dateCreated:
          type: string
          format: date-time
        amount:
          type: number
          format: double
          minimum: 0
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
        error:
          type: string
        message:
          type: string

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

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "definitions": {
    "RequestCategory": {
      "type": "string",
      "enum": ["education", "medical", "sport", "insurance", "pension", "charity"]
    },
    "RequestStatus": {
      "type": "string",
      "enum": ["created", "sent_to_partner", "sent_to_fns", "prefilled", "verification", "refund", "completed", "rejected"]
    },
    "PassportInfo": {
      "type": "object",
      "additionalProperties": false,
      "required": ["series", "number", "issuedBy", "issuedDate"],
      "properties": {
        "series": {
          "type": "string",
          "minLength": 4,
          "maxLength": 4,
          "pattern": "^\\d{4}$"
        },
        "number": {
          "type": "string",
          "minLength": 6,
          "maxLength": 6,
          "pattern": "^\\d{6}$"
        },
        "issuedBy": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200
        },
        "issuedDate": {
          "type": "string",
          "format": "date",
          "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
        }
      }
    },
    "BirthCertificateInfo": {
      "type": "object",
      "additionalProperties": false,
      "required": ["series", "number", "issuedDate", "issuedBy"],
      "properties": {
        "series": {
          "type": "string",
          "minLength": 4,
          "maxLength": 4,
          "pattern": "^\\d{4}$"
        },
        "number": {
          "type": "string",
          "minLength": 6,
          "maxLength": 6,
          "pattern": "^\\d{6}$"
        },
        "issuedDate": {
          "type": "string",
          "format": "date",
          "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
        },
        "issuedBy": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200
        }
      }
    },
    "RecipientInfo": {
      "type": "object",
      "additionalProperties": false,
      "required": ["type", "fullName"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["self", "spouse", "parent", "child"]
        },
        "fullName": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200,
          "pattern": "^[А-ЯЁ][а-яё]+\\s[А-ЯЁ][а-яё]+\\s[А-ЯЁ][а-яё]+$"
        },
        "inn": {
          "type": "string",
          "minLength": 10,
          "maxLength": 12,
          "pattern": "^\\d{10,12}$"
        },
        "passport": {
          "$ref": "#/definitions/PassportInfo"
        },
        "birthCertificate": {
          "$ref": "#/definitions/BirthCertificateInfo"
        }
      }
    },
    "AccountDetails": {
      "type": "object",
      "additionalProperties": false,
      "required": ["bankName", "bik", "accountNumber"],
      "properties": {
        "bankName": {
          "type": "string",
          "minLength": 1,
          "maxLength": 200
        },
        "bik": {
          "type": "string",
          "minLength": 9,
          "maxLength": 9,
          "pattern": "^\\d{9}$"
        },
        "accountNumber": {
          "type": "string",
          "minLength": 20,
          "maxLength": 20,
          "pattern": "^\\d{20}$"
        }
      }
    },
    "CreateRequest": {
      "type": "object",
      "additionalProperties": false,
      "required": ["transactionId", "category", "recipient"],
      "properties": {
        "transactionId": {
          "type": "string",
          "format": "uuid"
        },
        "category": {
          "$ref": "#/definitions/RequestCategory"
        },
        "recipient": {
          "$ref": "#/definitions/RecipientInfo"
        },
        "accountDetails": {
          "$ref": "#/definitions/AccountDetails"
        }
      }
    },
    "Link": {
      "type": "object",
      "additionalProperties": false,
      "required": ["href", "method"],
      "properties": {
        "href": {
          "type": "string",
          "format": "uri"
        },
        "method": {
          "type": "string",
          "enum": ["GET", "POST", "PUT", "PATCH", "DELETE"]
        },
        "title": {
          "type": "string"
        }
      }
    },
    "CreateResponse": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "requestId": {
          "type": "string",
          "format": "uuid"
        },
        "status": {
          "$ref": "#/definitions/RequestStatus"
        },
        "_links": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
            "self": {
              "$ref": "#/definitions/Link"
            }
          }
        }
      }
    },
    "Request": {
      "type": "object",
      "additionalProperties": false,
      "required": ["id", "transactionId", "category", "status", "dateCreated", "dateUpdated"],
      "properties": {
        "id": {
          "type": "string",
          "format": "uuid"
        },
        "transactionId": {
          "type": "string",
          "format": "uuid"
        },
        "category": {
          "$ref": "#/definitions/RequestCategory"
        },
        "status": {
          "$ref": "#/definitions/RequestStatus"
        },
        "recipient": {
          "$ref": "#/definitions/RecipientInfo"
        },
        "accountDetails": {
          "$ref": "#/definitions/AccountDetails"
        },
        "amount": {
          "type": "number",
          "minimum": 0
        },
        "fnsId": {
          "type": "string"
        },
        "dateCreated": {
          "type": "string",
          "format": "date-time"
        },
        "dateUpdated": {
          "type": "string",
          "format": "date-time"
        }
      }
    },
    "GetRequest": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "request": {
          "$ref": "#/definitions/Request"
        },
        "_links": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
            "list": {
              "$ref": "#/definitions/Link"
            }
          }
        }
      }
    },
    "RequestSummary": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "requestId": {
          "type": "string",
          "format": "uuid"
        },
        "status": {
          "$ref": "#/definitions/RequestStatus"
        },
        "dateCreated": {
          "type": "string",
          "format": "date-time"
        },
        "amount": {
          "type": "number",
          "minimum": 0
        },
        "_links": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
            "self": {
              "$ref": "#/definitions/Link"
            }
          }
        }
      }
    },
    "ListRequest": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "items": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/RequestSummary"
          }
        },
        "totalCount": {
          "type": "integer"
        },
        "_links": {
          "type": "object",
          "additionalProperties": false,
          "properties": {
            "self": {
              "$ref": "#/definitions/Link"
            },
            "create": {
              "$ref": "#/definitions/Link"
            }
          }
        }
      }
    },
    "ErrorResponse": {
      "type": "object",
      "additionalProperties": false,
      "properties": {
        "status": {
          "type": "integer"
        },
        "error": {
          "type": "string"
        },
        "message": {
          "type": "string"
        }
      }
    }
  }
}
```