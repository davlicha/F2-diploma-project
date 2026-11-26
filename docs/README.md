# Документація кваліфікаційного проєкту

Навігаційний дашборд документації за принципом Documentation as Code.

## Зміст пояснювальної записки

| Файл | Розділ |
| --- | --- |
| [0_intro.md](./0_intro.md) | Вступ та науковий апарат |
| [1_analysis.md](./1_analysis.md) | Розділ 1. Аналіз предметної області та постановка задачі |
| [2_design.md](./2_design.md) | Розділ 2. Проєктування архітектури системи |
| [3_implementation.md](./3_implementation.md) | Розділ 3. Реалізація, тестування та розгортання |
| [4_conclusions.md](./4_conclusions.md) | Висновки |

## Супровідні документи

- [Thesis Proposal](./proposal.md)
- [Software Requirements Specification (ISO/IEC/IEEE 29148)](./srs.md)
- [Architecture Decision Records](./adr/)
  - [ADR-001: Вибір фреймворку оркестрування агентів](./adr/0001-agent-orchestration-framework.md)

## Архітектура (C4 Model)

- [Рівень 1: System Context](./2_design.md#діаграма-рівня-1-system-context)
- [Рівень 2: Container](./2_design.md#діаграма-рівня-2-container)
- [Рівень 3: Component — Agent Orchestrator](./2_design.md#діаграма-рівня-3-component-agent-orchestrator)

## Дані, контракти та алгоритми

- [2.4. Модель даних (ERD, PostgreSQL 16 + pgvector)](./2_design.md#24-проєктування-та-моделювання-баз-даних)
- [2.5. API-контракт](./2_design.md#25-специфікація-міжсервісних-контрактів-api) — [openapi.yaml](./api/openapi.yaml)
- [2.6. Алгоритми критичних вузлів](./2_design.md#26-моделювання-логіки-критичних-вузлів)
- [2.7. Інтеграційний аудит (Production Readiness Report)](./2_design.md#27-інтеграційний-аудит-та-верифікація-цілісності-доменів)

## Календарний план

| № з/п | Назва етапу кваліфікаційної роботи | Термін виконання | Статус |
| --- | --- | --- | --- |
