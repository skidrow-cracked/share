# REGIONALNE SERWISY - MASTERPLAN ARCHITEKTURY SYSTEMU

**Multi-Tenant Regional Portal Management System**

---

## SPIS TRESCI

### Podstawowa Dokumentacja

1. [Architektura Systemu - Wprowadzenie](#1-architektura-systemu---wprowadzenie)
2. [Struktura Domen i Hosting](#2-struktura-domen-i-hosting)
3. [Baza Danych - Architektura Multi-Tenant](#3-baza-danych---architektura-multi-tenant)
4. [System Uprawnien (RBAC)](#4-system-uprawnien-rbac)
5. [API - Specyfikacja Kompletna](#5-api---specyfikacja-kompletna)
6. [Panel Administracyjny - Centralny](#6-panel-administracyjny---centralny)
7. [Panel Administracyjny - Per Serwis](#7-panel-administracyjny---per-serwis)
8. [System Scrapingu i Cron Jobs](#8-system-scrapingu-i-cron-jobs)
9. [SEO i Struktury Danych](#9-seo-i-struktury-danych)
10. [Monitoring, Logi i Raportowanie](#10-monitoring-logi-i-raportowanie)
11. [Wdrozenie - Plan Etapowy](#11-wdrozenie---plan-etapowy)
12. [Bezpieczenstwo](#12-bezpieczenstwo)
13. [Zaleznosci i Korelacje](#13-zaleznosci-i-korelacje)
14. [Podsumowanie](#14-podsumowanie)

### Rozszerzona Dokumentacja (Nowe Sekcje)

21. [Strategia Testowania](#21-strategia-testowania-)
22. [CI/CD i DevOps](#22-cicd-i-devops-)
23. [Dokumentacja API (OpenAPI/Swagger)](#23-dokumentacja-api-openapiswagger-)
24. [Troubleshooting i FAQ](#24-troubleshooting-i-faq-)
25. [Backup i Disaster Recovery](#25-backup-i-disaster-recovery-)

---

## 1. ARCHITEKTURA SYSTEMU - WPROWADZENIE

### 1.1 Wizja Systemu

System **Regionalne Serwisy** to zaawansowana platforma multi-tenant pozwalajaca na zarzadzanie wieloma niezaleznymi serwisami regionalnymi z jednego centralnego panelu administracyjnego.

### 1.2 Glowne Zalozenia

| Aspekt | Opis |
|--------|------|
| **Multi-Tenancy** | Kazdy serwis to oddzielny tenant z wlasnymi danymi |
| **Centralne Zarzadzanie** | Jeden panel do sterowania wszystkimi serwisami |
| **Izolacja Danych** | Dane kazdego serwisu sa logicznie odseparowane |
| **Wspoldzielenie Zasobow** | Wspolne szablony, moduly, zrodla danych |
| **Skalowalnosc** | Latwe dodawanie nowych serwisow |
| **Elastycznosc** | Mozliwosc indywidualnej konfiguracji kazdego serwisu |

### 1.3 Diagram Architektury High-Level

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         UZYTKOWNICY SYSTEMU                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │ Super Admin │  │   Admin     │  │  Redaktor   │  │   Uzytkownik    │ │
│  │ (System)    │  │ (Serwis)    │  │ (Content)   │  │   (Public)      │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └─────────────────┘ │
└─────────┼────────────────┼────────────────┼─────────────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    CENTRALNY PANEL STEROWANIA                           │
│                    serwisy-lokalne-sterowanie.pl                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  Dashboard │ Domeny │ Uzytkownicy │ Uprawnienia │ Ustawienia    │  │
│  │  Szablony  │ Moduly │ Zrodla      │ Cron        │ Logi          │  │
│  │  Statystyki│ Raporty│ SEO Global  │ Backup      │ Monitoring    │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          │                     │                     │
          ▼                     ▼                     ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  SERWIS 1       │   │  SERWIS 2       │   │  SERWIS N       │
│ 4torun.pl       │   │ 4bydgoszcz.pl   │   │ 4warszawa.pl    │
├─────────────────┤   ├─────────────────┤   ├─────────────────┤
│ - Wiadomosci    │   │ - Wiadomosci    │   │ - Wiadomosci    │
│ - Kronika       │   │ - Kronika       │   │ - Kronika       │
│ - Firmy         │   │ - Firmy         │   │ - Firmy         │
│ - Praca         │   │ - Praca         │   │ - Praca         │
│ - Nekrologi     │   │ - Nekrologi     │   │ - Nekrologi     │
│ - Przewodnik    │   │ - Przewodnik    │   │ - Przewodnik    │
└─────────────────┘   └─────────────────┘   └─────────────────┘
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      WSPOLNA INFRASTRUKTURA                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  PostgreSQL  │  │    Redis     │  │ Elasticsearch│  │   MinIO      │ │
│  │  (Database)  │  │   (Cache)    │  │   (Search)   │  │  (Storage)   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  RabbitMQ    │  │   Puppeteer  │  │   Python     │  │   Node.js    │ │
│  │  (Queue)     │  │  (Scraper)   │  │  (Workers)   │  │    (API)     │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.4 Struktura Katalogów na Hosting

```
/home/host988956/
├── domains/
│   ├── serwisy-lokalne-sterowanie.pl/
│   │   ├── public_html/           # Panel centralny (Next.js/React)
│   │   ├── private/
│   │   │   ├── config/
│   │   │   ├── logs/
│   │   │   └── temp/
│   │   └── backup/
│   │
│   ├── 4torun.pl/
│   │   ├── public_html/           # Frontend serwisu (SSG/SSR)
│   │   ├── private/
│   │   │   ├── cache/
│   │   │   ├── uploads/
│   │   │   └── logs/
│   │   └── backup/
│   │
│   ├── 4bydgoszcz.pl/
│   │   ├── public_html/
│   │   └── ...
│   │
│   └── [n-domen-regionalnych]/
│
├── shared/                        # Wspolne zasoby
│   ├── templates/                 # Szablony wspolne
│   ├── modules/                   # Moduly wspolne
│   ├── assets/                    # Grafiki, fonty
│   └── scripts/                   # Skrypty wspolne
│
├── workers/                       # Python workers
│   ├── scrapers/
│   ├── processors/
│   └── cron/
│
├── logs/                          # Logi systemowe
│   ├── system/
│   ├── access/
│   └── error/
│
└── config/                        # Konfiguracja globalna
    ├── database.yml
    ├── services.yml
    └── security.yml
```

---

## 2. STRUKTURA DOMEN I HOSTING

### 2.1 Konfiguracja Domen

#### Domena Centralna (Panel Sterowania)
```
Domena: serwisy-lokalne-sterowanie.pl
Typ: Aplikacja administracyjna (SPA/SSR)
Technologia: Next.js 14 + React 18
```

#### Domeny Regionalne (Serwisy)
```
Przyklady:
- 4torun.pl
- 4bydgoszcz.pl
- 4warszawa.pl
- 4gdansk.pl
- itp.

Typ: Serwisy publiczne (SSG/SSR)
Technologia: Next.js 14 + React 18
```

### 2.2 Struktura URL - Panel Centralny

```
# DASHBOARD
GET  /                                    # Dashboard glowny
GET  /dashboard                           # Statystyki wszystkich serwisow
GET  /dashboard/metrics                   # Metryki systemowe
GET  /dashboard/alerts                    # Alerty i powiadomienia

# ZARZADZANIE DOMENAMI
GET  /admin/domeny                        # Lista wszystkich domen
POST /admin/domeny                        # Dodaj nowa domene
GET  /admin/domeny/:domainId              # Szczegoly domeny
PUT  /admin/domeny/:domainId              # Edytuj domene
DELETE /admin/domeny/:domainId            # Usun domene
GET  /admin/domeny/:domainId/dashboard    # Dashboard domeny
GET  /admin/domeny/:domainId/statystyki   # Statystyki domeny
GET  /admin/domeny/:domainId/logi         # Logi domeny

# PRZEKIEROWANIE DO ZARZADZANIA KONKRETNA DOMENA
GET  /admin/domeny/:domainId/content/*    # Zarzadzanie trescia
GET  /admin/domeny/:domainId/users/*      # Zarzadzanie uzytkownikami
GET  /admin/domeny/:domainId/settings/*   # Ustawienia domeny
GET  /admin/domeny/:domainId/seo/*        # SEO domeny
GET  /admin/domeny/:domainy/design/*      # Wyglad i szablony

# GLOBALNE USTAWIENIA (WSZYSTKIE DOMENY)
GET  /admin/global/banners                # Globalne bannery
POST /admin/global/banners                # Dodaj globalny banner
GET  /admin/global/menus                  # Globalne menu
POST /admin/global/menus                  # Dodaj globalne menu
GET  /admin/global/widgets                # Globalne widgety
POST /admin/global/widgets                # Dodaj globalny widget
GET  /admin/global/sources                # Globalne zrodla danych
POST /admin/global/sources                # Dodaj globalne zrodlo

# MASOWE OPERACJE
POST /admin/mass/banners                  # Dodaj banner do wielu domen
POST /admin/mass/menus                    # Dodaj menu do wielu domen
POST /admin/mass/content                  # Dodaj tresc do wielu domen
POST /admin/mass/updates                  # Aktualizacja wielu domen
POST /admin/mass/backup                   # Backup wielu domen

# ZRODLA DANYCH (SCRAPING)
GET  /admin/sources                       # Lista zrodel
POST /admin/sources                       # Dodaj zrodlo
GET  /admin/sources/:sourceId             # Szczegoly zrodla
PUT  /admin/sources/:sourceId             # Edytuj zrodlo
DELETE /admin/sources/:sourceId           # Usun zrodlo
GET  /admin/sources/:sourceId/logs        # Logi scrapingu
POST /admin/sources/:sourceId/run         # Uruchom scraping
GET  /admin/sources/:sourceId/schedule    # Harmonogram

# CRON JOBS
GET  /admin/cron                          # Lista zadan cron
POST /admin/cron                          # Dodaj zadanie
GET  /admin/cron/:jobId                   # Szczegoly zadania
PUT  /admin/cron/:jobId                   # Edytuj zadanie
DELETE /admin/cron/:jobId                 # Usun zadanie
GET  /admin/cron/:jobId/logs              # Logi wykonania
POST /admin/cron/:jobId/run               # Uruchom recznie

# UZYTKOWNICY SYSTEMU
GET  /admin/users                         # Lista uzytkownikow
POST /admin/users                         # Dodaj uzytkownika
GET  /admin/users/:userId                 # Szczegoly uzytkownika
PUT  /admin/users/:userId                 # Edytuj uzytkownika
DELETE /admin/users/:userId               # Usun uzytkownika
GET  /admin/users/:userId/permissions     # Uprawnienia uzytkownika
PUT  /admin/users/:userId/permissions     # Zmien uprawnienia
GET  /admin/users/:userId/activity        # Aktywnosc uzytkownika

# ROLE I UPRAWNIENIA
GET  /admin/roles                         # Lista rol
POST /admin/roles                         # Dodaj role
GET  /admin/roles/:roleId                 # Szczegoly roli
PUT  /admin/roles/:roleId                 # Edytuj role
DELETE /admin/roles/:roleId               # Usun role
GET  /admin/permissions                   # Lista uprawnien

# SZABLONY
GET  /admin/templates                     # Lista szablonow
POST /admin/templates                     # Dodaj szablon
GET  /admin/templates/:templateId         # Szczegoly szablonu
PUT  /admin/templates/:templateId         # Edytuj szablon
DELETE /admin/templates/:templateId       # Usun szablon
POST /admin/templates/:templateId/apply   # Zastosuj do domen

# MODULY
GET  /admin/modules                       # Lista modulow
POST /admin/modules                       # Dodaj modul
GET  /admin/modules/:moduleId             # Szczegoly modulu
PUT  /admin/modules/:moduleId             # Edytuj modul
DELETE /admin/modules/:moduleId           # Usun modul
POST /admin/modules/:moduleId/activate    # Aktywuj w domenach

# STATYSTYKI I RAPORTY
GET  /admin/statistics                    # Statystyki ogolne
GET  /admin/statistics/traffic            # Ruch na stronach
GET  /admin/statistics/content            # Statystyki tresci
GET  /admin/statistics/users              # Aktywnosc uzytkownikow
GET  /admin/statistics/seo                # Wyniki SEO
GET  /admin/reports                       # Raporty
POST /admin/reports/generate              # Generuj raport
GET  /admin/reports/:reportId             # Pobierz raport

# LOGI SYSTEMOWE
GET  /admin/logs                          # Logi systemowe
GET  /admin/logs/system                   # Logi aplikacji
GET  /admin/logs/access                   # Logi dostepu
GET  /admin/logs/error                    # Logi bledow
GET  /admin/logs/scraper                  # Logi scrapera
GET  /admin/logs/cron                     # Logi cron

# BACKUP I RESTORE
GET  /admin/backup                        # Lista backupow
POST /admin/backup/create                 # Utworz backup
POST /admin/backup/:backupId/restore      # Przywroc backup
DELETE /admin/backup/:backupId            # Usun backup
POST /admin/backup/schedule               # Harmonogram backup

# USTAWIENIA SYSTEMU
GET  /admin/settings                      # Ustawienia globalne
PUT  /admin/settings                      # Zapisz ustawienia
GET  /admin/settings/email                # Ustawienia email
PUT  /admin/settings/email                # Zapisz ustawienia email
GET  /admin/settings/security             # Ustawienia bezpieczenstwa
PUT  /admin/settings/security             # Zapisz ustawienia security
GET  /admin/settings/integrations         # Integracje zewnetrzne
PUT  /admin/settings/integrations         # Zapisz integracje

# API TOKENS
GET  /admin/api-tokens                    # Lista tokenow
POST /admin/api-tokens                    # Generuj token
DELETE /admin/api-tokens/:tokenId         # Uniewaznij token
```

### 2.3 Struktura URL - Serwis Regionalny (Przyklad)

```
# STRONA GLOWNA
GET  /                                    # Strona glowna
GET  /wiadomosci                          # Lista wiadomosci
GET  /wiadomosci/:slug                    # Pojedyncza wiadomosc
GET  /kronika-policyjna                   # Kronika policyjna
GET  /kronika-policyjna/:slug             # Pojedynczy wpis
GET  /firmy                               # Katalog firm
GET  /firmy/:category/:slug               # Profil firmy
GET  /ogloszenia                          # Ogloszenia
GET  /ogloszenia/:slug                    # Ogloszenie
GET  /praca                               # Oferty pracy
GET  /praca/:slug                         # Oferta pracy
GET  /nekrologi                           # Nekrologi
GET  /nekrologi/:slug                     # Nekrolog
GET  /przewodnik                          # Przewodnik
GET  /przewodnik/:slug                    # Miejsce w przewodniku
GET  /ludzie                              # Ludzie
GET  /ludzie/:slug                        # Profil osoby

# PANEL ADMINISTRACYJNY SERWISU
GET  /admin                               # Dashboard serwisu
GET  /admin/wiadomosci                    # Lista wpisow
POST /admin/wiadomosci                    # Dodaj wpis
GET  /admin/wiadomosci/:id                # Edytuj wpis
PUT  /admin/wiadomosci/:id                # Zapisz zmiany
DELETE /admin/wiadomosci/:id              # Usun wpis
GET  /admin/kronika-policyjna             # Lista (analogicznie)
GET  /admin/firmy                         # Lista firm
GET  /admin/ogloszenia                    # Lista ogloszen
GET  /admin/praca                         # Lista ofert pracy
GET  /admin/nekrologi                     # Lista nekrologow
GET  /admin/przewodnik                    # Lista miejsc
GET  /admin/ludzie                        # Lista osob

# USTAWIENIA SERWISU
GET  /admin/settings                      # Ustawienia serwisu
PUT  /admin/settings                      # Zapisz ustawienia
GET  /admin/settings/seo                  # Ustawienia SEO
GET  /admin/settings/design               # Wyglad
GET  /admin/settings/banners              # Bannery
GET  /admin/settings/menus                # Menu
GET  /admin/settings/widgets              # Widgety

# UZYTKOWNICY SERWISU
GET  /admin/users                         # Lista uzytkownikow serwisu
POST /admin/users                         # Dodaj uzytkownika
GET  /admin/users/:userId                 # Szczegoly
PUT  /admin/users/:userId                 # Edytuj
DELETE /admin/users/:userId               # Usun

# STATYSTYKI SERWISU
GET  /admin/statistics                    # Statystyki
GET  /admin/statistics/traffic            # Ruch
GET  /admin/statistics/content            # Tresci
GET  /admin/statistics/seo                # SEO

# LOGI SERWISU
GET  /admin/logs                          # Logi
GET  /admin/logs/content                  # Logi tresci
GET  /admin/logs/users                    # Logi uzytkownikow
```

---

## 3. BAZA DANYCH - ARCHITEKTURA MULTI-TENANT

### 3.1 Strategia Multi-Tenancy

Wybrana strategia: **Shared Database, Separate Schema per Tenant**

```
Database: regional_services
│
├── Schema: public                          # Dane wspolne (centrala)
│   ├── users                               # Uzytkownicy systemu
│   ├── roles                               # Role globalne
│   ├── permissions                         # Uprawnienia
│   ├── user_roles                          # Powiazania user-role
│   ├── role_permissions                    # Powiazania role-permission
│   ├── templates                           # Szablony wspolne
│   ├── modules                             # Moduly systemu
│   ├── sources                             # Zrodla danych (scraping)
│   ├── cron_jobs                           # Zadania cron
│   ├── api_tokens                          # Tokeny API
│   ├── system_logs                         # Logi systemowe
│   └── global_settings                     # Ustawienia globalne
│
├── Schema: tenant_4torun_pl                # Dane 4torun.pl
│   ├── posts                               # Wpisy (CPT)
│   ├── post_meta                           # Meta dane wpisow
│   ├── categories                          # Kategorie
│   ├── tags                                # Tagi
│   ├── comments                            # Komentarze
│   ├── ratings                             # Oceny
│   ├── banners                             # Bannery
│   ├── menus                               # Menu
│   ├── menu_items                          # Pozycje menu
│   ├── widgets                             # Widgety
│   ├── companies                           # Firmy
│   ├── jobs                                # Oferty pracy
│   ├── obituaries                          # Nekrologi
│   ├── guide_items                         # Przewodnik
│   ├── people                              # Ludzie
│   ├── attachments                         # Zalaczniki
│   ├── seo_meta                            # Meta dane SEO
│   ├── redirects                           # Przekierowania
│   ├── custom_fields                       # Pola niestandardowe
│   ├── audit_logs                          # Logi audytowe
│   └── domain_settings                     # Ustawienia domeny
│
├── Schema: tenant_4bydgoszcz_pl            # Dane 4bydgoszcz.pl
│   └── [analogiczne tabele]
│
└── Schema: tenant_[n]                      # Dane kolejnych domen
    └── [analogiczne tabele]
```



### 3.2 Schemat Bazy Danych - SQL DDL

```sql
-- =====================================================
-- SCHEMA: PUBLIC (DANE CENTRALNE)
-- =====================================================

-- Tabela uzytkownikow systemu (globalna)
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
    email_verified_at TIMESTAMP,
    last_login_at TIMESTAMP,
    login_count INTEGER DEFAULT 0,
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP,
    preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);

-- Tabela rol (globalna)
CREATE TABLE public.roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    level INTEGER DEFAULT 0, -- 0=super_admin, 1=admin, 2=editor, 3=moderator, 4=user
    is_system BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Wstawienie domyslnych rol
INSERT INTO public.roles (name, slug, description, level, is_system) VALUES
('Super Administrator', 'super_admin', 'Pelny dostep do calego systemu', 0, true),
('Administrator Domeny', 'domain_admin', 'Zarzadzanie konkretna domena', 1, true),
('Redaktor', 'editor', 'Zarzadzanie trescia', 2, true),
('Moderator', 'moderator', 'Moderacja komentarzy i ocen', 3, true),
('Uzytkownik', 'user', 'Standardowy uzytkownik', 4, true);

-- Tabela uprawnien (globalna)
CREATE TABLE public.permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) UNIQUE NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    resource VARCHAR(50) NOT NULL, -- np. 'posts', 'users', 'domains'
    action VARCHAR(50) NOT NULL,   -- np. 'create', 'read', 'update', 'delete', 'manage'
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela powiazan uzytkownik-rola (z wskazaniem domeny)
CREATE TABLE public.user_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
    role_id UUID NOT NULL REFERENCES public.roles(id) ON DELETE CASCADE,
    domain_id VARCHAR(100), -- NULL = globalna rola, '4torun.pl' = rola w domenie
    granted_by UUID REFERENCES public.users(id),
    expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, role_id, domain_id)
);

-- Tabela powiazan rola-uprawnienie
CREATE TABLE public.role_permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    role_id UUID NOT NULL REFERENCES public.roles(id) ON DELETE CASCADE,
    permission_id UUID NOT NULL REFERENCES public.permissions(id) ON DELETE CASCADE,
    domain_id VARCHAR(100), -- NULL = globalnie, lub konkretna domena
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(role_id, permission_id, domain_id)
);

-- Tabela domen/serwisow
CREATE TABLE public.domains (
    id VARCHAR(100) PRIMARY KEY, -- np. '4torun.pl'
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    city VARCHAR(100) NOT NULL,
    region VARCHAR(100),
    country VARCHAR(100) DEFAULT 'Polska',
    language VARCHAR(10) DEFAULT 'pl',
    timezone VARCHAR(50) DEFAULT 'Europe/Warsaw',
    currency VARCHAR(10) DEFAULT 'PLN',
    is_active BOOLEAN DEFAULT true,
    is_maintenance BOOLEAN DEFAULT false,
    maintenance_message TEXT,
    schema_name VARCHAR(100) NOT NULL, -- np. 'tenant_4torun_pl'
    database_connection VARCHAR(100) DEFAULT 'default',
    template_id UUID,
    primary_color VARCHAR(7) DEFAULT '#1e3a5f',
    secondary_color VARCHAR(7) DEFAULT '#dd6b20',
    logo_url TEXT,
    favicon_url TEXT,
    social_links JSONB DEFAULT '{}',
    contact_info JSONB DEFAULT '{}',
    seo_settings JSONB DEFAULT '{}',
    analytics_settings JSONB DEFAULT '{}',
    scraping_settings JSONB DEFAULT '{}',
    features_enabled JSONB DEFAULT '{}', -- { "comments": true, "ratings": true }
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id),
    expires_at TIMESTAMP
);

-- Tabela szablonow
CREATE TABLE public.templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    version VARCHAR(20) DEFAULT '1.0.0',
    is_active BOOLEAN DEFAULT true,
    is_default BOOLEAN DEFAULT false,
    structure JSONB NOT NULL, -- definicja struktury szablonu
    styles JSONB DEFAULT '{}', -- zmienne CSS
    components JSONB DEFAULT '{}', -- dostepne komponenty
    files_path TEXT, -- sciezka do plikow szablonu
    preview_image_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

-- Tabela modulow
CREATE TABLE public.modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    version VARCHAR(20) DEFAULT '1.0.0',
    is_active BOOLEAN DEFAULT true,
    is_core BOOLEAN DEFAULT false, -- czy modul jest wymagany
    config_schema JSONB, -- schema konfiguracji
    default_config JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela powiazan domena-modul
CREATE TABLE public.domain_modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    domain_id VARCHAR(100) NOT NULL REFERENCES public.domains(id) ON DELETE CASCADE,
    module_id UUID NOT NULL REFERENCES public.modules(id) ON DELETE CASCADE,
    config JSONB DEFAULT '{}',
    is_active BOOLEAN DEFAULT true,
    order_index INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(domain_id, module_id)
);

-- Tabela zrodel danych (scraping)
CREATE TABLE public.sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL, -- 'rss', 'api', 'scraper', 'xml', 'json'
    url TEXT NOT NULL,
    method VARCHAR(10) DEFAULT 'GET',
    headers JSONB DEFAULT '{}',
    auth_type VARCHAR(20), -- 'none', 'basic', 'bearer', 'api_key'
    auth_config JSONB,
    fetch_config JSONB DEFAULT '{}', -- { "timeout": 30, "retries": 3 }
    parser_config JSONB NOT NULL, -- konfiguracja parsera
    mapping_config JSONB NOT NULL, -- mapowanie pol
    schedule VARCHAR(100) DEFAULT '0 */6 * * *', -- cron expression
    is_active BOOLEAN DEFAULT true,
    last_run_at TIMESTAMP,
    last_run_status VARCHAR(20), -- 'success', 'error', 'running'
    last_run_message TEXT,
    last_run_items_count INTEGER DEFAULT 0,
    total_runs INTEGER DEFAULT 0,
    total_items_fetched INTEGER DEFAULT 0,
    error_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

-- Tabela powiazan zrodlo-domena (ktore domeny uzywaja zrodla)
CREATE TABLE public.source_domains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id UUID NOT NULL REFERENCES public.sources(id) ON DELETE CASCADE,
    domain_id VARCHAR(100) NOT NULL REFERENCES public.domains(id) ON DELETE CASCADE,
    is_active BOOLEAN DEFAULT true,
    custom_config JSONB DEFAULT '{}',
    last_run_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(source_id, domain_id)
);

-- Tabela zadan cron
CREATE TABLE public.cron_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    schedule VARCHAR(100) NOT NULL, -- cron expression
    command TEXT NOT NULL, -- komenda do wykonania
    arguments JSONB DEFAULT '{}',
    is_active BOOLEAN DEFAULT true,
    run_as_user VARCHAR(100) DEFAULT 'www-data',
    timeout_seconds INTEGER DEFAULT 300,
    retry_count INTEGER DEFAULT 0,
    retry_delay_seconds INTEGER DEFAULT 60,
    notify_on_failure BOOLEAN DEFAULT true,
    notify_emails TEXT[],
    last_run_at TIMESTAMP,
    last_run_status VARCHAR(20),
    last_run_duration_ms INTEGER,
    last_run_output TEXT,
    last_run_error TEXT,
    next_run_at TIMESTAMP,
    run_count INTEGER DEFAULT 0,
    fail_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

-- Tabela logow zadan cron
CREATE TABLE public.cron_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    cron_job_id UUID NOT NULL REFERENCES public.cron_jobs(id) ON DELETE CASCADE,
    started_at TIMESTAMP NOT NULL,
    finished_at TIMESTAMP,
    status VARCHAR(20) NOT NULL, -- 'running', 'success', 'error', 'timeout'
    duration_ms INTEGER,
    output TEXT,
    error TEXT,
    triggered_by VARCHAR(50) DEFAULT 'scheduler' -- 'scheduler', 'manual'
);

-- Tabela tokenow API
CREATE TABLE public.api_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    domain_id VARCHAR(100) REFERENCES public.domains(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    token_hash VARCHAR(255) UNIQUE NOT NULL,
    token_preview VARCHAR(20) NOT NULL, -- pierwsze 20 znakow tokena
    scopes TEXT[] DEFAULT '{}',
    expires_at TIMESTAMP,
    last_used_at TIMESTAMP,
    use_count INTEGER DEFAULT 0,
    ip_restrictions TEXT[],
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

-- Tabela logow systemowych (centralna)
CREATE TABLE public.system_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    level VARCHAR(20) NOT NULL, -- 'debug', 'info', 'warning', 'error', 'critical'
    category VARCHAR(50) NOT NULL, -- 'auth', 'database', 'scraper', 'api', 'system'
    domain_id VARCHAR(100) REFERENCES public.domains(id),
    user_id UUID REFERENCES public.users(id),
    action VARCHAR(100) NOT NULL,
    message TEXT NOT NULL,
    context JSONB DEFAULT '{}',
    ip_address INET,
    user_agent TEXT,
    request_id UUID,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indeksy dla system_logs
CREATE INDEX idx_system_logs_level ON public.system_logs(level);
CREATE INDEX idx_system_logs_category ON public.system_logs(category);
CREATE INDEX idx_system_logs_domain ON public.system_logs(domain_id);
CREATE INDEX idx_system_logs_user ON public.system_logs(user_id);
CREATE INDEX idx_system_logs_created ON public.system_logs(created_at);
CREATE INDEX idx_system_logs_action ON public.system_logs(action);

-- Tabela ustawien globalnych
CREATE TABLE public.global_settings (
    id VARCHAR(100) PRIMARY KEY,
    category VARCHAR(50) NOT NULL,
    key VARCHAR(100) NOT NULL,
    value JSONB,
    description TEXT,
    is_editable BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID REFERENCES public.users(id),
    UNIQUE(category, key)
);

-- Partycjonowanie system_logs po miesiacach
CREATE TABLE public.system_logs_y2024m01 PARTITION OF public.system_logs
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE public.system_logs_y2024m02 PARTITION OF public.system_logs
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
-- ... i tak dalej dla kolejnych miesiecy

```



### 3.3 Schemat Bazy Danych - Tenant (Pojedynczy Serwis)

```sql
-- =====================================================
-- SCHEMA: TENANT (DANE POJEDYNCZEGO SERWISU)
-- Uwaga: Ta struktura jest tworzona dla kazdej domeny
-- =====================================================

-- Tabela wpisow (Custom Post Types)
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_type VARCHAR(50) NOT NULL, -- 'wiadomosci', 'kronika-policyjna', 'firmy', itp.
    title VARCHAR(500) NOT NULL,
    slug VARCHAR(500) UNIQUE NOT NULL,
    excerpt TEXT,
    content TEXT NOT NULL,
    content_rendered TEXT, -- przetworzony HTML
    status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'published', 'scheduled', 'archived'
    visibility VARCHAR(20) DEFAULT 'public', -- 'public', 'private', 'password'
    password_hash VARCHAR(255),
    featured BOOLEAN DEFAULT false,
    sticky BOOLEAN DEFAULT false,
    allow_comments BOOLEAN DEFAULT true,
    allow_ratings BOOLEAN DEFAULT true,
    view_count INTEGER DEFAULT 0,
    unique_view_count INTEGER DEFAULT 0,
    rating_sum INTEGER DEFAULT 0,
    rating_count INTEGER DEFAULT 0,
    rating_average DECIMAL(3,2) DEFAULT 0,
    comment_count INTEGER DEFAULT 0,
    author_id UUID,
    published_at TIMESTAMP,
    scheduled_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP,
    created_by UUID REFERENCES public.users(id),
    updated_by UUID REFERENCES public.users(id),
    published_by UUID REFERENCES public.users(id),
    source_id UUID REFERENCES public.sources(id),
    source_url TEXT,
    source_data JSONB, -- oryginalne dane ze zrodla
    external_id VARCHAR(255), -- ID w zewnetrznym systemie
    seo_title VARCHAR(500),
    seo_description TEXT,
    seo_keywords TEXT[],
    og_title VARCHAR(500),
    og_description TEXT,
    og_image_url TEXT,
    schema_type VARCHAR(50) DEFAULT 'NewsArticle',
    schema_data JSONB,
    template VARCHAR(100) DEFAULT 'default',
    sort_order INTEGER DEFAULT 0,
    lang VARCHAR(10) DEFAULT 'pl'
);

-- Indeksy dla posts
CREATE INDEX idx_posts_type ON posts(post_type);
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_slug ON posts(slug);
CREATE INDEX idx_posts_featured ON posts(featured) WHERE featured = true;
CREATE INDEX idx_posts_sticky ON posts(sticky) WHERE sticky = true;
CREATE INDEX idx_posts_published ON posts(published_at);
CREATE INDEX idx_posts_created ON posts(created_at);
CREATE INDEX idx_posts_search ON posts USING gin(to_tsvector('polish', title || ' ' || COALESCE(content, '')));

-- Tabela meta danych wpisow
CREATE TABLE post_meta (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    meta_key VARCHAR(100) NOT NULL,
    meta_value JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, meta_key)
);

CREATE INDEX idx_post_meta_key ON post_meta(meta_key);
CREATE INDEX idx_post_meta_post ON post_meta(post_id);

-- Tabela kategorii
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_id UUID REFERENCES categories(id),
    post_type VARCHAR(50) NOT NULL, -- dla jakiego CPT
    color VARCHAR(7),
    icon VARCHAR(50),
    image_url TEXT,
    seo_title VARCHAR(500),
    seo_description TEXT,
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    item_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_type ON categories(post_type);
CREATE INDEX idx_categories_slug ON categories(slug);
CREATE INDEX idx_categories_parent ON categories(parent_id);

-- Tabela powiazan wpis-kategoria
CREATE TABLE post_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    category_id UUID NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    is_primary BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, category_id)
);

CREATE INDEX idx_post_categories_post ON post_categories(post_id);
CREATE INDEX idx_post_categories_cat ON post_categories(category_id);

-- Tabela tagow
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7),
    seo_title VARCHAR(500),
    seo_description TEXT,
    item_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tags_slug ON tags(slug);
CREATE INDEX idx_tags_search ON tags USING gin(to_tsvector('polish', name));

-- Tabela powiazan wpis-tag
CREATE TABLE post_tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, tag_id)
);

CREATE INDEX idx_post_tags_post ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag ON post_tags(tag_id);

-- Tabela komentarzy
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES comments(id),
    author_name VARCHAR(100),
    author_email VARCHAR(255),
    author_ip INET,
    author_user_agent TEXT,
    content TEXT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'approved', 'rejected', 'spam'
    is_verified BOOLEAN DEFAULT false,
    like_count INTEGER DEFAULT 0,
    dislike_count INTEGER DEFAULT 0,
    reply_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    moderated_at TIMESTAMP,
    moderated_by UUID REFERENCES public.users(id)
);

CREATE INDEX idx_comments_post ON comments(post_id);
CREATE INDEX idx_comments_status ON comments(status);
CREATE INDEX idx_comments_parent ON comments(parent_id);
CREATE INDEX idx_comments_created ON comments(created_at);

-- Tabela ocen/ratingow
CREATE TABLE ratings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id UUID, -- NULL = anonimowy
    ip_address INET,
    user_agent TEXT,
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(post_id, user_id) WHERE user_id IS NOT NULL,
    UNIQUE(post_id, ip_address) WHERE user_id IS NULL
);

CREATE INDEX idx_ratings_post ON ratings(post_id);
CREATE INDEX idx_ratings_user ON ratings(user_id);

-- Tabela bannerow
CREATE TABLE banners (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'image', 'html', 'script'
    position VARCHAR(50) NOT NULL, -- 'header', 'sidebar_top', 'sidebar_bottom', 'content_top', 'content_bottom', 'footer', 'popup'
    content TEXT NOT NULL, -- HTML, URL obrazka lub kod skryptu
    image_url TEXT,
    link_url TEXT,
    alt_text VARCHAR(255),
    target VARCHAR(20) DEFAULT '_blank', -- '_blank', '_self'
    weight INTEGER DEFAULT 0, -- priorytet wyswietlania
    start_date TIMESTAMP,
    end_date TIMESTAMP,
    display_rules JSONB DEFAULT '{}', -- { "pages": ["home", "post"], "post_types": ["wiadomosci"], "devices": ["desktop", "mobile"] }
    impression_limit INTEGER,
    click_limit INTEGER,
    impression_count INTEGER DEFAULT 0,
    click_count INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

CREATE INDEX idx_banners_position ON banners(position);
CREATE INDEX idx_banners_active ON banners(is_active) WHERE is_active = true;

-- Tabela menu
CREATE TABLE menus (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    location VARCHAR(50) NOT NULL, -- 'header', 'footer', 'sidebar', 'mobile'
    description TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela pozycji menu
CREATE TABLE menu_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    menu_id UUID NOT NULL REFERENCES menus(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES menu_items(id),
    title VARCHAR(200) NOT NULL,
    url TEXT NOT NULL,
    target VARCHAR(20) DEFAULT '_self',
    icon VARCHAR(50),
    css_class VARCHAR(100),
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true,
    display_rules JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_menu_items_menu ON menu_items(menu_id);
CREATE INDEX idx_menu_items_parent ON menu_items(parent_id);
CREATE INDEX idx_menu_items_order ON menu_items(sort_order);

-- Tabela widgetow
CREATE TABLE widgets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'recent_posts', 'popular_posts', 'categories', 'tags', 'custom_html', 'weather', 'air_quality', 'social', 'newsletter', 'search'
    position VARCHAR(50) NOT NULL, -- 'sidebar', 'header', 'footer', 'content'
    area VARCHAR(50) NOT NULL, -- 'sidebar_main', 'sidebar_secondary', 'footer_1', 'footer_2', etc.
    config JSONB DEFAULT '{}', -- konfiguracja specyficzna dla typu
    content TEXT, -- dla typu custom_html
    sort_order INTEGER DEFAULT 0,
    display_rules JSONB DEFAULT '{}',
    cache_time INTEGER DEFAULT 3600, -- czas cache w sekundach
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_widgets_position ON widgets(position);
CREATE INDEX idx_widgets_area ON widgets(area);
CREATE INDEX idx_widgets_active ON widgets(is_active) WHERE is_active = true;

-- Tabela firm (rozszerzone pola dla CPT 'firmy')
CREATE TABLE companies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    category_id UUID REFERENCES categories(id),
    description TEXT,
    address JSONB, -- { "street": "", "city": "", "postal": "", "country": "" }
    phone VARCHAR(50)[],
    email VARCHAR(255)[],
    website VARCHAR(255),
    social_links JSONB,
    opening_hours JSONB, -- { "monday": { "open": "08:00", "close": "16:00" } }
    logo_url TEXT,
    gallery_urls TEXT[],
    coordinates POINT, -- [lat, lng]
    is_verified BOOLEAN DEFAULT false,
    is_premium BOOLEAN DEFAULT false,
    premium_expires_at TIMESTAMP,
    rating_average DECIMAL(3,2) DEFAULT 0,
    rating_count INTEGER DEFAULT 0,
    view_count INTEGER DEFAULT 0,
    metadata JSONB, -- dodatkowe pola niestandardowe
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_companies_category ON companies(category_id);
CREATE INDEX idx_companies_premium ON companies(is_premium) WHERE is_premium = true;
CREATE INDEX idx_companies_location ON companies USING gist(coordinates);

-- Tabela ofert pracy (rozszerzone pola dla CPT 'praca')
CREATE TABLE jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    title VARCHAR(300) NOT NULL,
    slug VARCHAR(300) UNIQUE NOT NULL,
    company_id UUID REFERENCES companies(id),
    company_name VARCHAR(200),
    company_logo_url TEXT,
    location JSONB, -- { "city": "", "region": "", "country": "", "remote": false }
    job_type VARCHAR(50), -- 'full_time', 'part_time', 'contract', 'internship', 'remote'
    salary JSONB, -- { "min": 5000, "max": 8000, "currency": "PLN", "period": "month", "negotiable": true }
    description TEXT,
    requirements TEXT[],
    responsibilities TEXT[],
    benefits TEXT[],
    application_url TEXT,
    application_email VARCHAR(255),
    application_deadline TIMESTAMP,
    external_id VARCHAR(255),
    external_source VARCHAR(100),
    external_url TEXT,
    is_featured BOOLEAN DEFAULT false,
    view_count INTEGER DEFAULT 0,
    application_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_jobs_company ON jobs(company_id);
CREATE INDEX idx_jobs_type ON jobs(job_type);
CREATE INDEX idx_jobs_featured ON jobs(is_featured) WHERE is_featured = true;
CREATE INDEX idx_jobs_deadline ON jobs(application_deadline);

-- Tabela nekrologow (rozszerzone pola dla CPT 'nekrologi')
CREATE TABLE obituaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    person_name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    birth_date DATE,
    death_date DATE NOT NULL,
    age INTEGER,
    photo_url TEXT,
    biography TEXT,
    funeral_info JSONB, -- { "location": "", "date": "", "address": "" }
    condolences_enabled BOOLEAN DEFAULT true,
    condolences_count INTEGER DEFAULT 0,
    condolences TEXT[],
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_obituaries_death ON obituaries(death_date);

-- Tabela miejsc w przewodniku (rozszerzone pola dla CPT 'przewodnik')
CREATE TABLE guide_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    category_id UUID REFERENCES categories(id),
    description TEXT,
    address JSONB,
    coordinates POINT,
    phone VARCHAR(50),
    email VARCHAR(255),
    website VARCHAR(255),
    opening_hours JSONB,
    admission JSONB, -- { "price": "", "free": false, "currency": "PLN" }
    gallery_urls TEXT[],
    features JSONB, -- { "parking": true, "wheelchair_access": true, "wifi": true }
    rating_average DECIMAL(3,2) DEFAULT 0,
    rating_count INTEGER DEFAULT 0,
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_guide_category ON guide_items(category_id);
CREATE INDEX idx_guide_location ON guide_items USING gist(coordinates);

-- Tabela osob (rozszerzone pola dla CPT 'ludzie')
CREATE TABLE people (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    birth_date DATE,
    birth_place VARCHAR(100),
    death_date DATE,
    biography TEXT,
    profession VARCHAR(200),
    company_id UUID REFERENCES companies(id),
    photo_url TEXT,
    gallery_urls TEXT[],
    social_links JSONB,
    website VARCHAR(255),
    achievements TEXT[],
    education TEXT[],
    career TEXT[],
    external_source_url TEXT,
    view_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_people_profession ON people(profession);
CREATE INDEX idx_people_company ON people(company_id);

-- Tabela zalacznikow (media)
CREATE TABLE attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID REFERENCES posts(id) ON DELETE SET NULL,
    filename VARCHAR(255) NOT NULL,
    original_name VARCHAR(255) NOT NULL,
    file_path TEXT NOT NULL,
    file_url TEXT NOT NULL,
    file_size INTEGER NOT NULL, -- w bajtach
    mime_type VARCHAR(100) NOT NULL,
    width INTEGER, -- dla obrazow
    height INTEGER,
    alt_text VARCHAR(255),
    caption TEXT,
    description TEXT,
    is_featured BOOLEAN DEFAULT false,
    sort_order INTEGER DEFAULT 0,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES public.users(id)
);

CREATE INDEX idx_attachments_post ON attachments(post_id);
CREATE INDEX idx_attachments_type ON attachments(mime_type);

-- Tabela SEO meta danych
CREATE TABLE seo_meta (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type VARCHAR(50) NOT NULL, -- 'post', 'category', 'tag', 'page'
    entity_id UUID NOT NULL,
    url_path VARCHAR(500) NOT NULL,
    title VARCHAR(500),
    description TEXT,
    keywords TEXT[],
    og_title VARCHAR(500),
    og_description TEXT,
    og_image_url TEXT,
    og_type VARCHAR(50) DEFAULT 'website',
    twitter_card VARCHAR(20) DEFAULT 'summary_large_image',
    twitter_title VARCHAR(500),
    twitter_description TEXT,
    twitter_image_url TEXT,
    canonical_url TEXT,
    robots_meta VARCHAR(100), -- 'index,follow', 'noindex,nofollow', etc.
    schema_type VARCHAR(50),
    schema_data JSONB,
    hreflang JSONB, -- { "pl": "/pl/post", "en": "/en/post" }
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(entity_type, entity_id),
    UNIQUE(url_path)
);

CREATE INDEX idx_seo_entity ON seo_meta(entity_type, entity_id);
CREATE INDEX idx_seo_path ON seo_meta(url_path);

-- Tabela przekierowan
CREATE TABLE redirects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_path VARCHAR(500) NOT NULL,
    target_path VARCHAR(500) NOT NULL,
    type VARCHAR(20) DEFAULT '301', -- '301', '302', '307'
    is_active BOOLEAN DEFAULT true,
    hit_count INTEGER DEFAULT 0,
    last_hit_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_redirects_source ON redirects(source_path);

-- Tabela pol niestandardowych (ACF-like)
CREATE TABLE custom_fields (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) NOT NULL,
    label VARCHAR(200),
    description TEXT,
    type VARCHAR(50) NOT NULL, -- 'text', 'textarea', 'number', 'select', 'checkbox', 'radio', 'date', 'image', 'gallery', 'repeater'
    config JSONB, -- konfiguracja pola
    default_value JSONB,
    validation_rules JSONB,
    applicable_to VARCHAR(50)[], -- dla jakich CPT
    is_active BOOLEAN DEFAULT true,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tabela wartosci pol niestandardowych
CREATE TABLE custom_field_values (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    field_id UUID NOT NULL REFERENCES custom_fields(id) ON DELETE CASCADE,
    post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    value JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(field_id, post_id)
);

CREATE INDEX idx_cfv_field ON custom_field_values(field_id);
CREATE INDEX idx_cfv_post ON custom_field_values(post_id);

-- Tabela logow audytowych (per tenant)
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id),
    action VARCHAR(100) NOT NULL, -- 'create', 'update', 'delete', 'publish', 'unpublish'
    entity_type VARCHAR(50) NOT NULL, -- 'post', 'category', 'user', 'settings'
    entity_id UUID,
    entity_title VARCHAR(500),
    changes JSONB, -- { "before": {}, "after": {} }
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_created ON audit_logs(created_at);

-- Tabela ustawien domeny
CREATE TABLE domain_settings (
    id VARCHAR(100) PRIMARY KEY,
    category VARCHAR(50) NOT NULL,
    key VARCHAR(100) NOT NULL,
    value JSONB,
    description TEXT,
    is_editable BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_by UUID REFERENCES public.users(id),
    UNIQUE(category, key)
);

-- Funkcja aktualizacji timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Triggery dla aktualizacji timestamp
CREATE TRIGGER update_posts_updated_at BEFORE UPDATE ON posts FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_categories_updated_at BEFORE UPDATE ON categories FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_tags_updated_at BEFORE UPDATE ON tags FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_banners_updated_at BEFORE UPDATE ON banners FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_menus_updated_at BEFORE UPDATE ON menus FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_menu_items_updated_at BEFORE UPDATE ON menu_items FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_widgets_updated_at BEFORE UPDATE ON widgets FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_companies_updated_at BEFORE UPDATE ON companies FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_jobs_updated_at BEFORE UPDATE ON jobs FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_guide_items_updated_at BEFORE UPDATE ON guide_items FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
CREATE TRIGGER update_people_updated_at BEFORE UPDATE ON people FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

```

---

### 3.4 Diagramy ERD (Entity Relationship Diagram)

#### ERD Schematu Public (Centralnego)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 ERD: SCHEMA PUBLIC (DANE CENTRALNE)                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────┐       ┌──────────────┐       ┌──────────┐                │
│  │  users   │◄─────►│  user_roles  │◄─────►│  roles   │                │
│  └──────────┘  1:N  └──────────────┘  N:1  └──────────┘                │
│       │                                            │                    │
│       │ 1:N                                        │ 1:N                │
│       ▼                                            ▼                    │
│  ┌──────────┐                              ┌──────────────┐            │
│  │audit_logs│                              │role_permissions│           │
│  └──────────┘                              └──────────────┘            │
│                                                   │                     │
│                                                   │ N:1                │
│                                                   ▼                     │
│                                            ┌─────────────┐             │
│                                            │ permissions │             │
│                                            └─────────────┘             │
│                                                                         │
│  ┌──────────┐       ┌─────────────┐       ┌──────────────┐             │
│  │ domains  │◄─────►│domain_users │◄─────►│    users     │             │
│  └──────────┘  1:N  └─────────────┘  N:1  └──────────────┘             │
│       │                                                                │
│       │ 1:N                                                            │
│       ├───► sources                                                    │
│       │       (scraping config)                                        │
│       │                                                                │
│       ├───► cron_jobs                                                  │
│       │       (scheduled tasks)                                        │
│       │                                                                │
│       └───► domain_settings                                            │
│               (configuration)                                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### ERD Schematu Tenant (Per Domena)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                ERD: SCHEMA TENANT (PER DOMENA)                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────┐       ┌─────────────────┐       ┌─────────────┐          │
│  │  posts   │◄─────►│ post_categories │◄─────►│ categories  │          │
│  └──────────┘  1:N  └─────────────────┘  N:1  └─────────────┘          │
│       │                                                               │
│       │ 1:N                                                           │
│       ├───► post_tags ◄─────► tags                                    │
│       │                                                               │
│       ├───► comments                                                  │
│       │                                                               │
│       ├───► ratings                                                   │
│       │                                                               │
│       └───► custom_field_values                                       │
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐              │
│  │  companies  │◄───►│   jobs      │◄───►│applications │              │
│  └─────────────┘     └─────────────┘     └─────────────┘              │
│       │                                                               │
│       ├───► company_reviews                                           │
│       └───► company_categories ◄────► categories                      │
│                                                                         │
│  ┌─────────────┐     ┌─────────────┐                                 │
│  │ obituaries  │◄───►│ condolences │                                 │
│  └─────────────┘     └─────────────┘                                 │
│                                                                         │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐                   │
│  │ banners  │       │  menus   │◄─────►│menu_items│                   │
│  └──────────┘       └──────────┘       └──────────┘                   │
│                                                                         │
│  ┌──────────┐       ┌──────────┐                                     │
│  │ widgets  │       │  media   │                                     │
│  └──────────┘       └──────────┘                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### Relacje Między Schematami

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RELACJE MIEDZY SCHEMATAMI                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   SCHEMA: public              SCHEMA: tenant_{domain}                   │
│   ──────────────              ─────────────────────                     │
│                                                                         │
│   ┌──────────┐                ┌──────────┐                             │
│   │  users   │────────────────►│  posts   │                             │
│   │  (id)    │   author_id    │ (author) │                             │
│   └──────────┘                └──────────┘                             │
│                                                                         │
│   ┌──────────┐                ┌──────────┐                             │
│   │  domains │────────────────►│  tables  │                             │
│   │  (id)    │   FK reference │ (domain) │                             │
│   └──────────┘                └──────────┘                             │
│                                                                         │
│   ┌──────────┐                ┌──────────┐                             │
│   │ sources  │────────────────►│  posts   │                             │
│   │  (id)    │   source_id    │ (source) │                             │
│   └──────────┘                └──────────┘                             │
│                                                                         │
│                                                                         │
│   MECHANIZM ROUTINGU:                                                   │
│   ───────────────────                                                   │
│                                                                         │
│   Request: 4torun.pl/api/posts                                          │
│       │                                                                 │
│       ▼                                                                 │
│   Nginx (server_name 4torun.pl)                                         │
│       │                                                                 │
│       ▼                                                                 │
│   Middleware: detectDomain('4torun.pl')                                 │
│       │                                                                 │
│       ▼                                                                 │
│   Prisma: SET search_path = tenant_4torun_pl, public                   │
│       │                                                                 │
│       ▼                                                                 │
│   SQL queries use tenant_4torun_pl.posts                                │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.5 Klucze Obcie (Foreign Keys) - Podsumowanie

| Tabela Źródłowa | Tabela Docelowa | Kolumna FK | Zachowanie ON DELETE |
|-----------------|-----------------|------------|---------------------|
| user_roles | users | user_id | CASCADE |
| user_roles | roles | role_id | CASCADE |
| user_roles | domains | domain_id | SET NULL |
| role_permissions | roles | role_id | CASCADE |
| role_permissions | permissions | permission_id | CASCADE |
| domain_users | domains | domain_id | CASCADE |
| domain_users | users | user_id | CASCADE |
| sources | domains | domain_id | CASCADE |
| posts (tenant) | users (public) | author_id | SET NULL |
| posts (tenant) | sources (public) | source_id | SET NULL |
| comments | posts | post_id | CASCADE |
| comments | users | user_id | SET NULL |
| post_categories | posts | post_id | CASCADE |
| post_categories | categories | category_id | CASCADE |
| media | users | uploaded_by | SET NULL |

---

## 4. SYSTEM UPRAWNIEN (RBAC)

### 4.1 Model RBAC (Role-Based Access Control)

```
┌─────────────────────────────────────────────────────────────────┐
│                    HIERARCHIA ROL                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Level 0: SUPER ADMIN                                           │
│  ├── Dostep do wszystkich domen                                 │
│  ├── Zarzadzanie uzytkownikami systemu                          │
│  ├── Zarzadzanie rolami i uprawnieniami                         │
│  ├── Konfiguracja globalna                                      │
│  ├── Dostep do logow systemowych                                │
│  └── Zarzadzanie szablonami i modulami                          │
│                                                                 │
│  Level 1: ADMIN DOMENY                                          │
│  ├── Pelny dostep do przypisanej domeny                         │
│  ├── Zarzadzanie uzytkownikami domeny                           │
│  ├── Zarzadzanie trescia (CRUD)                                 │
│  ├── Ustawienia domeny (SEO, wyglad)                            │
│  └── Dostep do statystyk i logow domeny                         │
│                                                                 │
│  Level 2: REDAKTOR                                              │
│  ├── Tworzenie i edycja wlasnych wpisow                         │
│  ├── Publikowanie wlasnych wpisow                               │
│  ├── Edycja wszystkich wpisow (jesli nadane)                    │
│  ├── Zarzadzanie kategoriami i tagami                           │
│  └── Moderacja komentarzy                                       │
│                                                                 │
│  Level 3: MODERATOR                                             │
│  ├── Moderacja komentarzy                                       │
│  ├── Moderacja ocen                                             │
│  └── Raportowanie tresci                                        │
│                                                                 │
│  Level 4: UZYTKOWNIK                                            │
│  ├── Dodawanie komentarzy                                       │
│  ├── Ocenianie wpisow                                           │
│  └── Edycja wlasnego profilu                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Szczegolowe Uprawnienia

| Uprawnienie | Resource | Action | Opis |
|-------------|----------|--------|------|
| `domains:manage` | domains | manage | Zarzadzanie domenami (dodawanie, usuwanie) |
| `domains:read` | domains | read | Podglad domen |
| `users:manage:all` | users | manage | Zarzadzanie wszystkimi uzytkownikami |
| `users:manage:domain` | users | manage | Zarzadzanie uzytkownikami wlasnej domeny |
| `users:read` | users | read | Podglad uzytkownikow |
| `roles:manage` | roles | manage | Zarzadzanie rolami |
| `posts:create` | posts | create | Tworzenie wpisow |
| `posts:read:all` | posts | read | Czytanie wszystkich wpisow |
| `posts:read:own` | posts | read | Czytanie tylko wlasnych wpisow |
| `posts:update:all` | posts | update | Edycja wszystkich wpisow |
| `posts:update:own` | posts | update | Edycja tylko wlasnych wpisow |
| `posts:delete:all` | posts | delete | Usuwanie wszystkich wpisow |
| `posts:delete:own` | posts | delete | Usuwanie tylko wlasnych wpisow |
| `posts:publish` | posts | publish | Publikowanie wpisow |
| `categories:manage` | categories | manage | Zarzadzanie kategoriami |
| `tags:manage` | tags | manage | Zarzadzanie tagami |
| `comments:moderate` | comments | moderate | Moderacja komentarzy |
| `comments:delete` | comments | delete | Usuwanie komentarzy |
| `ratings:moderate` | ratings | moderate | Moderacja ocen |
| `banners:manage` | banners | manage | Zarzadzanie bannerami |
| `menus:manage` | menus | manage | Zarzadzanie menu |
| `widgets:manage` | widgets | manage | Zarzadzanie widgetami |
| `settings:manage` | settings | manage | Zarzadzanie ustawieniami |
| `seo:manage` | seo | manage | Zarzadzanie SEO |
| `sources:manage` | sources | manage | Zarzadzanie zrodlami danych |
| `cron:manage` | cron | manage | Zarzadzanie cron jobs |
| `logs:read` | logs | read | Podglad logow |
| `statistics:read` | statistics | read | Podglad statystyk |
| `backup:manage` | backup | manage | Zarzadzanie backupami |
| `templates:manage` | templates | manage | Zarzadzanie szablonami |
| `modules:manage` | modules | manage | Zarzadzanie modulami |
| `mass_operations:execute` | mass_operations | execute | Wykonywanie operacji masowych |

### 4.3 Przypisanie Uprawnien do Rol

```sql
-- SUPER ADMIN - wszystkie uprawnienia we wszystkich domenach
INSERT INTO public.role_permissions (role_id, permission_id, domain_id)
SELECT 
    (SELECT id FROM public.roles WHERE slug = 'super_admin'),
    id,
    NULL
FROM public.permissions;

-- ADMIN DOMENY - wszystkie uprawnienia w przypisanej domenie
-- (domena bedzie wskazana w user_roles)

-- REDAKTOR - typowe uprawnienia do tresci
INSERT INTO public.role_permissions (role_id, permission_id)
SELECT 
    (SELECT id FROM public.roles WHERE slug = 'editor'),
    id
FROM public.permissions
WHERE slug IN (
    'posts:create', 'posts:read:all', 'posts:update:own', 'posts:delete:own', 'posts:publish',
    'categories:read', 'tags:read', 'comments:moderate', 'media:manage'
);

-- MODERATOR - uprawnienia do moderacji
INSERT INTO public.role_permissions (role_id, permission_id)
SELECT 
    (SELECT id FROM public.roles WHERE slug = 'moderator'),
    id
FROM public.permissions
WHERE slug IN (
    'comments:moderate', 'comments:delete', 'ratings:moderate'
);
```

### 4.4 Implementacja Middleware - Express.js / Fastify

#### Middleware Autentykacji JWT

```typescript
// middleware/auth.ts
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';

interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    email: string;
    roles: string[];
    permissions: string[];
    domainId?: string;
  };
}

export const authenticateJWT = (
  req: AuthenticatedRequest,
  res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Brak tokena autentykacji'
    });
  }
  
  const token = authHeader.substring(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({
      error: 'Unauthorized',
      message: 'Nieprawidłowy lub wygasły token'
    });
  }
};
```

#### Middleware Autoryzacji - Sprawdzanie Uprawnień

```typescript
// middleware/authorization.ts
import { Request, Response, NextFunction } from 'express';

interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    permissions: string[];
    roles: string[];
  };
}

/**
 * Middleware sprawdzające czy użytkownik ma konkretne uprawnienie
 */
export const requirePermission = (permission: string) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    // Super admin ma wszystkie uprawnienia
    if (req.user.roles.includes('super_admin')) {
      return next();
    }
    
    if (!req.user.permissions.includes(permission)) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Brak uprawnienia: ${permission}`,
        required: permission,
        userPermissions: req.user.permissions
      });
    }
    
    next();
  };
};

/**
 * Middleware sprawdzające czy użytkownik ma jedno z uprawnień
 */
export const requireAnyPermission = (...permissions: string[]) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (req.user.roles.includes('super_admin')) {
      return next();
    }
    
    const hasAnyPermission = permissions.some(p => 
      req.user!.permissions.includes(p)
    );
    
    if (!hasAnyPermission) {
      return res.status(403).json({
        error: 'Forbidden',
        message: 'Wymagane jedno z uprawnień: ' + permissions.join(', ')
      });
    }
    
    next();
  };
};

/**
 * Middleware sprawdzające własność zasobu (own vs all)
 */
export const requireOwnershipOrPermission = (
  ownPermission: string,
  allPermission: string
) => {
  return async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (req.user.roles.includes('super_admin')) {
      return next();
    }
    
    // Jeśli ma uprawnienie do wszystkich - pozwól
    if (req.user.permissions.includes(allPermission)) {
      return next();
    }
    
    // Jeśli ma uprawnienie do własnych - sprawdź własność
    if (req.user.permissions.includes(ownPermission)) {
      const resourceId = req.params.id;
      const resource = await getResourceById(resourceId); // funkcja pomocnicza
      
      if (resource?.author_id === req.user.id) {
        return next();
      }
      
      return res.status(403).json({
        error: 'Forbidden',
        message: 'Nie jesteś właścicielem tego zasobu'
      });
    }
    
    return res.status(403).json({ error: 'Forbidden' });
  };
};
```

#### Middleware dla Ról

```typescript
// middleware/roles.ts
export const requireRole = (...allowedRoles: string[]) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    const hasRole = req.user.roles.some(role => allowedRoles.includes(role));
    
    if (!hasRole) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Wymagana rola: ${allowedRoles.join(' lub ')}`
      });
    }
    
    next();
  };
};
```

### 4.5 Implementacja Middleware - Next.js API Routes

```typescript
// app/api/middleware.ts (Next.js 13+ App Router)
import { NextRequest, NextResponse } from 'next/server';
import { verify } from 'jsonwebtoken';

export async function middleware(request: NextRequest) {
  // Sprawdź czy ścieżka wymaga autentykacji
  const protectedPaths = ['/api/admin', '/api/protected'];
  const isProtected = protectedPaths.some(path => 
    request.nextUrl.pathname.startsWith(path)
  );
  
  if (!isProtected) {
    return NextResponse.next();
  }
  
  const token = request.headers.get('authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return NextResponse.json(
      { error: 'Unauthorized', message: 'Brak tokena' },
      { status: 401 }
    );
  }
  
  try {
    const decoded = verify(token, process.env.JWT_SECRET!);
    
    // Dodaj user do headers dla użycia w API
    const requestHeaders = new Headers(request.headers);
    requestHeaders.set('x-user-id', decoded.sub as string);
    requestHeaders.set('x-user-roles', JSON.stringify(decoded.roles));
    requestHeaders.set('x-user-permissions', JSON.stringify(decoded.permissions));
    
    return NextResponse.next({
      request: { headers: requestHeaders }
    });
  } catch {
    return NextResponse.json(
      { error: 'Unauthorized', message: 'Nieprawidłowy token' },
      { status: 401 }
    );
  }
}

export const config = {
  matcher: ['/api/:path*']
};
```

### 4.6 Przykłady Użycia w Routing

```typescript
// routes/posts.ts
import { Router } from 'express';
import { authenticateJWT, requirePermission, requireOwnershipOrPermission } from '../middleware';
import { PostController } from '../controllers';

const router = Router();

// Publiczne endpointy - nie wymagają autentykacji
router.get('/posts', PostController.list);
router.get('/posts/:id', PostController.getById);

// Chronione endpointy - wymagają autentykacji
router.post('/posts',
  authenticateJWT,
  requirePermission('posts:create'),
  PostController.create
);

router.put('/posts/:id',
  authenticateJWT,
  requireOwnershipOrPermission('posts:update:own', 'posts:update:all'),
  PostController.update
);

router.delete('/posts/:id',
  authenticateJWT,
  requireOwnershipOrPermission('posts:delete:own', 'posts:delete:all'),
  PostController.remove
);

router.post('/posts/:id/publish',
  authenticateJWT,
  requirePermission('posts:publish'),
  PostController.publish
);

export default router;
```

### 4.7 Komponenty React - Chronione UI

```typescript
// components/auth/ProtectedComponent.tsx
import { useAuth } from '@/hooks/useAuth';

interface ProtectedComponentProps {
  permission?: string;
  role?: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

export const ProtectedComponent: React.FC<ProtectedComponentProps> = ({
  permission,
  role,
  children,
  fallback = null
}) => {
  const { user, hasPermission, hasRole } = useAuth();
  
  if (!user) return fallback;
  
  if (permission && !hasPermission(permission)) {
    return fallback;
  }
  
  if (role && !hasRole(role)) {
    return fallback;
  }
  
  return <>{children}</>;
};

// components/auth/ProtectedRoute.tsx
import { useRouter } from 'next/router';
import { useAuth } from '@/hooks/useAuth';
import { useEffect } from 'react';

export const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, isLoading } = useAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!isLoading && !user) {
      router.push('/login');
    }
  }, [user, isLoading, router]);
  
  if (isLoading) return <div>Ładowanie...</div>;
  if (!user) return null;
  
  return <>{children}</>;
};
```

### 4.8 Hook useAuth

```typescript
// hooks/useAuth.ts
import { useContext, createContext, useState, useEffect } from 'react';

interface AuthContextType {
  user: {
    id: string;
    email: string;
    roles: string[];
    permissions: string[];
  } | null;
  hasPermission: (permission: string) => boolean;
  hasRole: (role: string) => boolean;
  isLoading: boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export const AuthProvider: React.FC = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    // Pobierz user z localStorage / API
    const fetchUser = async () => {
      const token = localStorage.getItem('access_token');
      if (token) {
        const response = await fetch('/api/auth/me', {
          headers: { Authorization: `Bearer ${token}` }
        });
        if (response.ok) {
          const userData = await response.json();
          setUser(userData);
        }
      }
      setIsLoading(false);
    };
    fetchUser();
  }, []);
  
  const hasPermission = (permission: string): boolean => {
    if (!user) return false;
    if (user.roles.includes('super_admin')) return true;
    return user.permissions.includes(permission);
  };
  
  const hasRole = (role: string): boolean => {
    return user?.roles.includes(role) ?? false;
  };
  
  return (
    <AuthContext.Provider value={{ user, hasPermission, hasRole, isLoading }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) throw new Error('useAuth must be used within AuthProvider');
  return context;
};
```

---

## 5. API - SPECYFIKACJA KOMPLETNA

### 5.1 Architektura API

```
┌─────────────────────────────────────────────────────────────────┐
│                    WARSTWY API                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. API GATEWAY                                                 │
│     ├── Routing do odpowiedniego serwisu                      │
│     ├── Rate Limiting                                           │
│     ├── Authentication (JWT)                                    │
│     ├── Authorization (RBAC)                                    │
│     ├── Request/Response Validation                             │
│     ├── Caching                                                 │
│     └── Logging                                                 │
│                                                                 │
│  2. CENTRAL API (serwisy-lokalne-sterowanie.pl/api)             │
│     ├── /v1/admin/* - Zarzadzanie systemem                      │
│     ├── /v1/domains/* - Zarzadzanie domenami                    │
│     ├── /v1/users/* - Zarzadzanie uzytkownikami                 │
│     ├── /v1/sources/* - Zarzadzanie zrodlami                    │
│     ├── /v1/templates/* - Zarzadzanie szablonami                │
│     ├── /v1/modules/* - Zarzadzanie modulami                    │
│     └── /v1/metrics/* - Metryki i statystyki                    │
│                                                                 │
│  3. DOMAIN API (4torun.pl/api, 4bydgoszcz.pl/api)               │
│     ├── /v1/content/* - Publikowane tresci                      │
│     ├── /v1/public/* - Publiczne dane                           │
│     └── /v1/webhook/* - Webhooki dla zewnetrznych systemow     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 API - Centralne (Panel Sterowania)

#### Autentykacja

```yaml
POST /api/v1/auth/login
Summary: Logowanie do systemu
Request:
  body:
    email: string (email)
    password: string
    remember_me: boolean (optional)
Response:
  200:
    access_token: string (JWT)
    refresh_token: string
    expires_in: integer (seconds)
    user:
      id: uuid
      email: string
      first_name: string
      last_name: string
      roles: array
      permissions: array
  401:
    error: "Invalid credentials"

POST /api/v1/auth/refresh
Summary: Odswiezenie tokenu
Request:
  body:
    refresh_token: string
Response:
  200:
    access_token: string
    expires_in: integer

POST /api/v1/auth/logout
Summary: Wylogowanie
Headers:
  Authorization: Bearer {token}
Response:
  200:
    message: "Logged out successfully"

GET /api/v1/auth/me
Summary: Dane zalogowanego uzytkownika
Headers:
  Authorization: Bearer {token}
Response:
  200:
    user:
      id: uuid
      email: string
      first_name: string
      last_name: string
      avatar_url: string
      roles: array
      permissions: array
      domains: array
```

#### Zarzadzanie Domenami

```yaml
GET /api/v1/domains
Summary: Lista wszystkich domen
Headers:
  Authorization: Bearer {token}
Query:
  page: integer (default: 1)
  limit: integer (default: 20, max: 100)
  search: string (optional)
  status: string (optional: active, inactive, maintenance)
  sort: string (default: created_at, options: name, created_at, updated_at)
  order: string (default: desc, options: asc, desc)
Response:
  200:
    data:
      - id: string
        name: string
        slug: string
        city: string
        is_active: boolean
        is_maintenance: boolean
        created_at: datetime
        updated_at: datetime
        posts_count: integer
        users_count: integer
    meta:
      current_page: integer
      last_page: integer
      total: integer
      per_page: integer

POST /api/v1/domains
Summary: Tworzenie nowej domeny
Headers:
  Authorization: Bearer {token}
Request:
  body:
    name: string (required)
    slug: string (required, unique)
    city: string (required)
    region: string
    description: text
    primary_color: string (hex)
    secondary_color: string (hex)
    template_id: uuid
    admin_email: string (email)
Response:
  201:
    domain:
      id: string
      name: string
      slug: string
      schema_name: string
      status: string
      created_at: datetime
  400:
    error: "Validation failed"
    details: object

GET /api/v1/domains/{domainId}
Summary: Szczegoly domeny
Headers:
  Authorization: Bearer {token}
Response:
  200:
    id: string
    name: string
    slug: string
    description: text
    city: string
    region: string
    country: string
    language: string
    timezone: string
    is_active: boolean
    is_maintenance: boolean
    maintenance_message: text
    template_id: uuid
    template: object
    primary_color: string
    secondary_color: string
    logo_url: string
    favicon_url: string
    social_links: object
    contact_info: object
    seo_settings: object
    analytics_settings: object
    features_enabled: object
    posts_count: integer
    users_count: integer
    last_activity_at: datetime
    created_at: datetime
    updated_at: datetime

PUT /api/v1/domains/{domainId}
Summary: Aktualizacja domeny
Headers:
  Authorization: Bearer {token}
Request:
  body:
    name: string
    description: text
    city: string
    region: string
    is_active: boolean
    is_maintenance: boolean
    maintenance_message: text
    primary_color: string
    secondary_color: string
    seo_settings: object
    features_enabled: object
Response:
  200:
    domain: object

DELETE /api/v1/domains/{domainId}
Summary: Usuniecie domeny (soft delete)
Headers:
  Authorization: Bearer {token}
Query:
  force: boolean (default: false, czy usunac fizycznie)
Response:
  200:
    message: "Domain deleted successfully"

GET /api/v1/domains/{domainId}/dashboard
Summary: Dashboard domeny (metryki)
Headers:
  Authorization: Bearer {token}
Response:
  200:
    metrics:
      posts:
        total: integer
        published: integer
        draft: integer
        by_type: object
      users:
        total: integer
        active_today: integer
        new_this_week: integer
      traffic:
        views_today: integer
        views_this_week: integer
        views_this_month: integer
        unique_visitors: integer
      engagement:
        comments_count: integer
        ratings_count: integer
        average_rating: number
    recent_activity:
      - type: string
        description: string
        user: object
        created_at: datetime
    alerts:
      - type: string
        message: string
        severity: string

GET /api/v1/domains/{domainId}/statistics
Summary: Szczegolowe statystyki domeny
Headers:
  Authorization: Bearer {token}
Query:
  from: date
  to: date
  group_by: string (day, week, month)
Response:
  200:
    period:
      from: date
      to: date
    traffic:
      labels: array
      views: array
      unique_visitors: array
    content:
      posts_created: integer
      posts_published: integer
      comments_added: integer
    engagement:
      ratings_given: integer
      average_rating: number
      shares_count: integer
    top_content:
      - post_id: uuid
        title: string
        views: integer
        rating: number

GET /api/v1/domains/{domainId}/content/posts
Summary: Lista wpisow w domenie
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  type: string (wiadomosci, kronika-policyjna, firmy, etc.)
  status: string (draft, published, scheduled, archived)
  category: uuid
  author: uuid
  search: string
  from: date
  to: date
  sort: string
  order: string
Response:
  200:
    data: array of posts
    meta: pagination

POST /api/v1/domains/{domainId}/content/posts
Summary: Tworzenie wpisu w domenie
Headers:
  Authorization: Bearer {token}
Request:
  body:
    post_type: string (required)
    title: string (required)
    slug: string (optional, auto-generate)
    content: text (required)
    excerpt: text
    status: string (draft, published, scheduled)
    published_at: datetime
    categories: array of uuid
    tags: array of string
    featured: boolean
    allow_comments: boolean
    allow_ratings: boolean
    seo_title: string
    seo_description: text
    og_image_url: string
    custom_fields: object
Response:
  201:
    post: object

PUT /api/v1/domains/{domainId}/content/posts/{postId}
Summary: Aktualizacja wpisu
Headers:
  Authorization: Bearer {token}
Request:
  body: (jak wyzej)
Response:
  200:
    post: object

DELETE /api/v1/domains/{domainId}/content/posts/{postId}
Summary: Usuniecie wpisu
Headers:
  Authorization: Bearer {token}
Query:
  force: boolean
Response:
  200:
    message: "Post deleted"

POST /api/v1/domains/{domainId}/content/posts/{postId}/duplicate
Summary: Duplikowanie wpisu
Headers:
  Authorization: Bearer {token}
Response:
  201:
    post: object (nowy wpis)

POST /api/v1/domains/{domainId}/content/posts/{postId}/publish
Summary: Publikowanie wpisu
Headers:
  Authorization: Bearer {token}
Response:
  200:
    post: object

POST /api/v1/domains/{domainId}/content/posts/{postId}/unpublish
Summary: Cofniecie publikacji
Headers:
  Authorization: Bearer {token}
Response:
  200:
    post: object
```

#### Zarzadzanie Uzytkownikami

```yaml
GET /api/v1/users
Summary: Lista uzytkownikow systemu
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  search: string
  role: string
  domain: string
  is_active: boolean
Response:
  200:
    data: array of users
    meta: pagination

POST /api/v1/users
Summary: Tworzenie uzytkownika
Headers:
  Authorization: Bearer {token}
Request:
  body:
    email: string (required)
    password: string (required)
    first_name: string (required)
    last_name: string (required)
    phone: string
    is_active: boolean
    roles: array
      - role_id: uuid
        domain_id: string (optional)
Response:
  201:
    user: object

GET /api/v1/users/{userId}
Summary: Szczegoly uzytkownika
Headers:
  Authorization: Bearer {token}
Response:
  200:
    id: uuid
    email: string
    first_name: string
    last_name: string
    phone: string
    avatar_url: string
    is_active: boolean
    is_super_admin: boolean
    email_verified_at: datetime
    last_login_at: datetime
    login_count: integer
    roles: array
    permissions: array
    domains: array
    created_at: datetime

PUT /api/v1/users/{userId}
Summary: Aktualizacja uzytkownika
Headers:
  Authorization: Bearer {token}
Request:
  body:
    first_name: string
    last_name: string
    phone: string
    is_active: boolean
Response:
  200:
    user: object

DELETE /api/v1/users/{userId}
Summary: Usuniecie uzytkownika
Headers:
  Authorization: Bearer {token}
Response:
  200:
    message: "User deleted"

PUT /api/v1/users/{userId}/roles
Summary: Aktualizacja rol uzytkownika
Headers:
  Authorization: Bearer {token}
Request:
  body:
    roles: array
      - role_id: uuid
        domain_id: string (null dla globalnych)
Response:
  200:
    roles: array

PUT /api/v1/users/{userId}/password
Summary: Zmiana hasla uzytkownika
Headers:
  Authorization: Bearer {token}
Request:
  body:
    password: string (required, min 8 znakow)
    password_confirmation: string
Response:
  200:
    message: "Password updated"

GET /api/v1/users/{userId}/activity
Summary: Aktywnosc uzytkownika
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  from: date
  to: date
Response:
  200:
    data:
      - action: string
        entity_type: string
        entity_title: string
        created_at: datetime
    meta: pagination
```

#### Operacje Masowe

```yaml
POST /api/v1/mass/banners
Summary: Dodanie banneru do wielu domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string (domain IDs)
    banner:
      name: string
      type: string
      position: string
      content: text
      link_url: string
      start_date: datetime
      end_date: datetime
      is_active: boolean
Response:
  200:
    results:
      - domain_id: string
        banner_id: uuid
        status: string (success, error)
        message: string

POST /api/v1/mass/menus
Summary: Dodanie menu do wielu domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string
    menu:
      name: string
      location: string
      items: array
        - title: string
          url: string
          target: string
Response:
  200:
    results: array

POST /api/v1/mass/content
Summary: Dodanie tresci do wielu domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string
    content:
      post_type: string
      title: string
      content: text
      status: string
      categories: array
      tags: array
Response:
  200:
    results: array

POST /api/v1/mass/updates
Summary: Masowa aktualizacja wielu domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string
    updates:
      settings: object (opcjonalnie)
      template_id: uuid (opcjonalnie)
      features_enabled: object (opcjonalnie)
Response:
  200:
    results: array

POST /api/v1/mass/backup
Summary: Backup wielu domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string
    type: string (full, database, files)
Response:
  200:
    backup_id: uuid
    status: string
    estimated_time: integer
```

#### Zrodla Danych (Scraping)

```yaml
GET /api/v1/sources
Summary: Lista zrodel danych
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  type: string
  is_active: boolean
Response:
  200:
    data: array of sources
    meta: pagination

POST /api/v1/sources
Summary: Tworzenie zrodla danych
Headers:
  Authorization: Bearer {token}
Request:
  body:
    name: string (required)
    slug: string (required)
    type: string (required: rss, api, scraper, xml, json)
    url: string (required)
    method: string (default: GET)
    headers: object
    auth_type: string
    auth_config: object
    fetch_config: object
    parser_config: object (required)
    mapping_config: object (required)
    schedule: string (cron expression)
    domains: array of string (domeny uzywajace zrodla)
Response:
  201:
    source: object

GET /api/v1/sources/{sourceId}
Summary: Szczegoly zrodla
Headers:
  Authorization: Bearer {token}
Response:
  200:
    source: object

PUT /api/v1/sources/{sourceId}
Summary: Aktualizacja zrodla
Headers:
  Authorization: Bearer {token}
Request:
  body: (jak przy tworzeniu)
Response:
  200:
    source: object

DELETE /api/v1/sources/{sourceId}
Summary: Usuniecie zrodla
Headers:
  Authorization: Bearer {token}
Response:
  200:
    message: "Source deleted"

POST /api/v1/sources/{sourceId}/run
Summary: Reczne uruchomienie scrapingu
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string (opcjonalnie, jesli puste - wszystkie)
Response:
  202:
    job_id: uuid
    status: "queued"

GET /api/v1/sources/{sourceId}/logs
Summary: Logi scrapingu
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  status: string
Response:
  200:
    data: array of logs
    meta: pagination

GET /api/v1/sources/{sourceId}/schedule
Summary: Harmonogram zrodla
Headers:
  Authorization: Bearer {token}
Response:
  200:
    schedule: string (cron)
    next_runs: array of datetime
```

#### Cron Jobs

```yaml
GET /api/v1/cron
Summary: Lista zadan cron
Headers:
  Authorization: Bearer {token}
Response:
  200:
    data: array of cron jobs

POST /api/v1/cron
Summary: Tworzenie zadania cron
Headers:
  Authorization: Bearer {token}
Request:
  body:
    name: string (required)
    slug: string (required)
    schedule: string (cron expression, required)
    command: string (required)
    arguments: object
    is_active: boolean
    notify_on_failure: boolean
    notify_emails: array of string
Response:
  201:
    cron_job: object

POST /api/v1/cron/{jobId}/run
Summary: Reczne uruchomienie zadania
Headers:
  Authorization: Bearer {token}
Response:
  202:
    job_id: uuid
    status: "running"

GET /api/v1/cron/{jobId}/logs
Summary: Logi wykonania
Headers:
  Authorization: Bearer {token}
Response:
  200:
    data: array of logs
```

#### Szablony

```yaml
GET /api/v1/templates
Summary: Lista szablonow
Headers:
  Authorization: Bearer {token}
Response:
  200:
    data: array of templates

POST /api/v1/templates
Summary: Tworzenie szablonu
Headers:
  Authorization: Bearer {token}
Request:
  body:
    name: string
    slug: string
    description: text
    structure: object
    styles: object
    components: object
Response:
  201:
    template: object

POST /api/v1/templates/{templateId}/apply
Summary: Zastosowanie szablonu do domen
Headers:
  Authorization: Bearer {token}
Request:
  body:
    domains: array of string
    options: object (force, preserve_customizations)
Response:
  200:
    results: array
```

#### Logi Systemowe

```yaml
GET /api/v1/logs
Summary: Logi systemowe
Headers:
  Authorization: Bearer {token}
Query:
  page: integer
  limit: integer
  level: string (debug, info, warning, error, critical)
  category: string
  domain: string
  user: uuid
  action: string
  from: datetime
  to: datetime
  search: string
Response:
  200:
    data: array of logs
    meta: pagination
    summary:
      by_level: object
      by_category: object

GET /api/v1/logs/export
Summary: Eksport logow
Headers:
  Authorization: Bearer {token}
Query:
  format: string (csv, json, xlsx)
  from: datetime
  to: datetime
Response:
  200:
    download_url: string
```

### 5.3 API - Domenowe (Publiczne)

#### Content API

```yaml
GET /api/v1/content/posts
Summary: Lista wpisow (publiczna)
Query:
  page: integer (default: 1)
  limit: integer (default: 20, max: 50)
  type: string (wiadomosci, kronika-policyjna, firmy, ogloszenia, praca, nekrologi, przewodnik, ludzie)
  category: string (slug)
  tag: string (slug)
  search: string
  featured: boolean
  sort: string (created_at, updated_at, published_at, views, rating)
  order: string (desc, asc)
  from: date
  to: date
Response:
  200:
    data:
      - id: uuid
        type: string
        title: string
        slug: string
        excerpt: string
        featured_image: object
        author: object
        categories: array
        tags: array
        published_at: datetime
        view_count: integer
        rating_average: number
        comment_count: integer
    meta: pagination

GET /api/v1/content/posts/{slug}
Summary: Pojedynczy wpis (publiczna)
Response:
  200:
    id: uuid
    type: string
    title: string
    slug: string
    excerpt: string
    content: string
    featured_image: object
    gallery: array
    author: object
    categories: array
    tags: array
    published_at: datetime
    view_count: integer
    rating_average: number
    comment_count: integer
    allow_comments: boolean
    allow_ratings: boolean
    seo_meta: object
    schema_data: object
    related_posts: array

GET /api/v1/content/categories
Summary: Lista kategorii
Query:
  type: string
Response:
  200:
    data: array of categories

GET /api/v1/content/tags
Summary: Lista tagow
Query:
  search: string
  limit: integer
Response:
  200:
    data: array of tags

POST /api/v1/content/posts/{slug}/rate
Summary: Ocenianie wpisu
Request:
  body:
    rating: integer (1-5)
    comment: string (opcjonalnie)
Response:
  201:
    rating: object

POST /api/v1/content/posts/{slug}/comment
Summary: Dodawanie komentarza
Request:
  body:
    content: text
    parent_id: uuid (opcjonalnie, dla odpowiedzi)
    author_name: string (dla niezalogowanych)
    author_email: string (dla niezalogowanych)
Response:
  201:
    comment: object
```

#### Public Data API

```yaml
GET /api/v1/public/menus/{location}
Summary: Menu
Response:
  200:
    location: string
    items: array

GET /api/v1/public/widgets/{area}
Summary: Widgety
Response:
  200:
    area: string
    widgets: array

GET /api/v1/public/banners/{position}
Summary: Bannery
Query:
  page: string (home, post, etc.)
Response:
  200:
    position: string
    banners: array

GET /api/v1/public/settings
Summary: Ustawienia publiczne
Response:
  200:
    site_name: string
    site_description: string
    contact_info: object
    social_links: object
    colors: object
```

### 5.4 Webhooki

```yaml
POST /api/v1/webhooks/scraper
Summary: Webhook dla zakonczonego scrapingu
Headers:
  X-Webhook-Secret: string
Request:
  body:
    source_id: uuid
    domain_id: string
    status: string (success, error)
    items_processed: integer
    items_created: integer
    items_updated: integer
    errors: array
    started_at: datetime
    finished_at: datetime
Response:
  200:
    received: true

POST /api/v1/webhooks/backup
Summary: Webhook dla zakonczonego backupu
Headers:
  X-Webhook-Secret: string
Request:
  body:
    backup_id: uuid
    domain_id: string
    status: string
    file_size: integer
    download_url: string
    created_at: datetime
Response:
  200:
    received: true
```

---



## 6. PANEL ADMINISTRACYJNY - CENTRALNY

### 6.1 Struktura Interfejsu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HEADER                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │ [Logo]  Dashboard  Domeny  Uzytkownicy  Zrodla  Ustawienia        [Q] [Bell] [User ▼] ││
│  └─────────────────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐  ┌─────────────────────────────────────────────────────┐  │
│  │              │  │                                                     │  │
│  │  SIDEBAR     │  │              MAIN CONTENT AREA                      │  │
│  │              │  │                                                     │  │
│  │  Dashboard   │  │  ┌───────────────────────────────────────────────┐  │  │
│  │  ─────────   │  │  │  BREADCRUMBS: Home / Domeny / 4torun.pl       │  │  │
│  │  Domeny      │  │  └───────────────────────────────────────────────┘  │  │
│  │    Lista     │  │                                                     │  │
│  │    Dodaj     │  │  ┌───────────────────────────────────────────────┐  │  │
│  │    Grupy     │  │  │           PAGE TITLE                          │  │  │
│  │  ─────────   │  │  │  [Primary Action Button]                      │  │  │
│  │  Tresci      │  │  └───────────────────────────────────────────────┘  │  │
│  │    Wszystkie │  │                                                     │  │
│  │    Wiadomosci│  │  ┌───────────────────────────────────────────────┐  │  │
│  │    Kronika   │  │  │                                               │  │  │
│  │    Firmy     │  │  │           CONTENT CARDS / TABLES              │  │  │
│  │    Praca     │  │  │                                               │  │  │
│  │    Nekrologi │  │  │                                               │  │  │
│  │  ─────────   │  │  └───────────────────────────────────────────────┘  │  │
│  │  Uzytkownicy │  │                                                     │  │
│  │  ─────────   │  │  ┌───────────────────────────────────────────────┐  │  │
│  │  Zrodla      │  │  │           PAGINATION / FOOTER                   │  │  │
│  │  Cron        │  │  └───────────────────────────────────────────────┘  │  │
│  │  ─────────   │  │                                                     │  │
│  │  Szablony    │  │                                                     │  │
│  │  Moduly      │  │                                                     │  │
│  │  ─────────   │  │                                                     │  │
│  │  Statystyki  │  │                                                     │  │
│  │  Logi        │  │                                                     │  │
│  │  Backup      │  │                                                     │  │
│  │  ─────────   │  │                                                     │  │
│  │  Ustawienia  │  │                                                     │  │
│  └──────────────┘  └─────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Szczegolowa Specyfikacja Podstron

#### 6.2.1 Dashboard Glowny

**URL:** `/`

**Komponenty:**
```typescript
interface DashboardPage {
  // Metryki systemowe (karty)
  systemMetrics: {
    totalDomains: number;
    activeDomains: number;
    totalUsers: number;
    totalPosts: number;
    systemHealth: 'healthy' | 'warning' | 'critical';
    lastBackupAt: Date;
  };
  
  // Wykres ruchu (30 dni)
  trafficChart: {
    labels: string[];
    views: number[];
    uniqueVisitors: number[];
  };
  
  // Top domeny
  topDomains: {
    domainId: string;
    domainName: string;
    viewsToday: number;
    viewsThisMonth: number;
    postsCount: number;
  }[];
  
  // Aktywnosc systemu
  recentActivity: {
    id: string;
    user: { name: string; avatar: string };
    action: string;
    entity: string;
    entityName: string;
    timestamp: Date;
  }[];
  
  // Alerty
  alerts: {
    id: string;
    severity: 'info' | 'warning' | 'error' | 'critical';
    message: string;
    domainId?: string;
    createdAt: Date;
  }[];
  
  // Zadania cron (najblizsze)
  upcomingCronJobs: {
    id: string;
    name: string;
    nextRunAt: Date;
  }[];
}
```

**Funkcjonalnosc:**
- Szybki przeglad stanu systemu
- Nawigacja do najczesciej uzywanych funkcji
- Podglad alertow wymagajacych uwagi
- Wykresy ruchu (Chart.js lub Recharts)

#### 6.2.2 Lista Domen

**URL:** `/admin/domeny`

**Komponenty:**
```typescript
interface DomainsListPage {
  // Filtrowanie i wyszukiwanie
  filters: {
    search: string;
    status: 'all' | 'active' | 'inactive' | 'maintenance';
    city: string;
    sortBy: 'name' | 'created_at' | 'posts_count' | 'last_activity';
    sortOrder: 'asc' | 'desc';
  };
  
  // Tabela domen
  domains: {
    id: string;
    name: string;
    slug: string;
    city: string;
    isActive: boolean;
    isMaintenance: boolean;
    postsCount: number;
    usersCount: number;
    templateName: string;
    lastActivityAt: Date;
    createdAt: Date;
  }[];
  
  // Akcje masowe
  bulkActions: [
    'activate',
    'deactivate',
    'maintenance_on',
    'maintenance_off',
    'delete'
  ];
}
```

**Funkcjonalnosc:**
- Tabela z sortowaniem i filtrowaniem
- Szybkie akcje (edytuj, ustawienia, podglad)
- Akcje masowe na zaznaczonych domenach
- Eksport do CSV/Excel
- Paginacja

#### 6.2.3 Szczegoly Domeny

**URL:** `/admin/domeny/:domainId`

**Zakladki:**
```
┌─────────────────────────────────────────────────────────────────┐
│ [4torun.pl]  [Dashboard] [Tresci] [Uzytkownicy] [Ustawienia]   │
│              [Wyglad] [SEO] [Statystyki] [Logi]                │
└─────────────────────────────────────────────────────────────────┘
```

**Dashboard domeny:**
- Metryki: posty, uzytkownicy, wyswietlenia, komentarze
- Wykresy ruchu (dzienny, tygodniowy, miesieczny)
- Ostatnia aktywnosc w domenie
- Top tresci

**Zakladka Tresci:**
- Podglad wszystkich wpisow w domenie
- Filtrowanie po typie, statusie, dacie
- Szybkie akcje: edytuj, publikuj, usun

**Zakladka Uzytkownicy:**
- Lista uzytkownikow z rolami w tej domenie
- Dodawanie nowych uzytkownikow
- Zarzadzanie uprawnieniami

**Zakladka Ustawienia:**
- Podstawowe: nazwa, opis, miasto
- Kolory: primary, secondary
- Logo i favicon (upload)
- Kontakt i social media
- Jezyk i strefa czasowa

**Zakladka Wyglad:**
- Wybor szablonu
- Konfiguracja menu
- Konfiguracja widgetow
- Bannery

**Zakladka SEO:**
- Globalne ustawienia SEO
- Struktura URL
- Robots.txt
- Sitemap

**Zakladka Statystyki:**
- Szczegolowe statystyki ruchu
- Zrodla ruchu
- Najpopularniejsze tresci
- Dane demograficzne

#### 6.2.4 Masowe Operacje

**URL:** `/admin/mass-operations`

**Operacje:**

1. **Masowe Dodawanie Bannerow:**
   - Wybor domen (wszystkie / wybrane / grupy)
   - Formularz banneru
   - Podglad przed zastosowaniem
   - Raport wykonania

2. **Masowe Dodawanie Menu:**
   - Wybor domen
   - Budowniczy menu (drag & drop)
   - Lokalizacja menu (header, footer, etc.)

3. **Masowe Dodawanie Tresci:**
   - Wybor domen
   - Wybor typu tresci
   - Formularz tresci
   - Opcje publikacji (teraz, zaplanuj, szkic)

4. **Masowe Aktualizacje:**
   - Wybor domen
   - Wybor ustawien do aktualizacji
   - Podglad zmian
   - Potwierdzenie

5. **Masowe Backup:**
   - Wybor domen
   - Typ backupu (pelny, baza danych, pliki)
   - Harmonogram (opcjonalnie)

#### 6.2.5 Zarzadzanie Zrodlami Danych

**URL:** `/admin/zrodla`

**Lista Zrodel:**
- Tabela ze wszystkimi zrodlami
- Status (aktywne / nieaktywne)
- Ostatnie uruchomienie
- Liczba pobranych elementow
- Szybkie akcje (uruchom, edytuj, logi)

**Formularz Zrodla (Tworzenie/Edycja):**
```typescript
interface SourceForm {
  // Podstawowe
  name: string;
  type: 'rss' | 'api' | 'scraper' | 'xml' | 'json';
  
  // Polaczenie
  url: string;
  method: 'GET' | 'POST';
  headers: Record<string, string>;
  
  // Autentykacja
  authType: 'none' | 'basic' | 'bearer' | 'api_key';
  authConfig: {
    username?: string;
    password?: string;
    token?: string;
    apiKey?: string;
    apiKeyHeader?: string;
  };
  
  // Parser (zalezy od typu)
  parserConfig: {
    // Dla RSS
    itemSelector?: string;
    titleSelector?: string;
    contentSelector?: string;
    dateSelector?: string;
    imageSelector?: string;
    
    // Dla JSON/API
    rootPath?: string;
    mapping?: Record<string, string>;
  };
  
  // Mapowanie pol
  mappingConfig: {
    title: string; // np. "{{title}}" lub "{{item.title}}"
    content: string;
    excerpt: string;
    featuredImage: string;
    publishedAt: string;
    author: string;
    categories: string;
    tags: string;
  };
  
  // Harmonogram
  schedule: string; // cron expression
  
  // Domeny
  domains: string[]; // ktore domeny uzywaja tego zrodla
}
```

**Testowanie Zrodla:**
- Przycisk "Testuj polaczenie"
- Przycisk "Pobierz probke"
- Podglad sparsowanych danych
- Wizualny mapping pol

**Logi Zrodla:**
- Tabela wykonan
- Status (success / error / running)
- Liczba elementow
- Czas wykonania
- Szczegoly bledow

#### 6.2.6 Zarzadzanie Cron Jobs

**URL:** `/admin/cron`

**Lista Zadan:**
- Nazwa, harmonogram, status
- Ostatnie wykonanie, nastepne wykonanie
- Liczba sukcesow / porazek
- Szybkie akcje (uruchom, edytuj, wylacz)

**Formularz Zadania Cron:**
```typescript
interface CronJobForm {
  name: string;
  description: string;
  
  // Harmonogram
  schedule: string; // cron expression
  // Lub przy uzyciu buildera:
  scheduleBuilder: {
    minute: string;
    hour: string;
    dayOfMonth: string;
    month: string;
    dayOfWeek: string;
  };
  
  // Komenda
  commandType: 'scraper' | 'backup' | 'cleanup' | 'custom';
  command: string;
  arguments: Record<string, any>;
  
  // Opcje
  timeout: number; // sekundy
  retryCount: number;
  notifyOnFailure: boolean;
  notifyEmails: string[];
}
```

**Kreator Cron (Wizualny):**
- Wybor czestotliwosci (co minute, co godzine, codziennie, etc.)
- Wybor konkretnych godzin/dni
- Podglad nastepnych 5 wykonan
- Walidacja expression

#### 6.2.7 Zarzadzanie Uzytkownikami

**URL:** `/admin/uzytkownicy`

**Lista Uzytkownikow:**
- Tabela z filtrami i sortowaniem
- Kolumny: imie, email, role, domeny, status, ostatnie logowanie
- Akcje: edytuj, zresetuj haslo, zablokuj, usun

**Formularz Uzytkownika:**
```typescript
interface UserForm {
  // Podstawowe
  email: string;
  firstName: string;
  lastName: string;
  phone: string;
  avatar: File;
  
  // Status
  isActive: boolean;
  emailVerified: boolean;
  
  // Role i uprawnienia
  roles: {
    roleId: string;
    domainId: string | null; // null = globalnie
  }[];
  
  // Dostep do domen
  domainAccess: string[]; // jesli nie super admin
}
```

**Drzewo Uprawnien:**
- Wizualne przedstawienie uprawnien
- Grupowane po resource (posts, users, settings)
- Checkboxy dla kazdego uprawnienia
- Podglad efektywnych uprawnien

#### 6.2.8 Logi Systemowe

**URL:** `/admin/logi`

**Filtrowanie:**
- Poziom (debug, info, warning, error, critical)
- Kategoria (auth, database, scraper, api)
- Domena
- Uzytkownik
- Zakres dat
- Wyszukiwanie tekstowe

**Tabela Logow:**
- Timestamp (z dokladnoscia do ms)
- Poziom (kolorowanie)
- Kategoria
- Domena
- Uzytkownik
- Akcja
- Wiadomosc (skrocona)
- Szczegoly (rozwijane)

**Eksport:**
- CSV, JSON, Excel
- Wybor zakresu
- Wybor pol

**Logi w Czasie Rzeczywistym:**
- Auto-odswiezanie (WebSocket)
- Podglad "live" logow
- Filtrowanie w locie

#### 6.2.9 Ustawienia Systemowe

**URL:** `/admin/ustawienia`

**Sekcje:**

1. **Ustawienia Ogolne:**
   - Nazwa systemu
   - Logo systemu
   - Jezyk domyslny
   - Strefa czasowa

2. **Email:**
   - SMTP settings
   - Szablony emaili
   - Test wysylki

3. **Bezpieczenstwo:**
   - Polityka hasel
   - 2FA (opcjonalnie)
   - Ograniczenia logowan
   - Whitelist IP

4. **Integracje:**
   - Google Analytics
   - Facebook Pixel
   - OpenWeatherMap
   - GIOS
   - Inne API

5. **Backup:**
   - Harmonogram backupu
   - Miejsce przechowywania
   - Retencja

6. **API:**
   - Rate limiting
   - CORS settings
   - Webhook secrets

---

## 7. PANEL ADMINISTRACYJNY - PER SERWIS

### 7.1 Struktura

Kazdy serwis regionalny ma wlasny panel administracyjny dostepny pod:
`https://4torun.pl/admin`

### 7.2 Dashboard Serwisu

**Komponenty:**
- Szybkie statystyki (dzisiejsze wyswietlenia, nowe wpisy, komentarze)
- Wykres ruchu (7 dni)
- Ostatnie wpisy (do zaakceptowania, opublikowane)
- Ostatnie komentarze (do moderacji)
- Popularne tresci
- Alerty (np. "5 wpisow czeka na publikacje")

### 7.3 Zarzadzanie Trescia

**Lista Wpisow:**
- Tabela z filtrami (typ, status, kategoria, autor, data)
- Szybkie akcje (podglad, edytuj, usun, duplikuj)
- Statusy: szkic, zaplanowany, opublikowany, archiwowany
- Informacje o autorze i dacie
- Liczba wyswietlen i ocen

**Edytor Wpisow:**
```typescript
interface PostEditor {
  // Lewa kolumna (glowna)
  title: string;
  slug: string; // auto-generate z mozliwoscia edycji
  content: RichTextEditor; // TipTap / Slate.js
  excerpt: TextArea; // auto-generate z content
  featuredImage: ImageUpload;
  gallery: ImageUpload[];
  
  // Prawa kolumna (sidebar)
  publishPanel: {
    status: 'draft' | 'published' | 'scheduled';
    publishedAt: DateTimePicker;
    visibility: 'public' | 'private' | 'password';
    password: string; // jesli visibility=password
    author: Select; // lista uzytkownikow
    actions: ['save_draft', 'preview', 'publish', 'schedule'];
  };
  
  categoriesPanel: {
    categories: TreeSelect;
    tags: TagInput;
    addNewCategory: Button;
  };
  
  featuredPanel: {
    isFeatured: Checkbox;
    isSticky: Checkbox;
    allowComments: Checkbox;
    allowRatings: Checkbox;
  };
  
  seoPanel: {
    seoTitle: string; // z licznikiem znakow
    seoDescription: TextArea; // z licznikiem
    keywords: TagInput;
    ogImage: ImageUpload;
    schemaType: Select;
    schemaPreview: JSONPreview;
  };
  
  customFieldsPanel: {
    fields: DynamicForm; // zalezy od post_type
  };
}
```

**Funkcje Edytora:**
- Autosave (co 30 sekund)
- Podglad na zywo
- Historia wersji (porownywanie)
- Media library (przegladarka zalacznikow)
- Linkowanie wewnetrzne (wyszukiwarka wpisow)
- SEO analysis (czytelnosc, slowa kluczowe, meta)

### 7.4 Zarzadzanie Kategoriami i Tagami

**Kategorie:**
- Drzewo kategorii (drag & drop do zmiany hierarchii)
- Szybkie dodawanie
- Edycja: nazwa, slug, opis, kolor, ikona, obrazek
- Liczba wpisow w kategorii
- SEO ustawienia per kategoria

**Tagi:**
- Chmura tagow
- Fuzja tagow (laczenie duplikatow)
- Masowe operacje

### 7.5 Zarzadzanie Uzytkownikami Serwisu

**Lista:**
- Tabela uzytkownikow domeny
- Role w tej domenie
- Aktywnosc
- Akcje: edytuj, zmien role, zablokuj

**Formularz:**
- Dane osobowe
- Role w domenie
- Uprawnienia (indywidualne)

### 7.6 Zarzadzanie Bannerami

**Lista Bannerow:**
- Podglad wizualny
- Pozycja na stronie
- Okres wyswietlania
- Statystyki (wyswietlenia, klikniecia, CTR)
- Status (aktywny / nieaktywny)

**Formularz:**
- Nazwa
- Typ (obrazek, HTML, skrypt)
- Pozycja (header, sidebar, content, footer, popup)
- Zawartosc (upload / edytor)
- Link (URL, target)
- Okres wyswietlania
- Reguly wyswietlania (strony, typy wpisow, urzadzenia)

### 7.7 Zarzadzanie Menu

**Budowniczy Menu (Drag & Drop):**
```typescript
interface MenuBuilder {
  // Struktura drzewiasta
  items: MenuItem[];
  
  // Panel dodawania
  addPanel: {
    type: 'link' | 'page' | 'category' | 'custom';
    link: {
      title: string;
      url: string;
      target: '_self' | '_blank';
      icon: IconPicker;
    };
  };
  
  // Opcje
  locations: string[]; // header, footer, sidebar
  preview: boolean; // podglad na zywo
}
```

### 7.8 Zarzadzanie Widgetami

**Lista Widgetow:**
- Dostepne pozycje (sidebar_main, sidebar_secondary, footer_1-4)
- Lista widgetow w kazdej pozycji
- Drag & drop do zmiany kolejnosci

**Typy Widgetow:**
- Ostatnie wpisy
- Popularne wpisy
- Kategorie
- Tagi
- Autorzy
- Newsletter (formularz)
- Social media
- Pogoda
- Jakosc powietrza
- Wyszukiwarka
- Reklama
- Wlasny HTML

**Formularz Widgetu:**
- Wybor typu
- Konfiguracja specyficzna (np. dla "Ostatnie wpisy": typ, liczba, pokazuj obrazek)
- Reguly wyswietlania

### 7.9 Ustawienia Serwisu

**Podstawowe:**
- Nazwa serwisu
- Slogan / opis
- Logo, favicon
- Kolory (primary, secondary)
- Jezyk i strefa czasowa

**Kontakt:**
- Adres
- Telefon
- Email
- Godziny otwarcia
- Mapa (wspolrzedne)

**Social Media:**
- Facebook
- Twitter/X
- Instagram
- LinkedIn
- YouTube
- Inne

**SEO:**
- Tytul domyslny
- Opis domyslny
- Struktura permalinkow
- Robots.txt (edytor)
- Schema.org (typ domyslny)

**Integracje:**
- Google Analytics ID
- Facebook Pixel ID
- OpenWeather API
- GIOS (stacja pomiarowa)
- Inne

---



## 8. SYSTEM SCRAPINGU I CRON JOBS

### 8.1 Architektura Scrapingu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SYSTEM SCRAPINGU                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │   SCHEDULER  │────▶│    QUEUE     │────▶│   WORKER     │                │
│  │   (Cron)     │     │  (RabbitMQ)  │     │   (Python)   │                │
│  └──────────────┘     └──────────────┘     └──────┬───────┘                │
│                                                   │                         │
│                          ┌────────────────────────┼──────────────────┐      │
│                          ▼                        ▼                  ▼      │
│                   ┌──────────────┐        ┌──────────────┐   ┌──────────┐   │
│                   │   Fetcher    │        │   Parser     │   │  Saver   │   │
│                   │  (HTTP)      │        │ (BeautifulSoup│   │(Database)│   │
│                   └──────────────┘        │  / JSON)     │   └──────────┘   │
│                                           └──────────────┘                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Komponenty Scrapingu

#### Scheduler (Node.js / Cron)
```typescript
interface ScraperScheduler {
  // Planowanie zadan
  scheduleJob(sourceId: string, cronExpression: string): void;
  
  // Wywolanie reczne
  triggerJob(sourceId: string, domainIds?: string[]): Promise<JobId>;
  
  // Monitorowanie
  getJobStatus(jobId: string): JobStatus;
  getQueueStatus(): QueueStatus;
}
```

#### Worker (Python)
```python
# scraper_worker.py
import asyncio
import aiohttp
import json
from bs4 import BeautifulSoup
from datetime import datetime
from typing import Dict, List, Optional
import pika
import psycopg2
from dataclasses import dataclass

@dataclass
class ScrapedItem:
    title: str
    content: str
    excerpt: str
    published_at: Optional[datetime]
    author: Optional[str]
    featured_image: Optional[str]
    categories: List[str]
    tags: List[str]
    external_url: str
    external_id: Optional[str]
    source_data: Dict

class ScraperWorker:
    def __init__(self, config: Dict):
        self.config = config
        self.db_connection = psycopg2.connect(config['database_url'])
        self.queue_connection = pika.BlockingConnection(
            pika.URLParameters(config['rabbitmq_url'])
        )
        
    async def fetch_data(self, source: Dict) -> str:
        """Pobieranie danych ze zrodla"""
        headers = source.get('headers', {})
        headers['User-Agent'] = 'RegionalneSerwisyBot/1.0'
        
        # Autentykacja
        auth_config = source.get('auth_config', {})
        if source.get('auth_type') == 'bearer':
            headers['Authorization'] = f"Bearer {auth_config['token']}"
        elif source.get('auth_type') == 'api_key':
            headers[auth_config['api_key_header']] = auth_config['api_key']
        
        timeout = source.get('fetch_config', {}).get('timeout', 30)
        
        async with aiohttp.ClientSession() as session:
            async with session.request(
                method=source.get('method', 'GET'),
                url=source['url'],
                headers=headers,
                timeout=aiohttp.ClientTimeout(total=timeout)
            ) as response:
                response.raise_for_status()
                return await response.text()
    
    def parse_data(self, raw_data: str, source: Dict) -> List[ScrapedItem]:
        """Parsowanie danych zaleznie od typu zrodla"""
        parser_type = source['type']
        parser_config = source['parser_config']
        
        if parser_type == 'rss':
            return self._parse_rss(raw_data, parser_config)
        elif parser_type == 'json':
            return self._parse_json(raw_data, parser_config)
        elif parser_type == 'html':
            return self._parse_html(raw_data, parser_config)
        elif parser_type == 'api':
            return self._parse_api(raw_data, parser_config)
        else:
            raise ValueError(f"Unknown parser type: {parser_type}")
    
    def _parse_html(self, html: str, config: Dict) -> List[ScrapedItem]:
        """Parsowanie HTML (np. Policja, Urzad Miasta)"""
        soup = BeautifulSoup(html, 'html.parser')
        items = []
        
        # Selektor glowny
        item_selector = config.get('item_selector', 'article')
        elements = soup.select(item_selector)
        
        for element in elements:
            try:
                item = ScrapedItem(
                    title=self._extract_text(element, config.get('title_selector')),
                    content=self._extract_html(element, config.get('content_selector')),
                    excerpt=self._extract_text(element, config.get('excerpt_selector')),
                    published_at=self._extract_date(element, config.get('date_selector')),
                    author=self._extract_text(element, config.get('author_selector')),
                    featured_image=self._extract_image(element, config.get('image_selector')),
                    categories=self._extract_list(element, config.get('categories_selector')),
                    tags=self._extract_list(element, config.get('tags_selector')),
                    external_url=self._extract_url(element, config.get('url_selector')),
                    external_id=self._extract_attr(element, config.get('id_selector')),
                    source_data={'raw_html': str(element)}
                )
                items.append(item)
            except Exception as e:
                logger.error(f"Error parsing item: {e}")
                continue
        
        return items
    
    def _extract_text(self, element, selector: str) -> str:
        """Wyciaganie tekstu za pomoca selektora CSS"""
        if not selector:
            return ''
        found = element.select_one(selector)
        return found.get_text(strip=True) if found else ''
    
    def _extract_html(self, element, selector: str) -> str:
        """Wyciaganie HTML za pomoca selektora CSS"""
        if not selector:
            return ''
        found = element.select_one(selector)
        return str(found) if found else ''
    
    def _extract_image(self, element, selector: str) -> Optional[str]:
        """Wyciaganie URL obrazka"""
        if not selector:
            return None
        img = element.select_one(selector)
        if img:
            return img.get('src') or img.get('data-src')
        return None
    
    def _extract_url(self, element, selector: str) -> str:
        """Wyciaganie URL (z atrybutu href lub data)"""
        if not selector:
            return ''
        link = element.select_one(selector)
        if link:
            url = link.get('href')
            # Obsluga base64 encoded URLs (jak w 4torun.pl)
            if not url and link.get('data'):
                import base64
                import urllib.parse
                encoded = link.get('data')
                decoded = base64.b64decode(encoded).decode('utf-8')
                url = urllib.parse.unquote(decoded)
            return url
        return ''
    
    def save_items(self, items: List[ScrapedItem], source_id: str, domain_id: str, mapping_config: Dict):
        """Zapisywanie sparsowanych elementow do bazy"""
        cursor = self.db_connection.cursor()
        
        saved_count = 0
        updated_count = 0
        
        for item in items:
            try:
                # Sprawdzenie czy wpis juz istnieje (po external_id lub external_url)
                cursor.execute("""
                    SELECT id FROM posts 
                    WHERE domain_id = %s AND (external_id = %s OR source_url = %s)
                """, (domain_id, item.external_id, item.external_url))
                
                existing = cursor.fetchone()
                
                # Mapowanie pol
                post_data = self._map_fields(item, mapping_config)
                post_data['source_id'] = source_id
                post_data['external_url'] = item.external_url
                post_data['external_id'] = item.external_id
                post_data['source_data'] = json.dumps(item.source_data)
                
                if existing:
                    # Aktualizacja istniejacego wpisu
                    cursor.execute("""
                        UPDATE posts SET
                            title = %s,
                            content = %s,
                            excerpt = %s,
                            updated_at = NOW(),
                            source_data = %s
                        WHERE id = %s
                    """, (
                        post_data['title'],
                        post_data['content'],
                        post_data['excerpt'],
                        post_data['source_data'],
                        existing[0]
                    ))
                    updated_count += 1
                else:
                    # Tworzenie nowego wpisu
                    cursor.execute("""
                        INSERT INTO posts (
                            domain_id, post_type, title, slug, content, excerpt,
                            status, source_id, source_url, external_id, source_data,
                            created_at, updated_at
                        ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, NOW(), NOW())
                        RETURNING id
                    """, (
                        domain_id,
                        post_data.get('post_type', 'wiadomosci'),
                        post_data['title'],
                        self._generate_slug(post_data['title']),
                        post_data['content'],
                        post_data['excerpt'],
                        post_data.get('status', 'published'),
                        source_id,
                        item.external_url,
                        item.external_id,
                        post_data['source_data']
                    ))
                    saved_count += 1
                
                self.db_connection.commit()
                
            except Exception as e:
                logger.error(f"Error saving item: {e}")
                self.db_connection.rollback()
                continue
        
        return {'saved': saved_count, 'updated': updated_count}
    
    def _map_fields(self, item: ScrapedItem, mapping_config: Dict) -> Dict:
        """Mapowanie pol zrodla na pola bazy danych"""
        result = {}
        
        for db_field, source_pattern in mapping_config.items():
            # Proste mapowanie: "{{title}}" -> item.title
            if source_pattern.startswith('{{') and source_pattern.endswith('}}'):
                attr_name = source_pattern[2:-2]
                result[db_field] = getattr(item, attr_name, '')
            else:
                # Statyczna wartosc
                result[db_field] = source_pattern
        
        return result
    
    def _generate_slug(self, title: str) -> str:
        """Generowanie slug z tytulu"""
        import re
        slug = re.sub(r'[^\w\s-]', '', title.lower())
        slug = re.sub(r'[-\s]+', '-', slug)
        return slug[:200]
    
    async def process_job(self, job_data: Dict):
        """Glowna metoda przetwarzania zadania"""
        source_id = job_data['source_id']
        domain_ids = job_data.get('domain_ids', [])
        
        # Pobieranie konfiguracji zrodla
        source = self._get_source_config(source_id)
        
        # Pobieranie danych
        raw_data = await self.fetch_data(source)
        
        # Parsowanie
        items = self.parse_data(raw_data, source)
        
        # Zapisywanie dla kazdej domeny
        results = []
        for domain_id in domain_ids:
            result = self.save_items(
                items, 
                source_id, 
                domain_id, 
                source['mapping_config']
            )
            results.append({
                'domain_id': domain_id,
                **result
            })
        
        return {
            'source_id': source_id,
            'items_found': len(items),
            'domains_processed': results
        }

# Uruchamianie workera
if __name__ == '__main__':
    worker = ScraperWorker(config={
        'database_url': 'postgresql://...',
        'rabbitmq_url': 'amqp://...'
    })
    worker.start_consuming()
```

### 8.3 Konfiguracja Zrodel - Przyklady

#### Zrodlo: Policja Torun
```json
{
  "name": "Policja Torun - Wiadomosci",
  "slug": "policja-torun",
  "type": "html",
  "url": "https://torun.policja.gov.pl/kb3/informacje/wiadomosci/",
  "method": "GET",
  "headers": {
    "Accept": "text/html",
    "Accept-Language": "pl-PL,pl;q=0.9"
  },
  "fetch_config": {
    "timeout": 30,
    "retries": 3,
    "delay_between_requests": 1
  },
  "parser_config": {
    "item_selector": "article.news-item, .news-list article",
    "title_selector": "h2 a, .news-title",
    "content_selector": ".news-content, .article-content",
    "excerpt_selector": ".news-lead, .article-lead",
    "date_selector": ".news-date, .article-date, time",
    "image_selector": ".news-image img, .article-image img",
    "url_selector": "h2 a, .news-title a",
    "id_selector": "article[data-id]"
  },
  "mapping_config": {
    "post_type": "kronika-policyjna",
    "title": "{{title}}",
    "content": "{{content}}",
    "excerpt": "{{excerpt}}",
    "status": "published"
  },
  "schedule": "0 */6 * * *",
  "domains": ["4torun.pl"],
  "is_active": true
}
```

#### Zrodlo: Urzad Miasta Torun
```json
{
  "name": "Urzad Miasta Torun - Aktualnosci",
  "slug": "torun-aktualnosci",
  "type": "html",
  "url": "https://www.torun.pl/pl/aktualnosci",
  "parser_config": {
    "item_selector": ".news-item, article.node--type-article",
    "title_selector": ".news-title a, h2 a",
    "content_selector": ".field--name-body",
    "excerpt_selector": ".field--name-field-lead",
    "date_selector": ".news-date, time",
    "image_selector": ".field--name-field-image img",
    "url_selector": ".news-title a"
  },
  "mapping_config": {
    "post_type": "wiadomosci",
    "title": "{{title}}",
    "content": "{{content}}",
    "excerpt": "{{excerpt}}",
    "status": "published"
  },
  "schedule": "0 */4 * * *",
  "domains": ["4torun.pl"],
  "is_active": true
}
```

#### Zrodlo: Wikipedia (API)
```json
{
  "name": "Wikipedia - Przewodnik",
  "slug": "wikipedia-guide",
  "type": "api",
  "url": "https://pl.wikipedia.org/api/rest_v1/page/summary/{title}",
  "method": "GET",
  "headers": {
    "Accept": "application/json",
    "Api-User-Agent": "RegionalneSerwisy/1.0"
  },
  "parser_config": {
    "root_path": "",
    "mapping": {
      "title": "title",
      "content": "extract",
      "image": "thumbnail.source"
    }
  },
  "mapping_config": {
    "post_type": "przewodnik",
    "title": "{{title}}",
    "content": "{{content}}",
    "featured_image": "{{image}}"
  },
  "schedule": "0 0 * * 0",
  "domains": ["4torun.pl", "4bydgoszcz.pl"],
  "is_active": true
}
```

### 8.4 System Cron Jobs

#### Typowe Zadania Cron

| Zadanie | Harmonogram | Opis |
|---------|-------------|------|
| `scraper-all` | Co 6h | Uruchom wszystkie aktywne scrapery |
| `scraper-policja` | Co 6h | Scraping policja.gov.pl |
| `scraper-miasto` | Co 4h | Scraping urzedu miasta |
| `backup-full` | Codziennie 2:00 | Pelny backup wszystkich domen |
| `backup-incremental` | Co 4h | Backup przyrostowy |
| `cleanup-logs` | Co tydzien | Czyszczenie starych logow |
| `cleanup-cache` | Codziennie 3:00 | Czyszczenie cache |
| `generate-sitemap` | Codziennie 4:00 | Generowanie sitemap.xml |
| `update-search-index` | Co godzine | Aktualizacja indeksu Elasticsearch |
| `send-newsletter` | Co tydzien | Wysylka newsletterow |
| `check-domains-health` | Co godzine | Sprawdzenie dostepnosci domen |
| `sync-analytics` | Co 6h | Synchronizacja danych analytics |

#### Konfiguracja Cron (crontab)
```bash
# Systemowe zadania cron
# /etc/cron.d/regionalne-serwisy

# Scraping
0 */6 * * * root /usr/bin/python3 /home/host988956/workers/scraper.py --source=policja
0 */4 * * * root /usr/bin/python3 /home/host988956/workers/scraper.py --source=miasto

# Backup
0 2 * * * root /home/host988956/scripts/backup.sh --type=full
0 */4 * * * root /home/host988956/scripts/backup.sh --type=incremental

# Cleanup
0 3 * * * root /usr/bin/php /home/host988956/scripts/cleanup.php
0 4 * * 0 root /usr/bin/php /home/host988956/scripts/cleanup-logs.php --days=30

# Sitemap
0 4 * * * root /usr/bin/php /home/host988956/scripts/generate-sitemap.php

# Search index
0 * * * * root /usr/bin/php /home/host988956/scripts/update-search-index.php

# Monitoring
0 * * * * root /usr/bin/php /home/host988956/scripts/health-check.php
```

### 8.5 Obsługa Błędów i Odporność (Resilience)

#### Retry Logic z Exponential Backoff

```python
import asyncio
import random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1, max_delay=60):
    """
    Dekorator implementujący retry z exponential backoff.
    
    Args:
        max_retries: Maksymalna liczba prób
        base_delay: Podstawowe opóźnienie (sekundy)
        max_delay: Maksymalne opóźnienie (sekundy)
    """
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_retries + 1):
                try:
                    return await func(*args, **kwargs)
                except (aiohttp.ClientError, asyncio.TimeoutError) as e:
                    last_exception = e
                    if attempt == max_retries:
                        raise last_exception
                    
                    # Exponential backoff: 1s, 2s, 4s...
                    delay = min(base_delay * (2 ** attempt), max_delay)
                    # Dodaj jitter (0-1s) aby uniknąć thundering herd
                    delay += random.uniform(0, 1)
                    
                    logger.warning(
                        f"Attempt {attempt + 1}/{max_retries + 1} failed: {str(e)}. "
                        f"Retrying in {delay:.1f}s..."
                    )
                    await asyncio.sleep(delay)
            
            raise last_exception
        return wrapper
    return decorator

# Użycie w workerze
class ScraperWorker:
    @retry_with_backoff(max_retries=3, base_delay=1)
    async def fetch_data(self, source: Dict) -> str:
        """Pobieranie danych ze źródła z retry logic."""
        async with aiohttp.ClientSession() as session:
            async with session.get(
                source['url'],
                headers=self._get_headers(source),
                timeout=aiohttp.ClientTimeout(total=30)
            ) as response:
                response.raise_for_status()
                return await response.text()
```

#### Circuit Breaker Pattern

```python
from enum import Enum
import time

class CircuitState(Enum):
    CLOSED = "closed"      # Normalna praca - zapytania przechodzą
    OPEN = "open"          # Awaria - zapytania odrzucane od razu
    HALF_OPEN = "half_open"  # Test czy usługa działa

class CircuitBreaker:
    """
    Circuit Breaker - wzór projektowy zapobiegający kaskadowym awariom.
    """
    def __init__(
        self,
        failure_threshold=5,      # Ile błędów otwiera obwód
        recovery_timeout=60,      # Po ilu sekundach próba recovery
        half_open_max_calls=3     # Ile prób w stanie half-open
    ):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.half_open_max_calls = half_open_max_calls
        
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = None
        self.half_open_calls = 0
        self._lock = asyncio.Lock()
    
    async def call(self, func, *args, **kwargs):
        async with self._lock:
            if self.state == CircuitState.OPEN:
                if time.time() - self.last_failure_time >= self.recovery_timeout:
                    self.state = CircuitState.HALF_OPEN
                    self.half_open_calls = 0
                    logger.info("Circuit breaker entering HALF_OPEN state")
                else:
                    raise CircuitBreakerOpen("Circuit breaker is OPEN")
            
            if self.state == CircuitState.HALF_OPEN:
                if self.half_open_calls >= self.half_open_max_calls:
                    raise CircuitBreakerOpen("Circuit breaker HALF_OPEN limit reached")
                self.half_open_calls += 1
        
        try:
            result = await func(*args, **kwargs)
            await self._on_success()
            return result
        except Exception as e:
            await self._on_failure()
            raise e
    
    async def _on_success(self):
        async with self._lock:
            if self.state == CircuitState.HALF_OPEN:
                self.success_count += 1
                if self.success_count >= self.half_open_max_calls:
                    self._reset()
                    logger.info("Circuit breaker CLOSED - service recovered")
            else:
                self.failure_count = 0
    
    async def _on_failure(self):
        async with self._lock:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.state == CircuitState.HALF_OPEN:
                self.state = CircuitState.OPEN
                logger.warning("Circuit breaker OPEN - service still failing")
            elif self.failure_count >= self.failure_threshold:
                self.state = CircuitState.OPEN
                logger.warning(f"Circuit breaker OPEN after {self.failure_count} failures")
    
    def _reset(self):
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.half_open_calls = 0

class CircuitBreakerOpen(Exception):
    pass

# Użycie
breaker = CircuitBreaker(failure_threshold=5, recovery_timeout=60)

async def fetch_with_circuit_breaker(source):
    return await breaker.call(fetch_data, source)
```

#### Dead Letter Queue (DLQ)

```python
class ScrapingQueue:
    """
    Kolejka z Dead Letter Queue dla nieudanych itemów.
    """
    def __init__(self, rabbitmq_url: str):
        self.connection = pika.BlockingConnection(pika.URLParameters(rabbitmq_url))
        self.channel = self.connection.channel()
        
        # Główna kolejka
        self.channel.queue_declare(queue='scraping', durable=True)
        # DLQ - dla itemów które wielokrotnie się nie udało
        self.channel.queue_declare(queue='scraping_dlq', durable=True)
        # Kolejka retry - z opóźnieniem
        self.channel.queue_declare(
            queue='scraping_retry',
            arguments={
                'x-message-ttl': 3600000,  # 1h TTL
                'x-dead-letter-exchange': '',
                'x-dead-letter-routing-key': 'scraping'
            }
        )
    
    def publish(self, item: Dict, retry_count: int = 0):
        """Publikowanie zadania do kolejki."""
        item['retry_count'] = retry_count
        item['published_at'] = datetime.utcnow().isoformat()
        
        self.channel.basic_publish(
            exchange='',
            routing_key='scraping',
            body=json.dumps(item),
            properties=pika.BasicProperties(
                delivery_mode=2,  # Persistent
                content_type='application/json'
            )
        )
    
    def publish_to_dlq(self, item: Dict, error: str):
        """Przeniesienie do DLQ po wyczerpaniu retry."""
        item['error'] = error
        item['failed_at'] = datetime.utcnow().isoformat()
        
        self.channel.basic_publish(
            exchange='',
            routing_key='scraping_dlq',
            body=json.dumps(item),
            properties=pika.BasicProperties(delivery_mode=2)
        )
        
        logger.error(f"Item moved to DLQ: {item.get('id')}, error: {error}")
    
    def schedule_retry(self, item: Dict, delay_hours: int = 1):
        """Zaplanowanie retry z opóźnieniem."""
        item['retry_after'] = (datetime.utcnow().isoformat(),)
        
        self.channel.basic_publish(
            exchange='',
            routing_key='scraping_retry',
            body=json.dumps(item),
            properties=pika.BasicProperties(
                delivery_mode=2,
                expiration=str(delay_hours * 3600000)  # TTL w ms
            )
        )
```

#### Timeout i Connection Handling

```python
class RobustHTTPClient:
    """
    Klient HTTP z obsługą timeoutów i connection pooling.
    """
    def __init__(self):
        self.timeout_config = aiohttp.ClientTimeout(
            total=60,           # Całkowity timeout
            connect=10,         # Timeout na nawiązanie połączenia
            sock_read=30        # Timeout na odczyt danych
        )
        
        self.connector = aiohttp.TCPConnector(
            limit=100,                    # Max połączeń
            limit_per_host=10,            # Max połączeń per host
            ttl_dns_cache=300,            # Cache DNS (5 min)
            use_dns_cache=True,
            enable_cleanup_closed=True,   # Czyszczenie zamkniętych
            force_close=False             # Keep-alive
        )
    
    async def fetch(
        self,
        url: str,
        headers: Dict = None,
        allow_redirects: bool = True,
        max_redirects: int = 10
    ) -> str:
        """Pobieranie danych z obsługą błędów."""
        async with aiohttp.ClientSession(
            connector=self.connector,
            timeout=self.timeout_config
        ) as session:
            try:
                async with session.get(
                    url,
                    headers=headers,
                    allow_redirects=allow_redirects,
                    max_redirects=max_redirects,
                    ssl=False  # Opcjonalnie - dla developmentu
                ) as response:
                    if response.status == 429:  # Too Many Requests
                        retry_after = int(response.headers.get('Retry-After', 60))
                        raise RateLimitError(f"Rate limited. Retry after {retry_after}s")
                    
                    response.raise_for_status()
                    return await response.text()
                    
            except aiohttp.ClientConnectorError as e:
                logger.error(f"Connection error: {e}")
                raise ConnectionError(f"Cannot connect to {url}: {e}")
            except asyncio.TimeoutError:
                logger.error(f"Timeout error: {url}")
                raise TimeoutError(f"Request timeout: {url}")
            except aiohttp.TooManyRedirects:
                logger.error(f"Too many redirects: {url}")
                raise RedirectError(f"Redirect loop detected: {url}")

class RateLimitError(Exception):
    pass

class ConnectionError(Exception):
    pass

class RedirectError(Exception):
    pass
```

### 8.6 Anti-Detection i Rotacja Proxy

#### Rotacja User-Agent

```python
USER_AGENTS = [
    # Chrome Windows
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36",
    # Firefox Windows
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/121.0",
    # Chrome Mac
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    # Safari Mac
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.1 Safari/605.1.15",
    # Chrome Linux
    "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
    # Mobile
    "Mozilla/5.0 (iPhone; CPU iPhone OS 17_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.1 Mobile/15E148 Safari/604.1",
    "Mozilla/5.0 (Linux; Android 10; SM-G973F) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Mobile Safari/537.36"
]

class UserAgentRotator:
    """Rotator User-Agentów z zapobieganiem powtórzeniom."""
    
    def __init__(self):
        self.recent_agents = []
        self.max_recent = 3  # Ile ostatnich nie powtarzać
    
    def get_random(self) -> str:
        available = [ua for ua in USER_AGENTS if ua not in self.recent_agents]
        if not available:
            available = USER_AGENTS
        
        chosen = random.choice(available)
        self.recent_agents.append(chosen)
        
        if len(self.recent_agents) > self.max_recent:
            self.recent_agents.pop(0)
        
        return chosen

ua_rotator = UserAgentRotator()
```

#### Proxy Rotation

```python
class ProxyRotator:
    """
    Zarządzanie pulą proxy z rotacją i health check.
    """
    def __init__(self, proxy_list: List[str]):
        self.proxies = proxy_list
        self.failed_proxies = set()
        self.current_index = 0
        self._lock = asyncio.Lock()
    
    async def get_next(self) -> Optional[str]:
        """Pobierz następne działające proxy."""
        async with self._lock:
            available = [
                p for p in self.proxies 
                if p not in self.failed_proxies
            ]
            
            if not available:
                # Wszystkie proxy zawiodły - reset
                self.failed_proxies.clear()
                available = self.proxies
            
            proxy = available[self.current_index % len(available)]
            self.current_index += 1
            return proxy
    
    def mark_failed(self, proxy: str):
        """Oznacz proxy jako niedziałające."""
        self.failed_proxies.add(proxy)
        logger.warning(f"Proxy marked as failed: {proxy}")

# Konfiguracja proxy per źródło
PROXY_CONFIGS = {
    'policja': {
        'enabled': False,
        'proxy_list': []
    },
    'olx': {
        'enabled': True,
        'proxy_list': [
            'http://proxy1:8080',
            'http://proxy2:8080',
            'http://proxy3:8080'
        ]
    }
}
```

#### Stealth Headers

```python
STEALTH_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8',
    'Accept-Language': 'pl-PL,pl;q=0.9,en-US;q=0.8,en;q=0.7',
    'Accept-Encoding': 'gzip, deflate, br',
    'DNT': '1',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
    'Sec-Fetch-Dest': 'document',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-Site': 'none',
    'Sec-Fetch-User': '?1',
    'Cache-Control': 'max-age=0'
}

def get_stealth_headers(user_agent: str) -> Dict[str, str]:
    """Generowanie zestawu nagłówków imitujących przeglądarkę."""
    headers = STEALTH_HEADERS.copy()
    headers['User-Agent'] = user_agent
    
    # Dodaj referer dla niektórych żądań (losowo)
    if random.random() > 0.7:
        headers['Referer'] = 'https://www.google.com/'
    
    return headers
```

#### Rate Limiting per Source

```python
class RateLimiter:
    """Rate limiter z zapisywaniem stanu w Redis."""
    
    def __init__(self, redis_client):
        self.redis = redis_client
        self.limits = {
            'default': (1, 2),      # 1 request per 2 seconds
            'policja': (1, 3),      # 1 request per 3 seconds
            'olx': (1, 5),          # 1 request per 5 seconds (agresywna ochrona)
            'facebook': (1, 10),    # 1 request per 10 seconds
            'pracuj': (1, 2)        # 1 request per 2 seconds
        }
    
    async def acquire(self, source_id: str):
        """Sprawdź i poczekaj jeśli trzeba."""
        rate, period = self.limits.get(source_id, self.limits['default'])
        key = f"rate_limit:{source_id}"
        
        while True:
            current = await self.redis.get(key)
            if not current:
                await self.redis.setex(key, period, '1')
                return
            
            await asyncio.sleep(period / rate)

# Użycie w workerze
rate_limiter = RateLimiter(redis_client)

async def fetch_with_rate_limit(source):
    await rate_limiter.acquire(source['id'])
    return await fetch_data(source)
```

#### Cloudflare Bypass (opcjonalnie)

```python
# Dla stron chronionych przez Cloudflare
# Wymaga: pip install cloudscraper

import cloudscraper

class CloudflareBypassClient:
    """Klient omijający Cloudflare challenges."""
    
    def __init__(self):
        self.scraper = cloudscraper.create_scraper(
            browser={
                'browser': 'chrome',
                'platform': 'windows',
                'desktop': True
            }
        )
    
    def fetch(self, url: str, headers: Dict = None) -> str:
        """Pobierz stronę omijając Cloudflare."""
        response = self.scraper.get(url, headers=headers)
        return response.text

# Uwaga: cloudscraper jest synchroniczny
# Dla async użyj: curl_cffi lub playwright
```

### 8.7 Monitoring i Alerting Scrapingu

```python
class ScrapingMonitor:
    """Monitorowanie procesu scrapingu z metrykami."""
    
    def __init__(self):
        self.metrics = {
            'items_scraped': 0,
            'items_failed': 0,
            'retries': 0,
            'circuit_breaker_opens': 0,
            'rate_limit_hits': 0
        }
    
    def record_success(self, source_id: str, items_count: int):
        self.metrics['items_scraped'] += items_count
        logger.info(f"Source {source_id}: scraped {items_count} items")
    
    def record_failure(self, source_id: str, error: str):
        self.metrics['items_failed'] += 1
        logger.error(f"Source {source_id}: failed - {error}")
        
        # Alert jeśli za dużo błędów
        if self.metrics['items_failed'] > 10:
            self.send_alert(f"High failure rate for {source_id}")
    
    def send_alert(self, message: str):
        # Integracja z Slack/Email
        pass
```

---

## 9. SEO I STRUKTURY DANYCH

### 9.1 Struktura SEO Globalna

#### Meta Tagi (Wymagane)
```html
<!-- Podstawowe -->
<title>{page_title} | {site_name}</title>
<meta name="description" content="{page_description}">
<meta name="keywords" content="{keywords}">
<meta name="robots" content="{robots_directive}">
<meta name="author" content="{site_name}">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Open Graph -->
<meta property="og:title" content="{og_title}">
<meta property="og:description" content="{og_description}">
<meta property="og:image" content="{og_image}">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:url" content="{canonical_url}">
<meta property="og:type" content="{og_type}">
<meta property="og:site_name" content="{site_name}">
<meta property="og:locale" content="pl_PL">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{twitter_title}">
<meta name="twitter:description" content="{twitter_description}">
<meta name="twitter:image" content="{twitter_image}">

<!-- Canonical -->
<link rel="canonical" href="{canonical_url}">

<!-- Hreflang -->
<link rel="alternate" hreflang="pl" href="{url_pl}">
<link rel="alternate" hreflang="x-default" href="{url_default}">

<!-- Dodatkowe -->
<meta name="theme-color" content="{primary_color}">
<link rel="icon" type="image/x-icon" href="{favicon_url}">
<link rel="apple-touch-icon" href="{apple_touch_icon_url}">
```

#### Schema.org (JSON-LD)

**Strona Glowna:**
```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "4Torun",
  "url": "https://4torun.pl",
  "description": "Regionalny serwis informacyjny Torunia",
  "publisher": {
    "@type": "Organization",
    "name": "4Torun",
    "logo": {
      "@type": "ImageObject",
      "url": "https://4torun.pl/logo.png"
    }
  },
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://4torun.pl/szukaj?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
```

**Artykul (NewsArticle):**
```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Tytul artykulu",
  "description": "Opis artykulu",
  "image": [
    "https://4torun.pl/image-1200x800.jpg",
    "https://4torun.pl/image-800x600.jpg"
  ],
  "datePublished": "2024-02-12T10:00:00+01:00",
  "dateModified": "2024-02-12T12:00:00+01:00",
  "author": {
    "@type": "Organization",
    "name": "Redakcja 4Torun"
  },
  "publisher": {
    "@type": "Organization",
    "name": "4Torun",
    "logo": {
      "@type": "ImageObject",
      "url": "https://4torun.pl/logo.png"
    }
  },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://4torun.pl/wiadomosci/tytul-artykulu"
  }
}
```

**Firma (LocalBusiness):**
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Nazwa Firmy",
  "image": "https://4torun.pl/firma/logo.jpg",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "ul. Przykladowa 1",
    "addressLocality": "Torun",
    "postalCode": "87-100",
    "addressCountry": "PL"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": "53.0138",
    "longitude": "18.5984"
  },
  "url": "https://4torun.pl/firmy/kategoria/nazwa-firmy",
  "telephone": "+48123456789",
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
      "opens": "08:00",
      "closes": "16:00"
    }
  ]
}
```

**Oferta Pracy (JobPosting):**
```json
{
  "@context": "https://schema.org",
  "@type": "JobPosting",
  "title": "Stanowisko",
  "description": "Opis stanowiska",
  "datePosted": "2024-02-12",
  "validThrough": "2024-03-12",
  "employmentType": "FULL_TIME",
  "hiringOrganization": {
    "@type": "Organization",
    "name": "Nazwa Firmy",
    "sameAs": "https://firma.pl"
  },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "addressLocality": "Torun",
      "addressRegion": "kujawsko-pomorskie",
      "addressCountry": "PL"
    }
  },
  "baseSalary": {
    "@type": "MonetaryAmount",
    "currency": "PLN",
    "value": {
      "@type": "QuantitativeValue",
      "minValue": 5000,
      "maxValue": 8000,
      "unitText": "MONTH"
    }
  }
}
```

**Wydarzenie (Event):**
```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Nazwa wydarzenia",
  "startDate": "2024-03-01T18:00:00+01:00",
  "endDate": "2024-03-01T22:00:00+01:00",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "Miejsce wydarzenia",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "ul. Przykladowa 1",
      "addressLocality": "Torun",
      "postalCode": "87-100",
      "addressCountry": "PL"
    }
  },
  "image": "https://4torun.pl/event-image.jpg",
  "description": "Opis wydarzenia",
  "offers": {
    "@type": "Offer",
    "url": "https://4torun.pl/bilety",
    "price": "50",
    "priceCurrency": "PLN",
    "availability": "https://schema.org/InStock"
  }
}
```

### 9.2 Struktura URL

#### Formaty Permalinkow
```
# Wpisy
/wiadomosci/{slug}
/kronika-policyjna/{slug}
/firmy/{slug}
/ogloszenia/{slug}
/praca/{slug}
/nekrologi/{slug}
/przewodnik/{slug}
/ludzie/{slug}

# Kategorie (hierarchiczne)
/firmy/{category-slug}/
/firmy/{parent-category}/{child-category}/

# Archiwum (daty)
/2024/              # Rok
/2024/02/           # Miesiac
/2024/02/12/        # Dzien

# Autorzy
/autor/{author-slug}/

# Tagi
/tag/{tag-slug}/

# Wyszukiwanie
/szukaj?q={query}

# Strony statyczne
/o-nas
/regulamin
/polityka-prywatnosci
/kontakt

# Mapy
/mapa-strony          # HTML sitemap
/sitemap.xml          # XML sitemap
/sitemap-posts.xml    # Sitemap wpisow
/sitemap-categories.xml # Sitemap kategorii
```

#### Przyjazne URL-e (Examples)
```
# Wiadomosci
https://4torun.pl/wiadomosci/walentynkowy-bal-paczusia-w-toruniu
https://4torun.pl/wiadomosci/nowe-inwestycje-w-toruniu-2024

# Kronika
https://4torun.pl/kronika-policyjna/smiertelny-wypadek-na-autostradzie

# Firmy (z kategoria)
https://4torun.pl/firmy/dentysta-torun/dentysta-dr-kowalski

# Praca
https://4torun.pl/praca/praca-na-hali-work-profit

# Archiwum
https://4torun.pl/2024/02/           # Luty 2024
https://4torun.pl/2024/02/12/        # 12 lutego 2024
```

### 9.3 Sitemap.xml

#### Struktura Sitemap
```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://4torun.pl/sitemap-posts.xml</loc>
    <lastmod>2024-02-12T10:00:00+00:00</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://4torun.pl/sitemap-categories.xml</loc>
    <lastmod>2024-02-12T10:00:00+00:00</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://4torun.pl/sitemap-tags.xml</loc>
    <lastmod>2024-02-12T10:00:00+00:00</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://4torun.pl/sitemap-static.xml</loc>
    <lastmod>2024-02-01T00:00:00+00:00</lastmod>
  </sitemap>
</sitemapindex>
```

#### Sitemap Wpisow
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://4torun.pl/wiadomosci/tytul-wpisu</loc>
    <lastmod>2024-02-12T10:00:00+00:00</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
  </url>
  <!-- ... -->
</urlset>
```

#### Priorytety w Sitemap
| Typ | Priorytet | Changefreq |
|-----|-----------|------------|
| Strona glowna | 1.0 | daily |
| Wpisy | 0.8 | daily |
| Kategorie | 0.6 | weekly |
| Tagi | 0.4 | weekly |
| Archiwum | 0.3 | weekly |
| Strony statyczne | 0.5 | monthly |

### 9.4 Robots.txt

```
User-agent: *
Allow: /

# Disallow admin areas
Disallow: /admin/
Disallow: /api/
Disallow: /login
Disallow: /register

# Disallow search results
Disallow: /szukaj?

# Disallow duplicate content
Disallow: /tag/*?page=
Disallow: /autor/*?page=

# Allow specific bots
User-agent: Googlebot
Allow: /
Crawl-delay: 1

User-agent: Bingbot
Allow: /
Crawl-delay: 2

# Sitemap
Sitemap: https://4torun.pl/sitemap.xml
```

---

## 10. MONITORING, LOGI I RAPORTOWANIE

### 10.1 System Monitoringu

#### Metryki do Monitorowania

**Metryki Aplikacji:**
- Czas odpowiedzi HTTP (p95, p99)
- Liczba requestow na minute
- Liczba bledow 4xx, 5xx
- Wykorzystanie CPU/RAM
- Wykorzystanie dysku
- Liczba polaczen do bazy danych
- Rozmiar cache

**Metryki Biznesowe:**
- Liczba wyswietlen stron
- Unikalni uzytkownicy
- Sesje
- Wspolczynnik odrzucen (bounce rate)
- Sredni czas na stronie
- Konwersje (komentarze, oceny)

**Metryki Scrapingu:**
- Liczba pobranych elementow
- Liczba bledow scrapingu
- Czas wykonania
- Blokady IP (rate limiting)

#### Dashboard Monitoringu
```
┌─────────────────────────────────────────────────────────────────┐
│ SYSTEM HEALTH                      STATUS: HEALTHY              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────┐ │
│  │ CPU Usage   │ │ RAM Usage   │ │ DB Conn     │ │ Disk Usage │ │
│  │ 45%         │ │ 62%         │ │ 12/100      │ │ 78%        │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────┘ │
│                                                                 │
│  REQUESTS (Last 24h)                                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  [Wykres liniowy: requests/minute przez 24h]             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ERROR RATE (Last 24h)                                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  [Wykres: % bledow 4xx i 5xx]                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  TOP DOMAINS BY TRAFFIC (Today)                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  1. 4torun.pl        15,420 views                        │   │
│  │  2. 4bydgoszcz.pl    12,105 views                        │   │
│  │  3. 4warszawa.pl     28,900 views                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
│  RECENT ALERTS                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  [14:32] ERROR: Scraper 'policja-torun' failed           │   │
│  │  [13:15] WARNING: High memory usage on worker-2          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 System Logowania

#### Poziomy Logowania
| Poziom | Uzycie | Retencja |
|--------|--------|----------|
| DEBUG | Rozwoj, debugowanie | 7 dni |
| INFO | Standardowe operacje | 30 dni |
| WARNING | Nieprawidlowosci, ale dziala | 90 dni |
| ERROR | Bledy wymagajace uwagi | 365 dni |
| CRITICAL | Krytyczne awarie | 365 dni |

#### Struktura Logu
```json
{
  "timestamp": "2024-02-12T14:32:15.123Z",
  "level": "ERROR",
  "category": "scraper",
  "domain_id": "4torun.pl",
  "user_id": "user-uuid",
  "action": "fetch_data",
  "message": "Failed to fetch data from source",
  "context": {
    "source_id": "policja-torun",
    "url": "https://torun.policja.gov.pl/...",
    "error": "Connection timeout",
    "retry_count": 3
  },
  "ip_address": "192.168.1.1",
  "user_agent": "Mozilla/5.0...",
  "request_id": "req-uuid",
  "duration_ms": 30000
}
```

#### Logi Audytowe
```json
{
  "timestamp": "2024-02-12T14:32:15.123Z",
  "user_id": "user-uuid",
  "action": "update",
  "entity_type": "post",
  "entity_id": "post-uuid",
  "entity_title": "Tytul wpisu",
  "changes": {
    "before": {
      "title": "Stary tytul",
      "status": "draft"
    },
    "after": {
      "title": "Nowy tytul",
      "status": "published"
    }
  },
  "ip_address": "192.168.1.1"
}
```

### 10.3 Implementacja Systemu Logowania (Winston)

#### Konfiguracja Loggera

```typescript
// lib/logger.ts
import winston from 'winston';
import DailyRotateFile from 'winston-daily-rotate-file';

const { combine, timestamp, json, errors, printf } = winston.format;

// Format dla developmentu (czytelniejszy)
const devFormat = printf(({ level, message, timestamp, ...metadata }) => {
  let msg = `${timestamp} [${level.toUpperCase()}]: ${message}`;
  if (Object.keys(metadata).length > 0) {
    msg += ` ${JSON.stringify(metadata)}`;
  }
  return msg;
});

// Transporty - pliki rotowane dzienne
const fileTransport = new DailyRotateFile({
  filename: 'logs/application-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  zippedArchive: true,
  maxSize: '20m',
  maxFiles: '30d'
});

const errorTransport = new DailyRotateFile({
  filename: 'logs/error-%DATE%.log',
  datePattern: 'YYYY-MM-DD',
  zippedArchive: true,
  maxSize: '20m',
  maxFiles: '90d',
  level: 'error'
});

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  defaultMeta: {
    service: 'regionalne-serwisy',
    environment: process.env.NODE_ENV
  },
  format: combine(
    timestamp(),
    errors({ stack: true }),
    json()
  ),
  transports: [
    fileTransport,
    errorTransport
  ],
  exceptionHandlers: [
    new winston.transports.File({ filename: 'logs/exceptions.log' })
  ],
  rejectionHandlers: [
    new winston.transports.File({ filename: 'logs/rejections.log' })
  ]
});

// W developmentu dodaj konsolę
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: combine(
      timestamp(),
      devFormat
    )
  }));
}

// Logger kontekstowy (z request_id, user_id)
export const createContextLogger = (context: {
  requestId?: string;
  userId?: string;
  domainId?: string;
}) => {
  return logger.child(context);
};
```

#### Middleware Logowania HTTP

```typescript
// middleware/requestLogger.ts
import { Request, Response, NextFunction } from 'express';
import { createContextLogger } from '../lib/logger';
import { v4 as uuidv4 } from 'uuid';

export const requestLogger = (req: Request, res: Response, next: NextFunction) => {
  const requestId = req.headers['x-request-id'] as string || uuidv4();
  req.requestId = requestId;
  
  const startTime = Date.now();
  const contextLogger = createContextLogger({
    requestId,
    userId: req.user?.id,
    domainId: req.domain?.id
  });
  
  // Logowanie startu requestu
  contextLogger.info('HTTP Request Started', {
    method: req.method,
    url: req.url,
    userAgent: req.headers['user-agent'],
    ip: req.ip
  });
  
  // Logowanie zakończenia requestu
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    contextLogger.info('HTTP Request Completed', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      durationMs: duration,
      contentLength: res.get('content-length')
    });
  });
  
  next();
};
```

### 10.4 Metryki i Monitoring (Prometheus)

#### Konfiguracja Metryk

```typescript
// lib/metrics.ts
import client from 'prom-client';

// Rejestr globalny
export const register = new client.Registry();

// Dodaj defaultowe metryki (CPU, pamięć, itp.)
client.collectDefaultMetrics({ register });

// Customowe metryki

// 1. Czas trwania requestów HTTP
export const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10]
});

// 2. Licznik requestów
export const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

// 3. Aktywne połączenia
export const activeConnections = new client.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

// 4. Metryki biznesowe
export const postsCreatedTotal = new client.Counter({
  name: 'posts_created_total',
  help: 'Total number of posts created',
  labelNames: ['domain_id', 'post_type']
});

export const scrapingItemsTotal = new client.Counter({
  name: 'scraping_items_total',
  help: 'Total items scraped',
  labelNames: ['source_id', 'status']
});

// 5. Błędy
export const errorsTotal = new client.Counter({
  name: 'errors_total',
  help: 'Total number of errors',
  labelNames: ['type', 'route']
});

// Rejestracja metryk
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(postsCreatedTotal);
register.registerMetric(scrapingItemsTotal);
register.registerMetric(errorsTotal);
```

#### Middleware Metryk

```typescript
// middleware/metrics.ts
import { Request, Response, NextFunction } from 'express';
import {
  httpRequestDuration,
  httpRequestTotal,
  activeConnections
} from '../lib/metrics';

export const metricsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  activeConnections.inc();
  
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route?.path || req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode.toString())
      .observe(duration);
    
    httpRequestTotal
      .labels(req.method, route, res.statusCode.toString())
      .inc();
    
    activeConnections.dec();
  });
  
  next();
};

// Endpoint metryk (dla Prometheus)
export const metricsEndpoint = async (req: Request, res: Response) => {
  const { register } = await import('../lib/metrics');
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
};
```

### 10.5 Health Check Endpoint

```typescript
// routes/health.ts
import { Router } from 'express';
import { PrismaClient } from '@prisma/client';
import Redis from 'ioredis';

const router = Router();
const prisma = new PrismaClient();
const redis = new Redis(process.env.REDIS_URL);

interface HealthCheck {
  status: 'healthy' | 'unhealthy' | 'degraded';
  timestamp: string;
  checks: {
    database: { status: string; responseTime: number };
    redis: { status: string; responseTime: number };
    disk: { status: string; freeSpace: string };
    memory: { status: string; usage: string };
  };
  version: string;
}

// Liveness probe (czy aplikacja żyje)
router.get('/health/live', (req, res) => {
  res.status(200).json({ status: 'alive' });
});

// Readiness probe (czy gotowa do przyjmowania ruchu)
router.get('/health/ready', async (req, res) => {
  const checks = await Promise.all([
    checkDatabase(),
    checkRedis()
  ]);
  
  const allHealthy = checks.every(c => c.status === 'ok');
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'not_ready',
    checks: {
      database: checks[0],
      redis: checks[1]
    }
  });
});

// Pełny health check (szczegółowy)
router.get('/health', async (req, res) => {
  const startTime = Date.now();
  
  const [dbCheck, redisCheck, diskCheck, memoryCheck] = await Promise.all([
    checkDatabase(),
    checkRedis(),
    checkDiskSpace(),
    checkMemory()
  ]);
  
  const responseTime = Date.now() - startTime;
  
  const isHealthy = 
    dbCheck.status === 'ok' && 
    redisCheck.status === 'ok' &&
    diskCheck.status === 'ok';
  
  const isDegraded = 
    dbCheck.status === 'ok' && 
    redisCheck.status === 'ok' &&
    (diskCheck.status !== 'ok' || memoryCheck.status !== 'ok');
  
  const status: HealthCheck['status'] = isHealthy 
    ? 'healthy' 
    : isDegraded 
      ? 'degraded' 
      : 'unhealthy';
  
  const healthCheck: HealthCheck = {
    status,
    timestamp: new Date().toISOString(),
    checks: {
      database: dbCheck,
      redis: redisCheck,
      disk: diskCheck,
      memory: memoryCheck
    },
    version: process.env.APP_VERSION || 'unknown'
  };
  
  res.status(isHealthy ? 200 : isDegraded ? 200 : 503).json(healthCheck);
});

// Funkcje pomocnicze
async function checkDatabase() {
  const start = Date.now();
  try {
    await prisma.$queryRaw`SELECT 1`;
    return {
      status: 'ok',
      responseTime: Date.now() - start
    };
  } catch (error) {
    return {
      status: 'error',
      responseTime: Date.now() - start,
      error: error.message
    };
  }
}

async function checkRedis() {
  const start = Date.now();
  try {
    await redis.ping();
    return {
      status: 'ok',
      responseTime: Date.now() - start
    };
  } catch (error) {
    return {
      status: 'error',
      responseTime: Date.now() - start,
      error: error.message
    };
  }
}

async function checkDiskSpace() {
  // Implementacja sprawdzania miejsca na dysku
  // Użyj: import checkDiskSpace from 'check-disk-space';
  return {
    status: 'ok',
    freeSpace: '50GB'
  };
}

async function checkMemory() {
  const used = process.memoryUsage();
  const total = require('os').totalmem();
  const usage = (used.heapUsed / total) * 100;
  
  return {
    status: usage > 90 ? 'warning' : 'ok',
    usage: `${usage.toFixed(2)}%`
  };
}

export default router;
```

### 10.6 Alerting i Powiadomienia

```typescript
// lib/alerts.ts
import { WebhookClient } from 'discord.js';
import nodemailer from 'nodemailer';

interface Alert {
  severity: 'info' | 'warning' | 'error' | 'critical';
  title: string;
  message: string;
  metadata?: Record<string, any>;
  timestamp: Date;
}

class AlertManager {
  private discordWebhook?: WebhookClient;
  private emailTransporter?: nodemailer.Transporter;
  
  constructor() {
    // Discord Webhook (opcjonalnie)
    if (process.env.DISCORD_WEBHOOK_URL) {
      this.discordWebhook = new WebhookClient({
        url: process.env.DISCORD_WEBHOOK_URL
      });
    }
    
    // Email (opcjonalnie)
    if (process.env.SMTP_HOST) {
      this.emailTransporter = nodemailer.createTransport({
        host: process.env.SMTP_HOST,
        port: parseInt(process.env.SMTP_PORT || '587'),
        auth: {
          user: process.env.SMTP_USER,
          pass: process.env.SMTP_PASS
        }
      });
    }
  }
  
  async sendAlert(alert: Alert): Promise<void> {
    // Log do systemu
    logger[alert.severity](alert.title, {
      message: alert.message,
      ...alert.metadata
    });
    
    // Krytyczne alerty - wysyłaj wszędzie
    if (alert.severity === 'critical') {
      await Promise.all([
        this.sendDiscord(alert),
        this.sendEmail(alert)
      ]);
    }
    
    // Błędy - wysyłaj na Discord
    if (alert.severity === 'error') {
      await this.sendDiscord(alert);
    }
  }
  
  private async sendDiscord(alert: Alert): Promise<void> {
    if (!this.discordWebhook) return;
    
    const colors = {
      info: 0x3498db,
      warning: 0xf1c40f,
      error: 0xe74c3c,
      critical: 0x9b59b6
    };
    
    await this.discordWebhook.send({
      embeds: [{
        title: alert.title,
        description: alert.message,
        color: colors[alert.severity],
        fields: Object.entries(alert.metadata || {}).map(([key, value]) => ({
          name: key,
          value: String(value).substring(0, 1000),
          inline: true
        })),
        timestamp: alert.timestamp.toISOString()
      }]
    });
  }
  
  private async sendEmail(alert: Alert): Promise<void> {
    if (!this.emailTransporter) return;
    
    await this.emailTransporter.sendMail({
      from: process.env.ALERT_FROM_EMAIL,
      to: process.env.ALERT_TO_EMAIL,
      subject: `[${alert.severity.toUpperCase()}] ${alert.title}`,
      text: `${alert.message}\n\nMetadata: ${JSON.stringify(alert.metadata, null, 2)}`
    });
  }
}

export const alertManager = new AlertManager();

// Automatyczne alerty na podstawie metryk
export const setupMetricAlerts = () => {
  // Alert przy wysokim error rate
  setInterval(async () => {
    const { errorsTotal } = await import('./metrics');
    // Logika sprawdzania progu i wysyłania alertu
  }, 60000);
  
  // Alert przy wysokim zużyciu pamięci
  setInterval(async () => {
    const usage = process.memoryUsage();
    if (usage.heapUsed > 0.9 * require('os').totalmem()) {
      await alertManager.sendAlert({
        severity: 'warning',
        title: 'High Memory Usage',
        message: `Memory usage is at ${(usage.heapUsed / 1024 / 1024).toFixed(2)}MB`,
        timestamp: new Date()
      });
    }
  }, 300000); // co 5 min
};
```

### 10.7 Konfiguracja Prometheus i Grafana

#### docker-compose.monitoring.yml

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

volumes:
  prometheus_data:
  grafana_data:
```

#### prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alerts.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'regionalne-serwisy-api'
    static_configs:
      - targets: ['api:3001']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'regionalne-serwisy-frontend'
    static_configs:
      - targets: ['frontend:3000']
    scrape_interval: 10s

  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['redis-exporter:9121']
```

### 10.8 Raportowanie

#### Typy Raportow

1. **Raport Dzienny (Email)**
   - Podsumowanie ruchu (dzien vs wczoraj)
   - Nowe wpisy
   - Nowi uzytkownicy
   - Bledy systemowe
   - Status scrapingu

2. **Raport Tygodniowy**
   - Trendy ruchu (wykres tygodniowy)
   - Top 10 tresci
   - Top 10 zrodel ruchu
   - Podsumowanie scrapingu
   - Wykorzystanie zasobow

3. **Raport Miesieczny**
   - Pelna analiza ruchu
   - Analiza SEO (pozycje, indeksowanie)
   - Podsumowanie finansowe (jesli dotyczy)
   - Cele i KPI
   - Rekomendacje

4. **Raport Customowy**
   - Wybor zakresu dat
   - Wybor metryk
   - Wybor domen
   - Format: PDF, Excel, CSV

#### Przyklad Raportu (Fragment)
```
═══════════════════════════════════════════════════════════════
RAPORT DZIENNY - 12.02.2024
System: Regionalne Serwisy
═══════════════════════════════════════════════════════════════

RAUCH NA STRONACH
─────────────────
Laczne wyswietlenia:     45,230 (+12% vs wczoraj)
Unikalni uzytkownicy:    12,450 (+8% vs wczoraj)
Nowi uzytkownicy:        1,230
Sredni czas sesji:       4m 32s
Wspolczynnik odrzucen:   42% (-3pp vs wczoraj)

TOP DOMENY
──────────
1. 4torun.pl        15,420 wyswietlen (34%)
2. 4bydgoszcz.pl    12,105 wyswietlen (27%)
3. 4warszawa.pl      8,900 wyswietlen (20%)

TOP WPISY
─────────
1. "Śmiertelny wypadek na A1"          3,420 wysw.
2. "Nowa inwestycja w centrum"         2,890 wysw.
3. "Koncert w Filharmonii"             1,560 wysw.

SCRAPING
────────
Zrodla przetworzone:    12
Nowe wpisy:            45
Zaktualizowane:        12
Bledy:                  0

BLEDY SYSTEMOWE
───────────────
Poziom ERROR:           3 (wszystkie obsluzone)
Poziom WARNING:        12
Status:                STABILNY

NASTEPNE ZADANIA CRON
─────────────────────
14:00 - Backup przyrostowy
16:00 - Scraper: policja
18:00 - Scraper: miasto
20:00 - Generowanie sitemap

═══════════════════════════════════════════════════════════════
```

---



## 11. WDROZENIE - PLAN ETAPOWY

### 11.1 Fazy Projektu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MAPA ETAPOW WDROZENIA                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ETAP 1: PRZYGOTOWANIE INFRASTRUKTURY          [Tydzien 1-2]               │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Konfiguracja hostingu (domeny, SSL)                           │       │
│  │ - Instalacja PostgreSQL, Redis, RabbitMQ                        │       │
│  │ - Konfiguracja srodowiska Node.js i Python                      │       │
│  │ - Setup monitoringu i logowania                                 │       │
│  │ - Konfiguracja backupu                                          │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 2: BAZA DANYCH I API                     [Tydzien 2-4]               │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Tworzenie schematow bazy danych (public + tenant)             │       │
│  │ - Implementacja API centralnego (Node.js/Express)               │       │
│  │ - System autentykacji i autoryzacji                             │       │
│  │ - System uprawnien (RBAC)                                       │       │
│  │ - Testy API                                                     │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 3: PANEL ADMINISTRACYJNY CENTRALNY       [Tydzien 4-7]               │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Setup projektu Next.js                                        │       │
│  │ - Layout i komponenty UI                                        │       │
│  │ - Modul zarzadzania domenami                                    │       │
│  │ - Modul zarzadzania uzytkownikami                               │       │
│  │ - Modul masowych operacji                                       │       │
│  │ - Modul zrodel danych                                           │       │
│  │ - Modul cron jobs                                               │       │
│  │ - System logow                                                  │       │
│  │ - Testy panelu                                                  │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 4: SYSTEM SCRAPINGU                      [Tydzien 6-8]               │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Implementacja workerow Python                                 │       │
│  │ - System kolejkowania (RabbitMQ)                                │       │
│  │ - Parsery dla roznych typow zrodel                              │       │
│  │ - Konfiguracja cron jobs                                        │       │
│  │ - Testy scrapingu                                               │       │
│  │ - Monitoring bledow                                             │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 5: SERWIS REGIONALNY (Frontend)          [Tydzien 8-11]              │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Setup projektu Next.js dla serwisow                           │       │
│  │ - System szablonow i motywow                                    │       │
│  │ - Komponenty wspolne (header, footer, cards)                    │       │
│  │ - Strony: Home, Archiwum, Wpis                                  │       │
│  │ - CPT: Wiadomosci, Kronika, Firmy, Praca, Nekrologi, Przewodnik│       │
│  │ - System komentarzy i ocen                                      │       │
│  │ - SEO (meta tagi, schema.org, sitemap)                          │       │
│  │ - Testy frontendu                                               │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 6: PANEL ADMINISTRACYJNY SERWISU         [Tydzien 11-13]             │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Dashboard serwisu                                             │       │
│  │ - Edytor wpisow (rich text)                                     │       │
│  │ - Zarzadzanie kategoriami i tagami                              │       │
│  │ - Zarzadzanie bannerami i menu                                  │       │
│  │ - Zarzadzanie widgetami                                         │       │
│  │ - Ustawienia serwisu                                            │       │
│  │ - Testy panelu serwisu                                          │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 7: WDROZENIE PIERWSZEJ DOMENY            [Tydzien 13-14]             │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Konfiguracja 4torun.pl                                        │       │
│  │ - Import poczatkowych danych                                    │       │
│  │ - Konfiguracja zrodel scrapingu                                 │       │
│  │ - Testy end-to-end                                              │       │
│  │ - Optymalizacja wydajnosci                                      │       │
│  │ - Uruchomienie produkcyjne                                      │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
│  ETAP 8: DOKUMENTACJA I PRZEKAZANIE            [Tydzien 14-15]             │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ - Dokumentacja techniczna                                       │       │
│  │ - Dokumentacja uzytkownika                                      │       │
│  │ - Szkolenie administratorow                                     │       │
│  │ - Szkolenie redaktorow                                          │       │
│  │ - Ostateczne testy                                              │       │
│  │ - Uruchomienie oficjalne                                        │       │
│  └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 11.2 Szczegolowe Taski

#### Sprint 1-2: Infrastruktura
| Task | Estymacja | Opis |
|------|-----------|------|
| Setup hosting | 4h | Konfiguracja kont hostingowych, domen |
| SSL certificates | 2h | Lets Encrypt dla wszystkich domen |
| PostgreSQL install | 4h | Instalacja, konfiguracja, security |
| Redis install | 2h | Instalacja, konfiguracja cache |
| RabbitMQ install | 3h | Instalacja, konfiguracja kolejek |
| Node.js setup | 2h | Instalacja, PM2 |
| Python setup | 2h | Instalacja, venv |
| Monitoring setup | 4h | Prometheus, Grafana (lub prostsze) |
| Backup config | 3h | Automatyczne backupy |
| CI/CD pipeline | 6h | Github Actions / GitLab CI |

#### Sprint 3-4: Baza Danych i API Centralne
| Task | Estymacja | Opis |
|------|-----------|------|
| Schema public | 8h | Tabele centralne |
| Schema tenant template | 6h | Szablon schema dla tenantow |
| Migrations system | 4h | System migracji (node-pg-migrate) |
| API Auth | 8h | JWT, logowanie, rejestracja |
| API RBAC | 8h | System uprawnien |
| API Users | 6h | CRUD uzytkownikow |
| API Domains | 8h | CRUD domen |
| API Sources | 6h | CRUD zrodel |
| API Cron | 4h | CRUD cron jobs |
| API Tests | 8h | Testy jednostkowe i integracyjne |

#### Sprint 5-7: Panel Centralny
| Task | Estymacja | Opis |
|------|-----------|------|
| Next.js setup | 4h | Projekt, konfiguracja |
| UI Kit | 8h | Komponenty, layout, nawigacja |
| Dashboard view | 6h | Strona glowna panelu |
| Domains list | 6h | Lista domen z filtrami |
| Domain detail | 8h | Szczegoly domeny z zakladkami |
| Users management | 8h | Zarzadzanie uzytkownikami |
| Sources management | 8h | Zarzadzanie zrodlami |
| Mass operations | 10h | System masowych operacji |
| Logs viewer | 6h | Przegladarka logow |
| Settings | 6h | Ustawienia systemowe |

#### Sprint 6-8: Scraping
| Task | Estymacja | Opis |
|------|-----------|------|
| Worker architecture | 6h | Struktura workerow Python |
| HTTP fetcher | 4h | Pobieranie danych, retry logic |
| HTML parser | 8h | BeautifulSoup parser |
| JSON parser | 4h | Parser JSON/API |
| RSS parser | 4h | Parser RSS |
| Database saver | 6h | Zapisywanie do PostgreSQL |
| Queue consumer | 6h | RabbitMQ consumer |
| Scheduler | 4h | Harmonogram zadan |
| Error handling | 6h | Obsluga bledow, retry |
| Tests | 6h | Testy workerow |

#### Sprint 8-11: Frontend Serwisu
| Task | Estymacja | Opis |
|------|-----------|------|
| Next.js setup | 4h | Projekt serwisu |
| Theme system | 8h | System motywow |
| Layout components | 8h | Header, footer, sidebar |
| Post card | 4h | Komponent karty wpisu |
| Home page | 8h | Strona glowna |
| Archive page | 6h | Strona archiwum |
| Single post | 8h | Strona pojedynczego wpisu |
| CPT views | 16h | Widoki dla wszystkich CPT |
| Comments system | 6h | Komentarze |
| Rating system | 4h | Oceny |
| Search | 6h | Wyszukiwanie |
| SEO setup | 8h | Meta, schema, sitemap |

#### Sprint 11-13: Panel Serwisu
| Task | Estymacja | Opis |
|------|-----------|------|
| Dashboard | 6h | Dashboard serwisu |
| Post editor | 16h | Edytor wpisow (TipTap) |
| Media library | 8h | Biblioteka mediow |
| Categories | 6h | Zarzadzanie kategoriami |
| Menus | 8h | Budowniczy menu |
| Widgets | 8h | Zarzadzanie widgetami |
| Banners | 6h | Zarzadzanie bannerami |
| Settings | 6h | Ustawienia serwisu |

#### Sprint 14-15: Finalizacja
| Task | Estymacja | Opis |
|------|-----------|------|
| Performance opt | 8h | Optymalizacja |
| Security audit | 6h | Audyt bezpieczenstwa |
| Documentation | 10h | Dokumentacja |
| Training | 8h | Szkolenie |
| Launch | 4h | Uruchomienie |

### 11.3 Stack Technologiczny

#### Backend
| Komponent | Technologia | Wersja |
|-----------|-------------|--------|
| Runtime | Node.js | 20 LTS |
| Framework | Express.js / Fastify | latest |
| Language | TypeScript | 5.x |
| Validation | Zod | latest |
| ORM | Prisma | latest |
| Auth | Passport.js + JWT | latest |
| Documentation | OpenAPI / Swagger | latest |

#### Frontend (Panel Centralny i Serwisy)
| Komponent | Technologia | Wersja |
|-----------|-------------|--------|
| Framework | Next.js | 14 |
| Language | TypeScript | 5.x |
| Styling | Tailwind CSS | 3.x |
| Components | shadcn/ui + Radix | latest |
| State | Zustand / React Query | latest |
| Forms | React Hook Form + Zod | latest |
| Editor | TipTap | latest |
| Charts | Recharts | latest |

#### Scraping Workers
| Komponent | Technologia | Wersja |
|-----------|-------------|--------|
| Language | Python | 3.11+ |
| HTTP | aiohttp | latest |
| Parser | BeautifulSoup4 | latest |
| Queue | pika (RabbitMQ) | latest |
| Database | psycopg2 / asyncpg | latest |
| Scheduler | APScheduler | latest |

#### Baza Danych i Cache
| Komponent | Technologia | Wersja |
|-----------|-------------|--------|
| Database | PostgreSQL | 15+ |
| Cache | Redis | 7+ |
| Queue | RabbitMQ | 3.12+ |
| Search | Elasticsearch | 8.x (opcjonalnie) |

#### Monitoring i Logi
| Komponent | Technologia | Uzycie |
|-----------|-------------|--------|
| Monitoring | Prometheus + Grafana | Metryki |
| Logi | Winston (Node) / structlog (Python) | Logowanie |
| APM | Sentry | Bledy |

### 11.4 Struktura Projektow

```
/home/host988956/projects/
├── admin-panel/                    # Panel centralny (Next.js)
│   ├── src/
│   │   ├── app/                   # Next.js 14 App Router
│   │   │   ├── (dashboard)/
│   │   │   │   ├── page.tsx       # Dashboard
│   │   │   │   ├── domeny/
│   │   │   │   ├── uzytkownicy/
│   │   │   │   ├── zrodla/
│   │   │   │   ├── cron/
│   │   │   │   └── logi/
│   │   │   ├── api/               # API Routes
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── ui/               # Komponenty UI
│   │   │   ├── forms/            # Formularze
│   │   │   └── layouts/          # Layouty
│   │   ├── lib/
│   │   │   ├── api.ts            # Klient API
│   │   │   ├── auth.ts           # Autentykacja
│   │   │   └── utils.ts          # Utils
│   │   └── types/
│   │       └── index.ts          # TypeScript types
│   ├── public/
│   ├── package.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   └── tsconfig.json
│
├── regional-service/              # Szablon serwisu regionalnego (Next.js)
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx           # Home
│   │   │   ├── [type]/
│   │   │   │   └── [slug]/
│   │   │   ├── admin/             # Panel serwisu
│   │   │   └── api/
│   │   ├── components/
│   │   ├── lib/
│   │   └── types/
│   └── ...
│
├── api-server/                    # API Centralne (Node.js)
│   ├── src/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   ├── utils/
│   │   └── app.ts
│   ├── prisma/
│   │   └── schema.prisma
│   ├── package.json
│   └── tsconfig.json
│
└── scraper-worker/                # Workerzy scrapingu (Python)
    ├── src/
    │   ├── fetchers/
    │   ├── parsers/
    │   ├── savers/
    │   ├── workers/
    │   ├── config.py
    │   └── main.py
    ├── requirements.txt
    └── Dockerfile
```

---

## 12. BEZPIECZENSTWO

### 12.1 Lista Kontrolna Bezpieczenstwa

#### Autentykacja i Autoryzacja
- [x] JWT z odpowiednim czasem zycia (access: 15min, refresh: 7dni)
- [x] Bezpieczne przechowywanie hasel (bcrypt, salt rounds 12+)
- [x] Rate limiting na logowaniu (5 prob na 15 minut)
- [x] Weryfikacja email przy rejestracji
- [x] Opcjonalne 2FA (TOTP)
- [x] Mechanizm blokady konta po probach wlamania
- [x] RBAC z granularnymi uprawnieniami

#### Ochrona Danych
- [x] Szyfrowanie polaczen (HTTPS/TLS 1.3)
- [x] Hashowanie wrazliwych danych w bazie
- [x] Sanitizacja danych wejsciowych (XSS)
- [x] Parametryzowane zapytania SQL (SQL Injection)
- [x] Walidacja typow (Zod)
- [x] CSRF protection

#### API Security
- [x] Rate limiting (100 req/min dla public, 1000 dla auth)
- [x] CORS whitelist
- [x] API keys dla zewnetrznych integracji
- [x] Request validation
- [x] Logging wszystkich requestow

#### Infrastruktura
- [x] Firewall (blokowanie niepotrzebnych portow)
- [x] Regularne aktualizacje systemu
- [x] Backup danych (szyfrowany)
- [x] Monitoring logow bezpieczenstwa
- [x] Ograniczenie dostepu SSH (klucze, nie hasla)

### 12.2 Naglowki Bezpieczenstwa

```javascript
// Express middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'", "https://api.example.com"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
}));
```

---

## 13. ZALEZNOSCI I KORELACJE

### 13.1 Diagram Zaleznosci

```
Domena Centralna (serwisy-lokalne-sterowanie.pl)
├── Zalezy od:
│   ├── PostgreSQL (schema: public)
│   ├── Redis (sesje, cache)
│   ├── RabbitMQ (kolejki zadan)
│   └── API Server (Node.js)
│
├── Wplywa na:
│   ├── Domeny Regionalne (konfiguracja)
│   ├── Scraping (zrodla, harmonogram)
│   ├── Uzytkownicy (role, uprawnienia)
│   └── Szablony (dostepne dla domen)
│
└── Korzysta z:
    ├── Zewnetrzne API (pogoda, jakosc powietrza)
    └── Email (powiadomienia)

Domena Regionalna (4torun.pl)
├── Zalezy od:
│   ├── PostgreSQL (schema: tenant_4torun_pl)
│   ├── Redis (cache)
│   ├── API Server (dane)
│   └── Domena Centralna (ustawienia)
│
├── Wplywa na:
│   ├── Domena Centralna (logi, statystyki)
│   └── Scraping (dane do importu)
│
└── Korzysta z:
    ├── Szablony (z centrali)
    ├── Zrodla danych (scraping)
    └── Zewnetrzne API (pogoda)

Scraper Worker
├── Zalezy od:
│   ├── RabbitMQ (kolejka zadan)
│   ├── PostgreSQL (schema: public + tenant)
│   └── Zrodla danych (URL-e)
│
├── Wplywa na:
│   └── Domeny Regionalne (nowe wpisy)
│
└── Korzysta z:
    ├── Zewnetrzne strony (pobieranie)
    └── Proxy (opcjonalnie)
```

### 13.2 Korrelacje Danych

| Tabela Centralna | Tabela Tenant | Relacja | Opis |
|------------------|---------------|---------|------|
| public.users | - | 1:N user_roles | Uzytkownik moze miec role w wielu domenach |
| public.domains | schema.posts | 1:N | Domena ma wiele wpisow |
| public.sources | schema.posts | 1:N | Zrodlo moze tworzyc wiele wpisow |
| public.templates | public.domains | N:1 | Wiele domen moze uzywac tego samego szablonu |
| public.cron_jobs | - | - | Niezalezne, ale moze wplywac na dane |
| schema.posts | schema.comments | 1:N | Wpis ma wiele komentarzy |
| schema.posts | schema.ratings | 1:N | Wpis ma wiele ocen |
| schema.categories | schema.posts | N:M (post_categories) | Kategorie maja wiele wpisow |
| schema.tags | schema.posts | N:M (post_tags) | Tagi maja wiele wpisow |

---

## 14. PODSUMOWANIE

### 14.1 Co Zostalo Zaprojektowane

1. **Architektura Multi-Tenant** - System pozwalajacy zarzadzac wieloma serwisami regionalnymi z jednego panelu
2. **Baza Danych** - PostgreSQL z separacja schema per tenant
3. **API** - REST API z pelna dokumentacja OpenAPI
4. **Panel Centralny** - Kompletny panel do zarzadzania wszystkimi aspektami systemu
5. **System Uprawnien** - RBAC z rolami globalnymi i per-domena
6. **Scraping** - Automatyczne pobieranie danych ze zrodel zewnetrznych
7. **SEO** - Pelna optymalizacja pod katalogi i wyszukiwarki
8. **Monitoring** - System logow i raportow

### 14.2 Kluczowe Funkcjonalnosci

- Masowe operacje na wielu domenach
- Elastyczny system szablonow
- Automatyczny scraping z wielu zrodel
- Zaawansowany system uprawnien
- Pelne wsparcie SEO
- Monitoring i raportowanie

### 14.3 Technologie Wykorzystane

- **Frontend**: Next.js 14, React, TypeScript, Tailwind CSS
- **Backend**: Node.js, Express/Fastify, TypeScript, Prisma
- **Scraping**: Python, aiohttp, BeautifulSoup
- **Baza Danych**: PostgreSQL, Redis, RabbitMQ
- **Hosting**: Wspoldzielony (zgodnie z wymaganiami)

### 14.4 Nastepne Kroki

1. Review dokumentacji
2. Szacowanie kosztow i czasu
3. Rozpoczecie implementacji (Etap 1)
4. Regularne przeglady postepow

---

## ZALACZNIKI

### A. Slownik Pojec
| Pojecie | Definicja |
|---------|-----------|
| **CPT** | Custom Post Type - niestandardowy typ wpisu |
| **RBAC** | Role-Based Access Control - kontrola dostepu oparta na rolach |
| **Tenant** | Najemca - oddzielny serwis w systemie multi-tenant |
| **Scraper** | Narzedzie do automatycznego pobierania danych |
| **Slug** | Przyjazny dla URL identyfikator (np. "tytul-wpisu") |
| **Schema** | Schemat bazy danych |
| **Sitemap** | Mapa strony w formacie XML |

### B. Linki i Zasoby
- Dokumentacja Next.js: https://nextjs.org/docs
- Dokumentacja Prisma: https://www.prisma.io/docs
- Dokumentacja PostgreSQL: https://www.postgresql.org/docs/
- BeautifulSoup: https://www.crummy.com/software/BeautifulSoup/

### C. Kontakt i Wsparcie
- Autor dokumentacji: [Twoje Imie]
- Data utworzenia: 12 lutego 2026
- Wersja: 1.0

---

**KONIEC DOKUMENTACJI**

*Dokument zawiera 14 sekcji, 60+ podsekcji, 30+ tabele, 40+ przykladow kodu.*
*Calkowity czas przygotowania: ~8h*
*Rekomendowana implementacja: 15 tygodni (3.5 miesiaca)*

## 21. STRATEGIA TESTOWANIA 🧪

### 21.1 Strategia Testowania - Test Pyramid

System Regionalne Serwisy wymaga wielopoziomowej strategii testowania zapewniającej jakość kodu, stabilność API oraz poprawne działanie interfejsów użytkownika.

```
                    /\
                   /  \
                  / E2E \          ← 10% testów (Cypress/Playwright)
                 /--------\
                /          \
               / Integration \     ← 20% testów (Supertest + TestContainers)
              /--------------\
             /                \
            /    UNIT TESTS     \   ← 70% testów (Jest)
           /____________________\
```

#### Cele Pokrycia Testami

| Typ Testu | Pokrycie Cel | Narzędzie | Priorytet |
|-----------|-------------|-----------|-----------|
| Unit Tests | >70% linii kodu | Jest | Krytyczny |
| Integration Tests | >50% endpointów API | Supertest + PostgreSQL | Wysoki |
| E2E Tests | Krytyczne ścieżki użytkownika | Cypress/Playwright | Wysoki |
| Database Tests | 100% migracji | pg-tap | Średni |
| Load Tests | Progi wydajności | k6/Artillery | Średni |
| Security Tests | OWASP Top 10 | ZAP + Snyk | Krytyczny |

#### Środowiska Testowe

```yaml
# environments.yml
environments:
  dev:
    database: "regionalne_test_dev"
    redis: "redis://localhost:6379/1"
    api_url: "http://localhost:3001"
    frontend_url: "http://localhost:3000"
    
  staging:
    database: "regionalne_test_staging"
    redis: "redis://staging-redis:6379/2"
    api_url: "https://api-staging.serwisy-lokalne.pl"
    frontend_url: "https://staging.serwisy-lokalne.pl"
    
  test:
    database: "regionalne_test_${CI_JOB_ID}"
    redis: "redis://test-redis:6379/15"
    api_url: "http://test-api:3001"
    use_testcontainers: true
```

---

### 21.2 Unit Tests

#### Konfiguracja Jest

```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.test.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
    '^@api/(.*)$': '<rootDir>/src/api/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/types/**',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
  coverageReporters: ['text', 'text-summary', 'lcov', 'html'],
};

export default config;
```

```typescript
// jest.setup.ts
import '@testing-library/jest-dom';
import { server } from './src/mocks/server';

// MSW server setup for API mocking
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Mock next/router
jest.mock('next/router', () => ({
  useRouter: () => ({
    route: '/',
    pathname: '/',
    query: {},
    asPath: '/',
    push: jest.fn(),
    replace: jest.fn(),
  }),
}));

// Mock next/image
jest.mock('next/image', () => ({
  __esModule: true,
  default: (props: any) => {
    // eslint-disable-next-line @next/next/no-img-element
    return <img {...props} alt={props.alt || ''} />;
  },
}));
```

#### Testy Utils - Normalizacja i Walidacja

```typescript
// src/utils/__tests__/normalization.test.ts
import { 
  normalizePhone, 
  normalizeSlug, 
  normalizeAddress,
  extractDistrict 
} from '../normalization';

describe('normalizePhone', () => {
  it('should normalize various phone formats', () => {
    expect(normalizePhone('123-456-789')).toBe('+48123456789');
    expect(normalizePhone('(56) 123 45 67')).toBe('+48561234567');
    expect(normalizePhone('+48 123 456 789')).toBe('+48123456789');
    expect(normalizePhone('123456789')).toBe('+48123456789');
  });

  it('should handle invalid phone numbers', () => {
    expect(normalizePhone('')).toBeNull();
    expect(normalizePhone('abc')).toBeNull();
    expect(normalizePhone('12')).toBeNull();
  });

  it('should preserve existing country code', () => {
    expect(normalizePhone('+49 123 456 789')).toBe('+49123456789');
  });
});

describe('normalizeSlug', () => {
  it('should create URL-friendly slugs', () => {
    expect(normalizeSlug('Wiadomość z miasta')).toBe('wiadomosc-z-miasta');
    expect(normalizeSlug('POLICJA - zdarzenie drogowe!')).toBe('policja-zdarzenie-drogowe');
    expect(normalizeSlug('  Multiple   spaces  ')).toBe('multiple-spaces');
  });

  it('should handle Polish characters', () => {
    expect(normalizeSlug('Łódź - miasto łodzi')).toBe('lodz-miasto-lodzi');
    expect(normalizeSlug('Żółć i źdźbło')).toBe('zolc-i-zdzblo');
  });

  it('should truncate long slugs', () => {
    const longTitle = 'a'.repeat(300);
    expect(normalizeSlug(longTitle).length).toBeLessThanOrEqual(220);
  });
});

describe('extractDistrict', () => {
  it('should extract district from address', () => {
    expect(extractDistrict('ul. Długa 1, Chełmińskie Przedmieście, Toruń'))
      .toBe('Chełmińskie Przedmieście');
    expect(extractDistrict('ul. Słowackiego 5, Bydgoszcz - Bielawy'))
      .toBe('Bielawy');
  });

  it('should return null for addresses without district', () => {
    expect(extractDistrict('ul. Długa 1, Toruń')).toBeNull();
  });
});
```

```typescript
// src/utils/__tests__/validation.test.ts
import { 
  validateNIP, 
  validateREGON, 
  validateEmail,
  validatePostalCode,
  isValidDateRange 
} from '../validation';

describe('validateNIP', () => {
  it('should validate correct NIP numbers', () => {
    expect(validateNIP('1234563218')).toBe(true);
    expect(validateNIP('876-543-21-98')).toBe(true);
  });

  it('should reject invalid NIP numbers', () => {
    expect(validateNIP('1234567890')).toBe(false);
    expect(validateNIP('')).toBe(false);
    expect(validateNIP('123')).toBe(false);
  });
});

describe('validateREGON', () => {
  it('should validate 9-digit REGON', () => {
    expect(validateREGON('123456785')).toBe(true);
  });

  it('should validate 14-digit REGON', () => {
    expect(validateREGON('12345678512347')).toBe(true);
  });

  it('should reject invalid REGON', () => {
    expect(validateREGON('123456789')).toBe(false);
  });
});

describe('isValidDateRange', () => {
  it('should validate correct date ranges', () => {
    expect(isValidDateRange('2024-01-01', '2024-12-31')).toBe(true);
    expect(isValidDateRange('2024-06-15', '2024-06-15')).toBe(true); // Same day
  });

  it('should reject invalid date ranges', () => {
    expect(isValidDateRange('2024-12-31', '2024-01-01')).toBe(false); // End before start
    expect(isValidDateRange('', '2024-12-31')).toBe(false);
  });
});
```

#### Testy API Handlers (Mocked DB)

```typescript
// src/api/handlers/__tests__/posts.test.ts
import { createMocks } from 'node-mocks-http';
import { POST, GET } from '@/app/api/posts/route';
import { prismaMock } from '@/mocks/prisma';

describe('Posts API', () => {
  describe('POST /api/posts', () => {
    it('should create a new post with valid data', async () => {
      const postData = {
        title: 'Test Post',
        content: 'Test content',
        category_id: 1,
        domain_id: 1,
      };

      prismaMock.posts.create.mockResolvedValue({
        id: 1,
        ...postData,
        slug: 'test-post',
        created_at: new Date(),
        updated_at: new Date(),
      });

      const { req } = createMocks({
        method: 'POST',
        body: postData,
      });

      const response = await POST(req);
      const data = await response.json();

      expect(response.status).toBe(201);
      expect(data.title).toBe('Test Post');
      expect(data.slug).toBe('test-post');
    });

    it('should reject post without required fields', async () => {
      const { req } = createMocks({
        method: 'POST',
        body: { title: '' },
      });

      const response = await POST(req);
      expect(response.status).toBe(400);
    });

    it('should reject unauthorized requests', async () => {
      const { req } = createMocks({
        method: 'POST',
        body: { title: 'Test' },
        headers: { authorization: '' },
      });

      const response = await POST(req);
      expect(response.status).toBe(401);
    });
  });

  describe('GET /api/posts', () => {
    it('should return paginated posts', async () => {
      const mockPosts = Array.from({ length: 10 }, (_, i) => ({
        id: i + 1,
        title: `Post ${i + 1}`,
        slug: `post-${i + 1}`,
        created_at: new Date(),
      }));

      prismaMock.posts.findMany.mockResolvedValue(mockPosts);
      prismaMock.posts.count.mockResolvedValue(25);

      const { req } = createMocks({
        method: 'GET',
        query: { page: '1', limit: '10' },
      });

      const response = await GET(req);
      const data = await response.json();

      expect(response.status).toBe(200);
      expect(data.posts).toHaveLength(10);
      expect(data.pagination.total).toBe(25);
      expect(data.pagination.totalPages).toBe(3);
    });
  });
});
```

#### Testy React Components (React Testing Library)

```typescript
// src/components/__tests__/NewsCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { NewsCard } from '@/components/NewsCard';
import { formatDistanceToNow } from 'date-fns';
import { pl } from 'date-fns/locale';

const mockPost = {
  id: 1,
  title: 'Ważna wiadomość z Torunia',
  slug: 'wazna-wiadomosc-z-torunia',
  excerpt: 'To jest krótki opis wiadomości...',
  featured_image: '/uploads/test-image.jpg',
  category: { name: 'Wiadomości', slug: 'wiadomosci' },
  published_at: new Date().toISOString(),
  author: { name: 'Jan Kowalski' },
  view_count: 150,
};

describe('NewsCard', () => {
  it('renders post information correctly', () => {
    render(<NewsCard post={mockPost} variant="default" />);
    
    expect(screen.getByText(mockPost.title)).toBeInTheDocument();
    expect(screen.getByText(mockPost.excerpt)).toBeInTheDocument();
    expect(screen.getByText(mockPost.category.name)).toBeInTheDocument();
    expect(screen.getByText(mockPost.author.name)).toBeInTheDocument();
  });

  it('displays formatted publish date', () => {
    render(<NewsCard post={mockPost} variant="default" />);
    
    const formattedDate = formatDistanceToNow(new Date(mockPost.published_at), {
      addSuffix: true,
      locale: pl,
    });
    
    expect(screen.getByText(formattedDate)).toBeInTheDocument();
  });

  it('links to correct post page', () => {
    render(<NewsCard post={mockPost} variant="default" />);
    
    const link = screen.getByRole('link');
    expect(link).toHaveAttribute('href', `/${mockPost.category.slug}/${mockPost.slug}`);
  });

  it('renders featured variant correctly', () => {
    render(<NewsCard post={mockPost} variant="featured" />);
    
    const card = screen.getByTestId('news-card');
    expect(card).toHaveClass('featured');
  });

  it('shows view count when provided', () => {
    render(<NewsCard post={mockPost} variant="default" showViews />);
    
    expect(screen.getByText('150 wyświetleń')).toBeInTheDocument();
  });
});
```

```typescript
// src/components/__tests__/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from '@/components/LoginForm';

const mockSubmit = jest.fn();

describe('LoginForm', () => {
  beforeEach(() => {
    mockSubmit.mockClear();
  });

  it('renders all form fields', () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/hasło/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /zaloguj/i })).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    const submitButton = screen.getByRole('button', { name: /zaloguj/i });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText(/email jest wymagany/i)).toBeInTheDocument();
      expect(screen.getByText(/hasło jest wymagane/i)).toBeInTheDocument();
    });
    
    expect(mockSubmit).not.toHaveBeenCalled();
  });

  it('validates email format', async () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    await userEvent.type(emailInput, 'invalid-email');
    
    fireEvent.click(screen.getByRole('button', { name: /zaloguj/i }));
    
    await waitFor(() => {
      expect(screen.getByText(/nieprawidłowy format email/i)).toBeInTheDocument();
    });
  });

  it('submits form with valid data', async () => {
    render(<LoginForm onSubmit={mockSubmit} />);
    
    await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
    await userEvent.type(screen.getByLabelText(/hasło/i), 'password123');
    
    fireEvent.click(screen.getByRole('button', { name: /zaloguj/i }));
    
    await waitFor(() => {
      expect(mockSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('shows loading state during submission', async () => {
    mockSubmit.mockImplementation(() => new Promise(() => {})); // Never resolves
    
    render(<LoginForm onSubmit={mockSubmit} />);
    
    await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
    await userEvent.type(screen.getByLabelText(/hasło/i), 'password123');
    
    fireEvent.click(screen.getByRole('button', { name: /zaloguj/i }));
    
    await waitFor(() => {
      expect(screen.getByRole('button')).toBeDisabled();
      expect(screen.getByText(/logowanie/i)).toBeInTheDocument();
    });
  });
});
```

---

### 21.3 Integration Tests

#### Konfiguracja Supertest z TestContainers

```typescript
// tests/integration/setup.ts
import { PostgreSqlContainer } from '@testcontainers/postgresql';
import { RedisContainer } from '@testcontainers/redis';
import { execSync } from 'child_process';
import { PrismaClient } from '@prisma/client';

let postgresContainer: any;
let redisContainer: any;
export let prisma: PrismaClient;

beforeAll(async () => {
  // Start PostgreSQL container
  postgresContainer = await new PostgreSqlContainer()
    .withDatabase('regionalne_test')
    .withUsername('test')
    .withPassword('test')
    .start();

  // Start Redis container
  redisContainer = await new RedisContainer().start();

  // Set environment variables
  process.env.DATABASE_URL = postgresContainer.getConnectionUri();
  process.env.REDIS_URL = redisContainer.getConnectionUrl();

  // Run migrations
  execSync('npx prisma migrate deploy', { stdio: 'inherit' });

  // Initialize Prisma
  prisma = new PrismaClient();
}, 120000);

afterAll(async () => {
  await prisma?.$disconnect();
  await postgresContainer?.stop();
  await redisContainer?.stop();
});

beforeEach(async () => {
  // Clean database before each test
  const tables = await prisma.$queryRaw`
    SELECT tablename FROM pg_tables WHERE schemaname = 'public'
  `;
  
  for (const { tablename } of tables as any[]) {
    if (tablename !== '_prisma_migrations') {
      await prisma.$executeRawUnsafe(`TRUNCATE TABLE "${tablename}" CASCADE`);
    }
  }
});
```

```typescript
// tests/integration/api/posts.integration.test.ts
import request from 'supertest';
import { app } from '@/app';
import { prisma } from '../setup';
import { createTestUser, createTestDomain, authToken } from '../helpers';

describe('Posts API Integration', () => {
  let user: any;
  let domain: any;
  let token: string;

  beforeEach(async () => {
    user = await createTestUser({ role: 'editor' });
    domain = await createTestDomain();
    token = authToken(user);
  });

  describe('POST /api/posts', () => {
    it('should create post and store in database', async () => {
      const postData = {
        title: 'Integration Test Post',
        content: 'Content for integration test',
        category_id: 1,
        domain_id: domain.id,
      };

      const response = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${token}`)
        .send(postData)
        .expect(201);

      // Verify response
      expect(response.body.title).toBe(postData.title);
      expect(response.body.slug).toBe('integration-test-post');

      // Verify database
      const dbPost = await prisma.posts.findUnique({
        where: { id: response.body.id },
      });
      expect(dbPost).toBeTruthy();
      expect(dbPost?.title).toBe(postData.title);
    });

    it('should handle concurrent post creation', async () => {
      const postData = {
        title: 'Concurrent Post',
        content: 'Content',
        category_id: 1,
        domain_id: domain.id,
      };

      // Create 5 posts simultaneously
      const promises = Array.from({ length: 5 }, (_, i) =>
        request(app)
          .post('/api/posts')
          .set('Authorization', `Bearer ${token}`)
          .send({ ...postData, title: `Concurrent Post ${i}` })
      );

      const responses = await Promise.all(promises);
      
      // All should succeed
      responses.forEach((res, i) => {
        expect(res.status).toBe(201);
        expect(res.body.title).toBe(`Concurrent Post ${i}`);
      });

      // Verify unique slugs were generated
      const slugs = responses.map(r => r.body.slug);
      expect(new Set(slugs).size).toBe(5);
    });

    it('should rollback on validation error', async () => {
      const initialCount = await prisma.posts.count();

      await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${token}`)
        .send({ title: '', content: 'Invalid' })
        .expect(400);

      const finalCount = await prisma.posts.count();
      expect(finalCount).toBe(initialCount);
    });
  });

  describe('GET /api/posts', () => {
    beforeEach(async () => {
      // Seed test data
      await prisma.posts.createMany({
        data: Array.from({ length: 25 }, (_, i) => ({
          title: `Post ${i + 1}`,
          slug: `post-${i + 1}`,
          content: `Content ${i + 1}`,
          domain_id: domain.id,
          category_id: 1,
          published_at: new Date(),
        })),
      });
    });

    it('should return paginated results', async () => {
      const response = await request(app)
        .get('/api/posts?page=1&limit=10')
        .expect(200);

      expect(response.body.posts).toHaveLength(10);
      expect(response.body.pagination).toEqual({
        page: 1,
        limit: 10,
        total: 25,
        totalPages: 3,
      });
    });

    it('should filter by domain', async () => {
      const otherDomain = await createTestDomain({ name: 'Other' });
      
      await prisma.posts.create({
        data: {
          title: 'Other Domain Post',
          slug: 'other-domain-post',
          content: 'Other content',
          domain_id: otherDomain.id,
          category_id: 1,
        },
      });

      const response = await request(app)
        .get(`/api/posts?domain_id=${domain.id}`)
        .expect(200);

      expect(response.body.posts.every((p: any) => p.domain_id === domain.id)).toBe(true);
    });
  });
});
```

#### Testy Przepływów (Scraping → DB)

```typescript
// tests/integration/scraping.integration.test.ts
import request from 'supertest';
import { app } from '@/app';
import { prisma } from './setup';

describe('Scraping Integration Flow', () => {
  it('should scrape, parse and save content to database', async () => {
    // 1. Create scraper configuration
    const scraperConfig = await prisma.scraper_configs.create({
      data: {
        name: 'Test Police Scraper',
        source_type: 'POLICE_RSS',
        source_url: 'https://test.police.gov.pl/feed',
        domain_id: 1,
        parser_config: {
          selectors: {
            title: 'h1.post-title',
            content: 'div.post-content',
            date: 'span.publish-date',
          },
        },
        is_active: true,
      },
    });

    // 2. Trigger scraping (mock external API)
    const response = await request(app)
      .post('/api/scraper/run')
      .send({ config_id: scraperConfig.id })
      .expect(200);

    // 3. Verify scraping job was created
    const job = await prisma.scraper_jobs.findFirst({
      where: { config_id: scraperConfig.id },
    });
    expect(job).toBeTruthy();
    expect(job?.status).toBe('completed');

    // 4. Verify normalized content was created
    const posts = await prisma.posts.findMany({
      where: {
        metadata: {
          path: ['source', 'scraper_id'],
          equals: scraperConfig.id,
        },
      },
    });
    expect(posts.length).toBeGreaterThan(0);

    // 5. Verify audit log
    const auditLog = await prisma.audit_logs.findFirst({
      where: { entity_type: 'scraper_job' },
      orderBy: { created_at: 'desc' },
    });
    expect(auditLog).toBeTruthy();
  });

  it('should handle duplicate content detection', async () => {
    // Create existing post
    await prisma.posts.create({
      data: {
        title: 'Existing Post',
        slug: 'existing-post',
        content: 'Existing content',
        domain_id: 1,
        category_id: 1,
        source_url: 'https://test.police.gov.pl/post/123',
      },
    });

    // Try to scrape same content
    const response = await request(app)
      .post('/api/scraper/run')
      .send({ config_id: 1 })
      .expect(200);

    // Verify deduplication worked
    const posts = await prisma.posts.findMany({
      where: { source_url: 'https://test.police.gov.pl/post/123' },
    });
    expect(posts).toHaveLength(1);
  });
});
```

---

### 21.4 E2E Tests

#### Konfiguracja Cypress

```typescript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    supportFile: 'cypress/support/e2e.ts',
    specPattern: 'cypress/e2e/**/*.cy.ts',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    requestTimeout: 10000,
    responseTimeout: 10000,
    env: {
      apiUrl: 'http://localhost:3001',
      adminEmail: 'admin@test.com',
      adminPassword: 'TestPassword123!',
    },
    setupNodeEvents(on, config) {
      // Database seeding task
      on('task', {
        async seedDatabase() {
          // Seed test data via API or direct DB connection
          const { seed } = require('./cypress/support/seed');
          return await seed();
        },
        async cleanupDatabase() {
          const { cleanup } = require('./cypress/support/seed');
          return await cleanup();
        },
      });
      return config;
    },
  },
  component: {
    devServer: {
      framework: 'next',
      bundler: 'webpack',
    },
  },
});
```

```typescript
// cypress/support/e2e.ts
import './commands';

// Global beforeEach - seed database
deforeEach(() => {
  cy.task('seedDatabase');
});

// Global afterEach - cleanup
afterEach(() => {
  cy.task('cleanupDatabase');
});

// Handle uncaught exceptions
Cypress.on('uncaught:exception', (err) => {
  // Return false to prevent Cypress from failing the test
  if (err.message.includes('ResizeObserver')) {
    return false;
  }
});
```

```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      loginAsAdmin(): Chainable<void>;
      createPost(postData: any): Chainable<void>;
      createDomain(domainData: any): Chainable<void>;
      getByTestId(testId: string): Chainable<JQuery<HTMLElement>>;
    }
  }
}

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.getByTestId('email-input').type(email);
    cy.getByTestId('password-input').type(password);
    cy.getByTestId('login-submit').click();
    cy.url().should('not.include', '/login');
  });
});

Cypress.Commands.add('loginAsAdmin', () => {
  cy.login(Cypress.env('adminEmail'), Cypress.env('adminPassword'));
});

Cypress.Commands.add('createPost', (postData: any) => {
  cy.request({
    method: 'POST',
    url: `${Cypress.env('apiUrl')}/api/posts`,
    headers: {
      Authorization: `Bearer ${window.localStorage.getItem('token')}`,
    },
    body: postData,
  });
});

Cypress.Commands.add('createDomain', (domainData: any) => {
  cy.request({
    method: 'POST',
    url: `${Cypress.env('apiUrl')}/api/domains`,
    headers: {
      Authorization: `Bearer ${window.localStorage.getItem('token')}`,
    },
    body: domainData,
  });
});

Cypress.Commands.add('getByTestId', (testId: string) => {
  return cy.get(`[data-testid="${testId}"]`);
});
```

#### Scenariusze E2E

```typescript
// cypress/e2e/auth/login.cy.ts
describe('Authentication', () => {
  beforeEach(() => {
    cy.visit('/login');
  });

  it('should login with valid credentials', () => {
    cy.getByTestId('email-input').type('admin@test.com');
    cy.getByTestId('password-input').type('TestPassword123!');
    cy.getByTestId('login-submit').click();

    cy.url().should('include', '/dashboard');
    cy.getByTestId('user-menu').should('be.visible');
  });

  it('should show error for invalid credentials', () => {
    cy.getByTestId('email-input').type('wrong@email.com');
    cy.getByTestId('password-input').type('wrongpassword');
    cy.getByTestId('login-submit').click();

    cy.getByTestId('login-error')
      .should('be.visible')
      .and('contain', 'Nieprawidłowe dane logowania');
  });

  it('should validate required fields', () => {
    cy.getByTestId('login-submit').click();

    cy.getByTestId('email-error')
      .should('be.visible')
      .and('contain', 'Email jest wymagany');
    cy.getByTestId('password-error')
      .should('be.visible')
      .and('contain', 'Hasło jest wymagane');
  });

  it('should persist session after page reload', () => {
    cy.loginAsAdmin();
    cy.visit('/dashboard');
    cy.reload();
    cy.getByTestId('user-menu').should('be.visible');
  });

  it('should logout successfully', () => {
    cy.loginAsAdmin();
    cy.getByTestId('user-menu').click();
    cy.getByTestId('logout-button').click();

    cy.url().should('include', '/login');
    cy.visit('/dashboard');
    cy.url().should('include', '/login');
  });
});
```

```typescript
// cypress/e2e/posts/create-post.cy.ts
describe('Create Post', () => {
  beforeEach(() => {
    cy.loginAsAdmin();
    cy.visit('/admin/posts/new');
  });

  it('should create a new post', () => {
    const postTitle = 'Test Post ' + Date.now();

    cy.getByTestId('post-title').type(postTitle);
    cy.getByTestId('post-category').select('Wiadomości');
    cy.getByTestId('post-content').type('This is test content for the post.');
    
    // Upload featured image
    cy.getByTestId('image-upload').selectFile('cypress/fixtures/test-image.jpg');
    cy.getByTestId('upload-progress').should('not.exist');

    cy.getByTestId('save-draft').click();

    cy.getByTestId('success-message')
      .should('be.visible')
      .and('contain', 'Wpis zapisany');

    // Verify post appears in list
    cy.visit('/admin/posts');
    cy.getByTestId('posts-table').should('contain', postTitle);
  });

  it('should validate required fields', () => {
    cy.getByTestId('publish-post').click();

    cy.getByTestId('title-error')
      .should('be.visible')
      .and('contain', 'Tytuł jest wymagany');
    cy.getByTestId('category-error')
      .should('be.visible')
      .and('contain', 'Kategoria jest wymagana');
  });

  it('should auto-generate slug from title', () => {
    cy.getByTestId('post-title').type('Test Post Title');
    cy.getByTestId('post-slug').should('have.value', 'test-post-title');
  });

  it('should schedule post for future date', () => {
    cy.getByTestId('post-title').type('Scheduled Post');
    cy.getByTestId('post-category').select('Wiadomości');
    cy.getByTestId('publish-date').type('2030-01-01T12:00');
    cy.getByTestId('schedule-post').click();

    cy.getByTestId('status-badge').should('contain', 'Zaplanowany');
  });
});
```

```typescript
// cypress/e2e/domains/create-domain.cy.ts
describe('Create Domain', () => {
  beforeEach(() => {
    cy.loginAsAdmin();
    cy.visit('/admin/domains/new');
  });

  it('should create a new domain', () => {
    const domainName = 'testcity' + Date.now();

    cy.getByTestId('domain-name').type(domainName);
    cy.getByTestId('domain-slug').should('have.value', domainName);
    cy.getByTestId('domain-title').type('Test City Portal');
    cy.getByTestId('domain-city').type('Test City');
    cy.getByTestId('domain-region').select('kujawsko-pomorskie');
    
    // Configure theme
    cy.getByTestId('primary-color').clear().type('#ff0000');
    cy.getByTestId('secondary-color').clear().type('#00ff00');

    cy.getByTestId('create-domain').click();

    cy.url().should('include', '/admin/domains');
    cy.getByTestId('domains-table').should('contain', domainName);
  });

  it('should validate unique domain name', () => {
    cy.createDomain({ name: 'existing', title: 'Existing' });
    
    cy.visit('/admin/domains/new');
    cy.getByTestId('domain-name').type('existing');
    cy.getByTestId('create-domain').click();

    cy.getByTestId('name-error')
      .should('be.visible')
      .and('contain', 'Domena o tej nazwie już istnieje');
  });
});
```

```typescript
// cypress/e2e/scraping/scraping.cy.ts
describe('Scraping and Display', () => {
  beforeEach(() => {
    cy.loginAsAdmin();
  });

  it('should configure and run scraper', () => {
    cy.visit('/admin/scrapers/new');

    // Configure scraper
    cy.getByTestId('scraper-name').type('Test Police Scraper');
    cy.getByTestId('scraper-type').select('POLICE_RSS');
    cy.getByTestId('source-url').type('https://policja.gov.pl/rss');
    cy.getByTestId('target-domain').select('4torun.pl');

    // Add parser rules
    cy.getByTestId('add-selector').click();
    cy.getByTestId('selector-name-0').type('title');
    cy.getByTestId('selector-value-0').type('h1.article-title');

    cy.getByTestId('save-scraper').click();
    cy.getByTestId('success-message').should('be.visible');

    // Run scraper
    cy.getByTestId('run-scraper').click();
    cy.getByTestId('scraping-status').should('contain', 'Trwa scrapowanie');
    cy.getByTestId('scraping-status', { timeout: 30000 }).should('contain', 'Zakończono');

    // Verify scraped content
    cy.visit('/admin/content/pending');
    cy.getByTestId('pending-list').should('have.length.at.least', 1);
  });

  it('should display scraped content on frontend', () => {
    // Create test post via API
    cy.createPost({
      title: 'Scraped News',
      content: 'Content from scraping',
      category: 'news',
      status: 'published',
    });

    cy.visit('/');
    cy.getByTestId('news-section').should('contain', 'Scraped News');
    
    cy.getByTestId('news-card').first().click();
    cy.url().should('include', '/news/');
    cy.getByTestId('article-content').should('contain', 'Content from scraping');
  });
});
```

#### Page Object Model

```typescript
// cypress/pages/BasePage.ts
export abstract class BasePage {
  protected abstract url: string;

  visit(): void {
    cy.visit(this.url);
  }

  getElement(testId: string): Cypress.Chainable<JQuery<HTMLElement>> {
    return cy.getByTestId(testId);
  }

  waitForLoading(): void {
    cy.getByTestId('loading-spinner').should('not.exist');
  }
}

// cypress/pages/LoginPage.ts
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  protected url = '/login';

  elements = {
    emailInput: () => this.getElement('email-input'),
    passwordInput: () => this.getElement('password-input'),
    submitButton: () => this.getElement('login-submit'),
    errorMessage: () => this.getElement('login-error'),
  };

  login(email: string, password: string): void {
    this.elements.emailInput().type(email);
    this.elements.passwordInput().type(password);
    this.elements.submitButton().click();
  }

  assertLoginError(message: string): void {
    this.elements.errorMessage()
      .should('be.visible')
      .and('contain', message);
  }
}

// cypress/pages/DashboardPage.ts
import { BasePage } from './BasePage';

export class DashboardPage extends BasePage {
  protected url = '/dashboard';

  elements = {
    userMenu: () => this.getElement('user-menu'),
    postsLink: () => this.getElement('nav-posts'),
    domainsLink: () => this.getElement('nav-domains'),
    statsCards: () => this.getElement('stats-cards'),
  };

  navigateToPosts(): void {
    this.elements.postsLink().click();
  }

  navigateToDomains(): void {
    this.elements.domainsLink().click();
  }

  assertLoggedIn(): void {
    this.elements.userMenu().should('be.visible');
  }
}

// cypress/pages/PostEditorPage.ts
import { BasePage } from './BasePage';

export class PostEditorPage extends BasePage {
  protected url = '/admin/posts/new';

  elements = {
    titleInput: () => this.getElement('post-title'),
    slugInput: () => this.getElement('post-slug'),
    categorySelect: () => this.getElement('post-category'),
    contentEditor: () => this.getElement('post-content'),
    imageUpload: () => this.getElement('image-upload'),
    saveDraftButton: () => this.getElement('save-draft'),
    publishButton: () => this.getElement('publish-post'),
    successMessage: () => this.getElement('success-message'),
  };

  fillPost(data: {
    title: string;
    category: string;
    content: string;
    image?: string;
  }): void {
    this.elements.titleInput().type(data.title);
    this.elements.categorySelect().select(data.category);
    this.elements.contentEditor().type(data.content);
    
    if (data.image) {
      this.elements.imageUpload().selectFile(data.image);
    }
  }

  saveAsDraft(): void {
    this.elements.saveDraftButton().click();
  }

  publish(): void {
    this.elements.publishButton().click();
  }

  assertSuccess(message: string): void {
    this.elements.successMessage()
      .should('be.visible')
      .and('contain', message);
  }
}

// Usage in tests
describe('Using Page Objects', () => {
  const loginPage = new LoginPage();
  const dashboardPage = new DashboardPage();
  const postEditorPage = new PostEditorPage();

  beforeEach(() => {
    loginPage.visit();
    loginPage.login('admin@test.com', 'TestPassword123!');
    dashboardPage.assertLoggedIn();
  });

  it('should create post using page objects', () => {
    dashboardPage.navigateToPosts();
    postEditorPage.visit();
    postEditorPage.fillPost({
      title: 'Page Object Test',
      category: 'Wiadomości',
      content: 'Content created with page object pattern',
    });
    postEditorPage.saveAsDraft();
    postEditorPage.assertSuccess('Wpis zapisany');
  });
});
```

---

### 21.5 Database Tests

#### Testy Migracji

```sql
-- tests/database/migrations.test.sql
-- Test: Verify all migrations are applied
SELECT COUNT(*) as migration_count 
FROM _prisma_migrations 
WHERE finished_at IS NOT NULL;

-- Expected: migration_count >= expected_count

-- Test: Verify schema consistency
SELECT 
  table_name,
  column_name,
  data_type,
  is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;

-- Test: Verify foreign key constraints
SELECT
  tc.constraint_name,
  tc.table_name,
  kcu.column_name,
  ccu.table_name AS foreign_table_name,
  ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

```typescript
// tests/database/migrations.test.ts
import { PostgreSqlContainer } from '@testcontainers/postgresql';
import { execSync } from 'child_process';
import { PrismaClient } from '@prisma/client';

describe('Database Migrations', () => {
  let container: any;
  let prisma: PrismaClient;

  beforeAll(async () => {
    container = await new PostgreSqlContainer().start();
    process.env.DATABASE_URL = container.getConnectionUri();
  }, 60000);

  afterAll(async () => {
    await container?.stop();
  });

  beforeEach(() => {
    prisma = new PrismaClient({
      datasources: {
        db: { url: container.getConnectionUri() },
      },
    });
  });

  afterEach(async () => {
    await prisma.$disconnect();
  });

  it('should apply all migrations successfully', () => {
    expect(() => {
      execSync('npx prisma migrate deploy', { stdio: 'pipe' });
    }).not.toThrow();
  });

  it('should have all required tables', async () => {
    execSync('npx prisma migrate deploy', { stdio: 'pipe' });

    const tables = await prisma.$queryRaw`
      SELECT table_name 
      FROM information_schema.tables 
      WHERE table_schema = 'public'
    `;

    const tableNames = (tables as any[]).map(t => t.table_name);
    
    const requiredTables = [
      'domains',
      'users',
      'posts',
      'categories',
      'scraper_configs',
      'audit_logs',
    ];

    requiredTables.forEach(table => {
      expect(tableNames).toContain(table);
    });
  });

  it('should rollback migrations cleanly', () => {
    // Apply migrations
    execSync('npx prisma migrate deploy', { stdio: 'pipe' });
    
    // Rollback last migration
    expect(() => {
      execSync('npx prisma migrate resolve --rolled-back "migration_name"', { 
        stdio: 'pipe' 
      });
    }).not.toThrow();
  });
});
```

#### Testy Constraintów

```typescript
// tests/database/constraints.test.ts
import { prisma } from './setup';

describe('Database Constraints', () => {
  describe('Unique Constraints', () => {
    it('should enforce unique domain names', async () => {
      await prisma.domains.create({
        data: { name: 'unique-domain', title: 'Test' },
      });

      await expect(
        prisma.domains.create({
          data: { name: 'unique-domain', title: 'Test 2' },
        })
      ).rejects.toThrow(/unique constraint/);
    });

    it('should enforce unique slugs per domain', async () => {
      const domain = await prisma.domains.create({
        data: { name: 'test-slug', title: 'Test' },
      });

      await prisma.posts.create({
        data: {
          title: 'Post 1',
          slug: 'test-post',
          domain_id: domain.id,
          category_id: 1,
        },
      });

      await expect(
        prisma.posts.create({
          data: {
            title: 'Post 2',
            slug: 'test-post',
            domain_id: domain.id,
            category_id: 1,
          },
        })
      ).rejects.toThrow(/unique constraint/);
    });
  });

  describe('Foreign Key Constraints', () => {
    it('should prevent deleting domain with posts', async () => {
      const domain = await prisma.domains.create({
        data: { name: 'domain-with-posts', title: 'Test' },
      });

      await prisma.posts.create({
        data: {
          title: 'Test Post',
          slug: 'test-post',
          domain_id: domain.id,
          category_id: 1,
        },
      });

      await expect(
        prisma.domains.delete({ where: { id: domain.id } })
      ).rejects.toThrow(/foreign key constraint/);
    });

    it('should cascade delete related records', async () => {
      const domain = await prisma.domains.create({
        data: { name: 'cascade-test', title: 'Test' },
      });

      await prisma.domain_settings.create({
        data: {
          domain_id: domain.id,
          primary_color: '#000000',
        },
      });

      await prisma.domains.delete({
        where: { id: domain.id },
      });

      const settings = await prisma.domain_settings.findFirst({
        where: { domain_id: domain.id },
      });

      expect(settings).toBeNull();
    });
  });

  describe('Check Constraints', () => {
    it('should enforce valid email format', async () => {
      await expect(
        prisma.users.create({
          data: {
            email: 'invalid-email',
            password_hash: 'hash',
          },
        })
      ).rejects.toThrow();
    });

    it('should enforce positive prices', async () => {
      await expect(
        prisma.classifieds.create({
          data: {
            title: 'Test',
            price: -100,
            domain_id: 1,
          },
        })
      ).rejects.toThrow(/check constraint/);
    });
  });
});
```

#### Seed Data dla Testów

```typescript
// tests/database/seeds/test-data.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function seedTestData() {
  // Create test domain
  const domain = await prisma.domains.create({
    data: {
      name: 'test-domain',
      title: 'Test Domain',
      slug: 'test-domain',
      city: 'Test City',
      region: 'test-region',
      settings: {
        create: {
          primary_color: '#3b82f6',
          secondary_color: '#10b981',
        },
      },
    },
  });

  // Create categories
  const categories = await prisma.categories.createMany({
    data: [
      { name: 'Wiadomości', slug: 'wiadomosci', domain_id: domain.id },
      { name: 'Sport', slug: 'sport', domain_id: domain.id },
      { name: 'Kultura', slug: 'kultura', domain_id: domain.id },
    ],
  });

  // Create test users with different roles
  const users = await prisma.users.createMany({
    data: [
      {
        email: 'admin@test.com',
        password_hash: '$2b$10$...', // hashed
        role: 'super_admin',
      },
      {
        email: 'editor@test.com',
        password_hash: '$2b$10$...',
        role: 'editor',
      },
      {
        email: 'moderator@test.com',
        password_hash: '$2b$10$...',
        role: 'moderator',
      },
    ],
  });

  // Create test posts
  const posts = await prisma.posts.createMany({
    data: Array.from({ length: 20 }, (_, i) => ({
      title: `Test Post ${i + 1}`,
      slug: `test-post-${i + 1}`,
      content: `Content for test post ${i + 1}`,
      excerpt: `Excerpt ${i + 1}`,
      domain_id: domain.id,
      category_id: 1,
      status: i % 5 === 0 ? 'draft' : 'published',
      published_at: i % 5 === 0 ? null : new Date(),
    })),
  });

  // Create scraper configs
  const scrapers = await prisma.scraper_configs.createMany({
    data: [
      {
        name: 'Police RSS',
        source_type: 'POLICE_RSS',
        source_url: 'https://policja.gov.pl/rss',
        domain_id: domain.id,
        is_active: true,
      },
      {
        name: 'City News',
        source_type: 'CITY_FEED',
        source_url: 'https://city.gov.pl/feed',
        domain_id: domain.id,
        is_active: true,
      },
    ],
  });

  return { domain, categories, users, posts, scrapers };
}

export async function cleanupTestData() {
  const tables = [
    'audit_logs',
    'scraper_jobs',
    'scraper_configs',
    'posts',
    'categories',
    'domain_settings',
    'domain_users',
    'users',
    'domains',
  ];

  for (const table of tables) {
    await prisma.$executeRawUnsafe(`TRUNCATE TABLE "${table}" CASCADE`);
  }
}
```

---

### 21.6 Load Testing

#### Konfiguracja k6

```typescript
// load-tests/config/options.js
export const options = {
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 200 },   // Ramp up to 200 users
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],   // 95% requests under 500ms
    http_req_failed: ['rate<0.1'],      // Less than 0.1% errors
  },
};

// load-tests/config/constants.js
export const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';
export const API_URL = __ENV.API_URL || 'http://localhost:3001';

export const ENDPOINTS = {
  home: '/',
  news: '/wiadomosci',
  apiPosts: '/api/posts',
  apiCategories: '/api/categories',
  login: '/api/auth/login',
};

export const USERS = [
  { email: 'loadtest1@test.com', password: 'Test123!' },
  { email: 'loadtest2@test.com', password: 'Test123!' },
  { email: 'loadtest3@test.com', password: 'Test123!' },
];
```

```javascript
// load-tests/scenarios/frontend-load.js
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Rate, Trend } from 'k6/metrics';
import { options, BASE_URL, ENDPOINTS } from '../config/options';

const errorRate = new Rate('errors');
const pageLoadTime = new Trend('page_load_time');

export { options };

export default function () {
  group('Homepage', () => {
    const start = Date.now();
    const response = http.get(`${BASE_URL}${ENDPOINTS.home}`);
    const duration = Date.now() - start;
    
    pageLoadTime.add(duration);
    
    const success = check(response, {
      'homepage status is 200': (r) => r.status === 200,
      'homepage loads under 1s': (r) => duration < 1000,
      'homepage has content': (r) => r.body.includes('Regionalny'),
    });
    
    errorRate.add(!success);
  });

  group('News Page', () => {
    const response = http.get(`${BASE_URL}${ENDPOINTS.news}`);
    
    const success = check(response, {
      'news page status is 200': (r) => r.status === 200,
      'news page has articles': (r) => r.body.includes('article'),
    });
    
    errorRate.add(!success);
  });

  group('API - Posts List', () => {
    const response = http.get(`${BASE_URL}${ENDPOINTS.apiPosts}?page=1&limit=10`);
    
    const success = check(response, {
      'posts API status is 200': (r) => r.status === 200,
      'posts API returns JSON': (r) => r.headers['Content-Type'].includes('json'),
      'posts API has data': (r) => JSON.parse(r.body).posts !== undefined,
    });
    
    errorRate.add(!success);
  });

  sleep(Math.random() * 3 + 1); // Random sleep 1-4 seconds
}
```

```javascript
// load-tests/scenarios/api-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';
import { API_URL, ENDPOINTS, USERS } from '../config/constants';

const errorRate = new Rate('errors');

export const options = {
  scenarios: {
    constant_request_rate: {
      executor: 'constant-arrival-rate',
      rate: 1000,                          // 1000 iterations per minute
      timeUnit: '1m',
      duration: '10m',
      preAllocatedVUs: 100,
      maxVUs: 200,
    },
  },
  thresholds: {
    http_req_duration: ['p(99)<1000'],     // 99% under 1s
    errors: ['rate<0.05'],                 // Less than 5% errors
  },
};

let authToken = null;

export function setup() {
  // Login and get token
  const loginRes = http.post(`${API_URL}${ENDPOINTS.login}`, {
    email: USERS[0].email,
    password: USERS[0].password,
  });
  
  if (loginRes.status === 200) {
    authToken = JSON.parse(loginRes.body).token;
  }
  
  return { token: authToken };
}

export default function (data) {
  const headers = {
    'Content-Type': 'application/json',
  };
  
  if (data.token) {
    headers['Authorization'] = `Bearer ${data.token}`;
  }

  // GET posts
  const getRes = http.get(`${API_URL}${ENDPOINTS.apiPosts}?page=1&limit=20`, { headers });
  
  check(getRes, {
    'GET posts status 200': (r) => r.status === 200,
    'GET posts response time < 200ms': (r) => r.timings.duration < 200,
  });

  // GET categories
  const catRes = http.get(`${API_URL}${ENDPOINTS.apiCategories}`, { headers });
  
  check(catRes, {
    'GET categories status 200': (r) => r.status === 200,
  });

  sleep(0.1);
}
```

```javascript
// load-tests/scenarios/scraping-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';
import { API_URL } from '../config/constants';

const rateLimitHits = new Rate('rate_limit_hits');

export const options = {
  stages: [
    { duration: '1m', target: 10 },     // 10 concurrent scraping jobs
    { duration: '5m', target: 10 },
    { duration: '1m', target: 20 },     // Increase to 20
    { duration: '5m', target: 20 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {
    rate_limit_hits: ['rate<0.1'],       // Less than 10% rate limited
  },
};

export default function () {
  // Simulate multiple scraper requests
  const scraperConfigs = [1, 2, 3, 4, 5];
  
  for (const configId of scraperConfigs) {
    const response = http.post(`${API_URL}/api/scraper/run`, {
      config_id: configId,
    });
    
    // Check if rate limited (429)
    if (response.status === 429) {
      rateLimitHits.add(1);
    }
    
    check(response, {
      'scraper accepts or rate limits': (r) => r.status === 200 || r.status === 429,
    });
    
    sleep(1); // Wait between requests
  }
  
  sleep(5); // Wait between cycles
}
```

#### Konfiguracja Artillery

```yaml
# load-tests/artillery/frontend.yml
config:
  target: 'http://localhost:3000'
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 120
      arrivalRate: 10
      rampTo: 50
      name: "Ramp up"
    - duration: 300
      arrivalRate: 50
      name: "Sustained load"
  plugins:
    metrics-by-endpoint:
      useOnlyRequestNames: true

scenarios:
  - name: "Browse homepage"
    weight: 40
    requests:
      - get:
          url: "/"
          name: "homepage"
          
  - name: "Browse news"
    weight: 30
    requests:
      - get:
          url: "/wiadomosci"
          name: "news_list"
      - get:
          url: "/wiadomosci?page={{ $randomInt(1, 5) }}"
          name: "news_paginated"
          
  - name: "Read article"
    weight: 20
    requests:
      - get:
          url: "/wiadomosci/sample-post-{{ $randomInt(1, 100) }}"
          name: "article_detail"
          
  - name: "API calls"
    weight: 10
    requests:
      - get:
          url: "/api/posts?limit=10"
          name: "api_posts"
      - get:
          url: "/api/categories"
          name: "api_categories"
```

---

### 21.7 Security Tests

#### OWASP ZAP

```yaml
# security-tests/zap/zap-baseline.yml
# ZAP Baseline Scan Configuration

spider:
  maxDepth: 5
  threadCount: 4

scanners:
  - id: 40012
    name: "Cross Domain Script Inclusion"
    enabled: true
  - id: 40014
    name: "Cross Domain Misconfiguration"
    enabled: true
  - id: 40016
    name: "Web Browser XSS Protection Not Enabled"
    enabled: true
  - id: 40017
    name: "Cross Domain JavaScript Source File Included"
    enabled: true
  - id: 40018
    name: "SQL Injection"
    enabled: true
  - id: 40019
    name: "SQL Injection MySQL"
    enabled: true
  - id: 40020
    name: "SQL Injection Hypersonic"
    enabled: true
  - id: 40021
    name: "SQL Injection Oracle"
    enabled: true
  - id: 40022
    name: "SQL Injection PostgreSQL"
    enabled: true
  - id: 40024
    name: "SQL Injection SQLite"
    enabled: true
  - id: 40026
    name: "Cross Site Scripting (DOM Based)"
    enabled: true
  - id: 40027
    name: "Cross Site Scripting (Reflected)"
    enabled: true
  - id: 40028
    name: "ELMAH Information Leak"
    enabled: true
  - id: 40029
    name: "Trace.axd Information Leak"
    enabled: true
  - id: 40032
    name: ".htaccess Information Leak"
    enabled: true

# Ignore certain alerts
ignore:
  - ruleId: 10027
    reason: "Informational - Suspicious comments"
  - ruleId: 10096
    reason: "Informational - Timestamp disclosure"

# Context for authentication
context:
  name: "Regionalne Serwisy"
  includePaths:
    - "http://localhost:3000.*"
    - "http://localhost:3001.*"
  excludePaths:
    - ".*\\.js"
    - ".*\\.css"
    - ".*\\.png"
    - ".*\\.jpg"
```

```bash
#!/bin/bash
# security-tests/zap/run-zap-scan.sh

TARGET_URL=${1:-"http://localhost:3000"}
API_URL=${2:-"http://localhost:3001"}
REPORT_DIR="reports"

echo "Starting OWASP ZAP Baseline Scan..."
echo "Target: $TARGET_URL"
echo "API: $API_URL"

mkdir -p $REPORT_DIR

# Run baseline scan on frontend
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t $TARGET_URL \
  -c security-tests/zap/zap-baseline.yml \
  -r $REPORT_DIR/zap-report-frontend.html \
  -w $REPORT_DIR/zap-report-frontend.md

# Run API scan on backend
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
  -t $API_URL/openapi.json \
  -f openapi \
  -c security-tests/zap/zap-baseline.yml \
  -r $REPORT_DIR/zap-report-api.html \
  -w $REPORT_DIR/zap-report-api.md

# Run full scan (takes longer, more thorough)
# docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
#   -t $TARGET_URL \
#   -r $REPORT_DIR/zap-full-report.html

echo "Scan complete. Reports saved to $REPORT_DIR/"
```

#### npm audit

```json
// package.json audit configuration
{
  "scripts": {
    "security:audit": "npm audit --audit-level=moderate",
    "security:audit:fix": "npm audit fix",
    "security:audit:ci": "npm audit --audit-level=moderate --production",
    "security:outdated": "npm outdated",
    "security:check": "npm-run-all security:audit security:outdated"
  }
}
```

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday at 2 AM
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        continue-on-error: true
        
      - name: Check for outdated packages
        run: npm outdated || true
        
      - name: Upload audit report
        uses: actions/upload-artifact@v4
        with:
          name: audit-report
          path: audit-report.json
```

#### Snyk Integration

```yaml
# .snyk - Snyk policy file
version: v1.25.0
ignore:
  SNYK-JS-LODASH-1018905:
    - '*':
        reason: 'No patch available, not exploitable in our context'
        expires: '2024-12-31T00:00:00.000Z'
  SNYK-JS-AXIOS-1038255:
    - '*':
        reason: 'Fixed in code review'
        expires: '2024-12-31T00:00:00.000Z'
patch: {}
```

```yaml
# .github/workflows/snyk.yml
name: Snyk Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 3 * * *'  # Daily at 3 AM

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --sarif-file-output=snyk.sarif
          
      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: snyk.sarif
```

#### Security Testy Manualne

```typescript
// security-tests/manual/security-checklist.ts
/**
 * Security Testing Checklist
 * 
 * Run these tests manually or automate via Cypress/Selenium
 */

export const securityTests = {
  authentication: [
    'Test login with SQL injection attempts: admin\' OR \'1\'=\'1',
    'Test login with XSS in email: <script>alert(1)</script>@test.com',
    'Verify password complexity requirements',
    'Test brute force protection (rate limiting)',
    'Verify session timeout after inactivity',
    'Test concurrent session handling',
    'Verify password reset token expiration',
  ],
  
  authorization: [
    'Access admin endpoints as regular user',
    'Access other user\'s resources by modifying IDs',
    'Test horizontal privilege escalation',
    'Test vertical privilege escalation',
    'Verify role-based access control on all endpoints',
    'Test CORS configuration',
  ],
  
  inputValidation: [
    'Test XSS payloads in all input fields',
    'Test SQL injection in search and filters',
    'Test file upload with malicious extensions',
    'Test file upload with oversized files',
    'Test path traversal in file parameters',
    'Verify output encoding in all responses',
  ],
  
  sessionManagement: [
    'Verify secure flag on session cookies',
    'Verify httpOnly flag on session cookies',
    'Verify SameSite cookie attribute',
    'Test session fixation protection',
    'Verify logout invalidates session',
    'Test session hijacking via stolen cookie',
  ],
  
  dataProtection: [
    'Verify sensitive data encryption at rest',
    'Verify HTTPS enforcement',
    'Check for sensitive data in logs',
    'Test for information disclosure in error messages',
    'Verify PII handling compliance (GDPR)',
  ],
};
```

---

### 21.8 Accessibility Tests

#### axe-core Configuration

```typescript
// a11y-tests/axe-config.ts
import { RunOptions } from 'axe-core';

export const axeConfig: RunOptions = {
  runOnly: {
    type: 'tag',
    values: ['wcag2a', 'wcag2aa', 'wcag21aa', 'best-practice'],
  },
  rules: {
    'color-contrast': { enabled: true },
    'document-title': { enabled: true },
    'html-has-lang': { enabled: true },
    'landmark-one-main': { enabled: true },
    'page-has-heading-one': { enabled: true },
    'region': { enabled: true },
    'skip-link': { enabled: true },
  },
};

// Priority rules for critical a11y issues
export const criticalRules = [
  'aria-roles',
  'aria-required-attr',
  'aria-required-children',
  'aria-required-parent',
  'aria-valid-attr-value',
  'aria-valid-attr',
  'button-name',
  'color-contrast',
  'image-alt',
  'label',
  'link-name',
];
```

```typescript
// cypress/e2e/a11y/accessibility.cy.ts
import { axeConfig } from '../../../a11y-tests/axe-config';

describe('Accessibility Tests', () => {
  beforeEach(() => {
    cy.injectAxe();
  });

  describe('Homepage', () => {
    it('should have no accessibility violations', () => {
      cy.visit('/');
      cy.configureAxe(axeConfig);
      cy.checkA11y();
    });

    it('should pass color contrast requirements', () => {
      cy.visit('/');
      cy.configureAxe({
        runOnly: ['color-contrast'],
      });
      cy.checkA11y();
    });
  });

  describe('News Page', () => {
    it('should have no accessibility violations', () => {
      cy.visit('/wiadomosci');
      cy.checkA11y();
    });
  });

  describe('Login Page', () => {
    it('should have accessible form elements', () => {
      cy.visit('/login');
      cy.checkA11y(null, {
        rules: {
          'label': { enabled: true },
          'button-name': { enabled: true },
        },
      });
    });

    it('should have proper focus management', () => {
      cy.visit('/login');
      cy.getByTestId('email-input').focus();
      cy.checkA11y();
    });
  });

  describe('Admin Panel', () => {
    beforeEach(() => {
      cy.loginAsAdmin();
    });

    it('should have accessible navigation', () => {
      cy.visit('/dashboard');
      cy.checkA11y('[data-testid="main-navigation"]', {
        runOnly: ['landmark', 'aria-roles'],
      });
    });

    it('should have accessible data tables', () => {
      cy.visit('/admin/posts');
      cy.checkA11y('[data-testid="posts-table"]', {
        runOnly: ['aria-roles', 'aria-required-children'],
      });
    });
  });

  describe('Keyboard Navigation', () => {
    it('should be fully navigable by keyboard', () => {
      cy.visit('/');
      cy.get('body').tab();
      
      // Tab through main elements
      const tabbableElements = [
        'skip-link',
        'logo-link',
        'nav-news',
        'nav-sport',
        'nav-business',
        'search-input',
        'first-article',
      ];

      tabbableElements.forEach((testId) => {
        cy.focused().should('have.attr', 'data-testid', testId);
        cy.focused().tab();
      });
    });

    it('should show focus indicators', () => {
      cy.visit('/');
      cy.get('a').first().focus();
      cy.get('a').first().should('have.css', 'outline');
    });
  });
});
```

#### Lighthouse CI

```json
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: [
        'http://localhost:3000/',
        'http://localhost:3000/wiadomosci',
        'http://localhost:3000/login',
        'http://localhost:3000/dashboard',
      ],
      numberOfRuns: 3,
      startServerCommand: 'npm run start',
      startServerReadyPattern: 'Ready on',
      startServerReadyTimeout: 120000,
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        'categories:performance': ['error', { minScore: 0.8 }],
        'categories:accessibility': ['error', { minScore: 0.95 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        'color-contrast': 'error',
        'document-title': 'error',
        'html-has-lang': 'error',
        'image-alt': 'error',
        'label': 'error',
        'link-name': 'error',
        'list': 'error',
        'listitem': 'error',
        'meta-viewport': 'error',
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build application
        run: npm run build
        
      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
          
      - name: Upload Lighthouse reports
        uses: actions/upload-artifact@v4
        with:
          name: lighthouse-reports
          path: '.lighthouseci/'
```

#### WCAG 2.1 AA Compliance Checklist

```markdown
<!-- a11y-tests/WCAG21-AA-CHECKLIST.md -->
# WCAG 2.1 AA Compliance Checklist

## 1. Perceivable

### 1.1 Text Alternatives
- [ ] All images have meaningful alt text
- [ ] Decorative images have empty alt
- [ ] Complex images have extended descriptions
- [ ] Form inputs have associated labels
- [ ] Buttons have accessible names

### 1.2 Time-based Media
- [ ] Videos have captions
- [ ] Videos have transcripts
- [ ] Audio content has transcripts

### 1.3 Adaptable
- [ ] Content is readable without CSS
- [ ] Tables have proper headers
- [ ] Form fields have associated labels
- [ ] Content order is logical

### 1.4 Distinguishable
- [ ] Color contrast ratio >= 4.5:1 for normal text
- [ ] Color contrast ratio >= 3:1 for large text
- [ ] Color contrast ratio >= 3:1 for UI components
- [ ] Text can be resized up to 200%
- [ ] Content reflows at 320px width

## 2. Operable

### 2.1 Keyboard Accessible
- [ ] All functionality available via keyboard
- [ ] No keyboard traps
- [ ] Skip links provided
- [ ] Focus order is logical
- [ ] Focus indicator is visible

### 2.2 Enough Time
- [ ] No time limits without option to extend
- [ ] Moving content can be paused
- [ ] Session timeout warnings provided

### 2.3 Seizures and Physical Reactions
- [ ] No content flashes more than 3 times per second

### 2.4 Navigable
- [ ] Page titles are descriptive
- [ ] Links have descriptive text
- [ ] Multiple ways to find pages
- [ ] Focus indicator visible

### 2.5 Input Modalities
- [ ] Touch targets are at least 44x44px
- [ ] Motion actions can be disabled
- [ ] Functionality doesn't rely on single pointer gestures

## 3. Understandable

### 3.1 Readable
- [ ] HTML lang attribute is set
- [ ] Language changes are marked

### 3.2 Predictable
- [ ] Navigation is consistent
- [ ] Components with same function look consistent
- [ ] No unexpected context changes

### 3.3 Input Assistance
- [ ] Error messages are clear
- [ ] Required fields are marked
- [ ] Error prevention for legal/financial data
- [ ] Suggestions for error correction

## 4. Robust

### 4.1 Compatible
- [ ] Valid HTML markup
- [ ] ARIA used correctly
- [ ] Name, role, value available for components
```

---

### 21.9 CI/CD Integration

#### GitHub Actions Workflow

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '20'
  DATABASE_URL: postgresql://test:test@localhost:5432/regionalne_test

jobs:
  # Unit Tests
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit -- --coverage
        
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unit
          name: unit-tests

  # Integration Tests
  integration-tests:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: regionalne_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
          
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run database migrations
        run: npx prisma migrate deploy
        
      - name: Run integration tests
        run: npm run test:integration -- --coverage
        
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: integration
          name: integration-tests

  # E2E Tests
  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build application
        run: npm run build
        
      - name: Run Cypress tests
        uses: cypress-io/github-action@v6
        with:
          start: npm start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
          browser: chrome
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
          
      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: cypress-screenshots
          path: cypress/screenshots

  # Database Tests
  database-tests:
    name: Database Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: regionalne_test
        ports:
          - 5432:5432
          
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run database tests
        run: npm run test:db

  # Lint and Type Check
  lint-and-typecheck:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run ESLint
        run: npm run lint
        
      - name: Run TypeScript check
        run: npm run typecheck
        
      - name: Check formatting
        run: npm run format:check

  # Security Audit
  security-audit:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        
      - name: Run Snyk test
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # Coverage Report
  coverage:
    name: Coverage Report
    needs: [unit-tests, integration-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download all coverage reports
        uses: actions/download-artifact@v4
        
      - name: Upload to Codecov
        uses: codecov/codecov-action@v3
        with:
          fail_ci_if_error: true
          verbose: true
```

#### Coverage Reporting (Codecov)

```yaml
# codecov.yml
codecov:
  require_ci_to_pass: yes
  notify:
    wait_for_ci: yes

coverage:
  precision: 2
  round: down
  range: "70...100"
  
  status:
    project:
      default:
        target: 70%
        threshold: 5%
        paths:
          - "src"
      frontend:
        target: 70%
        paths:
          - "src/components"
          - "src/pages"
          - "src/hooks"
      backend:
        target: 70%
        paths:
          - "src/api"
          - "src/lib"
      
    patch:
      default:
        target: 70%
        threshold: 5%

parsers:
  gcov:
    branch_detection:
      conditional: yes
      loop: yes
      method: no
      macro: no

comment:
  layout: "reach,diff,flags,files,footer"
  behavior: default
  require_changes: no
  require_base: no
  require_head: yes
```

```typescript
// scripts/merge-coverage.ts
import * as fs from 'fs';
import * as path from 'path';

/**
 * Merge coverage reports from multiple test runs
 */
async function mergeCoverage() {
  const coverageDir = './coverage';
  const reports = [
    './coverage/unit/coverage-final.json',
    './coverage/integration/coverage-final.json',
  ];

  const merged: any = { total: {}, files: {} };

  for (const reportPath of reports) {
    if (fs.existsSync(reportPath)) {
      const report = JSON.parse(fs.readFileSync(reportPath, 'utf8'));
      
      for (const [file, data] of Object.entries(report)) {
        if (!merged.files[file]) {
          merged.files[file] = data;
        } else {
          // Merge coverage data
          merged.files[file].statementMap = {
            ...merged.files[file].statementMap,
            ...((data as any).statementMap || {}),
          };
        }
      }
    }
  }

  // Write merged report
  if (!fs.existsSync(coverageDir)) {
    fs.mkdirSync(coverageDir, { recursive: true });
  }

  fs.writeFileSync(
    path.join(coverageDir, 'coverage-merged.json'),
    JSON.stringify(merged.files, null, 2)
  );

  console.log('Coverage reports merged successfully');
}

mergeCoverage().catch(console.error);
```

---

### 21.10 Test Data Management

#### Fixtures / Factories

```typescript
// tests/factories/PostFactory.ts
import { faker } from '@faker-js/faker/locale/pl';
import { PrismaClient, PostStatus } from '@prisma/client';

const prisma = new PrismaClient();

interface PostFactoryOptions {
  domainId: number;
  categoryId: number;
  status?: PostStatus;
  overrides?: Partial<any>;
}

export class PostFactory {
  static async create(options: PostFactoryOptions) {
    const title = faker.lorem.sentence();
    
    return await prisma.posts.create({
      data: {
        title,
        slug: faker.helpers.slugify(title).toLowerCase(),
        content: faker.lorem.paragraphs(3),
        excerpt: faker.lorem.paragraph(),
        featured_image: faker.image.url(),
        domain_id: options.domainId,
        category_id: options.categoryId,
        status: options.status || 'published',
        published_at: options.status === 'published' ? new Date() : null,
        view_count: faker.number.int({ min: 0, max: 10000 }),
        metadata: {
          author: faker.person.fullName(),
          source: 'factory',
        },
        ...options.overrides,
      },
    });
  }

  static async createMany(count: number, options: PostFactoryOptions) {
    return Promise.all(
      Array.from({ length: count }, () => this.create(options))
    );
  }
}

// tests/factories/UserFactory.ts
import { faker } from '@faker-js/faker/locale/pl';
import bcrypt from 'bcryptjs';

interface UserFactoryOptions {
  role?: string;
  overrides?: Partial<any>;
}

export class UserFactory {
  static async create(options: UserFactoryOptions = {}) {
    const password = 'TestPassword123!';
    const passwordHash = await bcrypt.hash(password, 10);

    return await prisma.users.create({
      data: {
        email: faker.internet.email(),
        password_hash: passwordHash,
        first_name: faker.person.firstName(),
        last_name: faker.person.lastName(),
        role: options.role || 'editor',
        is_active: true,
        email_verified_at: new Date(),
        ...options.overrides,
      },
    });
  }

  static async createAdmin(options: UserFactoryOptions = {}) {
    return this.create({ ...options, role: 'super_admin' });
  }
}

// tests/factories/DomainFactory.ts
import { slugify } from '@/utils/slugify';

interface DomainFactoryOptions {
  overrides?: Partial<any>;
}

export class DomainFactory {
  static async create(options: DomainFactoryOptions = {}) {
    const city = faker.location.city();
    const name = `4${slugify(city)}`;

    return await prisma.domains.create({
      data: {
        name,
        slug: name,
        title: `4${city} - Regionalny Portal`,
        city,
        region: faker.helpers.arrayElement([
          'kujawsko-pomorskie',
          'mazowieckie',
          'malopolskie',
          'slaskie',
        ]),
        is_active: true,
        settings: {
          create: {
            primary_color: faker.color.rgb(),
            secondary_color: faker.color.rgb(),
            font_family: 'Inter',
          },
        },
        ...options.overrides,
      },
    });
  }
}
```

#### Seeding

```typescript
// tests/seeds/development.seed.ts
import { PostFactory } from '../factories/PostFactory';
import { UserFactory } from '../factories/UserFactory';
import { DomainFactory } from '../factories/DomainFactory';

export async function seedDevelopmentData() {
  console.log('Seeding development data...');

  // Create domains
  const domains = await Promise.all([
    DomainFactory.create({ overrides: { name: '4torun', city: 'Toruń' } }),
    DomainFactory.create({ overrides: { name: '4bydgoszcz', city: 'Bydgoszcz' } }),
    DomainFactory.create({ overrides: { name: '4warszawa', city: 'Warszawa' } }),
  ]);

  // Create users
  const users = await Promise.all([
    UserFactory.createAdmin({ overrides: { email: 'admin@example.com' } }),
    UserFactory.create({ role: 'editor', overrides: { email: 'editor@example.com' } }),
    UserFactory.create({ role: 'moderator', overrides: { email: 'moderator@example.com' } }),
  ]);

  // Create posts for each domain
  for (const domain of domains) {
    const categories = await prisma.categories.createMany({
      data: [
        { name: 'Wiadomości', slug: 'wiadomosci', domain_id: domain.id },
        { name: 'Sport', slug: 'sport', domain_id: domain.id },
        { name: 'Kultura', slug: 'kultura', domain_id: domain.id },
      ],
    });

    await PostFactory.createMany(50, {
      domainId: domain.id,
      categoryId: 1,
      status: 'published',
    });

    await PostFactory.createMany(10, {
      domainId: domain.id,
      categoryId: 1,
      status: 'draft',
    });
  }

  // Create test scraper configs
  await prisma.scraper_configs.createMany({
    data: domains.map(domain => ({
      name: `Police Scraper - ${domain.city}`,
      source_type: 'POLICE_RSS',
      source_url: 'https://policja.gov.pl/rss',
      domain_id: domain.id,
      is_active: true,
    })),
  });

  console.log('Development data seeded successfully');
  return { domains, users };
}
```

#### Mock External APIs

```typescript
// tests/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // Mock weather API
  http.get('https://api.openweathermap.org/data/2.5/weather', () => {
    return HttpResponse.json({
      coord: { lon: 18.6, lat: 53.01 },
      weather: [{ id: 800, main: 'Clear', description: 'clear sky' }],
      main: {
        temp: 20.5,
        feels_like: 19.8,
        humidity: 65,
        pressure: 1013,
      },
      wind: { speed: 3.5, deg: 180 },
      clouds: { all: 0 },
      name: 'Toruń',
    });
  }),

  // Mock police RSS feed
  http.get('https://policja.gov.pl/rss', () => {
    return HttpResponse.xml(`
      <?xml version="1.0" encoding="UTF-8"?>
      <rss version="2.0">
        <channel>
          <title>Komenda Miejska Policji w Toruniu</title>
          <item>
            <title>Zdarzenie drogowe na ul. Długiej</title>
            <link>https://policja.gov.pl/123</link>
            <pubDate>Mon, 12 Feb 2024 10:00:00 GMT</pubDate>
            <description>Wypadek samochodowy bez osób poszkodowanych.</description>
          </item>
          <item>
            <title>Kradzież w sklepie</title>
            <link>https://policja.gov.pl/124</link>
            <pubDate>Mon, 12 Feb 2024 09:30:00 GMT</pubDate>
            <description>Ujęcie sprawcy kradzieży.</description>
          </item>
        </channel>
      </rss>
    `);
  }),

  // Mock OpenAI API
  http.post('https://api.openai.com/v1/chat/completions', () => {
    return HttpResponse.json({
      id: 'chatcmpl-test',
      object: 'chat.completion',
      created: Date.now(),
      model: 'gpt-4',
      choices: [{
        index: 0,
        message: {
          role: 'assistant',
          content: JSON.stringify({
            normalized_title: 'Zdarzenie drogowe na ulicy Długiej w Toruniu',
            category: 'news',
            tags: ['wypadek', 'drogówka', 'toruń'],
            sentiment: 'neutral',
          }),
        },
        finish_reason: 'stop',
      }],
    });
  }),

  // Mock image upload (MinIO/S3)
  http.post('http://minio:9000/uploads', () => {
    return HttpResponse.json({
      etag: '"test-etag"',
      location: 'https://cdn.example.com/uploads/test-image.jpg',
    });
  }),
];
```

```typescript
// tests/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// jest.setup.ts
import { server } from './tests/mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

### 21.11 Podsumowanie Testów

| Typ Testu | Narzędzie | Pokrycie | Częstotliwość | Czas Wykonania |
|-----------|-----------|----------|---------------|----------------|
| Unit Tests | Jest | >70% | Każdy PR | ~2 min |
| Integration Tests | Supertest + TestContainers | >50% | Każdy PR | ~5 min |
| E2E Tests | Cypress | Krytyczne ścieżki | Każdy PR + Nightly | ~10 min |
| Database Tests | pg-tap + Prisma | 100% migracji | Każdy PR | ~3 min |
| Load Tests | k6/Artillery | Progi wydajności | Weekly + Release | ~30 min |
| Security Tests | ZAP + Snyk | OWASP Top 10 | Weekly | ~20 min |
| A11y Tests | axe-core + Lighthouse | WCAG 2.1 AA | Każdy PR | ~5 min |

**KONIEC SEKCJI 21**



---

## 22. CI/CD i DevOps 🚀

**Cel:** Zapewnienie automatycznego, bezpiecznego i powtarzalnego procesu wdrażania zmian na środowiska staging i production. Pipeline CI/CD eliminuje błędy ludzkie, przyspiesza deployment i zapewnia wysoką jakość kodu poprzez automatyczne testy.

---

### 22.1 Git Workflow

**Model:** Git Flow (uproszczony) - zoptymalizowany pod ciągłe dostarczanie.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           GIT WORKFLOW                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   feature/login-page                                                    │
│          │                                                              │
│          ▼                                                              │
│   ╔═══════════╗    PR #123    ╔═══════════╗    PR #124    ╔═══════════╗ │
│   ║  feature  ║──────────────▶║  develop  ║──────────────▶║  staging  ║ │
│   ║    /*     ║   (review)    ║           ║   (review)    ║           ║ │
│   ╚═══════════╝               ╚═════╤═════╝               ╚═════╤═════╝ │
│                                     │                           │       │
│                                     │         hotfix/bug-456    │       │
│                                     │              │              │       │
│                                     │              ▼              │       │
│                                     │      ╔═══════════╗          │       │
│                                     └──────║  hotfix   ║──────────┘       │
│                                            ║    /*     ║   (direct PR)    │
│                                            ╚═════╤═════╝   to main        │
│                                                  │                        │
│                                                  ▼                        │
│   ╔═══════════╗    PR #125    ╔═══════════╗   ╔═══════════╗             │
│   ║  staging  ║──────────────▶║   main    ║◄──║  hotfix   ║             │
│   ║           ║  (approved)   ║ (production║   ║   (fast)  ║             │
│   ╚═══════════╝               ║   ready)  ║   ╚═══════════╝             │
│                               ╚═════╤═════╝                             │
│                                     │                                    │
│                                     ▼                                    │
│                            [AUTO-DEPLOY]                                │
│                              production                                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 22.1.1 Opis Branchy

| Branch | Cel | Ochrona | Deployment |
|--------|-----|---------|------------|
| `main` | Kod produkcyjny | 🔒 Wymagany PR + 2 approve | Auto → Production |
| `staging` | Pre-produkcja, testy QA | 🔒 Wymagany PR + 1 approve | Auto → Staging |
| `develop` | Integracja feature'ów | 🔒 Wymagany PR | Ręczny (dev) |
| `feature/*` | Nowe funkcjonalności | ❌ Push wolny | Lokalnie |
| `hotfix/*` | Krytyczne naprawy | 🔒 Fast-track PR | Auto → Production |
| `release/*` | Przygotowanie release'u (opcjonalnie) | 🔒 PR required | Ręczny |

#### 22.1.2 Konwencja Nazewnictwa Branchy

```bash
# Feature branches
feature/SCRAPER-123-dodanie-scrapera-policji
feature/FRONTEND-456-nowy-komponent-hero
feature/API-789-optmalizacja-zapytan

# Bug fix branches
fix/BUG-321-naprawa-paginacji
fix/SEO-654-poprawa-meta-tagow

# Hotfix branches (krytyczne)
hotfix/CRITICAL-001-naprawa-logowania
hotfix/SECURITY-002-patch-xss

# Release branches (opcjonalnie)
release/v1.2.0
release/v2.0.0-beta
```

#### 22.1.3 Konwencja Commit Message

```bash
# Format: <type>(<scope>): <subject>
# Example: feat(scraper): dodano obsługę paginacji dla policji

type:
  feat:     Nowa funkcjonalność
  fix:      Naprawa błędu
  docs:     Zmiany w dokumentacji
  style:    Formatowanie (bez zmian w kodzie)
  refactor: Refaktoryzacja kodu
  perf:     Optymalizacja wydajności
  test:     Dodanie/testy
  chore:    Zadania konserwacyjne (build, deps)
  ci:       Zmiany w CI/CD
  security: Poprawki bezpieczeństwa

scope:
  scraper   - moduł scrapowania
  api       - backend API
  frontend  - aplikacja frontendowa
  db        - baza danych/migracje
  auth      - autentykacja
  admin     - panel administracyjny
  config    - konfiguracja
  deps      - zależności

# Przykłady:
git commit -m "feat(scraper): dodano parser dla gazetek Biedronka"
git commit -m "fix(api): naprawiono N+1 queries w endpoincie /news"
git commit -m "perf(frontend): zoptymalizowano lazy loading obrazków"
git commit -m "docs: aktualizacja README z instrukcją deploymentu"
git commit -m "security(auth): dodano rate limiting dla logowania"
```

---

### 22.2 GitHub Actions Pipeline

#### 22.2.1 Główny Workflow CI/CD

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main, staging, develop]
  workflow_dispatch:  # Ręczne uruchomienie

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'
  REGISTRY: ghcr.io

jobs:
  # ═══════════════════════════════════════════════════════════
  # JOB 1: LINT & FORMAT CHECK
  # ═══════════════════════════════════════════════════════════
  lint:
    name: 🔍 Lint & Format Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Get pnpm store directory
        shell: bash
        run: |
          echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

      - name: Setup pnpm cache
        uses: actions/cache@v3
        with:
          path: ${{ env.STORE_PATH }}
          key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-pnpm-store-

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run ESLint
        run: pnpm lint

      - name: Run Prettier check
        run: pnpm format:check

      - name: Type check
        run: pnpm type-check

  # ═══════════════════════════════════════════════════════════
  # JOB 2: UNIT TESTS
  # ═══════════════════════════════════════════════════════════
  test-unit:
    name: 🧪 Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run unit tests
        run: pnpm test:unit --coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

  # ═══════════════════════════════════════════════════════════
  # JOB 3: INTEGRATION TESTS
  # ═══════════════════════════════════════════════════════════
  test-integration:
    name: 🔗 Integration Tests
    runs-on: ubuntu-latest
    needs: lint
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Setup test database
        run: pnpm prisma migrate deploy
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db

      - name: Run integration tests
        run: pnpm test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret-key

  # ═══════════════════════════════════════════════════════════
  # JOB 4: SECURITY SCAN
  # ═══════════════════════════════════════════════════════════
  security-scan:
    name: 🔒 Security Scan
    runs-on: ubuntu-latest
    needs: lint
    permissions:
      actions: read
      contents: read
      security-events: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Run npm audit
        run: npm audit --audit-level=moderate
        continue-on-error: true

      - name: Check for secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: main
          head: HEAD

  # ═══════════════════════════════════════════════════════════
  # JOB 5: BUILD APPLICATION
  # ═══════════════════════════════════════════════════════════
  build:
    name: 🏗️ Build Application
    runs-on: ubuntu-latest
    needs: [test-unit, test-integration, security-scan]
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
      version: ${{ steps.version.outputs.version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Generate version
        id: version
        run: |
          VERSION=$(date +'%Y.%m.%d')-${GITHUB_SHA::7}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
          echo "VERSION=$VERSION" >> $GITHUB_ENV

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=${{ steps.version.outputs.version }}
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push API image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./docker/Dockerfile.api
          push: true
          tags: ${{ steps.meta.outputs.tags }}-api
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            VERSION=${{ steps.version.outputs.version }}

      - name: Build and push Frontend image
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./docker/Dockerfile.frontend
          push: true
          tags: ${{ steps.meta.outputs.tags }}-frontend
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            VERSION=${{ steps.version.outputs.version }}

  # ═══════════════════════════════════════════════════════════
  # JOB 6: DEPLOY TO STAGING
  # ═══════════════════════════════════════════════════════════
  deploy-staging:
    name: 🚀 Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/staging' || github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.serwisy-lokalne.pl

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.STAGING_SSH_KEY }}

      - name: Deploy to Staging
        run: |
          ./scripts/deploy.sh \
            --environment staging \
            --version ${{ needs.build.outputs.version }} \
            --host ${{ secrets.STAGING_HOST }}

      - name: Run database migrations
        run: |
          ssh ${{ secrets.STAGING_USER }}@${{ secrets.STAGING_HOST }} \
            "cd /opt/app && docker-compose exec -T api pnpm prisma migrate deploy"

      - name: Health check
        run: |
          sleep 10
          curl -f https://staging.serwisy-lokalne.pl/api/health || exit 1

      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          text: 'Staging deployment ${{ job.status }}'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  # ═══════════════════════════════════════════════════════════
  # JOB 7: E2E TESTS (after staging deploy)
  # ═══════════════════════════════════════════════════════════
  test-e2e:
    name: 🎭 E2E Tests
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/staging' || github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install Playwright
        run: |
          npm install -g @playwright/test
          npx playwright install-deps

      - name: Run E2E tests against staging
        run: pnpm test:e2e
        env:
          BASE_URL: https://staging.serwisy-lokalne.pl

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/

  # ═══════════════════════════════════════════════════════════
  # JOB 8: DEPLOY TO PRODUCTION
  # ═══════════════════════════════════════════════════════════
  deploy-production:
    name: 🚀 Deploy to Production
    runs-on: ubuntu-latest
    needs: [build, test-e2e]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://serwisy-lokalne.pl

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.8.0
        with:
          ssh-private-key: ${{ secrets.PRODUCTION_SSH_KEY }}

      - name: Backup database before deploy
        run: |
          ssh ${{ secrets.PRODUCTION_USER }}@${{ secrets.PRODUCTION_HOST }} \
            "cd /opt/app && ./scripts/backup-db.sh pre-deploy-${{ needs.build.outputs.version }}"

      - name: Deploy to Production (Blue/Green)
        run: |
          ./scripts/deploy-blue-green.sh \
            --version ${{ needs.build.outputs.version }} \
            --host ${{ secrets.PRODUCTION_HOST }} \
            --blue-port 3000 \
            --green-port 3001

      - name: Run database migrations
        run: |
          ssh ${{ secrets.PRODUCTION_USER }}@${{ secrets.PRODUCTION_HOST }} \
            "cd /opt/app && docker-compose exec -T api pnpm prisma migrate deploy"

      - name: Health check
        run: |
          sleep 10
          curl -f https://serwisy-lokalne.pl/api/health || exit 1

      - name: Check error rate
        run: |
          ./scripts/monitor-error-rate.sh \
            --url https://serwisy-lokalne.pl \
            --threshold 5 \
            --duration 300

      - name: Notify Slack
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          text: 'Production deployment ${{ job.status }} - v${{ needs.build.outputs.version }}'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

      - name: Create GitHub Release
        if: success()
        uses: softprops/action-gh-release@v1
        with:
          tag_name: v${{ needs.build.outputs.version }}
          name: Release ${{ needs.build.outputs.version }}
          generate_release_notes: true
```

#### 22.2.2 Workflow dla PR (szybki check)

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    branches: [main, staging, develop]

jobs:
  pr-check:
    name: Quick PR Validation
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: '8'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Type check
        run: pnpm type-check

      - name: Unit tests (changed files only)
        run: pnpm test:unit --changedSince=origin/main

      - name: Check PR title
        uses: amannn/action-semantic-pull-request@v5
        with:
          requireScope: true
          scopes: |
            scraper
            api
            frontend
            db
            auth
            admin
            config
            deps
            ci
```

---

### 22.3 Branch Protection Rules

#### 22.3.1 Konfiguracja GitHub

```yaml
# .github/settings.yml (przez Probot Settings)
branches:
  - name: main
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 2
        dismiss_stale_reviews: true
        require_code_owner_reviews: true
      required_status_checks:
        strict: true
        contexts:
          - "🔍 Lint & Format Check"
          - "🧪 Unit Tests"
          - "🔗 Integration Tests"
          - "🔒 Security Scan"
          - "🏗️ Build Application"
      enforce_admins: true
      required_linear_history: true
      allow_force_pushes: false
      allow_deletions: false
      required_signatures: true

  - name: staging
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 1
        dismiss_stale_reviews: true
      required_status_checks:
        strict: true
        contexts:
          - "🔍 Lint & Format Check"
          - "🧪 Unit Tests"
          - "🔗 Integration Tests"
          - "🔒 Security Scan"
      enforce_admins: false
      allow_force_pushes: false
      allow_deletions: false

  - name: develop
    protection:
      required_pull_request_reviews:
        required_approving_review_count: 1
      required_status_checks:
        strict: false
        contexts:
          - "🔍 Lint & Format Check"
          - "🧪 Unit Tests"
      enforce_admins: false
```

#### 22.3.2 Code Owners

```
# .github/CODEOWNERS
# Global owners
* @lead-developer @tech-lead

# API and backend
/apps/api/ @backend-team @lead-developer
/apps/api/src/scrapers/ @scraper-team @backend-team
/apps/api/prisma/ @database-admin @backend-team

# Frontend
/apps/frontend/ @frontend-team @lead-developer
/apps/frontend/components/ @ui-team @frontend-team

# Infrastructure and DevOps
/docker/ @devops-team
/.github/workflows/ @devops-team @lead-developer
/scripts/ @devops-team

# Documentation
/docs/ @tech-writer @lead-developer
*.md @tech-writer

# Security-sensitive files
/apps/api/src/auth/ @security-team @lead-developer
/apps/api/src/middleware/security/ @security-team
```

---

### 22.4 Environments i Konfiguracja

#### 22.4.1 Struktura Środowisk

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ENVIRONMENTS ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐              │
│  │ Development  │─────▶│   Staging    │─────▶│  Production  │              │
│  │  (local)     │      │  (staging)   │      │   (prod)     │              │
│  └──────────────┘      └──────────────┘      └──────────────┘              │
│         │                     │                     │                       │
│         ▼                     ▼                     ▼                       │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐              │
│  │ Localhost    │      │ staging.*    │      │ www.*        │              │
│  │ Docker       │      │ Cloud VPS    │      │ Cloud VPS    │              │
│  │ SQLite/Local │      │ PostgreSQL   │      │ PostgreSQL   │              │
│  │              │      │ Redis        │      │ Redis        │              │
│  └──────────────┘      └──────────────┘      └──────────────┘              │
│                                                                              │
│  URL: localhost:3000   URL: staging.*.pl     URL: www.*.pl                  │
│  DB:  local            DB:  staging-db       DB:  production-db             │
│  SSL: no               SSL: Let's Encrypt    SSL: Let's Encrypt             │
│  CDN: no               CDN: no               CDN: CloudFlare                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 22.4.2 Environment Variables per Environment

```bash
# .env.development (lokalny development)
NODE_ENV=development
APP_ENV=development

# Database
DATABASE_URL="postgresql://user:pass@localhost:5432/serwisy_dev"

# Redis
REDIS_URL="redis://localhost:6379"

# API
API_URL="http://localhost:3001"
API_PORT=3001

# Frontend
FRONTEND_URL="http://localhost:3000"
FRONTEND_PORT=3000

# Auth
JWT_SECRET="dev-secret-do-not-use-in-production"
JWT_EXPIRES_IN="7d"

# External APIs (mock/sandbox)
OPENWEATHER_API_KEY="test-key"
GOOGLE_MAPS_API_KEY="test-key"

# Feature flags
ENABLE_SCRAPERS=true
ENABLE_NOTIFICATIONS=false
ENABLE_ANALYTICS=false

# Logging
LOG_LEVEL="debug"
LOG_PRETTY=true
```

```bash
# .env.staging (staging environment)
NODE_ENV=production
APP_ENV=staging

# Database
DATABASE_URL="postgresql://user:pass@staging-db:5432/serwisy_staging"

# Redis
REDIS_URL="redis://staging-redis:6379"

# API
API_URL="https://staging.serwisy-lokalne.pl/api"
API_PORT=3001

# Frontend
FRONTEND_URL="https://staging.serwisy-lokalne.pl"
FRONTEND_PORT=3000

# Auth
JWT_SECRET="${STAGING_JWT_SECRET}"  # From GitHub Secrets
JWT_EXPIRES_IN="1d"

# External APIs (production keys)
OPENWEATHER_API_KEY="${STAGING_WEATHER_API_KEY}"
GOOGLE_MAPS_API_KEY="${STAGING_MAPS_API_KEY}"

# Feature flags
ENABLE_SCRAPERS=true
ENABLE_NOTIFICATIONS=true
ENABLE_ANALYTICS=false

# Logging
LOG_LEVEL="info"
LOG_PRETTY=false
```

```bash
# .env.production (production environment)
NODE_ENV=production
APP_ENV=production

# Database (with read replica)
DATABASE_URL="postgresql://user:pass@prod-primary:5432/serwisy_prod"
DATABASE_READ_URL="postgresql://user:pass@prod-replica:5432/serwisy_prod"

# Redis (cluster)
REDIS_URL="redis://prod-redis:6379"

# API
API_URL="https://serwisy-lokalne.pl/api"
API_PORT=3001

# Frontend
FRONTEND_URL="https://serwisy-lokalne.pl"
FRONTEND_PORT=3000

# Auth
JWT_SECRET="${PRODUCTION_JWT_SECRET}"  # From GitHub Secrets (rotated monthly)
JWT_EXPIRES_IN="7d"

# External APIs
OPENWEATHER_API_KEY="${PRODUCTION_WEATHER_API_KEY}"
GOOGLE_MAPS_API_KEY="${PRODUCTION_MAPS_API_KEY}"

# Feature flags
ENABLE_SCRAPERS=true
ENABLE_NOTIFICATIONS=true
ENABLE_ANALYTICS=true

# Logging & Monitoring
LOG_LEVEL="warn"
LOG_PRETTY=false
SENTRY_DSN="${PRODUCTION_SENTRY_DSN}"
DATADOG_API_KEY="${PRODUCTION_DATADOG_KEY}"

# Performance
CACHE_TTL=3600
CDN_URL="https://cdn.serwisy-lokalne.pl"
```

---

### 22.5 Deployment Strategy

#### 22.5.1 Blue/Green Deployment

```bash
#!/bin/bash
# scripts/deploy-blue-green.sh

set -e

# Parametry
VERSION=$1
BLUE_PORT=3000
GREEN_PORT=3001
HEALTH_CHECK_URL="/api/health"
MAX_RETRIES=30
RETRY_DELAY=5

# Kolory dla outputu
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${YELLOW}🚀 Starting Blue/Green Deployment...${NC}"
echo "Version: $VERSION"

# Sprawdź który kolor jest aktualnie aktywny
check_active_color() {
    if curl -sf "http://localhost:$BLUE_PORT$HEALTH_CHECK_URL" > /dev/null 2>&1; then
        echo "blue"
    else
        echo "green"
    fi
}

ACTIVE_COLOR=$(check_active_color)
echo -e "${GREEN}✓ Active color: $ACTIVE_COLOR${NC}"

# Ustal nowy kolor (inactive)
if [ "$ACTIVE_COLOR" = "blue" ]; then
    NEW_COLOR="green"
    NEW_PORT=$GREEN_PORT
    OLD_PORT=$BLUE_PORT
else
    NEW_COLOR="blue"
    NEW_PORT=$BLUE_PORT
    OLD_PORT=$GREEN_PORT
fi

echo "Deploying to: $NEW_COLOR (port $NEW_PORT)"

# 1. Pobierz nowe obrazy
echo -e "${YELLOW}📦 Pulling new images...${NC}"
docker pull ghcr.io/serwisy-lokalne/api:$VERSION
docker pull ghcr.io/serwisy-lokalne/frontend:$VERSION

# 2. Uruchom nowe kontenery (green/blue)
echo -e "${YELLOW}🐳 Starting new containers ($NEW_COLOR)...${NC}"
COMPOSE_PROJECT_NAME="app-$NEW_COLOR" docker-compose -f docker-compose.$NEW_COLOR.yml up -d

# 3. Health check nowych kontenerów
echo -e "${YELLOW}🏥 Health check...${NC}"
RETRIES=0
while [ $RETRIES -lt $MAX_RETRIES ]; do
    if curl -sf "http://localhost:$NEW_PORT$HEALTH_CHECK_URL" > /dev/null 2>&1; then
        echo -e "${GREEN}✓ New containers are healthy${NC}"
        break
    fi
    RETRIES=$((RETRIES + 1))
    echo "  Attempt $RETRIES/$MAX_RETRIES..."
    sleep $RETRY_DELAY
done

if [ $RETRIES -eq $MAX_RETRIES ]; then
    echo -e "${RED}✗ Health check failed! Rolling back...${NC}"
    COMPOSE_PROJECT_NAME="app-$NEW_COLOR" docker-compose -f docker-compose.$NEW_COLOR.yml down
    exit 1
fi

# 4. Migracja bazy danych (przed switch)
echo -e "${YELLOW}🗄️ Running database migrations...${NC}"
docker-compose exec -T api-$NEW_COLOR pnpm prisma migrate deploy

# 5. Przełącz Nginx na nowy kolor
echo -e "${YELLOW}🔄 Switching traffic to $NEW_COLOR...${NC}"
sed -i "s/proxy_pass http:\/\/localhost:$OLD_PORT/proxy_pass http:\/\/localhost:$NEW_PORT/g" /etc/nginx/sites-enabled/app
nginx -t && nginx -s reload

echo -e "${GREEN}✓ Traffic switched to $NEW_COLOR${NC}"

# 6. Poczekaj chwilę i sprawdź czy wszystko działa
echo -e "${YELLOW}⏱️ Monitoring for 60 seconds...${NC}"
sleep 60

# 7. Sprawdź error rate
ERROR_RATE=$(curl -s "http://localhost:$NEW_PORT/api/metrics/error-rate" || echo "0")
if (( $(echo "$ERROR_RATE > 5" | bc -l) )); then
    echo -e "${RED}⚠️ High error rate detected ($ERROR_RATE%)! Rolling back...${NC}"
    # Rollback
    sed -i "s/proxy_pass http:\/\/localhost:$NEW_PORT/proxy_pass http:\/\/localhost:$OLD_PORT/g" /etc/nginx/sites-enabled/app
    nginx -s reload
    COMPOSE_PROJECT_NAME="app-$NEW_COLOR" docker-compose -f docker-compose.$NEW_COLOR.yml down
    exit 1
fi

echo -e "${GREEN}✓ Error rate acceptable: $ERROR_RATE%${NC}"

# 8. Zatrzymaj stare kontenery (po 5 minutach)
echo -e "${YELLOW}🛑 Old containers will be stopped in 5 minutes...${NC}"
(
    sleep 300
    COMPOSE_PROJECT_NAME="app-$ACTIVE_COLOR" docker-compose -f docker-compose.$ACTIVE_COLOR.yml down
    echo "Old containers ($ACTIVE_COLOR) stopped"
) &

echo -e "${GREEN}🎉 Deployment completed successfully!${NC}"
echo "Active color: $NEW_COLOR (port $NEW_PORT)"
echo "Previous color: $ACTIVE_COLOR (will be stopped in 5 min)"
```

#### 22.5.2 Canary Deployment (opcjonalnie)

```yaml
# docker-compose.canary.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx/canary.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - api-stable
      - api-canary

  api-stable:
    image: ghcr.io/serwisy-lokalne/api:stable
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3

  api-canary:
    image: ghcr.io/serwisy-lokalne/api:canary
    environment:
      - NODE_ENV=production
      - CANARY=true
    deploy:
      replicas: 1  # 25% traffic
```

```nginx
# nginx/canary.conf - prosty load balancing z wagami
upstream backend {
    server api-stable:3000 weight=75;
    server api-canary:3000 weight=25;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

#### 22.5.3 Rollback Procedure

```bash
#!/bin/bash
# scripts/rollback.sh

set -e

VERSION=$1  # Opcjonalnie: konkretna wersja do przywrócenia

echo "🔄 Initiating rollback..."

# 1. Znajdź poprzednią działającą wersję
if [ -z "$VERSION" ]; then
    VERSION=$(docker images ghcr.io/serwisy-lokalne/api --format "{{.Tag}}" | grep -v "latest\|canary" | head -2 | tail -1)
    echo "Rolling back to version: $VERSION"
fi

# 2. Sprawdź który kolor jest aktywny
if docker-compose -f docker-compose.blue.yml ps | grep -q "Up"; then
    ACTIVE_COLOR="blue"
    ROLLBACK_COLOR="green"
else
    ACTIVE_COLOR="green"
    ROLLBACK_COLOR="blue"
fi

echo "Active: $ACTIVE_COLOR, Rolling back to: $ROLLBACK_COLOR"

# 3. Uruchom poprzednią wersję na nieaktywnym kolorze
docker pull ghcr.io/serwisy-lokalne/api:$VERSION
COMPOSE_PROJECT_NAME="app-$ROLLBACK_COLOR" VERSION=$VERSION docker-compose -f docker-compose.$ROLLBACK_COLOR.yml up -d

# 4. Poczekaj na health check
sleep 30

# 5. Przełącz traffic
sed -i "s/proxy_pass http:\/\/localhost:[0-9]*/proxy_pass http:\/\/localhost:$( [ $ROLLBACK_COLOR = 'blue' ] && echo 3000 || echo 3001)/g" /etc/nginx/sites-enabled/app
nginx -s reload

echo "✅ Rollback completed to version $VERSION"

# 6. Notify
slack-notify "⚠️ ROLLBACK EXECUTED" "Rolled back to $VERSION" "#ff0000"
```

---

### 22.6 Docker Configuration

#### 22.6.1 Dockerfile dla API

```dockerfile
# docker/Dockerfile.api
# ═══════════════════════════════════════════════════════════
# STAGE 1: Dependencies
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat openssl

WORKDIR /app

# Install pnpm
RUN npm install -g pnpm@8

# Copy package files
COPY package.json pnpm-lock.yaml* ./
COPY apps/api/package.json ./apps/api/
COPY packages/database/package.json ./packages/database/
COPY packages/shared/package.json ./packages/shared/

# Install dependencies
RUN pnpm install --frozen-lockfile

# ═══════════════════════════════════════════════════════════
# STAGE 2: Builder
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS builder

WORKDIR /app

RUN npm install -g pnpm@8

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/api/node_modules ./apps/api/node_modules

# Copy source code
COPY . .

# Generate Prisma Client
RUN pnpm --filter database prisma generate

# Build application
RUN pnpm --filter api build

# ═══════════════════════════════════════════════════════════
# STAGE 3: Runner (production)
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3001

RUN apk add --no-cache dumb-init curl

# Create non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 apiuser

# Copy built application
COPY --from=builder --chown=apiuser:nodejs /app/apps/api/dist ./dist
COPY --from=builder --chown=apiuser:nodejs /app/apps/api/package.json ./
COPY --from=builder --chown=apiuser:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=apiuser:nodejs /app/apps/api/node_modules ./node_modules
COPY --from=builder --chown=apiuser:nodejs /app/apps/api/prisma ./prisma

# Copy Prisma schema and migrations
COPY --from=builder /app/packages/database/prisma ./prisma

USER apiuser

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3001/api/health || exit 1

CMD ["dumb-init", "node", "dist/main.js"]
```

#### 22.6.2 Dockerfile dla Frontend

```dockerfile
# docker/Dockerfile.frontend
# ═══════════════════════════════════════════════════════════
# STAGE 1: Dependencies
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat

WORKDIR /app

RUN npm install -g pnpm@8

COPY package.json pnpm-lock.yaml* ./
COPY apps/frontend/package.json ./apps/frontend/
COPY packages/ui/package.json ./packages/ui/
COPY packages/shared/package.json ./packages/shared/

RUN pnpm install --frozen-lockfile

# ═══════════════════════════════════════════════════════════
# STAGE 2: Builder
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS builder

WORKDIR /app

RUN npm install -g pnpm@8

COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build arguments for environment variables
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_APP_NAME
ARG NEXT_PUBLIC_ANALYTICS_ID

ENV NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}
ENV NEXT_PUBLIC_APP_NAME=${NEXT_PUBLIC_APP_NAME}
ENV NEXT_PUBLIC_ANALYTICS_ID=${NEXT_PUBLIC_ANALYTICS_ID}

# Build Next.js app
RUN pnpm --filter frontend build

# ═══════════════════════════════════════════════════════════
# STAGE 3: Runner
# ═══════════════════════════════════════════════════════════
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Copy only necessary files
COPY --from=builder /app/apps/frontend/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/apps/frontend/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/frontend/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV HOSTNAME="0.0.0.0"

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/api/health || exit 1

CMD ["node", "server.js"]
```

#### 22.6.3 Docker Compose dla Development

```yaml
# docker-compose.yml (development)
version: '3.8'

services:
  # ═════════════════════════════════════════════════════════
  # PostgreSQL Database
  # ═════════════════════════════════════════════════════════
  postgres:
    image: postgres:16-alpine
    container_name: serwisy-postgres
    environment:
      POSTGRES_USER: devuser
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: serwisy_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U devuser -d serwisy_dev"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ═════════════════════════════════════════════════════════
  # Redis Cache
  # ═════════════════════════════════════════════════════════
  redis:
    image: redis:7-alpine
    container_name: serwisy-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # ═════════════════════════════════════════════════════════
  # MinIO (S3-compatible storage)
  # ═════════════════════════════════════════════════════════
  minio:
    image: minio/minio:latest
    container_name: serwisy-minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  # ═════════════════════════════════════════════════════════
  # Backend API
  # ═════════════════════════════════════════════════════════
  api:
    build:
      context: .
      dockerfile: docker/Dockerfile.api
      target: runner
    container_name: serwisy-api
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://devuser:devpass@postgres:5432/serwisy_dev
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev-jwt-secret-change-in-production
      STORAGE_ENDPOINT: minio
      STORAGE_PORT: 9000
      STORAGE_USE_SSL: "false"
      STORAGE_ACCESS_KEY: minioadmin
      STORAGE_SECRET_KEY: minioadmin
      STORAGE_BUCKET: serwisy-uploads
    ports:
      - "3001:3001"
    volumes:
      - ./apps/api:/app/apps/api
      - /app/apps/api/node_modules
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      minio:
        condition: service_healthy
    command: pnpm --filter api dev

  # ═════════════════════════════════════════════════════════
  # Frontend
  # ═════════════════════════════════════════════════════════
  frontend:
    build:
      context: .
      dockerfile: docker/Dockerfile.frontend
      target: runner
    container_name: serwisy-frontend
    environment:
      NODE_ENV: development
      NEXT_PUBLIC_API_URL: http://localhost:3001
    ports:
      - "3000:3000"
    volumes:
      - ./apps/frontend:/app/apps/frontend
      - /app/apps/frontend/node_modules
      - /app/apps/frontend/.next
    depends_on:
      - api
    command: pnpm --filter frontend dev

  # ═════════════════════════════════════════════════════════
  # Scraper Worker (separate container for heavy tasks)
  # ═════════════════════════════════════════════════════════
  scraper-worker:
    build:
      context: .
      dockerfile: docker/Dockerfile.api
    container_name: serwisy-scraper
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://devuser:devpass@postgres:5432/serwisy_dev
      REDIS_URL: redis://redis:6379
      WORKER_TYPE: scraper
    volumes:
      - ./apps/api:/app/apps/api
      - /app/apps/api/node_modules
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    profiles:
      - scraper  # Only starts with: docker-compose --profile scraper up

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

---

### 22.7 Secrets Management

#### 22.7.1 GitHub Secrets

```yaml
# Lista wymaganych sekretów w GitHub Repository
# Settings > Secrets and variables > Actions

# ═══════════════════════════════════════════════════════════
# DEPLOYMENT SECRETS
# ═══════════════════════════════════════════════════════════
STAGING_SSH_KEY: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  ... (staging server private key)
  -----END OPENSSH PRIVATE KEY-----

STAGING_HOST: staging.serwisy-lokalne.pl
STAGING_USER: deploy

PRODUCTION_SSH_KEY: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  ... (production server private key)
  -----END OPENSSH PRIVATE KEY-----

PRODUCTION_HOST: serwisy-lokalne.pl
PRODUCTION_USER: deploy

# ═══════════════════════════════════════════════════════════
# DATABASE SECRETS
# ═══════════════════════════════════════════════════════════
STAGING_DATABASE_URL: postgresql://user:pass@staging-db:5432/serwisy_staging
PRODUCTION_DATABASE_URL: postgresql://user:pass@prod-db:5432/serwisy_prod

# ═══════════════════════════════════════════════════════════
# JWT & AUTH
# ═══════════════════════════════════════════════════════════
STAGING_JWT_SECRET: staging-jwt-secret-min-32-chars-long
PRODUCTION_JWT_SECRET: production-jwt-secret-min-32-chars-long

# ═══════════════════════════════════════════════════════════
# EXTERNAL API KEYS
# ═══════════════════════════════════════════════════════════
STAGING_WEATHER_API_KEY: xxx
PRODUCTION_WEATHER_API_KEY: xxx

STAGING_MAPS_API_KEY: xxx
PRODUCTION_MAPS_API_KEY: xxx

# ═══════════════════════════════════════════════════════════
# MONITORING & NOTIFICATIONS
# ═══════════════════════════════════════════════════════════
SLACK_WEBHOOK_URL: https://hooks.slack.com/services/xxx
SENTRY_DSN: https://xxx@sentry.io/xxx
DATADOG_API_KEY: xxx

# ═══════════════════════════════════════════════════════════
# CONTAINER REGISTRY (auto-generated)
# ═══════════════════════════════════════════════════════════
GITHUB_TOKEN: ${{ github.token }}  # Auto-generated
```

#### 22.7.2 Lokalne .env files (gitignored)

```gitignore
# .gitignore
# Environment files
.env
.env.local
.env.*.local
.env.development
.env.staging
.env.production

# Except examples
!.env.example
!.env.development.example
```

```bash
# .env.example (template dla developerów)
# Skopiuj do .env.development i uzupełnij

# Database
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# Redis
REDIS_URL="redis://localhost:6379"

# JWT (generate with: openssl rand -base64 32)
JWT_SECRET="your-jwt-secret-here"
JWT_EXPIRES_IN="7d"

# External APIs
OPENWEATHER_API_KEY=""
GOOGLE_MAPS_API_KEY=""
FACEBOOK_APP_ID=""
FACEBOOK_APP_SECRET=""

# Storage (MinIO/S3)
STORAGE_ENDPOINT="localhost"
STORAGE_PORT="9000"
STORAGE_USE_SSL="false"
STORAGE_ACCESS_KEY="minioadmin"
STORAGE_SECRET_KEY="minioadmin"
STORAGE_BUCKET="serwisy-uploads"

# Email
SMTP_HOST=""
SMTP_PORT="587"
SMTP_USER=""
SMTP_PASS=""
```

#### 22.7.3 SOPS (Secrets OPerationS) - opcjonalnie

```yaml
# .sops.yaml - szyfrowanie sekretów w repo
# Uwaga: wymaga zainstalowania Mozilla SOPS

creation_rules:
  - path_regex: secrets/.*\.yaml$
    kms: arn:aws:kms:eu-central-1:xxx:key/xxx
    # lub
    pgp: 'FINGERPRINT_KEY_ADMINA'

  - path_regex: secrets/development/.*\.yaml$
    kms: arn:aws:kms:eu-central-1:xxx:key/dev-key
```

```bash
# Szyfrowanie sekretów
sops encrypt secrets/production/database.yaml > secrets/production/database.enc.yaml

# Odszyfrowanie i edycja
sops secrets/production/database.enc.yaml
```

---

### 22.8 Automated Deployment Scripts

#### 22.8.1 Główny skrypt deploymentu

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

# ═══════════════════════════════════════════════════════════
# KONFIGURACJA
# ═══════════════════════════════════════════════════════════
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
VERSION=""
ENVIRONMENT=""
HOST=""
DRY_RUN=false

# Kolory
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ═══════════════════════════════════════════════════════════
# FUNKCJE POMOCNICZE
# ═══════════════════════════════════════════════════════════
log_info() { echo -e "${BLUE}[INFO]${NC} $1"; }
log_success() { echo -e "${GREEN}[OK]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; }

usage() {
    cat << EOF
Użycie: $0 [OPCJE]

Opcje:
    -e, --environment    Środowisko (staging|production)
    -v, --version        Wersja do deploymentu (domyślnie: latest)
    -h, --host           Host docelowy
    -d, --dry-run        Symulacja (bez rzeczywistych zmian)
    --help               Wyświetl tę pomoc

Przykłady:
    $0 -e staging -v 2024.02.12-abc1234
    $0 -e production -v 2024.02.12-abc1234 -h serwisy-lokalne.pl
EOF
}

# Parsowanie argumentów
while [[ $# -gt 0 ]]; do
    case $1 in
        -e|--environment)
            ENVIRONMENT="$2"
            shift 2
            ;;
        -v|--version)
            VERSION="$2"
            shift 2
            ;;
        -h|--host)
            HOST="$2"
            shift 2
            ;;
        -d|--dry-run)
            DRY_RUN=true
            shift
            ;;
        --help)
            usage
            exit 0
            ;;
        *)
            log_error "Nieznana opcja: $1"
            usage
            exit 1
            ;;
    esac
done

# Walidacja
if [[ -z "$ENVIRONMENT" ]]; then
    log_error "Wymagany parametr: --environment"
    usage
    exit 1
fi

if [[ "$ENVIRONMENT" != "staging" && "$ENVIRONMENT" != "production" ]]; then
    log_error "Nieprawidłowe środowisko. Użyj: staging lub production"
    exit 1
fi

# Domyślne wartości
if [[ -z "$VERSION" ]]; then
    VERSION="latest"
    log_warn "Nie podano wersji, używam: latest"
fi

if [[ -z "$HOST" ]]; then
    if [[ "$ENVIRONMENT" == "staging" ]]; then
        HOST="staging.serwisy-lokalne.pl"
    else
        HOST="serwisy-lokalne.pl"
    fi
fi

log_info "════════════════════════════════════════════════════"
log_info "  Deployment Configuration"
log_info "════════════════════════════════════════════════════"
log_info "Environment: $ENVIRONMENT"
log_info "Version:     $VERSION"
log_info "Host:        $HOST"
log_info "Dry Run:     $DRY_RUN"
log_info "════════════════════════════════════════════════════"

if [[ "$DRY_RUN" == true ]]; then
    log_warn "TRYB SYMULACJI - żadne zmiany nie zostaną wprowadzone"
    exit 0
fi

# ═══════════════════════════════════════════════════════════
# DEPLOYMENT STEPS
# ═══════════════════════════════════════════════════════════

# 1. Pre-deployment checks
log_info "Step 1: Pre-deployment checks"

# Sprawdź czy obraz istnieje
if ! docker pull "ghcr.io/serwisy-lokalne/api:$VERSION" > /dev/null 2>&1; then
    log_error "Obraz API w wersji $VERSION nie istnieje!"
    exit 1
fi

if ! docker pull "ghcr.io/serwisy-lokalne/frontend:$VERSION" > /dev/null 2>&1; then
    log_error "Obraz Frontend w wersji $VERSION nie istnieje!"
    exit 1
fi

log_success "Obrazy Docker istnieją"

# 2. SSH do serwera i deployment
log_info "Step 2: Remote deployment via SSH"

ssh "deploy@$HOST" << EOF
    set -e
    
    echo "[REMOTE] Changing to app directory..."
    cd /opt/serwisy-lokalne
    
    echo "[REMOTE] Pulling new images..."
    docker pull ghcr.io/serwisy-lokalne/api:$VERSION
    docker pull ghcr.io/serwisy-lokalne/frontend:$VERSION
    
    echo "[REMOTE] Updating docker-compose..."
    export VERSION=$VERSION
    docker-compose -f docker-compose.$ENVIRONMENT.yml up -d
    
    echo "[REMOTE] Cleaning up old images..."
    docker image prune -f
    
    echo "[REMOTE] Deployment complete!"
EOF

if [[ $? -ne 0 ]]; then
    log_error "Deployment zakończony błędem!"
    exit 1
fi

log_success "Kontenery zaktualizowane"

# 3. Database migration
log_info "Step 3: Database migration"

ssh "deploy@$HOST" "cd /opt/serwisy-lokalne && docker-compose -f docker-compose.$ENVIRONMENT.yml exec -T api npx prisma migrate deploy"

if [[ $? -ne 0 ]]; then
    log_error "Migracja bazy danych nie powiodła się!"
    exit 1
fi

log_success "Migracja zakończona"

# 4. Health check
log_info "Step 4: Health check"

HEALTH_URL="https://$HOST/api/health"
RETRIES=0
MAX_RETRIES=30

while [[ $RETRIES -lt $MAX_RETRIES ]]; do
    if curl -sf "$HEALTH_URL" > /dev/null 2>&1; then
        log_success "Health check passed"
        break
    fi
    
    RETRIES=$((RETRIES + 1))
    log_warn "Health check attempt $RETRIES/$MAX_RETRIES..."
    sleep 5
done

if [[ $RETRIES -eq $MAX_RETRIES ]]; then
    log_error "Health check failed after $MAX_RETRIES attempts!"
    exit 1
fi

# 5. Post-deployment verification
log_info "Step 5: Post-deployment verification"

# Sprawdź wersję API
DEPLOYED_VERSION=$(curl -sf "$HEALTH_URL" | grep -o '"version":"[^"]*"' | cut -d'"' -f4)
log_info "Deployed version: $DEPLOYED_VERSION"

# ═══════════════════════════════════════════════════════════
# NOTIFICATION
# ═══════════════════════════════════════════════════════════
log_info "Step 6: Sending notifications"

# Slack notification (jeśli skonfigurowane)
if [[ -n "$SLACK_WEBHOOK_URL" ]]; then
    curl -s -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"✅ Deployment successful: $ENVIRONMENT v$VERSION\"}" \
        "$SLACK_WEBHOOK_URL" > /dev/null
fi

log_success "════════════════════════════════════════════════════"
log_success "  Deployment Completed Successfully!"
log_success "════════════════════════════════════════════════════"
log_info "Environment: $ENVIRONMENT"
log_info "Version:     $VERSION"
log_info "Host:        $HOST"
log_info "URL:         https://$HOST"
```

#### 22.8.2 Skrypt backupu bazy danych

```bash
#!/bin/bash
# scripts/backup-db.sh

set -e

BACKUP_NAME=${1:-$(date +%Y%m%d_%H%M%S)}
BACKUP_DIR="/opt/backups/postgres"
RETENTION_DAYS=30

# Upewnij się że katalog istnieje
mkdir -p "$BACKUP_DIR"

log_info() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] INFO: $1"; }

log_info "Starting database backup: $BACKUP_NAME"

# Wykonaj backup
docker-compose exec -T postgres pg_dumpall -c -U postgres | \
    gzip > "$BACKUP_DIR/$BACKUP_NAME.sql.gz"

if [[ $? -eq 0 ]]; then
    log_info "Backup completed: $BACKUP_DIR/$BACKUP_NAME.sql.gz"
    log_info "Size: $(du -h $BACKUP_DIR/$BACKUP_NAME.sql.gz | cut -f1)"
else
    echo "Backup failed!" >&2
    exit 1
fi

# Usuń stare backupy
log_info "Cleaning up backups older than $RETENTION_DAYS days"
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

log_info "Backup process completed"
```

---

### 22.9 Monitoring Deployments

#### 22.9.1 Health Check Endpoint

```typescript
// apps/api/src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, PrismaHealthIndicator } from '@nestjs/terminus';

@Controller('api/health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private prisma: PrismaHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  async check() {
    return this.health.check([
      () => this.prisma.pingCheck('database'),
      () => ({
        api: {
          status: 'up',
          version: process.env.VERSION || 'unknown',
          timestamp: new Date().toISOString(),
        },
      }),
    ]);
  }

  @Get('ready')
  async readiness() {
    // Sprawdź czy wszystkie zależności są gotowe
    return {
      ready: true,
      checks: {
        database: await this.checkDatabase(),
        redis: await this.checkRedis(),
      },
    };
  }

  @Get('live')
  liveness() {
    // Prosty check czy aplikacja żyje
    return { status: 'alive' };
  }
}
```

#### 22.9.2 Monitoring Error Rate

```bash
#!/bin/bash
# scripts/monitor-error-rate.sh

URL=$1
THRESHOLD=${2:-5}  # Domyślnie 5%
DURATION=${3:-300} # Domyślnie 5 minut

END_TIME=$(($(date +%s) + DURATION))
TOTAL_REQUESTS=0
ERROR_REQUESTS=0

echo "Monitoring error rate for ${DURATION}s..."

while [[ $(date +%s) -lt $END_TIME ]]; do
    # Wykonaj request i sprawdź status
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$URL/api/health")
    
    TOTAL_REQUESTS=$((TOTAL_REQUESTS + 1))
    
    if [[ $HTTP_CODE -ge 400 ]]; then
        ERROR_REQUESTS=$((ERROR_REQUESTS + 1))
    fi
    
    # Oblicz procent błędów
    if [[ $TOTAL_REQUESTS -gt 0 ]]; then
        ERROR_RATE=$(echo "scale=2; ($ERROR_REQUESTS / $TOTAL_REQUESTS) * 100" | bc)
        echo "Requests: $TOTAL_REQUESTS, Errors: $ERROR_REQUESTS, Rate: $ERROR_RATE%"
        
        if (( $(echo "$ERROR_RATE > $THRESHOLD" | bc -l) )); then
            echo "ERROR: Error rate $ERROR_RATE% exceeds threshold $THRESHOLD%!"
            exit 1
        fi
    fi
    
    sleep 5
done

echo "Monitoring complete. Final error rate: $ERROR_RATE%"
```

#### 22.9.3 Deployment Dashboard (Grafana)

```json
{
  "dashboard": {
    "title": "Deployment Monitoring",
    "panels": [
      {
        "title": "Deployment Status",
        "type": "stat",
        "targets": [
          {
            "expr": "deployment_status{environment=~\"$environment\"}"
          }
        ]
      },
      {
        "title": "Error Rate (5m)",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m]) / rate(http_requests_total[5m]) * 100",
            "legendFormat": "Error Rate %"
          }
        ],
        "alert": {
          "name": "High Error Rate",
          "condition": "B",
          "evaluator": {
            "type": "gt",
            "params": [5]
          }
        }
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

---

### 22.10 Database Migrations w CI/CD

#### 22.10.1 Prisma Migration Strategy

```yaml
# .github/workflows/migrate.yml
name: Database Migration

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

jobs:
  migrate:
    runs-on: ubuntu-latest
    environment: ${{ github.event.inputs.environment }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Generate Prisma Client
        run: pnpm prisma generate
      
      - name: Backup database before migration
        run: |
          ssh deploy@${{ secrets.HOST }} \
            "cd /opt/app && ./scripts/backup-db.sh pre-migration-$(date +%s)"
      
      - name: Run migrations
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: pnpm prisma migrate deploy
      
      - name: Verify migrations
        run: |
          pnpm prisma migrate status
          pnpm prisma db seed --preview-feature
      
      - name: Notify on failure
        if: failure()
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK_URL }} \
            -H 'Content-type: application/json' \
            -d '{"text":"❌ Database migration failed!"}'
```

#### 22.10.2 Migration Safety Checklist

```bash
#!/bin/bash
# scripts/migration-check.sh

echo "🔍 Pre-migration safety checks..."

# 1. Sprawdź czy są niezastosowane migracjie
UNAPPLIED=$(npx prisma migrate status --preview-feature 2>&1 | grep -c "not yet applied" || true)

if [[ $UNAPPLIED -eq 0 ]]; then
    echo "✅ All migrations are up to date"
    exit 0
fi

echo "⚠️ Found $UNAPPLIED unapplied migrations"

# 2. Sprawdź czy migracje są destructive
for file in prisma/migrations/*/migration.sql; do
    if [[ -f "$file" ]]; then
        if grep -q "DROP TABLE\|DROP COLUMN\|ALTER TABLE.*DROP" "$file"; then
            echo "⚠️ WARNING: Destructive changes detected in $file"
            echo "   Review before applying!"
        fi
    fi
done

# 3. Sprawdź rozmiar tabel (dla migracji potencjalnie długich)
echo "📊 Table sizes:"
psql "$DATABASE_URL" -c "
SELECT schemaname, relname, pg_size_pretty(pg_total_relation_size(relid)) 
FROM pg_stat_user_tables 
ORDER BY pg_total_relation_size(relid) DESC 
LIMIT 10;
"

echo "✅ Safety checks complete"
```

#### 22.10.3 Rollback Plan

```bash
#!/bin/bash
# scripts/migration-rollback.sh

set -e

BACKUP_NAME=$1

echo "🔄 Migration Rollback Procedure"
echo "════════════════════════════════"

if [[ -z "$BACKUP_NAME" ]]; then
    echo "Usage: $0 <backup-name>"
    echo "Available backups:"
    ls -la /opt/backups/postgres/
    exit 1
fi

echo "⚠️ WARNING: This will RESTORE database from backup!"
echo "Backup: $BACKUP_NAME"
read -p "Are you sure? Type 'yes' to continue: " confirm

if [[ $confirm != "yes" ]]; then
    echo "Aborted."
    exit 0
fi

# 1. Zatrzymaj aplikację
echo "🛑 Stopping application..."
docker-compose stop api

# 2. Restore z backupu
echo "📦 Restoring database..."
gunzip < "/opt/backups/postgres/$BACKUP_NAME.sql.gz" | \
    docker-compose exec -T postgres psql -U postgres

# 3. Uruchom aplikację
echo "▶️ Starting application..."
docker-compose start api

# 4. Health check
echo "🏥 Health check..."
sleep 10
curl -f http://localhost:3001/api/health || exit 1

echo "✅ Rollback completed!"
```

---

### 22.11 Zadania Implementacyjne dla DevOps

| # | Zadanie | Szacowanie | Priorytet |
|---|---------|-----------|-----------|
| 22.1 | Konfiguracja GitHub Actions workflow | 4h | Krytyczny |
| 22.2 | Setup branch protection rules | 1h | Krytyczny |
| 22.3 | Konfiguracja GitHub Secrets | 2h | Krytyczny |
| 22.4 | Stworzenie Dockerfile dla API | 3h | Krytyczny |
| 22.5 | Stworzenie Dockerfile dla Frontend | 3h | Krytyczny |
| 22.6 | Konfiguracja docker-compose dev | 2h | Wysoki |
| 22.7 | Implementacja Blue/Green deployment | 6h | Wysoki |
| 22.8 | Skrypt deploymentu przez SSH | 3h | Wysoki |
| 22.9 | Health check endpoints | 2h | Wysoki |
| 22.10 | Automatyczny rollback | 4h | Średni |
| 22.11 | Monitoring error rate post-deploy | 3h | Średni |
| 22.12 | Slack notifications | 2h | Średni |
| 22.13 | Database migration automation | 3h | Średni |
| 22.14 | Backup automation | 2h | Średni |
| 22.15 | Grafana dashboard dla deploymentów | 4h | Niski |

---

### 22.12 Checklist Przed Pierwszym Deployem

```markdown
## Pre-Deployment Checklist

### 🔐 Security
- [ ] Wszystkie secrets dodane do GitHub
- [ ] SSH kliki wygenerowane i dodane do serwerów
- [ ] Firewall skonfigurowany (tylko 80, 443, 22)
- [ ] Fail2ban zainstalowany

### 🗄️ Database
- [ ] PostgreSQL zainstalowany i skonfigurowany
- [ ] Użytkownik bazy danych utworzony
- [ ] Pierwsza migracja wykonana
- [ ] Backup schedule skonfigurowany

### 🐳 Docker
- [ ] Docker zainstalowany na serwerze
- [ ] Docker Compose zainstalowany
- [ ] Images przetestowane lokalnie

### 🌐 DNS & SSL
- [ ] DNS A record wskazuje na serwer
- [ ] Let's Encrypt skonfigurowany
- [ ] SSL certificate auto-renewal

### 📊 Monitoring
- [ ] Sentry DSN skonfigurowany
- [ ] Health check endpoint działa
- [ ] Log aggregation (opcjonalnie)

### 🔔 Notifications
- [ ] Slack webhook skonfigurowany
- [ ] Email dla krytycznych alertów
```

---

**KONIEC SEKCJI 22**


---

## 23. DOKUMENTACJA API (OpenAPI/Swagger) 📚

### 23.1 Specyfikacja OpenAPI 3.0

#### Plik Konfiguracyjny

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: Regionalne Serwisy API
  description: API dla systemu zarządzania regionalnymi portalami
  version: 1.0.0
  contact:
    name: Support
    email: api@serwisy-lokalne.pl
  license:
    name: Proprietary

servers:
  - url: https://serwisy-lokalne.pl/api/v1
    description: Production
  - url: https://staging.serwisy-lokalne.pl/api/v1
    description: Staging
  - url: http://localhost:3001/api/v1
    description: Local Development

paths:
  # Authentication
  /auth/login:
    post:
      summary: Logowanie użytkownika
      tags: [Authentication]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginRequest'
      responses:
        200:
          description: Successful login
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LoginResponse'
        401:
          description: Invalid credentials
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /auth/refresh:
    post:
      summary: Odświeżenie tokenu
      tags: [Authentication]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                refresh_token:
                  type: string
      responses:
        200:
          description: Token refreshed
        401:
          description: Invalid refresh token

  # Domains
  /domains:
    get:
      summary: Lista domen
      tags: [Domains]
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        200:
          description: List of domains
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DomainListResponse'

    post:
      summary: Tworzenie nowej domeny
      tags: [Domains]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateDomainRequest'
      responses:
        201:
          description: Domain created
        403:
          description: Insufficient permissions

  /domains/{domainId}:
    get:
      summary: Szczegóły domeny
      tags: [Domains]
      security:
        - bearerAuth: []
      parameters:
        - name: domainId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        200:
          description: Domain details
        404:
          description: Domain not found

  # Posts
  /domains/{domainId}/posts:
    get:
      summary: Lista wpisów w domenie
      tags: [Posts]
      security:
        - bearerAuth: []
      parameters:
        - name: domainId
          in: path
          required: true
          schema:
            type: string
        - name: type
          in: query
          schema:
            type: string
            enum: [news, police, business, job, obituary]
        - name: status
          in: query
          schema:
            type: string
            enum: [draft, published, archived]
        - name: category
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        200:
          description: List of posts

    post:
      summary: Tworzenie nowego wpisu
      tags: [Posts]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreatePostRequest'
      responses:
        201:
          description: Post created
        422:
          description: Validation error

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    # Request/Response schemas
    LoginRequest:
      type: object
      required: [email, password]
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        remember_me:
          type: boolean

    LoginResponse:
      type: object
      properties:
        access_token:
          type: string
        refresh_token:
          type: string
        expires_in:
          type: integer
        user:
          $ref: '#/components/schemas/User'

    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
        first_name:
          type: string
        last_name:
          type: string
        roles:
          type: array
          items:
            type: string

    CreateDomainRequest:
      type: object
      required: [name, slug, city]
      properties:
        name:
          type: string
          example: "4Toruń"
        slug:
          type: string
          example: "4torun.pl"
        city:
          type: string
          example: "Toruń"
        admin_email:
          type: string
          format: email

    CreatePostRequest:
      type: object
      required: [title, content, post_type]
      properties:
        title:
          type: string
          maxLength: 200
        content:
          type: string
        excerpt:
          type: string
          maxLength: 500
        post_type:
          type: string
          enum: [news, police, business, job, obituary]
        status:
          type: string
          enum: [draft, published, scheduled]
        category_id:
          type: string
          format: uuid
        tags:
          type: array
          items:
            type: string
        featured_image:
          type: string
          format: uri

    DomainListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Domain'
        meta:
          $ref: '#/components/schemas/PaginationMeta'

    Domain:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        slug:
          type: string
        city:
          type: string
        is_active:
          type: boolean
        created_at:
          type: string
          format: date-time

    PaginationMeta:
      type: object
      properties:
        current_page:
          type: integer
        total_pages:
          type: integer
        total_items:
          type: integer
        items_per_page:
          type: integer
        has_next_page:
          type: boolean

    Error:
      type: object
      properties:
        error:
          type: string
        message:
          type: string
        code:
          type: string
        timestamp:
          type: string
          format: date-time
```

### 23.2 Swagger UI

#### Konfiguracja Express

```typescript
// app.ts
import swaggerUi from 'swagger-ui-express';
import YAML from 'yamljs';

const swaggerDocument = YAML.load('./openapi.yaml');

// Swagger UI dostępne pod /api-docs
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, {
  explorer: true,
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'Regionalne Serwisy API'
}));

// Raw OpenAPI spec
app.get('/api-spec.yaml', (req, res) => {
  res.sendFile(path.join(__dirname, 'openapi.yaml'));
});
```

### 23.3 Generowanie Kodu z OpenAPI

```bash
# Generowanie klienta TypeScript
npx openapi-typescript-codegen \
  --input ./openapi.yaml \
  --output ./src/api-client \
  --client axios

# Generowanie server stubs
npx openapi-generator-cli generate \
  -i ./openapi.yaml \
  -g typescript-node \
  -o ./src/api-stubs
```

---

## 24. TROUBLESHOOTING I FAQ 🔧

### 24.1 Częste Problemy - Baza Danych

#### Problem: "Connection refused" do PostgreSQL

**Objawy:**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Rozwiązania:**
1. Sprawdź czy PostgreSQL działa:
   ```bash
   sudo systemctl status postgresql
   sudo service postgresql status
   ```

2. Sprawdź konfigurację pg_hba.conf:
   ```bash
   sudo nano /etc/postgresql/15/main/pg_hba.conf
   # Upewnij się, że masz:
   # host all all 127.0.0.1/32 md5
   ```

3. Sprawdź port:
   ```bash
   sudo netstat -plunt | grep 5432
   ```

4. Restart usługi:
   ```bash
   sudo systemctl restart postgresql
   ```

#### Problem: "Schema does not exist"

**Objawy:**
```
Error: schema "tenant_4torun_pl" does not exist
```

**Rozwiązania:**
1. Utwórz schemat:
   ```sql
   CREATE SCHEMA tenant_4torun_pl;
   ```

2. Sprawdź search_path:
   ```sql
   SHOW search_path;
   SET search_path = tenant_4torun_pl, public;
   ```

3. Uruchom migracje:
   ```bash
   npx prisma migrate deploy
   ```

### 24.2 Częste Problemy - Scraping

#### Problem: "Connection timeout" przy scrapingu

**Rozwiązania:**
1. Zwiększ timeout w kodzie:
   ```python
   timeout = aiohttp.ClientTimeout(total=60)
   ```

2. Sprawdź czy strona docelowa działa:
   ```bash
   curl -I https://torun.policja.gov.pl
   ```

3. Sprawdź czy nie jesteś zablokowany:
   ```python
   # Dodaj rotację User-Agent
   headers = {'User-Agent': 'Mozilla/5.0...'}
   ```

4. Użyj proxy:
   ```python
   proxy = 'http://proxy:8080'
   async with aiohttp.ClientSession() as session:
       async with session.get(url, proxy=proxy) as resp:
           ...
   ```

#### Problem: "Rate limited" (429 Too Many Requests)

**Rozwiązania:**
1. Dodaj opóźnienie między requestami:
   ```python
   await asyncio.sleep(2)  # 2 sekundy
   ```

2. Zmniejsz częstotliwość cron:
   ```bash
   # Zamiast co 1h -> co 6h
   0 */6 * * * /usr/bin/python3 scraper.py
   ```

3. Użyj różnych IP (proxy rotating)

### 24.3 Częste Problemy - Frontend

#### Problem: "Hydration mismatch" w Next.js

**Rozwiązania:**
1. Upewnij się, że dane serwerowe i klienckie są identyczne
2. Użyj `suppressHydrationWarning` dla dat:
   ```tsx
   <time suppressHydrationWarning>{date}</time>
   ```
3. Użyj `useEffect` dla komponentów zależnych od window:
   ```tsx
   const [mounted, setMounted] = useState(false);
   useEffect(() => setMounted(true), []);
   if (!mounted) return null;
   ```

#### Problem: "Module not found" przy budowaniu

**Rozwiązania:**
1. Wyczyść cache:
   ```bash
   rm -rf node_modules .next
   npm install
   ```

2. Sprawdź case-sensitivity (Linux vs Windows):
   ```bash
   # Linux jest case-sensitive!
   import { Button } from './Button'  # ✓
   import { Button } from './button'  # ✗ na Linux
   ```

### 24.4 Częste Problemy - Deployment

#### Problem: "Permission denied" przy deploy

**Rozwiązania:**
1. Sprawdź uprawnienia:
   ```bash
   ls -la /home/host988956/domains/
   ```

2. Napraw uprawnienia:
   ```bash
   sudo chown -R host988956:host988956 /home/host988956/domains/
   sudo chmod -R 755 /home/host988956/domains/
   ```

#### Problem: "Port already in use"

**Rozwiązania:**
1. Znajdź proces:
   ```bash
   sudo lsof -i :3001
   ```

2. Zakończ proces:
   ```bash
   sudo kill -9 <PID>
   ```

3. Lub użyj innego portu w .env

### 24.5 Debugging

#### Logi Błędów

```bash
# Logi aplikacji
tail -f /home/host988956/domains/*/logs/error.log

# Logi systemowe
sudo journalctl -u postgresql -f
sudo journalctl -u nginx -f

# Logi PM2
pm2 logs

# Logi scrapera
tail -f /var/log/scraper.log
```

#### Narzędzia Debugowania

```bash
# Test API curl
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'

# Test bazy danych
psql -d regional_services -c "SELECT COUNT(*) FROM public.domains;"

# Test Redis
redis-cli ping

# Test RabbitMQ
rabbitmqctl status
```

### 24.6 FAQ

**Q: Jak dodać nową domenę?**
A: Użyj panelu centralnego (serwisy-lokalne.pl) → Domeny → Dodaj, lub skryptu:
```bash
./scripts/create-domain.sh nowadomena.pl "Nazwa Miasta"
```

**Q: Gdzie są logi?**
A: Logi aplikacji są w `/home/host988956/domains/{domain}/logs/`. Logi systemowe przez `journalctl`.

**Q: Jak zresetować hasło admina?**
A: 
```bash
psql -d regional_services -c "UPDATE users SET password_hash = '\$2b\$12\$...' WHERE email = 'admin@example.com';"
```

**Q: Jak wykonać backup ręcznie?**
A:
```bash
./scripts/backup.sh full
```

**Q: Jak przywrócić backup?**
A:
```bash
./scripts/restore.sh 20240212
```

**Q: Jak sprawdzić czy scraper działa?**
A:
```bash
pm2 status
# lub
curl http://localhost:3001/api/health
```

---

**KONIEC SEKCJI 23-24**


---

## 25. BACKUP I DISASTER RECOVERY 💾

### 25.1 Strategia Backupu

#### Klasyfikacja Danych

| Poziom | Dane | Częstotliwość Backupu | Retencja |
|--------|------|----------------------|----------|
| **Krytyczne** | Baza danych (public + tenants), Pliki użytkowników | Co 6h (inkrementalny), Codziennie 2:00 (pełny) | 30 dni pełnych, 7 dni inkrementalnych |
| **Ważne** | Konfiguracje, Logi, Statystyki | Codziennie | 90 dni |
| **Standardowe** | Cache, Temp files | Co tydzień | 7 dni |

#### Architektura Backupu

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ARCHITEKTURA BACKUPU                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         PRODUKCJA                                    │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ PostgreSQL   │  │ Redis        │  │ Pliki       │              │   │
│  │  │ (Primary)    │  │ (Cache)      │  │ (Uploads)   │              │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │   │
│  │         │                 │                 │                       │   │
│  │         └─────────────────┼─────────────────┘                       │   │
│  │                           ▼                                         │   │
│  │                  ┌────────────────┐                                │   │
│  │                  │ Backup Service │                                │   │
│  │                  │ (Cron + Scripts)│                                │   │
│  │                  └───────┬────────┘                                │   │
│  │                          │                                         │   │
│  └──────────────────────────┼─────────────────────────────────────────┘   │
│                             │                                              │
│                             ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    LOKALNE STORAGE                                   │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ /backup/full │  │ /backup/incr │  │ /backup/logs│              │   │
│  │  │ (7 dni)      │  │ (24h)        │  │ (90 dni)    │              │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │   │
│  │         │                 │                 │                       │   │
│  └─────────┼─────────────────┼─────────────────┼───────────────────────┘   │
│            │                 │                 │                            │
│            └─────────────────┼─────────────────┘                            │
│                              ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    ZEWNĘTRZNE STORAGE                               │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │   │
│  │  │ AWS S3       │  │ Backblaze B2 │  │ Google CS    │              │   │
│  │  │ (Glacier)    │  │ (B2)         │  │ (Nearline)   │              │   │
│  │  │ - Pełne      │  │ - Codzienne  │  │ - Kopie      │              │   │
│  │  │ - Miesięczne │  │ - Tygodniowe │  │ - Kwartalne  │              │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 25.2 PostgreSQL Backup

#### pg_dump per Schema

```bash
#!/bin/bash
# scripts/backup-postgres.sh

set -e

BACKUP_DIR="/backup/postgres/$(date +%Y%m%d)"
RETENTION_DAYS=30

# Tworzenie katalogu
mkdir -p "$BACKUP_DIR"

# Backup schematu public (centralnego)
echo "[$(date)] Backing up public schema..."
pg_dump \
    -h localhost \
    -U backup_user \
    -d regional_services \
    -n public \
    -Fc \
    -f "$BACKUP_DIR/public_$(date +%H%M).dump"

# Backup wszystkich schematów tenant
for schema in $(psql -h localhost -U backup_user -d regional_services -t -c "SELECT schema_name FROM information_schema.schemata WHERE schema_name LIKE 'tenant_%'"); do
    domain=$(echo "$schema" | sed 's/tenant_//; s/_/./g')
    echo "[$(date)] Backing up $schema ($domain)..."
    
    pg_dump \
        -h localhost \
        -U backup_user \
        -d regional_services \
        -n "$schema" \
        -Fc \
        -f "$BACKUP_DIR/${schema}_$(date +%H%M).dump" \
        2>/dev/null || echo "Warning: Failed to backup $schema"
done

# Kompresja
cd "$BACKUP_DIR"
tar czf "../postgres_$(date +%Y%m%d_%H%M).tar.gz" .
cd ..
rm -rf "$BACKUP_DIR"

# Usuń stare backupy (>30 dni)
find /backup/postgres -name "postgres_*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "[$(date)] PostgreSQL backup completed"
```

#### Continuous Archiving (WAL)

```ini
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/postgres/wal/%f'
max_wal_senders = 3
wal_keep_size = 1GB
```

#### Szyfrowanie Backupu

```bash
#!/bin/bash
# Szyfrowanie backupu GPG

gpg --symmetric \
    --cipher-algo AES256 \
    --compress-algo 1 \
    --output "$BACKUP_FILE.gpg" \
    "$BACKUP_FILE"

# Usuń nieszyfrowany plik
rm "$BACKUP_FILE"
```

### 25.3 File Backup

#### Rclone do Cloud Storage

```bash
#!/bin/bash
# scripts/backup-files.sh

# Konfiguracja rclone
# rclone config: name=backblaze, type=b2

# Upload plików
rclone sync \
    /home/host988956/domains \
    backblaze:regionalne-serwisy-backup/domains \
    --exclude ".git/**" \
    --exclude "node_modules/**" \
    --exclude "*.log" \
    --transfers 4 \
    --progress \
    --log-file /var/log/backup-files.log

# Upload bazy
rclone sync \
    /backup/postgres \
    backblaze:regionalne-serwisy-backup/postgres \
    --transfers 2

echo "[$(date)] File backup completed"
```

#### Monitoring Ważności Certyfikatów SSL

```bash
#!/bin/bash
# scripts/check-ssl-expiry.sh

DOMAINS=("serwisy-lokalne.pl" "4torun.pl" "4bydgoszcz.pl")
ALERT_DAYS=7

for domain in "${DOMAMAS[@]}"; do
    expiry_date=$(echo | openssl s_client -servername "$domain" -connect "$domain:443" 2>/dev/null | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
    expiry_timestamp=$(date -d "$expiry_date" +%s)
    current_timestamp=$(date +%s)
    days_until_expiry=$(( (expiry_timestamp - current_timestamp) / 86400 ))
    
    if [ $days_until_expiry -lt $ALERT_DAYS ]; then
        echo "ALERT: SSL certificate for $domain expires in $days_until_expiry days!" | mail -s "SSL Expiry Alert" admin@example.com
    fi
done
```

### 25.4 Weryfikacja Backupu

#### Automatyczny Test Restore

```bash
#!/bin/bash
# scripts/test-restore.sh (uruchamiany co tydzień)

TEST_DB="test_restore_$(date +%s)"
BACKUP_FILE=$(ls -t /backup/postgres/postgres_*.tar.gz | head -1)

echo "Testing restore from: $BACKUP_FILE"

# Odtworzenie do testowej bazy
createdb "$TEST_DB"
pg_restore -d "$TEST_DB" -j 4 "$BACKUP_FILE"

# Weryfikacja integralności
psql -d "$TEST_DB" -c "SELECT COUNT(*) FROM public.domains;"
psql -d "$TEST_DB" -c "SELECT COUNT(*) FROM public.users;"

# Cleanup
dropdb "$TEST_DB"

echo "[$(date)] Restore test completed successfully"
```

### 25.5 Disaster Recovery Plan

#### Cele RTO/RPO

| Metryka | Cel | Opis |
|---------|-----|------|
| **RTO** (Recovery Time Objective) | 4h | Maksymalny czas przywrócenia systemu |
| **RPO** (Recovery Point Objective) | 6h | Maksymalna utrata danych (ostatni backup) |

#### Scenariusze Awarii

**Scenariusz 1: Awaria Bazy Danych**
```
1. Automatyczny failover do replica (jeśli skonfigurowane)
2. Ręczne przełączenie aplikacji na replica
3. Diagnostyka primary node
4. Naprawa lub odbudowa primary
5. Powrót do normalnej pracy
```

**Scenariusz 2: Uszkodzenie Danych (Logical)**
```
1. Wstrzymanie zapisów (maintenance mode)
2. Identyfikacja punktu w czasie przed awarią
3. PITR (Point-in-Time Recovery)
4. Weryfikacja danych
5. Wznowienie pracy
```

**Scenariusz 3: Awaria Całego Serwera**
```
1. Uruchomienie nowego serwera (z template)
2. Odtworzenie konfiguracji z backupu
3. Odtworzenie bazy danych
4. Odtworzenie plików
5. Aktualizacja DNS
6. Weryfikacja
```

### 25.6 Procedury Recovery

#### Full Recovery Procedure

```bash
#!/bin/bash
# scripts/full-recovery.sh

set -e

BACKUP_DATE="$1"  # YYYYMMDD
if [ -z "$BACKUP_DATE" ]; then
    echo "Usage: $0 <backup_date>"
    exit 1
fi

echo "🚨 STARTING FULL RECOVERY FROM $BACKUP_DATE"

# 1. Zatrzymaj aplikacje
pm2 stop all
echo "✓ Applications stopped"

# 2. Odtworzenie bazy
echo "Restoring database..."
pg_restore -d regional_services "/backup/postgres/postgres_${BACKUP_DATE}_0200.tar.gz"
echo "✓ Database restored"

# 3. Odtworzenie plików
echo "Restoring files..."
tar xzf "/backup/files/files_${BACKUP_DATE}.tar.gz" -C /
echo "✓ Files restored"

# 4. Migracje (jeśli potrzeba)
npx prisma migrate deploy
echo "✓ Migrations applied"

# 5. Restart usług
pm2 start all
systemctl restart nginx
echo "✓ Services restarted"

# 6. Weryfikacja
curl -sf http://localhost/health || exit 1
echo "✓ Health check passed"

echo "🎉 Recovery completed successfully!"
```

#### Point-in-Time Recovery (PITR)

```bash
#!/bin/bash
# Odtworzenie do konkretnego momentu

RECOVERY_TIME="2024-02-12 14:30:00"

# 1. Przygotowanie
mkdir -p /tmp/recovery

# 2. Odtworzenie base backup
pg_basebackup -D /tmp/recovery -Ft -z -P

# 3. Konfiguracja recovery
 cat > /tmp/recovery/recovery.conf << EOF
restore_command = 'cp /backup/postgres/wal/%f %p'
recovery_target_time = '$RECOVERY_TIME'
recovery_target_action = 'promote'
EOF

# 4. Start PostgreSQL w trybie recovery
pg_ctl -D /tmp/recovery start

# 5. Export danych i import do produkcji
pg_dump -d "host=/tmp/recovery dbname=regional_services" -n tenant_4torun_pl | psql -d regional_services
```

### 25.7 Runbook - Procedura Awaryjna

```
╔═══════════════════════════════════════════════════════════════════════════════╗
║                         RUNBOOK - DISASTER RECOVERY                           ║
╚═══════════════════════════════════════════════════════════════════════════════╝

KROK 1: OCENA SYTUACJI (5 min)
├── Sprawdź monitoring (Grafana/Datadog)
├── Sprawdź logi błędów
├── Określ zakres awarii (baza, aplikacja, infrastruktura)
└── Zdecyduj: FAILOVER vs REPAIR

KROK 2: KOMUNIKACJA (5 min)
├── Powiadom zespół (Slack #incidents)
├── Powiadom klientów (jeśli wymagane)
└── Uruchom status page (status.serwisy-lokalne.pl)

KROK 3: FAILOVER (jeśli replika dostępna) (15 min)
├── Przełącz DNS na replica
├── Aktywuj replica jako primary
└── Weryfikacja działania

KROK 4: RECOVERY (jeśli brak repliki) (2-4h)
├── Uruchom nowy serwer (z template)
├── Odtwórz bazę z backupu
├── Odtwórz pliki
└── Przełącz DNS

KROK 5: WERYFIKACJA (15 min)
├── Health check wszystkich usług
├── Test logowania
├── Test głównych funkcji
└── Sprawdź logi błędów

KROK 6: POST-INCIDENT (24h)
├── Raport z awarii
├── Root cause analysis
├── Aktualizacja procedur
└── Test przywracania

KONTAKTY EMERGENCY:
├── Tech Lead: +48 XXX XXX XXX
├── DevOps: +48 XXX XXX XXX
├── Dostawca Hosting: +48 XXX XXX XXX
└── Dostawca Cloud: support@cloudprovider.com
```

### 25.8 Automatyzacja Backupu

#### Główny Skrypt Orchestrator

```bash
#!/bin/bash
# scripts/backup.sh - Główny skrypt backupu

set -e

BACKUP_TYPE="${1:-incremental}"  # full, incremental, all
TIMESTAMP=$(date +%Y%m%d_%H%M)
LOG_FILE="/var/log/backup-${TIMESTAMP}.log"

exec 1> >(tee -a "$LOG_FILE")
exec 2>&1

echo "[$(date)] Starting $BACKUP_TYPE backup..."

# Funkcja powiadomień
notify() {
    local status="$1"
    local message="$2"
    
    # Slack
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"Backup $status: $message\"}" \
        "$SLACK_WEBHOOK_URL" 2>/dev/null || true
    
    # Email (jeśli błąd)
    if [ "$status" = "FAILED" ]; then
        echo "$message" | mail -s "Backup Failed" admin@example.com
    fi
}

# Wykonaj backup
if [ "$BACKUP_TYPE" = "full" ] || [ "$BACKUP_TYPE" = "all" ]; then
    echo "Running PostgreSQL full backup..."
    /scripts/backup-postgres.sh || { notify "FAILED" "PostgreSQL backup failed"; exit 1; }
    
    echo "Running file backup..."
    /scripts/backup-files.sh || { notify "FAILED" "File backup failed"; exit 1; }
fi

if [ "$BACKUP_TYPE" = "incremental" ]; then
    echo "Running incremental backup..."
    /scripts/backup-incremental.sh || { notify "FAILED" "Incremental backup failed"; exit 1; }
fi

# Weryfikacja
echo "Verifying backup..."
/scripts/verify-backup.sh || { notify "FAILED" "Backup verification failed"; exit 1; }

# Cleanup
echo "Cleaning up old backups..."
/scripts/cleanup-old-backups.sh

echo "[$(date)] Backup completed successfully"
notify "SUCCESS" "Backup $BACKUP_TYPE completed at $TIMESTAMP"
```

#### Cron Configuration

```bash
# /etc/cron.d/regionalne-serwisy-backup

# Pełny backup codziennie o 2:00
0 2 * * * root /home/host988956/scripts/backup.sh full

# Inkrementalny backup co 6h
0 */6 * * * root /home/host988956/scripts/backup.sh incremental

# Test restore co niedzielę
0 6 * * 0 root /home/host988956/scripts/test-restore.sh

# Sprawdzenie SSL co dzień
0 9 * * * root /home/host988956/scripts/check-ssl-expiry.sh
```

---

**KONIEC SEKCJI 25 - BACKUP I DISASTER RECOVERY**

---

## ZAKOŃCZENIE DOKUMENTACJI

*Dokumentacja kompletna - wszystkie sekcje zostały zaktualizowane i rozszerzone.*

*Wersja: 4.0 (Final)*  
*Data: 12 lutego 2026*  
*Autor: System Architect + AI Agents*  
*Status: Ready for Implementation*

