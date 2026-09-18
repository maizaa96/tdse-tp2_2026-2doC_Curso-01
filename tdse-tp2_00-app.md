### Opción 1: Estructura Jerárquica

- **`task_dta_list`** (`task_dta_t [3]`)
  - **`task_dta_list[0]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 4
    - `BCET` (`uint32_t`): 4
    - `WCET` (`uint32_t`): 7
  - **`task_dta_list[1]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 3
    - `BCET` (`uint32_t`): 3
    - `WCET` (`uint32_t`): 5
  - **`task_dta_list[2]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 2
    - `BCET` (`uint32_t`): 2
    - `WCET` (`uint32_t`): 4

---

### Opción 2: Formato de Tabla

| Elemento | NOE | LET | BCET | WCET | Tipo de Dato |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `task_dta_list[0]` | 83967 | 4 | 4 | 7 | `uint32_t` |
| `task_dta_list[1]` | 83967 | 3 | 3 | 5 | `uint32_t` |
| `task_dta_list[2]` | 83967 | 2 | 2 | 4 | `uint32_t` |
