# warmup/orders.py 버그 수정 및 테스트 통과 보고서 (`goal_orders.md`)

## 1. 개요 및 목표

- **목표**: `warmup/test_orders.py`의 모든 테스트 케이스가 통과하도록 `warmup/orders.py`의 버그를 분석 및 수정
- **대상 파일**:
  - 원본 코드: [orders.py](file:///c:/workspaces/review-thermo/warmup/orders.py)
  - 검증 테스트: [test_orders.py](file:///c:/workspaces/review-thermo/warmup/test_orders.py)
- **최종 결과**: `8 passed in 0.03s` (모든 테스트 통과)

---

## 2. 그동안 쌓인 실패 로그

스케줄러 주기 실행 및 코드 수정 전 실행에서 검출된 실패 로그 요약입니다.

### 초기 및 주기 실행 시 실패 로그 (pytest warmup/test_orders.py)

```text
================================== FAILURES ===================================
__________________ test_add_tag_mutable_default_argument_bug __________________

    def test_add_tag_mutable_default_argument_bug():
        """
        기본 인자 tags=[]가 호출 간 공유되는 가변 기본 인자 버그 감지.
        독립된 두 호출 시 태그 목록이 누적되지 않아야 한다.
        """
        order1 = {"id": 1}
        order2 = {"id": 2}
    
        add_tag(order1, "tag1")
        add_tag(order2, "tag2")
    
        # order2는 'tag1'이 누적되지 않고 ['tag2']만 가지고 있어야 함
>       assert order2["tags"] == ["tag2"]
E       AssertionError: assert ['new', 'tag1', 'tag2'] == ['tag2']
E         
E         At index 0 diff: 'new' != 'tag2'
E         Left contains 2 more items, first extra item: 'tag1'
E         Use -v to get more diff

warmup\test_orders.py:40: AssertionError
__________________ test_get_page_pagination[1-10-expected0] ___________________

page = 1, size = 10, expected = [0, 1, 2, 3, 4, 5, ...]

    @pytest.mark.parametrize(
        ("page", "size", "expected"),
        [
            (1, 10, list(range(0, 10))),     # 1페이지는 0~9번 항목이어야 함 (현재 버그: 10~19 반환)
            (2, 10, list(range(10, 20))),    # 2페이지는 10~19번 항목이어야 함
            (3, 10, [20, 21, 22]),           # 마지막 부분 페이지
        ],
    )
    def test_get_page_pagination(page, size, expected):
        """1-based 페이지네이션이 의도대로 동작하는지 검증 (1페이지 오프셋 버그 감지)."""
        orders = list(range(23))
>       assert get_page(orders, page=page, size=size) == expected
E       assert [10, 11, 12, 13, 14, 15, ...] == [0, 1, 2, 3, 4, 5, ...]
E         
E         At index 0 diff: 10 != 0
E         Use -v to get more diff

warmup\test_orders.py:57: AssertionError
__________________ test_get_page_pagination[2-10-expected1] ___________________

page = 2, size = 10, expected = [10, 11, 12, 13, 14, 15, ...]

    @pytest.mark.parametrize(
        ("page", "size", "expected"),
        [
            (1, 10, list(range(0, 10))),
            (2, 10, list(range(10, 20))),
            (3, 10, [20, 21, 22]),
        ],
    )
    def test_get_page_pagination(page, size, expected):
        orders = list(range(23))
>       assert get_page(orders, page=page, size=size) == expected
E       assert [20, 21, 22] == [10, 11, 12, 13, 14, 15, ...]
E         
E         At index 0 diff: 20 != 10
E         Right contains 7 more items, first extra item: 13
E         Use -v to get more diff

warmup\test_orders.py:57: AssertionError
__________________ test_get_page_pagination[3-10-expected2] ___________________

page = 3, size = 10, expected = [20, 21, 22]

    @pytest.mark.parametrize(
        ("page", "size", "expected"),
        [
            (1, 10, list(range(0, 10))),
            (2, 10, list(range(10, 20))),
            (3, 10, [20, 21, 22]),
        ],
    )
    def test_get_page_pagination(page, size, expected):
        orders = list(range(23))
>       assert get_page(orders, page=page, size=size) == expected
E       assert [] == [20, 21, 22]
E         
E         Right contains 3 more items, first extra item: 20
E         Use -v to get more diff

warmup\test_orders.py:57: AssertionError
_________________________ test_get_page_default_size __________________________

    def test_get_page_default_size():
        """기본 size=10을 적용하여 1페이지를 요청했을 때 첫 10개 항목이 반환되는지 검증."""
        orders = list(range(15))
>       assert get_page(orders, page=1) == list(range(10))
E       assert [10, 11, 12, 13, 14] == [0, 1, 2, 3, 4, 5, ...]
E         
E         At index 0 diff: 10 != 0
E         Right contains 5 more items, first extra item: 5
E         Use -v to get more diff

warmup\test_orders.py:63: AssertionError
=========================== short test summary info ===========================
FAILED warmup/test_orders.py::test_add_tag_mutable_default_argument_bug - AssertionError: assert ['new', 'tag1', 'tag2'] == ['tag2']
FAILED warmup/test_orders.py::test_get_page_pagination[1-10-expected0] - assert [10, 11, 12, 13, 14, 15, ...] == [0, 1, 2, 3, 4, 5, ...]
FAILED warmup/test_orders.py::test_get_page_pagination[2-10-expected1] - assert [20, 21, 22] == [10, 11, 12, 13, 14, 15, ...]
FAILED warmup/test_orders.py::test_get_page_pagination[3-10-expected2] - assert [] == [20, 21, 22]
FAILED warmup/test_orders.py::test_get_page_default_size - assert [10, 11, 12, 13, 14] == [0, 1, 2, 3, 4, 5, ...]
5 failed, 3 passed in 0.08s
```

---

## 3. 수정한 코드 원본 -> 수정한 코드 결과본

### 3.1 원본 코드 (`warmup/orders.py`)

```python
"""주문 관리 유틸리티 - 버그 찾기 연습용 (고치지 말고 찾기만 하세요)."""

# 두 수를 더하는 함수
def add(a, b):
    """두 수를 더한다"""
    return a + b

def average_amount(amounts):
    """주문 금액의 평균을 구한다."""
    total = sum(amounts)  # 금액을 모두 더한다
    return total / len(amounts)  # 개수로 나눈다

def add_tag(order, tag, tags=[]):
    """주문에 태그를 붙인다."""
    tags.append(tag)  # 태그 목록에 추가
    order["tags"] = tags
    return order

def get_page(orders, page, size=10):
    """주문 목록에서 page번째(1부터 시작) 페이지를 돌려준다."""
    start = page * size  # 시작 위치
    end = start + size  # 끝 위치
    return orders[start:end]

# © 2026 김연지. All rights reserved. 수강생 개인 학습용 · 무단 강의·복제·배포·수정 금지 (LICENSE.md)
```

### 3.2 수정한 결과 코드 (`warmup/orders.py`)

```python
"""주문 관리 유틸리티 - 버그 찾기 연습용 (고치지 말고 찾기만 하세요)."""

# 두 수를 더하는 함수
def add(a, b):
    """두 수를 더한다"""
    return a + b

def average_amount(amounts):
    """주문 금액의 평균을 구한다."""
    total = sum(amounts)  # 금액을 모두 더한다
    return total / len(amounts)  # 개수로 나눈다

def add_tag(order, tag, tags=None):
    """주문에 태그를 붙인다."""
    if tags is None:
        tags = []
    else:
        tags = list(tags)
    tags.append(tag)  # 태그 목록에 추가
    order["tags"] = tags
    return order

def get_page(orders, page, size=10):
    """주문 목록에서 page번째(1부터 시작) 페이지를 돌려준다."""
    start = (page - 1) * size  # 시작 위치
    end = start + size  # 끝 위치
    return orders[start:end]

# © 2026 김연지. All rights reserved. 수강생 개인 학습용 · 무단 강의·복제·배포·수정 금지 (LICENSE.md)
```

### 3.3 코드 변경 비교 (Diff)

```diff
--- warmup/orders.py (원본)
+++ warmup/orders.py (수정본)
@@ -10,13 +10,17 @@
     total = sum(amounts)  # 금액을 모두 더한다
     return total / len(amounts)  # 개수로 나눈다
 
-def add_tag(order, tag, tags=[]):
+def add_tag(order, tag, tags=None):
     """주문에 태그를 붙인다."""
+    if tags is None:
+        tags = []
+    else:
+        tags = list(tags)
     tags.append(tag)  # 태그 목록에 추가
     order["tags"] = tags
     return order
 
 def get_page(orders, page, size=10):
     """주문 목록에서 page번째(1부터 시작) 페이지를 돌려준다."""
-    start = page * size  # 시작 위치
+    start = (page - 1) * size  # 시작 위치
     end = start + size  # 끝 위치
     return orders[start:end]
```

---

## 4. 왜 그렇게 수정했는지 (원인 및 해결 방안)

### ① `add_tag` 함수: 파이썬 가변 기본 인자(Mutable Default Argument) 버그 해결
- **문제 원인**:
  - 파이썬의 함수 정의 시 기본 인자(`tags=[]`)는 함수가 정의되는 시점(컴파일 타임)에 단 한 번만 평가되어 생성됩니다.
  - 따라서 함수를 여러 번 호출할 때 `tags` 인자를 전달하지 않으면 동일한 리스트 인스턴스를 계속 참조하게 되어, 이전 호출에서 추가한 태그가 다음 호출에까지 누적되는 부작용이 발생합니다 (`['new', 'tag1', 'tag2']`).
- **수정 내용**:
  - 기본값을 불변 객체인 `tags=None`으로 변경했습니다.
  - 함수 본문에서 `tags is None`일 경우 매 호출마다 새로운 빈 리스트(`tags = []`)를 생성하도록 처리하고, 외부에서 전달된 경우에도 복사본(`list(tags)`)을 만들어 독립성을 보장했습니다.

### ② `get_page` 함수: 1-Based 페이지네이션 오프셋 계산 버그 해결
- **문제 원인**:
  - 함수의 명세는 `page번째(1부터 시작) 페이지`를 요구합니다.
  - 기존 코드의 `start = page * size`는 0-based 오프셋에 적합한 수식입니다. 이로 인해 1페이지(`page=1, size=10`)를 요청했을 때 `start = 1 * 10 = 10`이 되어 0~9번 항목이 아닌 10~19번 항목이 반환되는 오프셋 오차가 발생했습니다.
- **수정 내용**:
  - 시작 인덱스를 `start = (page - 1) * size`로 수정했습니다.
  - `page=1, size=10`일 때: `start = (1 - 1) * 10 = 0`, `end = 10` → `[0:10]` (1페이지 10개)
  - `page=2, size=10`일 때: `start = (2 - 1) * 10 = 10`, `end = 20` → `[10:20]` (2페이지 10개)
  - `page=3, size=10`일 때: `start = (3 - 1) * 10 = 20`, `end = 30` → `[20:23]` (3페이지 잔여 항목)
  - 이로써 모든 페이지네이션 케이스가 정상 동작합니다.

---

## 5. 최종 테스트 통과 결과

```text
pytest warmup\test_orders.py -v
============================= test session starts =============================
platform win32 -- Python 3.11.9, pytest-9.1.1, pluggy-1.7.0
rootdir: C:\workspaces\review-thermo
configfile: pyproject.toml
plugins: anyio-4.15.1
collected 8 items

warmup/test_orders.py::test_average_amount_normal PASSED                 [ 12%]
warmup/test_orders.py::test_average_amount_empty_raises_zero_division PASSED [ 25%]
warmup/test_orders.py::test_add_tag_basic PASSED                         [ 37%]
warmup/test_orders.py::test_add_tag_mutable_default_argument_bug PASSED  [ 50%]
warmup/test_orders.py::test_get_page_pagination[1-10-expected0] PASSED   [ 62%]
warmup/test_orders.py::test_get_page_pagination[2-10-expected1] PASSED   [ 75%]
warmup/test_orders.py::test_get_page_pagination[3-10-expected2] PASSED   [ 87%]
warmup/test_orders.py::test_get_page_default_size PASSED                 [100%]

============================== 8 passed in 0.03s ==============================
```
