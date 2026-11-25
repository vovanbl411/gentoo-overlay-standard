# Справочник по автоматизированным обновлениям

[Принятый Standard v0.1](https://github.com/vovanbl411/gentoo-overlay-standard/blob/becf8221ff745d722eb74c0e91435088c40b9ac7/STANDARD.md)
задаёт критерии. Справочник показывает,
как существующие jobs и скрипты покрывают эти стадии и что остаётся
проверить при внедрении. Наименования jobs, скриптов, outputs и полей
артефактов принадлежат оверлею.

Для discovery подходят `schedule` и `workflow_dispatch`, как в
аудированных workflow. Интервалы и входы ручного запуска выбирает проект.
У всех шести автоматизированных потоков Phase 0 update workflow имел
свою concurrency group с `cancel-in-progress: false`. Это помогает
упорядочить запуски. Перед записью всё равно проверяются удалённая база,
ветки и PR. CI репозитория действует отдельно от обновлений.
[Наблюдения Phase 0](../audits/baseline.md#автоматизация-и-границы-публикации)

| Стадия и ID | Сопоставление с реализациями |
| --- | --- |
| Discover · A1, R1–R2 | Noctalia вызывает `watch_noctalia_release.py` в prepare job, проверяет релиз и provenance. Umbriel читает snapshot и lock со связанным компонентом. Luciole совмещает discovery и staging. Исходы и причины остановки выражаются средствами проекта. |
| Prepare · A2, P1 | Read-only prepare job создаёт временную копию. Noctalia ротирует версии и обновляет README, Umbriel готовит согласованную пару. Zed совмещает discovery и локальную подготовку в `check`. Записи в удалённое состояние репозитория делает контур публикации. |
| Validate · A3, V1–V2 | У Noctalia и Umbriel применимые Gentoo-проверки идут в prepare. Zed сверяет исходники и crates, Manifest, патч и resolver. Readiness относится к итоговому кандидату и появляется после обязательных предпубликационных проверок. Успешный workflow с пропущенным шагом не подтверждает этот шаг. |
| Handoff · A4, H1–H2 | Zed использует отдельный read-only job и два артефакта. Luciole передаёт ID попытки и checksums. Umbriel проверяет состав полученных файлов пары и применимый diff. Форматы и доказательства различаются. |
| Publish · A5, P1–P4, R3 | Привилегированный job использует код из доверенной ревизии и принимает handoff после проверки. В Luciole публикация ограничена repo/ref и проверяет remote base, ветку и открытый PR. У Zed отдельно публикуются или переиспользуются distfile assets. |
| Accept · A6, V1–V2, G1–G3 | Владелец проверяет точный PR head, целевую ветку, diff, обязательные checks и ручные сценарии проекта. Принятие происходит через ручной merge. Для PR от `GITHUB_TOKEN` учитывается approval запусков проверок. |

## Передача и запись

При приёме handoff сверяются запуск или попытка, база, кандидат,
provenance, состав и целостность артефакта, разрешённые типы и пути, а
также относящиеся к кандидату доказательства. Публикующий job берёт правила из
доверенной ревизии. По локальной политике проверяются итоговый diff и
состояние пакетов. Переданные файлы не исполняются как код публикации.

Перед первой удалённой записью P2 требует сверить текущий SHA удалённой
целевой ветки с базой проверенного handoff. SHA запуска, локальный checkout
и ref ветки кандидата отвечают на разные вопросы. Допуск ограничен
разрешёнными repo/ref и успешной проверкой. Dry-run может выполнять
чтение, подготовку и проверки, а удалённые записи исключены.

При конфликтующем PR или ветке запись останавливается либо применяется
проверенная локальная процедура. Luciole переиспользует ветку с тем же
деревом и родительским коммитом. Noctalia использует lease для
automation branch, а право заменить обнаруженное состояние проверяется
отдельно по P3. Draft PR открывает кандидата для ревью. Если после
частичного сбоя остались ветка или assets, сопровождающий сверяет
завершённый шаг и состояние перед повтором или ручным восстановлением.

## Доказательства и границы

| Точная ревизия | Что показывает исходный код | Что проверить при внедрении |
| --- | --- | --- |
| Noctalia `cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c` | Разделены prepare и publish. Handoff сверяется с checkout `main`, diff ограничен ротацией, Manifest и README. Lease привязан к ref ветки кандидата. [Workflow][n-workflow] | Явный допуск repo/ref и отдельная сверка удалённой `main` до push по P2. При lease проверить принадлежность ветки и отсутствие блокирующего PR по P3. Подтвердить связь проверенного архива с коммитом по R1. |
| Umbriel `87878e3b8a96e35361833f2385249bba6b1f41a2` | Snapshot пары из lock, проверка состава и diff, повторная проверка PR и сопоставление дерева и родителя для повторного использования ветки. [Workflow][u-workflow] | Допуск repo/ref по P2 и доказуемая связь полученных файлов пары с результатом Gentoo-проверок по H1–H2. |
| Luciole `4fae56beb67b2a8c47c3427162c36fce823bbaf3` | Явные repo/ref, проверка handoff текущей попытки, remote base и состояния ветки/PR. Gentoo gate ограничен amd64 и default USE с `-test`. [Workflow][l-workflow] · [Publish][l-publish] | Подтвердить фактическую защиту принятой ветки и required checks по G1–G3. Доказательства gate относятся к его области. Meson, полная сборка и runtime проверяются отдельно по локальной политике. |
| Zed `3a19639d2b097be7bf4c65f0fc0e6a094d45244f` | Раздельные handoff и assets, проверка целостности и повторного использования distfile release. PR-скрипт проверяет конфликтующие PR и ветку. [Workflow][z-workflow] · [Assets][z-assets] · [PR][z-pr] | Сверять удалённую `main` и выполнить проверки конфликтов и точного diff до первой записи, включая distfile release (P2–P3, H2). Учитывать, что release может сохраниться после сбоя PR. |

Таблица описывает реализацию на аудированных SHA. Для фактически
выполненных, пропущенных и неизвестных проверок используйте
[историю запусков Phase 0](../audits/baseline.md#проверки-и-подтверждённые-запуски).
Работы при внедрении перечислены в [сверке стандарта](../STANDARD.md#сверка-с-phase-0).

[n-workflow]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/workflows/noctalia-release-watcher.yml
[u-workflow]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/.github/workflows/prepare-umbriel-snapshot.yml
[l-workflow]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/.github/workflows/prepare-luciole-snapshot.yml
[l-publish]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/scripts/publish_luciole.py
[z-workflow]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.github/workflows/zed-release-watcher.yml
[z-assets]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/crates_github.py
[z-pr]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/ci/publish-release-pr.sh
