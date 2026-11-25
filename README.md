# Gentoo Overlay Standard

Этот репозиторий хранит общие инженерные требования для Gentoo-оверлеев
`vovanbl411`. Каждый оверлей остаётся самостоятельным репозиторием и
определяет собственную политику упаковки и принятое состояние.

Действует [Standard v0.1](https://github.com/vovanbl411/gentoo-overlay-standard/blob/becf8221ff745d722eb74c0e91435088c40b9ac7/STANDARD.md).
[Проект Standard v0.2](STANDARD.md) добавляет требование S3 к
сопровождаемости и ждёт отдельного решения владельца.

## Phase 0. Инвентаризация и аудит завершены

[Базовый аудит](https://github.com/vovanbl411/gentoo-overlay-standard/blob/638130a268da300cda9cd5ac3abb6e053691b409/audits/baseline.md) принят в коммите
`638130a268da300cda9cd5ac3abb6e053691b409`. В нём зафиксированы ревизии и
результаты сравнения Zed, Noctalia, Umbriel, Luciole, Spotatui, ASUS NumberPad
и Coucou. Vicinae и Spotify Player архивированы и исключены из миграции.

## Phase 1. Принятие стандарта

Владелец принял Standard v0.1 на ревизии
`becf8221ff745d722eb74c0e91435088c40b9ac7`.

## Phase 2. Справочные шаблоны

В этой фазе подготовлены [шаблоны](templates/README.md),
[чек-лист внедрения](templates/adoption-checklist.md) и
[справочник по обновлениям](docs/update-workflow-reference.md).
[Проверка расположения тестов](audits/test-layout.md) фиксирует результаты
и границы эксперимента. Внедрение в существующих оверлеях требует отдельного
решения владельца для каждого проекта. [Правила работы](AGENTS.md)

## Основание проекта v0.2

Техническая миграция Noctalia вошла в `main` на ревизии
`14b374edae214d15876f27c60b246d48f193a57c` через [PR #25](https://github.com/vovanbl411/noctalia-overlay/pull/25).
Документация этой ревизии связывает стадии подготовки и публикации
с заданиями workflow и скриптами и поясняет, почему некоторые проверки
пропускаются. На этом опыте основан критерий S3 для автоматизации.
Сопровождающий должен иметь возможность проследить путь кандидата
и определить причину остановки.
[Документация Noctalia на ревизии миграции](https://github.com/vovanbl411/noctalia-overlay/blob/14b374edae214d15876f27c60b246d48f193a57c/docs/CI_AND_SECURITY.md)

Полная hosted candidate validation и окончательная приёмка пилота
остаются открытыми.
