# Конспект по локализации в Next.js с next-intl

## Библиотека next-intl

[`next-intl`](https://next-intl.dev) — библиотека для интернационализации (i18n) Next.js приложений, упрощающая локализацию благодаря глубокой интеграции, типобезопасности и поддержке всех типов рендеринга (SSR, SSG, ISR, RSC).

### Особенности

- **Интеграция с Next.js**: Поддержка App Router, Pages Router и React Server Components с автоматической маршрутизацией по локалям.
- **Типобезопасность**: Автодополнение и проверка ключей переводов с TypeScript.
- **Хуки и серверные API**: `useTranslations`, `useFormatter` и `getTranslations` для удобной работы с переводами.
- **ICU Message Syntax**: Поддержка интерполяции, множественных чисел и rich text.
- **Форматирование дат и чисел**: Автоматическая локализация с учётом часовых поясов.
- **Маршрутизация**: Локализация путей (например, `/en/about`, `/de/ueber-uns`).
- **Производительность**: Оптимизация для SSG и Server Components.
- **Интеграция с TMS**: Совместимость с Crowdin для управления переводами.
#### Важный момент
* Эта библа поддерживает два вида перевода, по роутингу типа `http://your-web-app/ru/home.site` или же просто использование перевода по факту, с определением языка через заголовки.

> [!WARNING] ВАЖНО
> ДАЛЬНЕЙШЕЕ ОПИСАНИЕ СДЕЛАНО ДЛЯ ПЕРЕВОДА САЙТА ПО ЗАГОЛОВКАМ ЗАПРОСА, НЕ ПО РОУТИНГУ. Для перевода по роутингу смотрите документацию


### Преимущества над react-intl

- Нативная поддержка Next.js маршрутизации.
- Меньший размер клиентского бандла за счёт Server Components.
- Простая конфигурация и встроенная поддержка TypeScript.

### Инициализация

#### Шаги

1. **Установка**:
    
    ```bash
    npm install next-intl
    ```
    
    
2. **Файлы переводов**:  
    Создайте JSON-файлы в `messages`, например:
    
    ```json
{
  "Home": {
    "title": "Welcome to my app",
    "description": "This is a multilingual Next.js app"
  }
}
    ```
    
3. **Настройка Server Components**:  
    Создайте `i18n/request.ts`:
    
    ```typescript
import { getRequestConfig } from 'next-intl/server'
import { cookies } from 'next/headers'

const RTL_LANGS = []
const VERIFIED_LANGS = []
export const LANG_NAMES = {
	ru: 'Russian',
}

export const getLocale = async () => {
  const cookieStore = await cookies()
  const cookieLocale = cookieStore.get('locale')?.value

  if (cookieLocale && VERIFIED_LANGS.includes(cookieLocale)) return cookieLocale
  return 'en'
}

export const isRTL = (locale: string) => {
  const lang = locale.split('-')[0]
  return RTL_LANGS.includes(lang)
}

export default getRequestConfig(async () => {
  const locale = await getLocale()

  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default,
  }
})
    ```
    
6. **Интеграция в layout**:  
    В `app/[locale]/layout.tsx`:
    
    ```typescript
import { NextIntlClientProvider } from 'next-intl'
import { getLocale } from 'next-intl/server'
import { isRTL } from '@/i18n/request'

const RootLayout: React.FC<Readonly<{ children: React.ReactNode }>> = async ({
  children,
}) => {
  const locale = await getLocale()
  const isRTLang = isRTL(locale)

  return (
    <html lang={locale} dir={isRTLang ? 'rtl' : 'ltr'}>
      <body>
        <NextIntlClientProvider>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  )
}
    ```
    
7. **Использование переводов**:  
    В компонентах:
    
    ```typescript
'use client';
import { useTranslations } from 'next-intl';

export default function HomePage() {
  const t = useTranslations('Home');
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}
    ```
    

#### Описание

- **Структура проекта**:
    
    ```
    web-app/
    ├── src/
	│   ├── app/
	│   │   └── layout.tsx
	│   ├── i18n/
	│   │   └── request.ts
    ├── messages/
    │   ├── en.json
    │   ├── ru.json
    ```
    
- **Подробности**: См. [официальную документацию next-intl](https://next-intl.dev/).