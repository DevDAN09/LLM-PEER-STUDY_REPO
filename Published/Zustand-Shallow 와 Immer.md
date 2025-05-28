### 문제 상황 정의

1. Zustand에서 Immer를 사용할 경우 불변성 보장을 위해서 새로운 객체를 제공한다.
2. 새로운 상태 객체를 전달 받은 항상 리렌더링이 발생한다

### Immer의 동작 방식
- Immer의 produce 함수는 상태를 변경할 때, 실제로 변경된 부분만 새로운 참조를 가진 객체를 반환함
- **변경이 없는 경우**: Immer는 **원본 객체**를 그대로 반환합니다(참조 동일). 이는 Immer의 최적화 기능 포함됨
- **변경이 있는 경우**: 변경된 내용에 따라 새로운 객체가 생성되며, 해당 객체의 참조는 다름 -> 랜더링 발생


### Zustand에서 리랜더링을 방지하는 방법
1. 객체를 구독하지 말고, 값을 구독하기
```
const value = useStore((state) => state.data.value);
```
2. zustand/shallow 
```
import shallow from 'zustand/shallow'; const { data } = useStore((state) => ({ data: state.data }), shallow);
```
### useStore/shallow 의 동작방식

기본 동작 방식 : 레퍼런스 비교
shallow 동작 방식 : 얇은 비교

**얇은 복사?**
보통 C언어에서 생각하는 얇은 복사의 경우 새로운 메모리 공간을 할당하고 해당 메모리에 데이터를 복사하는 깊은 복사의 반대 개념을 이해하고 있다면 Reference copy를 생각하지만 JS 생태계에서는 다른 복사를 의미한다 **객체의 최상위 값 (top-level)** 비교가 핵심

> C와 JS의 차이
    - **C**: 얕은 복사는 주로 포인터가 가리키는 힙 메모리를 공유하며, 메모리 관리(할당/해제)가 개발자의 책임. 이로 인해 이중 해제 같은 심각한 오류 가능.
    - **JavaScript**: 얕은 복사는 중첩 객체의 참조를 공유하며, GC가 메모리를 자동 관리하므로 이중 해제 문제는 없지만, 중첩 객체 수정 시 부작용 발생.
	- **공통점**: 둘 다 최상위 데이터만 복사하고, 내부 참조(포인터 또는 객체 참조)는 공유.

-> 결론적으로 useStore는 shallow 를 적용할 경우, 해당 상태의 리렌더링 방지의 여부를 결정하는 기준을 reference comperison 에서 value comperison으로 바뀌도록 한다.