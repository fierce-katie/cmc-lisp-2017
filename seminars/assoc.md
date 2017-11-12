# Семинар 1 (09.11.2017)

## Ассоциативные списки

Ассоциативный список можно представить в виде списочной структуры языка Lisp
вида `((key_1 value_1) ... (key_n value_n))`, где все значения `key_i` различны,
атомарны и имеют одинаковый тип. В общем случае `value` может быть любым
S-выражением.

Если `l = ((key_1 value_1) ... (key_n value_n))`, то
```
    (key_1 value_1) = (car l)
    key_1 = (caar l)
    value_1 = (cadar l)
```

Задачи:

1. `(map-with-keys f l)` -- изменить значения в списке, функция `f` принимает в
   качестве аргументов ключ и старое значение:
```lisp
    (defun map-with-keys (f l)
        (cond
            ((null l) NIL)
            (T
                (let*
                    ((pair (car l))
                     (key (car pair))
                     (val (cadr pair))
                     (newval (funcall f key val))
                    )
                    (cons (list key newval) (map-with-keys f (cdr l)))
                )
            )
        )
    )
```

2. `(insert k v l)` -- добавить новое значение для ключа `k` (если для данного
   ключа уже есть значение, оно заменяется на новое):
```lisp
    (defun insert (k v l)
        (cond
            ((null l) (list (list k v)))
            ((equal k (caar l)) (cons (list k v) (cdr l)))
            (T (cons (car l) (insert k v (cdr l))))
        )
    )
```

3. `(delete-assoc k l)` -- удалить значение из ассоциативного списка:
```lisp
    (defun delete-assoc (k l)
        (cond
            ((null l) NIL)
            ((equal k (caar l)) (cdr l))
            (T (cons (car l) (delete-assoc k (cdr l))))
        )
    )
```

4. `(member-assoc e l)` -- проверить, есть ли в ассоциативном списке значение `e`:
```lisp
    (defun member-assoc (v l)
        (cond
            ((null l) NIL)
            ((equal v (cadar l)) T)
            (T (member-assoc v (cdr l)))
        )
    )
```

5. `(find-assoc k l)|` -- найти значение по ключу, вернуть `NIL`, если оно отсутствует:
```lisp
    (defun find-assoc (k l)
        (cond
            ((null l) NIL)
            ((equal k (caar l)) (cadar l))
            (T (find-assoc k (cdr l)))
        )
    )
```

6. `(adjust k f l)` -- применить функцию `f` к значению, находящемуся по ключу
   `k`:
```lisp
    (defun adjust (k f l)
        (cond
            ((null l) NIL)
            ((equal k (caar l)) (cons (list k (funcall f (cadar l))) (cdr l)))
            (T (cons (car l) (adjust k f (cdr l))))
        )
    )
```

7. `(filter f l)` -- оставить в списке только те пары, значения в которых
   удовлетворяют предикату `f`:
```lisp
    (defun filter (f l)
        (cond
            ((null l) NIL)
            ((funcall f (cadar l))
                (cons (car l) (filter f (cdr l)))
            )
            (T (filter f (cdr l)))
        )
    )
```
Решение, приведённое на семинаре (с использованием свёртки):
```lisp
    (defun filter (f l)
        (reduceR
            #'(lambda (x acc) ; x = (k v)
                (if (funcall f (cadr x))
                    (cons x acc)
                    acc
                )
            )
            NIL ; начальное значение acc
            l
        )
    )
```
Пары в списке будут проверяться, начиная с последней. Те пары, значения в
которых удовлетворяют предикату, будут добавляться в начало списка `acc`.
В результате в отфильтрованном списке пары будут идти в том же порядке, как
и в исходном.
Если использовать левую свёртку, то элементы будут обрабатываться начиная с
начала списка, поэтому для сохранения порядка элементов в свёртке нужно
использовать следующую функцию:
```lisp
    (lambda (acc x)
        (if (funcall f (cadr x))
            (append acc (list x)) ; вместо (cons x acc)
            acc)
        )
    )
```

8. `(union-assoc l1 l2)` -- объединить ассоциативные списки (в случае конфликта
   значений используется первое):
```lisp
    (defun union-assoc (l1 l2)
        (cond
            ((null l1) l2)
            ((null l2) l1)
            (T
                (let*
                    ((pair (car l1))
                     (key (car pair))
                    )
                    (cond
                        ((find-assoc key l2) (union-assoc (cdr l1) l2))
                        (T (cons pair (union-assoc (cdr l1) l2)))
                    )
                )
            )
        )
    )
```

## Ассоциативные списки с упорядоченными ключами

Задачи:

1. `(union-assoc-ord l1 l2)` -- объединить ассоциативные списки с
   упорядоченными ключами (в случае конфликта значений используется первое):
```lisp
    (defun union-assoc-ord (l1 l2)
        (cond
            ((null l1) l2)
            ((null l2) l1)
            (T
                (let*
                    ((p1 (car l1))
                     (p2 (car l2))
                     (k1 (car p1))
                     (k2 (car p2))
                    )
                    (cond
                        ((= k1 k2) (cons p1 (union-assoc-ord (cdr l1) (cdr l2))))
                        ((< k1 k2) (cons p1 (union-assoc-ord (cdr l1) l2)))
                        (T (cons p2 (union-assoc-ord l1 (cdr l2))))
                    )
                )
            )
        )
    )
```

2. `(find-ord k l)` -- найти значение по ключу, вернуть `NIL`, если оно отсутствует:
```lisp
    (defun find-ord (k l)
        (cond
            ((null l) NIL)
            ((= k (caar l)) (cadar l))
            ((> (caar l) k) NIL)
            (T (find-ord k (cdr l)))
        )
    )
```
