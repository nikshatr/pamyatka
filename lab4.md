---
title: 'Лабораторная работа 4'
author: 'А.В. Родионов'
fontsize: '14pt'
geometry: 'margin=2cm'
paper: 'a4'
lang: 'russian'
template: 'default.html'
---

## Стратегия ветвления "Git flow"

Благодаря простой и эффективной модели ветвления в Git, разработка программного продукта может быть
существенно облегчена и упорядочена, за счет внедрения оптимальной для команды разработчиков модели
ветвления. В следующих разделах будет описана модель ветвления,
[предложенная](http://nvie.com/posts/a-successful-git-branching-model/) в 2010 году разработчиком
[Vincent Driessen](http://nvie.com/about/), которая после своего опубликования стала фактическим
стандартом работы с Git над крупными проектами. Эта стратегия получила название "Git flow", и
представляет собой набор правил, которых должна придерживаться команда в жизненном цикле разработки
ПО. Для облегчения работы, существует набор [расширений](https://github.com/nvie/gitflow) Git,
которые дополняют стандартные его команды, однако их установка необязательна и применять Git flow
можно и без них, в стандартной поставке Git.

### Главный репозиторий

Git flow предполагает, что существует один "главный" репозиторий, общий для всех участников команды.
Традиционно, он называется `origin`. Основные потоки изменений циркулируют между ним и локальными
репозиториями разработчиков. В то же время, в большом проекте, где над новыми возможностями могут
работать сразу по несколько человек, параллельно с главным репозиторием может существовать несколько
побочных. Они служат для обмена внутри мини-команд, чтобы не загружать изменения в `origin` до
того как они будут окончательно утверждены. В практическом смысле, побочные репозитории представляют
собой обычные удаленные репозитории, создаваемые для нескольких участников и известные только
им.

### Основные ветки

Главный репозиторий в течении всего времени жизни проекта содержит две ветки: `master` и `develop`.
В ветку `master` изменения попадают только при их окончательной готовности к релизу. Если строго
следовать этому правилу, ветка `master` всегда содержит исходный код последней опубликованной
версии ПО. С помощью тэгов, коммиты в ветке `master` помечаются
[семантическими версиями](http://semver.org/lang/ru/) релизов ПО и, с помощью систем автоматического развертывания, собираются в дистрибутивы и доставляются заказчикам.

Ветка `develop` является главной веткой процесса разработки, в которой накапливаются изменения в
процессе подготовки к очередному релизу. Как правило, к этой ветке подключена система непрерывной
интеграции, запускающая автоматизированное тестирование при каждом добавлении изменений.

Помимо основных, в проекте имеются вспомогательные ветки, которые удаляются после того, как выполнят
свою задачу. Их назначение и жизненный цикл описан ниже.

### Ветки новых возможностей (`feature`)

В `feature`-ветках происходит непосредственное развитие проекта. Название такой ветки может быть
произвольным, но не должно быть равно `master` или `develop`, а так же не должно начинаться со слов
`release-` и `hotfix-`. `Feature`-ветки должны ответвляться только от ветки `develop` и сливаться
обратно только с ней. Как правило, ветки `feature` не сохраняются в главном репозитории и существуют
только в локальных копиях разработчиков. Создаются такие ветки командой:

    git checkout -b <имя feature-ветки> develop

После завершения работы над конкретной функциональностью, которой была посвящена ветка, она
сливается с веткой `develop` командами:

    git checkout develop
    git merge <имя feature-ветки> --no-ff

Затем ветка удаляется, а изменения отправляются на `origin`:

    git branch -d <имя feature-ветки>
    git push origin develop

Флаг `--no-ff` обеспечивает создание коммита при слиянии, даже если изменения представляли собой перемотку вперед. Впоследствии это позволяет отследить историю изменений в проекте по журналу Git.

### Релизные ветки (`release`)

Эти ветки создаются для подготовки к очередному релизу. Когда команда принимает решение, что
достаточное количество стабильных изменений было добавлено к проекту, от ветки `develop`
ответвляется новая, с названием `release-<номер новой версии>`. С этого момента развитие проекта в
ветке `develop` может продолжиться, но новые изменения из нее уже не должны попадать в ветку
текущего релиза. Наоборот, доработки и исправления, которые будут внесены при подготовке релиза,
должны будут вливаться в `develop`. За счет этого, они будут учтены при добавлении новых
возможностей и подготовке будущих релизов. Номер очередной версии релиза заранее неизвестен,
назначается при ответвлении соответствующей ветки и в дальнейшем не меняется.

Релизная ветка создается командой `git checkout -b release-<номер версии> develop`. Далее, как
правило, новый номер версии вносится в исходные файлы проекта и формируется коммит с записью об этом
в журнале. Затем, код ветки тестируется и в нем исправляются ошибки, но существенные изменения и
новые возможности не вносятся. Такого рода развитие проекта должно будет пройти через
`feature`-ветки и слиться с `develop` для включения в будущие версии.

По окончании работы над релизом, соответствующая ветка сливается с веткой `master`. Для этого
выполняются следующие дествия:

    git checkout master
    git merge release-<версия релиза> --no-ff
    git tag -a <версия релиза>
    git push origin master
    git push origin --tags

Таким образом очередная версия становится готовой для автоматического развертывания. Так как она
помечается соответствующим тэгом, при необходимости к ней можно будет вернуться для доработки.

Далее, релизная ветка сливается с веткой `develop`, чтобы изменения, которые в ней накопились в ходе
подготовки, были задействованы в дальнейшей работе. Возможно, также, потребуется разрешение
конфликтов слияния.

    git checkout develop
    git merge release-<версия релиза> --no-ff

С этого момента ветка релиза не нужна и может быть удалена из репозитория, а изменения в ветках
`develop` и `master` -- отправлены на `origin`

    git branch -d release-<версия релиза> 
    git push origin develop

### Ветки работы над ошибками (`hotfix`)

Служат для исправления ошибок, выявленных в ходе эксплуатации уже опубликованных версий ПО.
Аналогичны релизным веткам, но создаются в экстренных случаях и изменяют только
[патч](http://semver.org/lang/ru/#section-1)-версию релиза. Как правило, версия продукта, в которой
была обнаружена ошибка, известна, и соответствующий коммит можно найти по ее тэгу. Ветка `hotfix`
ответвляется от этого коммита в `master`:

    git checkout <тэг версии с ошибкой>
    git checkout -b hotfix-<новая патч-версия> 

Далее следует изменить версию ПО в исходном коде и сформировать коммит с соответствующей записью в
журнале. После этого производится поиск и устранение ошибки в одном или более дополнительных
коммитов. По окончанию этой работы, ветка `hotfix` сливается с `master`.

    git checkout master
    git merge hotfix-<новая патч-версия> --no-ff
    git tag -a <новая патч-версия>
    git push origin master
    git push origin --tags

Затем изменения из `hotfix`-ветки сливаются в `develop` и ветка удаляется.

    git checkout develop
    git merge hotfix-<новая патч-версия> --no-ff
    git push origin develop
    git branch -d hotfix-<новая патч-версия>

Если на момент работы над `hotfix`-веткой параллельно ведется работа над `release`-веткой, то
изменения сливаются прежде всего с ней. Это гарантирует, что соответствующие исправления будут
включены в подготавливаемый релиз, а впоследствии -- и в `develop`. Также, в этом случае слияние
`hotfix` с `develop` уже не обязательно, но может производиться, если исправления требуют срочной
интеграции в процесс разработки.

В итоге, ветка `hotfix` безопасно удаляется командой `git branch -d hotfix-<новая патч-версия>`.

### Задание для самостоятельной работы

* Инициализируйте новый репозиторий и задайте для него локальный или удаленный `origin`
* Создайте в нем новый проект и ветки `master` и `develop`
* Подготовьте в проекте изменения, соответствующие новым возможностям
* Подготовьте релиз с присвоением очередного номера версии
* Произведите изменения, имитирующие исправления в уже опубликованном релизе
* Добавьте следующий релиз, учитывающий исправления и изменения в проекте

Все изменения должны производиться по соглашениям Git flow, приведенным в разделах выше.
