# История стандарта и состояние внедрения

- [Базовый аудит Phase 0](https://github.com/vovanbl411/gentoo-overlay-standard/blob/638130a268da300cda9cd5ac3abb6e053691b409/audits/baseline.md)
  фиксирует семь активных оверлеев на аудированных ревизиях. Vicinae и
  Spotify Player архивированы и исключены из внедрения.
- [Принятый Standard v0.1](https://github.com/vovanbl411/gentoo-overlay-standard/blob/becf8221ff745d722eb74c0e91435088c40b9ac7/STANDARD.md)
  владелец утвердил по итогам Phase 1. В Phase 2 появились справочные
  шаблоны и [эксперимент с расположением тестов](../audits/test-layout.md).
  Сканирование всего оверлея с тестами под `scripts/` в этом эксперименте
  не добавило предупреждений о категории.
- [Предыдущий проект v0.2][previous] добавил S3 и критерии сопровождаемости
  на основе технической миграции Noctalia. [Прежний справочник][old-guide]
  описывает автоматизацию по аудированным SHA, а [каталог шаблонов на
  той же ревизии][old-templates] перечисляет использованные источники.

[previous]: https://github.com/vovanbl411/gentoo-overlay-standard/blob/b90ec059d72fd5d55e9b0f1e47ba11acac7089b0/STANDARD.md
[old-guide]: https://github.com/vovanbl411/gentoo-overlay-standard/blob/b90ec059d72fd5d55e9b0f1e47ba11acac7089b0/docs/update-workflow-reference.md
[old-templates]: https://github.com/vovanbl411/gentoo-overlay-standard/blob/b90ec059d72fd5d55e9b0f1e47ba11acac7089b0/templates/README.md#аудированные-источники

## Пилот Noctalia

Проверено по публичным данным GitHub 2026-11-26 UTC.

- **Техническая миграция v0.1.** [PR #25][pr25] слит в `main` на
  ревизии `14b374edae214d15876f27c60b246d48f193a57c`. В [CI после merge][ci25]
  фактически выполнился и прошёл шаг `Run Python unit tests`.
- **Сопровождаемость.** Проверенная ветка `feat/human-maintainability`
  имела HEAD `364d76dd12069e357d081f7904a476b31eb999d8`.
  [PR #26][pr26] слит 2026-11-26 в 00:42 UTC. [Текущая на дату
  проверки `main`][main-api] указывала на merge-коммит
  `45706256dc877fc13e76b0019fb6081dcd07c2fc`. Проверенный HEAD входит
  в него вторым родителем, а дерево обоих коммитов совпадает
  (`224ce718f52141757451c92521af09390c20142f`). [Объект merge-коммита][merge]
  · [Разбор принятой реализации](update-workflow-reference.md#пример-noctalia).
- **Repository CI.** В [CI для PR #26][ci26-pr] на проверенном HEAD и
  [CI после merge][ci26-main] на SHA `main` шаг `Run Python unit tests`
  фактически выполнился и прошёл. Это GitHub Actions на `ubuntu-24.04`,
  команда `PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -v`
  из корня оверлея. Она находит Python-тесты в `tests/`, показывает
  результаты по тестам и отключает запись `.pyc`.
  [Конфигурация CI][ci-source]
- **Hosted candidate validation — не подтверждена.** Публичный список
  запусков [релизного workflow][hosted] не показал ни одного запуска на
  ревизии миграции, проверенном HEAD или merge SHA. В [историческом
  запуске 45663950280][old-run] на `cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c`
  Prepare завершился успешно, но ротация, Manifest и `pkgcheck`, создание
  и загрузка handoff, readiness и Publish были пропущены. Обычный CI
  подтверждает unit-тесты. Для пилота по-прежнему нужны результаты
  проверок реального hosted-кандидата на точных базе и содержимом, а
  для принятия кандидата — применимые проверки владельца.
- **Effective GitHub settings — подтверждены частично.** Публичные
  [Protect main][ruleset] и [правила ветки][rules] показывали активный
  ruleset для default branch, запрет удаления и force-push, PR с
  разрешением обсуждений и нулём обязательных одобрений, актуальную
  базу и required check `Python unit tests` от интеграции `15368`.
  Legacy protection, bypass, дополнительные ограничения и эффективные
  права Actions, включая создание Draft PR и одобрение его проверок,
  полностью не подтверждены. Ожидаемые значения описаны в
  [эксплуатационном справочнике][operations]. Наблюдение за обычным PR не
  доказывает прохождение проверки на automation Draft PR или блокировку
  merge при отсутствующем либо упавшем check.
- **Приёмка по S2 открыта.** Владелец отдельно фиксирует версию, профиль,
  проверенные требования, доказательства, применимые исключения и решение
  по оверлею. Слияние #25 и #26 подтверждает принятие изменений в
  `main`. Подтверждение полного внедрения и отдельное решение об
  утверждении проекта v0.2 остаются открытыми.

[pr25]: https://github.com/vovanbl411/noctalia-overlay/pull/25
[pr26]: https://github.com/vovanbl411/noctalia-overlay/pull/26
[main-api]: https://api.github.com/repos/vovanbl411/noctalia-overlay/branches/main
[merge]: https://api.github.com/repos/vovanbl411/noctalia-overlay/git/commits/45706256dc877fc13e76b0019fb6081dcd07c2fc
[ci25]: https://github.com/vovanbl411/noctalia-overlay/actions/runs/45717502586
[ci26-pr]: https://github.com/vovanbl411/noctalia-overlay/actions/runs/45730557426
[ci26-main]: https://github.com/vovanbl411/noctalia-overlay/actions/runs/45730616435
[ci-source]: https://github.com/vovanbl411/noctalia-overlay/blob/45706256dc877fc13e76b0019fb6081dcd07c2fc/.github/workflows/ci.yml
[hosted]: https://github.com/vovanbl411/noctalia-overlay/actions/workflows/noctalia-release-watcher.yml
[old-run]: https://github.com/vovanbl411/noctalia-overlay/actions/runs/45663950280
[ruleset]: https://github.com/vovanbl411/noctalia-overlay/rules/23505842
[rules]: https://api.github.com/repos/vovanbl411/noctalia-overlay/rules/branches/main
[operations]: https://github.com/vovanbl411/noctalia-overlay/blob/45706256dc877fc13e76b0019fb6081dcd07c2fc/docs/CI_AND_SECURITY.md

## Сверка с Phase 0

На 2026-11-25 UTC `main` был default branch у всех семи оверлеев.
В четырёх репозиториях публично подтверждались rulesets и их required
checks. Для трёх публичный список rulesets был пуст, а источник Pages
не подтвердился. Legacy protection, дополнительные checks, права Actions
и bypass остались UNKNOWN. DRIFT в аудите означает расхождение с
действовавшим на тот момент соглашением.
[Настройки на дату аудита](../audits/baseline.md#документация-и-настройки-github)

Профили в предыдущем проекте соответствовали рассмотренным потокам.
Coucou использовал ручное обновление, Noctalia, Spotatui и ASUS NumberPad
готовили релизы, Umbriel и Luciole работали со снимками, а Zed отдельно
публиковал distfiles. Эти примеры привязаны к SHA ниже.

Форматы handoff различались. Zed передавал два артефакта и использовал
отдельную read-only проверку. Noctalia передавала provenance и ротацию,
Umbriel — файлы согласованной пары, Luciole — ID попытки и checksum,
Spotatui — план ротации, ASUS NumberPad — Manifest с повторным построением.
[Доказательства Phase 0](../audits/baseline.md#автоматизация-и-границы-публикации)

Сверка охватывает [семь активных аудированных SHA](../audits/baseline.md#ревизии-и-метод).
Для каждого оверлея внедрение требует отдельного принятия по S2,
проверки полноты D2–D4 и L1 и подтверждения применимых G1–G3. Настройки
и неизвестные результаты проверок требуют доказательств при внедрении.

### Zed

`3a19639d2b097be7bf4c65f0fc0e6a094d45244f`

Release gate с Portage Manifest, сверкой источников и resolver, read-only
handoff и отдельным distfile release. Сохранение старых версий и Wayland-only
допустимы. Handoff проверяется по SHA запуска (`GITHUB_SHA`) и при применении
сверяется с SHA локального checkout `main` (`git rev-parse HEAD`).
`refs/heads/automation/zed-…` — ref ветки кандидата.

**Вопросы на момент сверки.** По P2 сверять текущий SHA удалённого
`refs/heads/main` с базой проверенного handoff до первой удалённой записи,
включая создание distfile release. В аудированной ревизии свежесть базы
проверяется на момент checkout. Отдельной сверки удалённой целевой ветки перед
записью нет. Перенести проверку конфликтов и точного PR diff до первой
удалённой записи по H2/P3. На этой ревизии GitHub Release с distfiles может
появиться до этих проверок в PR-скрипте. Подтвердить G2 для `Overlay policy` и
уточнить текущую версию в checkpoint. Полная сборка и runtime для 1.29.3 и
arm64 не подтверждены.

### Noctalia

`cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c`

Manifest, `pkgcheck`, проверка ротации и RU/EN README, локальный virtual
без distfiles и Manifest, lease-обновление ветки. Локальная политика допускает отдельные категории
неподписанных тегов. В handoff записывается SHA запуска (`GITHUB_SHA`), при
применении он сверяется с SHA локального checkout `main` (`git rev-parse
HEAD`). Lease использует удалённый ref ветки кандидата
`refs/heads/automation/noctalia-…`.

**Вопросы на момент сверки.** Закрепить P2 для разрешённых repo/ref. До первой
удалённой записи (push ветки кандидата) сверять текущий SHA удалённого
`refs/heads/main` с базой проверенного handoff. В аудированной ревизии перед
push отдельно сверяется только ref ветки кандидата. Перед lease-update
подтвердить принадлежность ветки и повторно проверить блокирующий PR по P3.
Подтвердить по R1, что байты архива по тегу относятся к разрешённому коммиту.
Прохождение условных Gentoo-шагов на проверенном scheduled run и runtime
текущего релиза неизвестны.

### Umbriel

`87878e3b8a96e35361833f2385249bba6b1f41a2`

Связанная пара из lock, Manifest, resolver, проверка файлов пары и идентичной
ветки.

**Вопросы на момент сверки.** Закрепить P2 явным контролем repo/ref в Publish.
Подтвердить H1/H2 для связи содержимого переданных файлов пары с результатом
Gentoo-проверок. Уточнить документированные resolver overrides по действующему
сценарию. Полная сборка и runtime текущей пары неизвестны.

### Luciole

`4fae56beb67b2a8c47c3427162c36fce823bbaf3`

Точные repo/ref, handoff по ID попытки, сверка tree и родителя при повторе.
Gate для amd64 и default USE с `-test`.

**Вопросы на момент сверки.** Добавить `AGENTS.md`. Подтвердить effective
protection и выбранные required checks. Сборка, Meson и Wayland runtime
остаются отдельными сценариями.

### Spotatui

`b00245be9dbe3adbee039428dd4e1cb5daedc55b`

Ротация, сверка CRATES/Manifest и scan кандидата. Тесты в `scripts/` допустимы
при документированной команде.

**Вопросы на момент сверки.** Добавить `AGENTS.md` и закрепить P2. Дополнить
R1 доказательством связи релиза и проверенного архива с неизменяемым
источником. Уточнить инструкцию об approval для PR и локальную политику
tracked cache. Шаг подготовки кандидата в [запуске 45648443992][spotatui-run]
завершился с кодом 1. Загрузка handoff и Publish были пропущены. HTTP 404 получен при
извлечении лога шага. Причина сбоя кандидата остаётся UNKNOWN. Сборка 0.48.0 и
результаты для расширенного USE неизвестны.

### ASUS NumberPad

`a0a9b4277f6b5f9823666014914df7210814d908`

Handoff с Manifest и повторной подготовкой. Проверка patch и Python compile,
ручное обновление `python-periphery`. README документирует один ручной тест
7.3.0 на ASUE1410:01 04F3:3230 с `b7402`, Python 3.14 и Umbriel.

**Вопросы на момент сверки.** Добавить `AGENTS.md`, ограничить Publish по P2 и
решить, требуется ли `pkgcheck` для merge помимо `test`. Полная сборка,
отдельный shutdown-сценарий и другие устройства не подтверждены этим тестом.

### Coucou

`b4710550a048e7b15e653699be95c1ebf00f1a68`

Manual, pinned upstream commit, условия дистрибуции artwork, сведения о
частичной offline-сборке.

**Вопросы на момент сверки.** Добавить `AGENTS.md`, дополнить README путём
установки и обновления и описать состояние отката. Подтвердить ручные проверки
и защиту принятого состояния. Полный ebuild, Gentoo runtime и лицензионный
инвентарь native archive остаются открытыми.

Дополнительная сверка использовала исходники по аудированным SHA.
Порядок записи Zed виден в [workflow][zed-workflow] и [PR-скрипте][zed-pr-script].
Noctalia проверяет [tag и VERSION][noctalia-watcher], а [workflow][noctalia-workflow]
использует свежий remote ref для lease. Umbriel передаёт [файлы и состояние пары][umbriel-handoff]
и применяет их к checkout целевой ветки. Spotatui получает [архив по тегу][spotatui-archive]
и сверяет версию пакета и Manifest.

## Вопросы, зафиксированные в предыдущем проекте

Предыдущий проект v0.2 оставлял решение о принятии владельцу.
Вопросы ниже относятся к аудированным ревизиям. Перед решением о
внедрении нужно сверить их с текущим состоянием. Для Noctalia новую
реализацию и оставшиеся шаги фиксирует [состояние пилота](#пилот-noctalia).

1. После принятия v0.1 определить порядок отдельного внедрения по оверлеям и допустимые локальные исключения.
2. Выбрать обязательные checks и режим защиты для каждого репозитория, включая `pkgcheck` ASUS NumberPad. Подтвердить effective settings, bypass и способ одобрения automation PR-проверок.
3. Подтвердить границы допустимого восстановления веток, в том числе lease Noctalia, и локальные сценарии build/runtime до Accept. Дорогие Gentoo-прогоны и аппаратные проверки нужны только там, где этого требует политика принятия.
4. Для Spotatui отдельно согласовать выяснение причины сбоя подготовки кандидата (код 1) и определить роль tracked `metadata/md5-cache`.

[zed-workflow]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.github/workflows/zed-release-watcher.yml#L182-L231
[zed-pr-script]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/ci/publish-release-pr.sh
[noctalia-watcher]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/scripts/watch_noctalia_release.py#L347-L430
[noctalia-workflow]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/workflows/noctalia-release-watcher.yml#L222-L247
[umbriel-handoff]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/umbriel_handoff.py
[spotatui-archive]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/scripts/release_automation.py#L383-L397
[spotatui-run]: https://github.com/vovanbl411/spotatui-overlay/actions/runs/45648443992/job/139411464088#step:6:3467
