# ТЗ и пошаговый план рефакторинга и улучшения существующего UI-компонента FAQ (Kkk.tsx)

> **Существующий компонент:** [`src/components/Kkk.tsx`](file:///c:/Users/Admin/Documents/kursify/src/components/Kkk.tsx)  
> **UX-макет практиканта:** [`doc/Internship/kursify_FAQ.html`](file:///c:/Users/Admin/Documents/kursify/doc/Internship/kursify_FAQ.html)  
> **Цель:** Обновить имеющийся компонент `Kkk.tsx` детализированными UX-улучшениями из HTML-макета (стекломорфизм, анимация через CSS Grid, a11y, полная поддержка `faqPairs` автошкол из Sanity CMS).

---

## 🔍 Сравнение: Что у нас есть в `Kkk.tsx` vs Что добавил практикант в HTML-макет

| Элемент | Текущий `Kkk.tsx` | Улучшения из `kursify_FAQ.html` | Задача рефакторинга |
| :--- | :--- | :--- | :--- |
| **Анимация открытия** | `max-h-[1200px]` (возможна задержка закрытия) | **CSS Grid (`grid-template-rows: 0fr ➔ 1fr`)** | Заменить `max-height` на независящую от высоты анимацию CSS Grid |
| **Кнопка переключателя** | Простые линии в `div` | **Glass Button (`.glass-btn`)** с бликом и Plus/Minus | Перенести верстку стеклянной круглой кнопки `+ / -` |
| **Доступность (a11y)** | Отсутствуют атрибуты `aria` | Полный набор `aria-expanded`, `aria-controls`, `role="region"` | Добавить идентификаторы и ARIA-состояния для скриноплайнеров |
| **Поддержка данных** | Только `kkkData` для посадочных страниц | Добавить поддержку `schoolName`, `faqPairs`, `lastUpdated` | Сделать компонент универсальным (и для посадочных страниц, и для автошкол) |
| **Стекломорфизм** | Базовый `backdrop-blur-[20px]` | **Кастомные платформы с градиентной подсветкой** | Обновить Tailwind-классы бордеров и свечения при открытии |

---

## 🛠️ Пошаговый план изменений в [`src/components/Kkk.tsx`](file:///c:/Users/Admin/Documents/kursify/src/components/Kkk.tsx)

### Шаг 1. Расширение типов и пропсов

Компонент должен принимать либо существующий `kkkData`, либо прямой массив `faqPairs` от конкретной автошколы:

```typescript
export interface FAQItem {
  _key?: string
  question: string
  /** Текст ответа (string) от бота или PortableText блоки из Sanity */
  answer: unknown
  category?: string
}

export interface KkkProps {
  /** Структура для общих посадочных страниц */
  kkkData?: {
    title: string
    eyebrow: string
    subtitle?: string
    showEyebrowDots?: boolean
    faqs: FAQItem[]
  }
  /** Прямые данные автошколы из Sanity */
  schoolName?: string
  faqPairs?: FAQItem[]
  lastUpdated?: string
  className?: string
}
```

---

### Шаг 2. Замена анимации `max-height` на `CSS Grid`

В текущем `Kkk.tsx` (строка 101):
```tsx
// ❌ Старое решение с задержкой закрытия длинных блоков:
className={`transition-all duration-500 ease-in-out ${isOpen ? 'max-h-[min(1200px,72vh)] opacity-100' : 'max-h-0 opacity-0 overflow-hidden'}`}
```

Заменить на решение из макета практиканта:
```tsx
// ✅ Новое плавное решение через CSS Grid:
<div
  id={panelId}
  role="region"
  aria-labelledby={buttonId}
  className={`grid transition-all duration-300 ease-out ${
    isOpen ? 'grid-template-rows-[1fr] opacity-100 mt-2' : 'grid-template-rows-[0fr] opacity-0'
  }`}
>
  <div className="overflow-hidden">
    <div className="px-5 sm:px-6 pb-6 pt-2 text-sm leading-relaxed text-[rgb(var(--text-secondary))] border-t border-black/5 dark:border-white/5">
      <AnswerBody answer={faq.answer} />
    </div>
  </div>
</div>
```

---

### Шаг 3. Внедрение Glass Button (`.glass-btn`) из HTML-макета

Заменить блок иконки (строки 95-98 в `Kkk.tsx`) на стеклянную кнопку с абсолютным позиционированием микро-блика:

```tsx
<div className="relative w-9 h-9 min-w-[36px] rounded-full flex items-center justify-center bg-blue-500/10 dark:bg-white/10 border border-blue-500/20 dark:border-white/15 text-blue-700 dark:text-slate-100 transition-all duration-200">
  {/* Декоративный блик хрусталя */}
  <span className="absolute top-[2px] left-[12%] right-[12%] h-[44%] rounded-full bg-gradient-to-b from-white/70 to-transparent pointer-events-none" />

  {/* Иконка Плюс / Минус */}
  <svg 
    className={`w-4 h-4 transition-transform duration-200 ${isOpen ? 'rotate-45 text-blue-600 dark:text-blue-400' : 'rotate-0'}`} 
    fill="none" 
    viewBox="0 0 24 24" 
    stroke="currentColor" 
    strokeWidth={2.5}
  >
    <path strokeLinecap="round" strokeLinejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
  </svg>
</div>
```

---

### Шаг 4. Обновление компонента `AnswerBody`

Убедиться, что `AnswerBody` одинаково хорошо рендерит как обычный текст от бота (`string`), так и PortableText из CMS:

```tsx
function AnswerBody({ answer }: { answer: unknown }) {
  if (typeof answer === 'string') {
    const t = answer.trim()
    if (!t) return null
    return <p className="whitespace-pre-wrap text-slate-700 dark:text-slate-300 font-normal leading-relaxed">{t}</p>
  }
  if (Array.isArray(answer) && answer.length > 0) {
    return (
      <div className="prose prose-sm prose-slate max-w-none text-slate-700 dark:text-slate-300 dark:prose-invert">
        <PortableText value={answer as never} />
      </div>
    )
  }
  return null
}
```

---

### Шаг 5. Добавление блока актуализации (Дата сканирования)

Если проп `lastUpdated` передан, добавить под списком вопросов плашку:

```tsx
{lastUpdated && (
  <div className="mt-6 text-center text-xs font-medium text-slate-400 dark:text-slate-500">
    Информация FAQ актуализирована через авто-скрейпер: {new Date(lastUpdated).toLocaleDateString('et-EE')}
  </div>
)}
```

---

## 🧪 Чек-лист проверки обновлённого компонента для практиканта

- [ ] **Сохранение обратной совместимости:** Все существующие вызовы `<Kkk kkkData={...} />` на главной и других страницах продолжают работать без ошибок.
- [ ] **Работа с автошколами:** Компонент корректно принимает `<Kkk schoolName={school.name} faqPairs={school.faqPairs} lastUpdated={school.faqLastScrapedAt} />`.
- [ ] **Тестирование CSS Grid:** Открытие и закрытие вопросов не вызывает задержек при разной длине текста ответа.
- [ ] **Доступность:** При наведении и навигации клавишей `Tab` виден чёткий фокус (`focus-visible:ring-2 focus-visible:ring-blue-500`).
- [ ] **Проверка `dark` режима:** Текст и фон платформ читаемы в обеих темах.

---

## 🚀 Фаза 2: Интеграция FAQ в Page Builder (Dashboard)

Чтобы инструкторы и автошколы могли самостоятельно добавлять и редактировать блоки FAQ на своих страницах, компонент будет интегрирован в профильный конструктор (`/dashboard/profile` и `/dashboard/school`).

### 1. Расширение типов Custom Blocks (`src/lib/customProfileBlocks.ts`)
- Добавить `'customFaq'` в массив `CUSTOM_BLOCK_TYPES`.
- Создать интерфейс `CustomFaqBlockData`:
  ```typescript
  export interface CustomFaqBlockData {
    _type: 'customFaq'
    _key: string
    heading?: string
    faqPairs: { question: string; answer: string; _key: string }[]
  }
  ```
- Обновить функции валидации (`validateCustomBlock`) и инициализации пустых блоков (`createEmptyCustomBlock`), чтобы поддерживать массив Q&A.

### 2. Обновление дашборда (редактора профиля)
- В `CustomBlocksEditor.tsx` добавить UI для типа `customFaq`:
  - Текстовое поле для заголовка блока (опционально, например "Частые вопросы").
  - Динамический список пар "Вопрос - Ответ" с возможностью добавления (`+ Lisa küsimus`), удаления и изменения порядка (drag & drop или кнопки вверх/вниз).
  - Поля ввода для вопроса (input) и ответа (textarea).

### 3. Рендеринг FAQ на публичной странице (`CustomBlocksList.tsx`)
- В компоненте `CustomBlockCard` добавить `case 'customFaq'`.
- Рендерить обновлённый компонент `<Kkk>` внутри карточки:
  ```tsx
  case 'customFaq':
    if (!block.faqPairs || block.faqPairs.length === 0) return null
    return (
      <div className="rounded-2xl border border-slate-200/60 dark:border-white/10 bg-white/50 dark:bg-white/[0.03] p-6 sm:p-8">
        {block.heading && (
          <h3 className="text-xl font-bold text-slate-900 dark:text-white mb-6 text-center">
            {block.heading}
          </h3>
        )}
        {/* Используем наш новый компонент FAQ */}
        <Kkk faqPairs={block.faqPairs} className="max-w-none" />
      </div>
    )
  ```

### 4. Обновление Sanity Schema (CMS)
- Добавить тип `customFaq` в схему `customBlocks` (если массив блоков синхронизируется напрямую с Sanity).
- В Sanity для `mentorType` (и `schoolType`) добавить поддержку структуры Q&A внутри массива кастомных блоков.

---

## ❓ Открытые вопросы (Требуется подтверждение)

> [!IMPORTANT]  
> 1. Нужно ли ограничивать максимальное количество вопросов в блоке FAQ для одного инструктора (например, не больше 10)?
> 2. Стоит ли добавить возможность перетаскивать (drag & drop) вопросы внутри блока в дашборде, или достаточно простых кнопок "Вверх / Вниз"?
> 3. Текст ответов в Page Builder будет обычным текстом (plain text) или нужно подключать Rich Text Editor (PortableText)? (Обычный текст реализовать быстрее).
