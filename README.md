# IKT Portfolio 2. pololetí

### Jméno: `Daniel Škrob`
### Téma: `Algoritmizace` `Strojové učení`

---

# Plán učení

| Plánované činnosti / dovednosti, které se naučím                       | Odhadovaný počet týdnů |
|------------------------------------------------------------------------|------------------------|
| Dynamické Programování                                                 | 2 týdny |
| Pokročilé grafové algoritmy a datové struktury                         | 2 týdny |
| Seznámení s PyTorch                                                    | 1 týden |
| Tvorba závěrečného ML projektu (např. rozpoznávání ručně psaných číslic) | 4 týdny |

---

# Dynamické Programování

Dynamické programování je technika, která řeší problémy rozdělením na **menší překrývající se podproblémy** — místo jejich opakovaného výpočtu si výsledky **zapamatujeme** (memoizace).

## Příklad: Fibonacciho čísla

Naivní rekurzivní řešení má exponenciální složitost `O(2ⁿ)`, protože stejné podproblémy počítá opakovaně.
```python
def fib(n):
    if n == 0:
        return 0
    if n == 1:
        return 1
    return fib(n-1) + fib(n-2)

print(fib(6))
```

Například `fib(1)` je při výpočtu `fib(5)` volána až **5× zbytečně**:

![Fibonacci memoization](https://miro.medium.com/v2/resize:fit:1400/1*YP_lKxOiTw3qW-s-d6kxuA.png)

## Řešení: Memoizace

Výsledky podproblémů si uložíme do slovníku a každý podproblém vypočítáme **pouze jednou** → složitost klesne na `O(n)`.
```python
memo = {}

def fib(n):
    if n in memo:
        return memo[n]
    if n == 0:
        return 0
    if n == 1:
        return 1
    memo[n] = fib(n-1) + fib(n-2)
    return memo[n]

print(fib(6))
```

## Řešení: Tabulace

Na rozdíl od memoizace (přístup shora dolů) tabulace řeší problém **zdola nahoru** — začneme od nejmenších podproblémů a postupně stavíme výsledek.
```python
def fib(n):
    if n == 0:
        return 0
    
    table = [0] * (n + 1)
    table[1] = 1

    for i in range(2, n + 1):
        table[i] = table[i-1] + table[i-2]
    
    return table[n]

print(fib(6))
```

Výsledky ukládáme do pole `table`, kde každý prvek závisí pouze na dvou předchozích. Složitost je opět `O(n)`, ale nepotřebujeme rekurzi ani slovník.

### Porovnání přístupů

| Přístup | Směr | Struktura | Rekurze |
|---|---|---|---|
| Naivní rekurze | shora dolů | — | ✅ |
| Memoizace | shora dolů | slovník | ✅ |
| Tabulace | zdola nahoru | pole | ❌ |

## Příklad: Coin Exchange Problem (Problém výměny mincí)

Máme sadu mincí o určitých nominálních hodnotách a cílovou částku. Snažíme se najít **minimální počet mincí** (a jejich konkrétní typy), které jsou potřeba ke složení této částky.

Níže uvádím dvě řešení pomocí tabulace. Obě postupují od nuly a postupně staví nejoptimálnější způsob pro sestavení každé hodnoty až po `target`.

### 1. Řešení: Ukládání celých sekvencí

V tomto přístupu si pro každou částku v paměti (`memo`) ukládáme přímo seznam mincí, které ji tvoří. Je to přímočaré, ale náročné na paměť při velkých cílových částkách.
```python
memo = {0: []}

nominals = {1, 3, 4}
target = 14

def calc_min(x):
    m = []
    for n in nominals:
        node = x - n
        if node in memo:
            candidate = memo[node] + [n]
            if len(m) == 0 or len(candidate) < len(m):
                m = candidate
    return m

for nominal in nominals:
    memo[nominal] = [nominal]

for i in range(1, target + 1):
    memo[i] = calc_min(i)

print(f"Částka {target}: {memo[target]}")
```

### 2. Řešení: Rekonstrukce pomocí parentů (Backtracking)

Druhé řešení je optimalizované. Místo celých seznamů si ukládáme pouze:
- `dp` — minimální počet mincí pro danou částku
- `parent` — odkaz na poslední použitou minci, která k tomuto minimu vedla

Výsledný seznam mincí pak získáme zpětně pomocí **backtrackingu**. Tento přístup je mnohem efektivnější a umožňuje řešit i velmi vysoké cílové částky (např. `10⁷`).
```python
import math

nominals = {1, 3, 4}
target = 14

dp = [math.inf] * (target + 1)
parent = [math.inf] * (target + 1)
dp[0] = 0

for nominal in nominals:
    if nominal <= target:
        dp[nominal] = 1
        parent[nominal] = nominal

for i in range(1, target + 1):
    if dp[i] != math.inf:
        continue

    candidate = math.inf
    n_candidate = math.inf

    for n in nominals:
        node = i - n
        if node >= 0:
            new_value = dp[node] + 1
            if new_value < candidate:
                candidate = new_value
                n_candidate = n

    dp[i] = candidate
    parent[i] = n_candidate

print(f"Minimální počet mincí: {dp[target]}")

# Backtracking — získání seznamu použitých mincí:
curr = target
while curr > 0:
    coin = parent[curr]
    print(coin)
    curr -= coin
```

### Porovnání přístupů

| Metoda | Co se ukládá | Paměťová náročnost | Výhoda |
|---|---|---|---|
| Seznamy | Celá pole mincí | Vysoká `O(n·k)` | Jednoduchý kód |
| Parenti | Jen jedno číslo | Nízká `O(n)` | Škálovatelnost pro velká data |