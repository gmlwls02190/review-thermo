# warmup/orders.py 테스트 실행 결과 보고서

## 1. 개요

- **대상 파일**: `warmup/orders.py`
- **테스트 파일**: `warmup/test_vorders.py`
- **테스트 프레임워크**: `pytest (v9.1.1)`
- **실행 일시**: 2026-10-23
- **실행 명령어**: `pytest -v warmup/test_vorders.py`

---

## 2. 테스트 요약

| 총 테스트 수 | 성공 (PASSED) | 실패 (FAILED) | 결과 |
|:---:|:---:|:---:|:---:|
| 5 | 1 | 4 | ❌ 실패 (버그 3종 검출) |

---

## 3. 테스트 케이스별 상세 결과 및 버그 분석

| 함수 | 테스트 함수 | 결과 | 검출된 버그 및 상세 분석 |
|---|---|:---:|---|
| `average_amount` | `test_average_amount_normal` | ✅ PASSED | 정상 주문 목록의 평균을 올바르게 계산 (`[300, 5000, 15200, 7900]` → `7100`). |
| `average_amount` | `test_average_amount_empty_list` | ❌ FAILED | **ZeroDivisionError 발생**: 빈 리스트(`[]`) 전달 시 `len(amounts)`가 0이 되어 0으로 나누기 오류가 발생함. |
| `add_tag` | `test_add_tag_independent_defaults` | ❌ FAILED | **가변 기본 인자(mutable default argument) 공유 버그**: `def add_tag(..., tags=[])` 형태로 정의되어 있어, 함수 호출 간에 동일한 리스트 인스턴스가 공유됨. 결과적으로 두 번째 주문의 tags에 이전 주문의 태그까지 누적됨. |
| `get_page` | `test_get_page_one` | ❌ FAILED | **1-based 오프셋 계산 버그**: `start = page * size`로 계산되어, `page=1, size=2`일 때 시작 인덱스가 0이 아닌 2(`1 * 2`)가 되어 `[1, 2]`가 아닌 `[3, 4]`를 반환함. (올바른 식: `(page - 1) * size`) |
| `get_page` | `test_get_page_two` | ❌ FAILED | **페이지 슬라이스 인덱스 버그**: `page=2, size=2`일 때 `start=4, end=6`이 되어 5번째 원소(`[5]`)만 반환됨. |

---

## 4. 실제 pytest 실행 로그

```text
============================= test session starts =============================
platform win32 -- Python 3.11.9, pytest-9.1.1, pluggy-1.7.0 -- C:\workspaces\review-thermo\.venv\Scripts\python.exe
cachedir: .pytest_cache
rootdir: C:\workspaces\review-thermo
configfile: pyproject.toml
plugins: anyio-4.15.1
collecting ... collected 5 items

warmup/test_vorders.py::test_average_amount_normal PASSED                [ 20%]
warmup/test_vorders.py::test_average_amount_empty_list FAILED            [ 40%]
warmup/test_vorders.py::test_add_tag_independent_defaults FAILED         [ 60%]
warmup/test_vorders.py::test_get_page_one FAILED                         [ 80%]
warmup/test_vorders.py::test_get_page_two FAILED                         [100%]

================================== FAILURES ===================================
_______________________ test_average_amount_empty_list ________________________

    def test_average_amount_empty_list():
        """주문 목록이 비어있을 때 ZeroDivisionError 없이 0을 반환하는지 검증한다."""
>       assert average_amount([]) == 0
               ^^^^^^^^^^^^^^^^^^

warmup\test_vorders.py:12: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

amounts = []

    def average_amount(amounts):
        """주문 금액의 평균을 구한다."""
        total = sum(amounts)  # 금액을 모두 더한다
>       return total / len(amounts)  # 개수로 나눈다
               ^^^^^^^^^^^^^^^^^^^^
E       ZeroDivisionError: division by zero

warmup\orders.py:11: ZeroDivisionError
______________________ test_add_tag_independent_defaults ______________________

    def test_add_tag_independent_defaults():
        """기본 태그 목록을 공유하지 않고 각 주문마다 독립적으로 태그가 생성되는지 검증한다."""
        first_order = add_tag({}, "전자제품")
        second_order = add_tag({}, "식품")
    
>       assert first_order["tags"] == ["전자제품"]
E       AssertionError: assert ['전자제품', '식품'] == ['전자제품']
E         
E         Left contains one more item: '식품'
E         
E         Full diff:
E           [
E               '전자제품',
E         +     '식품',
E           ]

warmup\test_vorders.py:20: AssertionError
______________________________ test_get_page_one ______________________________

    def test_get_page_one():
        """1페이지(1부터 시작) 요청 시 1번째부터 size개의 주문을 올바르게 반환하는지 검증한다."""
        orders = [1, 2, 3, 4, 5]
>       assert get_page(orders, page=1, size=2) == [1, 2]
E       AssertionError: assert [3, 4] == [1, 2]
E         
E         At index 0 diff: 3 != 1
E         
E         Full diff:
E           [
E         -     1,
E         ?     ^
E         +     3,
E         ?     ^
E               2,
E         +     4,
E           ]

warmup\test_vorders.py:27: AssertionError
______________________________ test_get_page_two ______________________________

    def test_get_page_two():
        """2페이지 요청 시 3번째부터 size개의 주문을 올바르게 반환하는지 검증한다."""
        orders = [1, 2, 3, 4, 5]
>       assert get_page(orders, page=2, size=2) == [3, 4]
E       AssertionError: assert [5] == [3, 4]
E         
E         At index 0 diff: 5 != 3
E         Right contains one more item: 4
E         
E         Full diff:
E           [
E         -     3,
E         -     4,
E               5,
E           ]

warmup\test_vorders.py:33: AssertionError
=========================== short test summary info ===========================
FAILED warmup/test_vorders.py::test_average_amount_empty_list - ZeroDivisionError: division by zero
FAILED warmup/test_vorders.py::test_add_tag_independent_defaults - AssertionError: assert ['전자제품', '식품'] == ['전자제품']
FAILED warmup/test_vorders.py::test_get_page_one - AssertionError: assert [3, 4] == [1, 2]
FAILED warmup/test_vorders.py::test_get_page_two - AssertionError: assert [5] == [3, 4]
========================= 4 failed, 1 passed in 0.06s =========================
```
