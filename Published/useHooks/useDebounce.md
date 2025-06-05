[사이트 링크](https://usehooks.com/usedebounce)

#### 한줄 요약
**Delay the execution of function or state update with useDebounce.**

함수의 실행 또는 상태의 업데이트를 지연시키는 훅

#### 설명 

useDebounce 훅은 입력 값에 대한 추가 변경 없이 지정된 시간이 경과할 때까지 함수나 상태 업데이트의 실행을 지연시키는 데 유용합니다. 이는 사용자 입력을 처리하거나 네트워크 요청을 트리거하는 등의 시나리오에서 특히 유용하며, 불필요한 계산을 효과적으로 줄이고 리소스 집약적인 작업이 입력 활동이 일시 중지된 후에만 수행되도록 보장합니다.

> 영어 원문
> The useDebounce hook is useful for delaying the execution of functions or state updates until a specified time period has passed without any further changes to the input value. This is especially useful in scenarios such as handling user input or triggering network requests, where it effectively reduces unnecessary computations and ensures that resource-intensive operations are only performed after a pause in the input activity.

#### 파라미터

| 이름    | 타입     | 설명                                  |
| ----- | ------ | ----------------------------------- |
| value | any    | 디바운스를 적용할 값                         |
| delay | number | 밀리세컨드 단위로 딜레이 시키며, 해당 시간 이후 최신값이 사용 |

#### 리턴 값

| Name           | Type | Description                                          |
| -------------- | ---- | ---------------------------------------------------- |
| debouncedValue | any  | 디바운스된 값이 반환됩니다. 값의 변경 없이 지연시간이 지나면  해당 값은 최신값으로 업데이트 |
