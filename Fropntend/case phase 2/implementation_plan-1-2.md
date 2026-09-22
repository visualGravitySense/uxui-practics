# Реализация компонента Perspectives (Feature Cards)

## Описание задачи
Создать новый компонент "Feature Cards / Perspectives" на основе HTML-прототипа (`feature-cards-perspectives.html`) с адаптацией под стек Kursify (React, Next.js, Tailwind, Framer Motion) и стилистику бренда (glassmorphism, цвета `#1668E3`, `#0D1B2A`, `#F1F6FD`). Компонент должен получать данные из Sanity CMS и быть доступен в Page Builder на Уровне 1 (секции профилей инструктора/школы) и Уровне 2 (маркетинговый Page Builder).

## Пользовательский обзор обязателен (User Review Required)

> [!WARNING]
> Для добавления секции `perspectivesBlock` в Уровень 1 (секции профиля инструктора и школы), потребуется обновить `instructorSections.ts` и `schoolSections.ts`. Также, возможно, потребуется добавить поля напрямую в схемы `instructor.ts` и `school.ts` (или они будут добавляться внутрь `customBlocks` массива? Уточнение ниже).

## Открытые вопросы (Open Questions)

> [!IMPORTANT]
> 1. **Уровень 1 (Секции профиля)**: Должен ли `perspectivesBlock` быть отдельной секцией в `instructorSections.ts` / `schoolSections.ts` (как `promise`, `stats`) с собственным полем в схеме документа Sanity, **ИЛИ** он должен быть добавлен как один из типов блоков в массив `customBlocks` (Уровень 3: `customProfileBlocks.ts`), где уже есть `customText`, `customImage`? (Из вашего описания кажется, что он должен быть отдельной секцией для Уровня 1 и 2, но я хочу уточнить архитектуру).
> 2. У вас есть отдельная папка для React компонентов блоков (например, `src/components/blocks/`)? Где вы предпочитаете разместить код самого компонента?
> 3. Иконки или изображения в карточках: вы предложили поле `image (Sanity image)`. Должно ли это быть всегда изображение, или нам стоит поддерживать иконки (например, Lucide/Heroicons), передаваемые строкой?

## Предлагаемые изменения

### Sanity Schemas (CMS)

#### [NEW] `sanity/schemas/objects/perspectivesBlock.ts`
Создать схему для отдельной карточки (`perspectiveCard`) и для самого блока (`perspectivesBlock`), который содержит массив карточек и общие настройки (например, заголовок секции, подзаголовок).
Структура карточки, как вы указали:
- `image` (image)
- `badgeNumber` (string)
- `title` (string)
- `subtitle` (string)
- `bulletPoints` (array of string)
- `escalationLine` (string)
- `footerTag` (string)

#### [MODIFY] Схемы профилей (`sanity/schemas/instructor.ts` и `school.ts` и/или `pageBuilderDraft.ts`)
Добавить поддержку нового блока `perspectivesBlock` в зависимости от ответов на вопросы выше.

---

### React Component

#### [NEW] `src/components/.../PerspectivesBlock.tsx`
Реализовать UI-компонент, который:
- Принимает данные из Sanity.
- Использует Tailwind CSS для стилей.
- Поддерживает Light / Dark mode с правильными цветами.
- Реализует адаптивную сетку (3 колонки на десктопе, 2+1 на планшете, 1 на мобильных).
- Добавляет glassmorphism-эффекты из прототипа.

---

### Page Builder Configs

#### [MODIFY] `src/lib/pageBuilderBlockCatalog.ts`
Добавить `perspectivesBlock` в каталог блоков маркетингового пейдж-билдера (Уровень 2).

#### [MODIFY] `src/lib/instructorSections.ts` и `src/lib/schoolSections.ts`
Добавить `perspectivesBlock` в массив констант секций для отображения и скрытия.

## План тестирования (Verification Plan)

### Ручное тестирование
1. Открыть Sanity Studio, убедиться, что можно создать и наполнить `perspectivesBlock`.
2. Добавить блок на страницу инструктора или школы, проверить корректное отображение.
3. Переключить темную/светлую тему, проверить контрастность и цвета.
4. Изменить размер окна браузера (desktop, tablet, mobile), убедиться в корректности сетки (3 -> 2+1 -> 1).
