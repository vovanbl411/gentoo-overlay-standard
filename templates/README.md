# Справочные шаблоны v0.1

Выбирайте нужные файлы и адаптируйте их в отдельном решении о внедрении.
[Standard v0.1](../STANDARD.md) содержит нормативные требования.
Существующие имена, расположение файлов и рабочие реализации можно
сохранить, если они выполняют применимые требования.

| Материал | Как использовать |
| --- | --- |
| [README RU](overlay/README.md) и [README EN](overlay/README.en.md) | Замените заполнители `⟨…⟩` данными оверлея. Сверьте версии, команды и ограничения между языками. Неприменимый раздел можно опустить или кратко пояснить. |
| [AGENTS.md](overlay/AGENTS.md) | Укажите принятую версию, профиль и локальную политику. Сошлитесь на действующие инструкции проекта. |
| [CI и границы публикации](overlay/docs/CI_AND_SECURITY.md) | Используйте, если этих сведений недостаточно в README. Опишите существующий поток, включая его ограничения. |
| [CI](overlay/.github/workflows/ci.yml) | Пример проверки Python unit-тестов для автоматизированных оверлеев. Инструкции по адаптации ниже. |
| [Dependabot](overlay/.github/dependabot.yml) | Еженедельное предложение обновлений GitHub Actions. |
| [.gitignore](overlay/.gitignore) | Пример исключений для локальных Python-файлов. Сверьте с намеренно отслеживаемыми путями. |
| [Чек-лист](adoption-checklist.md) | Фиксация доказательств и решения владельца по отдельному оверлею. |
| [Справочник по обновлениям](../docs/update-workflow-reference.md) | Примеры существующих стадий и пробелы, выявленные при сравнении со стандартом. |

## Адаптация CI

- В примере `main` — принятая ветка. Подставьте фактическую ветку в
  фильтры `pull_request` и `push`.
- Команда запускается из корня оверлея с Python 3 и предпосылками,
  перечисленными проектом. Замените `-s scripts/tests` на каталог
  существующих тестов, например `-s tests` или `-s scripts`.
  Сверьте импорты и пути fixtures, если раскладка меняется. При проверке
  команды смотрите число выполненных тестов. Нулевой набор не подтверждает
  V1. [Результаты эксперимента](../audits/test-layout.md)
- Сохраните имена работающих workflow и jobs, если они уже связаны с
  required checks. Если имена, события или источники меняются, сверьте
  фактические rulesets и legacy protection по G2. Учитывайте одобрение
  запусков для automation PR. Строка `name` в примере не задаёт имя
  обязательного check для других репозиториев.
- Локальные policy checks можно подключить к CI, если они нужны на PR.
  Укажите применимые Gentoo metadata и проверки источников в gate
  кандидата, а resolver, build и runtime — там, где их требует локальная
  политика. Пример CI запускает только unit-тесты.

## Аудированные источники

Ссылки ведут на точные ревизии из [Phase 0](../audits/baseline.md).
Использованы структура документации, способы описания ограничений и
рабочие команды проверки.

| Оверлей и SHA | Пути |
| --- | --- |
| Noctalia `cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c` | [README.md][n-ru], [README.en.md][n-en], [AGENTS.md][n-agents], [docs/CI_AND_SECURITY.md][n-doc], [.github/dependabot.yml][n-dependabot] |
| Umbriel `87878e3b8a96e35361833f2385249bba6b1f41a2` | [README.md][u-ru], [README.en.md][u-en], [AGENTS.md][u-agents], [.github/workflows/ci.yml][u-ci] |
| Zed `3a19639d2b097be7bf4c65f0fc0e6a094d45244f` | [README.md][z-ru], [README.en.md][z-en], [AGENTS.md][z-agents], [docs/CI_AND_SECURITY.md][z-doc], [.gitignore][z-ignore] |
| Luciole `4fae56beb67b2a8c47c3427162c36fce823bbaf3` | [.github/workflows/ci.yml][l-ci] |

[n-ru]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/README.md
[n-en]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/README.en.md
[n-agents]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/AGENTS.md
[n-doc]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/docs/CI_AND_SECURITY.md
[n-dependabot]: https://github.com/vovanbl411/noctalia-overlay/blob/cee4a4fb99cf08a39df1d8aa6613d8a8cf397b2c/.github/dependabot.yml
[u-ru]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/README.md
[u-en]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/README.en.md
[u-agents]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/AGENTS.md
[u-ci]: https://github.com/vovanbl411/umbriel-overlay/blob/87878e3b8a96e35361833f2385249bba6b1f41a2/.github/workflows/ci.yml
[z-ru]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/README.md
[z-en]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/README.en.md
[z-agents]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/AGENTS.md
[z-doc]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/docs/CI_AND_SECURITY.md
[z-ignore]: https://github.com/vovanbl411/zed-overlay/blob/3a19639d2b097be7bf4c65f0fc0e6a094d45244f/.gitignore
[l-ci]: https://github.com/vovanbl411/luciole-overlay/blob/4fae56beb67b2a8c47c3427162c36fce823bbaf3/.github/workflows/ci.yml
