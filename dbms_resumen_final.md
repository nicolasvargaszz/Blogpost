# DBMS — Resumen Examen Final · 01/07

---

## 1. NORMALIZACIÓN

### Dependencia Funcional (FD)
`X → Y` — X determina Y.

### Clausura X⁺
Todos los atributos que X determina (directo + transitivo).  
Clave candidata = el mínimo X tal que **X⁺ = todos los atributos**.

### Formas Normales — condición para NO violar

| NF | Condición |
|---|---|
| 1NF | Todos los atributos son atómicos |
| 2NF | Todo atributo no-clave depende de la **clave completa** (no de un subconjunto) |
| 3NF | Para toda FD X→A: X es superkey **ó** A pertenece a alguna clave candidata |
| BCNF | Para toda FD no-trivial X→Y: X debe ser **superkey** (sin excepción) |

> BCNF más estricto que 3NF. Si viola BCNF pero cumple 3NF → el lado derecho (A) es parte de alguna clave candidata.

### Descomposición
- **Lossless:** `R1 ∩ R2 → R1` **ó** `R1 ∩ R2 → R2`
- **Dependency preserving:** todas las FDs originales verificables en alguna subrelación sin joins
- BCNF → siempre lossless, puede perder FDs
- 3NF → siempre preserva FDs

---

## 2. TIMESTAMP ORDERING PROTOCOL (TO)

Cada txn recibe **ts** al iniciar. Más antigua = ts menor.  
Cada dato guarda: `RTS(X)` (mayor ts que leyó X) · `WTS(X)` (mayor ts que escribió X).  
Estado inicial: `RTS = WTS = 0`.

### Reglas (aplicar operación por operación en orden del schedule)

| Operación | Verificar | Resultado |
|---|---|---|
| **READ R(X)** de Ti | `WTS(X) > ts(Ti)` ? | SÍ → ROLLBACK · NO → OK, `RTS(X) = max(RTS(X), ts)` |
| **WRITE W(X)** de Ti | `RTS(X) > ts(Ti)` ? | SÍ → ROLLBACK |
| | `WTS(X) > ts(Ti)` ? | SÍ → ROLLBACK · NO → OK, `WTS(X) = ts(Ti)` |

> **La condición es `>`, no `≥`. Igual no hace rollback.**  
> Después de un ROLLBACK: las ops siguientes de esa txn se omiten, pero los RTS/WTS ya actualizados quedan.

---

## 3. SERIALIZABILIDAD POR CONFLICTO (CS)

**Conflicto:** mismo dato + distintas txns + al menos una escritura → R-W · W-R · W-W.

**Grafo de precedencia:** nodo por txn, arista Ti→Tj si la op de Ti es anterior en el schedule.

| Resultado | Conclusión |
|---|---|
| Hay ciclo | NOT Conflict Serializable |
| Sin ciclo | CS — orden serial = sort topológico |

---

## 4. SERIALIZABILIDAD POR VISTA (VS)

CS ⇒ VS automáticamente. Solo aplicás las 3 reglas si hay ciclo pero **escrituras ciegas** (W sin R previo del mismo dato en la misma txn).

| Regla | Condición |
|---|---|
| 1 — Lectura inicial | Si Ti lee el valor inicial de X en S → igual en S' |
| 2 — Reads-from | Si Ti lee el valor escrito por Tj en S → igual en S' |
| 3 — Escritura final | El último escritor de X en S es el mismo en S' |

> Atajo: CS ⊂ VS — todo CS es VS. VS incluye casos con escrituras ciegas que CS no acepta.

---

## 5. CASCADING / CASCADELESS / STRICT

| Categoría | Dirty Read | Dirty Write | Condición |
|---|---|---|---|
| **Cascading** | ✓ SÍ | puede | Alguna txn lee dato escrito por otra que NO commiteó |
| **Cascadeless** | ✗ NO | puede | Toda lectura es de un valor ya commiteado |
| **Strict** | ✗ NO | ✗ NO | Nadie lee ni sobreescribe dato no commiteado |

**Jerarquía:** Strict ⊂ Cascadeless ⊂ Cascading

### Cómo identificar (checklist de 2 preguntas)
1. ¿Alguna txn **lee** X escrita por otra que no commiteó? → **CASCADING**, parar.
2. ¿Alguna txn **escribe** X escrita por otra que no commiteó? → **CASCADELESS** (no Strict).
3. Ninguna de las dos → **STRICT**.

---

## 6. LO QUE DIJO EL PROFE

- **"Just conditions"** → condición matemática, no párrafos
- **Justificar** → qué regla aplica y por qué, sin texto de relleno
- Sale: Normalization · Timestamp · Concurrency Control
- Formato: ejercicios, no definiciones

---

## TRAMPAS FRECUENTES

| Error | Correcto |
|---|---|
| `WTS(X) = ts` hace rollback en READ | NO — la condición es `>`, no `≥` |
| Mismo schedule siempre da el mismo resultado en TO | NO — distinto orden de inicio → distintos ts → distinto resultado |
| BCNF siempre es mejor que 3NF | BCNF puede perder dependency preservation |
| VS solo aplica si hay ciclo | CS también es VS, siempre |
| Dirty write = Cascading | NO — dirty write sin dirty read = Cascadeless (pero no Strict) |
