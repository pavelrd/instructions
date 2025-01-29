# Флаги компилятора gcc повышающие безопасность программы

Включить вывод всех предупреждений:

<table>
  <tr><td>-Wall -Wextra </td></tr>
</table>

Обязательные флаги:

<table>
  <tr><td>-Werror=return-type -Werror=float-equal -Werror=unused-variable -Werror=switch -Werror=implicit-fallthrough -Werror=free-nonheap-object -Werror=sequence-point -Werror=return-local-addr -Werror=format=2 -Werror=shadow -Werror=duplicated-cond -Wduplicated-branches -Werror=type-limits -Werror=logical-op -Werror=parentheses -Werror=char-subscripts -Werror=address -Werror=memset-elt-size -Werror=memset-transposed-args</td></tr>
</table>

Рекомендуемые флаги:

<table>
  <tr><td>-Werror=uninitialized -Werror=init-self -Werror=conversion -Werror=old-style-cast</td></tr>
</table>

Флаги применяемые <b>только в отладочной сборке</b>. Существенно, в разы или даже десятки раз замедляют скорость работы программы, но позволяют найти серьезные ошибки во время выполнения программы. Работают только под Linux. Под Windows либо не работают либо работают частично (по состоянию на 2025 год). Одновременно использовать эти флаги нельзя, только по отдельности. Сначала первый, затем второй, или делать 2 отдельные сборки:

<table>
  <tr><td>-fsanitize=undefined</td><td>Находит UB(undefined behaviour)</td></tr>
  <tr><td>-fsanitize=address</td><td>Находит некорректное обращение к памяти кучи</td></tr>
</table>
