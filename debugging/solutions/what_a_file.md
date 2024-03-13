### 1. 
От нас просили найти имя файла. В подсказках есть указание на системный вызов `KernelBase::CreateFileW`.
Будем искать этот символ.

```
0:000> x KernelBase!CreateFileW
00007ffb`c7255d70 KERNELBASE!CreateFileW (CreateFileW)
``` 

### 2. 
Ставим туда breakpoint

```
bp 00007ffb`c7255d70
или
bp KERNELBASE!CreateFileW
```

### 3. 
Перезапускаем или жмём далее и ловим брейкпоинт

```
0:000> g
Breakpoint 0 hit
KERNELBASE!CreateFileW:
00007ffb`c7255d70 4883ec58        sub     rsp,58h
```

### 4. 
Надо понимать, как работает вызов этой системной функции. Можно посмотреть в гугле.
Первым аргументом там идёт имя файла. Оно кладётся в регистр `rcx`.
То есть нам просто нужно вывести то, что в нём находится при заходе в функцию.

```
0:000> du rcx
0000019d`ce7e5910  "c6ef63f2-794a-4421-b79d-903a9740"
0000019d`ce7e5950  "56de"
```

Если значение не влезает, `du` его переносит. При вызове `dc` становится понятнее.

```
0:000> dc rcx
0000019d`ce7e5910  00360063 00660065 00330036 00320066  c.6.e.f.6.3.f.2.
0000019d`ce7e5920  0037002d 00340039 002d0061 00340034  -.7.9.4.a.-.4.4.
0000019d`ce7e5930  00310032 0062002d 00390037 002d0064  2.1.-.b.7.9.d.-.
0000019d`ce7e5940  00300039 00610033 00370039 00300034  9.0.3.a.9.7.4.0.
0000019d`ce7e5950  00360035 00650064 abab0000 abababab  5.6.d.e.........
0000019d`ce7e5960  abababab abababab feeeabab feeefeee  ................
0000019d`ce7e5970  feeefeee feeefeee feeefeee feeefeee  ................
0000019d`ce7e5980  00000000 00000000 00000000 00000000  ................
```

Видим, что это какой-то гуид `c6ef63f2-794a-4421-b79d-903a974056de`

### 5.
Если натыкаться на последующие брейкпоинты, то увидим какие-то другие гуиды. В целом это и есть решение.
Можно написать цикл, который делает всё то же самое (втыкается в bp и делает printf rcx).