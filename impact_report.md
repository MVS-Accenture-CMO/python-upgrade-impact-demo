# Python 3.12+ Upgrade Impact Report

## Executive summary
Upgrading this repository to Python 3.12 will fail at import/compile time due to:
- Removed standard library modules: `imp` and `asyncore` (PEP 594 removals)
- Python 2-only syntax in `src/legacy_calc.py` (print statement)
- Outdated dependencies in `requirements.txt` (`numpy==1.18.0`, `requests==2.19.0`) that do not officially support Python 3.12

Overall risk rating: HIGH

---

## Files impacted

- `src/legacy_calc.py`
  - Line 1: `import imp`
    - Category: Removed stdlib module (PEP 594)
    - Impact: Raises `ModuleNotFoundError` on Python 3.12+
    - Why it breaks: `imp` has been deprecated for years and was removed; functionality moved to `importlib`
    - Compatible alternative: Replace with `importlib` equivalents, e.g. `importlib.import_module`, `importlib.reload`, or `importlib.util.spec_from_file_location(...); module_from_spec(...); spec.loader.exec_module(module)`
  - Line 2: `import asyncore`
    - Category: Removed stdlib module (PEP 594)
    - Impact: Raises `ModuleNotFoundError` on Python 3.12+
    - Why it breaks: `asyncore` (and `asynchat`, `smtpd`) were removed by PEP 594; migration is to `asyncio` or higher-level libraries
    - Compatible alternative: Use `asyncio` (e.g., `asyncio.start_server`, `asyncio.open_connection`, or asyncio streams) or remove if unused
  - Line 5: `print "Result:", a + b`
    - Category: Incompatible Python syntax (Python 2 print statement)
    - Impact: Compile-time `SyntaxError` under Python 3.x, preventing import and execution
    - Why it breaks: Python 3 requires `print(...)` function calls
    - Compatible alternatives: `print("Result:", a + b)` or `print(f"Result: {a + b}")`

- `src/legacy_utils.py`
  - No breaking changes detected for Python 3.12. Uses `collections.Counter`, which is still supported.

---

## Issue categories
- Removed/Deprecated stdlib modules (PEP 594 removals): `imp`, `asyncore`
- Incompatible syntax: Python 2 print statement
- Dependency support gaps for Python 3.12

---

## Dependency risks (`requirements.txt`)
- `numpy==1.18.0`
  - Risk: No wheels for Python 3.12; unsupported on modern CPython versions
  - Impact: Installation likely fails (wheel not found or build from source fails)
  - Recommendation: Upgrade to `numpy>=1.26` (or latest stable compatible with 3.12)
- `requests==2.19.0`
  - Risk: Very old release; not tested against Python 3.12 and may constrain `urllib3` to legacy versions without 3.12 support
  - Impact: May install but is unsupported; potential runtime/network stack incompatibilities
  - Recommendation: Upgrade to `requests>=2.31`

---

## Suggested remediation
1) Replace removed stdlib modules
   - `imp` → `importlib` equivalents
   - `asyncore` → `asyncio` (or remove if unused)
2) Fix Python 3 syntax
   - Update print statement to an f-string or `print(...)`
3) Update dependencies for Python 3.12 support
   - `numpy` to `>=1.26` (or the latest compatible)
   - `requests` to `>=2.31`
4) Optional CI hardening
   - Add a `python -m py_compile` step on Python 3.12 and fail the job on syntax/import errors (remove `|| true` if present)

---

## Overall upgrade risk rating
HIGH — current code will not import/compile on Python 3.12 and key dependencies are below supported versions. Addressing the small code changes plus dependency bumps should reduce risk to LOW.
