# Решение задачи о свойствах системы переписывания для исходной системы


Используя нефундированный порядок $\succ_{\text{L}}$:

- Читаем справа налево
- Лексикографически оцениваем ($\prec_{\text{lex}}$) по алфавиту: $a \prec b \prec c$
- Если суффиксы равны, то больше то слово, что длиннее: $\varepsilon \prec a$

И запустив алгоритм Кнуту-Бендикса, можно заметить, что он пополняет систему такими правилами, 
что появляется рост числа $a$ во время вычисления нормальных форм, из чего по логике
следует, что система не является завершаемой.

Тем не менее, я все же считаю, что система терменируема, 
так как $acc$ не могут вызывать новые переписывания бесконечно.

## Локальная конфлюэнтность и пополняемость по Кнуту-Бендиксу

Для исходной системы переписываний:

```math
\left\{ \begin{aligned}

    aabc  &\rightarrow bbaa \\
    b  &\rightarrow ccaa \\
    bc  &\rightarrow a \\
    aac  &\rightarrow ε \\

\end{aligned} \right.
```

### Локальная конфлюэнтность

1. Найдем критические пары

Рассмотрим строку $aabc$:

```mermaid
stateDiagram-v2
    aabc --> bbaa : aabc -> bbaa
        bbaa --> ccaabaa : b -> ccaa
            ccaabaa --> ccaaccaaaa : b -> ccaa
                ccaaccaaaa --> cccaaaa : aac -> ε
                    note right of cccaaaa: НФ
        bbaa --> bccaaaa : b -> ccaa
            bccaaaa --> acaaaa : bc -> a
                note right of acaaaa: НФ
            bccaaaa --> ccaaccaaa : b -> ccaa
                ccaaccaaa --> cccaaaa : aac -> ε

    aabc --> aaa : bc -> a
        note right of aaa: НФ

    aabc --> aaccaac : b -> ccaa
        aaccaac --> c : aac -> ε
            note right of c: НФ
```

Рассмотрим также строку $bc$:

```mermaid
stateDiagram-v2
    bc --> a : bc -> a
        note right of a: НФ
    bc --> ccaac : b -> ccaa
        ccaac --> cc : aac -> ε
            note right of cc: НФ
```

Как видно из диаграмм, многие ветви сходятся в различные нормальные формы, что доказывает отсутствие локальной и, как следствие, глобальной конфлюэнтности.

### Пополняемость по Кнуту-Бендиксу

Запустим алгоритм Кнута-Бендикса на нашей системе.

Найдем правила 

1. $cc \rightarrow a$ из строки $bb$
2. $ac \rightarrow ca$ из строки $ccc$
3. $caaa \rightarrow a$ из строки $bc$


```mermaid
stateDiagram-v2
    bc --> ccaac : b -> ccaa
        ccaac --> ccaca : ac -> ca
            ccaca --> cccaa : ac -> ca
                cccaa --> acaa : cc -> a
                    acaa --> caaa : ac -> ca
                        note right of caaa: НФ
                cccaa --> caaa : cc -> a
                    note right of caaa: НФ
            ccaca --> aaca : cc -> a
                aaca --> acaa : ac -> ca
                    acaa --> caaa : ac -> ca
                        note right of caaa: НФ
                aaca --> a : aac -> ε
                    note right of a: НФ
        ccaac --> cc : aac -> ε
            cc --> a : cc -> a
                note right of a: НФ
        ccaac --> aaac : cc -> a
            aaac --> aaca : ac -> ca
                aaca --> acaa : ac -> ca
                    acaa --> caaa : ac -> ca
                        note right of caaa: НФ
                aaca --> a : aac -> ε
                    note right of a: НФ
            aaac --> a : aac -> ε
                note right of a: НФ
    bc --> a : bc -> a
        note right of a: НФ

```

4. $ca \rightarrow aaaa$ из строки $caaac$

```mermaid
stateDiagram-v2
    caaac --> ac : caaa -> a
        ac --> ca : ac -> ca
            note right of ca: НФ
    caaac --> caaca : ac -> ca
        caaca --> cacaa : ac -> ca
            cacaa --> ccaaa : ac -> ca
                ccaaa --> ca : caaa -> a
                    note right of ca: НФ
                ccaaa --> aaaa : cc -> a
                    note right of aaaa: НФ
        caaca --> ca : aac -> ε
            note right of ca: НФ
    caaac --> ca : aac -> ε
        note right of ca: НФ
```

5. $aaaaaa \rightarrow a$ из строки $caaa$

```mermaid
stateDiagram-v2
    caaa --> a : caaa -> a
        note right of a: НФ
    caaa --> aaaaaa : ca -> aaaa
        note right of aaaaaa: НФ
```


Далее программа [реализации алгоритма Кнуту-Бендикса](src/bin/kb.rs) 
застревает на этапе поиска нормальных форм слова $aaaaaabc$
```
Current string: aacaaaaaaaaaaaacaaaaaaaaaaaaaaaaaaaaaa
Normal forms so far: {aa}
Processing strings: {cacaabaacaaaaa, aaacaabaacaaaaaa, caaabcaaaaaaaaaa, aabcaaaaaaacaaaaaaaaaaaaaa, bbacaaaaaaaaaaaaaaaaaaaaaaaaaaa, ...}
Normal forms so far: {aa}
Applying rule aaaaaa -> a on acaaaaaaaaaaaaaacaaaaaaaaaaaaaaaaaaaaa with result {acaaaaaaaaacaaaaaaaaaaaaaaaaaaaaa}
Applying rule aaaaaa -> a on acaaaaaaaaaaaaaacaaaaaaaaaaaaaaaaaaaaa with result {acaaaaaaaaacaaaaaaaaaaaaaaaaaaaaa}
...
```