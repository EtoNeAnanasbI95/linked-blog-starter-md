# TanStack Table: Библиотека для создания таблиц

[**TanStack Table**](https://tanstack.com/table/latest/docs/introduction) — headless библиотека для создания таблиц и датагридов, предоставляющая логику для сортировки, фильтрации, пагинации и других функций, с полной свободой в стилизации. Она поддерживает React, Vue, Solid, Svelte.

## Особенности

- **Headless API**: Только логика (сортировка, фильтрация, пагинация), без стилей.
- **Кросс-фреймворковая поддержка**: React, Vue, Solid, Svelte и др.
- **Типобезопасность**: Полная поддержка TypeScript.
- **Виртуализация**: Поддержка больших данных через `@tanstack/virtual`.
- **Модульность**: Используйте только нужные функции.
- **Гибкость**: Динамические колонки, вложенные данные, кастомные фильтры.

## Преимущества

- **По сравнению с AG Grid**: Нет встроенного UI, меньше бандл, больше свободы.
- **По сравнению с react-table**: Кросс-фреймворковая, улучшенная типизация.
- **По сравнению с самописными таблицами**: Экономия времени на логику.

## Пример: Простая таблица с сортировкой (React)

```typescript
import {
  useReactTable,
  getCoreRowModel,
  getFilteredRowModel,
  getSortedRowModel,
  getGroupedRowModel,
  getExpandedRowModel,
} from '@tanstack/react-table'
import { useState } from 'react';

interface Contact {
  attachment: boolean
  category: string
  organization: string
  name: string
  surname: string
  patranymic: string
  email: string
  workPhone: string
  post: string
  prefix: string
  suffix: string
  storeAs: string
}

export const data: Contact[] = [
  {
    category: 'Клиент',
    organization: 'ООО ТехноПром',
    name: 'Александр',
    surname: 'Козлов',
    patranymic: 'Михайлович',
    email: 'akozlov@technoprom.ru',
    workPhone: '+79991234567',
    post: 'Технический директор',
    prefix: 'г-н',
    suffix: '',
    storeAs: 'Козлов А.М.',
  },
  {
    category: 'Партнёр',
    organization: 'ООО ТехноПром',
    name: 'Екатерина',
    surname: 'Соколова',
    patranymic: 'Андреевна',
    email: 'esokolova@technoprom.ru',
    workPhone: '+79992345678',
    post: 'Менеджер по продажам',
    prefix: 'г-жа',
    suffix: '',
    storeAs: 'Соколова Е.А.',
  },
]

const columns: ColumnDef<Contact>[] = [
  {
    header: () => <Attach20Filled />,
    accessorKey: 'attachment',
    enableSorting: true,
    size: 20,
  },
  {
    header: 'Категория',
    accessorKey: 'category',
    enableSorting: true,
  },
  {
    header: 'Организация',
    accessorKey: 'organization',
    enableSorting: true,
  },
  {
    header: 'Фамилия Имя Отчество',
    accessorFn: (row) => {
      const parts = [row.surname, row.name, row.patranymic].filter(Boolean)
      return parts.join(' ')
    },
    enableSorting: true,
  },
  {
    header: 'Электронная почта',
    accessorKey: 'email',
    enableSorting: true,
  },
  {
    header: 'Рабочий телефон',
    accessorKey: 'workPhone',
    enableSorting: true,
  },
  {
    header: 'Должность',
    accessorKey: 'post',
    enableSorting: true,
  },
]

export default function DataTable() {
  const table = useReactTable({
    data: data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getGroupedRowModel: getGroupedRowModel(),
    getExpandedRowModel: getExpandedRowModel(),
    enableColumnResizing: true,
    enableSorting: true,
    columnResizeMode: 'onChange',
    groupedColumnMode: false,
    initialState: {
      expanded: true,
      grouping: ['organization'],
    },
  })

  return (
    <table className="w-full shrink border-separate border-spacing-0">
      <thead>
        {table.getHeaderGroups().map((headerGroup) => (
          <tr key={headerGroup.id}>
            {headerGroup.headers.map((header) => (
              <th
                key={header.id}
                className="relative"
                style={{
                  width: header.getSize(),
                }}
              >
                <button
                  className="flex w-full cursor-pointer items-center"
                  onClick={header.column.getToggleSortingHandler()}
                >
                  <span className="text-neutral-foreground-1-rest font-semibold">
                    {flexRender(
                      header.column.columnDef.header,
                      header.getContext(),
                    )}
                  </span>
                  {header.isPlaceholder ? null : (
                    <>
                      {{
                        asc: (
                          <ChevronUp16Filled
                            className={'!transition-transform'}
                          />
                        ),
                        desc: (
                          <ChevronUp16Filled
                            className={'rotate-180 !transition-transform'}
                          />
                        ),
                      }[header.column.getIsSorted() as string] ?? ''}
                    </>
                  )}
                </button>
                <button
                  onMouseDown={header.getResizeHandler()}
                  onTouchStart={header.getResizeHandler()}
                  className="bg-neutral-stroke-1-rest absolute top-0 right-0 h-full w-2 cursor-col-resize opacity-0 transition-opacity hover:opacity-100"
                />
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <React.Fragment key={row.id}>
            {row.getIsGrouped() ? (
              <tr className="bg-neutral-background-2-hover">
                <td colSpan={columns.length}>
                  <button
                    className="flex cursor-pointer items-center font-semibold"
                    onClick={() => row.toggleExpanded()}
                  >
                    <ChevronUp12Filled
                      className={`text-neutral-foreground-2-rest mx-[6px] my-[10px] !transition-transform ${row.getIsExpanded() ? 'rotate-180' : ''}`}
                    />
                    <span className="text-neutral-foreground-2-rest font-semibold">
                      {row.groupingValue as React.ReactNode}
                    </span>
                  </button>
                </td>
              </tr>
            ) : null}
            {row.getIsExpanded() &&
              row.subRows.map((subRow) => (
                <tr
                  key={subRow.id}
                  className="hover:bg-neutral-background-2-hover cursor-pointer"
                  onDoubleClick={() => setActiveContact(subRow.original)}
                >
                  {subRow.getVisibleCells().map((cell) => (
                    <td
                      key={cell.id}
                      style={{
                        width: cell.column.getSize(),
                      }}
                    >
                      <span className="text-neutral-foreground-1-rest">
                        {cell.getValue() as React.ReactNode}
                      </span>
                    </td>
                  ))}
                </tr>
              ))}
          </React.Fragment>
        ))}
      </tbody>
    </table>
  );
}
```

## Инициализация

1. **Установка**:

```bash
npm install @tanstack/react-table
```

2. **Конфигурация**:

- Определите данные и колонки.
- Используйте `useReactTable` с нужными моделями.
- Рендерьте таблицу в HTML или компонентах фреймворка.

3. **Кросс-фреймворковая работа**:

- Vue: `@tanstack/vue-table` с `useVueTable`.
- Solid: `@tanstack/solid-table` с `createSolidTable`.
- Аналогичные API для других фреймворков.

## Итог

TanStack Table — универсальная библиотека для таблиц, которая подходит для всех фреймворков. Её headless API и хуки упрощают работу с данными, сохраняя гибкость. Подробности: [документация TanStack Table](https://tanstack.com/table/v8).