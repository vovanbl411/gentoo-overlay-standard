# Phase 2. Расположение Python-тестов

25 ноября 2026 года для каждой раскладки запустили `unittest` и
`pkgcheck` и сравнили результаты с неизменённой копией того же оверлея.
Материалы Phase 0 остаются историческими. Сбой scheduled run Spotatui
в этот эксперимент не входит. [Наблюдение Phase 0](baseline.md#spotatui-и-корневой-tests)

## Ревизии и окружение

| Оверлей | Полный SHA | Контроль и проверяемый scope |
| --- | --- | --- |
| Spotatui | `b00245be9dbe3adbee039428dd4e1cb5daedc55b` | Тест в `scripts/`. Весь оверлей и отдельно `media-sound/spotatui`, профиль `default/linux/amd64/23.0`, `--exit`. [CI][spot-ci] |
| Umbriel | `87878e3b8a96e35361833f2385249bba6b1f41a2` | Тест и вспомогательный runner в `tests/`. Scope из gate кандидата — `gui-wm/umbriel gui-libs/xdg-desktop-portal-umbriel`, `--exit`, без явного `--profile`. Дополнительно проверен весь оверлей. [Workflow][umb-workflow] · [Gentoo validator][umb-validator] |
| Luciole | `4fae56beb67b2a8c47c3427162c36fce823bbaf3` | Только контроль Python CI, `tests/`. [CI][luc-ci] |

Для scan использованы аудированные образы `linux/amd64`.

- `gentoo/stage3@sha256:d8536e52677e3efe4caf048ca588bc685bc2d24d7fdfc882de1cdc6d38c81d96`
  с Python 3.14.7.
- `gentoo/portage@sha256:ce69e5b771bfe9404731e846b835346c34f2c82817ac3626f13af198689a1961`
  с деревом `/var/db/repos/gentoo`. Значение `metadata/timestamp.chk` —
  `Tue, 15 Sep 2026 00:45:00 +0000`.
- `pkgcheck 0.10.50` и `pkgcore 0.12.50` установлены готовыми wheels
  в одноразовом venv без сборки. Попытка
  `emerge --getbinpkgonly --oneshot dev-util/pkgcheck` остановилась из-за
  цикла бинарных зависимостей `libselinux` ↔ `glibc`. Способ установки
  инструмента отличается от аудированного `emerge --oneshot`. Образы,
  дерево Gentoo и проверяемые аргументы сохранены.
- Локальный прогон выполнен на Gentoo 2.18 x86_64 с Python 3.14.10,
  `pkgcheck 0.10.50`, `pkgcore 0.12.50` и дерево с
  `metadata/timestamp.chk` от `Tue, 24 Nov 2026 21:15:00 +0000`.
  Python unit-тесты выполнены локально.

## Повтор проверки

Для каждого SHA распакуйте `git archive <sha>` в пустой временный
каталог `control` и сделайте отдельные копии. Переместите
`test_release_automation.py` у Spotatui, а у Umbriel —
`test_automation.py` вместе с `run_after_rotation.py`, из исходного
каталога в `tests/`, `scripts/` и `scripts/tests/`. Удалите опустевший
исходный каталог в изменённых копиях. Ревизии и содержимое пакетов
одинаковы в контроле и вариантах.

Из корня каждой копии запустите команду, подставив каталог тестов.
Для контроля используйте каталог на аудированном SHA.

```sh
PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s "$test_dir" -p 'test_*.py' -v
```

Для Gentoo смонтируйте дерево из Portage как read-only volume в Stage3,
временные копии — read-only bind mount. Кэш оставьте во временной
файловой системе контейнера. Зарегистрируйте дерево `gentoo` в
`repos.conf` с `auto-sync = no`. Для Umbriel зарегистрируйте
`umbriel-overlay` с `masters = gentoo`, `location` на текущую копию и
`auto-sync = no`. Используйте venv в контейнере.

```sh
python3 -m venv /tmp/pkgcheck-venv
export PATH="/tmp/pkgcheck-venv/bin:$PATH"
python3 -m pip --disable-pip-version-check install --only-binary=:all: \
  pkgcheck==0.10.50 pkgcore==0.12.50
```

В контейнере pip также установил `charset-normalizer 3.5.3`, `lxml 6.1.3`,
`pathspec 1.1.1`, `py-landlock 0.1.1`, `snakeoil 0.11.7`,
`tree-sitter 0.26.1` и `tree-sitter-bash 0.25.1`.

Запускайте соответствующую пару команд из корня каждой копии.
Первая команда в паре проверяет весь overlay, вторая — заданные атомы.

```sh
# Spotatui
pkgcheck scan --cache-dir /tmp/pkgcheck-cache --repo "$PWD" \
  --profile default/linux/amd64/23.0 --exit
pkgcheck scan --cache-dir /tmp/pkgcheck-cache --repo "$PWD" \
  --profile default/linux/amd64/23.0 media-sound/spotatui --exit

# Umbriel
pkgcheck scan --cache-dir /tmp/pkgcheck-cache --repo "$PWD" --exit
pkgcheck scan --cache-dir /tmp/pkgcheck-cache --repo "$PWD" \
  gui-wm/umbriel gui-libs/xdg-desktop-portal-umbriel --exit
```

## Python

| Раскладка | Spotatui, тесты / код выхода | Umbriel, тесты / код выхода |
| --- | --- | --- |
| Контроль | 20 / 0 | 20 / 0 |
| `tests/test_*.py` | 20 / 0 | 20 / 0 |
| `scripts/test_*.py` | 20 / 0 | 20 / 1. Сценарий ротации ищет прежний `tests/`. |
| `scripts/tests/test_*.py` | 20 / 0 | 1 ошибка импорта / 1. Suite не загрузился. |

Отдельно на временных копиях Umbriel для обеих раскладок под `scripts/`
из списка путей, копируемых в fixture ротации, убран корневой `tests`,
а путь вспомогательного runner указан по новому расположению. Для
`scripts/tests/` глубина вычисления корня изменена с `parents[1]` на
`parents[2]`. После этих поправок обе раскладки выполнили 20 тестов с
кодом 0. Контроль Luciole выполнил 45 тестов с кодом 0 командой с
`-s tests`. Umbriel показывает, почему перемещение работающего suite
может потребовать изменений локальных путей.

## pkgcheck

Все 16 запусков в закреплённых образах завершились с кодом 0. Локальное
повторение дало те же диагностические сообщения и коды выхода во всех 16
случаях. В таблице указаны каталоги, перечисленные в `UnknownCategoryDirs`.
«—» означает отсутствие этого сообщения.

| Раскладка | Spotatui, весь overlay | Spotatui, атом | Umbriel, весь overlay | Umbriel, два атома |
| --- | --- | --- | --- | --- |
| Контроль | — | — | `automation`, `tests` | — |
| `tests/test_*.py` | `tests` | — | `automation`, `tests` | — |
| `scripts/test_*.py` | — | — | `automation` | — |
| `scripts/tests/test_*.py` | — | — | `automation` | — |

Во всех вариантах остался исходный `RedundantVersion` для fallback Spotatui
`0.47.0-r1` и оба `RedundantVersion` для предыдущей пары Umbriel
`0.1.0_pre20261123014939` и `0.1.0_p20261122051915`. Umbriel
уже имел диагностику `automation` в контроле. Единственная новая диагностика
в матрице — `tests` в whole-overlay scan Spotatui. В Umbriel перенос под
`scripts/` убрал `tests` из исходного сообщения.

## Рекомендация и границы

Для новых наборов тестов предпочтителен `scripts/tests/`. Оба варианта
под `scripts/` показали одинаковую диагностику. Вложенный каталог
позволяет отделить тесты и fixtures от рабочих скриптов и указать точный
каталог discovery. Существующие раскладки сохраняются, если задокументированы
воспроизводимая команда и область проверки.

Проверялись аудированные деревья оверлеев. Подготовка нового кандидата,
генерация Manifest, сборка и runtime в эксперимент не входили.
Использовался `--exit` из выбранных workflow. Другие пороги, наборы
проверок и версии инструментов могут давать иной результат.

[spot-ci]: https://github.com/vovanbl411/spotatui-overlay/blob/b00245be9dbe3adbee039428dd4e1cb5daedc55b/.github/workflows/ci.yml
[umb-workflow]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/.github/workflows/prepare-umbriel-snapshot.yml
[umb-validator]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/scripts/gentoo_validate.py
[luc-ci]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/.github/workflows/ci.yml
