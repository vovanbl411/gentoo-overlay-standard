# Phase 0. Базовый аудит Gentoo-оверлеев

## Ревизии и метод

На 2026-11-25 18:33:36 UTC все девять репозиториев были публично доступны.
Ветка и архивация взяты из GitHub REST API, точный HEAD — из
`git ls-remote --symref https://github.com/vovanbl411/<repo>.git HEAD`.
Таблица фиксирует общее время начала этой сверки. Настройки и Actions
прочитаны в том же сеансе 25 ноября UTC.

| Репозиторий | Default | Полный SHA | Архив | Профиль для сравнения |
| --- | --- | --- | --- | --- |
| [zed-overlay](https://github.com/vovanbl411/zed-overlay) · [API][zed-api] | main | [`3a19639d2b097be7bf4c65f0fc0e6a094d45244f`][zed-tree] | нет | Extended release |
| [noctalia-overlay](https://github.com/vovanbl411/noctalia-overlay) · [API][noctalia-api] | main | [`cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c`][noctalia-tree] | нет | Release |
| [umbriel-overlay](https://github.com/vovanbl411/umbriel-overlay) · [API][umbriel-api] | main | [`87878e3b8a96e35361833f2385249bba6b1f41a2`][umbriel-tree] | нет | Snapshot |
| [luciole-overlay](https://github.com/vovanbl411/luciole-overlay) · [API][luciole-api] | main | [`4fae56beb67b2a8c47c3427162c36fce823bbaf3`][luciole-tree] | нет | Snapshot |
| [spotatui-overlay](https://github.com/vovanbl411/spotatui-overlay) · [API][spotatui-api] | main | [`b00245be9dbe3adbee039428dd4e1cb5daedc55b`][spotatui-tree] | нет | Release |
| [asus-numberpad-overlay](https://github.com/vovanbl411/asus-numberpad-overlay) · [API][asus-numberpad-api] | main | [`a0a9b4277f6b5f9823666014914df7210814d908`][asus-numberpad-tree] | нет | Release |
| [coucou-overlay](https://github.com/vovanbl411/coucou-overlay) · [API][coucou-api] | main | [`b4710550a048e7b15e653699be95c1ebf00f1a68`][coucou-tree] | нет | Manual |
| [vicinae-overlay][vicinae-api] | main | `20be3e8caaa2714dd629b897f0e34ae1c2550af4` | да | вне аудита исходников |
| [spotify-player-overlay][spotify-player-api] | main | `fd449a08bde8e7338e5277439da6322bbf73e94b` | да | вне аудита исходников |

Ссылки на деревья ведут в репозитории GitHub по указанному SHA. Для ASUS
NumberPad и Coucou SHA совпал с локальным HEAD, файлы прочитаны через
`git archive <sha>`. Для остальных пяти получены точные коммиты в
изолированные временные Git-репозитории. Профили в таблице служат для
сравнения. В задании Phase 0 они имеют статус предварительных категорий.

Исходное состояние `gentoo-overlay-standard` — пустая ветка `main` без
коммитов и файлов в `git status --porcelain`. `origin` указывает на
`git@github.com:vovanbl411/gentoo-overlay-standard.git`, локальная ссылка
`origin/main` отсутствовала. По завершении три разрешённых файла остаются
неотслеживаемыми. HEAD по-прежнему без коммита, индекс пуст. Рабочие деревья оверлеев использованы только
для чтения. Временные копии и логи проверок находятся вне репозитория стандарта.

## Состав и политика упаковки

У всех семи `metadata/layout.conf` задаёт `masters = gentoo` и
`thin-manifests = true`. `profiles/repo_name` совпадает с именем
репозитория. В прочитанных Manifest только записи `DIST` с `BLAKE2B` и
`SHA512`. У каждого пакета есть `metadata.xml`. Почти везде указан upstream
`remote-id`, а у виртуального пакета Noctalia описано локальное назначение.
Обычные пакеты используют EAPI 8 и `~amd64`, у Zed дополнительно `~arm64`.
Виртуальный пакет Noctalia сохраняет EAPI 7 и свой набор KEYWORDS.

| Оверлей | Пакеты, версии и откат | Источники, совместимость и ограничения |
| --- | --- | --- |
| Zed | `app-editors/zed` содержит 1.25.3, 1.26.2, 1.27.0, 1.28.1, 1.29.2, 1.29.3. Подготовка добавляет релиз без ротации. Сохранённые ebuild доступны для выбора точной версии. | Stable GitHub tag, версия привязана к Wayland-only patch в `files/`. Архив исходников сверяется с деревом Git по точному commit. Есть отдельный overlay crates archive, Git crates и WebRTC pin с разными файлами для amd64 и arm64. Патч исключает X11 и локальный захват экрана. [Пакет][zed-pkg] · [Prepare][zed-prepare] · [Проверка исходников][zed-candidate] |
| Noctalia | `gui-apps/noctalia` 5.6.0 и fallback 5.5.0. Ротация удаляет прежний fallback и обновляет таблицы RU/EN. `virtual/notification-daemon-0-r1` сопровождается отдельно и обходится без Manifest, поскольку не скачивает distfiles. | Stable `vX.Y.Z`, tag → commit → сверка `VERSION`. Annotated tag проверяется по данным GitHub verification. Если GitHub сообщает `verified=true`, скрипт сверяет signed payload. Лёгкие теги и непротиворечивые ответы с `verified=false` допускаются. Изменения трёх packaging-файлов отмечаются для ревью. Ebuild скачивает архив по тегу. Локальный virtual добавляет Noctalia как провайдера уведомлений и сохраняет остальные ветви зависимостей. [Пакет][noctalia-pkg] · [Virtual][noctalia-virtual] · [Watcher][noctalia-watch] · [Ротация][noctalia-prepare] |
| Umbriel | `gui-wm/umbriel` 0.1.0_pre20261125122920 / 0.1.0_pre20261123014939 и `gui-libs/xdg-desktop-portal-umbriel` 0.1.0_p20261124204031 / 0.1.0_p20261122051915. Откат к согласованной паре `previous` из `automation/accepted-pairs.json`. При совпадении portal SHA ebuild используется повторно. | Разрешает `cachix` в полный SHA, читает portal SHA из `flake.lock` того же коммита. Оба `SRC_URI` закреплены по commit. `screencast` в Umbriel требует portal. Состояние совместимости управляется парой и точными атомами в инструкциях. [Umbriel][umbriel-pkg] · [Portal][umbriel-portal] · [Пары][umbriel-state] · [Watcher][umbriel-watch] · [Prepare][umbriel-prepare] |
| Luciole | `gui-apps/luciole` 0.1.0_pre20261125155229 / 0.1.0_pre20261124203731. Сохраняется предыдущий принятый ebuild и его distfile. | Полный SHA из upstream `main`, `VERSION` и UTC committer timestamp. Проверяет прямое продвижение истории, сравнивает packaging tree. Gentoo gate ограничен default USE с `-test`. Сборке нужен C++23 и подходящая стандартная библиотека, сеансу — Wayland/EGL. Тесты Meson имеют отдельные требования к Glycin. [Пакет][luciole-pkg] · [Prepare][luciole-prepare] · [README][luciole-readme] |
| Spotatui | `media-sound/spotatui` 0.48.0 и fallback 0.47.0-r1. Ротация сохраняет ревизию текущего ebuild как fallback. Только здесь обнаружены `profiles/categories`, `repo-name` в layout и tracked `metadata/md5-cache`. | Stable GitHub tag, Rust `CRATES` из `Cargo.lock` и отдельные записи для source/crates. Проверяются выбранные Cargo features, native deps, лицензии и интеграции на предмет смены контракта. USE mapping и ALSA backend сохраняются, musl исключён, Spotify требует Premium. [Пакет][spotatui-pkg] · [Профили][spotatui-profiles] · [Cache][spotatui-cache] · [Скрипт][spotatui-release] · [README][spotatui-readme] |
| ASUS NumberPad | `sys-apps/asus-numberpad-driver-7.3.0` и `dev-python/python-periphery-2.4.1`. Fallback драйвера пока отсутствует. При следующем bump остаются текущий ebuild и shutdown patch, при третьем удаляется более старый вариант. `python-periphery` из PyPI обновляется вручную. | Stable tag драйвера, сверка commit и архива. Проверка upstream сравнивает license, requirements, installer/Nix, packaging и статические imports. В `files/` лежат shutdown patch и OpenRC/systemd service-файлы. Доступ к устройствам и проверка на реальном оборудовании остаются ручными. [Драйвер][asus-numberpad-pkg] · [Python][asus-numberpad-python] · [Скрипт][asus-numberpad-release] · [README][asus-numberpad-readme] |
| Coucou | Один экспериментальный `gui-apps/coucou-0.5.9`, автоматической ротации и второго ebuild в дереве нет. | Linux release `linux-v0.5.9` закреплён полным commit `74363634b037108393e450428e5f207515c2c29b`. Cargo/npm lockfiles, локальный npm cache без lifecycle hooks и prebuilt static sherpa-onnx 1.13.8. `RESTRICT="mirror bindist test"`, отдельные условия artwork в `licenses/coucou-assets`. Полный лицензионный инвентарь native archive открыт. [Пакет][coucou-pkg] · [README][coucou-readme] · [Лицензия][coucou-license] |

## Автоматизация и границы публикации

Шесть оверлеев используют GitHub Actions для push/PR и отдельный
`schedule` с `workflow_dispatch` для кандидатов. Все jobs выполняются на
`ubuntu-24.04`. Проверенные workflow используют полные SHA для
`actions/checkout` (`3d3c42e5aac5ba805825da76410c181273ba90b1`),
`upload-artifact` (`cf430e030ddbb5b0abf93d22962f4752f3646cd9`) и
`download-artifact` (`9000827ccba6bdab643e8b6fd33ac0654aef8333`).
Checkout использует `persist-credentials: false`. CI имеет `contents: read`.
Update workflows задают собственную concurrency group с
`cancel-in-progress: false`. Запуск с записью ждёт готовый handoff.
На уровне workflow права пустые, кроме Zed с `contents: read`. Подготовка
имеет `contents: read` и обычно `pull-requests: read` (у Zed только
`contents: read`). Publish имеет `contents: write` и `pull-requests: write`.
PR открывается как Draft, принятие остаётся ручным. Наличие такой логики
в YAML фиксирует права запрашиваемого токена, а не состояние настроек GitHub.

| Оверлей и entry point | Расписание UTC, job graph и timeout в минутах | Discover → Prepare → Validate |
| --- | --- | --- |
| Zed [watcher][zed-workflow] | 06:17, `check → validate-release-handoff → publish-draft-release-pr`, publish также зависит от check. 90 / — / —. `independent_validation` разрешает проверку без публикации. | `validate-release-candidate.sh` вызывает `release_handoff.py --prepare`, объединяя stable release lookup и копирование ebuild/patch. Дальше `independent-crates.sh` генерирует или переиспользует архив, сверяет tag, исходники, lock, Manifest, patch и Portage resolver. No-update даёт `handoff_ready=false`. Validation-only проводит полный gate только при наличии нового релиза. [Orchestrator][zed-orchestrator] · [Candidate][zed-candidate] · [Watch][zed-watch] |
| Noctalia [watcher][noctalia-workflow] | 06:17, `prepare → publish`, 20 / 10, `dry_run`. | `watch_noctalia_release.py` → JSON/summary/outputs. `release_handoff.py copy-workspace` → копия без `.git`, `prepare_noctalia_release.py` → ротация и outputs. Проверяются policy, Manifest и `pkgcheck scan --exit`. No-update, существующий PR или dry-run не дают разрешения на публикацию. |
| Umbriel [snapshot][umbriel-workflow] | 06:23, `prepare → publish`, 30 / 10, `dry_run`. | `watch_umbriel.py` проверяет open PR перед upstream, пишет discovery JSON. `prepare_umbriel.py` обновляет пару и выдаёт preparation JSON, затем копия проверяется контейнером через `gentoo_validate.py`. Dry-run даёт план, прежняя пара даёт no-update. |
| Luciole [snapshot][luciole-workflow] | 06:43, `prepare → publish`, 35 / 10, `dry_run`. | `prepare_luciole.py` совмещает discovery/staging за пределами checkout. Перед выдачей кандидата проверяет открытый automation PR и SHA, пишет receipt и outputs. `validate_luciole.py validate` проверяет Gentoo-кандидат и создаёт handoff, `verify` повторно сверяет его на хосте. [Validate][luciole-validate] |
| Spotatui [release][spotatui-workflow] | 05:23, `prepare → publish`, 50 / 10, `dry_run`. | `release_automation.py discover` → release JSON/summary. `prepare` в Docker скачивает теговые архивы, сравнивает контракты, использует pycargoebuild и формирует изолированный кандидат. Portage генерирует Manifest, затем `check-manifest` и `pkgcheck`. Изменения packaging contract останавливают обновление. |
| ASUS NumberPad [release][asus-numberpad-workflow] | 06:17, `prepare → publish`, — / —, `dry_run`. | `release.py watch` → tag/commit/update, затем `prepare` ротирует копию `git archive HEAD`. Gate генерирует Manifest, запускает `ebuild clean prepare`, проверяет синтаксис `numberpad.py`, сканирует оба пакета и проверяет workspace и tag. Packaging changes или конфликт automation branch останавливают запуск. |
| Coucou [дерево][coucou-tree] | Workflow отсутствует. | В дереве только ручная упаковка и описание ограничений. Автоматических Discover, Prepare, Validate, Handoff и Publish нет. |

В таблице «—» означает отсутствие явного `timeout-minutes`. При ошибке
подготовки publish не получает handoff. No-update и штатный
пропуск могут дать успешный статус запуска. Ошибка после push может
оставить automation branch. У Zed перед push отдельно публикуется
distfile release, который тоже может сохраниться после сбоя PR.

| Оверлей | Handoff и проверка base / diff | Publish, защита от конфликта и восстановление |
| --- | --- | --- |
| Zed | `release-handoff` (14 дней) содержит JSON, ebuild, patch и Manifest. `crates-distribution` (14 дней) содержит архив и JSON. Логи проверки живут 7 дней. Read-only job сверяет состав, base, bundle и SHA256 metadata, publish повторно проверяет текущий `main`, policy и точный staged diff. | Publish ограничен `refs/heads/main`, `crates_github.py publish` дополнительно сверяет репозиторий. Публикует или переиспользует проверенный GitHub Release с архивом и метаданными, сверяя assets. `publish-release-pr.sh` использует API, пропускает существующий open PR и останавливается для ветки без PR. [Handoff][zed-handoff] · [Bundle][zed-bundle] · [Assets][zed-assets] · [PR][zed-publish] |
| Noctalia | `release-handoff` (30 дней) содержит JSON с provenance/rotation/base, candidate, Manifest и оба README. `apply` проверяет состав и совпадение с актуальным `main`. Diff допускает добавление candidate и удаление только старого fallback вместе с обновлением Manifest и README. | Publish checkout и PR base — `main`. Допуск к publish зависит от readiness, отдельного условия по event ref или точному имени repo в YAML нет. Watcher проверяет open PR, повторный push той же ветки с `--force-with-lease` разрешён. PR создаётся через API. [Handoff][noctalia-handoff] · [Watcher][noctalia-watch] · [Workflow][noctalia-workflow] |
| Umbriel | `umbriel-handoff` (2 дня) передаёт JSON, файлы пары, Manifest и README. `apply` сверяет состав и base с checkout `main`, `verify-diff` проверяет точные изменения обоих пакетов, таблиц и accepted-pairs. | Checkout и PR target — `main`, discovery привязан к именованному overlay. YAML не проверяет event ref/repo отдельным условием. Publish повторно проверяет правило одного open automation PR и использует прежнюю ветку только при совпадении tree и родителя. PR через API. [Handoff][umbriel-handoff] · [Workflow][umbriel-workflow] |
| Luciole | `luciole-handoff-${{ github.run_attempt }}` (2 дня), передаётся ID текущей попытки, JSON с receipt/base/checksums и только package-файлы. Проверяется состав и воспроизводится содержимое ebuild из accepted template и SHA. Точный diff ограничен ротацией package-файлов и Manifest. | Оба jobs ограничены точным repo и `main`. `publish_luciole.py publish` проверяет remote main, identity родителя/tree и open PR до записи и перед API-запросом. Идентичная ветка переиспользуется, конфликт останавливает push/PR. [Validate][luciole-validate] · [Publish][luciole-publish] |
| Spotatui | `spotatui-release-handoff` (1 день) содержит README, Manifest, candidate и JSON. Проверяются состав, base из prepare job и метаданных, rotation plan и diff с разрешённым удалением fallback. | Checkout и PR base — `main`. Workflow не задаёт отдельный gate по event ref или имени repo. При любом прежнем PR для этой ветки или существующей ветке публикация останавливается, push без force, Draft PR через API. [Скрипт][spotatui-release] · [Workflow][spotatui-workflow] |
| ASUS NumberPad | `numberpad-handoff` (1 день) содержит только Manifest и JSON с tag, commit и base. `recheck` вновь выполняет discovery и хеширует архив, `apply` требует чистый checkout на base и заново выполняет ротацию. `verify` ограничивает diff README, драйвером, Manifest и версионными patches. | Checkout и PR target берутся из default branch. Отдельного gate по event ref или имени repo нет. Workflow сверяет remote base, использует обычный push и `gh pr create --draft`. Открытый automation PR останавливает обновление, остаточную ветку без PR нужно разобрать вручную. [Скрипт][asus-numberpad-release] · [Workflow][asus-numberpad-workflow] |
| Coucou | Автоматического артефакта и проверки публикации нет. | Подготовку и принятие определяет владелец. Из условий artwork следуют ограничения на дистрибуцию. [README][coucou-readme] |

## Проверки и подтверждённые запуски

Все шесть update workflows используют одну пару образов
`gentoo/portage@sha256:ce69e5b771bfe9404731e846b835346c34f2c82817ac3626f13af198689a1961`
и `gentoo/stage3@sha256:d8536e52677e3efe4caf048ca588bc685bc2d24d7fdfc882de1cdc6d38c81d96`.
Spotatui и ASUS NumberPad используют эту же пару в обычном CI. Для
`pkgcheck`, установленного через `emerge`, версия определяется деревом и
конфигурацией контейнера. Число тестов ниже измерено в Phase 0 командой
`python3 -B -m unittest discover -s tests` (`-s scripts` у Spotatui)
во временных копиях. Gentoo-контейнеры workflows не запускались.

| Оверлей | Обычный push / PR CI и локальные unit-тесты | Проверка кандидата в реализации | Сборка и runtime |
| --- | --- | --- | --- |
| Zed | Push в `main` и PR. `overlay_policy.py`, `bash -n`, 119 тестов. Все эти локальные проверки прошли. Unit-тесты охватывают policy, архивы/Cargo, handoff и shell orchestration. [CI][zed-ci] · [Tests][zed-tests] | Сверка Git tree и архива, Cargo.lock/crates, Manifest и offline source mapping, `patch --dry-run`, `emerge -pv --oneshot` для точного amd64-атома. `pkgcheck` и `portage.vercmp` в этом пути отсутствуют. [Candidate][zed-candidate] | `CHECKPOINT.md` фиксирует нативную сборку, проверку линковки и Wayland runtime 1.21.0 на amd64. Также описан исторический `unpack prepare`/offline Cargo metadata 1.25.2. Свидетельство для 1.29.3 и arm64 отсутствует. [Checkpoint][zed-checkpoint] |
| Noctalia | Push/PR, 55 тестов watcher, rotation, policy README, provenance и handoff. [CI][noctalia-ci] · [Tests][noctalia-tests] | `ebuild … manifest`, `pkgcheck scan --exit`. Версии сравниваются числовыми кортежами. Для ручного изменения virtual README предлагает отдельно `pkgcheck` и `emerge -pv`. | README требует `emerge -pv`, установки кандидата и ручной проверки панели, уведомлений, сети, звука и блокировки в Niri. Выполнение для текущего релиза не подтверждено. [README][noctalia-readme] |
| Umbriel | Push/PR, 20 тестов состояния, discovery, rotation, handoff и diff, в том числе прогон suite после двух ротаций. [CI][umbriel-ci] · [Tests][umbriel-tests] | Portage `vercmp` для изменившихся компонентов, Manifest для сохранённых пар и точный набор distfiles, scan двух атомов, `emerge -pv` для current pair. Для amd64 фиксируются временные USE overrides и при необходимости точные `~amd64` atoms для пары и `sdbus-c++`. [Validator][umbriel-validator] | Перед merge README требует ручной проверки сессии, включая вывод, ввод, screencast и picker. Результаты сборки и runtime для текущей пары остаются открытыми. [README][umbriel-readme] |
| Luciole | Push/PR, timeout 10, 45 тестов SHA/истории, staging, Manifest, handoff и branch recovery. Контейнерные команды в тестах подменяются. [CI][luciole-ci] · [Tests][luciole-tests] | `vercmp`, Portage Manifest, сверка байтов обоих архивов и сохранение хеша fallback, `emerge -pv --with-bdeps=y --autounmask=n` для candidate с `-test`, временные USE/keyword overrides и scan одного атома. Docker запускается как `linux/amd64`. [Gentoo validator][luciole-validator] · [Validate][luciole-validate] | Сборка, тесты Meson и Wayland runtime проверяются отдельно. Документированный GCC/libstdc++ 16.2 относится к upstream. [README][luciole-readme] |
| Spotatui | Push/PR, независимые `python` и `pkgcheck` (timeout 30). 20 тестов packaging contract и handoff. `check-manifest` на checkout прошёл для 850 crate aliases. CI сканирует весь overlay в профиле `default/linux/amd64/23.0`. [CI][spotatui-ci] · [Tests][spotatui-tests] | Генерация Manifest и сверка CRATES, scan урезанного дерева с тем же профилем. Resolver и сборку README оставляет ручной проверке. [Workflow][spotatui-workflow] | README сообщает об успешной сборке и воспроизведении для раннего минимального пакета. Результат для 0.48.0 и дополнительных USE-флагов неизвестен. [README][spotatui-readme] |
| ASUS NumberPad | Push/PR, независимые `test` (unit-тесты и whitespace) и `pkgcheck` для обоих пакетов с `--exit=error,warning`. 13 тестов rotation, PR/branch, upstream guard, handoff/allowlist. [CI][asus-numberpad-ci] · [Tests][asus-numberpad-tests] | Portage Manifest, `clean prepare`, проверка применения patch и Python `compile()` без исполнения, scan двух пакетов. Полный compile/install и resolver не входят в этот gate. [Workflow][asus-numberpad-workflow] | README оставляет проверку shutdown patch и работу устройства ручному тесту. Для 7.3.0 результат аппаратной проверки в изученных материалах отсутствует. [README][asus-numberpad-readme] |
| Coucou | CI и unit-тестов overlay нет. | Нет автоматического gate. | README сообщает об offline-сборке frontend и hook relay. Полный ebuild и запуск в Gentoo требуют проверки. [README][coucou-readme] |

Публичная история запусков фиксирует результат workflow и отдельных jobs.

- **Zed.** CI на audited SHA прошёл. В более позднем scheduled run на том же SHA прошёл `check`, а handoff validation и publish пропущены. Исторический run для 1.28.1 подтверждает успешные check, handoff и publish. [CI][zed-ci-run] · [Schedule][zed-scheduled-run] · [Исторический run][zed-publish-run] · [Draft PR #46][zed-pr]
- **Noctalia и ASUS NumberPad.** В проверенных scheduled runs на audited SHA прошёл `prepare`, publish пропущен. Это не подтверждает прохождение условных Gentoo-шагов. [Noctalia][noctalia-run] · [ASUS NumberPad][asus-numberpad-run]
- **Umbriel и Luciole.** Push CI прошёл на audited SHA. Более ранние ручные запуски подтверждают, что оба jobs завершились успешно на `b810d492ff2643f734945760d17c0f8767101bc4` и `db1eae4db9ce43e68650d0d35e009f8091692b4a` соответственно. По workflow это подтверждает прохождение candidate gate для этих запусков. Следующий проверенный Umbriel schedule пропустил publish. [Umbriel CI][umbriel-ci-run] · [Umbriel dispatch][umbriel-run] · [Umbriel schedule][umbriel-scheduled-run] · [Luciole CI][luciole-ci-run] · [Luciole dispatch][luciole-run]
- **Spotatui.** На audited SHA ручной run 45464109167 пропустил publish. Scheduled run 45648443992 завершился с exit 1 в шаге подготовки кандидата и Gentoo-проверок, загрузка handoff и publish пропущены. Лог шага вернул HTTP 404. Какой вызов завершился ошибкой, неизвестно. [Ручной run][spotatui-manual-run] · [Неуспешный шаг][spotatui-failed-run]
- **Coucou.** На публичной [странице Actions][coucou-actions] запусков не видно. В дереве audited SHA нет workflow.

### Spotatui и корневой `tests/`

Комментарий в [workflow CI][spotatui-ci] подтверждён для проверки всего оверлея.
На временной копии audited SHA выполнен
`pkgcheck scan --config no --cache-dir <tmp-cache> --repo "$PWD" --profile default/linux/amd64/23.0 --exit`
до и после копирования одного теста в корневой `tests/`. Использованы
локальные `pkgcheck 0.10.50`, `pkgcore 0.12.50` и Gentoo tree с
`metadata/timestamp.chk` от 2026-11-24 21:15:00 UTC. В обоих прогонах
выведен `RedundantVersion` для fallback и получен exit 0. Только второй
добавил `UnknownCategoryDirs: unknown category dir: tests`. В отдельном
прогоне Umbriel с его существующим `tests/` и двумя явными атомами
выведены два `RedundantVersion`, exit 0, предупреждения о категории нет.
Эффект зависит от scope и политики `--exit`. Его точное поведение в
закреплённых контейнерах CI не проверено. Связь с наблюдаемой ошибкой
scheduled run Spotatui не установлена.

## Документация и настройки GitHub

| Оверлей | Документация, рабочие файлы и обслуживание | Публичные настройки и required checks |
| --- | --- | --- |
| Zed | RU/EN README с установкой, ограничениями Wayland и ручной проверкой. Есть `AGENTS.md`, `CHECKPOINT.md`, RU/EN CI/security docs, `docs/` с Jekyll handbook, `mise.toml` для build/preview, опциональный Portage post-sync hook в `contrib/`. `.gitignore` покрывает Python, `.history/` и `_site/`. Weekly Dependabot для Actions. [README][zed-readme] · [Правила][zed-agents] · [CI docs][zed-docs] · [Mise][zed-mise] · [Dependabot][zed-dependabot] · [Сайт][zed-site] | Description «Gentoo overlay for building Zed natively with a Wayland-only feature set.». Pages опубликован, `has_pages=true`, виден исторический deployment. Required `Overlay policy`, strict. [Ruleset][zed-rules] · [Pages run][zed-pages-run] |
| Noctalia | RU/EN README с установкой, влиянием virtual, review PR и проверкой после обновления. Есть `AGENTS.md` и RU/EN CI/security docs. `.gitignore` покрывает локальные резервные, Python, history и типичные файлы с секретами. Weekly Dependabot для Actions. [README][noctalia-readme] · [Правила][noctalia-agents] · [CI docs][noctalia-docs] · [Dependabot][noctalia-dependabot] | Description «Personal Gentoo overlay for Noctalia». `has_pages=false`. Required `Python unit tests`, strict. [Ruleset][noctalia-rules] |
| Umbriel | RU/EN README с точной установкой пары и откатом, настройками GitHub и повтором контейнерной проверки. `AGENTS.md` описывает источник пары, обновление и handoff. `automation/` хранит состояние, сценарий проверки после ротации в `tests/`. [README][umbriel-readme] · [Правила][umbriel-agents] | Description отсутствует. `has_pages=false`. Required `Python unit tests`, strict. [Ruleset][umbriel-rules] |
| Luciole | RU/EN README с установкой, локальным staging и Gentoo validation, automation PR, восстановлением ветки и требованиями сеанса. [README][luciole-readme] | Description отсутствует. `has_pages=false`. Список rulesets вернул `[]`. [Rulesets API][luciole-rules-api] |
| Spotatui | RU README с установкой, USE-флагами, release workflow и ограничениями подтверждённой сборки. `.gitignore` содержит `.history/`. Тесты находятся в `scripts/`. [README][spotatui-readme] · [CI][spotatui-ci] | Description отсутствует. `has_pages=false`. Список rulesets вернул `[]`. [Rulesets API][spotatui-rules-api] |
| ASUS NumberPad | EN README с установкой, OpenRC/systemd, доступом к устройству, удалением, версиями, release workflow и ограничениями upstream guard. [README][asus-numberpad-readme] | Description отсутствует. `has_pages=false`. Required `test`, strict выключен. `pkgcheck` не включён в этот ruleset. [Ruleset][asus-numberpad-rules] |
| Coucou | Краткий EN README с provenance, результатами частичной сборки и условиями дистрибуции. Инструкций Portage по подключению, обновлению или откату нет. [README][coucou-readme] | Description отсутствует. `has_pages=false`. Список rulesets вернул `[]`. [Rulesets API][coucou-rules-api] |

У всех семи `topics=[]`. Только Zed, Noctalia и Umbriel имеют
`AGENTS.md`. `.gitignore` найден у Zed, Noctalia и Spotatui,
Dependabot YAML — у Zed и Noctalia. Локальный task runner найден только
у Zed. У остальных файлов с этими назначениями в проверенных деревьях
нет. Полные деревья и repository metadata доступны из таблицы ревизий.

Четыре активных публичных rulesets действуют для `~DEFAULT_BRANCH`,
требуют PR и разрешения review conversations, запрещают удаление и
non-fast-forward. В них `required_approving_review_count=0`, а источник
перечисленных required checks — GitHub Actions. Ручное принятие владельцем
поддерживается процессом и отсутствием auto-merge в рабочих сценариях.
Публичный ruleset не подтверждает состав владельцев прав merge или
bypass. Сведения в README о рекомендуемых настройках GitHub следует
читать как инструкции.

**UNKNOWN для всех семи.** Legacy branch protection и дополнительные
required checks, Actions allowlist/enabled policy, default workflow token
permissions, флаг создания/одобрения PR и фактические bypass-права.
Запросы к endpoints branch protection и Actions permissions вернули
HTTP 401. Запросы к branch details, ruleset details и API отдельных jobs
вернули HTTP 403 из-за rate limit. Четыре конфигурации rulesets и
результаты отдельных jobs выше подтверждены публичными HTML-страницами.
Пустой список rulesets оставляет состояние legacy protection неизвестным.
Pages API вернул 404, поэтому настройки источника Pages не подтверждены.
[Пример branch protection API][settings-branch] · [Actions permissions API][settings-actions] · [Token permissions API][settings-token] · [Pages API][settings-pages]

## Сравнение контрактов

| Пересечение | Установленные различия | Что можно взять за образец |
| --- | --- | --- |
| `gentoo_validate.py` | Umbriel запускает проверку внутри уже зарегистрированного overlay и сопоставляет две пары. Luciole получает отдельный staged inventory и возвращает один проверенный Manifest. Отличаются состав атомов, resolver overrides, правила fallback и защита байтов архивов. | Фиксировать ревизию Gentoo tree, проверяемые атомы, окружение, критерии успеха и границы resolver. [Umbriel][umbriel-validator] · [Luciole][luciole-validator] |
| `overlay_policy.py` | Zed проверяет layout, tracked files и Wayland patch. Noctalia проверяет две версии и синхронность локализованных README. Umbriel верифицирует JSON-пары, SHA и таблицы. Luciole, ASUS и Spotatui держат policy в своих инструментах. | Определять policy по инвариантам и проверять отказ на неверном состоянии. [Zed][zed-policy] · [Noctalia][noctalia-policy] · [Umbriel][umbriel-policy] |
| Release и snapshot prepare | Zed добавляет новый релиз и готовит distfile release, Noctalia ротирует копию и сообщает packaging changes, Spotatui пересобирает CRATES, ASUS повторяет локальную ротацию при apply. Umbriel исходит из upstream lock, Luciole — из продвижения одного upstream main. | Полезны отдельные outcomes no-update / готовый кандидат / остановка для ревью, явный неизменяемый источник и тесты ротации. Точное число сохранённых версий и packager guards принадлежат проекту. [Матрица автоматизации](#автоматизация-и-границы-публикации) |
| Handoff и PR publication | Zed применяет двухартефактный контракт с отдельным read-only gate. Luciole связывает artifact ID и checksums с переиспользованием идентичной ветки. Umbriel сравнивает tree и родителя, Noctalia допускает lease-update, Spotatui блокирует любой прежний PR на эту ветку. ASUS передаёт только Manifest и повторяет план. | Небольшой артефакт, base SHA, допустимый diff, явная обработка stale base и частичного сбоя, отдельный write job и ручной merge. Допустимые пути и recovery policy зависят от проекта. [Матрица автоматизации](#автоматизация-и-границы-публикации) |

## Результаты аудита

- **BLOCKER — Spotatui, конкретный scheduled run.** Ошибка candidate step
  остановила формирование handoff и публикацию. Повторяемость причины и
  её влияние на следующий релиз остаются UNKNOWN. [Неуспешный шаг][spotatui-failed-run]
- **DRIFT — Zed, статус в checkpoint.** `CHECKPOINT.md` называет 1.29.2
  текущим релизом, хотя в том же SHA присутствуют 1.29.3 ebuild и patch.
  Зафиксированная там ручная проверка 1.21.0 остаётся историческим
  свидетельством. [Checkpoint][zed-checkpoint] · [Пакет][zed-pkg]
- **DRIFT — Umbriel, описание resolver flags.** Комментарий
  `gentoo_validate.py` говорит о четырёх USE overrides, а кортеж содержит
  пять. Это расхождение документации с текущим кодом, без вывода о
  корректности отдельного флага. [Validator][umbriel-validator]
- **DRIFT — Spotatui, описание PR-триггера.** README говорит, что создание
  PR через `GITHUB_TOKEN` не запускает следующий workflow. Для
  `pull_request` с `opened`, `synchronize` или `reopened` актуальная
  документация GitHub описывает запуск, ожидающий ручного одобрения.
  Подавление `push` остаётся отдельным поведением. [README][spotatui-readme] · [Документация GitHub][github-trigger]
- **EXPECTED DIFFERENCE.** Wayland-only Zed и его отдельный distfile
  release, локальный virtual Noctalia, согласованные пары Umbriel,
  требования к тестам Luciole, Cargo/ALSA у Spotatui, проверка hardware и
  отдельная PyPI-зависимость ASUS, лицензия artwork Coucou. Retention и
  rollback различаются по модели пакета. [Политика упаковки](#состав-и-политика-упаковки)
- **OPTIONAL — Spotatui, контракт cache.** Две tracked cache-записи
  сейчас соответствуют MD5 обоих ebuild. Текущий handoff и diff
  исключают обновление cache, поэтому ротация сама не пересоздаёт эти
  записи. Назначение tracked cache, потребители и внешний способ
  обновления требуют решения владельца. Отказ Portage из-за этого на
  данном SHA не установлен. [Cache][spotatui-cache] · [Workflow][spotatui-workflow]

## Вопросы перед Phase 1

1. **UNKNOWN.** Какой вызов в Spotatui candidate step завершился ошибкой?
   Нужен доступный лог этого шага или отдельное воспроизведение на той же
   паре образов. Заодно следует проверить категорию `tests/` на pinned
   окружении с фактическим `--exit`.
2. **UNKNOWN.** Какие legacy protections, дополнительные required checks,
   Actions policy и bypass-права действуют сейчас? Для подтверждения
   нужны read-only сведения из настроек владельца. Отдельно следует решить, какие checks обязательны для merge, включая
   `pkgcheck` ASUS NumberPad.
3. **UNKNOWN.** Какие версии и USE/архитектуры имеют ручные подтверждения
   сборки и runtime? Сохранять отдельно unit-тесты, metadata/resolver,
   prepare, полный build, runtime и owner acceptance. Для Coucou также
   нужен полный список лицензий prebuilt native archive.
4. Решить с владельцем, какие политики допустимы для stale base, закрытого
   PR и осиротевшей automation branch. При сравнении учесть ограничения
   repo/ref, независимый read-only gate Zed и сверку удалённых refs Luciole.

## Предложения для Phase 1

Согласовать словарь шести стадий, минимальный формат доказательств,
разделы README и критерии принятия для ручного, release, snapshot и
extended release процессов. Вынести на решение владельца интерфейс
результатов discovery, требования к handoff и к разграничению write job,
привязку результата к base SHA, правила required checks и ручной
проверки. Для каждого проекта отдельно зафиксировать source provenance,
retention, rollback, совместимость и исключения. Начать с уже работающих
контрактов и ограничить Phase 1 согласованием соглашений. Перенос кода,
шаблоны и миграция требуют отдельного решения.

[zed-api]: https://api.github.com/repos/vovanbl411/zed-overlay
[zed-tree]: https://github.com/vovanbl411/zed-overlay/tree/3a19639d2b097be7bf4c65f0fc0e6a094d45244f
[zed-pkg]: https://github.com/vovanbl411/zed-overlay/tree/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/app-editors/zed
[zed-prepare]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/prepare_zed_release.py
[zed-candidate]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/ci/independent-crates.sh
[zed-workflow]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.github/workflows/zed-release-watcher.yml
[zed-orchestrator]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/ci/validate-release-candidate.sh
[zed-watch]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/watch_zed_release.py
[zed-handoff]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/release_handoff.py
[zed-bundle]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/crates_distribution.py
[zed-assets]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/crates_github.py
[zed-publish]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/ci/publish-release-pr.sh
[zed-ci]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.github/workflows/ci.yml
[zed-tests]: https://github.com/vovanbl411/zed-overlay/tree/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/tests
[zed-checkpoint]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/CHECKPOINT.md
[zed-readme]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/README.md
[zed-agents]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/AGENTS.md
[zed-docs]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/docs/CI_AND_SECURITY.md
[zed-mise]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/mise.toml
[zed-dependabot]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.github/dependabot.yml
[zed-policy]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/scripts/overlay_policy.py
[zed-ci-run]: https://github.com/vovanbl411/zed-overlay/actions/runs/45272748485
[zed-scheduled-run]: https://github.com/vovanbl411/zed-overlay/actions/runs/45662465140
[zed-publish-run]: https://github.com/vovanbl411/zed-overlay/actions/runs/44148805330
[zed-pr]: https://github.com/vovanbl411/zed-overlay/pull/46
[zed-rules]: https://github.com/vovanbl411/zed-overlay/rules/23906510
[zed-pages-run]: https://github.com/vovanbl411/zed-overlay/actions/runs/45272747672
[noctalia-api]: https://api.github.com/repos/vovanbl411/noctalia-overlay
[noctalia-tree]: https://github.com/vovanbl411/noctalia-overlay/tree/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c
[noctalia-pkg]: https://github.com/vovanbl411/noctalia-overlay/tree/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/gui-apps/noctalia
[noctalia-virtual]: https://github.com/vovanbl411/noctalia-overlay/tree/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/virtual/notification-daemon
[noctalia-watch]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/scripts/watch_noctalia_release.py
[noctalia-prepare]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/scripts/prepare_noctalia_release.py
[noctalia-workflow]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/workflows/noctalia-release-watcher.yml
[noctalia-handoff]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/scripts/release_handoff.py
[noctalia-ci]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/workflows/ci.yml
[noctalia-tests]: https://github.com/vovanbl411/noctalia-overlay/tree/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/tests
[noctalia-readme]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/README.md
[noctalia-agents]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/AGENTS.md
[noctalia-docs]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/docs/CI_AND_SECURITY.md
[noctalia-dependabot]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/dependabot.yml
[noctalia-policy]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/scripts/overlay_policy.py
[noctalia-run]: https://github.com/vovanbl411/noctalia-overlay/actions/runs/45663950280
[noctalia-rules]: https://github.com/vovanbl411/noctalia-overlay/rules/23505842
[umbriel-api]: https://api.github.com/repos/vovanbl411/umbriel-overlay
[umbriel-tree]: https://github.com/vovanbl411/umbriel-overlay/tree/87878e3b8a96e35361833f2385249bba6b1f41a2
[umbriel-pkg]: https://github.com/vovanbl411/umbriel-overlay/tree/87878e3b8a96e35361833f2385249bba6b1f41a2/gui-wm/umbriel
[umbriel-portal]: https://github.com/vovanbl411/umbriel-overlay/tree/87878e3b8a96e35361833f2385249bba6b1f41a2/gui-libs/xdg-desktop-portal-umbriel
[umbriel-state]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/automation/accepted-pairs.json
[umbriel-watch]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/watch_umbriel.py
[umbriel-prepare]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/prepare_umbriel.py
[umbriel-workflow]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/.github/workflows/prepare-umbriel-snapshot.yml
[umbriel-handoff]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/umbriel_handoff.py
[umbriel-ci]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/.github/workflows/ci.yml
[umbriel-tests]: https://github.com/vovanbl411/umbriel-overlay/tree/87878e3b8a96e35361833f2385249bba6b1f41a2/tests
[umbriel-validator]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/gentoo_validate.py
[umbriel-readme]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/README.md
[umbriel-agents]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/AGENTS.md
[umbriel-policy]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/overlay_policy.py
[umbriel-ci-run]: https://github.com/vovanbl411/umbriel-overlay/actions/runs/45671536597
[umbriel-run]: https://github.com/vovanbl411/umbriel-overlay/actions/runs/45643612923
[umbriel-scheduled-run]: https://github.com/vovanbl411/umbriel-overlay/actions/runs/45664702620
[umbriel-rules]: https://github.com/vovanbl411/umbriel-overlay/rules/25914466
[luciole-api]: https://api.github.com/repos/vovanbl411/luciole-overlay
[luciole-tree]: https://github.com/vovanbl411/luciole-overlay/tree/4fae56beb67b2a8c47c3427162c36fce823bbaf3
[luciole-pkg]: https://github.com/vovanbl411/luciole-overlay/tree/4fae56beb67b2a8c47c3427162c36fce823bbaf3/gui-apps/luciole
[luciole-prepare]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/scripts/prepare_luciole.py
[luciole-readme]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/README.md
[luciole-workflow]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/.github/workflows/prepare-luciole-snapshot.yml
[luciole-validate]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/scripts/validate_luciole.py
[luciole-publish]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/scripts/publish_luciole.py
[luciole-ci]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/.github/workflows/ci.yml
[luciole-tests]: https://github.com/vovanbl411/luciole-overlay/tree/4fae56beb67b2a8c47c3427162c36fce823bbaf3/tests
[luciole-validator]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/scripts/gentoo_validate.py
[luciole-ci-run]: https://github.com/vovanbl411/luciole-overlay/actions/runs/45671346323
[luciole-run]: https://github.com/vovanbl411/luciole-overlay/actions/runs/45666211658
[luciole-rules-api]: https://api.github.com/repos/vovanbl411/luciole-overlay/rulesets
[spotatui-api]: https://api.github.com/repos/vovanbl411/spotatui-overlay
[spotatui-tree]: https://github.com/vovanbl411/spotatui-overlay/tree/b00245be9dbe3adbee039428dd4e1cb5daedc55b
[spotatui-pkg]: https://github.com/vovanbl411/spotatui-overlay/tree/b00245be9dbe3adbee039428dd4e1cb5daedc55b/media-sound/spotatui
[spotatui-profiles]: https://github.com/vovanbl411/spotatui-overlay/tree/b00245be9dbe3adbee039428dd4e1cb5daedc55b/profiles
[spotatui-cache]: https://github.com/vovanbl411/spotatui-overlay/tree/b00245be9dbe3adbee039428dd4e1cb5daedc55b/metadata/md5-cache
[spotatui-release]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/scripts/release_automation.py
[spotatui-readme]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/README.md
[spotatui-workflow]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/.github/workflows/prepare-release.yml
[spotatui-ci]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/.github/workflows/ci.yml
[spotatui-tests]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/scripts/test_release_automation.py
[spotatui-manual-run]: https://github.com/vovanbl411/spotatui-overlay/actions/runs/45464109167
[spotatui-failed-run]: https://github.com/vovanbl411/spotatui-overlay/actions/runs/45648443992/job/139411464088#step:6:3467
[spotatui-rules-api]: https://api.github.com/repos/vovanbl411/spotatui-overlay/rulesets
[asus-numberpad-api]: https://api.github.com/repos/vovanbl411/asus-numberpad-overlay
[asus-numberpad-tree]: https://github.com/vovanbl411/asus-numberpad-overlay/tree/a0a9b4277f6b5f9823666014914df7210814d908
[asus-numberpad-pkg]: https://github.com/vovanbl411/asus-numberpad-overlay/tree/a0a9b4277f6b5f9823666014914df7210814d908/sys-apps/asus-numberpad-driver
[asus-numberpad-python]: https://github.com/vovanbl411/asus-numberpad-overlay/tree/a0a9b4277f6b5f9823666014914df7210814d908/dev-python/python-periphery
[asus-numberpad-release]: https://github.com/vovanbl411/asus-numberpad-overlay/blob/a0a9b4277f6b5f9823666014914df7210814d908/scripts/release.py
[asus-numberpad-readme]: https://github.com/vovanbl411/asus-numberpad-overlay/blob/a0a9b4277f6b5f9823666014914df7210814d908/README.md
[asus-numberpad-workflow]: https://github.com/vovanbl411/asus-numberpad-overlay/blob/a0a9b4277f6b5f9823666014914df7210814d908/.github/workflows/release.yml
[asus-numberpad-ci]: https://github.com/vovanbl411/asus-numberpad-overlay/blob/a0a9b4277f6b5f9823666014914df7210814d908/.github/workflows/ci.yml
[asus-numberpad-tests]: https://github.com/vovanbl411/asus-numberpad-overlay/tree/a0a9b4277f6b5f9823666014914df7210814d908/tests
[asus-numberpad-run]: https://github.com/vovanbl411/asus-numberpad-overlay/actions/runs/45663029974
[asus-numberpad-rules]: https://github.com/vovanbl411/asus-numberpad-overlay/rules/26209977
[coucou-api]: https://api.github.com/repos/vovanbl411/coucou-overlay
[coucou-tree]: https://github.com/vovanbl411/coucou-overlay/tree/b4710550a048e7b15e653699be95c1ebf00f1a68
[coucou-pkg]: https://github.com/vovanbl411/coucou-overlay/tree/b4710550a048e7b15e653699be95c1ebf00f1a68/gui-apps/coucou
[coucou-readme]: https://github.com/vovanbl411/coucou-overlay/blob/b4710550a048e7b15e653699be95c1ebf00f1a68/README.md
[coucou-license]: https://github.com/vovanbl411/coucou-overlay/blob/b4710550a048e7b15e653699be95c1ebf00f1a68/licenses/coucou-assets
[coucou-actions]: https://github.com/vovanbl411/coucou-overlay/actions
[coucou-rules-api]: https://api.github.com/repos/vovanbl411/coucou-overlay/rulesets
[vicinae-api]: https://api.github.com/repos/vovanbl411/vicinae-overlay
[spotify-player-api]: https://api.github.com/repos/vovanbl411/spotify-player-overlay
[zed-site]: https://vovanbl411.github.io/zed-overlay/
[settings-branch]: https://api.github.com/repos/vovanbl411/zed-overlay/branches/main/protection
[settings-actions]: https://api.github.com/repos/vovanbl411/zed-overlay/actions/permissions
[settings-token]: https://api.github.com/repos/vovanbl411/zed-overlay/actions/permissions/workflow
[settings-pages]: https://api.github.com/repos/vovanbl411/zed-overlay/pages
[github-trigger]: https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow
