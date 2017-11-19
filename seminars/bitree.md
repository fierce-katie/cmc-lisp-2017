# Семинар 2 (16.11.2017)

## Двоичные деревья

Двоичное дерево может быть либо пустым, либо представлять собой узел, хранящий
атомарное значение и два поддерева (левое и правое):
```
  Tree ::= NIL
         | (atom Tree Tree)
```
Например:
```
       1               (1
      / \                (2
     2   3                 (4 NIL NIL)
    / \                    (5 NIL NIL)
   4   5                 )
                         (3 NIL NIL)
                       )
```

1. `(maptree f t)` -- применить функцию `f` ко всем элементам дерева:
```lisp
    (defun maptree (f tree)
        (if (null tree)
            NIL
            (let
                ((node (car tree))
                 (left (cadr tree))
                 (right (caddr tree))
                )
                (list (funcall f node) (maptree f left) (maptree f right))
            )
        )
    )
```

3. `(depth t)` -- подсчитать глубину дерева:
```lisp
    (defun depth (tree)
        (if (null tree)
            0
            (+ 1 (max (depth (cadr tree)) (depth (caddr tree))))
        )
    )
```

4. `(isomorphic t1 t2)` -- проверить, являются ли два дерева изоморфными:
```lisp
    (defun isomorphic (x y)
        (cond
            ((null x) (null y))
            ((null y) (null x))
            ((isomorphic (cadr x) (cadr y))
                (isomorphic (caddr x) (caddr y))
            )
            (T NIL)
        )
    )
```

5. `(is-tree s)` -- проверить, является ли произвольное S-выражение двоичным деревом:
```lisp
    (defun is-tree (s)
        (cond
            ((null s) T)
            ((listp s)
                (and
                    (= (length s) 3)
                    (atom (car s))
                    (is-tree (cadr s))
                    (is-tree (caddr s))
                )
            )
            (T NIL)
        )
    )
```

6. `(foldtree f a t)` -- реализовать свёртку для дерева (функция `f` от трёх аргументов:
значение в текущем узле, результат свёртки левого поддерева и правого поддерева):
```lisp
    (defun foldtree (f acc tree)
        (if
            (null tree)
            acc
            (let*
                ((node (car tree))
                 (left (cadr tree))
                 (right (caddr tree))
                 (leftres (foldtree f acc left))
                 (rightres (foldtree f acc right))
                )
                (funcall f node leftres rightres)
            )
        )
    )
```
Подсчёт суммы элементов в дереве (предполагается, что в узлах числовые значения):
```lisp
    (setq tree
       '(1
            (2
                (4 NIL NIL)
                (5 NIL NIL)
            )
            (3 NIL NIL)
        )
    )
    (foldtree #'+ 0 tree) ; 0 -- значение для пустого дерева
```
Процесс свёртки:
```

       (+ 1 11 3) = 15 --> 1
                          / \
                         /   \
     (+ 2 4 5) = 11 --> 2     3 <-- (+ 3 0 0) = 3
                       / \
                      /   \
   (+ 4 0 0) = 4 --> 4     5 <-- (+ 5 0 0) = 5

```

7. `(to-list t)` -- получить список элементов дерева (с помощью свёртки):
```lisp
    (defun to-list (tree)
        (foldtree #'(lambda (n l r) (cons n (append l r))) () tree)
    )
```

## Двоичные деревья поиска

Свойства двоичного дерева поиска (предполагается, что все значения в дереве числовые):
- у всех узлов **левого** поддерева значения **меньше**, чем значение в текущем узле;
- у всех узлов **правого** поддерева значения **больше**, чем значение в текущем узле.

Например:
```
          8                 (8
         / \                    (3
        /   \                       (1 NIL NIL)
       3    13                      (6 NIL NIL)
      / \   /                   )
     1   6 10                   (13
                                    (10 NIL NIL)
                                    NIL
                                )
                            )
```

1. `(find-stree e tree)` -- проверить, содержится ли в дереве данное значение (вернуть `T` или `NIL`):
```lisp
    (defun find-stree (e tree)
        (cond
            ((null tree) NIL)
            ((= e (car tree)) T)
            ((< e (car tree)) (find-stree e (cadr tree)))
            (T (find-stree e (caddr tree)))
        )
    )
```

2. `(maxs t)` -- найти максимальный элемент в дереве поиска (`NIL` для пустого дерева):
```lisp
    (defun maxs (tree)
        (cond
            ((null tree) NIL)
            ((null (caddr tree)) (car tree))
            (T (maxs (caddr tree)))
        )
    )
```
С использованием свёртки (работает для произвольного двоичного дерева с числовыми значениями в узлах):
```lisp
    (defun max-tree (tree)
        (foldtree #'max (car tree) tree)
    )
```
