# Техническая архитектура OpenCrimea

## Обзор проекта

OpenCrimea — open-source платформа аренды жилья для Крыма, аналог Airbnb. Платформа должна обеспечивать поиск, бронирование и управление арендой жилья с учетом местных особенностей и ограничений.

## Tech Stack

### Frontend
- **Framework**: Next.js 14+ (React-based)
  - Server-side rendering для SEO
  - App Router для лучшей производительности
  - TypeScript для type safety
- **UI Kit**: Tailwind CSS + Headless UI
  - Быстрая разработка
  - Кастомизация под брэнд
  - Responsive design
- **State Management**: Zustand
  - Легче Redux, достаточно для MVP
  - TypeScript support
- **Maps**: OpenStreetMap + Leaflet
  - Open-source альтернатива Google Maps
  - Нет ограничений по санкциям
- **Форматирование**: Prettier + ESLint
- **Тестирование**: Jest + Testing Library

### Backend
- **Runtime**: Node.js 20+ LTS
- **Framework**: Fastify
  - Высокая производительность
  - TypeScript support
  - Меньше overhead чем Express
- **Language**: TypeScript
- **Authentication**: JWT + Refresh Tokens
- **API**: REST (GraphQL для v2)
- **Validation**: Zod
- **File Upload**: Multer + Sharp (обработка изображений)
- **Email**: Nodemailer
- **Планировщик задач**: node-cron

### База данных
- **Primary DB**: PostgreSQL 16+
  - ACID compliance
  - Spatial data support для геолокации
  - JSON support для гибкой схемы
- **Cache**: Redis
  - Session storage
  - API caching
  - Real-time features
- **Search**: PostgreSQL Full-Text Search
  - Встроенное решение для MVP
  - Upgrade до Elasticsearch в будущем
- **File Storage**: MinIO (S3-compatible)
  - Open-source object storage
  - Совместимость с S3 API

### Инфраструктура и DevOps
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (production)
- **Proxy**: Nginx
- **Monitoring**: Prometheus + Grafana
- **Logging**: Winston + ELK Stack
- **CI/CD**: GitHub Actions
- **Package Manager**: pnpm

## Архитектурные решения

### MVP: Модульный Монолит

Для MVP выбираем модульный монолит по следующим причинам:

#### Преимущества для MVP:
- **Скорость разработки**: Быстрое прототипирование и итерации
- **Простота деплоя**: Один контейнер, одна база данных
- **Отладка**: Проще трейсинг и дебаг
- **Команда**: Подходит для малых команд (до 10 разработчиков)
- **Ресурсы**: Меньше overhead на инфраструктуру

#### Структура модулей:
```
src/
├── modules/
│   ├── auth/           # Аутентификация
│   ├── users/          # Пользователи
│   ├── properties/     # Недвижимость
│   ├── bookings/       # Бронирования
│   ├── payments/       # Платежи
│   ├── reviews/        # Отзывы
│   ├── chat/           # Чат
│   └── notifications/  # Уведомления
├── shared/
│   ├── database/       # DB connections
│   ├── utils/          # Утилиты
│   ├── types/          # TypeScript types
│   └── middleware/     # Middleware
└── core/
    ├── server.ts       # Fastify server
    ├── config.ts       # Конфигурация
    └── app.ts          # Инициализация приложения
```

#### Migration Path к Микросервисам:
- Domain boundaries уже определены модулями
- Shared database → Database-per-service
- In-process calls → HTTP/gRPC calls
- Монолитный деплой → Независимые деплои

## Ключевые модули

### 1. Аутентификация (Auth)
**Функции:**
- Регистрация/авторизация пользователей
- Email верификация
- Восстановление пароля
- Social login (VK, Telegram)

**Технологии:**
- JWT + Refresh tokens
- bcrypt для хеширования паролей
- Email verification через nodemailer
- Rate limiting против брутфорса

### 2. Пользователи (Users)
**Функции:**
- Профили хостов и гостей
- KYC верификация
- Документооборот
- Предпочтения

**Особенности:**
- Верификация паспорта РФ
- Загрузка документов
- Система рейтингов

### 3. Недвижимость (Properties)
**Функции:**
- Создание/редактирование объявлений
- Галерея фотографий
- Геолокация
- Календарь доступности
- Ценообразование

**Технологии:**
- PostGIS для геоданных
- Image optimization (Sharp)
- календарь блокировок

### 4. Поиск (Search)
**Функции:**
- Поиск по критериям
- Фильтрация
- Геопоиск
- Сортировка

**Технологии:**
- PostgreSQL Full-Text Search
- Spatial queries (PostGIS)
- Индексирование для производительности

### 5. Бронирование (Bookings)
**Функции:**
- Создание бронирований
- Подтверждение хостом
- Календарь занятости
- Отмена бронирований

**Бизнес-логика:**
- Проверка доступности
- Автоматическое подтверждение
- Политика отмен

### 6. Платежи (Payments)
**Функции:**
- Прием платежей
- Эскроу система
- Выплаты хостам
- Возвраты

**Провайдеры:**
- ЮKassa (основной для РФ)
- Сбербанк API
- Tinkoff Acquiring
- Криптоплатежи (опционально)

### 7. Чат (Chat)
**Функции:**
- Сообщения между хостом и гостем
- Уведомления в реальном времени
- История переписки

**Технологии:**
- WebSocket (Socket.IO)
- Redis для real-time state
- Push notifications

### 8. Отзывы (Reviews)
**Функции:**
- Отзывы гостей о жилье
- Отзывы хостов о гостях
- Рейтинговая система
- Модерация

**Особенности:**
- Двусторонние отзывы
- Защита от фейковых отзывов
- Система флагов для модерации

### 9. Верификация (Verification)
**Функции:**
- Верификация личности
- Проверка документов
- Верификация номера телефона
- Верификация email

**Технологии:**
- ML для проверки документов
- SMS gateway для РФ номеров
- OCR для паспортов

## Инфраструктура (с учетом санкций)

### Российские облачные провайдеры

#### Основной выбор: Yandex Cloud
**Преимущества:**
- Полная локализация в РФ
- API совместимый с AWS
- Kubernetes service
- Managed PostgreSQL
- Object Storage (S3-compatible)
- CDN по всей России

**Сервисы:**
- **Compute**: Yandex Compute Cloud (виртуальные машины)
- **Container**: Yandex Kubernetes Engine
- **Database**: Yandex Managed PostgreSQL
- **Cache**: Yandex Managed Redis  
- **Storage**: Yandex Object Storage
- **CDN**: Yandex Cloud CDN
- **Load Balancer**: Yandex Application Load Balancer

#### Альтернативы:
- **VK Cloud**: (ex Mail.ru Cloud)
- **SberCloud**: Облако Сбербанка
- **Selectel**: Российский провайдер

### Архитектура развертывания

#### Staging/Production Environment:
```yaml
Production:
  Load Balancer (ALB)
  ├── Frontend (2x instances)
  ├── Backend (3x instances) 
  ├── Redis Cluster (3x nodes)
  └── PostgreSQL (Primary + 2x Read Replicas)

Monitoring:
  ├── Prometheus
  ├── Grafana  
  └── AlertManager

Storage:
  ├── Object Storage (images, documents)
  └── CDN (static assets)
```

#### Development:
- Docker Compose локально
- Staging в Kubernetes
- Feature branches → preview deployments

### Disaster Recovery:
- Ежедневные бекапы БД
- Cross-region replication
- RTO: 4 часа, RPO: 1 час

## CI/CD Pipeline

### GitHub Actions Workflow:

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: pnpm install
      - run: pnpm test
      - run: pnpm lint
      - run: pnpm type-check

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker images
        run: docker build -t opencrimea:${{ github.sha }} .
      - name: Push to registry
        run: docker push opencrimea:${{ github.sha }}

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: kubectl apply -f k8s/staging/

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to production
        run: kubectl apply -f k8s/production/
```

### Качество кода:
- **Pre-commit hooks**: Husky + lint-staged
- **Code coverage**: 80%+ для критических модулей
- **Security scanning**: Snyk
- **Dependency updates**: Renovate Bot

## Структура GitHub репозитория

### Monorepo структура:
```
opencrimea/
├── .github/
│   ├── workflows/         # CI/CD
│   ├── ISSUE_TEMPLATE/    # Шаблоны issues
│   └── PULL_REQUEST_TEMPLATE.md
├── apps/
│   ├── web/              # Frontend (Next.js)
│   ├── api/              # Backend (Fastify)
│   └── admin/            # Admin panel
├── packages/
│   ├── shared/           # Общие типы и утилиты
│   ├── ui/               # UI компоненты
│   └── config/           # Конфиги (ESLint, Prettier)
├── infrastructure/
│   ├── docker/           # Dockerfiles
│   ├── k8s/              # Kubernetes манифесты
│   └── terraform/        # Infrastructure as Code
├── docs/
│   ├── api/              # API документация
│   ├── deployment/       # Деплой инструкции
│   └── contributing/     # Контрибьюторам
├── scripts/
│   ├── dev/              # Development скрипты
│   ├── build/            # Build скрипты
│   └── migration/        # DB миграции
├── docker-compose.yml    # Local development
├── package.json          # Root package.json
├── pnpm-workspace.yaml   # PNPM workspace
├── README.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
└── LICENSE
```

### Конфигурация workspace:
```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### Branch Strategy:
- **main**: Production ready code
- **develop**: Integration branch
- **feature/***: Feature branches
- **hotfix/***: Hotfixes
- **release/***: Release preparation

## Лицензирование

### Рекомендуемая лицензия: AGPL-3.0

**Причины выбора AGPL-3.0:**
- **Copyleft**: Гарантирует, что форки останутся open-source
- **Network copyleft**: Защищает от SaaS-оберток без открытия кода
- **Community building**: Стимулирует контрибьюции обратно в проект
- **Commercial friendly**: Позволяет коммерческое использование

**Альтернативы:**
- **MIT**: Если нужна максимальная свобода
- **Apache 2.0**: Если нужна защита патентов
- **GPL-3.0**: Если не нужна network copyleft защита

### Contributor License Agreement (CLA):
```
# CLA-INDIVIDUAL.md
Individual Contributor License Agreement

By signing this CLA, you agree to license your contributions
under the same license as the project (AGPL-3.0).
```

## Безопасность

### Ключевые аспекты:
- **Authentication**: JWT с короткими TTL
- **Authorization**: RBAC модель
- **Data Protection**: Шифрование PII данных
- **API Security**: Rate limiting, CORS, CSP
- **Infrastructure**: Private networks, WAF, DDoS protection
- **Compliance**: Соответствие 152-ФЗ о персональных данных

### Security Headers:
```javascript
// Helmet.js конфигурация
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "*.yandex.net"],
      // ...
    }
  }
}))
```

## Мониторинг и логирование

### Метрики:
- **Application**: Response time, error rate, throughput
- **Business**: Bookings, conversions, revenue
- **Infrastructure**: CPU, memory, disk, network

### Логирование:
```javascript
// Структурированные логи
logger.info('Booking created', {
  userId: user.id,
  propertyId: property.id,
  amount: booking.amount,
  traceId: req.traceId
})
```

### Алерты:
- Error rate > 1%
- Response time > 2s (95th percentile)
- Database connections > 80%
- Disk usage > 85%

## Производительность

### Целевые метрики:
- **Time to First Byte**: < 200ms
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 3s
- **Core Web Vitals**: Green score

### Оптимизации:
- CDN для статических ресурсов
- Database indexing
- Redis кеширование
- Image optimization
- Code splitting (frontend)

## Масштабирование (Roadmap)

### Phase 1: MVP (0-10k users)
- Модульный монолит
- Single instance deployment
- PostgreSQL + Redis

### Phase 2: Growth (10k-100k users)
- Horizontal scaling backend
- Read replicas
- CDN implementation
- Performance optimizations

### Phase 3: Scale (100k+ users)
- Microservices migration
- Event-driven architecture
- Advanced caching
- Multi-region deployment

---

## Заключение

Данная архитектура обеспечивает:
- **Быстрый старт MVP** с модульным монолитом
- **Возможность масштабирования** к микросервисам
- **Соответствие ограничениям** (санкции, локальное законодательство)
- **Open-source friendly** структуру и лицензирование
- **Production-ready** инфраструктуру на российских облаках

Архитектура позволяет команде сфокусироваться на бизнес-логике в начале, а затем эволюционировать в более сложную систему по мере роста продукта и команды.