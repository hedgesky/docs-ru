# RSpecStamp

RSpecStamp позволяет автоматически добавлять _теги_ упавшим тестам (теги добавляются непосредственно в исходный код).

Основная задача RSpecStamp — упрощать рефакторинг тестов. Например, при смене глобальных настроек часть тестов может упасть. Вместо того, чтобы переписывать каждый тест, мы можем пометить их специальным тегом и подключать для него shared context, сохраняющий прошлое поведение.

## Пример: миграция с `Sidekiq::Testing.inline!` на `Sidekiq::Testing.fake!`

Использование `Sidekiq::Testing.inline!` в тестах по умолчанию считается _плохой практикой_ (см. [здесь](https://github.com/mperham/sidekiq/issues/3495)), так как это негативно сказывается на времени выполнения тестов. Тем не менее, такая конфигурация встречается довольно часто.

С помощью RSpecStamp можно быстро и без лишней работы мигрировать с `inline!` режима на `fake!`.
Для этого нужно выполнить следующие шаги.

Шаг 0. Для начала убедитесь, что все тесты проходят.

Шаг 1. Добавьте shared context для включения `inline!` режима для конкретного теста или группы:

```ruby
shared_context "sidekiq:inline", sidekiq: :inline do
  around(:each) { |ex| Sidekiq::Testing.inline!(&ex) }
end
```

Шаг 2. Переключитесь на `fake!` режим.

Шаг 3. Выполните команду `RSTAMP=sidekiq:inline bundle exed rspec`. Вероятно, часть тестов упадёт из-за изменения режима работы Sidekiq.

В финальном выводе вы увидите отчёт RSpecStamp о проделанной работе:

- Сколько файлов было исправлено.
- Сколько изменений было сделано.
- Сколько изменений не получилось применить.
- Сколько файлов было проигнорировано (см. ниже).

Шаг 4. Запустите тесты ещё раз — теперь они снова должны быть все «зелёными»!

**Примечание**: Вы можете запустить RSpecStamp в режиме предпросмотра, указав переменную окружения `RSTAMP_DRY_RUN=1`.

## Настройки

RSpecStamp по умолчанию игнорирует тесты из папки `spec/support` (где, как правило, хранятся shared examples).
Вы можете добавить дополнительные паттерны для исключения:

```ruby
TestProf::RSpecStamp.configure do |config|
  config.ignore_files << %r{spec/my_directory}
end
```
