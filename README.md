# REGIONALNE SERWISY - KOMPLETNY MASTERPLAN SYSTEMU

**Multi-Tenant Regional Portal Management System z Ultra UX Frontend**

**Wersja:** 3.0 (Zintegrowana)  
**Data:** 12 lutego 2026  
**Autor:** System Architect  
**Status:** Ready for Implementation

---

## SPIS TRESCI

1. [Wizja i Architektura Systemu](#1-wizja-i-architektura-systemu)
2. [Struktura Domen i Routing](#2-struktura-domen-i-routing)
3. [Baza Danych - Architektura Multi-Tenant](#3-baza-danych---architektura-multi-tenant)
4. [System Uprawnień RBAC](#4-system-uprawnień-rbac)
5. [API - Specyfikacja Kompletna](#5-api---specyfikacja-kompletna)
6. [Frontend - UI/UX Design System](#6-frontend---uiux-design-system)
7. [Komponenty Frontend (Na podstawie 4torun.pl)](#7-komponenty-frontend-na-podstawie-4torunpl)
8. [Panel Administracyjny - Centralny](#8-panel-administracyjny---centralny)
9. [Panel Administracyjny - Per Serwis](#9-panel-administracyjny---per-serwis)
10. [System Motywów i Personalizacji](#10-system-motywów-i-personalizacji)
11. [System Scrapingu i Cron Jobs](#11-system-scrapingu-i-cron-jobs)
12. [SEO i Struktury Danych](#12-seo-i-struktury-danych)
13. [Monitoring, Logi i Raportowanie](#13-monitoring-logi-i-raportowanie)
14. [Deployment i Dodawanie Nowych Portali](#14-deployment-i-dodawanie-nowych-portali)
15. [Wdrożenie - Plan Etapowy](#15-wdrożenie---plan-etapowy)
16. [Bezpieczeństwo](#16-bezpieczeństwo)

---

## 1. WIZJA I ARCHITEKTURA SYSTEMU

### 1.1 Wizja Systemu

System **Regionalne Serwisy** to zaawansowana platforma multi-tenant pozwalająca na zarządzanie wieloma niezależnymi serwisami regionalnymi z jednego centralnego panelu administracyjnego.

### 1.2 Diagram Architektury High-Level

```mermaid
graph TB
    subgraph "System Users"
        UA[Super Admin]
        UB[Domain Admin]
        UC[Editor]
        UD[Moderator]
        UE[User]
    end

    subgraph "Panel Centralny"
        PC[serwisy-lokalne-sterowanie.pl]
        PC_API[/API Gateway/]
        PC_DB[(PostgreSQL<br/>Schema: public)]
    end

    subgraph "Domeny Regionalne"
        D1[4torun.pl<br/>Next.js + React]
        D2[4bydgoszcz.pl]
        D3[4warszawa.pl]
        D4[...]
    end

    subgraph "Shared Infrastructure"
        REDIS[(Redis Cache)]
        RMQ[(RabbitMQ)]
        ES[(Elasticsearch)]
        MINIO[(MinIO Storage)]
    end

    subgraph "Python Workers"
        W1[Scraper Worker]
        W2[Cron Worker]
        W3[Email Worker]
    end

    UA --> PC
    UB --> PC
    UB --> D1
    UB --> D2
    UC --> D1
    UD --> D1
    UE --> D1
    UE --> D2
    UE --> D3

    PC --> PC_API
    PC_API --> PC_DB
    PC_API --> REDIS
    PC_API --> RMQ
    
    D1 --> PC_DB
    D2 --> PC_DB
    
    PC --> W1
    RMQ --> W1
    W1 --> D1
    W1 --> MINIO
```

### 1.3 Główne Założenia

| Aspekt | Opis |
|--------|------|
| **Multi-Tenancy** | Każdy serwis to oddzielny tenant z własnym schema PostgreSQL |
| **Centralne Zarządzanie** | Jeden panel do sterowania wszystkimi serwisami |
| **Współdzielenie Zasobów** | Wspólne szablony, moduły, źródła danych |
| **Skalowalność** | Łatwe dodawanie nowych serwisów (automatyczny deployment) |
| **Elastyczność** | Możliwość indywidualnej konfiguracji każdego serwisu (kolory, logo, motyw) |

### 1.4 Struktura Katalogów na Hosting

```
/home/host988956/
├── domains/
│   ├── serwisy-lokalne-sterowanie.pl/    # Panel centralny (Next.js)
│   │   ├── public_html/
│   │   │   ├── .next/                    # Build output
│   │   │   ├── src/
│   │   │   │   ├── app/                  # Next.js App Router
│   │   │   │   ├── components/
│   │   │   │   └── lib/
│   │   │   └── package.json
│   │   ├── logs/
│   │   └── backup/
│   │
│   ├── 4torun.pl/                        # Serwis regionalny (Next.js)
│   │   ├── public_html/
│   │   │   ├── .next/
│   │   │   ├── src/
│   │   │   │   ├── app/
│   │   │   │   │   ├── page.tsx          # Home
│   │   │   │   │   ├── [type]/           # CPT routes
│   │   │   │   │   └── admin/            # Panel serwisu
│   │   │   │   ├── components/
│   │   │   │   │   ├── layout/           # Header, Footer
│   │   │   │   │   ├── content/          # NewsCard, Hero
│   │   │   │   │   └── ui/               # shadcn/ui
│   │   │   │   ├── hooks/
│   │   │   │   └── lib/
│   │   │   ├── public/
│   │   │   │   ├── uploads/              # User uploads
│   │   │   │   └── images/
│   │   │   ├── .env.local                # Konfiguracja DB
│   │   │   ├── next.config.js
│   │   │   └── tailwind.config.ts
│   │   ├── logs/
│   │   └── backup/
│   │
│   └── [nowa-domena.pl]/                 # Nowa domena (auto-created z template)
│       └── public_html/                  # Klon template/default/
│
├── shared/
│   ├── templates/
│   │   └── default/                      # Szablon do klonowania
│   ├── assets/                           # Wspólne grafiki, fonty
│   └── scripts/                          # Skrypty deploymentu
│
├── workers/                              # Python workers
│   ├── scraper/
│   ├── cron/
│   └── venv/
│
└── logs/                                 # Logi systemowe
```

---

## 2. STRUKTURA DOMEN I ROUTING

### 2.1 Konfiguracja Domen

| Domena | Typ | Technologia | Opis |
|--------|-----|-------------|------|
| `serwisy-lokalne-sterowanie.pl` | Panel Centralny | Next.js 14 + React 18 | Zarządzanie wszystkimi serwisami |
| `4torun.pl` | Serwis Regionalny | Next.js 14 + React 18 | Frontend publiczny + panel admina |
| `4bydgoszcz.pl` | Serwis Regionalny | Next.js 14 + React 18 | Inny tenant, ten sam kod |
| `[nowa-domena.pl]` | Serwis Regionalny | Next.js 14 + React 18 | Auto-deployment z template |

### 2.2 Routing Panelu Centralnego

```
# Dashboard
GET  /                                    # Dashboard główny systemu
GET  /dashboard                           # Statystyki wszystkich serwisów

# Zarządzanie Domenami
GET  /admin/domeny                        # Lista wszystkich domen
POST /admin/domeny                        # Dodaj nową domenę
GET  /admin/domeny/:domainId              # Szczegóły domeny
GET  /admin/domeny/:domainId/dashboard    # Dashboard konkretnej domeny
GET  /admin/domeny/:domainId/content/*    # Zarządzanie treścią domeny

# Masowe Operacje
POST /admin/mass/banners                  # Dodaj banner do wielu domen
POST /admin/mass/menus                    # Dodaj menu do wielu domen
POST /admin/mass/content                  # Dodaj treść do wielu domen

# Źródła Danych
GET  /admin/sources                       # Lista źródeł scrapingu
POST /admin/sources                       # Dodaj źródło
POST /admin/sources/:sourceId/run         # Uruchom scraping

# Cron Jobs
GET  /admin/cron                          # Lista zadań cron
POST /admin/cron                          # Dodaj zadanie

# Użytkownicy i Uprawnienia
GET  /admin/users                         # Lista użytkowników systemu
GET  /admin/roles                         # Role i uprawnienia

# Szablony i Motywy
GET  /admin/templates                     # Lista szablonów
POST /admin/templates/:templateId/apply   # Zastosuj do domen

# Logi
GET  /admin/logs                          # Logi systemowe
```

### 2.3 Routing Serwisu Regionalnego (Frontend Publiczny)

```
# Strony Główne
GET  /                                    # Home Page
GET  /wiadomosci                          # Archiwum wiadomości
GET  /wiadomosci/:slug                    # Pojedyncza wiadomość
GET  /kronika-policyjna                   # Archiwum kroniki
GET  /kronika-policyjna/:slug             # Pojedynczy wpis
GET  /firmy                               # Katalog firm
GET  /firmy/:category/:slug               # Profil firmy
GET  /ogloszenia                          # Ogłoszenia
GET  /praca                               # Oferty pracy
GET  /nekrologi                           # Nekrologi
GET  /przewodnik                          # Przewodnik
GET  /ludzie                              # Ludzie urodzeni w mieście
GET  /pogoda                              # Szczegółowa pogoda

# Panel Admina Serwisu
GET  /admin                               # Dashboard serwisu
GET  /admin/posts                         # Lista wpisów
GET  /admin/posts/create                  # Tworzenie wpisu
GET  /admin/posts/:id/edit                # Edycja wpisu
GET  /admin/settings                      # Ustawienia serwisu
GET  /admin/settings/theme                # Edytor motywu
GET  /admin/settings/seo                  # Ustawienia SEO
GET  /admin/media                         # Biblioteka mediów
```

---

## 3. BAZA DANYCH - ARCHITEKTURA MULTI-TENANT

### 3.1 Strategia Multi-Tenancy: Shared Database, Separate Schema per Tenant

```mermaid
graph TB
    subgraph "PostgreSQL Database"
        subgraph "Schema: public (Centrala)"
            T1[users]
            T2[roles]
            T3[permissions]
            T4[domains]
            T5[sources]
            T6[cron_jobs]
            T7[templates]
            T8[modules]
        end
        
        subgraph "Schema: tenant_4torun_pl"
            P1[posts]
            P2[categories]
            P3[tags]
            P4[comments]
            P5[banners]
            P6[companies]
            P7[jobs]
            P8[obituaries]
        end
        
        subgraph "Schema: tenant_4bydgoszcz_pl"
            B1[posts]
            B2[categories]
            B3[comments]
        end
    end

    T4 -.->|konfiguruje| P1
    T4 -.->|konfiguruje| B1
    T5 -.->|zasilają| P1
    T5 -.->|zasilają| B1
```

### 3.2 Schemat SQL - Schema: public (Centrala)

```sql
-- Tabela użytkowników systemu (globalna)
CREATE TABLE public.users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    avatar_url TEXT,
    phone VARCHAR(20),
    is_active BOOLEAN DEFAULT true,
    is_super_admin BOOLEAN DEFAULT false,
    last_login_at TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela domen/serwisów
CREATE TABLE public.domains (
    id VARCHAR(100) PRIMARY KEY, -- np. '4torun.pl'
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    city VARCHAR(100) NOT NULL,
    region VARCHAR(100),
    is_active BOOLEAN DEFAULT true,
    schema_name VARCHAR(100) NOT NULL,
    theme_config JSONB DEFAULT '{}',
    seo_settings JSONB DEFAULT '{}',
    features_enabled JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela źródeł danych (scraping)
CREATE TABLE public.sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'rss', 'api', 'html'
    url TEXT NOT NULL,
    parser_config JSONB NOT NULL,
    mapping_config JSONB NOT NULL,
    schedule VARCHAR(100) DEFAULT '0 */6 * * *',
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela powiązań źródło-domena
CREATE TABLE public.source_domains (
    source_id UUID REFERENCES public.sources(id),
    domain_id VARCHAR(100) REFERENCES public.domains(id),
    PRIMARY KEY (source_id, domain_id)
);
```

### 3.3 Schemat SQL - Schema: tenant (Pojedynczy Serwis)

```sql
-- Wpisy (Custom Post Types)
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_type VARCHAR(50) NOT NULL, -- 'wiadomosci', 'kronika-policyjna', etc.
    title VARCHAR(500) NOT NULL,
    slug VARCHAR(500) UNIQUE NOT NULL,
    excerpt TEXT,
    content TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'draft',
    featured BOOLEAN DEFAULT false,
    view_count INTEGER DEFAULT 0,
    rating_average DECIMAL(3,2) DEFAULT 0,
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    source_url TEXT,
    external_id VARCHAR(255)
);

-- Kategorie
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    post_type VARCHAR(50) NOT NULL,
    color VARCHAR(7),
    icon VARCHAR(50),
    sort_order INTEGER DEFAULT 0
);

-- Bannery
CREATE TABLE banners (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    position VARCHAR(50) NOT NULL, -- 'header', 'sidebar', 'footer'
    content TEXT NOT NULL,
    image_url TEXT,
    link_url TEXT,
    start_date TIMESTAMP,
    end_date TIMESTAMP,
    is_active BOOLEAN DEFAULT true
);

-- Menu
CREATE TABLE menus (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    location VARCHAR(50) NOT NULL, -- 'header', 'footer', 'mobile'
    is_active BOOLEAN DEFAULT true
);

CREATE TABLE menu_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    menu_id UUID REFERENCES menus(id),
    parent_id UUID REFERENCES menu_items(id),
    title VARCHAR(200) NOT NULL,
    url TEXT NOT NULL,
    sort_order INTEGER DEFAULT 0
);
```

---

## 4. SYSTEM UPRAWNIEŃ RBAC

### 4.1 Hierarchia Ról

```mermaid
graph TD
    A[Super Admin<br/>System Level] --> B[Domain Admin<br/>Domain Level]
    B --> C[Editor<br/>Content Level]
    C --> D[Moderator<br/>Moderation Level]
    D --> E[User<br/>Public Level]
    
    A -.->|all domains| F[4torun.pl]
    A -.->|all domains| G[4bydgoszcz.pl]
    B -.->|assigned domains| F
    C -.->|assigned domains| F
```

### 4.2 Role i Uprawnienia

| Rola | Level | Zakres | Główne Uprawnienia |
|------|-------|--------|-------------------|
| **Super Admin** | 0 | Cały system | Wszystkie domeny, użytkownicy, ustawienia globalne |
| **Admin Domeny** | 1 | Przypisane domeny | Zarządzanie treścią, użytkownikami, ustawieniami domeny |
| **Redaktor** | 2 | Content | Tworzenie, edycja, publikowanie wpisów |
| **Moderator** | 3 | Moderacja | Komentarze, oceny, zgłoszenia |
| **Użytkownik** | 4 | Public | Komentowanie, ocenianie, profil |

---



## 5. API - SPECYFIKACJA KOMPLETNA

### 5.1 Endpointy Autentykacji

```yaml
POST /api/v1/auth/login
Request:
  body:
    email: string
    password: string
Response:
  200:
    access_token: string (JWT)
    refresh_token: string
    expires_in: integer
    user: object

GET /api/v1/auth/me
Headers:
  Authorization: Bearer {token}
Response:
  200:
    id: uuid
    email: string
    first_name: string
    last_name: string
    roles: array
    permissions: array
```

### 5.2 Endpointy Zarządzania Domenami

```yaml
GET /api/v1/domains
Response:
  200:
    data:
      - id: string
        name: string
        city: string
        is_active: boolean
        posts_count: integer

POST /api/v1/domains
Request:
  body:
    name: string (required)
    slug: string (required)
    city: string (required)
    admin_email: string
Response:
  201:
    domain: object
    message: "Domain created. Deployment in progress..."

GET /api/v1/domains/:domainId/content/posts
Query:
  type: string
  status: string
  page: integer
  limit: integer
Response:
  200:
    data: array of posts
    meta: pagination

POST /api/v1/domains/:domainId/content/posts
Request:
  body:
    post_type: string
    title: string
    content: string
    status: string
Response:
  201:
    post: object
```

### 5.3 Endpointy Scrapingu

```yaml
GET /api/v1/sources
Response:
  200:
    data: array of sources

POST /api/v1/sources
Request:
  body:
    name: string
    type: string (rss, api, html)
    url: string
    parser_config: object
    mapping_config: object
    domains: array of string
Response:
  201:
    source: object

POST /api/v1/sources/:sourceId/run
Response:
  202:
    job_id: uuid
    status: "queued"
```

---

## 6. FRONTEND - UI/UX DESIGN SYSTEM

### 6.1 Design Philosophy

**Inspiracja:** 4torun.pl - Ciepła, przyjazna estetyka regionalnego portalu  
**Cel:** Nowoczesny, szybki, dostępny (WCAG 2.1 AA), responsywny (Mobile First)

**Kluczowe Założenia:**
- Czytelna typografia dla seniorów (16px+ base)
- Wysoki kontrast dla dostępności
- Szybkie ładowanie (Core Web Vitals)
- Płynne animacje (60fps)
- Intuicyjna nawigacja (max 3 kliknięcia do treści)

### 6.2 System Kolorów (Konfigurowalny per Domena)

```css
:root {
  /* Primary - Główny kolor marki */
  --color-primary-50: #fef2f2;
  --color-primary-100: #fee2e2;
  --color-primary-200: #fecaca;
  --color-primary-300: #fca5a5;
  --color-primary-400: #f87171;
  --color-primary-500: #ef4444;  /* Główny - czerwony jak 4torun */
  --color-primary-600: #dc2626;
  --color-primary-700: #b91c1c;  /* Header/Footer */
  --color-primary-800: #991b1b;
  --color-primary-900: #7f1d1d;

  /* Secondary - Akcent */
  --color-secondary-500: #f97316;
  --color-secondary-600: #ea580c;

  /* Neutral - Tekst i tła */
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-600: #4b5563;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  /* Semantic */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  /* Background */
  --bg-primary: #ffffff;
  --bg-header: var(--color-primary-700);
  --bg-footer: var(--color-primary-700);

  /* Typography */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-serif: 'Merriweather', Georgia, serif;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);

  /* Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
}
```

### 6.3 Konfiguracja Motywu w Panelu Admina

```typescript
interface ThemeConfig {
  colors: {
    primary: string;        // #dc2626
    primaryDark: string;    // #b91c1c
    secondary: string;      // #f97316
    background: string;     // #ffffff
    text: string;          // #1f2937
  };
  typography: {
    headingFont: 'Inter' | 'Merriweather' | 'Roboto';
    bodyFont: 'Inter' | 'Open Sans' | 'Lato';
    baseSize: 16 | 17 | 18;
  };
  layout: {
    maxWidth: '1280px' | '1440px' | '100%';
    sidebarPosition: 'left' | 'right' | 'none';
    cardStyle: 'flat' | 'elevated' | 'bordered';
  };
  header: {
    style: 'default' | 'sticky' | 'transparent';
    height: 60 | 72 | 84;
    showSearch: boolean;
    showWeather: boolean;
  };
  assets: {
    logo: string;
    favicon: string;
    ogImage: string;
  };
}
```

### 6.4 Struktura Komponentów Next.js

```
src/
├── app/                          # Next.js App Router
│   ├── layout.tsx                # Root layout z ThemeProvider
│   ├── page.tsx                  # Strona główna
│   ├── globals.css               # Globalne style + CSS variables
│   ├── [type]/                   # Dynamic routes dla CPT
│   │   ├── page.tsx              # Archiwum
│   │   └── [slug]/
│   │       └── page.tsx          # Pojedynczy wpis
│   └── admin/                    # Panel admina serwisu
│       ├── page.tsx              # Dashboard
│       ├── posts/
│       ├── settings/
│       └── layout.tsx
│
├── components/
│   ├── layout/                   # Layout components
│   │   ├── Header.tsx            # Header z menu
│   │   ├── Footer.tsx            # Stopka
│   │   ├── Sidebar.tsx           # Sidebar
│   │   ├── InfoBar.tsx           # Data, pogoda, imieniny
│   │   └── Navigation.tsx        # Menu nawigacyjne
│   │
│   ├── content/                  # Content components
│   │   ├── NewsCard.tsx          # Karta wiadomości
│   │   ├── NewsGrid.tsx          # Grid wiadomości
│   │   ├── HeroSection.tsx       # Sekcja hero
│   │   ├── CategoryGrid.tsx      # Grid kategorii
│   │   ├── JobCard.tsx           # Karta oferty pracy
│   │   ├── ObituaryCard.tsx      # Karta nekrologu
│   │   ├── CommentSection.tsx    # Sekcja komentarzy
│   │   └── WeatherWidget.tsx     # Widget pogody
│   │
│   └── ui/                       # shadcn/ui komponenty
│       ├── button.tsx
│       ├── card.tsx
│       ├── input.tsx
│       └── dialog.tsx
│
├── lib/
│   ├── api.ts                    # Klient API
│   ├── utils.ts                  # Helpers
│   └── constants.ts              # Stałe
│
└── hooks/
    ├── useTheme.ts
    ├── usePosts.ts
    └── useWeather.ts
```

---

## 7. KOMPONENTY FRONTEND (NA PODSTAWIE 4TORUN.PL)

### 7.1 Header (Wersja Desktop + Mobile)

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  [LOGO: 4TORUŃ]                    [🔍] [✉️]                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│  🏠 WIADOMOŚCI  POLICJA  FIRMY  OGŁOSZENIA  PRACA  POGODA ▼  NEKROLOGI...     │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Implementacja:**
```typescript
interface HeaderProps {
  variant: 'default' | 'sticky' | 'transparent';
  logo: {
    src: string;
    alt: string;
    width: number;
    height: number;
  };
  navigation: NavItem[];
  showSearch: boolean;
  showSocial: boolean;
}

interface NavItem {
  label: string;
  href: string;
  icon?: string;
  children?: NavItem[];
  highlight?: boolean;
}

const Header: React.FC<HeaderProps> = ({ logo, navigation }) => {
  const [isScrolled, setIsScrolled] = useState(false);
  
  useEffect(() => {
    const handleScroll = () => setIsScrolled(window.scrollY > 50);
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return (
    <header className={cn(
      "transition-all duration-300",
      isScrolled ? "sticky top-0 shadow-md z-50" : ""
    )}>
      {/* Top bar z logo */}
      <div className="bg-white py-4">
        <div className="container mx-auto px-4 flex items-center justify-between">
          <Link href="/">
            <Image src={logo.src} alt={logo.alt} width={logo.width} height={logo.height} />
          </Link>
          <div className="flex items-center gap-4">
            <SearchButton />
            <SocialLinks />
          </div>
        </div>
      </div>
      
      {/* Navigation bar - kolor primary-700 */}
      <nav className="bg-primary-700 text-white">
        <div className="container mx-auto px-4">
          <ul className="flex items-center gap-1">
            {navigation.map((item) => (
              <li key={item.href} className="relative group">
                <Link 
                  href={item.href}
                  className="flex items-center gap-1 px-4 py-3 hover:bg-primary-800 transition-colors"
                >
                  {item.icon && <Icon name={item.icon} className="w-4 h-4" />}
                  <span>{item.label}</span>
                  {item.children && <ChevronDown className="w-4 h-4" />}
                </Link>
                
                {/* Dropdown */}
                {item.children && (
                  <ul className="absolute top-full left-0 bg-white text-gray-800 shadow-lg rounded-b-lg overflow-hidden hidden group-hover:block min-w-[200px]">
                    {item.children.map((child) => (
                      <li key={child.href}>
                        <Link 
                          href={child.href}
                          className="block px-4 py-2 hover:bg-gray-100"
                        >
                          {child.label}
                        </Link>
                      </li>
                    ))}
                  </ul>
                )}
              </li>
            ))}
          </ul>
        </div>
      </nav>
    </header>
  );
};
```

### 7.2 Info Bar (Data, Imieniny, Pogoda, Jakość Powietrza)

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  Czwartek, 12 lutego 2026 r.                    Pogoda         Jakość Powietrza │
│  Imieniny: Eulalii, Radosława i Modesta         🌤️ 3.36°C      😊 Bardzo Dobra  │
│                                                 Ciśnienie: 982 hPa              │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Implementacja:**
```typescript
interface InfoBarProps {
  showDate?: boolean;
  showNameDay?: boolean;
  showWeather?: boolean;
  showAirQuality?: boolean;
}

const InfoBar: React.FC<InfoBarProps> = ({
  showDate = true,
  showNameDay = true,
  showWeather = true,
  showAirQuality = true
}) => {
  const { date, nameDay } = useDateAndNameday();
  const { weather } = useWeather();
  const { airQuality } = useAirQuality();
  
  return (
    <div className="bg-gray-50 border-b py-2">
      <div className="container mx-auto px-4">
        <div className="flex flex-wrap items-center justify-between gap-4 text-sm">
          {/* Data i Imieniny */}
          {showDate && (
            <div className="flex items-center gap-4">
              <span className="font-medium text-gray-900">
                {formatDate(date, 'EEEE, d MMMM yyyy')} r.
              </span>
              {showNameDay && nameDay && (
                <span className="text-gray-600">
                  Imieniny: <span className="text-primary-600">{nameDay}</span>
                </span>
              )}
            </div>
          )}
          
          {/* Pogoda i Jakość Powietrza */}
          <div className="flex items-center gap-6">
            {showWeather && weather && (
              <div className="flex items-center gap-3">
                <WeatherIcon condition={weather.condition} className="w-8 h-8" />
                <div>
                  <span className="font-medium">{weather.temp}°C</span>
                  <span className="text-gray-500 text-xs ml-2">
                    Ciśnienie: {weather.pressure} hPa
                  </span>
                </div>
              </div>
            )}
            
            {showAirQuality && airQuality && (
              <div className="flex items-center gap-2">
                <AirQualityIndicator index={airQuality.index} />
                <span className="font-medium" style={{ color: airQuality.color }}>
                  {airQuality.label}
                </span>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
};
```

### 7.3 Hero Section (Główna Wiadomość + Posty Boczne)

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  ┌─────────────────────────────┐  ┌─────────────────────────────────────────┐  │
│  │                             │  │  Walentynka dla Ciebie – wyjątkowa      │  │
│  │  [Duże Zdjęcie 16:9]        │  │  akcja w komunikacji miejskiej Torunia  │  │
│  │                             │  ├─────────────────────────────────────────┤  │
│  │  [Overlay z gradientem]     │  │  Budowa nowego pasa do skrętu w         │  │
│  │                             │  │  prawo na ulicy Kraszewskiego           │  │
│  │  Tytuł głównej wiadomości  │  └─────────────────────────────────────────┘  │
│  │  z nakładką na zdjęciu     │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │
│  │                             │  │ [Mini 1] │ │ [Mini 2] │ │ [Mini 3] │        │
│  └─────────────────────────────┘  └──────────┘ └──────────┘ └──────────┘        │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.4 Karta Wpisu (News Card) - Warianty

**Wariant Default:**
```typescript
interface NewsCardProps {
  post: Post;
  variant: 'default' | 'featured' | 'compact' | 'horizontal' | 'overlay';
  showImage?: boolean;
  showExcerpt?: boolean;
  showMeta?: boolean;
  imageAspectRatio?: '16:9' | '4:3' | '1:1';
}

const NewsCard: React.FC<NewsCardProps> = ({ 
  post, 
  variant = 'default',
  showImage = true,
  showExcerpt = false,
  showMeta = true 
}) => {
  const variants = {
    default: "flex flex-col",
    featured: "flex flex-col md:flex-row gap-6",
    compact: "flex flex-col",
    horizontal: "flex flex-row gap-4",
    overlay: "relative aspect-video"
  };
  
  return (
    <article className={cn(
      "group border rounded-lg overflow-hidden transition-all duration-300",
      "hover:shadow-lg hover:-translate-y-1",
      variants[variant]
    )}>
      {showImage && (
        <div className={cn(
          "relative overflow-hidden",
          variant === 'overlay' ? "absolute inset-0" : "aspect-video"
        )}>
          <Image
            src={post.featuredImage}
            alt={post.title}
            fill
            className="object-cover transition-transform duration-500 group-hover:scale-105"
          />
          {variant === 'overlay' && (
            <div className="absolute inset-0 bg-gradient-to-t from-black/80 via-black/40 to-transparent" />
          )}
          <span className="absolute top-3 left-3 px-2 py-1 bg-primary-600 text-white text-xs rounded">
            {post.category}
          </span>
        </div>
      )}
      
      <div className={cn(
        "p-4",
        variant === 'overlay' && "absolute bottom-0 left-0 right-0 text-white"
      )}>
        <h3 className={cn(
          "font-semibold line-clamp-2 group-hover:text-primary-600 transition-colors",
          variant === 'featured' ? "text-xl md:text-2xl" : "text-base"
        )}>
          {post.title}
        </h3>
        
        {showExcerpt && (
          <p className="mt-2 text-gray-600 line-clamp-2 text-sm">
            {post.excerpt}
          </p>
        )}
        
        {showMeta && (
          <div className="flex items-center gap-4 mt-3 text-sm text-gray-500">
            <span className="flex items-center gap-1">
              <Eye className="w-4 h-4" />
              {post.viewCount.toLocaleString()}
            </span>
            <span className="flex items-center gap-1">
              <MessageCircle className="w-4 h-4" />
              {post.commentCount}
            </span>
            <span className="flex items-center gap-1">
              <Star className="w-4 h-4" />
              {post.ratingAverage}
            </span>
            <span>{formatDate(post.publishedAt)}</span>
          </div>
        )}
      </div>
    </article>
  );
};
```

### 7.5 Sekcja Kategorii Firm (Grid z Ikonami)

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  KATEGORIE FIRM                                                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌────────────────────────┐ ┌────────────────────────┐ ┌──────────────────────┐│
│  │ 🦷 Dentysta Toruń      │ │ 💊 Apteka Toruń        │ │ 💰 Lombard Toruń     ││
│  └────────────────────────┘ └────────────────────────┘ └──────────────────────┘│
│  ┌────────────────────────┐ ┌────────────────────────┐ ┌──────────────────────┐│
│  │ 🌸 Kwiaciarnia Toruń   │ │ 🐾 Weterynarz Toruń    │ │ 📷 Fotograf Toruń    ││
│  └────────────────────────┘ └────────────────────────┘ └──────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.6 Nekrologi (Obituary Card) - Specjalny Komponent

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  Ś.P HONORATA SAWICKA                                        ✝️                │
│  zm. 19.05.2024 r.                                                            │
│  w wieku 65 lat                                                               │
│                                                                               │
│  Miejsce Uroczystości Pogrzebowej: Kościół w Toruniu                          │
│  Data Pogrzebu: 22.05.2024 r.                                                 │
│                                                                               │
│  🕯️ zapalonych zniczy (0)    szczegóły pogrzebu                               │
└─────────────────────────────────────────────────────────────────────────────────┘
```

**Implementacja:**
```typescript
interface ObituaryCardProps {
  obituary: {
    firstName: string;
    lastName: string;
    deathDate: Date;
    age: number;
    funeralLocation: string;
    funeralDate: Date;
    candlesCount: number;
  };
}

const ObituaryCard: React.FC<ObituaryCardProps> = ({ obituary }) => {
  return (
    <div className="border-2 border-gray-200 rounded-lg p-6 bg-white hover:border-gray-300 transition-colors">
      <div className="flex justify-between items-start">
        <div className="flex-1">
          <h3 className="text-xl font-serif text-gray-900 mb-2">
            Ś.P. {obituary.firstName.toUpperCase()} {obituary.lastName.toUpperCase()}
          </h3>
          
          <div className="space-y-1 text-gray-600 mb-4">
            <p>zm. {formatDate(obituary.deathDate, 'dd.MM.yyyy')} r.</p>
            <p>w wieku {obituary.age} lat</p>
          </div>
          
          <div className="bg-gray-50 p-3 rounded text-sm space-y-1">
            <p>
              <span className="font-medium">Miejsce Uroczystości:</span>
              {' '}{obituary.funeralLocation}
            </p>
            <p>
              <span className="font-medium">Data Pogrzebu:</span>
              {' '}{formatDate(obituary.funeralDate, 'dd.MM.yyyy')} r.
            </p>
          </div>
          
          <div className="flex items-center gap-4 mt-4">
            <button className="flex items-center gap-2 text-primary-600 hover:text-primary-700">
              <Candle className="w-5 h-5" />
              <span>zapalonych zniczy ({obituary.candlesCount})</span>
            </button>
            <button className="text-primary-600 hover:text-primary-700 underline">
              szczegóły pogrzebu
            </button>
          </div>
        </div>
        
        <Cross className="w-16 h-16 text-gray-400 ml-4" />
      </div>
    </div>
  );
};
```

### 7.7 Footer (Stopka)

**Wygląd:**
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              [KOLOR GŁÓWNY]                                    │
│  FIRMY                    O NAS                    REDAKCJA        DOŁĄCZ      │
│  ─────────────────────────────────────────────────────────────────────────────  │
│  🦷 Dentysta Toruń        4torun.pl - Twój lokalny   Skontaktuj się   [fb]     │
│  💊 Apteka Toruń          przewodnik po wioskach     z nami                    │
│  ...                      z Torunia i okolic.        Kontakt                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Najnowsze wpisy                                                               │
│  ⋙ Post 1              ⋙ Post 2                                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Kontakt   O Nas   Regulamin   Polityka Prywatności   © 2024 4torun.pl         │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---



## 8. PANEL ADMINISTRACYJNY - CENTRALNY

### 8.1 Struktura Interfejsu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HEADER: Logo | Dashboard | Domeny | Użytkownicy | Zrodla | Ustawienia   [Q]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐  ┌─────────────────────────────────────────────────────┐  │
│  │              │  │                                                     │  │
│  │  SIDEBAR     │  │              MAIN CONTENT AREA                      │  │
│  │              │  │                                                     │  │
│  │  Dashboard   │  │  ┌───────────────────────────────────────────────┐  │  │
│  │  Domeny      │  │  │  BREADCRUMBS: Home / Domeny / 4torun.pl       │  │  │
│  │    Lista     │  │  └───────────────────────────────────────────────┘  │  │
│  │    Dodaj     │  │                                                     │  │
│  │  Tresci      │  │  ┌───────────────────────────────────────────────┐  │  │
│  │  Uzytkownicy │  │  │           PAGE TITLE + ACTIONS                  │  │  │
│  │  Zrodla      │  │  └───────────────────────────────────────────────┘  │  │
│  │  Cron        │  │                                                     │  │
│  │  Szablony    │  │  ┌───────────────────────────────────────────────┐  │  │
│  │  Logi        │  │  │           CONTENT CARDS / TABLES              │  │  │
│  │  Ustawienia  │  │  └───────────────────────────────────────────────┘  │  │
│  └──────────────┘  └─────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Kluczowe Funkcjonalności Panelu Centralnego

| Moduł | Funkcjonalności |
|-------|----------------|
| **Dashboard** | Metryki systemowe, wykresy ruchu, top domeny, alerty |
| **Domeny** | Lista, dodawanie, edycja, usuwanie, dashboard per domena |
| **Treści** | Zarządzanie wszystkimi wpisami we wszystkich domenach |
| **Użytkownicy** | Lista, role, uprawnienia per domena |
| **Masowe Operacje** | Dodawanie bannerów/menu/wpisów do wielu domen naraz |
| **Źródła** | Konfiguracja scrapingu, testowanie, logi |
| **Cron Jobs** | Harmonogram zadań, logi wykonania |
| **Szablony** | Zarządzanie motywami, zastosowanie do domen |
| **Logi** | Przeglądarka logów systemowych z filtrami |

### 8.3 Formularz Dodawania Nowej Domeny

```typescript
interface CreateDomainForm {
  // Podstawowe
  name: string;           // Nazwa wyświetlana: "4Toruń"
  slug: string;           // ID domeny: "4torun.pl"
  city: string;           // Toruń
  region: string;         // kujawsko-pomorskie
  
  // Kontakt
  adminEmail: string;     // Email administratora
  contactPhone: string;   // Telefon kontaktowy
  
  // Wygląd
  theme: string;          // Wybór z predefiniowanych motywów
  primaryColor: string;   // #DC2626
  logo: File;             // Upload logo
  favicon: File;          // Upload favicon
  
  // Funkcjonalności
  enabledModules: string[];  // Wiadomości, Kronika, Firmy, Praca, Nekrologi
  
  // SEO
  siteTitle: string;
  siteDescription: string;
  
  // Zaawansowane
  customDomain: boolean;  // Czy użyć własnej domeny
  sslEnabled: boolean;    // Let's Encrypt
}

// Proces tworzenia:
// 1. Wypełnienie formularza (walidacja dostępności slug)
// 2. Upload assetów (logo, favicon)
// 3. Podgląd przed utworzeniem
// 4. Kliknięcie "Utwórz domenę"
// 5. System pokazuje progress deploymentu
// 6. Gotowe - link do nowej domeny
```

---

## 9. PANEL ADMINISTRACYJNY - PER SERWIS

### 9.1 Dashboard Serwisu

**Komponenty:**
- Szybkie statystyki (dzisiejsze wyświetlenia, nowe wpisy, komentarze)
- Wykres ruchu (7 dni)
- Ostatnie wpisy do zaakceptowania
- Ostatnie komentarze do moderacji
- Popularne treści

### 9.2 Edytor Wpisów (Rich Text)

```typescript
interface PostEditorProps {
  post?: Post;  // Jeśli undefined = nowy wpis
  postTypes: string[];  // Dostępne typy CPT
}

// Lewa kolumna (główna)
// - Tytuł (z auto-generowaniem slug)
// - Treść (TipTap Editor)
// - Wstęp (auto-generowany z treści)
// - Obrazek główny
// - Galeria

// Prawa kolumna (sidebar)
// - Panel publikacji (status, data, widoczność)
// - Kategorie i tagi
// - Ustawienia (polecany, przyklejony, komentarze)
// - SEO (meta tytuł, opis, słowa kluczowe)
// - Niestandardowe pola (per CPT)
```

### 9.3 Edytor Motywu (Theme Editor)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  EDYTOR MOTYWU: 4torun.pl                                          [Podgląd ▼] │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────────────────────────────────────────────────────┐│
│  │  KOLORY     │  │  PODSTAWOWE KOLORY                                          ││
│  │  Typografia │  │  ┌─────────────────────────────────────────────────────┐   ││
│  │  Layout     │  │  │  Primary    │  [███] #DC2626  [ColorPicker]      │   ││
│  │  Header     │  │  │  Secondary  │  [███] #F97316  [ColorPicker]      │   ││
│  │  Footer     │  │  │  Background │  [███] #FFFFFF  [ColorPicker]      │   ││
│  │  Zaawans.   │  │  │  Text       │  [███] #1F2937  [ColorPicker]      │   ││
│  └─────────────┘  │  └─────────────────────────────────────────────────────┘   ││
│                   │                                                             ││
│                   │  LOGO I FAVICON                                               ││
│                   │  ┌─────────────────────────────────────────────────────┐   ││
│                   │  │  Logo:  [Podgląd]  [Zmień]  [Usuń]                 │   ││
│                   │  │  Mobile: [Podgląd]  [Zmień]                         │   ││
│                   │  │  Favicon: [Podgląd]  [Zmień]                        │   ││
│                   │  └─────────────────────────────────────────────────────┘   ││
│                   │                                                             ││
│                   │  [Zapisz Zmiany]  [Resetuj]  [Eksportuj Motyw]             ││
│                   └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. SYSTEM MOTYWÓW I PERSONALIZACJI

### 10.1 Predefiniowane Motywy

```typescript
const predefinedThemes = {
  'red-modern': {
    name: 'Czerwony (jak 4torun.pl)',
    colors: {
      primary: '#DC2626',
      primaryDark: '#B91C1C',
      secondary: '#F97316',
      background: '#FFFFFF',
      text: '#1F2937'
    }
  },
  'blue-ocean': {
    name: 'Niebieski Morski',
    colors: {
      primary: '#2563EB',
      primaryDark: '#1D4ED8',
      secondary: '#06B6D4',
      background: '#F8FAFC',
      text: '#0F172A'
    }
  },
  'green-nature': {
    name: 'Zielony Eko',
    colors: {
      primary: '#059669',
      primaryDark: '#047857',
      secondary: '#84CC16',
      background: '#F0FDF4',
      text: '#064E3B'
    }
  },
  'dark-premium': {
    name: 'Ciemny Premium',
    colors: {
      primary: '#F59E0B',
      primaryDark: '#D97706',
      secondary: '#EF4444',
      background: '#111827',
      text: '#F9FAFB'
    }
  }
};
```

### 10.2 Dynamiczne Generowanie CSS

```typescript
// ThemeProvider component
const ThemeProvider: React.FC<{ config: ThemeConfig }> = ({ config, children }) => {
  useEffect(() => {
    const root = document.documentElement;
    
    // Kolory
    root.style.setProperty('--color-primary-500', config.colors.primary);
    root.style.setProperty('--color-primary-700', config.colors.primaryDark);
    root.style.setProperty('--color-secondary-500', config.colors.secondary);
    root.style.setProperty('--bg-primary', config.colors.background);
    root.style.setProperty('--text-primary', config.colors.text);
    
    // Fonty
    root.style.setProperty('--font-sans', config.typography.headingFont);
    
    // Zapisanie w localStorage dla SSR
    localStorage.setItem('theme', JSON.stringify(config));
  }, [config]);
  
  return <>{children}</>;
};
```

---

## 11. SYSTEM SCRAPINGU I CRON JOBS

### 11.1 Architektura Scrapingu

```mermaid
graph LR
    Schedule[Cron Schedule] --> Queue[RabbitMQ Queue]
    Queue --> Worker[Python Worker]
    Worker --> Fetch[HTTP Fetch]
    Fetch --> Parse[BeautifulSoup Parse]
    Parse --> DB[(PostgreSQL)]
    Parse --> Cache[Redis Cache Clear]
```

### 11.2 Konfiguracja Zródła (Przykład: Policja Toruń)

```json
{
  "name": "Policja Torun - Wiadomosci",
  "slug": "policja-torun",
  "type": "html",
  "url": "https://torun.policja.gov.pl/kb3/informacje/wiadomosci/",
  "parser_config": {
    "item_selector": "article.news-item",
    "title_selector": "h2 a",
    "content_selector": ".news-content",
    "date_selector": ".news-date",
    "image_selector": ".news-image img",
    "url_selector": "h2 a"
  },
  "mapping_config": {
    "post_type": "kronika-policyjna",
    "title": "{{title}}",
    "content": "{{content}}",
    "status": "published"
  },
  "schedule": "0 */6 * * *",
  "domains": ["4torun.pl"],
  "is_active": true
}
```

### 11.3 Worker Python (Fragment)

```python
# scraper_worker.py
import aiohttp
import beautifulsoup4 as bs
import base64
import urllib.parse

class ScraperWorker:
    async def fetch(self, url, headers=None):
        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=headers) as resp:
                return await resp.text()
    
    def parse_html(self, html, config):
        soup = bs.BeautifulSoup(html, 'html.parser')
        items = []
        
        for element in soup.select(config['item_selector']):
            # Obsługa base64 encoded URLs (jak w 4torun.pl)
            url_elem = element.select_one(config['url_selector'])
            url = url_elem.get('href') if url_elem else ''
            
            # Dekodowanie jeśli base64 w data attribute
            if not url and url_elem.get('data'):
                encoded = url_elem.get('data')
                decoded = base64.b64decode(encoded).decode('utf-8')
                url = urllib.parse.unquote(decoded)
            
            items.append({
                'title': element.select_one(config['title_selector']).text,
                'content': element.select_one(config['content_selector']).text,
                'url': url
            })
        
        return items
```

---

## 12. SEO I STRUKTURY DANYCH

### 12.1 Meta Tagi (Wymagane)

```html
<!-- Podstawowe -->
<title>{page_title} | {site_name}</title>
<meta name="description" content="{page_description}">
<meta name="robots" content="index,follow">

<!-- Open Graph -->
<meta property="og:title" content="{og_title}">
<meta property="og:description" content="{og_description}">
<meta property="og:image" content="{og_image}">
<meta property="og:url" content="{canonical_url}">
<meta property="og:type" content="article">

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image">

<!-- Canonical -->
<link rel="canonical" href="{canonical_url}">
```

### 12.2 Schema.org JSON-LD

**NewsArticle:**
```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Tytuł artykułu",
  "description": "Opis",
  "image": "https://.../image.jpg",
  "datePublished": "2024-02-12T10:00:00+01:00",
  "author": {
    "@type": "Organization",
    "name": "Redakcja 4Torun"
  },
  "publisher": {
    "@type": "Organization",
    "name": "4Torun"
  }
}
```

### 12.3 Struktura URL

```
# Wpisy
/wiadomosci/{slug}
/kronika-policyjna/{slug}
/firmy/{category}/{slug}
/praca/{slug}

# Archiwum dat
/2024/
/2024/02/
/2024/02/12/

# Sitemap
/sitemap.xml
/sitemap-posts.xml
/sitemap-categories.xml
```

---

## 13. MONITORING, LOGI I RAPORTOWANIE

### 13.1 Dashboard Monitoringu

```
┌─────────────────────────────────────────────────────────────────┐
│ SYSTEM HEALTH                      STATUS: HEALTHY              │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐ │
│  │ CPU: 45%    │ │ RAM: 62%    │ │ DB: 12/100  │ │ Disk: 78%  │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘ │
│                                                                 │
│  REQUESTS (Last 24h)                                            │
│  [Wykres liniowy]                                               │
│                                                                 │
│  TOP DOMAINS BY TRAFFIC                                         │
│  1. 4torun.pl        15,420 views (34%)                         │
│  2. 4bydgoszcz.pl    12,105 views (27%)                         │
└─────────────────────────────────────────────────────────────────┘
```

### 13.2 Struktura Logu

```json
{
  "timestamp": "2024-02-12T14:32:15.123Z",
  "level": "ERROR",
  "category": "scraper",
  "domain_id": "4torun.pl",
  "action": "fetch_data",
  "message": "Failed to fetch data",
  "context": {
    "source_id": "policja-torun",
    "error": "Connection timeout"
  },
  "ip_address": "192.168.1.1"
}
```

---

## 14. DEPLOYMENT I DODAWANIE NOWYCH PORTALI

### 14.1 Flow Dodawania Nowej Domeny

```mermaid
flowchart TD
    A["Admin clicks Add Domain"] --> B["Fill Form"]
    B --> C{"Validation"}
    C -->|Error| D["Show Errors"]
    D --> B
    C -->|OK| E["Create in Database"]
    E --> F["CREATE SCHEMA tenant_domain"]
    F --> G["CREATE TABLES"]
    G --> H["Copy Files"]
    H --> I["mkdir domains/domain/public_html"]
    I --> J["cp -r template/* public_html/"]
    J --> K["Configure Nginx"]
    K --> L["Reload Nginx"]
    L --> M["Domain Ready!"]
    M --> N["Send Email to Admin"]
    
    style M fill:#90EE90
```

### 14.2 Skrypt Deploymentu (Bash)

```bash
#!/bin/bash
# /home/host988956/scripts/create-domain.sh

DOMAIN=$1
CITY=$2
ADMIN_EMAIL=$3

echo "🚀 Tworzenie nowej domeny: $DOMAIN"

# 1. Tworzenie katalogów
mkdir -p /home/host988956/domains/$DOMAIN/{public_html,logs,backup}
echo "✓ Katalogi utworzone"

# 2. Kopiowanie template
cp -r /home/host988956/shared/templates/default/* /home/host988956/domains/$DOMAIN/public_html/
echo "✓ Template skopiowany"

# 3. Tworzenie .env.local
cat > /home/host988956/domains/$DOMAIN/public_html/.env.local << EOF
DATABASE_URL="postgresql://user:pass@localhost:5432/regional_services?schema=tenant_${DOMAIN//./_}"
NEXT_PUBLIC_DOMAIN="$DOMAIN"
NEXT_PUBLIC_CITY="$CITY"
API_URL="https://serwisy-lokalne-sterowanie.pl/api"
REDIS_URL="redis://localhost:6379/1"
EOF
echo "✓ Konfiguracja utworzona"

# 4. Tworzenie schema w PostgreSQL
psql -d regional_services -c "CREATE SCHEMA IF NOT EXISTS tenant_${DOMAIN//./_};"
echo "✓ Schema bazy danych utworzona"

# 5. Uruchomienie migracji
cd /home/host988956/domains/$DOMAIN/public_html
npm install --silent
npx prisma migrate dev --name init
echo "✓ Migracje wykonane"

# 6. Budowanie aplikacji
npm run build
echo "✓ Aplikacja zbudowana"

# 7. Konfiguracja Nginx
sudo tee /etc/nginx/sites-available/$DOMAIN << 'EOF'
server {
    listen 80;
    server_name $DOMAIN www.$DOMAIN;
    root /home/host988956/domains/$DOMAIN/public_html/dist;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }
    
    error_log /home/host988956/domains/$DOMAIN/logs/error.log;
    access_log /home/host988956/domains/$DOMAIN/logs/access.log;
}
EOF

sudo ln -s /etc/nginx/sites-available/$DOMAIN /etc/nginx/sites-enabled/ 2>/dev/null || true
sudo nginx -s reload
echo "✓ Nginx skonfigurowany"

# 8. Uruchomienie aplikacji (PM2)
PORT=3001 pm2 start npm --name "$DOMAIN" -- start
echo "✓ Aplikacja uruchomiona (PM2)"

# 9. Dodanie do centralnej bazy
curl -X POST https://serwisy-lokalne-sterowanie.pl/api/domains \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -d "{\"name\":\"$DOMAIN\",\"slug\":\"$DOMAIN\",\"city\":\"$CITY\"}" 2>/dev/null

echo ""
echo "🎉 Domena $DOMAIN została utworzona!"
echo "🔗 Panel admina: https://$DOMAIN/admin"
echo "📧 Email wysłany do: $ADMIN_EMAIL"
```

### 14.3 Template Serwisu (Do Klonowania)

```
/home/host988956/shared/templates/default/
├── src/
│   ├── app/
│   │   ├── layout.tsx              # Root layout z ThemeProvider
│   │   ├── page.tsx                # Strona główna
│   │   ├── globals.css             # CSS variables (dynamiczne)
│   │   ├── [type]/                 # Dynamic routes dla CPT
│   │   │   ├── page.tsx
│   │   │   └── [slug]/page.tsx
│   │   └── admin/                  # Panel admina
│   │       ├── layout.tsx
│   │       ├── page.tsx
│   │       └── posts/
│   │
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.tsx          # Header z menu
│   │   │   ├── Footer.tsx
│   │   │   ├── InfoBar.tsx
│   │   │   └── Navigation.tsx
│   │   ├── content/
│   │   │   ├── NewsCard.tsx
│   │   │   ├── NewsGrid.tsx
│   │   │   ├── HeroSection.tsx
│   │   │   ├── CategoryGrid.tsx
│   │   │   ├── JobCard.tsx
│   │   │   └── ObituaryCard.tsx
│   │   └── ui/                     # shadcn/ui
│   │
│   ├── lib/
│   │   ├── api.ts
│   │   └── utils.ts
│   └── hooks/
│       ├── useTheme.ts
│       └── usePosts.ts
│
├── public/
│   └── images/
│
├── prisma/
│   └── schema.prisma               # Wspólny schema
│
├── .env.example
├── next.config.js
├── tailwind.config.ts
└── package.json
```

---

## 15. WDROŻENIE - PLAN ETAPOWY

### 15.1 Diagram Etapów

```mermaid
graph LR
    E1[Etap 1<br/>Infrastruktura<br/>2 tyg] --> E2[Etap 2<br/>Baza + API<br/>2 tyg]
    E2 --> E3[Etap 3<br/>Panel Centralny<br/>3 tyg]
    E3 --> E4[Etap 4<br/>Scraping<br/>2 tyg]
    E4 --> E5[Etap 5<br/>Frontend<br/>4 tyg]
    E5 --> E6[Etap 6<br/>Panel Serwisu<br/>2 tyg]
    E6 --> E7[Etap 7<br/>Deployment<br/>1 tydz]
    E7 --> E8[Etap 8<br/>Dokumentacja<br/>1 tydz]
    
    style E1 fill:#e1f5fe
    style E2 fill:#e1f5fe
    style E3 fill:#e8f5e9
    style E4 fill:#e8f5e9
    style E5 fill:#fff3e0
    style E6 fill:#fff3e0
    style E7 fill:#fce4ec
    style E8 fill:#fce4ec
```

### 15.2 Szczegółowy Plan

| Etap | Czas | Główne Zadania | Dostarczalne |
|------|------|----------------|--------------|
| **Etap 1** | 2 tyg | Infrastruktura | Hosting, PostgreSQL, Redis, RabbitMQ, SSL |
| **Etap 2** | 2 tyg | Baza Danych + API | Schematy SQL, API centralne, Auth, RBAC |
| **Etap 3** | 3 tyg | Panel Centralny | Next.js, Dashboard, Zarządzanie domenami, Użytkownicy |
| **Etap 4** | 2 tyg | Scraping | Python workers, RabbitMQ, Parsers dla zródeł |
| **Etap 5** | 4 tyg | Frontend Serwisu | Next.js, Szablony, Komponenty, CPT views, SEO |
| **Etap 6** | 2 tyg | Panel Serwisu | Edytor wpisów, Media library, Ustawienia |
| **Etap 7** | 1 tydzień | Deployment | Skrypty automatyzacji, Testy, 4torun.pl live |
| **Etap 8** | 1 tydzień | Dokumentacja | Dokumentacja techniczna, Szkolenia |

**RAZEM: 17 tygodni (4 miesiące)**

### 15.3 Stack Technologiczny

| Warstwa | Technologia | Wersja |
|---------|-------------|--------|
| **Frontend** | Next.js + React + TypeScript | 14.x |
| **Styling** | Tailwind CSS + shadcn/ui | 3.x |
| **Backend API** | Node.js + Express/Fastify | 20 LTS |
| **ORM** | Prisma | latest |
| **Scraping** | Python + aiohttp + BeautifulSoup | 3.11+ |
| **Database** | PostgreSQL | 15+ |
| **Cache** | Redis | 7+ |
| **Queue** | RabbitMQ | 3.12+ |
| **Web Server** | Nginx | latest |
| **Process Manager** | PM2 | latest |

---

## 16. BEZPIECZEŃSTWO

### 16.1 Lista Kontrolna

- [x] JWT z krótkim czasem życia (access: 15min, refresh: 7dni)
- [x] Bezpieczne hasła (bcrypt, salt rounds 12+)
- [x] Rate limiting na logowaniu (5 prób / 15 min)
- [x] RBAC z granularnymi uprawnieniami
- [x] HTTPS/TLS 1.3 dla wszystkich domen
- [x] Parametryzowane zapytania SQL (SQL Injection protection)
- [x] Sanitizacja danych wejściowych (XSS protection)
- [x] CORS whitelist
- [x] API keys dla zewnętrznych integracji
- [x] Regularne backupy (szyfrowane)

### 16.2 Nagłówki Bezpieczeństwa (Nginx)

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';" always;
```

---

## ZALACZNIKI

### A. Słownik Pojęć

| Pojęcie | Definicja |
|---------|-----------|
| **CPT** | Custom Post Type - niestandardowy typ wpisu (wiadomości, kronika, firmy) |
| **RBAC** | Role-Based Access Control - kontrola dostępu oparta na rolach |
| **Tenant** | Najemca - oddzielny serwis w systemie multi-tenant |
| **Scraper** | Narzędzie do automatycznego pobierania danych ze źródeł zewnętrznych |
| **Slug** | Przyjazny dla URL identyfikator (np. "tytul-wpisu") |
| **Schema** | Schemat bazy danych (public, tenant_4torun_pl) |

### B. Linki i Zasoby

- Next.js Docs: https://nextjs.org/docs
- Tailwind CSS: https://tailwindcss.com
- shadcn/ui: https://ui.shadcn.com
- Prisma ORM: https://www.prisma.io/docs
- BeautifulSoup: https://www.crummy.com/software/BeautifulSoup/

### 15.3 Szczegółowy Plan Zadań (Task Breakdown)

Szczegółowy podział prac dla zespołu deweloperskiego. Każdy task zawiera estymację, odpowiedzialnego i kryteria akceptacji.

---

#### 📋 ETAP 1: INFRASTRUKTURA (2 tygodnie)

**Cel:** Przygotowanie środowiska produkcyjnego, instalacja i konfiguracja wszystkich komponentów infrastrukturalnych.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 1.1 | **Setup konta hostingowego** | DevOps | 4h | Konto aktywne, dostęp SSH działa, limity zweryfikowane | - |
| 1.2 | **Konfiguracja domen** | DevOps | 4h | Wszystkie domeny (serwisy-lokalne-sterowanie.pl, 4torun.pl, 4bydgoszcz.pl) wskazują na serwer, DNS rozwiązują się poprawnie | 1.1 |
| 1.3 | **Instalacja PostgreSQL 15** | DevOps | 6h | Baza zainstalowana, działa na porcie 5432, autostart skonfigurowany, firewall otwarty tylko dla localhost | 1.1 |
| 1.4 | **Konfiguracja PostgreSQL (security)** | DevOps | 4h | Utworzony użytkownik app_user, silne hasło, uprawnienia ograniczone, logowanie włączone | 1.3 |
| 1.5 | **Instalacja Redis** | DevOps | 3h | Redis działa na porcie 6379, autostart skonfigurowany, persistencja włączona | 1.1 |
| 1.6 | **Instalacja RabbitMQ** | DevOps | 4h | RabbitMQ działa (management UI dostępny), użytkownik app_user utworzony, vhost skonfigurowany | 1.1 |
| 1.7 | **Instalacja Node.js 20 LTS** | DevOps | 2h | Node v20.x zainstalowany, npm działa, n dostępny | 1.1 |
| 1.8 | **Instalacja Python 3.11** | DevOps | 2h | Python 3.11 zainstalowany, pip działa, venv dostępny | 1.1 |
| 1.9 | **Instalacja Nginx** | DevOps | 3h | Nginx zainstalowany, działa na porcie 80/443, konfiguracja domyślna działa | 1.1 |
| 1.10 | **Konfiguracja SSL (Let's Encrypt)** | DevOps | 4h | Certyfikaty SSL wygenerowane dla wszystkich domen, auto-renewal skonfigurowane (cron), redirect HTTP→HTTPS działa | 1.2, 1.9 |
| 1.11 | **Instalacja PM2** | DevOps | 2h | PM2 zainstalowany globalnie, logrotate skonfigurowane, startup script wygenerowany | 1.7 |
| 1.12 | **Tworzenie struktury katalogów** | DevOps | 3h | Katalogi /domains, /shared, /workers, /logs utworzone, uprawnienia ustawione (www-data), quoty skonfigurowane | 1.1 |
| 1.13 | **Konfiguracja firewall (UFW)** | DevOps | 3h | Otwarte porty: 22, 80, 443, 64321 (SSH custom), pozostałe zamknięte, reguły działają | 1.1 |
| 1.14 | **Setup backupu automatycznego** | DevOps | 6h | Skrypty backupu (baza + pliki), cron codziennie o 2:00, retencja 30 dni, test restore wykonany | 1.3, 1.12 |
| 1.15 | **Konfiguracja logrotate** | DevOps | 2h | Logi nginx, app, system rotowane codziennie, kompresja po 7 dniach, usuwanie po 90 dniach | 1.9 |
| 1.16 | **Instalacja monitoringu (opcjonalnie)** | DevOps | 4h | Netdata lub podobne zainstalowane, dashboard dostępny, alerty skonfigurowane | 1.1 |

**Deliverables Etapu 1:**
- [ ] Serwer gotowy, wszystkie usługi działają
- [ ] Dokumentacja dostępów (loginy, hasła w bezpiecznym miejscu)
- [ ] Test połączenia do każdej usługi
- [ ] Backup działa (przywrócony testowo)

---

#### 📋 ETAP 2: BAZA DANYCH I API (2 tygodnie)

**Cel:** Stworzenie architektury bazy danych multi-tenant oraz API centralnego z systemem autentykacji.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 2.1 | **Inicjalizacja projektu API** | Backend | 2h | Projekt Node.js + TypeScript utworzony, struktura katalogów, tsconfig, eslint, prettier | 1.7 |
| 2.2 | **Instalacja zależności API** | Backend | 2h | Express, Prisma, Zod, JWT, bcrypt, cors, helmet, rate-limit zainstalowane | 2.1 |
| 2.3 | **Konfiguracja Prisma** | Backend | 4h | Prisma zainicjalizowana, połączenie z DB działa, pierwszy model utworzony | 1.3, 2.2 |
| 2.4 | **Projektowanie schema public** | Backend | 8h | Wszystkie tabele centralne (users, roles, domains, sources, cron_jobs) zaprojektowane w Prisma, relacje zdefiniowane | 2.3 |
| 2.5 | **Projektowanie schema tenant** | Backend | 8h | Wszystkie tabele per tenant (posts, categories, comments, companies, jobs, etc.) zaprojektowane, relacje zdefiniowane | 2.4 |
| 2.6 | **Migracje bazy danych** | Backend | 4h | Pierwsza migracja utworzona i wykonana, schema public istnieje w DB | 2.4, 2.5 |
| 2.7 | **Setup struktury API** | Backend | 4h | Katalogi routes, controllers, services, middleware, utils utworzone, routing działa | 2.1 |
| 2.8 | **Middleware autentykacji JWT** | Backend | 6h | Middleware weryfikujące token JWT działa, obsługa błędów (401, 403), refresh token mechanism | 2.7 |
| 2.9 | **System RBAC (Role-Based Access Control)** | Backend | 8h | Middleware sprawdzające uprawnienia działa, helpery do sprawdzania ról, system działa per domena i globalnie | 2.8 |
| 2.10 | **API Auth (login, register, refresh, logout)** | Backend | 6h | Endpointy /auth/* działają, walidacja Zod, hasła hashowane (bcrypt), tokeny generowane poprawnie | 2.8 |
| 2.11 | **CRUD API dla domen** | Backend | 8h | Endpointy GET/POST/PUT/DELETE /domains działają, walidacja, obsługa błędów, paginacja | 2.9 |
| 2.12 | **CRUD API dla użytkowników** | Backend | 8h | Endpointy /users działają, zarządzanie rolami, przypisywanie do domen | 2.9 |
| 2.13 | **CRUD API dla źródeł (sources)** | Backend | 6h | Endpointy /sources działają, konfiguracja parserów zapisywana w JSONB | 2.9 |
| 2.14 | **API dla masowych operacji** | Backend | 6h | Endpointy /mass/* działają, obsługa dodawania do wielu domen naraz | 2.11 |
| 2.15 | **Walidacja requestów (Zod)** | Backend | 4h | Wszystkie endpointy walidują dane wejściowe, czytelne komunikaty błędów | Cały etap |
| 2.16 | **Obsługa błędów i logowanie** | Backend | 4h | Centralny error handler, logi w Winston/structlog, stack traces w dev | Cały etap |
| 2.17 | **Dokumentacja API (Swagger/OpenAPI)** | Backend | 4h | Specyfikacja OpenAPI dostępna pod /api-docs, wszystkie endpointy udokumentowane | Cały etap |
| 2.18 | **Testy jednostkowe API** | Backend | 8h | Testy dla głównych endpointów (auth, domains, users), coverage > 70% | Cały etap |

**Deliverables Etapu 2:**
- [ ] Baza danych z pełnym schematem (public + tenant template)
- [ ] API działa, wszystkie endpointy testowane w Postman/Insomnia
- [ ] Dokumentacja API dostępna online
- [ ] Testy jednostkowe przechodzą

---

#### 📋 ETAP 3: PANEL ADMINISTRACYJNY CENTRALNY (3 tygodnie)

**Cel:** Stworzenie interfejsu webowego do zarządzania całym systemem.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 3.1 | **Inicjalizacja projektu Next.js** | Frontend | 2h | Projekt Next.js 14 + TypeScript utworzony, App Router, Tailwind skonfigurowany | 1.7 |
| 3.2 | **Instalacja shadcn/ui** | Frontend | 2h | shadcn/ui zainicjalizowany, podstawowe komponenty zainstalowane (button, card, input, dialog, table) | 3.1 |
| 3.3 | **Setup klienta API** | Frontend | 4h | Klient HTTP (axios/fetch) z interceptors, obsługa tokenów, refresh token działa | 2.10 |
| 3.4 | **System autentykacji (frontend)** | Frontend | 6h | Formularz logowania działa, przechowywanie tokenów (httpOnly cookie lub secure localStorage), protected routes | 3.3 |
| 3.5 | **Layout główny panelu** | Frontend | 6h | Header, Sidebar, Main Content Area działają, responsywność działa (mobile menu), nawigacja między stronami | 3.2 |
| 3.6 | **Strona Dashboard** | Frontend | 8h | Metryki wyświetlane, wykresy (Chart.js/Recharts) działają, dane pobierane z API | 3.5 |
| 3.7 | **Lista domen (tabela)** | Frontend | 8h | Tabela z paginacją, sortowaniem, filtrowaniem, akcjami (edytuj, usuń), działa z API | 2.11 |
| 3.8 | **Formularz dodawania domeny** | Frontend | 8h | Formularz z walidacją (Zod), upload logo (drag&drop), podgląd przed utworzeniem | 3.7 |
| 3.9 | **Szczegóły domeny z zakładkami** | Frontend | 8h | Zakładki: Dashboard, Treści, Użytkownicy, Ustawienia działają, routing per zakładka | 3.7 |
| 3.10 | **Zarządzanie użytkownikami systemu** | Frontend | 8h | Lista użytkowników, formularz dodawania, przypisywanie ról per domena | 2.12 |
| 3.11 | **Zarządzanie źródłami scrapingu** | Frontend | 8h | Lista źródeł, formularz z konfiguracją parserów (JSON editor), przycisk "Uruchom scraping" | 2.13 |
| 3.12 | **Moduł masowych operacji** | Frontend | 10h | Interfejs do dodawania bannerów/menu/wpisów do wielu domen, wybór domen (checkboxy), podgląd zmian | 2.14 |
| 3.13 | **Przeglądarka logów** | Frontend | 8h | Tabela logów z filtrami (data, poziom, kategoria), paginacja, możliwość eksportu | 2.16 |
| 3.14 | **Zarządzanie szablonami** | Frontend | 6h | Lista szablonów, edytor podstawowy, zastosowanie do wybranych domen | 3.2 |
| 3.15 | **Obsługa błędów i toast notifications** | Frontend | 4h | Globalna obsługa błędów API, toast notifications działają, retry mechanism | Cały etap |
| 3.16 | **Responsywność panelu** | Frontend | 6h | Panel działa poprawnie na tabletach i mobilnych (testowanie), menu dostosowane | Cały etap |

**Deliverables Etapu 3:**
- [ ] Panel centralny działa pod serwisy-lokalne-sterowanie.pl
- [ ] Można dodać nową domenę przez interfejs
- [ ] Można zarządzać użytkownikami i źródłami
- [ ] Wszystkie funkcje przetestowane manualnie

---

#### 📋 ETAP 4: SYSTEM SCRAPINGU (2 tygodnie)

**Cel:** Implementacja workerów pobierających dane ze źródeł zewnętrznych.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 4.1 | **Inicjalizacja projektu Python** | Backend | 2h | Struktura katalogów, requirements.txt, venv | 1.8 |
| 4.2 | **Instalacja zależności Python** | Backend | 2h | aiohttp, beautifulsoup4, pika, psycopg2, python-dotenv zainstalowane | 4.1 |
| 4.3 | **Moduł połączenia z PostgreSQL** | Backend | 4h | Klient asyncpg/psycopg2 działa, połączenie pulowane, transakcje działają | 4.2 |
| 4.4 | **Moduł połączenia z RabbitMQ** | Backend | 4h | Consumer działa, reconnect po utracie połączenia, kolejki deklarowane | 4.2, 1.6 |
| 4.5 | **Moduł HTTP fetcher** | Backend | 6h | Pobieranie stron działa, obsługa timeoutów, retry logic (3 próby), headers User-Agent | 4.2 |
| 4.6 | **Parser HTML (BeautifulSoup)** | Backend | 8h | Selektory CSS działają, ekstrakcja tytułu, treści, daty, obrazków działa | 4.5 |
| 4.7 | **Parser RSS/Atom** | Backend | 4h | Parsowanie RSS działa, obsługa różnych formatów | 4.5 |
| 4.8 | **Parser JSON/API** | Backend | 4h | Parsowanie JSON działa, obsługa nested structures | 4.5 |
| 4.9 | **Dekodowanie base64 URL-i** | Backend | 3h | Źródła z base64 encoded URL (jak policja.gov.pl) są poprawnie dekodowane | 4.6 |
| 4.10 | **System mapowania pól** | Backend | 6h | Mapping konfiguracji na pola bazy działa, obsługa template strings | 4.6 |
| 4.11 | **Zapisywanie do bazy (upsert)** | Backend | 6h | Wpisy tworzone/aktualizowane poprawnie, obsługa duplikatów (external_id), transakcje | 4.3 |
| 4.12 | **Czyszczenie cache po scrapingu** | Backend | 3h | Po zapisaniu danych, cache Redis jest invalidowany dla danej domeny | 4.11, 1.5 |
| 4.13 | **Obsługa błędów i logowanie** | Backend | 4h | Błędy logowane szczegółowo, failed jobs wracają do kolejki (dead letter queue) | Cały etap |
| 4.14 | **Skrypt testujący parser** | Backend | 4h | Można przetestować parser na konkretnym URL bez zapisywania do bazy | 4.6 |
| 4.15 | **Konfiguracja cron jobs (systemowych)** | DevOps | 4h | Cron uruchamia scrapery wg harmonogramu, logi zapisywane | 1.1, 4.4 |
| 4.16 | **Monitoring workerów** | Backend | 3h | Health check endpoint, metryki (przetworzone elementy, błędy) | Cały etap |

**Deliverables Etapu 4:**
- [ ] Scraper działa i pobiera dane z policja.gov.pl
- [ ] Scraper działa i pobiera dane z torun.pl
- [ ] Dane pojawiają się w bazie
- [ ] Cron uruchamia scrapery automatycznie

---

#### 📋 ETAP 5: FRONTEND SERWISU REGIONALNEGO (4 tygodnie)

**Cel:** Stworzenie strony publicznej dla serwisów regionalnych (4torun.pl, itp.).

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 5.1 | **Setup projektu Next.js (kopia template)** | Frontend | 2h | Projekt sklonowany z template, działa lokalnie, połączenie z API centralnym | 3.1 |
| 5.2 | **Implementacja ThemeProvider** | Frontend | 6h | Kolory z API są dynamicznie wczytywane i stosowane, zmiany w panelu od razu widać na stronie | 5.1, 10.2 |
| 5.3 | **Komponent Header** | Frontend | 8h | Logo, menu, dropdowny, sticky header, mobile menu (hamburger), warianty (default/transparent) | 5.2 |
| 5.4 | **Komponent InfoBar** | Frontend | 6h | Data, imieniny, pogoda, jakość powietrza wyświetlane poprawnie, odświeżanie danych | 5.2 |
| 5.5 | **Komponent Navigation** | Frontend | 6h | Menu z CPT, dropdowny, wyróżnione elementy, responsywne | 5.3 |
| 5.6 | **Komponent HeroSection** | Frontend | 8h | Układ jak na zrzutach (główna wiadomość + boczne + mini), responsywny | 5.2 |
| 5.7 | **Komponent NewsCard + NewsGrid** | Frontend | 8h | Wszystkie warianty (default, featured, horizontal, overlay), Masonry/grid działa | 5.2 |
| 5.8 | **Komponent CategoryGrid (Firmy)** | Frontend | 6h | Grid z ikonami, hover effects, linki działają | 5.2 |
| 5.9 | **Komponent JobCard** | Frontend | 6h | Wyświetlanie ofert pracy, filtry, sortowanie | 5.2 |
| 5.10 | **Komponent ObituaryCard** | Frontend | 6h | Specjalny design dla nekrologów, znicze, formatowanie | 5.2 |
| 5.11 | **Strona Główna (Home)** | Frontend | 10h | Wszystkie sekcje zgodnie z projektem (Hero, Wiadomości, Kronika, Firmy, Praca, Przewodnik, Nekrologi, Ludzie), dane z API | 5.3-5.10 |
| 5.12 | **Strona Archiwum (lista wpisów)** | Frontend | 8h | Filtrowanie, paginacja, sidebar z kategoriami, działa dla wszystkich CPT | 5.7 |
| 5.13 | **Strona Pojedynczego Wpisu** | Frontend | 10h | Wyświetlanie treści, galeria, autor, źródło, oceny, komentarze, related posts | 5.7 |
| 5.14 | **System komentarzy (wyświetlanie + dodawanie)** | Frontend | 8h | Lista komentarzy, formularz dodawania (dla niezalogowanych też), odpowiedzi | 5.13 |
| 5.15 | **System ocen (rating)** | Frontend | 6h | Gwiazdki, średnia wyświetlana, możliwość głosowania | 5.13 |
| 5.16 | **Komponent WeatherPage (szczegóły)** | Frontend | 6h | Strona /pogoda z mapą, 5-dniową prognozą, szczegółami | 5.4 |
| 5.17 | **Komponent Footer** | Frontend | 6h | 3 kolumny, najnowsze wpisy, linki, social media, newsletter | 5.2 |
| 5.18 | **SEO (meta tagi)** | Frontend | 6h | Dynamiczne meta tagi per strona, Open Graph, Twitter Cards | 12 |
| 5.19 | **Generowanie sitemap.xml** | Frontend | 4h | Sitemap generowana dynamicznie, działa dla robotów | 5.18 |
| 5.20 | **Lazy loading obrazków** | Frontend | 4h | Blur placeholder, lazy loading działa, Lighthouse performance > 80 | 5.7 |
| 5.21 | **Responsywność (mobile)** | Frontend | 8h | Wszystkie strony działają na mobile (320px+), testowane | Cały etap |
| 5.22 | **Optymalizacja Core Web Vitals** | Frontend | 6h | LCP < 2.5s, FID < 100ms, CLS < 0.1, testy w PageSpeed Insights | Cały etap |

**Deliverables Etapu 5:**
- [ ] Strona 4torun.pl działa publicznie
- [ ] Wszystkie sekcje wyglądają jak na zrzutach
- [ ] Responsywność działa
- [ ] SEO meta tagi są poprawne
- [ ] Core Web Vitals na zielono

---

#### 📋 ETAP 6: PANEL ADMINISTRACYJNY SERWISU (2 tygodnie)

**Cel:** Panel zarządzania dla konkretnego serwisu regionalnego.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 6.1 | **Layout panelu serwisu** | Frontend | 4h | Sidebar nawigacyjny, header z info o domenie, działa na mobile | 5.1 |
| 6.2 | **Dashboard serwisu** | Frontend | 6h | Statystyki (wyświetlenia, wpisy, komentarze), wykres, aktywność | 6.1 |
| 6.3 | **Lista wpisów (tabela)** | Frontend | 8h | Tabela z filtrami (typ, status, kategoria), akcje (edytuj, usuń, podgląd), sortowanie | 6.1 |
| 6.4 | **Edytor wpisów (Rich Text)** | Frontend | 12h | TipTap editor działa, formatowanie, dodawanie obrazków, zapisywanie wersji | 6.3 |
| 6.5 | **Upload obrazków (Media Library)** | Frontend | 8h | Drag&drop upload, przeglądarka mediów, wybór do wpisu, usuwanie | 6.4 |
| 6.6 | **Zarządzanie kategoriami** | Frontend | 6h | Drzewo kategorii, dodawanie, edycja, usuwanie, sortowanie (drag&drop) | 6.1 |
| 6.7 | **Zarządzanie tagami** | Frontend | 4h | Lista tagów, autosuggest przy dodawaniu do wpisu, możliwość łączenia | 6.1 |
| 6.8 | **Budowniczy menu (Menu Builder)** | Frontend | 8h | Drag&drop do tworzenia menu, hierarchia, wybór linków (strony/wpisy/kategorie) | 6.1 |
| 6.9 | **Zarządzanie widgetami** | Frontend | 6h | Lista widgetów, konfiguracja per pozycja, włączanie/wyłączanie | 6.1 |
| 6.10 | **Zarządzanie bannerami** | Frontend | 6h | Lista bannerów, upload obrazków, daty wyświetlania, pozycje | 6.1 |
| 6.11 | **Ustawienia serwisu (ogólne)** | Frontend | 6h | Nazwa, opis, kontakt, social media, kolory (podstawowe) | 6.1 |
| 6.12 | **Edytor motywu (zaawansowany)** | Frontend | 8h | Color pickery, wybór fontów, podgląd na żywo, eksport/import | 6.11 |
| 6.13 | **Ustawienia SEO per serwis** | Frontend | 6h | Globalne meta tagi, robots.txt, struktura permalinków | 6.11 |

**Deliverables Etapu 6:**
- [ ] Panel /admin działa na 4torun.pl
- [ ] Można dodać, edytować, usunąć wpis
- [ ] Media library działa
- [ ] Można zmienić motyw kolorystyczny

---

#### 📋 ETAP 7: DEPLOYMENT I TESTOWANIE (1 tydzień)

**Cel:** Wdrożenie pierwszego serwisu produkcyjnie i testowanie całości.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 7.1 | **Konfiguracja 4torun.pl w panelu centralnym** | DevOps | 4h | Domena dodana przez panel, schema utworzona, pliki skopiowane | 3.8 |
| 7.2 | **Import początkowych danych** | Backend | 4h | Przykładowe wpisy dodane, kategorie utworzone, menu skonfigurowane | 7.1 |
| 7.3 | **Konfiguracja źródeł scrapingu dla 4torun.pl** | Backend | 3h | Policja i Urząd Miasta dodane jako źródła, pierwszy scraping wykonany | 4.16, 7.1 |
| 7.4 | **Konfiguracja SSL dla 4torun.pl** | DevOps | 2h | Certyfikat działa, redirect HTTP→HTTPS działa | 1.10, 7.1 |
| 7.5 | **Testy end-to-end (E2E)** | QA | 8h | Testy w Cypress/Playwright przechodzą (logowanie, dodanie wpisu, scraping) | Cały etap |
| 7.6 | **Testy wydajnościowe** | DevOps | 4h | Load testing (100 równoczesnych użytkowników), odpowiedź < 200ms | Cały etap |
| 7.7 | **Testy bezpieczeństwa (podstawowe)** | DevOps | 4h | Brak krytycznych podatności w npm audit, headers bezpieczeństwa działają | Cały etap |
| 7.8 | **Backup przed uruchomieniem** | DevOps | 2h | Pełny backup wykonany i zweryfikowany | 1.14 |
| 7.9 | **Uruchomienie produkcyjne** | DevOps | 2h | Strona dostępna publicznie, monitoring działa | Cały etap |
| 7.10 | **Poprawki po testach** | Cały zespół | 8h | Zgłoszone błędy naprawione, retesty przechodzą | 7.5-7.7 |

**Deliverables Etapu 7:**
- [ ] 4torun.pl działa produkcyjnie
- [ ] Testy E2E przechodzą
- [ ] Wydajność jest akceptowalna
- [ ] Backup działa

---

#### 📋 ETAP 8: DOKUMENTACJA I PRZEKAZANIE (1 tydzień)

**Cel:** Przygotowanie dokumentacji i szkolenie użytkowników.

| # | Zadanie | Odpowiedzialny | Estymacja | Kryteria Akceptacji | Zależności |
|---|---------|----------------|-----------|---------------------|------------|
| 8.1 | **Dokumentacja techniczna** | Backend | 8h | Architektura, API, deployment opisane, diagramy aktualne | Cały etap |
| 8.2 | **Dokumentacja użytkownika (panel centralny)** | Frontend | 6h | Instrukcja obsługi panelu centralnego ze zrzutami ekranu | 8.1 |
| 8.3 | **Dokumentacja użytkownika (panel serwisu)** | Frontend | 6h | Instrukcja dodawania wpisów, zarządzania treścią | 8.1 |
| 8.4 | **Szkolenie administratorów** | Cały zespół | 4h | Szkolenie przeprowadzone, materiały przekazane, pytania omówione | 8.2 |
| 8.5 | **Szkolenie redaktorów** | Frontend | 4h | Szkolenie z edytora wpisów, publikowania, zarządzania mediami | 8.3 |
| 8.6 | **Przekazanie dostępów i haseł** | DevOps | 2h | Hasła przekazane bezpiecznym kanałem, lista kont i uprawnień | Cały etap |
| 8.7 | **Ostateczne testy akceptacyjne (UAT)** | Klient | 4h | Klient akceptuje system lub zgłasza ostatnie poprawki | Cały etap |
| 8.8 | **Poprawki po UAT** | Cały zespół | 8h | Zgłoszenia z UAT naprawione | 8.7 |
| 8.9 | **Podpisanie protokołu odbioru** | PM | 1h | Dokument podpisany przez obie strony | 8.8 |

**Deliverables Etapu 8:**
- [ ] Dokumentacja kompletna
- [ ] Szkolenia przeprowadzone
- [ ] Protokół odbioru podpisany
- [ ] System przekazany do użytkowania

---

### 15.4 Harmonogram i Zależności Krytyczne

```mermaid
gantt
    title Project Timeline
    dateFormat  YYYY-MM-DD
    section Infrastructure
    Setup hosting            :done, infra1, 2026-02-17, 10d
    Install services         :done, infra2, after infra1, 4d
    
    section Database and API
    Database design          :active, db1, after infra2, 5d
    API implementation       :db2, after db1, 9d
    
    section Admin Panel
    Frontend setup           :admin1, after db2, 2d
    Module implementation    :admin2, after admin1, 19d
    
    section Scraping
    Worker implementation    :scrap1, after db2, 10d
    Cron configuration       :scrap2, after scrap1, 4d
    
    section Frontend
    UI Components            :front1, after admin1, 14d
    Public pages             :front2, after front1, 14d
    
    section Site Panel
    Edytor i funkcje         :site1, after front1, 10d
    
    section Deployment
    Testy i wdrożenie        :deploy1, after front2, 5d
    
    section Dokumentacja
    Docs i szkolenia         :doc1, after deploy1, 5d
```

**Ścieżka krytyczna:**
Infrastruktura → Baza + API → Panel Centralny (Setup) → Frontend (Komponenty) → Frontend (Strony) → Deployment

**Całkowity czas:** 17 tygodni (ok. 4 miesiące)

---

## 9. MODUŁ PIŁKI NOŻNEJ ⚽

Rozszerzenie systemu o moduł sportowy - automatyczne pobieranie i wyświetlanie danych piłkarskich dla drużyn regionalnych (na podstawie Sofascore).

### 9.1 Architektura Modułu

```mermaid
flowchart TB
    subgraph External["External Sources"]
        SS[Sofascore.com]
    end
    
    subgraph ScrapingLayer["Scraping Layer"]
        SW[Football Scraper Worker]
        API[REST API Client]
    end
    
    subgraph DataLayer["Data Layer"]
        DB_FOOTBALL[(football_teams)]
        DB_MATCHES[(football_matches)]
        DB_STANDINGS[(football_standings)]
        DB_SQUADS[(football_squads)]
    end
    
    subgraph FrontendLayer["Frontend"]
        W_NEXT[Widget Next Match]
        W_LAST[Widget Last Result]
        W_STAND[Widget Standings]
        PAGE_TEAM[Team Page]
        PAGE_TABLE[League Table]
    end
    
    SS -->|HTML/JSON| SW
    SW -->|Parsed Data| API
    API --> DB_FOOTBALL
    API --> DB_MATCHES
    API --> DB_STANDINGS
    API --> DB_SQUADS
    
    DB_FOOTBALL --> W_NEXT
    DB_MATCHES --> W_NEXT
    DB_MATCHES --> W_LAST
    DB_STANDINGS --> W_STAND
    DB_STANDINGS --> PAGE_TABLE
    DB_FOOTBALL --> PAGE_TEAM
    DB_MATCHES --> PAGE_TEAM
```

### 9.2 Struktura Danych

#### Model: FootballTeam
```typescript
interface FootballTeam {
  id: string;                    // UUID
  domain_id: string;             // FK do domains
  external_id: number;           // ID z Sofascore (np. 4901)
  slug: string;                  // "cracovia"
  name: string;                  // "Cracovia"
  short_name: string;            // "Cracovia"
  country_code: string;          // "PL"
  country_name: string;          // "Polska"
  logo_url: string;              // https://img.sofascore.com/api/v1/team/{id}/image
  stadium_name?: string;         // "Marshal Jozef Pilsudski Stadium"
  manager_name?: string;         // "Luka Elsner"
  manager_photo_url?: string;
  current_tournament_id?: number;
  current_tournament_name?: string;
  is_primary: boolean;           // Czy to główna drużyna miasta
  display_order: number;         // Kolejność wyświetlania
  is_active: boolean;
  created_at: Date;
  updated_at: Date;
}
```

#### Model: FootballMatch
```typescript
interface FootballMatch {
  id: string;
  external_id: number;           // ID meczu z Sofascore
  team_id: string;               // FK do football_teams
  opponent_id?: number;          // External ID przeciwnika
  opponent_name: string;
  opponent_short_name: string;
  opponent_logo_url: string;
  opponent_slug?: string;
  
  // Dane meczu
  match_date: Date;
  match_time?: string;           // "20:30"
  status: 'scheduled' | 'live' | 'finished' | 'postponed';
  is_home: boolean;              // Czy drużyna główna gra u siebie
  
  // Wyniki
  home_score?: number;
  away_score?: number;
  home_halftime_score?: number;
  away_halftime_score?: number;
  
  // Wynik z perspektywy drużyny głównej
  result?: 'W' | 'D' | 'L';      // Win, Draw, Loss
  
  // Turniej
  tournament_id: number;
  tournament_name: string;
  tournament_logo_url?: string;
  round?: string;                // "Runda 21"
  
  // Flagi
  is_featured: boolean;          // Wyróżniony na stronie głównej
  show_on_homepage: boolean;
  
  created_at: Date;
  updated_at: Date;
}
```

#### Model: FootballStandings
```typescript
interface FootballStandings {
  id: string;
  team_id: string;               // FK do football_teams
  tournament_id: number;
  tournament_name: string;
  season: string;                // "2025/2026"
  
  // Pozycja i statystyki
  position: number;
  played: number;
  won: number;
  drawn: number;
  lost: number;
  goals_for: number;
  goals_against: number;
  goal_difference: number;
  points: number;
  
  // Forma (ostatnie 5 meczów)
  form: string;                  // np. "WDRDW"
  form_details?: FormMatch[];    // Szczegóły formy
  
  // Status
  promotion_status?: 'champions' | 'conference' | 'relegation' | null;
  
  // Typ tabeli
  table_type: 'total' | 'home' | 'away';
  
  updated_at: Date;
}

interface FormMatch {
  match_id: number;
  result: 'W' | 'D' | 'L';
  opponent_name: string;
  score: string;                 // "2:1"
  date: Date;
}
```

#### Model: FootballPlayer
```typescript
interface FootballPlayer {
  id: string;
  team_id: string;
  name: string;
  position: 'forward' | 'midfielder' | 'defender' | 'goalkeeper';
  is_key_player: boolean;
  jersey_number?: number;
  nationality?: string;
  birth_date?: Date;
  photo_url?: string;
}
```

### 9.3 Schemat Bazy Danych (SQL)

```sql
-- Tabela drużyn
CREATE TABLE football_teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id UUID NOT NULL REFERENCES domains(id) ON DELETE CASCADE,
    external_id INTEGER NOT NULL,
    slug VARCHAR(50) NOT NULL,
    name VARCHAR(100) NOT NULL,
    short_name VARCHAR(50),
    country_code VARCHAR(2) DEFAULT 'PL',
    country_name VARCHAR(50) DEFAULT 'Polska',
    logo_url VARCHAR(255),
    stadium_name VARCHAR(100),
    manager_name VARCHAR(100),
    manager_photo_url VARCHAR(255),
    current_tournament_id INTEGER,
    current_tournament_name VARCHAR(100),
    is_primary BOOLEAN DEFAULT false,
    display_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(domain_id, external_id)
);

-- Tabela meczów
CREATE TABLE football_matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_id INTEGER NOT NULL UNIQUE,
    team_id UUID NOT NULL REFERENCES football_teams(id) ON DELETE CASCADE,
    opponent_id INTEGER,
    opponent_name VARCHAR(100) NOT NULL,
    opponent_short_name VARCHAR(50),
    opponent_logo_url VARCHAR(255),
    opponent_slug VARCHAR(50),
    match_date DATE NOT NULL,
    match_time TIME,
    status VARCHAR(20) DEFAULT 'scheduled',
    is_home BOOLEAN DEFAULT true,
    home_score INTEGER,
    away_score INTEGER,
    home_halftime_score INTEGER,
    away_halftime_score INTEGER,
    result CHAR(1),
    tournament_id INTEGER,
    tournament_name VARCHAR(100),
    tournament_logo_url VARCHAR(255),
    round_info VARCHAR(50),
    is_featured BOOLEAN DEFAULT false,
    show_on_homepage BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela tabeli ligowej
CREATE TABLE football_standings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES football_teams(id) ON DELETE CASCADE,
    tournament_id INTEGER NOT NULL,
    tournament_name VARCHAR(100),
    season VARCHAR(10) NOT NULL,
    position INTEGER NOT NULL,
    played INTEGER DEFAULT 0,
    won INTEGER DEFAULT 0,
    drawn INTEGER DEFAULT 0,
    lost INTEGER DEFAULT 0,
    goals_for INTEGER DEFAULT 0,
    goals_against INTEGER DEFAULT 0,
    goal_difference INTEGER DEFAULT 0,
    points INTEGER DEFAULT 0,
    form VARCHAR(5),
    form_details JSONB,
    promotion_status VARCHAR(20),
    table_type VARCHAR(10) DEFAULT 'total',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(team_id, tournament_id, season, table_type)
);

-- Tabela zawodników
CREATE TABLE football_squads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES football_teams(id) ON DELETE CASCADE,
    external_id INTEGER,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(20) NOT NULL,
    is_key_player BOOLEAN DEFAULT false,
    jersey_number INTEGER,
    nationality VARCHAR(50),
    birth_date DATE,
    photo_url VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indeksy
CREATE INDEX idx_football_matches_team_date ON football_matches(team_id, match_date);
CREATE INDEX idx_football_matches_status ON football_matches(status);
CREATE INDEX idx_football_matches_featured ON football_matches(is_featured) WHERE is_featured = true;
CREATE INDEX idx_football_standings_position ON football_standings(tournament_id, season, position);
```

### 9.4 Konfiguracja Scrapera

#### Nowy typ scrapera: `SOFASCORE_FOOTBALL`

```typescript
interface SofascoreFootballScraperConfig {
  type: 'SOFASCORE_FOOTBALL';
  teamId: number;                // np. 4901 dla Cracovii
  teamSlug: string;              // "cracovia"
  domainId: string;              // ID domeny w systemie
  
  // Co scrapować
  dataTypes: {
    matches: boolean;            // Mecze
    standings: boolean;          // Tabela
    squad: boolean;              // Skład
    form: boolean;               // Forma (wykres)
  };
  
  // Interwały
  intervals: {
    matches: number;             // minuty (np. 60)
    standings: number;           // minuty (np. 120)
    squad: number;               // minuty (np. 1440 - raz dziennie)
  };
  
  // Opcje wyświetlania
  displayOptions: {
    showOnHomepage: boolean;
    homepageWidgetType: 'next-match' | 'last-result' | 'standings' | 'mini-table' | 'all';
    highlightIfPlayingToday: boolean;  // Wyróżnij na stronie głównej jeśli mecz dziś
  };
}
```

#### Parser - selektory CSS

```python
SOFASCORE_SELECTORS = {
    # Dane drużyny
    'team_name': 'h1.textStyle_display.extraLarge',
    'team_logo': 'img[alt="{team_name}"]',
    'manager_name': 'span:contains("Trener") + span',
    'manager_photo': 'img[alt="{manager_name}"]',
    'stadium': '.p_lg svg + span',
    'tournament': '[href*="/tournament/"]',
    
    # Mecze
    'match_list': '.card-component .d_flex.flex-d_column',
    'match_item': 'a[href*="/match/"]',
    'match_date': 'bdi.textStyle_body.small',
    'match_time': 'span.textStyle_body.small',
    'match_home_team': '.d_flex.ai_center:first-child bdi',
    'match_away_team': '.d_flex.ai_center:last-child bdi',
    'match_score': 'span.score',
    'match_result': '.w_xl.h_xl.br_50%',  # W/D/L
    
    # Tabela
    'standings_table': '.tabs__content[role="tabpanel"]',
    'standings_row': 'a[href*="/team/"] + div',
    'position': '.w_xl.h_xl.br_50%',
    'team_in_table': 'img + div span',
    'stats': 'bdi.textStyle_table.medium',
    'form': '.d_flex.ai_center.jc_flex-start',
}
```

### 9.5 Komponenty Frontend

#### A. Widgety na Stronie Głównej

```typescript
// Widget następnego meczu
interface NextMatchWidgetProps {
  team: FootballTeam;
  match: FootballMatch;
  variant: 'large' | 'compact';
  showCountdown?: boolean;
}

// Widget ostatniego wyniku
interface LastResultWidgetProps {
  team: FootballTeam;
  match: FootballMatch;
  showStats?: boolean;
}

// Widget pozycji w tabeli
interface StandingsWidgetProps {
  team: FootballTeam;
  standings: FootballStandings;
  showTrend?: boolean;
}

// Mini tabela ligowa
interface MiniTableWidgetProps {
  tournamentName: string;
  teams: StandingsTeam[];
  highlightTeamId: string;
  maxRows?: number;
}
```

#### B. Komponenty Strony Drużyny

```typescript
// Hero drużyny
interface TeamHeroProps {
  team: FootballTeam;
  currentForm: ('W' | 'D' | 'L')[];
}

// Lista meczów
interface MatchesListProps {
  matches: FootballMatch[];
  showTournament?: boolean;
  showResults?: boolean;
  pagination?: boolean;
}

// Tabela ligowa
interface StandingsTableProps {
  standings: FootballStandings[];
  highlightTeamId: string;
  showForm?: boolean;
  filterTypes?: ('total' | 'home' | 'away')[];
}

// Wykres formy (pozycji)
interface FormChartProps {
  data: {
    week: number;
    position: number;
  }[];
  maxPosition: number;
}

// Skład drużyny
interface SquadDisplayProps {
  players: FootballPlayer[];
  groupByPosition?: boolean;
}
```

### 9.6 Design System - Piłka Nożna

#### Kolory Statusów
```css
:root {
  /* Wyniki meczów */
  --match-win: #15B168;           /* Zielony - wygrana */
  --match-win-bg: #dcfce7;        /* Jasny zielony tło */
  
  --match-draw: #808080;          /* Szary - remis */
  --match-draw-bg: #f3f4f6;       /* Jasny szary tło */
  
  --match-loss: #C7361F;          /* Czerwony - przegrana */
  --match-loss-bg: #fee2e2;       /* Jasny czerwony tło */
  
  /* Pozycje w tabeli */
  --position-champions: #FFD700;  /* Złoty - Liga Mistrzów */
  --position-conference: #3B82F6; /* Niebieski - Liga Konferencji */
  --position-relegation: #EF4444; /* Czerwony - spadek */
  --position-normal: #9CA3AF;     /* Szary - pozostałe */
}
```

#### Rozmiary Elementów
```css
/* Logo drużyny */
--team-logo-xl: 96px;      /* Hero */
--team-logo-lg: 64px;      /* Widgety */
--team-logo-md: 40px;      /* Lista meczów */
--team-logo-sm: 24px;      /* Tabela */

/* Forma (ostatnie mecze) */
--form-badge-size: 24px;

/* Wynik meczu */
--score-font-size: 1.5rem;
--score-font-weight: 700;
```

### 9.7 API Endpoints

```typescript
// Drużyny
GET   /api/v1/football/teams?domain={domain}
POST  /api/v1/football/teams           // Dodaj drużynę
GET   /api/v1/football/teams/:id
PUT   /api/v1/football/teams/:id
DELETE /api/v1/football/teams/:id

// Mecze
GET   /api/v1/football/matches?team={teamId}&status={status}&limit={n}
GET   /api/v1/football/matches/next?team={teamId}
GET   /api/v1/football/matches/last?team={teamId}
GET   /api/v1/football/matches/today?domain={domain}  // Wszystkie mecze dziś

// Tabela
GET   /api/v1/football/standings?team={teamId}&type={total|home|away}
GET   /api/v1/football/standings/tournament/{tournamentId}?season={season}

// Skład
GET   /api/v1/football/squads?team={teamId}

// Forma (dane do wykresu)
GET   /api/v1/football/form?team={teamId}&weeks={n}
```

### 9.8 Struktura Podstron

#### URL: `/pilka-nozna` lub `/sport`
Lista wszystkich drużyn przypisanych do domeny.

#### URL: `/druzyna/{slug}` (np. `/druzyna/cracovia`)
Szczegóły drużyny z zakładkami:
- **Mecze** - lista wszystkich meczów
- **Tabela** - pozycja w ligowej tabeli
- **Skład** - lista zawodników
- **Statystyki** - szczegółowe statystyki

#### Sekcje na stronie drużyny:
1. **Hero** - logo, nazwa, trener, stadion, flaga kraju
2. **Najbliższe mecze** - lista z datami i wynikami
3. **Ostatnia forma** - wizualizacja ostatnich 5-10 meczów
4. **Wyróżniony mecz** - najbliższy ważny mecz z odliczaniem
5. **Tabela ligowa** - pozycja drużyny w kontekście ligi
6. **Wykres pozycji** - SVG pokazujący zmiany pozycji w czasie
7. **O drużynie** - opis SEO

### 9.9 Integracja z Panelem Admina

#### Nowe uprawnienia:
```typescript
'football:manage'           // Dostęp do modułu piłkarskiego
'football:teams:create'     // Dodawanie drużyn
'football:teams:edit'       // Edycja drużyn
'football:teams:delete'     // Usuwanie drużyn
'football:matches:edit'     // Edycja meczów (ręczna korekta)
'football:scrapers:manage'  // Zarządzanie scraperami
```

#### Nowe menu:
```
⚽ Piłka Nożna
├── 📋 Drużyny
│   ├── Lista drużyn
│   └── Dodaj drużynę
├── ⚽ Mecze
│   ├── Wszystkie mecze
│   ├── Nadchodzące
│   └── Ostatnie wyniki
├── 📊 Tabela ligowa
│   └── Konfiguracja
├── 👤 Składy
│   └── Zarządzanie zawodnikami
└── ⚙️ Scraper
    ├── Logi
    └── Konfiguracja
```

### 9.10 Zadania Implementacyjne (Dodatkowe do Planu)

#### Nowe zadania w Etapie 3 (Backend API):

| # | Zadanie | Estymacja | Zależności |
|---|---------|-----------|------------|
| 3.15 | **Modele bazy football_teams** | 2h | 3.1 |
| 3.16 | **Modele bazy football_matches** | 2h | 3.15 |
| 3.17 | **Modele bazy football_standings** | 2h | 3.15 |
| 3.18 | **API endpoints dla drużyn** | 4h | 3.15 |
| 3.19 | **API endpoints dla meczów** | 4h | 3.16 |
| 3.20 | **API endpoints dla tabeli** | 3h | 3.17 |
| 3.21 | **Walidacja danych piłkarskich** | 2h | 3.18-3.20 |

#### Nowe zadania w Etapie 4 (Scraping):

| # | Zadanie | Estymacja | Zależności |
|---|---------|-----------|------------|
| 4.17 | **Parser Sofascore HTML** | 8h | 4.1 |
| 4.18 | **Scraper meczów (Sofascore)** | 6h | 4.17 |
| 4.19 | **Scraper tabeli ligowej** | 4h | 4.17 |
| 4.20 | **Scraper składów** | 4h | 4.17 |
| 4.21 | **Scheduler dla scrapingu piłkarskiego** | 3h | 4.18-4.20 |
| 4.22 | **Obsługa cache dla danych piłkarskich** | 2h | 4.21 |

#### Nowe zadania w Etapie 5 (Frontend Publiczny):

| # | Zadanie | Estymacja | Zależności |
|---|---------|-----------|------------|
| 5.14 | **Komponent TeamLogo** | 2h | 5.1 |
| 5.15 | **Komponent MatchCard** | 4h | 5.14 |
| 5.16 | **Komponent MatchesList** | 4h | 5.15 |
| 5.17 | **Komponent StandingsTable** | 6h | 5.1 |
| 5.18 | **Komponent FormChart (SVG)** | 6h | 5.1 |
| 5.19 | **Komponent SquadDisplay** | 3h | 5.1 |
| 5.20 | **Widget NextMatch** | 3h | 5.15 |
| 5.21 | **Widget LastResult** | 3h | 5.15 |
| 5.22 | **Widget MiniTable** | 3h | 5.17 |
| 5.23 | **Strona drużyny (/druzyna/{slug})** | 6h | 5.16, 5.17, 5.18 |
| 5.24 | **Sekcja Sport na stronie głównej** | 4h | 5.20-5.22 |

#### Nowe zadania w Etapie 6 (Panel Admina):

| # | Zadanie | Estymacja | Zależności |
|---|---------|-----------|------------|
| 6.14 | **Lista drużyn w panelu** | 4h | 6.1, 3.18 |
| 6.15 | **Formularz dodawania/edycji drużyny** | 6h | 6.14 |
| 6.16 | **Wyszukiwarka drużyn Sofascore** | 4h | 6.15 |
| 6.17 | **Zarządzanie meczami** | 4h | 6.1, 3.19 |
| 6.18 | **Podgląd tabeli ligowej** | 3h | 6.1, 3.20 |
| 6.19 | **Konfiguracja scrapera piłkarskiego** | 4h | 6.15 |

---

**KONIEC DOKUMENTACJI**

*Wersja: 3.1 (Zintegrowana z modułem Piłkarskim)*  
*Data: 12 lutego 2026*  
*Autor: System Architect*  
*Status: Ready for Implementation*

---

