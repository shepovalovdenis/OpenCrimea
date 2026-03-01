# Техническая архитектура OpenCrimea v2.0

## Обзор проекта

OpenCrimea — AI-first open-source платформа аренды жилья для Крыма с автоматизированным парсингом популярных площадок. Платформа объединяет в себе классический функционал бронирования с мощными AI-компонентами для автоматизации процессов модерации, ценообразования и поддержки.

## Ключевые инновации v2.0

### 🤖 AI-First подход
- Автоматическая модерация фото с помощью computer vision
- AI-генерация описаний объявлений
- Динамическое ценообразование на основе ML-алгоритмов  
- Intelligent чатбот для поддержки 24/7

### 🔄 Автоматический импорт контента
- Парсинг объявлений с Авито, Суточно.ру, ЦИАН
- Импорт по прямой ссылке одним кликом
- Дедубликация и валидация данных
- Синхронизация изменений цен и доступности

### 🚀 Self-Service Onboarding
- Пошаговый guided flow для новых хостов
- Автоматическая верификация документов
- AI-асситент для настройки первого объявления
- Gamification элементы для увеличения конверсии

## Tech Stack

### Frontend
- **Framework**: Next.js 14+ (React-based)
  - Server-side rendering для SEO
  - App Router для лучшей производительности
  - TypeScript для type safety
- **UI Kit**: Tailwind CSS + Headless UI + Framer Motion
  - Быстрая разработка
  - Анимации для onboarding flow
  - Responsive design
- **State Management**: Zustand + TanStack Query
  - Оптимизированное кеширование API
  - Real-time updates
- **Maps**: OpenStreetMap + Leaflet
  - Open-source альтернатива Google Maps
  - Нет ограничений по санкциям
- **AI Components**: 
  - React Flow для визуализации AI pipeline
  - Webcam для photo capture в onboarding

### Backend
- **Runtime**: Node.js 20+ LTS
- **Framework**: Fastify
  - Высокая производительность
  - TypeScript support
  - Plugin ecosystem для AI интеграций
- **Language**: TypeScript
- **Authentication**: JWT + Refresh Tokens
- **API**: REST + GraphQL для AI endpoints
- **Validation**: Zod
- **File Upload**: Multer + Sharp (обработка изображений)
- **Email**: Nodemailer
- **Task Queue**: BullMQ (для парсинга и AI обработки)
- **Планировщик**: node-cron + Agenda

### AI/ML Stack
- **Computer Vision**: OpenCV + TensorFlow.js
- **LLM Integration**: OpenAI API + Local LLama models
- **Image Processing**: Sharp + ImageMagick
- **OCR**: Tesseract.js для документов
- **ML Ops**: MLflow для управления моделями
- **Feature Store**: Redis для ML features
- **A/B Testing**: Posthog для AI experiments

### Parsing Infrastructure
- **Web Scraping**: Puppeteer + Playwright
- **Proxy Management**: Bright Data / ProxyMesh
- **Anti-Bot Detection**: Stealth plugins + User-Agent rotation  
- **Data Processing**: Apache Kafka для stream processing
- **Rate Limiting**: Bottleneck для crawler throttling

### База данных
- **Primary DB**: PostgreSQL 16+
  - Vector extensions для AI embeddings
  - ACID compliance
  - Spatial data support для геолокации
  - JSON support для гибкой схемы
- **Vector DB**: Pinecone/Qdrant для semantic search
- **Cache**: Redis + Redis Streams
  - Session storage
  - API caching
  - Real-time AI pipeline events
- **Analytics**: ClickHouse для parsing логов
- **Search**: Elasticsearch для продвинутого поиска
- **File Storage**: MinIO (S3-compatible)
  - AI-processed images
  - ML model artifacts

### Инфраструктура и DevOps
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (production)
- **Service Mesh**: Istio для AI microservices
- **API Gateway**: Kong + AI rate limiting
- **Monitoring**: Prometheus + Grafana + AI model metrics
- **Logging**: Winston + ELK Stack + AI pipeline tracing
- **CI/CD**: GitHub Actions + MLOps pipeline
- **Package Manager**: pnpm

## Архитектурные решения

### MVP: AI-Enhanced Модульный Монолит

Для MVP выбираем модульный монолит с выделенными AI-компонентами:

#### Преимущества для AI-first MVP:
- **Скорость разработки**: Быстрое прототипирование AI features
- **Простота ML Pipeline**: Единая кодовая база для экспериментов
- **Отладка AI**: Проще трейсинг AI решений
- **Команда**: Подходит для cross-functional AI команды
- **Итерации**: Быстрые A/B тесты AI алгоритмов

#### Структура модулей v2.0:
```
src/
├── modules/
│   ├── auth/                    # Аутентификация
│   ├── users/                   # Пользователи
│   ├── properties/              # Недвижимость
│   ├── bookings/               # Бронирования
│   ├── payments/               # Платежи
│   ├── reviews/                # Отзывы
│   ├── chat/                   # Чат + AI Assistant
│   ├── notifications/          # Уведомления
│   ├── onboarding/             # 🆕 Self-service onboarding
│   ├── parsers/               # 🆕 Web scraping modules
│   │   ├── avito/             # Авито парсер
│   │   ├── sutochno/          # Суточно.ру парсер
│   │   ├── cian/              # ЦИАН парсер
│   │   └── common/            # Общие утилиты
│   └── ai/                    # 🆕 AI Pipeline
│       ├── photo-moderation/  # Модерация фото
│       ├── content-generation/ # Генерация описаний
│       ├── pricing/           # AI ценообразование
│       ├── chatbot/           # AI чатбот
│       └── recommendation/    # Рекомендации
├── shared/
│   ├── database/              # DB connections
│   ├── utils/                 # Утилиты
│   ├── types/                 # TypeScript types
│   ├── middleware/            # Middleware
│   ├── ml/                    # 🆕 ML utilities
│   └── queues/               # 🆕 Task queues
└── core/
    ├── server.ts              # Fastify server
    ├── config.ts              # Конфигурация
    ├── ai-pipeline.ts         # 🆕 AI orchestration
    └── app.ts                 # Инициализация приложения
```

## Ключевые модули v2.0

### 🆕 1. AI Pipeline (ai/)

#### 1.1 Модерация фото (photo-moderation/)
**Функции:**
- Автоматическое определение качества фото
- Детекция неподходящего контента
- Классификация типов помещений
- Оценка привлекательности объявления

**Технологии:**
```typescript
// AI модерация pipeline
class PhotoModerationService {
  async moderatePhoto(imageBuffer: Buffer) {
    const analysis = await this.visionModel.analyze(imageBuffer)
    return {
      isAppropriate: analysis.nsfw_score < 0.1,
      quality: analysis.quality_score, // 0-1
      roomType: analysis.room_classification,
      suggestions: analysis.improvement_tips
    }
  }
}
```

#### 1.2 Генерация описаний (content-generation/)
**Функции:**
- Автоматическое создание привлекательных описаний
- SEO-оптимизированные тексты
- Адаптация под целевую аудиторию
- Мультиязычная поддержка (RU/UK/EN)

**Технологии:**
- OpenAI GPT-4 для качественных текстов
- Local Llama модели как fallback
- Prompt engineering для consistent outputs

#### 1.3 AI Ценообразование (pricing/)
**Функции:**
- Динамическое ценообразование в реальном времени
- Анализ конкурентов
- Сезонные корректировки
- Оптимизация для максимальной загрузки

**ML Features:**
- Геолокация и district popularity
- Historical booking patterns
- Competitor pricing
- Local events calendar
- Weather forecasts
- Property amenities scoring

#### 1.4 AI Чатбот (chatbot/)
**Функции:**
- 24/7 поддержка гостей и хостов
- Автоматические ответы на FAQ
- Routing к живым операторам
- Booking assistance

**Технологии:**
```typescript
class AIAssistant {
  async handleQuery(message: string, context: ChatContext) {
    const intent = await this.nlp.classifyIntent(message)
    
    switch(intent.category) {
      case 'booking_help':
        return this.bookingAssistant.help(message, context)
      case 'property_question':
        return this.propertyAssistant.answer(message, context)
      case 'escalate':
        return this.routeToHuman(context)
    }
  }
}
```

### 🆕 2. Модули парсинга (parsers/)

#### 2.1 Универсальная архитектура парсеров
```typescript
abstract class PropertyParser {
  abstract siteName: string
  abstract baseUrl: string
  
  async parseProperty(url: string): Promise<PropertyData> {
    const page = await this.browser.newPage()
    const data = await this.extractData(page, url)
    return this.normalizeData(data)
  }
  
  abstract extractData(page: Page, url: string): Promise<RawPropertyData>
  abstract normalizeData(raw: RawPropertyData): PropertyData
}
```

#### 2.2 Авито парсер (avito/)
**Особенности:**
- Anti-bot обход через residential proxies
- Captcha решение интеграция
- Rate limiting 1 req/5sec
- Geo-targeting для Крыма

#### 2.3 Суточно.ру парсер (sutochno/)
**Особенности:**
- API-first подход где возможно
- Fallback на web scraping
- Real-time availability sync
- Price change monitoring

#### 2.4 ЦИАН парсер (cian/)
**Особенности:**
- GraphQL endpoints
- Rental category filtering
- Photos bulk download
- Contact info extraction

#### 2.5 Import по ссылке
```typescript
class LinkImportService {
  async importByLink(url: string, userId: string) {
    const parser = this.getParserForUrl(url)
    const property = await parser.parseProperty(url)
    
    // AI обогащение данных
    const enhanced = await this.aiPipeline.enhance(property)
    
    // Создание объявления
    return this.createProperty(enhanced, userId)
  }
  
  detectPlatform(url: string): 'avito' | 'sutochno' | 'cian' | 'unknown' {
    // URL pattern matching
  }
}
```

### 🆕 3. Self-Service Onboarding (onboarding/)

#### 3.1 Многошаговый Flow
```typescript
interface OnboardingStep {
  id: string
  title: string
  description: string
  component: React.ComponentType
  validation: (data: any) => Promise<boolean>
  aiAssistance?: boolean
}

const ONBOARDING_STEPS: OnboardingStep[] = [
  {
    id: 'welcome',
    title: 'Добро пожаловать в OpenCrimea',
    description: 'Создайте свое первое объявление за 5 минут',
    component: WelcomeStep,
  },
  {
    id: 'import_or_create',
    title: 'Импорт или создание',
    description: 'Импортируйте с других платформ или создайте новое',
    component: ImportOrCreateStep,
    aiAssistance: true
  },
  {
    id: 'property_details',
    title: 'Детали недвижимости',
    component: PropertyDetailsStep,
    aiAssistance: true
  },
  {
    id: 'photos_upload',
    title: 'Фотографии',
    description: 'AI поможет выбрать лучшие фото',
    component: PhotosUploadStep,
    aiAssistance: true
  },
  {
    id: 'pricing_setup',
    title: 'Ценообразование',
    description: 'AI предложит оптимальную цену',
    component: PricingStep,
    aiAssistance: true
  },
  {
    id: 'verification',
    title: 'Верификация',
    component: VerificationStep
  },
  {
    id: 'completion',
    title: 'Готово!',
    component: CompletionStep
  }
]
```

#### 3.2 AI Assistant для onboarding
```typescript
class OnboardingAssistant {
  async getStepGuidance(step: string, userData: any) {
    const context = this.buildContext(step, userData)
    const guidance = await this.llm.generateGuidance(context)
    
    return {
      tips: guidance.tips,
      warnings: guidance.warnings,
      suggestions: guidance.suggestions,
      nextSteps: guidance.nextSteps
    }
  }
  
  async validateStepData(step: string, data: any) {
    return this.validator.validate(step, data)
  }
}
```

#### 3.3 Gamification элементы
```typescript
interface OnboardingProgress {
  currentStep: number
  totalSteps: number
  completedSteps: string[]
  achievements: Achievement[]
  xp: number
}

interface Achievement {
  id: string
  title: string
  description: string
  icon: string
  xp: number
  unlockedAt: Date
}
```

### 4. Расширенные модули

#### 4.1 Недвижимость (properties/) - Enhanced
**Новые функции:**
- AI-генерация описаний
- Автоматическая категоризация
- Smart pricing recommendations
- Photo quality scoring
- SEO meta generation

#### 4.2 Чат (chat/) - AI-Powered
**Новые функции:**
- AI-асистент встроен в чат
- Smart notifications prioritization
- Automated FAQ responses  
- Sentiment analysis для поддержки
- Language detection и auto-translation

#### 4.3 Поиск (search/) - Semantic Search
**Новые функции:**
- Vector-based semantic search
- AI-powered recommendations
- Natural language queries
- Image similarity search
- Smart filters prediction

### 🆕 5. Data Pipeline и синхронизация

#### 5.1 Real-time синхронизация
```typescript
class SyncService {
  async syncParsedProperties() {
    const updates = await this.parseService.getUpdates()
    
    for (const update of updates) {
      await this.processUpdate(update)
      await this.aiPipeline.reprocess(update.propertyId)
    }
  }
  
  async detectPriceChanges() {
    // Monitor competitor pricing
    // Update AI pricing models
    // Notify hosts about opportunities
  }
}
```

#### 5.2 Дедубликация
```typescript
class DeduplicationService {
  async findDuplicates(property: ParsedProperty) {
    // Semantic similarity using embeddings
    const embeddings = await this.embedding.generate(property)
    const similar = await this.vectorDB.search(embeddings, threshold: 0.85)
    
    return this.rankSimilarity(similar)
  }
}
```

## Инфраструктура AI-First (с учетом санкций)

### Российские AI-провайдеры

#### AI Services:
- **Yandex Cloud AI**: GPT-like модели, computer vision
- **SberCloud ML**: Готовые ML модели 
- **VK Cloud AI**: OCR, speech recognition
- **Локальные модели**: Saiga, GigaChat alternatives

#### Основной выбор: Hybrid Cloud
**Преимущества:**
- Локальные модели для базовых задач
- Cloud AI для сложных операций
- Fallback стратегия при ограничениях
- Cost optimization

### AI Infrastructure:
```yaml
AI Services:
  ├── Model Serving (TensorFlow Serving)
  ├── Feature Store (Redis + PostgreSQL)
  ├── Vector Database (Qdrant)
  ├── ML Pipeline (Kubeflow)
  └── Model Registry (MLflow)

GPU Compute:
  ├── NVIDIA A100 (для training)
  ├── T4 instances (для inference)
  └── CPU fallback (local models)
```

### Архитектура развертывания AI-Enhanced:

```yaml
Production:
  Load Balancer (ALB + AI rate limiting)
  ├── Frontend (2x instances)
  ├── Backend (3x instances)
  ├── AI Services (GPU instances)
  │   ├── Model Serving (3x replicas)
  │   ├── Background Processing (BullMQ)
  │   └── Vector Search (Qdrant cluster)
  ├── Parsing Infrastructure
  │   ├── Chromium Pool (5x instances)
  │   ├── Proxy Rotation Service
  │   └── Anti-Bot Detection
  ├── Redis Cluster (3x nodes)
  └── PostgreSQL (Primary + 2x Read Replicas)

ML Pipeline:
  ├── Feature Engineering
  ├── Model Training (scheduled)
  ├── A/B Testing Framework
  └── Performance Monitoring
```

## CI/CD Pipeline с MLOps

### GitHub Actions Workflow Enhanced:

```yaml
# .github/workflows/main.yml
name: AI-Enhanced CI/CD Pipeline

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
      - name: Test AI Models
        run: pnpm test:ai-models

  ml-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Model Performance Tests
        run: python -m pytest ml_tests/
      - name: Data Validation
        run: great-expectations checkpoint run
      - name: Model Drift Detection
        run: python scripts/check_model_drift.py

  build:
    needs: [test, ml-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker images
        run: |
          docker build -t opencrimea-app:${{ github.sha }} .
          docker build -t opencrimea-ai:${{ github.sha }} ./ai-services/
      - name: Build ML Models
        run: python scripts/build_models.py
      - name: Push to registry
        run: |
          docker push opencrimea-app:${{ github.sha }}
          docker push opencrimea-ai:${{ github.sha }}

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to staging
        run: kubectl apply -f k8s/staging/
      - name: Run AI Integration Tests
        run: python scripts/ai_integration_tests.py

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to production
        run: kubectl apply -f k8s/production/
      - name: Gradual AI Model Rollout
        run: python scripts/gradual_rollout.py
```

### MLOps Pipeline:
```yaml
# Model Training Pipeline
ml-pipeline:
  schedule:
    - cron: '0 2 * * 1' # Weekly retraining
  steps:
    - name: Data Collection
      run: python scripts/collect_training_data.py
    - name: Feature Engineering  
      run: python scripts/feature_engineering.py
    - name: Model Training
      run: python scripts/train_models.py
    - name: Model Validation
      run: python scripts/validate_models.py
    - name: Model Registration
      run: python scripts/register_model.py
    - name: A/B Test Setup
      run: python scripts/setup_ab_test.py
```

## Структура GitHub репозитория AI-Enhanced

### Monorepo структура v2.0:
```
opencrimea/
├── .github/
│   ├── workflows/         # CI/CD + MLOps
│   ├── ISSUE_TEMPLATE/    # Шаблоны issues
│   └── PULL_REQUEST_TEMPLATE.md
├── apps/
│   ├── web/              # Frontend (Next.js)
│   ├── api/              # Backend (Fastify)
│   ├── admin/            # Admin panel
│   └── ai-services/      # 🆕 AI microservices
├── packages/
│   ├── shared/           # Общие типы и утилиты
│   ├── ui/               # UI компоненты
│   ├── ai/               # 🆕 AI utilities и types
│   └── config/           # Конфиги (ESLint, Prettier)
├── ml/                   # 🆕 ML Pipeline
│   ├── models/           # Model definitions
│   ├── training/         # Training scripts
│   ├── data/             # Data processing
│   ├── experiments/      # Jupyter notebooks
│   └── evaluation/       # Model evaluation
├── parsers/              # 🆕 Web scraping modules
│   ├── avito/
│   ├── sutochno/
│   ├── cian/
│   └── common/
├── infrastructure/
│   ├── docker/           # Dockerfiles
│   ├── k8s/              # Kubernetes манифесты
│   ├── terraform/        # Infrastructure as Code
│   └── ai-infra/         # 🆕 AI infrastructure
├── docs/
│   ├── api/              # API документация
│   ├── ai/               # 🆕 AI pipeline docs
│   ├── deployment/       # Деплой инструкции
│   └── contributing/     # Контрибьюторам
├── scripts/
│   ├── dev/              # Development скрипты
│   ├── build/            # Build скрипты
│   ├── migration/        # DB миграции
│   └── ml/               # 🆕 ML scripts
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   └── ai/               # 🆕 AI model tests
├── docker-compose.yml    # Local development
├── docker-compose.ai.yml # 🆕 AI services locally
├── package.json          # Root package.json
├── pnpm-workspace.yaml   # PNPM workspace
├── mlproject             # 🆕 MLflow project
├── README.md
├── CONTRIBUTING.md
├── AI_ETHICS.md         # 🆕 AI ethics guidelines
├── CODE_OF_CONDUCT.md
└── LICENSE
```

## Безопасность AI-Enhanced

### AI Specific Security:
- **Model Security**: Защита от adversarial attacks
- **Data Privacy**: Псевдонимизация в AI training data
- **Prompt Injection**: Защита от LLM prompt injection
- **Model Watermarking**: Защита интеллектуальной собственности
- **Bias Detection**: Monitoring для AI fairness

### Security Headers + AI:
```javascript
// AI-enhanced security middleware
app.register(require('@fastify/helmet'), {
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      connectSrc: ["'self'", "*.openai.com", "*.yandex.net"],
      imgSrc: ["'self'", "data:", "*.yandex.net", "ai.generated"],
      workerSrc: ["'self'", "blob:"], // Web Workers для AI
    }
  }
})

// AI rate limiting
app.register(require('@fastify/rate-limit'), {
  max: 100, // Standard requests
  timeWindow: '1 minute',
  keyGenerator: (request) => request.user?.id || request.ip,
  // Special limits for AI endpoints
  onRequest: async (request, reply) => {
    if (request.url.startsWith('/api/ai/')) {
      // Stricter limits for AI operations
      await aiRateLimiter.consume(request.user?.id || request.ip)
    }
  }
})
```

## Мониторинг и логирование AI-Enhanced

### AI Метрики:
- **Model Performance**: Accuracy, latency, throughput
- **Business Impact**: Conversion improvement, user satisfaction  
- **Resource Usage**: GPU utilization, memory consumption
- **Data Quality**: Input validation, drift detection

### AI Логирование:
```typescript
// Structured AI logs
aiLogger.info('AI prediction completed', {
  modelName: 'price-prediction-v2',
  modelVersion: '2024.03.15',
  inputFeatures: features,
  prediction: result.price,
  confidence: result.confidence,
  executionTime: endTime - startTime,
  userId: user.id,
  traceId: req.traceId
})
```

### AI Алерты:
- Model accuracy drop > 5%
- AI service response time > 5s
- GPU memory usage > 90%
- Unusual prediction patterns detected
- Data drift detected
- Model bias metrics exceeded

## Производительность AI-Enhanced

### AI Performance Targets:
- **AI Photo Moderation**: < 2s per image
- **Text Generation**: < 3s for descriptions
- **Price Prediction**: < 500ms
- **Chatbot Response**: < 1s
- **Semantic Search**: < 800ms

### AI Оптимизации:
- Model quantization для production
- Batch processing для bulk operations
- GPU memory optimization
- Model caching strategies
- Async AI processing где возможно

## Масштабирование AI-First (Roadmap)

### Phase 1: AI MVP (0-10k users, 1k properties)
- Базовые AI модели
- Простой парсинг 3х сайтов
- Onboarding с AI assistance
- Single GPU instance

### Phase 2: AI Growth (10k-100k users, 10k properties)
- Advanced ML models
- Real-time sync парсинга
- A/B testing AI features
- Auto-scaling GPU instances
- Vector database optimization

### Phase 3: AI Scale (100k+ users, 50k+ properties)
- Custom trained models
- Multi-modal AI (text + images + video)
- Federated learning
- Edge AI deployment
- Advanced personalization

### Phase 4: AI Platform (200k+ users)
- AI Marketplace для третьих лиц
- API monetization
- White-label AI solutions
- Advanced analytics и insights

## AI Ethics и Compliance

### Этические принципы:
- **Transparency**: Понятные AI решения для пользователей
- **Fairness**: Отсутствие дискриминации в ценообразовании
- **Privacy**: Минимизация персональных данных в AI
- **Human Oversight**: Возможность человеческого вмешательства
- **Explainability**: Объяснимые AI рекомендации

### AI Compliance:
```typescript
// AI decision logging для audit trail
class AIDecisionLogger {
  async logDecision(decision: AIDecision) {
    await this.auditDB.insert({
      timestamp: new Date(),
      modelName: decision.model,
      input: decision.sanitizedInput, // No PII
      output: decision.result,
      confidence: decision.confidence,
      explanation: decision.reasoning,
      userId: decision.userId,
      canAppeal: true
    })
  }
}
```

## Cost Optimization AI

### AI Cost Management:
- **Tiered AI Services**: Разные модели для разных задач
- **Local Models**: Простые задачи на локальных моделях
- **Batch Processing**: Group AI operations
- **Cache Strategies**: Кеширование AI результатов
- **Resource Scheduling**: Off-peak training

### Cost Monitoring:
```typescript
interface AICostMetrics {
  tokenUsage: number
  gpuHours: number
  storageGb: number
  apiCalls: number
  totalCost: number
  costPerUser: number
  roi: number // Revenue impact от AI features
}
```

---

## Заключение v2.0

Данная AI-первая архитектура обеспечивает:

✅ **Автоматизированный контент-пайплайн** с парсингом и AI-обработкой  
✅ **Умный onboarding** с AI-асситентом и gamification  
✅ **Production-ready AI** с proper MLOps и мониторингом  
✅ **Масштабируемость** от MVP до AI Platform  
✅ **Соответствие ограничениям** российские AI-провайдеры и compliance  
✅ **Open-source friendly** с прозрачными AI решениями  

Архитектура позволяет команде создать современную AI-first платформу, которая автоматизирует рутинные операции, улучшает user experience и обеспечивает конкурентное преимущество через intelligent automation.