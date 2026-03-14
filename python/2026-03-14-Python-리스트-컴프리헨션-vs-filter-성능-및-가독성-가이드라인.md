### Context
* 파이썬에서 데이터 필터링 시 리스트 컴프리헨션(List Comprehension)은 간결함과 속도 덕분에 자주 사용되지만, 조건문이 중첩될 경우(`if a if b`) 코드 가독성이 급격히 떨어지는 문제가 발생합니다.
* 초보 개발자나 비전공자에게는 `filter()` 함수가 대안으로 보일 수 있으나, 대량 데이터 처리 환경에서는 두 방식 간의 성능 차이가 유의미하게 발생하므로 기술적 검증을 통한 가이드라인이 필요합니다.

### Core
* **성능 비교 예시 코드**
```python
import timeit

# 100만 개의 데이터 생성
data = range(1000000)

# 1. 리스트 컴프리헨션 (List Comprehension)
# 중첩 조건: [x for x in data if x % 2 == 0 if x % 3 == 0]
comp_stmt = "[x for x in data if x % 2 == 0 if x % 3 == 0]"
comp_time = timeit.timeit(comp_stmt, globals=globals(), number=100)

# 2. 필터 함수 (filter + lambda)
# filter(lambda x: x % 2 == 0 and x % 3 == 0, data)
filter_stmt = "list(filter(lambda x: x % 2 == 0 and x % 3 == 0, data))"
filter_time = timeit.timeit(filter_stmt, globals=globals(), number=100)

print(f"List Comprehension: {comp_time:.4f}s")
print(f"Filter + Lambda: {filter_time:.4f}s")
```
* **성능 결과**: 리스트 컴프리헨션이 `filter()` + `lambda` 조합보다 일반적으로 1.5배에서 2배가량 더 빠릅니다. 이는 `filter()`가 파이썬 수준에서 매번 함수(lambda)를 호출(Function Call Overhead)해야 하는 반면, 컴프리헨션은 내부 루프 내에서 최적화된 바이트코드로 실행되기 때문입니다.

### Insight
* **기술적 검증**: `filter()` 함수는 필터링 조건이 `None`(Truthiness 테스트)이거나 이미 정의된 C 언어 기반의 내장 함수를 사용할 때는 효율적입니다. 하지만 사용자 정의 `lambda`를 사용하는 순간 인터프리터의 함수 호출 비용이 발생하여 성능 저하의 원인이 됩니다.
* **프로그래밍 가이드라인**:
* 단순 필터링: 성능과 간결함을 위해 리스트 컴프리헨션 사용을 권장합니다.
* 복잡한 중첩 로직: 입문자나 비전공자가 포함된 팀 프로젝트에서는 가독성을 위해 `filter()`와 명명된 함수를 조합하거나, 차라리 명시적인 `for` 루프를 사용하는 것이 유지보수에 유리합니다.
* 대규모 데이터셋: 수만 건 이상의 데이터를 실시간으로 처리해야 한다면 가독성보다 성능을 우선하여 컴프리헨션 또는 `Generator Expression`을 선택해야 합니다.

**출처:** [Python Official Docs - List Comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)
**출처:** [Real Python - Python filter() Function: Filtering Iterables](https://realpython.com/python-filter-function/)
**출처:** [Python Speed - Performance Tips on Loops](https://wiki.python.org/moin/PythonSpeed/PerformanceTips#Loops)