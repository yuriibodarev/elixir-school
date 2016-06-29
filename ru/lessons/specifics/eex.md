---
layout: page
title: Embedded Elixir (EEx)
category: specifics
order: 3
lang: ru
---

По аналогии с ERB в Ruby и JSP в Java, в Elixir присутствует EEx или Embedded Elixir ("встраиваемый" Elixir код). С помощью EEx мы можем встраивать и интерпретировать код Elixir внутри любого другого текста.

{% include toc.html %}

## API

API EEx позволяет напрямую работать как со строками, так и с файлами.  API разделено на три основных компонента: простая интерпретация, определение функций и компиляция в AST (Абстрактное синтаксическое дерево).

### Интерпретация

Используя функции [`eval_string/3`](http://elixir-lang.org/docs/stable/eex/EEx.html#eval_string/3) и [`eval_file/2`](http://elixir-lang.org/docs/stable/eex/EEx.html#eval_file/3) мы можем выполнить интерпретацию кода, содержащегося в строке или файле. Это самые простые функции в API но и самые медленные, т.к. код каждый раз интерпретируется снова а не компилируется заранее.

```elixir
iex> EEx.eval_string "Hi, <%= name %>", [name: "Sean"]
"Hi, Sean"
```

### Определение функций

Более производительный и предпочтительный способ использования EEx, это создание функции из нашего шаблона с кодом, определив её внутри модуля который будет скомпилирован. Для этого мы должны подготовить шаблон до начала компиляции и использовать макросы [`function_from_string/5`](http://elixir-lang.org/docs/stable/eex/EEx.html#function_from_string/5) или [`function_from_file/5`](http://elixir-lang.org/docs/stable/eex/EEx.html#function_from_file/5).

Давайте разместим код нашего приветствия в отдельном файле и сгенерируем функцию `greeting` из нашего шаблона:

```elixir
# greeting.eex
Hi, <%= name %>

defmodule Example do
  require EEx
  EEx.function_from_file :def, :greeting, "greeting.eex", [:name]
end

iex> Example.greeting("Sean")
"Hi, Sean"
```

### Компиляция

И напоследок, EEx даёт нам возможность напрямую  генерировать абстрактное синтаксическое дерево Elixir из строки или файла с помощью функций [`compile_string/2`](http://elixir-lang.org/docs/stable/eex/EEx.html#compile_string/2) и [`compile_file/2`](http://elixir-lang.org/docs/stable/eex/EEx.html#compile_file/2). Хотя эти функции обычно используются внутри приведённых выше, они доступны вам на тот случай, если вы захотите реализовать собственную обработку встраиваемого Elixir кода.

## Tags

По умолчанию в шаблонах EEx поддерживаются четыре тега:

```elixir
<% выражение Elixir - выполнить код выражения %>
<%= выражение Elixir - заменить результатом вычисления выражения %>
<%% EEx цитирование - возвращает текст внутри тега %>
<%# комментарии - комментарии игнорируются %>
```

Все выражения, результат вычисления которых вы хотите увидеть в тексте, __должны__ использовать знак равенства (`=`).  Важно отметить - другие языки шаблонов трактуют условные выражения, такие как `if`, особым образом, в EEx это не так.  Без `=` ничего не отобразится в результирующем тексте:

```elixir
<%= if true do %>
  A truthful statement
<% else %>
  A false statement
<% end %>
```

## Интерпретатор

По умолчанию Elixir использует интерпретатор `EEx.SmartEngine`, который включает поддержку присваиваний (assignments) (таких, как `@name`):

```elixir
iex> EEx.eval_string "Hi, <%= @name %>", assigns: [name: "Sean"]
"Hi, Sean"
```

Присваивания в интерпретаторе `EEx.SmartEngine` полезны, потому что присваиваемые значения можно изменять без необходимости перекомпиляции шаблона.

Интересуетесь написания собственного интерпретатора? Всё что для этого необходимо вы можете подчерпнуть из документации на  [`EEx.Engine`](http://elixir-lang.org/docs/stable/eex/EEx.Engine.html).
